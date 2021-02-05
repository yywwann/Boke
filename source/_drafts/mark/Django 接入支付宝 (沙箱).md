# Django 接入支付宝 (沙箱)


##  支付宝公钥, 应用公钥, 应用私钥, APPID
- 注册[支付宝开放平台](https://openhome.alipay.com/platform/developerIndex.htm)
- 创建沙盒
- 下载[密钥生成工具](https://ideservice.alipay.com/ide/getPluginUrl.htm?clientType=assistant&platform=win&channelType=WEB)
### 通过软件生成`应用公钥`后, 加入沙箱应用的信息配置中,保存后便会生成`支付宝公钥`
- 软件界面
  ![](https://img-blog.csdnimg.cn/20200109133403197.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2ODc0NDgw,size_16,color_FFFFFF,t_70) 

- 沙盒界面![](https://docimg4.docs.qq.com/image/3OpkNoEB7wuoSKDJGj2GUw?w=869&h=320)
- 密钥![](https://docimg5.docs.qq.com/image/yZ_gKvGdKewa0Mw_-k-ORw?w=873&h=569)![](https://docimg10.docs.qq.com/image/3hgC-VYv2G5ylaNzWVwEBw?w=886&h=622)

#### 整理密钥
打开密钥文件夹
![](https://img-blog.csdnimg.cn/20200109133454134.png)
改名字
应用公钥2048.txt >>>>> app_public.txt
应用私钥2048.txt >>>>> app_private.txt
新建alipay_public.txt, 放入上面生成的`支付宝公钥`
> 坑点,1. 生成的三个文件中要以下面的格式修改
```
alipay_public.txt
					-----BEGIN PUBLIC KEY----- 
					密钥
					-----END PUBLIC KEY-----

app_public.txt
					-----BEGIN PUBLIC KEY----- 
					密钥
					-----END PUBLIC KEY-----
app_private.txt
					-----BEGIN RSA PRIVATE KEY-----
					密钥
					-----END RSA PRIVATE KEY-----
```



## 环境配置



