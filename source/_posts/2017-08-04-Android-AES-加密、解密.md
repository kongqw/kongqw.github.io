---
title: Android AES 加密、解密
date: 2017-08-04 15:32:49
tags: 
- 加密
- AES
- Android
---

[AES加密介绍](https://baike.baidu.com/item/aes/5903?fr=aladdin)

ASE 加密、解密的关键在于秘钥、只有使用加密时使用的秘钥，才可以解密。

生成秘钥的代码网上一大堆，下面的代码可生成一个秘钥

``` java
private SecretKey generateKey(String seed) throws Exception {
    // 获取秘钥生成器
    KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");
    // 通过种子初始化
    SecureRandom secureRandom = new SecureRandom();
    secureRandom.setSeed(seed.getBytes("UTF-8"));
    keyGenerator.init(128, secureRandom);
    // 生成秘钥并返回
    return keyGenerator.generateKey();
}
```

然后使用秘钥进行**加密**

``` java
private byte[] encrypt(String content, SecretKey secretKey) throws Exception {
    // 秘钥
    byte[] enCodeFormat = secretKey.getEncoded();
    // 创建AES秘钥
    SecretKeySpec key = new SecretKeySpec(enCodeFormat, "AES");
    // 创建密码器
    Cipher cipher = Cipher.getInstance("AES");
    // 初始化加密器
    cipher.init(Cipher.ENCRYPT_MODE, key);
    // 加密
    return cipher.doFinal(content.getBytes("UTF-8"));
}
```

**解密**

``` java
private byte[] decrypt(byte[] content, SecretKey secretKey) throws Exception {
    // 秘钥
    byte[] enCodeFormat = secretKey.getEncoded();
    // 创建AES秘钥
    SecretKeySpec key = new SecretKeySpec(enCodeFormat, "AES");
    // 创建密码器
    Cipher cipher = Cipher.getInstance("AES");
    // 初始化解密器
    cipher.init(Cipher.DECRYPT_MODE, key);
    // 解密
    return cipher.doFinal(content);
}
```

通常，如果加密和解密都是在同一个平台，比较简单，我们生成一个秘钥以后，将秘钥保存到本地，解密的时候直接获取本地的秘钥来解密就可以了，通常的使用场景为本地将xxx文件加密后上传保存或备份，需要的时候，下载再解密。这样上传的文件比较安全。


看上去很完美，下面问题来了，上述生产秘钥的方法，每次执行生成的秘钥都是不一样的。也就是说，加密时的秘钥如果没有保存到本地，解密的时候再次调用上述方法生成一个秘钥，那么将无法解密。

解决办法也有，使用如下方式生成秘钥，只要种子一样，生成的秘钥就是一样的。

``` java
private SecretKey generateKey(String seed) throws Exception {
    // 获取秘钥生成器
    KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");
    // 通过种子初始化
    SecureRandom secureRandom = SecureRandom.getInstance("SHA1PRNG", "Crypto");
    secureRandom.setSeed(seed.getBytes("UTF-8"));
    keyGenerator.init(128, secureRandom);
    // 生成秘钥并返回
    return keyGenerator.generateKey();
}
```

但是Android N（7.0）以后将不再支持，移除了`Crypto`。

``` Log
E/System:  ********** PLEASE READ ************ 
E/System:  * 
E/System:  * New versions of the Android SDK no longer support the Crypto provider.
E/System:  * If your app was relying on setSeed() to derive keys from strings, you
E/System:  * should switch to using SecretKeySpec to load raw key bytes directly OR
E/System:  * use a real key derivation function (KDF). See advice here : 
E/System:  * http://android-developers.blogspot.com/2016/06/security-crypto-provider-deprecated-in.html 
E/System:  *********************************** 
W/System.err: java.security.NoSuchProviderException: no such provider: Crypto
```

Google也对应给出了解决方案，详见 [Security "Crypto" provider deprecated in Android N](https://android-developers.googleblog.com/2016/06/security-crypto-provider-deprecated-in.html)


下面介绍另一种解决方案，我们不用种子生成秘钥，直接将password作为秘钥。

> 关于Android和IOS的同步问题，小伙伴也可以借鉴 [AES加密 - iOS与Java的同步实现](http://www.jianshu.com/p/df828a57cb8f)
>
> 如下方法 Android测试可行，IOS如果有小伙测试有问题也可以反馈给我。


**加密**

``` java
private byte[] encrypt(String content, String password) throws Exception {
    // 创建AES秘钥
    SecretKeySpec key = new SecretKeySpec(password.getBytes(), "AES/CBC/PKCS5PADDING");
    // 创建密码器
    Cipher cipher = Cipher.getInstance("AES");
    // 初始化加密器
    cipher.init(Cipher.ENCRYPT_MODE, key);
    // 加密
    return cipher.doFinal(content.getBytes("UTF-8"));
}
```
    
**解密**

``` java
private byte[] decrypt(byte[] content, String password) throws Exception {
    // 创建AES秘钥
    SecretKeySpec key = new SecretKeySpec(password.getBytes(), "AES/CBC/PKCS5PADDING");
    // 创建密码器
    Cipher cipher = Cipher.getInstance("AES");
    // 初始化解密器
    cipher.init(Cipher.DECRYPT_MODE, key);
    // 解密
    return cipher.doFinal(content);
}
```

**注意：**必须必须要注意的是，这里的`password`的长度，必须为`128`或`192`或`256`bits.也就是`16`或`24`或`32`byte。否则会报出如下错误：

``` Log
com.android.org.bouncycastle.jcajce.provider.symmetric.util.BaseBlockCipher$1: Key length not 128/192/256 bits.
```

> 至于数字、字母、中文都各自占几个字节，相信小伙伴的都是了解的，就不废话了。
> 也可以`byte[] password = new byte[16/24/32];`

最后：至于最开始提到生成秘钥的方法，为什么种子相同，所生成的秘钥不同，还没看具体实现。有知道的小伙伴还请先指点一二。



