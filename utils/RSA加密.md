## 前言
做了一个前端JS使用RSA加密，后端JAVA使用RSA解密的需求，记下这个RSA工具类

## RasUtils工具类

```java
package cn.dave.sms.utils;

import sun.misc.BASE64Decoder;
import sun.misc.BASE64Encoder;

import javax.crypto.Cipher;
import java.security.*;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;

/**
 * @ClassName: RSAUtils
 * @Description: RSA工具类
 * @author: dave
 */
public class RSAUtils {
    
    /**
     * @Title:        getKeyPair
     * @Description: 生成秘钥对
     * @param
     * @return:       java.security.KeyPair
     * @author        dave
     */
    public static KeyPair getKeyPair() throws Exception {
         
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("RSA");
        keyPairGenerator.initialize(2048);
        KeyPair keyPair = keyPairGenerator.generateKeyPair();
        return keyPair;
    }
    
    /**
     * @Title:        getPublicKey
     * @Description: 获取公钥(Base64编码)
     * @param keyPair
     * @return:       java.lang.String
     * @author        dave
     */
    static String getPublicKey(KeyPair keyPair) {
        PublicKey publicKey = keyPair.getPublic();
        byte[] bytes = publicKey.getEncoded();
        return new BASE64Encoder().encode((bytes));
    }
    
    /**
     * @Title:        getPrivateKey
     * @Description:   获取私钥(Base64编码)
     * @param keyPair
     * @return:       java.lang.String
     * @author        dave
     */
    static String getPrivateKey(KeyPair keyPair) {
      
        PrivateKey privateKey = keyPair.getPrivate();
        byte[] bytes = privateKey.getEncoded();
        return new BASE64Encoder().encode((bytes));
    }

    /**
     * @Title:        string2PublicKey
     * @Description:   将Base64编码后的公钥转换成PublicKey对象
     * @param pubStr
     * @return:       java.security.PublicKey
     * @author        dave
     */
    static PublicKey string2PublicKey(String pubStr) throws Exception {
        byte[] keyBytes = new BASE64Decoder().decodeBuffer((pubStr));
        X509EncodedKeySpec keySpec = new X509EncodedKeySpec(keyBytes);
        KeyFactory keyFactory = KeyFactory.getInstance("RSA");
        return keyFactory.generatePublic(keySpec);
    }

    /**
     * @Title:        string2PrivateKey
     * @Description:   将Base64编码后的私钥转换成PrivateKey对象
     * @param priStr
     * @return:       java.security.PrivateKey
     * @author        dave
     */     
    static PrivateKey string2PrivateKey(String priStr) throws Exception {
        byte[] keyBytes = new BASE64Decoder().decodeBuffer((priStr));
        PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(keyBytes);
        KeyFactory keyFactory = KeyFactory.getInstance("RSA");
        return keyFactory.generatePrivate(keySpec);
    }

    /**
     * @Title:        publicEncrypt
     * @Description:   公钥加密
     * @param content
     * @param publicKeyStr
     * @return:       java.lang.String
     * @author        dave
     * @date          2020/10/24 14:10
     */
    public static String publicEncrypt(String content, String publicKeyStr) throws Exception {
        PublicKey publicKey = RSAUtils.string2PublicKey(publicKeyStr);
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.ENCRYPT_MODE, publicKey);
        byte[] bytes = cipher.doFinal(content.getBytes());
        return new BASE64Encoder().encode(bytes);
    }

    /**
     * @Title:        privateDecrypt
     * @Description:   私钥解密
     * @param content
     * @param privateKeyStr
     * @return:       byte[]
     * @author        dave
     * @date          2020/10/24 14:11
     */
    public static byte[] privateDecrypt(String content, String privateKeyStr) throws Exception {
        PrivateKey privateKey = RSAUtils.string2PrivateKey(privateKeyStr);
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.DECRYPT_MODE, privateKey);
        byte[] bytes = cipher.doFinal(new BASE64Decoder().decodeBuffer(content));
        return bytes;
    }


    public static void main(String[] args) {
        try {
            //===============生成公钥和私钥，公钥传给客户端，私钥服务端保留==================
            KeyPair keyPair = RSAUtils.getKeyPair();
            String publicKeyStr = RSAUtils.getPublicKey(keyPair);
            String privateKeyStr = RSAUtils.getPrivateKey(keyPair);
            System.out.println("RSA公钥Base64编码:" + publicKeyStr);
            System.out.println("RSA私钥Base64编码:" + privateKeyStr);

            //=================开始加密=================
            String message = "dave";
            //用公钥加密
            String publicEncrypt = RSAUtils.publicEncrypt(message, publicKeyStr);
            System.out.println("公钥加密并Base64编码的结果：" + publicEncrypt);


            //===================开始解密================
            //用私钥解密
            byte[] privateDecrypt = RSAUtils.privateDecrypt(publicEncrypt, privateKeyStr);
            //解密后的明文
            System.out.println("解密后的明文: " + new String(privateDecrypt));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}




```
生成公钥和私钥，公钥存储在JS端，供JS加密使用；
私钥保存在Java后端，供解密用；

## js页面

```js
<!doctype html>
<html>
  <head>
    <title>JavaScript RSA Encryption</title>
    <script src="http://code.jquery.com/jquery-1.8.3.min.js"></script>
    <script src="jsencrypt.js"></script>
    <script type="text/javascript">

      // Call this code when the page is done loading.
      $(function() {

        // Run a quick encryption/decryption when they click.
        $('#testme').click(function() {

          // Encrypt with the public key...
          var encrypt = new JSEncrypt();
          encrypt.setPublicKey('刚才Java生成的公钥');
          var encrypted = encrypt.encrypt('要加密的内容');
          console.log(encrypted);
        });
      });
    </script>
  </head>
  <body>
    
  </body>
</html>

```