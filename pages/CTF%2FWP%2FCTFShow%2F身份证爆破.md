- 因为验证位的存在，身份证爆破实际上花不了多长时间
## 题解

进入题目，发现录取名单、学号查询界面和登录界面

下载录取名单得到学生的姓名和身份证的一部分

爆破身份证星号部分，得到学号

使用学号和身份证号登陆即可
## 知识点

遍历某个月的每一天

```python
import calendar

cl = calendar.Calendar()

for i in cl.itermonthdates(year, month):
print(i.strftime('%Y%m%d'))

```

验证身份证

```python

from id_validator import validator

validator.is_valid('xxxxxxxxxxxxxxxxxx')
```
-