### UPYUN核心架构 | OCSP Stapling 性能测试报告

        作者： 朱鸣啸
        
        时间： 2016.02.15

### OCSP Stapling 是如何让 UPYUN 更高效的

如果不使用 OCSP Stapling，那么当客户端访问 HTTPS 页面时，只能通过下载证书撤销列表(使用 CRLs 时)或者通过 OCSP 服务器(使用 OCSP 协议时)来验证证书的合法性。无论是何种方式，浏览器都需要发起两次无法并发的请求(第一次向 web server 发起，第二次会根据证书中的 URI 下载 CRLs 或者向 OCSP server 发起请求)。

OCSP Stapling 就是让客户端省掉了第二次请求。当使用 OCSP Stapling 时，客户端可以直接通过 web server 来验证证书是否被撤销。OCSP Stapling 在 UPYUN 上的具体工作流程可以分成以下几步：

1. UPYUN 会向 OCSP server 周期性的查询证书状态，然后会得到并缓存带有时间戳和数字签证的 OCSP response 。
2. 当客户端向 UPYUN 发出请求时，UPYUN 会将缓存的 OCSP response 传给客户端。
3. 客户端通过 OCSP response 获得证书的状态，然后根据证书状态进行接下来的工作。

### 具体的性能提升表现

为了测试 OCSP Stapling 的具体表现，我们对若干测试点进行了长达十二天的观察。

观察结果如下：


| Enable OCSP Stapling | Average SSL Connection Time |
| -------------------- | --------------------------- |
|         YES          |             0.179 秒        |
|         NO           |             0.230 秒        |

可以看到，使用 OCSP Stapling 可以在 SSL 握手阶段节约 50+ 毫秒， 性能提升了 22% 左右。

测试的具体结果可以参考 [OCSP Stapling 性能测试历史曲线图.pdf](https://attachments.tower.im/tower/1b57750d917b4a6d868b2e3d07d52f6a?filename=OCSP+%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95%E5%8E%86%E5%8F%B2%E6%9B%B2%E7%BA%BF%E5%9B%BE.pdf)


### 如何检查 OCSP Stapling 是否开启

在 Mac OS 和 Linux 终端，你可以通过以下方式查询：

    echo QUIT | openssl s_client -connect docs.upyun.com:443 -servername docs.upyun.com -tls1_2 -status 2> /dev/null | grep -A 17 'OCSP response:' | grep -B 17 'Next Update'

如果你已经开启 OCSP Stapling ， 你会得到以下内容：

    OCSP response: 
    ======================================
    OCSP Response Data:
    OCSP Response Status: successful (0x0)
    Response Type: Basic OCSP Response
    Version: 1 (0x0)
    Responder Id: 67980BC128748D9C635A14D902AD485AC445B9A8
    Produced At: Feb 11 01:54:09 2016 GMT
    Responses:
    Certificate ID:
      Hash Algorithm: sha1
      Issuer Name Hash: B18B0B019753072C7437D29DB3E18DA36CCE57E0
      Issuer Key Hash: D26FF796F4853F723C307D23DA85789BA37C5A7C
      Serial Number: 634E790AEE64DF3015DCC38C355C002B
    Cert Status: good
    This Update: Feb 11 01:54:09 2016 GMT
    Next Update: Feb 18 01:54:09 2016 GMT


当然你也可以通过 [Qualys SSL Labs 的 SSL Server Test](https://www.ssllabs.com/ssltest/) 进行更全面的检测。

### 参考文档
[MaxCDN Blog](https://www.maxcdn.com/blog/now-shipping-ocsp-stapling/)

[WildDog Blog](https://blog.wilddog.com/?p=615)
