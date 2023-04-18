+++
title = "C#与java aes加解密"
tags = ["c-sharp","java","aes加解密","搬运"]
date = "2023-04-18"
update = "2023-04-18"
enableGitalk = false
draft = false
+++


###### 最近在跟三方对接 对方采用AES加解密 作为一个资深neter Ctrl CV 是我最大的优点 所以我义正言辞的问他们要了demo

![1541998119616](https://img2018.cnblogs.com/blog/553504/201811/553504-20181112175040954-864870987.png)

###### java demo代码：
``` java
    public class EncryptDecryptTool
    {
        private static final String defaultCharset = "UTF-8";
        private static final String KEY_AES = "AES";
        private static final String KEY_MD5 = "MD5";
        private static MessageDigest md5Digest;
    
        static {
            try {
                md5Digest = MessageDigest.getInstance(KEY_MD5);
            } catch (NoSuchAlgorithmException e) {
                //
            }
        }
    
    
    public String encrypt(String message, String key) throws Exception
    {
        return doAES(message, key, Cipher.ENCRYPT_MODE);
    }
    
    
    public String decrypt(String message, String key) throws Exception
    {
            return doAES(message, key, Cipher.DECRYPT_MODE);
    }
    
    /**
     * 加解密
     *
     * @param data
     * @param key
     * @param mode
     * @return
     * @throws NoSuchPaddingException 
     * @throws NoSuchAlgorithmException 
     */
    private static String doAES(String data, String key, int mode) throws Exception
    {
     
                if (StringUtils.isBlank(data) || StringUtils.isBlank(key)) {
            return null;
        }
        boolean encrypt = mode == Cipher.ENCRYPT_MODE;
                byte[]
        content;
                if (encrypt) {
            content = data.getBytes(defaultCharset);
        } else {
            content = Base64.decodeBase64(data.getBytes());
        }
        SecretKeySpec keySpec = new SecretKeySpec(md5Digest.digest(key.getBytes(defaultCharset)), KEY_AES);
                Cipher cipher = Cipher.getInstance(KEY_AES);// 创建密码器
    cipher.init(mode, keySpec);// 初始化
                byte[] result = cipher.doFinal(content);
                if (encrypt) {
                    return new String(Base64.encodeBase64(result, false), defaultCharset);
                } else {
                    return new String(result, defaultCharset);
                }
           
        }
    }
    
       EncryptDecryptTool tool = new EncryptDecryptTool();
        try
        {
          //这里key的位数是个坑   之前找的一堆资料   java C#通用版啥的   都说key一定要是16位的   结果后来我发现 靠
            String msg = tool.decrypt("{\"Name\":\"20180122T155221\",\"OrderNo\":\"Test1000012059021180000008153\"}", "64个英文字母");
            System.out.println(msg);
        }
        catch (Exception e)
        {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
``` 

##### 哇咔咔 这不是大名鼎鼎的java嘛 但这难不倒我 不是还有IKVM 照样Ctrl CV

![1542000611421](https://img2018.cnblogs.com/blog/553504/201811/553504-20181112175040691-256096757.png)

##### 新建一个.NET Standard项目NuGet 安装IKVM

![1536719957239](https://img2018.cnblogs.com/blog/553504/201809/553504-20180912104232952-823857838.png)

##### C#主要代码
``` c#
    public class EncryptDecryptTool
        {
            private static readonly string KEY_AES = "AES";
            private static readonly string KEY_MD5 = "MD5";
            private static MessageDigest md5Digest;
    
            static EncryptDecryptTool()
            {
                try
                {
                    md5Digest = MessageDigest.getInstance(KEY_MD5);
                }
                catch (NoSuchAlgorithmException e)
                {
                    //
                }
            }
    
            public string encrypt(string message, string key)
            {
                return doAES(message, key, Cipher.ENCRYPT_MODE);
            }
    
            public string decrypt(string message, string key)
            {
                return doAES(message, key, Cipher.DECRYPT_MODE);
            }
    
            /// <summary>
            /// 
            /// </summary>
            /// <param name="data"></param>
            /// <param name="key"></param>
            /// <param name="mode"></param>
            /// <returns></returns>
            private static string doAES(string data, string key, int mode)
            {
    
    
                bool encrypt = mode == Cipher.ENCRYPT_MODE;
                byte[] content;
                if (encrypt)
                {
                    content = Encoding.UTF8.GetBytes(data);
                }
                else
                {
                    content = Convert.FromBase64String(data);
                }
                SecretKeySpec keySpec = new SecretKeySpec(md5Digest.digest(Encoding.UTF8.GetBytes(key)), KEY_AES);
                Cipher cipher = Cipher.getInstance(KEY_AES);// 创建密码器
                cipher.init(mode, keySpec);// 初始化
                byte[] result = cipher.doFinal(content);
                if (encrypt)
                {
                    return Convert.ToBase64String(result);
                }
                else
                {
                    return Encoding.UTF8.GetString(result);
                }
    
            }
           
        }
      static void Main(string[] args)
            {
                var encryptDecryptTool = new EncryptDecryptTool();
                Console.WriteLine(encryptDecryptTool.encrypt("jack","64位密钥"));
            }
```    

##### 运行NetCore控制台

![1542001527509](https://img2018.cnblogs.com/blog/553504/201811/553504-20181112175040313-1186847132.png)

![1542001571646](https://img2018.cnblogs.com/blog/553504/201811/553504-20181112175039754-1429131478.png)

##### 这是Net Framework Net Core以及IKVM不得不说的故事了 但是下午就要联调了 总不能给三方讲一千零一夜 没事这难不倒我 新建一个Net Framework控制台
``` c#
       static void Main(string[] args)
            {
                try
                {
                    var argsLength = args.Length;
                    if (argsLength > 1)
                    {
                        EncryptDecryptTool tool = new EncryptDecryptTool();
                        string result = string.Empty;
                        if (args[0] == "encrypt")
                        {
                            result = tool.encrypt(args[1]);
                        }
                        else
                        {
                            result = tool.decrypt(args[1]);
                        }
                        Console.WriteLine(result);
                    }
    
                }
                catch (Exception ex)
                {
                    Console.WriteLine(ex.Message);
                }
    
            }
```			

##### Net Core调用

``` c#       
          	    string path = @"D:\WebSite\AESTool.exe";
                var method="encrypt";
                var msg="this is msg";
                string fileName = path;
                Process p = new Process();
                p.StartInfo.UseShellExecute = false;
                p.StartInfo.RedirectStandardOutput = true;
                p.StartInfo.FileName = fileName;
                p.StartInfo.CreateNoWindow = true;
                p.StartInfo.Arguments = $"{method} {msg}";//参数以空格分隔
                p.Start();
                var output = await p.StandardOutput.ReadToEndAsync();
```				

##### 测试ok 没问题 就是时间有点久 加密解密每次差不多都要一秒

![1542005197418](https://img2018.cnblogs.com/blog/553504/201811/553504-20181112175039044-1947156885.png)

就这样跑了一段时间 今天闲下来 想起上次的一秒

![1542013331412](https://img2018.cnblogs.com/blog/553504/201811/553504-20181112175038234-243196817.png)

##### 嗯 写好辞职申请 备份 好的 我要开始重构了 嗯 知己知彼百战不殆 那我们先去维密上大概了解一下AES

![1542014447595](https://img2018.cnblogs.com/blog/553504/201811/553504-20181112175037146-2046522482.png)

好吧 我知道你也没看懂

![1542014542480](https://img2018.cnblogs.com/blog/553504/201811/553504-20181112175036612-2113328509.png)

##### 我们还是看代码吧 作为一个资深程序员的直觉

![1542014881987](https://img2018.cnblogs.com/blog/553504/201811/553504-20181112175036221-1199350786.png)

##### 之前查了一堆资料 都说java C# AES加解密通用版 key的位都必须是16位 害的笨菜鸟陷入了思维误区 我们再看上面那段代码 Debug进去看 发现是用MD5算法根据key生成长度16的byte数据 16啊 多么敏感的数字 就是哈希计算 靠

##### 果断google，Ctrl CV
``` c#
      public static class EncryptDecryptTool
        {
    
            private const string key = "qCOHfwhXgsZFBFSeZeGOlXtZbKOzApLBuZoWxrQjcmoxYHfrWZzdyFbvGuMcZqmC";
    
            /// <summary>
            /// MD5哈希计算
            /// </summary>
            /// <param name="key"></param>
            /// <returns></returns>
            public static byte[] ConvertStringToMD5(string key)
            {
    
                byte[] ByteData = Encoding.UTF8.GetBytes(key);
                MD5 oMd5 = MD5.Create();       
                byte[] HashData = oMd5.ComputeHash(ByteData);
                return HashData;
            }
    
    
            /// <summary>
            /// AES加密 
            /// </summary>
            /// <param name="toEncrypt"></param>
            /// <returns></returns>
            public static string Encrypt(string toEncrypt)
            {
                byte[] keyArray = ConvertStringToMD5(key);
                byte[] toEncryptArray = Encoding.UTF8.GetBytes(toEncrypt);
    
                RijndaelManaged rDel = new RijndaelManaged();
                rDel.Key = keyArray;
                rDel.Mode = CipherMode.ECB;
                rDel.Padding = PaddingMode.PKCS7;
    
                ICryptoTransform cTransform = rDel.CreateEncryptor();
                byte[] resultArray = cTransform.TransformFinalBlock(toEncryptArray, 0, toEncryptArray.Length);
    
                return Convert.ToBase64String(resultArray, 0, resultArray.Length);
            }
    
            /// <summary>
            /// AES解密
            /// </summary>
            /// <param name="toDecrypt"></param>
            /// <returns></returns>
            public static string Decrypt(string toDecrypt)
            {
                byte[] keyArray = ConvertStringToMD5(key);
                byte[] toEncryptArray = Convert.FromBase64String(toDecrypt);
    
                RijndaelManaged rDel = new RijndaelManaged();
                rDel.Key = keyArray;
                rDel.Mode = CipherMode.ECB;
                rDel.Padding = PaddingMode.PKCS7;
    
                ICryptoTransform cTransform = rDel.CreateDecryptor();
                byte[] resultArray = cTransform.TransformFinalBlock(toEncryptArray, 0, toEncryptArray.Length);
    
                return Encoding.UTF8.GetString(resultArray);
            }
```


### 参考资料
- [记一次Java AES 加解密 对应C# AES加解密 的一波三折](https://www.cnblogs.com/aishangyipiyema/p/9948011.html)