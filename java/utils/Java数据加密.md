# 数据加密

## 对称算法

### MD5 加密算法

> 1. MD5加密是一种将信息转换为32位16进制数的常用方法，通常用于保护密码和敏感数据。MD5加密的特点是其结果对于任何给定的数据都是唯一的，且几乎不可逆。

* apache.commons 加密工具类

    ```java
    import org.apache.commons.codec.digest.DigestUtils;
    1
    /** * MD5加密之方法一 * @explain 借助apache工具类DigestUtils实现 * @param str * 待加密字符串 * @return 16进制加密字符串 */
    public static String encryptToMD5(String str) {
    
        return DigestUtils.md5Hex(str);
    }　

    ```

* Spring 提供的加密工具类

    ```java
    import org.springframework.util.DigestUtils;
    import org.slf4j.logger;
    import org.slf4j.loggerFactory;
    public static String encrypt3ToMD5(String message) { 
    
        log.debug("MD5待加密字符串：\n" + message);
        String md5 = " ";
        try { 
        md5 = DigestUtils.md5DigestAsHex(message.getBytes("utf-8"))
        } catch (UnsupportedEncodingException e) { 
        e.printStackTrace();
        }
        log.degbug("MD5加密结果：\n" + md5)
        return md5;
    }
    ```

### SHA1 加密算法

> 1. SHA-1加密算法是加密算法的一种，它使用SHA-1算法、SHA-224算法、SHA-256算法、SHA-384算法和SHA-512算法等，并使用不同的哈希长度。
> 2. SHA-1加密算法与MD5加密算法类似，其特点是加密后的结果是唯一的，且不可逆。但SHA-1比MD5更安全，因为MD5加密算法的加密结果是固定的，而SHA加密算法的加密结果是动态的。

* apache.commons 加密工具类

```java
import org.apache.commons.codec.digest.DigestUtils;

public class SHA1Util {
    public static String encode(String str) {
        if (str == null) {
            throw new IllegalArgumentException("Input string cannot be null");
        }
        return DigestUtils.sha1Hex(str);
    }
}
```

* security 加密工具类

```java
import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import org.apache.commons.codec.binary.Hex;

public class SHA1Util {
    public static String encode(String str) {
        if (str == null) {
            throw new IllegalArgumentException("Input string cannot be null");
        }
        try {
            MessageDigest md = MessageDigest.getInstance("SHA-1");
            byte[] digest = md.digest(str.getBytes(StandardCharsets.UTF_8));
            return Hex.encodeHexString(digest);
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException("SHA-1 algorithm not found", e);
        }
    }
}

```

### DES 加密算法

> 1. DES算法是一种经典的对称密钥加密算法，广泛应用于金融、通信等领域的数据安全保护。
> 2. DES算法的核心是通过一系列复杂的置换和替换操作，将64位的明文和56位的密钥转换成64位的密文。下面将详细介绍DES算法的加密流程和关键步骤。
> 3. DES在安全性、效率和灵活性上比AES略差，但是也能保证安全，DES需要通过密钥进行加密，通过密钥进行解密，因此密钥很重要。

