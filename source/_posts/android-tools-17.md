---
title: android奇技淫巧 17 Android登录时使用到的登录信息加密和解密
date: 2019-09-17 21:55:10
tags:
  - android
---

Android App用户在登陆的时候，需要传递用户的一些隐私信息，比如手机号，比如账号，比如密码等，那么我们需要将这些数据进行加密传递个后台，不然的话如果被别人拦截到很容易造成用户信息的泄露

<!--more-->

我们在进行登录操作的时候，为了保护用户的信息安全，我们通常会对用户的传输信息进行加密操作，那么这篇博客就是专门把我们目前正在使用的加密总结一下

> 所有的信息都是经过处理的  使用的方式是  AES算法加密文本(需要传输的内容)  使用RSA算法加密AES的密钥

在当前的项目中加密的步骤分为以下几个内容

1. 首先获取到输入框输入的内容phone和pwd ，将获取到的phone和pwd封装成一个对象PhoneLoginModel
2. 将PhoneLoginModel转换成Json字符串
    1. 通过AES算法生成一个会话的密钥sercetKey 
    2. 将公钥转换成PEM格式的byte数组publicKeyArray
    3. 将publicKeyArray转换成PublicKey对象
    4. 通过Cipher对象将PublicKey对象转换成byte数组的密文cipherKey
    5. 将json字符串换成byte数组plaintTextArray
    6. 将plaintTextArray和secretKy结合生成对应的加密报文byte数组plaintKey
    7. 将生成的机密报文cipherKey和PlaintKey封装成对象LoginInfoModel
    8. 将LoginInfoModel对象通过Post请求发送给后台

具体代码实现

#### PhoneLoginModel
```
public class PhoneLoginModel {

    public String phone = "";
    public String password = "";

    public String verkey = "";

}

```

#### EncryptionInfoMode

```
public class EncryptionInfoMode {

    public SecretKey secretKey = null;

    /**
     * 私钥
     */
    public String cipherKey = "";

    /**
     * 加密内容
     */
    public String cipherText = "";

}
```

#### MainActivity

```
public class MainActivity extends AppCompatActivity {

    private EditText phone;
    private EditText password;
    private Button submit;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);


        phone = findViewById(R.id.phone);
        password = findViewById(R.id.password);
        submit = findViewById(R.id.submit);
        submit.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                String strPhone = phone.getText().toString();
                String strPassword = password.getText().toString();
                PhoneLoginModel model = new PhoneLoginModel();
                model.phone = strPhone;
                model.password = strPassword;
                EncryptionInfoMode encryptionInfoMode = BaseUtility.getEncryptionInfoMode(model);
                // 可以直接将encryptionInfoMode对象传递给后台，让后台解密，执行登录操作
            }
        });

    }
}
```

#### BaseUtility

```
public class BaseUtility {
    // 模拟公钥
    public static final String PUBLIC_KEY =
            "MIIBIjANBgkqhkiG9w0IOPDMANIDE<DISNCEWSOCAQEAmQEjEdEXNZewgCZt40SAtYx2p/+91Cesx6Ns3sSg22NQxOHW1Mnt/OBaAEyvZu733PlMQQGkm6AkJtswRr61" +
                    "Z5pRk4ZKMIxj8sx7m1+DpnJr5ZJDZ3fd98e6d3+1d4e+8e5=d74MY0NjRGvPVKWiVOHwXaY823RJAAfG1Ks3a4KL1E6RBXLjNaN85uthK05QLuCGSoXuyc2pQRuuOykt" +
                    "EjYiqRofwJBTkYTQ5nGPuDVzIzPmlmu10WMmY39dMXr1l00EVPobdnuuQu+dHsOqMrg0cRkMY/344SK7KKCI74YMKARDaXcRF2Kdulg8l4l46GG/29HBaUo/rvVlfFE2fwIDAQAB";

    public static EncryptionInfoMode getEncryptionInfoMode(PhoneLoginModel model){
        EncryptionInfoMode encryptionInfoMode = new EncryptionInfoMode();

        String text = new Gson().toJson(model,PhoneLoginModel.class);
        encryptionInfoMode = getEncryptionModel(text);
        return encryptionInfoMode;
    }

    public static EncryptionInfoMode getEncryptionModel(String text){
        EncryptionInfoMode mode = new EncryptionInfoMode();
        if (TextUtils.isEmpty(text)){
            return mode;
        }
    
        /**
         * 这里我们采用的加密方式总的来说还是通过AES的方式，因为AES的加密方式最简单也最迅速，但是如果仅仅使用AES的话
         * 会造成很多安全性的问题，因为一旦AES的密钥被破解，那么所有的信息都将会被暴露出来，所以我们采用的方式是AES加密密文，RSA加密AES密钥
         */
        try {
            // 生成AES加密算法对象
            KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");
            // 使用长度为128的密钥
            keyGenerator.init(128);
            // 生成简单AES密钥(这里我们还没有AES密钥进行加密处理，因为我们还要用这个对象先对密文进行加密)
            mode.secretKey = keyGenerator.generateKey();

            // 初始化数据，包括初始化文本(将需要传输的文本转换成byte数组)，对公钥进行解码(这里的公钥使用的是Base64已经加密过了)
            byte[] plaintText = text.getBytes();
            byte[] publicKeyText = Base64.decode(PUBLIC_KEY,Base64.DEFAULT);

            // 下面的代码分为两部分进行，第一部分是对密文的加密，也就是对需要传输的文本执行加密算法
            Cipher cipher = Cipher.getInstance("AES");
            // 设置加密模式
            cipher.init(Cipher.ENCRYPT_MODE,mode.secretKey);
            // 将密文转换成byte数组
            byte[] plaint = cipher.doFinal(plaintText);
            // 到这里，密文的加密已经完成了，但是因为AES加密的方式比较简单，很容易被别人捕获并且破解(破解是要通过密钥才能破解的)，所以当我们传递AES密钥的时候，需要对密钥进行加密处理


            // 第二部分，对AES的密钥进行加密
            // 声明RSA算法
            KeyFactory kf = KeyFactory.getInstance("RSA");
            X509EncodedKeySpec x509ks = new X509EncodedKeySpec(publicKeyText);
            // 生成一个PublicKey对象
            PublicKey pk = kf.generatePublic(x509ks);
            // 使用RSA算法对AES密钥进行加密
            Cipher publicKeyCipher = Cipher.getInstance("RSA/ECB/PKCS1Padding");
            // 设置加密模式，并且把需要加密的对象传递过来
            publicKeyCipher.init(Cipher.ENCRYPT_MODE,pk);
            // 获取AES密钥的明文
            byte[] plainKey = mode.secretKey.getEncoded();
            // 设置AES密钥的密文
            byte[] plaink = publicKeyCipher.doFinal(plainKey);

            // 最后需要将AES的密文和AES的密钥在使用Base64进行加密之后，就能直接传输给后台了
            mode.cipherText = new String(Base64.encode(plaint,Base64.NO_WRAP));
            mode.cipherKey = new String(Base64.encode(plaink,Base64.NO_WRAP));
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (NoSuchPaddingException e) {
            e.printStackTrace();
        } catch (InvalidKeyException e) {
            e.printStackTrace();
        } catch (BadPaddingException e) {
            e.printStackTrace();
        } catch (IllegalBlockSizeException e) {
            e.printStackTrace();
        } catch (InvalidKeySpecException e) {
            e.printStackTrace();
        }

        return mode;
    }

}
```

