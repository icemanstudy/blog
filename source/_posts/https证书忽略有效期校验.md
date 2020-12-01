---
title: https证书忽略有效期校验
date: 2018-02-08 18:28:12
author: iceman
categories:
- android
tags:
- https
index_img:
- /img/post/18.jpg
---

最近公司的数据统计平台报了大量ssl证书错误.类似于这种
```
javax.net.ssl.sslhandshakeexception: 
	com.android.org.bouncycastle.jce.exception.extcertpathvalidatorexception: 
		could not validate certificate: 
			current time: mon jan 04 13:33:18 格林尼治标准时间+0800 2010, 
				validation time: fri mar 08 20:00:00 格林尼治标准时间+0800 2013
```

拿着上报信息中的链接在浏览器打开,并没有证书错误,我们的后台也并没有做什么自定义证书,完全是系统默认的SSL Context.
结合异常信息,并检查对应域名的证书信息,可以断定是**手机系统时间修改后,超出证书有效期**,导致证书校验错误.
虽说这本质上是一个安全性措施,但是修改系统时间的场景也无法完全避免.初步推测是某些云测平台的系统时间有误导致.

**信任所有证书肯定是不行的.不仅有安全问题,同时各种app安全扫描平台基本上都把这个当高危项目.**

那么问题来了,如何单单将证书时间错误的问题忽略掉呢?

一开始我是这么干的,检测到指定Exception时,忽略掉.
关键代码都在这个自定义的TrustManager里面.在checkServerTrusted中对异常类型进行检查.
```java
final TrustManager[] trustAllCerts = new TrustManager[]{new X509TrustManager() {
            @Override
            public void checkClientTrusted(
                    java.security.cert.X509Certificate[] chain,
                    String authType) throws CertificateException {
            }

            @Override
            public void checkServerTrusted(
                    java.security.cert.X509Certificate[] chain,
                    String authType) throws CertificateException {

                try {
                    defaultTrustManager.checkServerTrusted(chain, authType);
                } catch (CertificateException e) {
                    e.printStackTrace();
                    Throwable t = e;
                    while (t != null) {
                        if (t instanceof CertificateExpiredException
                                || t instanceof CertificateNotYetValidException)
                            return;
                        t = t.getCause();
                    }
                    throw e;
                }

            }
```
这里面有个问题:
通常这个证书链是包含多个证书的,如果一检测到有效期相关的异常就return,会导致后面的所有证书都被信任了.不太安全.

更坑爹的是,实际使用中抛出来的是**CertPathValidatorException**!!!看这个异常的描述,不能保证触发它的一定是有效期问题.

既然TrustManager不能做文章了.那就再往下一层.在X509Certificate上看看吧.

X509Certificate有很多方法,其中和本文相关的**checkValidity()**和**checkValidity(Date date)**就是证书有效期检查相关的方法.

