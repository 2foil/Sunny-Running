# The Secret of Sunny Running

## Warning
&emsp;&emsp;本文仅供研究，使用者造成的任何后果由使用者自行承担，与作者无关。

&emsp;&emsp;本文研究成果源于对iOS版Sunny Running的逆向分析。

## License
&emsp;&emsp;GPL v3

## 将要使用到的一些关键值

### &emsp;API_ROOT
&emsp;&emsp;api根地址，http(s)://client4.aipao.me/api
### &emsp;UUID
&emsp;&emsp;应该是由微信登录生成的通用唯一识别码，可以通过抓包获取，请求为Login，UUID即为Login的IMEI参数。

&emsp;&emsp;同一台设备重新微信登录UUID不变，不同设备未测试。

### &emsp;token
&emsp;&emsp;登录后由服务器发送给客户端，生命周期猜测20min左右。

### &emsp;userid
&emsp;&emsp;登录后由服务器发送给客户端，用于标记runner，此值不变。

### &emsp;timespan和nonce
&emsp;&emsp;这两个参数位于请求头中，用于验证请求。

&emsp;&emsp;timespan为19位时间戳，可用13位时间戳+6位随机数模拟。

&emsp;&emsp;nonce为随机整数，猜测范围-1999999999~1999999999。

## 请求头签名解析（MD5值中字母大写）
&emsp;&emsp;请求头中有两处签名，分别为auth和sign

### &emsp;auth
* auth = 'B' + MD5(MD5(UUID) + ':;' + token)

### &emsp;sign
* sign = MD5(token + nonce + timespan + userid)

## 请求流程解析

&emsp;&emsp;登录和获取用户信息的请求头不需要加auth、sign、timespan和nonce，开始跑步和结束跑步需要添加。

&emsp;&emsp;变量用(xxx)表示。

### &emsp;微信登录（第一次登录）
&emsp;&emsp;API_ROOT/token/QM_Users/Login?wxCode=(微信的OAuth字段)&IMEI=(UUID)

&emsp;&emsp;此请求无法模拟，仅用于抓取UUID。

### &emsp;IMEICode登录
&emsp;&emsp;微信登录后服务器会回传IMEICode，有效期较长，用于长期登录。

&emsp;&emsp;API_ROOT/token/QM_Users/LoginSchool?IMEICode=(你的IMEICode)

&emsp;&emsp;此请求头中有auth字段，涉及到上次跑步的token，但是可省略，暂不清楚作用。

### &emsp;获取用户信息
&emsp;&emsp;API_ROOT/(token)/QM_Users/GS

&emsp;&emsp;此处请求头中有一个auth字段，没有填写也可请求，暂不清楚其作用。加密方式为aes-128-cbc-pkcs5，明文为上次跑步结束时间（mmss）+UUID。出于安全考虑不公开密钥和偏移量。

&emsp;&emsp;此处为上文提到的token，登录后获取，下同。

### &emsp;开始跑步
&emsp;&emsp;API_ROOT/(token)/QM_Runs/SRS?S1=(起始纬度)&S2=(起始经度)&S3=(要求长跑距离，男生2000，女生1600)

&emsp;&emsp;起始纬度建议范围：30.534485~30.535127

&emsp;&emsp;起始经度建议范围：114.366687~114.367427

&emsp;&emsp;请求成功后服务器会回传一个runid，之后的请求会用到。

### &emsp;结束跑步

&emsp;&emsp;首先生成一个不重复10位小写字母字符串，然后分别替换下面请求参数中的0~9数字。

&emsp;&emsp;用enc函数表示上面的处理。

&emsp;&emsp;API_ROOT/(token)/QM_Runs/ES?S1=(runid)&S2=(enc(5000))&S3=(enc(2000))&S4=(enc(跑步时间，单位s))&S5=(enc(跑步距离，单位m))&S6=&S7=1&S8=(刚才生成的小写字母串)&S9=(enc(步数))

## Author
&emsp;&emsp;Jason, iOS Developer & Cracker
