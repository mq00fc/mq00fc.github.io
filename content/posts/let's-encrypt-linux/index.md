+++
title = "Linux下使用Let's Encrypt配置Nginx SSL证书"
tags = ["Linux","SSL","编程"]
date = "2023-04-24"
update = "2023-04-24"
enableGitalk = false
draft = false
+++


## 前言
对于自己的某些网站或者是某些要求不高的网站，Let's Encrypt是一个肥肠良心的SSL证书颁发网站

Let's Encrypt支持多域名证书和通配符证书，下面让我们开始

## 使用acme.sh
之前尝试使用过Certbot但是效果不是很理想，主要是我的域名配置在阿里云，找了半天没有找到配置阿里云dns的插件.. [acme.sh项目地址](https://github.com/acmesh-official/acme.sh)

### 开始安装
``` shell
curl https://get.acme.sh | sh -s email=my@dreamogi.com
```

### 颁发证书
``` shell
acme.sh  --issue -d dreamogi.com  -d '*.dreamogi.com'  --dns dns_ali
```

{{< notice tip >}}
由于我这里使用的是**阿里云** 来配置域名解析修改，其他的域名提供商请查看此处[dnsapi](https://github.com/acmesh-official/acme.sh/wiki/dnsapi#11-use-aliyun-domain-api-to-automatically-issue-cert)
{{< /notice >}}

#### 简述阿里云自动配置dns
* 首先去阿里云网站申请AccessKey（请注意保管！）
{{< image 1.png >}}

* 然后修改acme的配置项
``` shell
cd /root/.acme.sh
```

* 修改account.conf
``` txt
export Ali_Key="***"
export Ali_Secret="***"
```


### 查看证书信息
``` shell
acme.sh --info -d dreamogi.com
```

### 自动续期
由于证书的有效期为3个月,所以需要进行自动续期，具体内容参见[How to renew the certs](https://github.com/acmesh-official/acme.sh#12-how-to-renew-the-certs)
```shell
acme.sh --renew -d dreamogi.com --force
```


### 生成证书后重载Nginx

```shell
acme.sh --install-cert -d dreamogi.com \
--key-file       /path/to/keyfile/in/nginx/key.pem  \
--fullchain-file /path/to/fullchain/nginx/cert.pem \
--reloadcmd     "service nginx force-reload"
```

如此便完成了在Nginx下ssl证书的配置
{{< image 2.png >}}
