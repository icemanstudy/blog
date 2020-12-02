---
title: android中的https使用自生成证书
date: 2018-02-03 18:15:32
categories:
- android
index_img:
- /img/post/6.jpg
excerpt:
- 不让你抓包,就是这么屌.
---
说起https就得从android app加密说起。

想hack一个android应用，无非就是反编译和抓包两种办法。

反编译一般通过加壳和混淆来防止。

抓包么，就只能通过请求参数加密和https了。

### 重点说说https。

简单来说，https是通过ssl来进行数据加密的http。大致流程如下：

>客户端和服务端使用非对称加密进行第一次握手，服务端保留私钥，把公钥发给客户端，客户端首先判断一下这个公钥是否有效，过期，颁发机构什么的。如果没问题，就先生成一个随机值，然后用这个公钥加密，发给服务端，此时只有持有私钥的服务端能解密，于是就得到了这个随机值。现在双方都有同样的随机值了，为后面进行的对称加密传输数据做准备，因为一直用公钥私钥这种非对称的加密太慢了。。。

具体到android 开发中如何实施呢：

#### 1.首先我们要有个证书：

ps：网上铺天盖地那种信任所有证书的方式就不谈了，代码量太少，不专业。<(▰˘◡˘▰)>

>keytool -genkey -alias icemantest -keystore fjajfd.keystore -validity 365

后面会让你输入名字，公司，地区，别名，密码，有效时间等等。

完成之后生成了名叫fjajfd.keystore的一个keystore。这个里面包含私钥。有的android签名用的证书是jks结尾的，也是使用jks证书库的，跟这个差不多。

#### 2.从keystore里面导出证书，一般.cer这样的。里面包含公钥，这个是可以公开的，谁都可以用来加密，但是只有私钥才能解密。

>keytool -export -alias icemantest -keystore fjajfd.keystore -file cer.cer

生成一个叫cer.cer的证书。

#### 3.android 里面支持的证书是bks证书库格式的，而我们生成的是jks格式的，于是需要转换一下。

这里需要用到工具bc库bcprov-jdk15on-146.jar。

>keytool -import -alias icemantest -file cer.cer -keystore bks.bks -storetype BKS -providerClass org.bouncycastle.jce.provider.BouncyCastleProvider -providerpath /Users/iceman/Downloads/bcprov-jdk15on-146.jar

现在bks.bks这个文件就是包含了公钥的android上可以使用的证书了。

### 现在以我们应用里面使用的证书举例如何在代码中使用这个证书。

文件xxx.bks放在raw目录里面，方便直接读取。

android中一般使用httpurlconnection和httpclient访问网络，这两种情况下使用证书的方式稍有不同，但核心都在一个叫SSLSocketFactory的类上面。

#### 先说httpurlconnection：

主机名验证:

这个可以使用默认的,看看这句明显是废话的代码就明白了:

```java
HttpsURLConnection.setDefaultHostnameVerifier(HttpsURLConnection.getDefaultHostnameVerifier());
```

证书验证:

##### 第一步:读取公钥信息到keystore中.

```java
private static KeyStore buildKeyStore(Context context, int certRawResId) throws KeyStoreException, CertificateException,NoSuchAlgorithmException, IOException {
	String keyStoreType = KeyStore.getDefaultType();
	KeyStore keyStore = KeyStore.getInstance(keyStoreType);
	InputStream inputStream = context.getResources().openRawResource(certRawResId);
	keyStore.load(inputStream, "xxxx".toCharArray());
	return keyStore;
}
```
 

##### 第二步:使用这个keystore来初始化一个trustmanager, trustmanager简单来说,就是管理是否信任服务端的.

```java
String tmfAlgorithm = TrustManagerFactory.getDefaultAlgorithm();
TrustManagerFactory tmf = null;
try { 
	tmf = TrustManagerFactory.getInstance(tmfAlgorithm);
	tmf.init(keyStore);
} catch (NoSuchAlgorithmException e){
	e.printStackTrace();
} catch (KeyStoreException e){
	e.printStackTrace();
}
```

##### 第三步,使用得到的trustmanager来初始化一个SslContext,进而得到SocketFactory:

```java
SSLContext sslContext = null;
try {
	sslContext = SSLContext.getInstance(“TLS”);
} catch (NoSuchAlgorithmException e){ 
	e.printStackTrace();
}
try {
	sslContext.init(null, tmf.getTrustManagers(), null);
} catch (KeyManagementException e) {
	e.printStackTrace(); } return sslContext.getSocketFactory();
```

得到SocketFactory之后就简单了,可以使用HttpsUrlconnetion的静态方法setDefaultSocketFactory来修改默认的,也可以在得到httpsurlconection以后调用setSSLSocketFactory方法来针对每个连接设置.

#### 然后说说apache的httpclient中使用方法

读取公钥至keystore跟前者完全一样.

不同的是apache中也有个SocketFactory类,可以直接用keystore参数来构造,其主机名验证方式也有预设的3个级别:

```java
SSLSocketFactory sf = new SSLSocketFactory(keyStore);
sf.setHostnameVerifier(SSLSocketFactory.STRICT_HOSTNAME_VERIFIER);
```

然后是分别绑定http和https至不同的端口并指定使用的socketfactory:

```java
SchemeRegistry registry = new SchemeRegistry();
registry.register(new Scheme(“http”, PlainSocketFactory.getSocketFactory(), 80));
registry.register(new Scheme(“https”, sf, 443));
return new SingleClientConnManager(getParams(), registry);
```

得到上面这个SingleClientConnManager之后,可以继承DefaultHttpClient,重写createClientConnectionManager来返回这个SingleClientConnManager,也可以直接使用这个SingleClientConnManager来构建一个defaultHttpClient.