于是用自定义的X509Certificate封装一层,把这两个方法给屏蔽掉.就可以既不影响其他的证书校验,就可以忽略日期异常了.
```java
private class EternalCertificate extends X509Certificate {
	//原始X509Certificate,其他校验流程还是要走的
        private final X509Certificate originalCertificate;

        public EternalCertificate(X509Certificate originalCertificate) {
            this.originalCertificate = originalCertificate;
        }

        @Override
        public void checkValidity() throws CertificateExpiredException, CertificateNotYetValidException {
            // 这里什么都不做,忽略掉有效期检查
        }

        @Override
        public void checkValidity(Date date) throws CertificateExpiredException, CertificateNotYetValidException {
            // 这里什么都不做,忽略掉有效期检查
        }

        @Override
        public int getVersion() {
            return originalCertificate.getVersion();
        }

        @Override
        public BigInteger getSerialNumber() {
            return originalCertificate.getSerialNumber();
        }

        @Override
        public Principal getIssuerDN() {
            return originalCertificate.getIssuerDN();
        }

        @Override
        public Principal getSubjectDN() {
            return originalCertificate.getSubjectDN();
        }

        @Override
        public Date getNotBefore() {
            return originalCertificate.getNotBefore();
        }

        @Override
        public Date getNotAfter() {
            return originalCertificate.getNotAfter();
        }

        @Override
        public byte[] getTBSCertificate() throws CertificateEncodingException {
            return originalCertificate.getTBSCertificate();
        }

        @Override
        public byte[] getSignature() {
            return originalCertificate.getSignature();
        }

        @Override
        public String getSigAlgName() {
            return originalCertificate.getSigAlgName();
        }

        @Override
        public String getSigAlgOID() {
            return originalCertificate.getSigAlgOID();
        }

        @Override
        public byte[] getSigAlgParams() {
            return originalCertificate.getSigAlgParams();
        }

        @Override
        public boolean[] getIssuerUniqueID() {
            return originalCertificate.getIssuerUniqueID();
        }

        @Override
        public boolean[] getSubjectUniqueID() {
            return originalCertificate.getSubjectUniqueID();
        }

        @Override
        public boolean[] getKeyUsage() {
            return originalCertificate.getKeyUsage();
        }

        @Override
        public int getBasicConstraints() {
            return originalCertificate.getBasicConstraints();
        }

        @Override
        public byte[] getEncoded() throws CertificateEncodingException {
            return originalCertificate.getEncoded();
        }

        @Override
        public void verify(PublicKey key) throws CertificateException, NoSuchAlgorithmException, InvalidKeyException, NoSuchProviderException,
                SignatureException {
            originalCertificate.verify(key);
        }

        @Override
        public void verify(PublicKey key, String sigProvider) throws CertificateException, NoSuchAlgorithmException, InvalidKeyException,
                NoSuchProviderException, SignatureException {
            originalCertificate.verify(key, sigProvider);
        }

        @Override
        public String toString() {
            return originalCertificate.toString();
        }

        @Override
        public PublicKey getPublicKey() {
            return originalCertificate.getPublicKey();
        }

        @Override
        public Set<String> getCriticalExtensionOIDs() {
            return originalCertificate.getCriticalExtensionOIDs();
        }

        @Override
        public byte[] getExtensionValue(String oid) {
            return originalCertificate.getExtensionValue(oid);
        }

        @Override
        public Set<String> getNonCriticalExtensionOIDs() {
            return originalCertificate.getNonCriticalExtensionOIDs();
        }

        @Override
        public boolean hasUnsupportedCriticalExtension() {
            return originalCertificate.hasUnsupportedCriticalExtension();
        }
    }
```

这里是checkServerTrusted中对证书链的处理
```java
@Override
    public void checkServerTrusted(X509Certificate[] chain, String authType) throws CertificateException {
        chain = Arrays.copyOf(chain, chain.length);
        X509Certificate[] newChain = new X509Certificate[chain.length];
        for (int i = 0; i < chain.length; i++) {
            newChain[i] = new EternalCertificate(chain[i]);
        }
        chain = newChain;
        this.innerTrustManager.checkServerTrusted(chain, authType);
    }
```
这里innerTrustManager是默认的trustManager.可以这样拿到,传给自定义的TrustManager,以便调用原本的相关方法.
```java
TrustManagerFactory trustManagerFactory = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
trustManagerFactory.init((KeyStore) null);
final X509TrustManager defaultTrustManager = (X509TrustManager) trustManagerFactory.getTrustManagers()[0];
```

