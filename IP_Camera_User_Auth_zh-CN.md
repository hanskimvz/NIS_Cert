# 用户识别、登录、会话相关事项

本文档摘录自**IP摄像机安全要求V3.0**中与登录（识别及认证）相关的要求事项。

## 摘要检查清单

| 项目 | 要求事项 | 条款 | 满足与否 |
|:---:|:---|:---:|:---:|
| ✅ | 基于用户账户·密码的识别及认证 | 2.2.1.1 | 满足 |
| ✅ | 连续认证失败时停用（5次以下） | 2.2.2.1 | 满足 |
| ✅ | 管理员认证失败时通知             | 2.2.2.2 | ?? |
| ✅ | 密码9位以上 + 4种字符组合   | 2.2.3.1 | 需改进 |
| ✅ | 防止认证信息重用（会话管理）      | 2.2.4.1 | 满足 |
| ✅ | 密码输入时遮蔽（✽）         | 2.2.5.1 | 满足 |
| ✅ | 认证失败时不提供具体原因       | 2.2.5.2 | 满足 |
| ✅ | 强制变更·停用默认账户         | 2.3.4.1 | ?? |
| ✅ | 强制变更·创建默认密码         | 2.3.4.2 | ?? |
| ✅ | 会话超时（10分钟以下）           | 2.7.1.1 | 满足 |
| ✅ | 禁止重复连接                     | 2.7.1.2 |
| ✅ | 密码salt + hash存储          | 2.4.2.1 | ?? |
| ✅ | 视频协议认证（ONVIF, RTSP）    | 2.1.1.1 |
| ✅ | 生成登录相关审计记录            | 2.8.1.1 |

---

## 1. 用户识别及认证

### 2.2.1.1 (必须) 基于用户账户·密码的识别及认证
**原文:** "产品必须提供基于用户账户·密码的识别及认证功能，以验证用户身份。"

- 为确认用户是产品的合法用户，必须执行识别及认证。
- 管理员必须能够为每个用户或用户组授予权限。
- 用户账户(ID)必须注册为唯一值，不得重复。
> 全部满足  
> 本设备使用Challenge Response方式。  
> 在Challenge阶段（第1阶段），将用户名（userName）发送到设备，设备返回包含nonce值（random，10位数）的信息。  
> 前端将返回的nonce值与realm（登录信息）、Password组合并加密后发送到设备，设备返回可使用1分钟的session，之后所有通信都通过此session进行。
```
export function calculateLoginPassword(username, password, realm, random) {
  const response = md5(`${username}:${random}:${md5(`${username}:${realm}:${password}`)}`)
  return response
}
```
---

## 2. 认证失败应对

### 2.2.2.1 (必须) 连续认证失败时停用识别及认证功能
**原文:** "当产品中用户认证连续失败达到设定次数时，识别及认证功能必须被停用。"

- 导致识别及认证被停用的连续认证失败次数必须固定或可设置为**5次以下**的值。
- 如果实现为在一定时间内停用认证功能，重新激活所需的时间必须固定或可设置为**5分钟以上**的值。
- 激活方法示例：账户锁定后在指定时间过后激活、账户锁定后提供其他识别及认证手段进行激活等
> 我们限制登录5分钟。  
> 统一显示"账户已锁定。请在%d秒后重试"的消息。
![login_limit](images/login_try_n5.png)
![login_limit](images/login_try_expired.png)


### 2.2.2.2 (必须) 管理员认证失败时立即通知
**原文:** "当产品中管理员认证连续失败达到设定次数时，必须通过管理员可立即确认的手段进行通知。"

- 必须通过报警、短信、电子邮件等**一种以上**的手段进行通知。
> 提供此功能。通过email提供。  
> 需要在设备初始化阶段能够设置email。

---

## 3. 密码安全性标准

### 2.2.3.1 (必须) 密码注册及变更时满足安全性标准
**原文:** "产品在密码注册及变更时必须满足<表1>的安全性标准。"

#### 遵守事项
| 项目 | 要求事项 | 备注 |
|:---:|:---|:---|
| 最小长度 | **9位以上** | 必须 |
| 包含大写字母 | 包含1个以上英文大写字母 | A-Z |
| 包含小写字母 | 包含1个以上英文小写字母 | a-z |
| 包含数字 | 包含1个以上数字 | 0-9 |
| 包含特殊字符 | 包含1个以上特殊字符 | !@#$%^&*() 等 |
| 复杂度 | **必须混合使用**以上4种类型 | 必须 |

