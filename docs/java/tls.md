# 自建证书以及实现TLS双向认证

## 双向认证流程
![](https://developers.cloudflare.com/access/static/mtls.png)

## 自建证书
1. 生成CA证书
   a. 生成CA私钥
   `openssl genrsa -out ca.key 2048`
   b. 生成CA证书
   `openssl req -new -x509 -days 3650 -key ca.key -out ca.crt`
2. 生成服务端证书
   a. 生成服务端私钥
   `openssl genrsa -out server.key 2048`
   b. 生成证书请求文件，服务端需要向CA机构申请签名证书，在申请签名证书之前依然是创建自己的CSR文件
   `openssl req -new -key server.key -out server.csr`
   c. 生成服务端证书，使用上面已经创建的CA证书，签发服务端证书
   `openssl x509 -req -CA ca.crt -CAkey ca.key -CAcreateserial -in server.csr -out server.crt -days 3650`
   d. 生成服务端证书链（如果需要）
   `cat server.crt >> server_chain.crt`
   `cat ca.crt >> server_chain.crt`
3. 生成客户端证书
   a. 生成服务端私钥
   `openssl genrsa -out client.key 2048`
   b. 生成证书请求文件，客户端需要向CA机构申请签名证书，在申请签名证书之前依然是创建自己的CSR文件
   `openssl req -new -key client.key -out client.csr`
   c. 生成服务端证书，使用上面已经创建的CA证书，签发客户端证书
   `openssl x509 -req -CA ca.crt -CAkey ca.key -CAcreateserial -in client.csr -out client.crt -days 3650`
   d. 生成客户端证书链（如果需要）
   `cat client.crt >> client_chain.crt`
   `cat ca.crt >> client_chain.crt`
4. Java端集成
   a. 生成p12格式证书
   `openssl pkcs12 -export -inkey client.key -in client.crt -out client.pfx`
   b. 创建本地信任证书库文件，并添加信任服务端证书
   `keytool -import -v -file server.cer -keystore client.truststore -storetype BKS -provider org.bouncycastle.jce.provider.BouncyCastleProvider -providerpath ./bcprov-jdk16-1.46.jar`
   [bcprov-jdk16-1.46.jar](https://repo1.maven.org/maven2/org/bouncycastle/bcprov-jdk16/1.46/bcprov-jdk16-1.46.jar)

注意： 
* Organization Name (eg, company): 表示组织名称，CA证书与签发的证书需要设置不一样！！
* Common Name (e.g. server FQDN or YOUR name): 表示域名

## 双向认证实现
### Nginx配置
```
    ...
    server {
        listen       443 ssl;
        # 服务端证书
        ssl_certificate      /home/peng/tls/server.crt;
        # 服务端私钥
        ssl_certificate_key  /home/peng/tls/server.key;

        # 是否需要验证客户端证书，off表示不验证客户端证书，也就是单向证书认证
        ssl_verify_client on;
        # CA证书或者客户端证书
        ssl_client_certificate /home/peng/tls/ca.crt;

        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;
        ...
    }
    ...
```
### cUrl测试
`curl -I --cacert ca.crt --cert client.crt --key client.key https://localhost`
### Java端实现，使用HttpClient
```
public class SSLHttpUtils {

    private static PoolingHttpClientConnectionManager connMgr;
    private static CloseableHttpClient httpclient;

    public static void init(String keyStoreFile, String keyStorePass, String trustStoreFile, String trustStorePass) {
        try {
            InputStream ksis = new FileInputStream(new File(keyStoreFile));
            InputStream tsis = new FileInputStream(new File(trustStoreFile));

            KeyStore ks = KeyStore.getInstance("PKCS12");
            ks.load(ksis, keyStorePass.toCharArray());

            Security.addProvider(new BouncyCastleProvider());
            KeyStore ts = KeyStore.getInstance("bks", "BC");
            ts.load(tsis, trustStorePass.toCharArray());

            SSLContext sslContext = SSLContexts.custom()
                    .loadKeyMaterial(ks, keyStorePass.toCharArray())
                    // 如果有服务器证书，并且需要验证服务端证书
                    //.loadTrustMaterial(ts, new TrustSelfSignedStrategy())
                    // 如果没有服务器证书，或者不需要验证服务端证书，可以采用自定义信任机制
                    .loadTrustMaterial(null, (TrustStrategy) (arg0, arg1) -> true)
                    .build();
            SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(sslContext);

            Registry<ConnectionSocketFactory> registry = RegistryBuilder
                    .<ConnectionSocketFactory>create()
                    .register("http", PlainConnectionSocketFactory.INSTANCE)
                    .register("https", sslsf).build();
            ksis.close();
            tsis.close();
            connMgr = new PoolingHttpClientConnectionManager(registry);
            httpclient = HttpClients.custom().setConnectionManager(connMgr).build();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static String post(String url, String requestBody) {
        HttpPost httpRequest = new HttpPost(url);
        String result = null;
        CloseableHttpResponse httpresponse = null;
        try {
            httpRequest.setEntity(new StringEntity(requestBody));
            httpresponse = httpclient.execute(httpRequest);
            int code = httpresponse.getStatusLine().getStatusCode();
            result = EntityUtils.toString(httpresponse.getEntity(), "UTF-8");
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }

}
```