[demo地址](https://github.com/niupuyue/blog_demo_android/tree/master/LoginSecret)

下面是关于加密内容的补充，大部分都是以图文的形式展现的

## 补充知识点

加密的分类：
1. 按可逆性：加密分为可逆算法和不可逆算法
2. 按对称性：加密可分为对称算法和非对称算法

一般加密分为以下几种，
1. Base64编码算法(可逆)
2. MD5加密(不可逆)  (还有一个sha1值)
3. DES加密 （对称，可逆）
4. AES加密 (对称，可逆)
5. RSA加密(非对称，可逆)

### 对称
对称加密算法是比较传统的加密体质，即通信双方在加密/解密的过程中使用他们共享的单一密钥，优点是算法简单，加密速度快，但是安全性较差。最常用的对称密码算法是DES，但是DES密钥的长度较短，已经不适合现在分布式开放网络对数据加密安全性的要求了。目前对称搞基数据加密标准AES取代了DES，使用的也比较多

### 非对称
非对称加密用于加/解密钥不同(公钥加密，私钥解密),密钥管理简单，RSA是非对称加密算法中最著名的共要密码算法，但是由于RSA算法都是进行大数计算，使得RSA最快的情况也会比AES慢，但是安全性比较高

## Base64算法
Base64其实不是安全领域的加密算法，因为他的加密解密算法都是公开的，Base64编码本质上是一种将二进制数据转成文本数据的方案，用处就是将一些不适合传输的数据内容进行编码来适合传输

字符串进行Base64编码
```
String encodedString = Base64.encode("paulniu".getBytes(),Base64.DEFAULT);
```
字符串进行Base64解码
```
String decodedString = new String(Base64.decode(encodedString,Base64.DEFAULT));
// 解析出来decodedString就是paulniu
```

## MD5算法
这是一种单向加密算法，只能加密，无法解密，用于密码存储等。对于MD5的安全性，网上有很多MD5解密的网站，破解方式一般都是采用穷举法。就是将所有MD5字典都遍历一遍，可想而知，如果MD5的内容越多，解密越困难。所以一般情况下对数据采用多次MD5加密或者采用加权法(就是加一段独有的字符串再进行加密)

```
public static String md5(String ss){
    if(TextUtils.isEmpty(ss)){
        return null;
    }
    MessageDigest md5 = null;
    try{
        md5 = MessageDigest.getInstance("MD5");
        byte[] bytes = md5.digest(ss.getBytes());
        String result = "";
        for(byte b:bytes){
            String temp = Integer.toHexString(b & 0xff);
            if(temp.length() == 1){
                temp = "0" + temp;
            }
            result+= temp;
        }
        return result;
    }catch(Exception e){
        e.printStackTrace();
    }
    return "";
}
```

## 对称加密
对称加密密钥是唯一的，加密和解密都是同一个密钥，AES速度上回避RSA快，但是因为只有一个密钥，如果这个密钥一旦被破解，那么所有的信息都将被泄露

使用场景：
- 本地数据加密(例如加密android中的SharedPreferences里面的敏感数据)
- 网络传输，登录接口使用post请求(安全性不好)
- 加密用户登陆结果信息并序列化到本地磁盘(例如将User对象序列化到本地磁盘，下次登录时反序列化到内存中)
- 网页交互数据加密

```
    try {
            Cipher cipher = Cipher.getInstance("AES");
            // 创建密钥
            SecretKey key = KeyGenerator.getInstance("AES").generateKey();
            // 设置操作模式
            cipher.init(Cipher.ENCRYPT_MODE,key);
            // 执行
            byte[] res = cipher.doFinal("paulniu".getBytes());
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (NoSuchPaddingException e) {
            e.printStackTrace();
        } catch (InvalidKeyException e) {
            e.printStackTrace();
        } catch (BadPaddingException e) {
            e.printStackTrace();
        } catch (IllegalBlockSizeException e) {
            e.printStackTrace();
        }
```