到这里问题差不多解决了.但是为安全着想,以及影响范围最小化的原则,我们最好是针对本公司的证书做这样的忽略.这里涉及到证书结构.顺便学习了一波.
Android用的是X509证书格式.这里是一个证书示例.
```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            05:6f:a7:c7:98:b4:9a:cd:68:a9:39:37:7b:bd:66:73
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=US, O=DigiCert Inc, CN=DigiCert SHA2 Secure Server CA
        Validity
            Not Before: Dec  5 00:00:00 2017 GMT
            Not After : Feb  5 12:00:00 2019 GMT
        Subject: C=CN, L=Changsha, O=xxxxxx xxxx xxxxx xxxxx Co., Ltd, OU=IT, CN=*.xxx.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:c0:31:45:83:9f:01:81:e8:b1:9f:75:ed:9b:45:
                    04:05:3b:6e:14:f1:68:35:b2:80:d1:bc:d3:53:40:
                    ac:b4:b6:fe:8d:f2:bf:c1:58:f8:b6:a8:54:b8:ae:
                    bf:01:b8:77:10:3a:7b:f8:33:90:67:01:70:d2:cc:
                    80:91:c0:02:7c:b0:87:8d:8d:c0:30:3c:e2:44:69:
                    ef:8a:07:80:b0:b9:8a:8e:e9:fd:28:08:9b:b5:7f:
                    ac:60:5d:6e:b2:c9:c9:b4:fc:12:0a:df:1a:5a:0b:
                    39:8d:5c:7f:f7:7d:16:1a:6a:f3:a7:dd:2c:81:aa:
                    b8:83:c2:60:11:f6:05:0d:6e:8b:75:36:44:74:a0:
                    96:fd:86:36:30:58:90:6a:13:48:a7:c9:16:af:72:
                    58:a5:f1:3c:39:95:93:e4:e4:78:78:f4:68:4a:59:
                    0c:e4:e2:7d:14:21:8a:91:ea:5d:5b:95:d0:dc:a4:
                    cb:64:9c:33:98:db:4b:52:5e:fa:0c:3a:f4:45:f6:
                    98:04:81:ec:45:1e:60:f9:a2:23:6c:84:f7:e2:d0:
                    bb:12:70:7f:e1:83:f6:9b:f2:6f:9d:53:ce:57:3d:
                    60:90:db:a6:b7:e0:15:bb:7a:7e:b0:53:ad:0a:97:
                    c2:91:4a:5c:a9:81:9a:ee:ad:17:80:68:a4:50:74:
                    19:c1
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Authority Key Identifier: 
                keyid:0F:80:61:1C:82:31:61:D5:2F:28:E7:8D:46:38:B4:2C:E1:C6:D9:E2

            X509v3 Subject Key Identifier: 
                EF:85:1D:4E:E3:57:C2:03:63:BD:70:C0:0F:4D:E5:58:42:60:E8:72
            X509v3 Subject Alternative Name: 
                DNS:*.xxx.com, DNS:*.api.xxx.com, DNS:*.bz.xxx.com, DNS:*.log.xxx.com
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage: 
                TLS Web Server Authentication, TLS Web Client Authentication
            X509v3 CRL Distribution Points: 

                Full Name:
                  URI:http://crl3.digicert.com/ssca-sha2-g6.crl

                Full Name:
                  URI:http://crl4.digicert.com/ssca-sha2-g6.crl

            X509v3 Certificate Policies: 
                Policy: 2.16.840.1.114412.1.1
                  CPS: https://www.digicert.com/CPS
                Policy: 2.23.140.1.2.2

            Authority Information Access: 
                OCSP - URI:http://ocsp.digicert.com
                CA Issuers - URI:http://cacerts.digicert.com/DigiCertSHA2SecureServerCA.crt

            X509v3 Basic Constraints: critical
                CA:FALSE
    Signature Algorithm: sha256WithRSAEncryption
         81:f9:b8:e6:ae:ee:e2:89:12:8d:d4:d0:6c:c0:04:45:f2:9e:
         5b:38:67:ea:12:f8:0f:50:2f:27:77:ab:45:79:31:4c:16:cc:
         2f:58:9b:7c:11:43:a7:88:c3:6e:ce:b0:89:fb:e4:c1:ac:8b:
         d3:ae:d0:ee:3c:cb:33:54:03:0c:aa:4f:6a:23:02:58:e5:ed:
         55:16:1a:d3:a8:95:1e:0a:6d:f9:49:25:53:02:9b:19:fb:22:
         e2:b1:8f:1c:22:ac:08:76:3f:fd:4f:d4:7e:57:17:93:2c:80:
         b0:0c:ff:d5:c9:bb:b2:bb:fc:95:61:2c:e9:94:f9:e2:e9:45:
         7c:02:64:e1:52:a3:8b:fa:48:8c:9b:5a:bd:76:f3:91:b0:3a:
         d9:27:6c:b6:35:38:ac:88:bf:48:9e:19:e3:17:59:7e:00:d9:
         e6:2f:bc:08:0a:c5:37:4d:ed:4d:14:78:d1:94:c3:40:6a:95:
         96:91:f6:38:2e:e4:63:f7:fd:f6:fb:25:a5:1c:b0:5a:29:e3:
         dd:d7:68:97:2a:58:26:fb:a1:18:e7:e3:80:94:6b:1f:b0:65:
         a6:65:1d:79:8f:d7:1a:3a:7b:7a:3c:db:a6:60:a9:99:de:57:
         f5:7a:bc:4d:05:e3:64:e2:a9:6f:a1:64:09:6f:a5:51:62:a2:
         da:b3:6d:f3


```