#### 禁止事项
| 项目 | 要求事项 | 示例 |
|:---:|:---|:---|
| 禁止包含ID | 密码中禁止包含与用户ID相同的字符串 | 如果ID是`admin`，则禁止`admin123!` |
| 连续相同字符 | 禁止连续使用相同的字符或数字 | 禁止`aaa`、`111`、`!!!` |
| 顺序字符 | 禁止键盘连续字符顺序排列 | 禁止`qwerty`、`asdf`、`1234` |
| 顺序数字 | 禁止连续数字顺序排列 | 禁止`123456`、`987654` |
| 以前的密码 | 禁止重用上次使用的密码 | 至少管理1次以上的历史记录 |

![add_user](images/add_user_n1.png)
> 额外改进事项：密码必须为8~32个字符，至少包含数字、大写字母、小写字母和特殊字符中的两种（不能包含' " ; : &）。 \
> => <u>密码必须为<font color="red">9~32个字符，必须包含</font>数字、大写字母、小写字母和特殊字符。</u>

> 添加用户时 
> payload:
``` 
{
    "method": "Security.addUserPlain",
    "params": {
        "cipher": "RPAC-256",
        "salt": "40c5a2b958132dd93d98e01385014d75ef15f0118c7b18472ccd0e0221c0a6df8721e57461cbb24f05c6e3f36d46ebfec5e8ee94d05cab8f1577ce8f27b065848335c4238eb005f6ce7d05b300e87144747e9de1eb2c24d19e4961fa586761c804b4fa009cb80187dea998da478d98afbbaeaafd9e8753d09a69221490879b627ffde50e8f9104f32c4b1152f8db7024f0412d538ec4e90ab68e1b74f236253c98cb521aa8ab6c48931a184b93e8cc92fd5b1bb24fc4564a1fd0e9cc809bc2a609bb247481b0a8b8befe82a0453e3da73866f729fbba1ad4859ba65e1a37fc3dbc89248bd0e76e63bcbf2865c17888df5d8558beebb807c5384184e7dfb3e230",
        "content": "7OH4rA0G4AgXLvivESVqU9rmqQ3cumIotXa/mJkjHcf/e9YllPKoO4Q5M09cEaQc7s4D5lie2lTb+4N4ur7k36xrNA76yfcujvbZG5oHzn2bE0h8lASSjnVLbrpT/WJaSGFj9wj6uqTbf9uuHkJBPo1v0+ghB8lzF4RAgzYvpRDxN9PHa6mHEtXV2PdMu7Xpl8d3yqa8jy97E/4qb5hGT6y9HMEm0sdqHPnhyCkT7M92E1WWEj8V+4xEUUqiWFWzTDp8Q99Qwy8ANws6rQqefJtHslNGHOuAhqJ68hl/Ro+E6ndBn0g3Qw9dA4HtvEejVaVRk8++qmY4lMZoJptHbDiQf81HzZ2nD6wf5kJHSt362haLyLZjsTVnW3wzakKFBLPBUhPBoG9fKeacvmMIhg=="
    },
    "id": 237,
    "session": "fc4b8e9722218f06aab2fd944080f77c"
}
```
> response:
```
{
    "id": 237,
    "params": {
        "content": "Tv5EjRzzLVq2/O3lntAZC3bTDGWwfdECx7KXoOxrlq0="
    },
    "result": true,
    "session": "fc4b8e9722218f06aab2fd944080f77c"
}
```
> 这样在payload的params.contents中以变换的信息（不暴露ID和password）进行传输。




---

## 4. 防止认证信息重用

### 2.2.4.1 (必须) 防止认证信息重用
**原文:** "产品必须防止用户认证信息的重用。"

- 可以通过加密会话ID或保证会话ID的唯一性（时间戳、会话过期时间设置等）来防止。
- 当检测到被禁止重用的认证信息的重用尝试时，**认证必须失败**，并且必须**生成认证失败事件的审计记录**。
- 会话过期时间必须设置为考虑到提供服务特性的**最小化**值。
> session由设备提供，nonce（random）基于timestamp生成，保证唯一性。   
> 会话过期时间默认为60秒。  
> 如果使用已使用的session、已过期的session、无效的session尝试连接，将记录到日志中。（时间、类型、日志内容：地址）

---

## 5. 认证反馈保护

### 2.2.5.1 (必须) 认证信息输出时不显示内容
**原文:** "产品在输出设备上显示认证使用的信息时，不得显示内容。"

- 认证使用的信息必须以不显示输入内容、用"✽"代替输入字符等形式输出。
- 用户登录时认证信息**不得以明文形式暴露在内存区域**。
> 满足要求

### 2.2.5.2 (必须) 认证失败时不提供失败原因
> 原文: "产品在识别及认证失败时，不得提供关于失败原因的反馈（不存在的账户(ID)、密码错误等）。"

