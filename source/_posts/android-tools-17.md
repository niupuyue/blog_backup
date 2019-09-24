---
title: android奇技淫巧 17 Android登录时使用到的登录信息加密和解密
date: 2019-09-17 21:55:10
tags:
  - android
---

Android App用户在登陆的时候，需要传递用户的一些隐私信息，比如手机号，比如账号，比如密码等，那么我们需要将这些数据进行加密传递个后台，不然的话如果被别人拦截到很容易造成用户信息的泄露

<!--more-->

我们在进行登录操作的时候，为了保护用户的信息安全，我们通常会对用户的传输信息进行加密操作，那么这篇博客就是专门把我们目前正在使用的加密总结一下

> 所有的信息都是经过处理的

在当前的项目中加密的步骤分为以下几个内容

1. 将用户的手机和密码封装成对象类型
2. 将对象类型转换成Json字符串
3. 将Json字符串换成byte[]数组
4. 声明一个AES加密的会话密钥
5. 加密报文，并且设置加密模式
6. 根据和服务端约定好的公钥生成一个公钥
7. 使用公钥对密钥进行加密
8. 最后把加密的密钥和密文发送给服务器

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
        try {
            // 生成会话密钥
            KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");// 使用AES加密算法
            keyGenerator.init(128);  // 使用128位长度的密钥
            mode.secretKey = keyGenerator.generateKey();  // 生成密钥

            // 通信报文，用户登录信息等
            byte[] plaintext = text.getBytes();
            byte[] pubk = Base64.decode(PUBLIC_KEY.getBytes(),Base64.DEFAULT);  // 公钥，预先设置在App中

            // 加密报文
            Cipher secretKeyCiper = Cipher.getInstance("AES");// 使用AES加密算法
            secretKeyCiper.init(Cipher.ENCRYPT_MODE,mode.secretKey);  // 设置加密模式
            byte[] ciphertext = secretKeyCiper.doFinal(plaintext);

            // 读取公钥
            KeyFactory kf = KeyFactory.getInstance("RSA"); // 使用RSA算法
            X509EncodedKeySpec x509ks = new X509EncodedKeySpec(pubk);
            PublicKey publicKey = kf.generatePublic(x509ks);

            // 用公钥对密钥进行加密
            Cipher publicKeyCipher = Cipher.getInstance("RSA/ECB/PKCS1Padding");  // 使用RSA算法
            publicKeyCipher.init(Cipher.ENCRYPT_MODE,publicKey); // 加密模式
            byte[] plainkey = mode.secretKey.getEncoded();  // 密钥铭文
            byte[] cipherkey = publicKeyCipher.doFinal(plainkey);// 密钥密文

            // 最后把加密的密钥cipherkey和密文ciphertext发送给服务器
            mode.cipherText = new String(Base64.encode(ciphertext,Base64.NO_WRAP));
            mode.cipherKey = new String(Base64.encode(cipherkey,Base64.NO_WRAP));
        }catch (Exception ex){
            ex.printStackTrace();
        }

        return mode;
    }

}
```

[demo地址](https://github.com/niupuyue/blog_demo_android/tree/master/LoginSecret)