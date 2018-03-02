# The Secret of Sunny Running

## Warning
&emsp;&emsp;本文仅供研究，使用者造成的任何后果由使用者自行承担，与作者无关。

&emsp;&emsp;本文研究成果源于对iOS版Sunny Running的逆向分析。

## License
&emsp;&emsp;GPL v3

## 将要使用到的一些关键值或函数

### &emsp;API_ROOT
&emsp;&emsp;api根地址，http(s)://client4.aipao.me/api

### &emsp;UUID
&emsp;&emsp;iOS端是由CFUUID组件生成的通用唯一识别码，不重装APP时保持不变。

### &emsp;IMEICode
&emsp;&emsp;首次登陆后得到的类似Token的标识，用于后期登陆，生命周期较长。

### &emsp;token
&emsp;&emsp;每次登录后由服务器发送给客户端，用于标记本次登陆，生命周期猜测20min左右。

### &emsp;userid
&emsp;&emsp;跑步者在爱跑后台中的id，登陆后由服务器发送给客户端，与请求头中的签名有关。

### &emsp;timespan和nonce
&emsp;&emsp;这两个参数位于请求头中，用于验证请求。

&emsp;&emsp;timespan为19位时间戳，可用13位时间戳+6位随机数模拟。

&emsp;&emsp;nonce为随机整数，猜测范围-1999999999~1999999999。

### &emsp;auth和sign
&emsp;&emsp;这两个参数位于请求头中，用于验证请求。

&emsp;&emsp;每处请求中auth和sign算法不尽相同，将在稍后请求流程解析中分析。

### &emsp;MD5
&emsp;&emsp;MD5加密函数，格式一律为32位大写。

## 请求流程解析，变量用(xxx)表示。

### &emsp;微信登录（第一次登录）
&emsp;&emsp;API_ROOT/(null)(此处非变量，就是"(null)")/QM_Users/Login?wxCode=(微信的OAuth字段)&IMEI=(UUID)

&emsp;&emsp;此请求无法模拟，仅用于抓取IMEICode。

&emsp;&emsp;请求头中有一处签名auth。

&emsp;&emsp;auth = "S1" + MD5(MD5(UUID) + ":" + "(null)")

### &emsp;IMEICode登录
&emsp;&emsp;微信登录后服务器会回传IMEICode，有效期较长，用于长期登录。

&emsp;&emsp;API_ROOT/token/QM_Users/LoginSchool?IMEICode=(你的IMEICode)

&emsp;&emsp;请求头中有一处签名auth。

&emsp;&emsp;auth = "S1"+ MD5(MD5(UUID) + ":" + (上次的token))

### &emsp;获取用户信息
&emsp;&emsp;API_ROOT/(token)/QM_Users/GS

&emsp;&emsp;请求头中有一处签名auth。auth = "C"+(密文)

&emsp;&emsp;加密方式为aes-128-cbc-pkcs5，明文为当前请求时间（mmss）+UUID。

&emsp;&emsp;key:osldaaasmkldospd&emsp;&emsp;gIv:0392030003920392

&emsp;&emsp;此处(token)为上文提到的token，登录后获取，下同。

### &emsp;开始跑步
&emsp;&emsp;API_ROOT/(token)/QM_Runs/SRS?S1=(起始纬度)&S2=(起始经度)&S3=(要求长跑距离，男生2000，女生1600)

&emsp;&emsp;请求头中有两处签名auth和sign。

&emsp;&emsp;auth = "B" + MD5(MD5(UUID) + ":;" + token)

&emsp;&emsp;sign = MD5(token + nonce + timespan + userid)

&emsp;&emsp;起始纬度建议范围：30.534485~30.535127  (信部操场)

&emsp;&emsp;起始经度建议范围：114.366687~114.367427  (信部操场)

&emsp;&emsp;请求成功后服务器会回传一个runid，之后的请求会用到。

### &emsp;结束跑步

&emsp;&emsp;首先生成一个不重复10位小写字母字符串，然后分别替换下面请求参数中的0~9数字。

&emsp;&emsp;用enc函数表示上面的处理过程。

&emsp;&emsp;API_ROOT/(token)/QM_Runs/ES?S1=(runid)&S2=(enc(5000))&S3=(enc(2000))&S4=(enc(跑步时间，单位s))&S5=(enc(跑步距离，单位m))&S6=&S7=1&S8=(刚才生成的小写字母串)&S9=(enc(步数))

&emsp;&emsp;请求头中有两处签名auth和sign。

&emsp;&emsp;auth = "B" + MD5(MD5(UUID) + ":;" + token)

&emsp;&emsp;sign = MD5(token + nonce + timespan + userid)

## Author
&emsp;&emsp;Jason, iOS Developer & Cracker