- 因输入错误的认证信息导致认证失败后，提示消息中不得提供可推测认证失败原因的反馈
- 示例：仅显示"登录失败。"（不提供账户不存在、密码错误等具体原因）
> 改进要求：仅输出"登录失败"  
> api响应中显示失败原因。
```
{
    "error": {
        "code": 268632085,
        "message": "Component error: User or password not valid!"
    },
    "id": 5,
    "params": {
        "remainLockSecond": 0,
        "remainLoginTimes": 3
    },
    "result": false,
    "session": 76594003
}
```
> 将error.message的"Component error: User or password not valid!"修改为仅显示"Component error: Login failed"

---

## 6. 默认账户及密码管理

### 2.3.4.1 (必须) 强制变更·停用默认提供的账户
**原文:** "产品必须在首次产品访问（Web浏览器访问等）时提供强制变更·停用默认提供账户的功能。"

- 首次访问时，必须在屏幕上显示默认提供的账户。
- 默认提供的账户必须在首次访问时**强制变更或停用(Disable)**。
- 如果没有默认提供的账户，**必须创建新账户**，之后必须能够进行产品的管理访问。
- 变更账户或创建新账户时，**不得允许可推测的名称（root、admin、公司名、摄像机型号等）**。
> 目前在init状态（首次访问）下设置admin的ID。 
> 根据说明内容，似乎意味着不是admin，而是应该直接输入用户（admin角色）的ID。 
![first_init](images/first_5.png)

### 2.3.4.2 (必须) 强制变更·创建默认(default)密码
**原文:** "产品必须在首次产品访问（Web浏览器访问等）时提供强制变更·创建管理员默认(default)密码的功能。"

- 如果存在默认(default)密码，首次产品访问时必须提供**变更默认(default)密码的功能**。
- 如果没有默认(default)密码，**必须创建新密码**。
- 密码必须遵守**2.2.3.1的安全性标准**。
- 产品的**所有功能只有在默认(default)密码变更或创建完成后才能**运行。

> ![first_init](images/init_n3.png)
> 没有默认密码，只需提供新建功能即可。

---

## 7. 会话管理

### 2.7.1.1 (必须) 会话锁定·终止功能
**原文:** "产品必须提供在管理员会话连接后一定时间内未使用时锁定或终止会话的功能。"

- 一定时间可由管理员固定或设置为**10分钟以下**的值。
- 锁定的会话在锁定时间过后，必须由管理员或通过各会话的用户认证功能解除。
- 会话锁定或终止功能运行时必须**生成审计记录**。
- 在视频监控功能中，当一定时间过后尝试切换到其他管理功能时，**必须要求用户认证**。

> 默认的session过期时间为60秒。   
> 前端可以请求keep-alive来延长session过期时间。 
> 每次鼠标动作时都请求会话延长。最后一次鼠标动作后1分钟，session将过期。

### 2.7.1.2 (必须) 禁止重复连接
**原文:** "产品不得允许使用相同的管理员账户或相同权限重复连接产品。"

- 用户登录后，如果从其他终端使用相同账户进行登录，**阻止新连接**或**要求终止之前的连接**。
- 不得允许使用相同权限重复登录。
- 阻止重复连接时必须**生成审计记录**。
- 对于以视频监控·PTZ等摄像机控制为目的连接的管理员账户及权限，可以不适用。
> 这部分似乎没有实现。不能对同一用户重复发放会话。  
> 同一用户使用不同会话连接也应被阻止，并记录到日志中。
![same_user_sessions](images/same_user_sessions.png)

---

## 8. 存储数据保护（密码存储）

### 2.4.2.1 (必须) 密码加密存储
**原文:** "将重要信息存储在产品内部时，必须按规定方式存储。"

- 产品用于用户识别及认证的密码必须**加密存储**。
- 用户密码必须使用**单向加密（哈希）或双向加密**进行存储。
- 执行单向加密时，需要在密码中添加**salt**这个随机生成的值。
  - salt值不需要保密，使用随机数生成器生成
  - salt大小必须**至少48bit以上**。
  - iteration count必须应用尽可能大的值。（**至少1000次以上**）
- 需要加密存储的密码及加密密钥**不能硬编码存储在产品中**。
> 这部分需要要求厂商按照要求进行存储。
> 目前salt似乎是256字节。需要了解实际存储的salt是什么样的。
---

## 9. 视频协议认证

### 2.1.1.1 (必须) 在视频协议中提供用户认证
**原文:** "在设备间联动及视频相关标准协议（ONVIF、RTSP等）中必须提供用户认证功能。"

- 在ONVIF、RTSP等标准协议中提供用户认证
- 使用Digest认证时遵守**RFC 7616**
- 加密算法安全强度**112bit以上**
> 计划使用RTSP over TLS。使用Digest  
> 需要按要求修改。


---

## 10. 登录相关审计记录

### 2.8.1.1 (必须) 登录相关审计事件记录
**原文:** "产品必须为主要审计事件生成审计记录。"

必须记录的登录相关审计事件：

