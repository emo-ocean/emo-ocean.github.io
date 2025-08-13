# Computer Network SMTP Lab
### email：zehaiyu589@gmail.com （源码文章最后附有）
### programming assignment : [SMTP.pdf](https://github.com/user-attachments/files/16276898/SMTP.pdf)
### programming assignment and others from 🥇 https://media.pearsoncmg.com/ph/esm/ecs_kurose_compnetwork_8/cw/

## 程序大致流程 👍 

> 建立TCP连接。建立连接后服务器将返回状态码220。username填写QQ邮箱，password并非QQ密码，而是校验码，代码之后会讲解获取相关方法。

```python
mailserver = "smtp.qq.com"
mailPort = 25
username = "   "
password = "   "
clientSocket = socket(AF_INET, SOCK_STREAM)
clientSocket.connect((mailserver,mailPort))

recv = clientSocket.recv(1024).decode()
print(recv)
if recv[:3] != '220':
    print('220 reply not received from server.')
```

> 发送"HELO"命令，开始与服务器的交互，服务器将返回状态码250（请求动作正确完成）。
```python
heloCommand = 'HELO \r\n'
clientSocket.send(heloCommand.encode())
recv1 = clientSocket.recv(1024).decode()
print(recv1)
if recv1[:3] != '250':
    print('250 reply not received from server.')
```

> 发送"AUTH LOGIN"命令，开始验证身份，服务器将返回状态码334（服务器等待用户输入验证信息），此处需要base64加密。

```python
loginCommand = "auth login\r\n"
clientSocket.send(loginCommand.encode())
recv0 = clientSocket.recv(1024).decode()
print(recv0)
if recv0[:3] != '334':
    print('334 reply not received from server.')
clientSocket.send(base64.b64encode(username.encode('utf-8')))
clientSocket.send("\r\n".encode())
clientSocket.send(base64.b64encode(password.encode('utf-8')))
clientSocket.send("\r\n".encode())
```

> mail from & mail to

```python
fromCommand = "MAIL FROM: <sender@qq.com>\r\n"
clientSocket.send(fromCommand.encode())
recv2 = clientSocket.recv(1024).decode()
print(recv2)
if recv2[:3] != '250':
    print('250 reply not received from server.')

rcptCommand = "RCPT TO: <receiver@qq.com>\r\n"
clientSocket.send(rcptCommand.encode())
recv3 = clientSocket.recv(1024).decode()
print(recv3)
if recv3[:3] != '250':
    print('250 reply not received from server.')
```

> 发送"DATA"命令，表示即将发送邮件内容，服务器将返回状态码354（开始邮件输入，以"."结束）

```python
dataCommand = "DATA\r\n"
clientSocket.send(dataCommand.encode())
recv4 = clientSocket.recv(1024).decode()
print(recv4)
if recv4[:3] != '354':
    print('354 reply not received from server.')
```

> 发送邮件内容并quit断开与邮件服务器的连接。

```python
quitCommand = "QUIT\r\n"
clientSocket.send(quitCommand.encode())
recv5 = clientSocket.recv(1024).decode()
print(recv5)
if recv4[:3] != '221':
    print('221 reply not received from server.')
```

## bug list

###  1.身份认证失败：535 Login Fail. Please enter your authorization code to login.

> 官网查询后：