```java
import javax.crypto.Cipher;
import javax.crypto.SecretKey;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.DESKeySpec;
import javax.crypto.spec.IvParameterSpec;
import java.nio.charset.StandardCharsets;
import java.util.Base64;

public class DesEncryptUtil {

    private static final String DES_ALGORITHM = "DES/CBC/PKCS5Padding";

    /**
     * DES加密
     */
    public static byte[] encrypt(byte[] message, String key, String iv) throws Exception {
        Cipher cipher = Cipher.getInstance(DES_ALGORITHM);

        SecretKey secretKey = generateKey(key);
        IvParameterSpec ivParameterSpec = new IvParameterSpec(iv.getBytes(StandardCharsets.UTF_8));

        cipher.init(Cipher.ENCRYPT_MODE, secretKey, ivParameterSpec);
        return cipher.doFinal(message);
    }

    /**
     * DES加密并转换为Base64
     */
    public static String encryptToBase64(String message, String key, String iv) throws Exception {
        byte[] encrypted = encrypt(message.getBytes(StandardCharsets.UTF_8), key, iv);
        return Base64.getEncoder().encodeToString(encrypted);
    }

    /**
     * DES解密
     */
    public static String decrypt(byte[] encryptedMessage, String key, String iv) throws Exception {
        Cipher cipher = Cipher.getInstance(DES_ALGORITHM);

        SecretKey secretKey = generateKey(key);
        IvParameterSpec ivParameterSpec = new IvParameterSpec(iv.getBytes(StandardCharsets.UTF_8));

        cipher.init(Cipher.DECRYPT_MODE, secretKey, ivParameterSpec);
        byte[] decryptedBytes = cipher.doFinal(encryptedMessage);
        return new String(decryptedBytes, StandardCharsets.UTF_8);
    }

    /**
     * DES解密从Base64解码
     */
    public static String decryptFromBase64(String encryptedBase64, String key, String iv) throws Exception {
        byte[] decodedMessage = Base64.getDecoder().decode(encryptedBase64);
        return decrypt(decodedMessage, key, iv);
    }

    /**
     * 生成DES密钥
     */
    private static SecretKey generateKey(String key) throws Exception {
        DESKeySpec desKeySpec = new DESKeySpec(key.getBytes(StandardCharsets.UTF_8));
        SecretKeyFactory keyFactory = SecretKeyFactory.getInstance("DES");
        return keyFactory.generateSecret(desKeySpec);
    }
}

```

### AES 加密算法

> 1. AES算法是加密算法的一种，它使用128位、192位或256位的密钥进行加密和解密，并使用不同的加密模式和填充方式。

```java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.security.SecureRandom;
import java.util.Base64;

import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;

/**
 * DES加解密工具
 */
public class AesEncryptUtil {

    /**
     * 加密
     * @param encData 要加密的数据
     * @param secretKey 密钥
     * @param vector 向量
     * @throws Exception
     */
    public static String Encrypt(byte[] encData, String secretKey, String vector) throws Exception {

        if (secretKey == null) {
            return null;
        }

        KeyGenerator kg = KeyGenerator.getInstance("AES"); // AES算法
        SecureRandom secureRandom = SecureRandom.getInstance("SHA1PRNG"); // 使用 SHA1PRNG 作为 SecureRandom
        secureRandom.setSeed(secretKey.getBytes()); // 设置种子
        kg.init(256, secureRandom); // 指定生成密钥 256位

        //kg.init(256,new SecureRandom(secretKey.getBytes())); //linux系统用


        SecretKey sk = kg.generateKey(); // 获取AES的密钥
        byte[] raw = sk.getEncoded(); // 获取AES密钥的字节数组

        SecretKeySpec skeySpec = new SecretKeySpec(raw, "AES"); // 根据密钥的字节数组生成SecretKeySpec

        Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding"); // 获取AES/CBC/PKCS5Padding加密器示例
        IvParameterSpec iv = new IvParameterSpec(vector.getBytes()); // 生成向量

        cipher.init(Cipher.ENCRYPT_MODE, skeySpec, iv); // 初始化加密器

        byte[] encrypted = cipher.doFinal(encData); // 进行加密
        return Base64.getEncoder().encodeToString(encrypted); //Base64编码 返回加密后的数据
    }
}

/**
 * 解密
 * @param encryptedData 要解密的数据
 * @param secretKey 密钥
 * @param vector 向量
 */
public static String Decrypt(String encryptedData, String secretKey, String vector) throws Exception {
    if (secretKey == null) {
        return null;
    }

    KeyGenerator kg = KeyGenerator.getInstance("AES");
    SecureRandom secureRandom = SecureRandom.getInstance("SHA1PRNG");
    secureRandom.setSeed(secretKey.getBytes());
    kg.init(256, secureRandom);

    SecretKey sk = kg.generateKey();
    byte[] raw = sk.getEncoded();

    SecretKeySpec skeySpec = new SecretKeySpec(raw, "AES");
    Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
    IvParameterSpec iv = new IvParameterSpec(vector.getBytes());

    cipher.init(Cipher.DECRYPT_MODE, skeySpec, iv);

    byte[] encryptedBytes = Base64.getDecoder().decode(encryptedData);
    byte[] decrypted = cipher.doFinal(encryptedBytes);
    return new String(decrypted, StandardCharsets.UTF_8);
}

```

## 非对称算法
