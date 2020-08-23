## 前言

本文记录 md5 和 des 加密

## md5工具类
```java
/**
     * Java MD5加密，并转化为16进制字符
     * @param buf
     * @return
     */
    public static String EncoderByMd5(String buf) {
        try {
            MessageDigest digist = MessageDigest.getInstance("MD5");
            byte[] rs = digist.digest(buf.getBytes("UTF-8"));
            StringBuffer digestHexStr = new StringBuffer();
            for (int i = 0; i < 16; i++) {
                digestHexStr.append(byteHEX(rs[i]));
            }
            return digestHexStr.toString();
        } catch (Exception e) {

        }
        return null;

    }

    public static String byteHEX(byte ib) {
        char[] Digit = { '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F' };
        char[] ob = new char[2];
        ob[0] = Digit[(ib >>> 4) & 0X0F];
        ob[1] = Digit[ib & 0X0F];
        String s = new String(ob);
        return s;
    }
```

## des加密
### 导入依赖
这里使用了一个工具包:

```java
<dependency>
  <groupId>commons-codec</groupId>
      <artifactId>commons-codec</artifactId>
  <version>1.12</version>                          
</dependency>
```
### 工具类
```java
package cn.dave.sms.utils;

import org.apache.commons.codec.DecoderException;
import org.apache.commons.codec.binary.Hex;

import javax.crypto.*;
import javax.crypto.spec.DESKeySpec;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.ObjectOutputStream;
import java.io.UnsupportedEncodingException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.security.SecureRandom;
import java.security.spec.InvalidKeySpecException;
import java.util.Map;

public class DesUtil {

    // 密钥，是加密解密的凭据，长度为8的倍数
    private static final String PASSWORD_CRYPT_KEY = "D39970C3";
    private final static String DES = "DES";

    /**
     * 加密
     * @param key
     * @param data
     * @return
     */
    public static String encryption(String key , Map<String,Object> data){
        return bytesToHexString(encryption(key.getBytes(),map2Byte(data)));
    }

    /**
     * 加密
     * @param key
     * @param data
     * @return
     */
    public static String encryption(String key , String data){
        return arr2HexStr(encryption(key.getBytes(),data.getBytes()),false);
    }
    /**
     * 加密
     * @param key
     * @param data
     * @return
     * @throws Exception
     */
    private static byte[] encryption(byte[] key,byte[] data) {

        try {
            SecureRandom sr = new SecureRandom();
            // 从原始密匙数据创建DESKeySpec对象
            DESKeySpec dks = new DESKeySpec(key);
            // 创建一个密匙工厂，然后用它把DESKeySpec转换成一个SecretKey对象
            SecretKeyFactory keyFactory = SecretKeyFactory.getInstance(DES);
            SecretKey secretKey = keyFactory.generateSecret(dks);
            // Cipher对象实际完成加密操作
            Cipher cipher = Cipher.getInstance(DES);
            // 用密匙初始化Cipher对象
            cipher.init(Cipher.ENCRYPT_MODE, secretKey, sr);
            // 正式执行加密操作
            return cipher.doFinal(data);
        }catch (Exception e){
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 解密
     * @param src
     * @param key
     * @return
     */
    public static String decrypt(String src, String key){
        byte[] decrypt = decrypt(hexItr2Arr(src), key.getBytes());
        return arr2Str(decrypt,"UTF-8");
    }

    public static byte[] decrypt(byte[] src, byte[] key){
        try{
            // DES算法要求有一个可信任的随机数源
            SecureRandom sr = new SecureRandom();
            // 从原始密匙数据创建一个DESKeySpec对象
            DESKeySpec dks = new DESKeySpec(key);

            // 创建一个密匙工厂，然后用它把DESKeySpec对象转换成
            // 一个SecretKey对象
            SecretKeyFactory keyFactory = SecretKeyFactory.getInstance(DES);

            SecretKey securekey = keyFactory.generateSecret(dks);

            // Cipher对象实际完成解密操作
            Cipher cipher = Cipher.getInstance(DES);

            // 用密匙初始化Cipher对象
            cipher.init(Cipher.DECRYPT_MODE, securekey, sr);

            // 现在，获取数据并解密
            // 正式执行解密操作
            return cipher.doFinal(src);
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (InvalidKeyException e) {
            e.printStackTrace();
        } catch (NoSuchPaddingException e) {
            e.printStackTrace();
        } catch (BadPaddingException e) {
            e.printStackTrace();
        } catch (InvalidKeySpecException e) {
            e.printStackTrace();
        } catch (IllegalBlockSizeException e) {
            e.printStackTrace();
        }
        return null;
    }
    /**
     * byte数组转换为16进制
     * @param bArr
     * @return
     */
    public static String bytesToHexString(byte[] bArr) {
        StringBuffer sb = new StringBuffer(bArr.length);
        String sTmp;

        for (int i = 0; i < bArr.length; i++) {
            sTmp = Integer.toHexString(0xFF & bArr[i]);
            if (sTmp.length() < 2)
                sb.append(0);
            sb.append(sTmp.toUpperCase());
        }

        return sb.toString();
    }

    /**
     * byte数组转化为16进制字符串
     * @param arr 数组
     * @param lowerCase 转换后的字母为是否为小写 可不传默认为true
     * @return
     */
    public static String arr2HexStr(byte[] arr,boolean lowerCase){
        return Hex.encodeHexString(arr, lowerCase);
    }

    /**
     * 将16进制字符串转换为byte数组
     * @param hexItr 16进制字符串
     * @return
     */
    public static byte[] hexItr2Arr(String hexItr){
        try {
            return Hex.decodeHex(hexItr);
        } catch (DecoderException e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 将byte数组转换为指定编码格式的普通字符串
     * @param arr byte数组
     * @param charset 编码格式 可不传默认为Charset.defaultCharset()
     * @return
     * @throws UnsupportedEncodingException
     */
    public static String arr2Str(byte[] arr,String charset)  {
        try {
            return new String(arr,charset);
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * map转换为byte数组
     * @param map
     * @return
     */
    public static byte[] map2Byte(Map<String,Object> map){
        try {
            ByteArrayOutputStream os = new ByteArrayOutputStream();
            ObjectOutputStream oos = new ObjectOutputStream(os);
            oos.writeObject(map);
            return os.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```