![image](https://github.com/user-attachments/assets/c450cc41-c56b-41c0-a0c7-e1459f7760d9)

> 登录邮箱，右上角setting->general->third party service（非谷歌浏览器按照左上角：设置-用户-第三方服务）并启用SMTP第三方服务，验证后得到16位校验码，替换为password即可
![@3WCM9R`GY G`TVTMBGXQJD](https://github.com/user-attachments/assets/3c3db3be-f0e9-480e-ab46-55235d36df47)


### 2.From格式：550 The "From" header is missing or invalid. Please follow RFC5322, RFC2047, RFC822 standard protocol.

>报错参考： [https://blog.csdn.net/qq_41158337/article/details/130193810](url)

> RFC5322关于from字段的格式和用途：

![image](https://github.com/user-attachments/assets/d037dd44-559e-4bc8-9345-03f3a9ecaca7)

> RFC2047第 2 和第 5 节描述了如何对邮件标头中的非 ASCII 字符进行编码。虽然这些章节并未直接讨论 From 字段，但适用于所有邮件标头字段。

![image](https://github.com/user-attachments/assets/19622ddd-5af0-46b2-9d42-bbaf9259560a)

> 由此根据RFC制定相关headers

```python
headers = "From: <2983643426@qq.com>\r\n"
headers += "To: <2983643426@qq.com>\r\n"
headers += "Subject: Test Email\r\n"
headers += "MIME-Version: 1.0\r\n"
headers += "Content-Type: text/plain; charset=\"utf-8\"\r\n"
headers += "Content-Transfer-Encoding: base64\r\n"
clientSocket.send(headers.encode())
```
> 至此成功发送邮件，程序最终使用了465加密端口并对代码进行了优化
![image](https://github.com/user-attachments/assets/69c51506-3b32-4df7-b8f7-9dae44e9df1c)


> source code: 注意替换你的邮箱sender@qq.com和收件人receiver's mail，并将你的16位校验码填入password

```python
from socket import *
import ssl
import base64

msg = "\r\n Sending successfully!"
endmsg = "\r\n.\r\n"

# Choose a mail server (e.g. Google mail server) and call it mailserver
mailserver = "smtp.qq.com"
# Create socket called clientSocket and establish a TCP connection with mailserver
# mailPort = 25
mailPort = 465
username = " yours@qq.com"
password = "   "
clientSocket = socket(AF_INET, SOCK_STREAM)
clientSocket.connect((mailserver,mailPort))

# Wrap the socket with SSL
clientSocket = ssl.create_default_context().wrap_socket(clientSocket, server_hostname=mailserver)

#send message about tcp connection with mailserver
recv = clientSocket.recv(1024).decode()
print(recv)
if recv[:3] != '220':
    print('220 reply not received from server.')


# Send HELO command and print server response.
heloCommand = 'HELO QQ\r\n'
clientSocket.send(heloCommand.encode())
recv1 = clientSocket.recv(1024).decode()
print(recv1)
if recv1[:3] != '250':
    print('250 reply not received from server.')

loginCommand = "AUTH LOGIN\r\n"
clientSocket.send(loginCommand.encode())
recv0 = clientSocket.recv(1024).decode()
print(recv0)
if recv0[:3] != '334':
    print('334 reply not received from server.')

# send username
clientSocket.send(base64.b64encode(username.encode('utf-8')))
clientSocket.send("\r\n".encode())
print(clientSocket.recv(1024).decode())

#send password
clientSocket.send(base64.b64encode(password.encode('utf-8')))
clientSocket.send("\r\n".encode())
print(clientSocket.recv(1024).decode())

# Send MAIL FROM command and print server response.
fromCommand = "MAIL FROM: <yours@qq.com>\r\n"
clientSocket.send(fromCommand.encode())
recv2 = clientSocket.recv(1024).decode()
print(recv2)
if recv2[:3] != '250':
    print('250 reply not received from server.')

# Send RCPT TO command and print server response.
rcptCommand = "RCPT TO: < receiver's mail  >\r\n"
clientSocket.send(rcptCommand.encode())
recv3 = clientSocket.recv(1024).decode()
print(recv3)
if recv3[:3] != '250':
    print('250 reply not received from server.')

# Send DATA command and print server response.
dataCommand = "DATA\r\n"
clientSocket.send(dataCommand.encode())
recv4 = clientSocket.recv(1024).decode()
print(recv4)
if recv4[:3] != '354':
    print('354 reply not received from server.')

# Send message data.
# clientSocket.send(msg.encode())
# Send message data with headers

headers = "From: <sender@qq.com>\r\n"
headers += "To: <receiver's mail >\r\n"
headers += "Subject: Emo Ocean Test Email\r\n"
headers += "MIME-Version: 1.0\r\n"
headers += "Content-Type: text/plain; charset=\"utf-8\"\r\n"
headers += "Content-Transfer-Encoding: base64\r\n"
clientSocket.send(headers.encode())

# Encode the message content
encoded_msg = base64.b64encode(msg.encode('utf-8'))

# Send the encoded message
clientSocket.send(encoded_msg + b"\r\n")

# Message ends with a single period.
clientSocket.send(endmsg.encode())

# Send QUIT command and get server response.
quitCommand = "QUIT\r\n"
clientSocket.send(quitCommand.encode())
recv5 = clientSocket.recv(1024).decode()
print(recv5)
if recv4[:3] != '221':
    print('221 reply not received from server.')

clientSocket.close()
```