| 分类 | 审计事件 | 实现与否|
|:---:|:---|:--:|
| 识别及认证 | 用户登录、登出 | ✅|
| 识别及认证 | 用户注册、变更、删除 |✅|
| 识别及认证 | 用户认证尝试达到限制时的应对行动 |
| 识别及认证 | 密码的所有变更 |✅ 用户修改？|
| 安全管理    | 默认账户(ID)·密码变更 |
| 安全管理    | 管理终端连接IP阻止 |
| 会话管理   | 用户会话锁定或会话终止 |X|
| 会话管理   | 检测到同一账户重复登录尝试时的应对行动 |X|
| 会话管理   | 基于同一会话数限制拒绝新会话 |X|

---

## 参考：当前实现的认证方式（Challenge-Response）登录

当前摄像机使用**将标准HTTP Digest认证包装成JSON-RPC形式的Challenge–Response基础用户认证**方式。

### 第1阶段：Login Challenge（挑战请求）
```
api: https://192.168.1.108/RPC2_Login, POST
payload: {
  "method": "global.login",
  "params": {
      "userName": "admin",
      "password": "",
      "clientType": "Web5.0"
  },
  "id": 4
}
```

响应字段：
| 字段 | 含义 |
|:---:|:---|
| `realm` | 认证域标识符（Digest标准） |
| `random` | **nonce**（一次性随机数） |
| `opaque` | 服务器状态保护令牌 |
| `qop` | quality of protection（`auth`） |
| `authorization` | 服务器内部验证用 |
| `session` | 认证前临时会话 |

### 第2阶段：认证响应
```
password字段计算：
export function calculateLoginPassword(username, password, realm, random) {
  const ha1 = md5(`${username}:${realm}:${password}`)
  const response = md5(`${username}:${random}:${ha1}`)
  return response
}
```

**核心**：摄像机绝不接收明文密码，在第1阶段下发nonce（random），在第2阶段通过Digest哈希进行验证的结构。


## 参考：当前实现的认证方式（Challenge-Response）Websocket

连接到websocket后，通过该连接的socket以challenge response方式进行认证，
认证通过后才能接收视频。
```
Request:
GET /live/realmonitor.xav?channel=1&subtype=0&method=0 HTTP/1.1
Accept-Sdp: Private
Authorization: WSSE profile="UsernameToken"
Connection: keep-alive
Cseq: 0
Host: 192.168.1.108:8086
Range: npt=0.000000-
WSSE: UsernameToken Username="admin", PasswordDigest="M23Dqy2txm6nduULaopvZ1qb4Gs=", Nonce="EFA5A9EC57A437755FDE391B657561EA", Created="2025-12-23T19:13:03Z"


Response:
HTTP/1.1 401 Unauthorized
Cache-Control: no-cache
Cseq: 0
Session-Id: 64201536
User-Agent: Http Stream Server/1.0
WWW-Authenticate:  Digest realm="Login to dd1176c88fde59a5d01a6be0c45b9630", nonce="e6eb5b84b4d365a45f8022f5a40d2459"


Request:
GET /live/realmonitor.xav?channel=1&subtype=0&method=0 HTTP/1.1
Accept-Sdp: Private
Authorization: Digest username="admin", realm="Login to dd1176c88fde59a5d01a6be0c45b9630", nonce="e6eb5b84b4d365a45f8022f5a40d2459", uri="/live/realmonitor.xav?channel=1&subtype=0", response="df761088b0aceb3a24317badac34d248"
Connection: keep-alive
Cseq: 1
Host: 192.168.1.108:8086
Range: npt=0.000000-


Response:
HTTP/1.1 200 OK
Cache-Control: no-cache
Connection: close
Content-Type: video/x-xav
Cseq: 1
KeepLive-Time: 60
Private-Length: 640
Private-Type: application/sdp
Range: npt=0.000000-
Session-Id: 64201536
User-Agent: Http Stream Server/1.0

v=0
o=- 2273699856 2273699856 IN IP4 0.0.0.0
s=Media Server
c=IN IP4 0.0.0.0
t=0 0
a=control:*
a=packetization-supported:DH
a=rtppayload-supported:DH
a=range:npt=now-
a=x-summary:Mainland
a=x-packetization-supported:IV
a=x-rtppayload-supported:IV
m=video 0 RTP/AVP 96
a=control:trackID=0
a=framerate:30.000000
a=rtpmap:96 H264/90000
a=recvonly
m=application 0 RTP/AVP 100
a=control:trackID=3
a=rtpmap:100 stream-assist-frame/90000
a=recvonly
m=application 0 RTP/AVP 107
a=control:trackID=4
a=rtpmap:107 vnd.onvif.metadata/90000
a=recvonly
m=audio 0 RTP/AVP 8
a=control:trackID=6
a=rtpmap:8 PCMA/8000
a=recvonly
```

---