关于里面的字段解释,可以对照这个来看
```
1. 证书版本号(Version)
版本号指明X.509证书的格式版本，现在的值可以为:
    1) 0: v1
    2) 1: v2
    3) 2: v3
也为将来的版本进行了预定义

2. 证书序列号(Serial Number)
序列号指定由CA分配给证书的唯一的"数字型标识符"。当证书被取消时，实际上是将此证书的序列号放入由CA签发的CRL中，
这也是序列号唯一的原因。

3. 签名算法标识符(Signature Algorithm)
签名算法标识用来指定由CA签发证书时所使用的"签名算法"。算法标识符用来指定CA签发证书时所使用的:
    1) 公开密钥算法
    2) hash算法
example: sha256WithRSAEncryption
须向国际知名标准组织(如ISO)注册

4. 签发机构名(Issuer)
此域用来标识签发证书的CA的X.500 DN(DN-Distinguished Name)名字。包括:
    1) 国家(C)
    2) 省市(ST)
    3) 地区(L)
    4) 组织机构(O)
    5) 单位部门(OU)
    6) 通用名(CN)
    7) 邮箱地址

5. 有效期(Validity)
指定证书的有效期，包括:
    1) 证书开始生效的日期时间
    2) 证书失效的日期和时间
每次使用证书时，需要检查证书是否在有效期内。

6. 证书用户名(Subject)
指定证书持有者的X.500唯一名字。包括:
    1) 国家(C)
    2) 省市(ST)
    3) 地区(L)
    4) 组织机构(O)
    5) 单位部门(OU)
    6) 通用名(CN)
    7) 邮箱地址

7. 证书持有者公开密钥信息(Subject Public Key Info)
证书持有者公开密钥信息域包含两个重要信息:
    1) 证书持有者的公开密钥的值
    2) 公开密钥使用的算法标识符。此标识符包含公开密钥算法和hash算法。
8. 扩展项(extension)
X.509 V3证书是在v2的基础上一标准形式或普通形式增加了扩展项，以使证书能够附带额外信息。标准扩展是指
由X.509 V3版本定义的对V2版本增加的具有广泛应用前景的扩展项，任何人都可以向一些权威机构，如ISO，来
注册一些其他扩展，如果这些扩展项应用广泛，也许以后会成为标准扩展项。

9. 签发者唯一标识符(Issuer Unique Identifier)
签发者唯一标识符在第2版加入证书定义中。此域用在当同一个X.500名字用于多个认证机构时，用一比特字符串
来唯一标识签发者的X.500名字。可选。

10. 证书持有者唯一标识符(Subject Unique Identifier)
持有证书者唯一标识符在第2版的标准中加入X.509证书定义。此域用在当同一个X.500名字用于多个证书持有者时，
用一比特字符串来唯一标识证书持有者的X.500名字。可选。

11. 签名算法(Signature Algorithm)
证书签发机构对证书上述内容的签名算法
example: sha256WithRSAEncryption

12. 签名值(Issuer's Signature)
证书签发机构对证书上述内容的签名值
```

所以在X509Certificate封装过程中如何识别,就根据实际情况自行决定了.