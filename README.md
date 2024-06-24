# InstallCert
java生成证书文件
这段代码是一个用于安装SSL证书的Java程序。它允许用户连接到指定的主机和端口，获取服务器的SSL证书链，并将所选择的服务器证书添加到本地信任库中。

具体来说，这个程序的主要功能包括：

1. 解析命令行参数：
从命令行参数中获取要连接的主机名和端口号。
可以指定可选的密码用于访问信任库。
2. 加载信任库：
程序首先尝试加载名为 jssecacerts 的信任库文件，若不存在则在默认Java安装路径下查找 lib/security 目录下的 jssecacerts 或 cacerts 文件。
3. 建立SSL连接：
使用加载的信任库初始化 SSL 上下文。
创建 SSL 套接字并尝试与指定的主机和端口建立连接，执行 SSL 握手过程。
4. 检查服务器证书：
如果握手过程中出现 SSLException，表示可能存在证书问题，程序将打印异常信息。
获取服务器发送的证书链，并计算证书的 SHA1 和 MD5 散列值。
5. 选择并添加证书：
用户可以选择要添加到信任库的证书。
程序将选定的证书以指定的别名添加到信任库文件中，并将修改后的信任库保存到 jssecacerts 文件中。

总体而言，这个程序的作用是允许用户检查和管理与特定主机建立的 SSL 连接的信任关系，可以用于处理需要定制 SSL 信任证书的特定情况，例如连接到私有服务器或处理特定的证书链配置。
 
尝试1
1、javac InstallCert.java将其进行编译
2、然后再执行java InstallCert www.baidu.com （这里我们用百度举例子，实际填写的就是你想要获取证书的目标网站）
3、在生成的时候需要输入一个1
Enter certificate to add to trusted keystore or 'g to quit.【1】
   jssecacerts，将它放入我们本地的 jdk的lib\security文件夹内就行了

   
尝试2
1.通过System.setProperty("javax.net.ssl.trustStore", "你的jssecacerts证书路径");
2.程序启动命令-Djavax.net.ssl.trustStore=你的jssecacerts证书路径 -Djavax.net.ssl.trustStorePassword=changeit
 
尝试3--不校验了，直接全部信任
原代码
 
URL console = new URL(url);
HttpURLConnection conn = (HttpURLConnection) console.openConnection();
if (conn instanceof HttpsURLConnection)  {
    SSLContext sc = SSLContext.getInstance("SSL");
    sc.init(null, new TrustManager[]{new TrustAnyTrustManager()}, new java.security.SecureRandom());
    ((HttpsURLConnection) conn).setSSLSocketFactory(sc.getSocketFactory());
    ((HttpsURLConnection) conn).setHostnameVerifier(new TrustAnyHostnameVerifier());
}
conn.connect();
InputStream inputStream = conn.getInputStream();
PdfReader pdfReader = new PdfReader(inputStream);
inputStream.close();
conn.disconnect();
 
现代码
 
private static class TrustAnyTrustManager implements X509TrustManager {
    //这个方法用于验证客户端的证书。在这里，方法体为空，表示不对客户端提供的证书进行任何验证。
    public void checkClientTrusted(X509Certificate[] chain, String authType) throws CertificateException {
    }
    //这个方法用于验证服务器的证书。同样，方法体为空，表示不对服务器提供的证书进行任何验证。
    public void checkServerTrusted(X509Certificate[] chain, String authType) throws CertificateException {
    }
    //这个方法返回一个信任的证书数组。在这里，返回空数组，表示不信任任何证书，也就是对所有证书都不做任何信任验证。
    public X509Certificate[] getAcceptedIssuers() {
        return new X509Certificate[]{};
    }
}
//这个方法用于验证主机名是否可信。在这里，无论传入的主机名是什么，方法始终返回 true，表示信任任何主机名。这就意味着对于 SSL 连接，不会对主机名进行真实的验证，而是始终接受所有主机名。
private static class TrustAnyHostnameVerifier implements HostnameVerifier {
    public boolean verify(String hostname, SSLSession session) {
        return true;
    }
}
 
