# 第五章 安全（非规范）

## 目录

- [第一章 - 介绍](01-Introduction.md)
- [第二章 – MQTT控制报文格式](02-ControlPacketFormat.md)
- [第三章 – MQTT控制报文](03-ControlPackets.md)
- [第四章 – 操作行为](04-OperationalBehavior.md)
- [第五章 – 安全（非规范）](05-Security.md)
- [第六章 – 使用WebSocket作为网络层](06-WebSocket.md)
- [第七章 – 一致性](07-Conformance.md)
- [附录B - 强制性规范声明（非规范）](08-AppendixB.md)
- [附录C - MQTT v5.0新特性总结（非规范）](09-AppendixC.md)

## 5.1 概述 Introduction

强烈建议提供TLS [\[RFC5246\]](http://www.rfc-editor.org/info/rfc5246)的服务端实现使用TCP端口8883（IANA服务名：secure-mqtt）。

安全是一个快速变化的领域，所以在设计安全解决方案时总是使用最新的建议。

解决方案需要考虑的风险包括：

-   设备可能会被盗用
-   客户端和服务端的静态数据可能是可访问的（可能会被修改）
-   协议行为可能有副作用（如计时器攻击）
-   拒绝服务攻击
-   通信可能会被拦截、修改、重定向或者泄露
-   虚假MQTT控制报文注入

MQTT方案通常部署在不安全的通信环境中。在这种情况下，协议实现通常需要提供这些机制：

-   用户和设备身份认证
-   服务端资源访问授权
-   MQTT控制报文和内嵌应用数据的完整性校验
-   MQTT控制报文和内嵌应用数据的隐私控制

作为传输层协议，MQTT仅关注消息传输，提供合适的安全功能是实现者的责任。使用TLS [\[RFC5246\]](http://www.rfc-editor.org/info/rfc5246)是比较普遍的选择。

除了技术上的安全问题外，还有地区因素（例如美国欧盟隐私盾框架[\[USEUPRIVSH\]](https://www.privacyshield.gov)），行业标准（例如第三方支付行业数据安全标准 [\[PCIDSS\]](https://www.pcisecuritystandards.org/pci_security/)），监管方面的考虑（例如萨斯班-奥克斯利法案[\[SARBANES\]](http://www.gpo.gov/fdsys/pkg/PLAW-107publ204/html/PLAW-107publ204.htm)）。

## 5.2 MQTT解决方案：安全和认证 MQTT solutions: security and certification

协议实现可能需要提供符合特定行业安全标准，如NIST网络安全框架 [\[NISTCSF\]](https://www.nist.gov/sites/default/files/documents/itl/preliminary-cybersecurity-framework.pdf)，第三方支付行业数据安全标准 [\[PCIDSS\]](https://www.pcisecuritystandards.org/pci_security/)，美国联邦信息处理标准 [\[FIPS1402\]](https://csrc.nist.gov/csrc/media/publications/fips/140/2/final/documents/fips1402.pdf) 和NSA加密组合B [\[NSAB\]](http://www.nsa.gov/ia/programs/suiteb_cryptography/)。

在MQTT的补充出版物（MQTT and the NIST Framework for Improving Critical Infrastructure Cybersecurity [\[MQTTNIST\]](http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html)）中可以找到在NIST网络安全框架 [\[NISTCSF\]](https://www.nist.gov/sites/default/files/documents/itl/preliminary-cybersecurity-framework.pdf) 中使用MQTT的指导。使用行业证明、独立审计和认证技术有助于满足合规要求。

## 5.3 轻量级的加密与受限设备 Lightweight crytography and constrained devices

广泛采用的加密算法是高级加密标准 [\[AES\]](https://csrc.nist.gov/csrc/media/publications/fips/197/final/documents/fips-197.pdf)。对AES提供了硬件支持的处理器有很多，但通常不包含嵌入式处理器。加密算法ChaCha20 [\[CHACHA20\]](https://tools.ietf.org/html/rfc7539) 软件加解密速度快很多，但不像AES那样广泛可用。

推荐使用为资源受限的低端设备特别优化过的轻量级加密国际标准ISO 29192 [\[ISO29192\]](https://www.iso.org/standard/56425.html)。

## 5.4 实现注意事项 Implementation notes

实现或使用MQTT时需要考虑许多安全问题。以下章节不应被视为*核对清单*。

协议实现时可以实现下面的一部分或全部：

### 5.4.1 客户端身份验证 Authentication of Clients by the Server

CONNECT报文包含用户名和密码字段。实现可以决定如何使用这些字段的内容。实现者可以提供自己的身份验证机制，或者使用外部的认证系统如LDAP [\[RFC4511\]](http://www.rfc-editor.org/info/rfc4511) 或Auth [\[RFC6749\]](http://www.rfc-editor.org/info/rfc6749)，还可以利用操作系统的认证机制。

MQTT v5.0提供了一种增强认证机制，如4.12节所述。使用此机制需要客户端和服务端双方的支持。

实现可以明文传递认证数据，混淆数据元素，或者不要求任何认证数据，但应该意识到这会增加中间人攻击和重放攻击的风险。5.4.5节介绍了确保数据私密的方法。

在客户端和服务端之间使用虚拟专用网（VPN）可以确保数据只被授权的客户端收到。

使用TLS [\[RFC5246\]](http://www.rfc-editor.org/info/rfc5246)时，服务端可以使用客户端发送的TLS证书验证客户端的身份。

实现可以允许客户端通过应用消息给服务端发送用于身份验证的凭证。

### 5.4.2 客户端授权 Authorization of Clients by the Server

如果客户端已经成功通过身份认证，服务端实现需要在接受连接之前执行授权检查。 

授权可以基于客户端提供的信息如用户名，客户端主机名/IP地址，或认证机制的结果。

具体来说，实现应该检查客户端是否被授权使用此客户标识符，因为客户标识符提供了对MQTT会话状态的访问（如4.1节所述）。此授权检查是为了防止某个客户端偶然或恶意的使用了已被其他客户端所使用的客户标识符。

实现应该提供发生在CONNECT之后的访问控制以限制客户端发布消息到特定主体或使用特定主体过滤器进行订阅的能力。实现需要考虑对具有广泛作用域的主题过滤器的访问限制，如"#"主题过滤器。

### 5.4.3 服务端身份验证 Authentication of the Server by the Client

MQTT协议不是双向信任的。基本认证没有提供客户端验证服务端身份的机制。某些形式的扩展认证允许双向认证。

但是使用TLS [\[RFC5246\]](http://www.rfc-editor.org/info/rfc5246)时，客户端可以使用服务端发送的TLS证书验证服务端的身份。从单IP多域名提供MQTT服务的实现应该考虑 [\[RFC6066\]](http://www.rfc-editor.org/info/rfc6066)第3节定义的TLS的SNI扩展。SNI允许客户端告诉服务端它要连接的服务端主机名。

实现可以允许服务端通过应用消息给客户端发送凭证用于身份验证。MQTT v5.0提供了一种增强的认证机制，如4.12节所述，它可以被客户端用于验证服务端。使用此机制需要客户端和服务端双方的支持。

在客户端和服务端之间使用虚拟专用网（VPN）可以确保客户端正连接的是预期的服务端。

### 5.4.4 应用消息和MQTT控制报文的完整性 Integrity of Application Messages and MQTT Control Packets

应用可以在应用消息中单独包含哈希值。这样做可以为PUBLISH报文的网络传输和静态数据提供内容的完整性检查。 

TLS [\[RFC5246\]](http://www.rfc-editor.org/info/rfc5246)提供了对网络传输的数据做完整性校验的哈希算法。

在客户端和服务端之间使用虚拟专用网（VPN）连接可以在VPN覆盖的网络段提供数据完整性检查。

### 5.4.5 应用消息和MQTT控制报文的保密性 Privacy of Application Messages and Control Packets

TLS [\[RFC5246\]](http://www.rfc-editor.org/info/rfc5246)可以对网络传输的数据加密。如果有效的 TLS 密码组合包含的加密算法为 NULL，那么它不会加密数据。要确保客户端和服务端的保密，应避免使用这些密码组合。

应用可以单独加密应用消息的内容。这可以提供应用消息传输途中和静态数据的私密性。但不能给应用消息的其它属性如主题名加密。

客户端和服务端实现可以加密存储静态数据，例如可以将应用消息作为会话的一部分存储。

在客户端和服务端之间使用虚拟专用网（VPN）连接可以在VPN覆盖的网络段保证数据的私密性。

### .5.4.6 消息传输的不可否认性 Non-repudiation of message transmission

应用设计者可能需要考虑适当的策略，以实现端到端的不可否认性（non-repudiation）。

### 5.4.7 检测客户端和服务端的盗用 Detecting compromise of Clients and Servers

使用TLS [\[RFC5246\]](http://www.rfc-editor.org/info/rfc5246)的客户端和服务端实现应该能够确保，初始化TLS连接时提供的 SSL 证书是与主机名（客户端要连接的或服务端将被连接的）关联的。

使用TLS [\[RFC5246\]](http://www.rfc-editor.org/info/rfc5246)的客户端和服务端实现，可以选择提供检查证书吊销列表（CRLs [\[RFC5280\]](http://www.rfc-editor.org/info/rfc5280)）和在线整数状态协议（OSCP）[\[RFC6960\]](http://www.rfc-editor.org/info/rfc6960)的功能，拒绝使用被吊销的整数。

物理部署可以将防篡改硬件与应用消息的特殊数据传输结合。例如，一个仪表可能会内置一个GPS以确保没有在未授权的地区使用。IEEE安全设备认证[\[IEEE8021AR\]](http://standards.ieee.org/findstds/standard/802.1AR-2009.html)就是用于实现这个机制的一个标准，它使用加密绑定标识符验证设备身份。

### 5.4.8 检测异常行为 Detecting abnormal behaviors

服务端实现可以监视客户端的行为，检测潜在的安全风险。例如：

-   重复的连接请求
-   重复的身份验证请求
-   连接的异常终止
-   主题扫描（请求发送或订阅大量主题）
-   发送无法送达的消息（没有订阅者的主题）
-   客户端连接但是不发送数据

发现违反安全规则的行为，服务端实现可以断开客户端连接。

服务端实现检测不受欢迎的行为，可以基于IP地址或客户端标识符实现一个动态黑名单列表。

服务部署可以使用网络层次控制（如果可用）实现基于IP地址或其它信息的速率限制或黑名单。

### 5.4.9 其它的安全注意事项 Other security considerations

如果客户端或服务端的SSL证书丢失，或者我们考虑证书被盗用或者被吊销(利用 CRLs [\[RFC5280\]](http://www.rfc-editor.org/info/rfc5280)和OSCP [\[RFC6960\]](http://www.rfc-editor.org/info/rfc6960)的情况。

客户端或服务端验证凭证时，如果发现用户名和密码丢失或被盗用，应该吊销或者重新发放。

在使用长连接时：

-   客户端和服务端使用TLS [\[RFC5246\]](http://www.rfc-editor.org/info/rfc5246)时应该允许重新协商会话以确认新的加密参数（替换会话密钥，更换密码组合，更换认证凭证）。
-   服务端可以关闭客户端的网络连接，并要求他们使用新的凭证重新验证身份。
-   服务端可以要求客户端使用4.12.1节 中描述的机制周期性的进行重新认证。

资源受限设备或使用受限网络的客户端可以使用TLS [RFC5246](http://www.rfc-editor.org/info/rfc5246)会话恢复，以降低TLS [RFC5246](http://www.rfc-editor.org/info/rfc5246)会话重连的成本。

连接到服务端的客户端与其它连接到服务端的客户端之间有一个信任传递关系，它们都有权在同一个主题上发布消息。

### 5.4.10 使用SOCKS代理 Use of SOCKS

客户端实现应该意识到某些环境要求使用SOCKSv5 [\[RFC1928\]](http://www.rfc-editor.org/info/rfc1928)代理创建出站的网络连接。某些MQTT实现可以利用安全隧道（如SSH）通过SOCKS代理。一个实现决定支持SOCKS时，它们应该同时支持匿名的和用户名密码验证的SOCKS代理。对于后一种情况，实现应该意识到SOCKS可能使用明文认证，因此应该避免使用相同的凭证连接 MQTT 服务器。

### 5.4.11 安全配置文件 Security profiles

实现者和方案设计者可能希望将安全当作配置文件集合应用到MQTT协议中。下面描述的是一个分层的安全等级结构。

#### 5.4.11.1 开放通信配置 Clear communication profile

使用开放通信配置时，MQTT协议运行在一个没有内置额外安全通信机制的开放网络上。

#### 5.4.11.2 安全网络通信配置 Secured network communication profile

使用安全网络通信配置时，MQTT协议运行在有安全控制的物理或虚拟网络上，如VPN或物理安全网络。

#### 5.4.11.3 安全传输配置 Secured transport profile

使用安全传输配置时，MQTT协议运行在使用TLS [\[RFC5246\]](http://www.rfc-editor.org/info/rfc5246) 的物理或虚拟网络上，它提供了身份认证，完整性和保密性。

使用内置的用户名和密码字段，TLS [\[RFC5246\]](http://www.rfc-editor.org/info/rfc5246) 客户端身份认证可被用于（或者代替）MQTT客户端认证。

#### 5.4.11.4 工业标准的安全配置 Industry specific security profiles

可以预料的是，MQTT协议被设计为支持很多工业标准的应用配置，每一种定义一个威胁模型和用于定位威胁的特殊安全机制。特殊的安全机制推荐从下面的方案中选择：

[\[NISTCSF\]](http://www.nsa.gov/ia/programs/suiteb_cryptography/) NIST网络安全框架  
[\[NIST7628\]](http://www.nsa.gov/ia/programs/suiteb_cryptography/) NISTIR 7628智能电网网络安全指南  
[\[FIPS1402\]](https://csrc.nist.gov/csrc/media/publications/fips/140/2/final/documents/fips1402.pdf) (FIPS PUB 140-2) 加密模块的安全要求  
[\[PCIDSS\]](https://www.pcisecuritystandards.org/pci_security/) PCI-DSS第三方支付行业数据安全标准  
[\[NSAB\]](http://www.nsa.gov/ia/programs/suiteb_cryptography/) NSA加密组合B

### 项目主页

- [MQTT v5.0协议草案中文版](https://github.com/hui6075/mqtt_v5)


