
# zb_otc 聊天


## Supervisor 配置文件

```conf
# /etc/supervisor/conf.d/zb_otc.conf

[program:comet]
command=/opt/zb_otc/srv/im/bin/comet -conf /opt/zb_otc/srv/im/etc/comet-example.toml
#autorestart=true

[program:logic]
command=/opt/zb_otc/srv/im/bin/logic -conf /opt/zb_otc/srv/im/etc/logic-example.toml
#autorestart=true

[program:userauth]
command=/opt/zb_otc/srv/app/bin/userauth -conf /opt/zb_otc/srv/app/etc/userauth.toml
#autorestart=true

[program:record]
command=/opt/zb_otc/srv/app/bin/record -conf /opt/zb_otc/srv/app/etc/record.toml
#autorestart=true

[program:generator]
command=/opt/zb_otc/srv/app/bin/generator -conf /opt/zb_otc/srv/app/etc/generator.toml
#autorestart=true

[program:oss]
command=/opt/zb_otc/srv/app/bin/oss -conf /opt/zb_otc/srv/app/etc/oss.toml
#autorestart=true

[program:offline-push]
command=/opt/zb_otc/srv/app/bin/offline-push -conf /opt/zb_otc/srv/app/etc/offline_push.toml
#autorestart=true
```




## Nginx 配置文件

```nginx
# /etc/nginx/cinf.d/zb_otc.conf
upstream record {
        server 127.0.0.1:18003;
}

upstream oss {
        server 127.0.0.1:18005;
}

upstream comet {
        server 127.0.0.1:3102;
}

server {
        
        listen       8888;
        server_name  localhost;
        add_header Access-Control-Allow-Origin *;
        
	location /record/ {
                proxy_pass http://record/;
                proxy_redirect default;
                proxy_set_header Host $host;
                proxy_set_header X-real-ip $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_cookie_path / /;
        }

        location /oss/ {
                proxy_pass http://oss/;
                proxy_redirect default;
                proxy_set_header Host $host;
                proxy_set_header X-real-ip $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_cookie_path / /;
        }

        location /sub/ {
                proxy_pass http://comet/sub;
                proxy_set_header Host $host;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
                proxy_read_timeout 86400;
        }
}

```



## Mysql 数据库

数据库

 ![](https://docimg3.docs.qq.com/image/wPz2-pn9n56tgHhRdjUbHQ?w=344&h=109)

数据库表

```sql
CREATE TABLE `dtalk_msg_content` (
  `mid` bigint(20) unsigned NOT NULL COMMENT '\n\n消息id\n',
  `seq` varchar(40) NOT NULL COMMENT '消息序列号',
  `sender_id` varchar(40) NOT NULL COMMENT '发送者',
  `receiver_id` varchar(40) NOT NULL COMMENT '接收者',
  `msg_type` tinyint(3) unsigned NOT NULL COMMENT '消息类型',
  `content` longtext NOT NULL COMMENT '消息内容',
  `create_time` bigint(20) NOT NULL COMMENT '创建时间',
  PRIMARY KEY (`mid`) USING BTREE,
  KEY `idx_sender_id_seq` (`sender_id`,`seq`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC;

CREATE TABLE `dtalk_msg_relation` (
  `mid` bigint(20) unsigned NOT NULL COMMENT '消息id',
  `owner_uid` varchar(40) NOT NULL COMMENT '索引用户',
  `other_uid` varchar(40) NOT NULL COMMENT '\n\n另一方用户\n',
  `type` tinyint(3) unsigned NOT NULL COMMENT '0->发件箱；1->收件箱',
  `state` tinyint(3) unsigned NOT NULL DEFAULT '0' COMMENT '0->未接受；1->已接收',
  `create_time` bigint(20) NOT NULL COMMENT '\n\n创建时间\n',
  PRIMARY KEY (`mid`,`owner_uid`) USING BTREE,
  KEY `idx_owneruid_otheruid_msgid` (`owner_uid`,`other_uid`,`mid`) USING BTREE,
  KEY `idx_owneruid_type_state` (`owner_uid`,`type`,`state`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC;

CREATE TABLE `dtalk_msg_version` (
  `uid` varchar(40) NOT NULL COMMENT '\n\n用户id\n',
  `version` bigint(20) DEFAULT NULL COMMENT '版本号',
  PRIMARY KEY (`uid`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC;

CREATE TABLE `dtalk_oss_config` (
  `app` varchar(20) NOT NULL COMMENT '应用类型',
  `oss_type` varchar(20) NOT NULL COMMENT '存储服务类型',
  `endpoint` varchar(255) DEFAULT NULL COMMENT '服务节点',
  `access_key_id` varchar(255) DEFAULT NULL,
  `access_key_secret` varchar(255) DEFAULT NULL,
  `role` varchar(255) DEFAULT NULL,
  `policy` varchar(255) DEFAULT NULL COMMENT '角色权限控制',
  `duration_seconds` int(11) DEFAULT NULL COMMENT '最大会话时间',
  PRIMARY KEY (`app`,`oss_type`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC;
```





## 配置文件修改

### oss.toml

[MySQL]
Host = "127.0.0.1" (改)
Port = 3306
User = "root" (改)
Pwd = "123456" (改)
Db = "zb_otc"

### record.toml

[MySQL]
Host = "127.0.0.1" (改)
Port = 3306
User = "root" (改)
Pwd = "123456" (改)
Db = "zb_otc"



[OTCService]
OrderAddress = "http://172.16.100.214:8080/receive/trade-opponent" (改成找币 otc 线上订单关系接口的路径)

### userauth.toml

[OTCService]
AuthAddress="http://172.16.100.214:8080/backend/info/get-info"(改成找币 otc 线上鉴权接口的路径)





## 阿里云 OSS 配置

需要配置 OSS STS

[[授权访问 - 对象存储 OSS - 阿里云 (aliyun.com)](https://help.aliyun.com/document_detail/32077.html?spm=5176.11065259.1996646101.searchclickresult.2d4523c2uCF6Ti)](https://help.aliyun.com/document_detail/100624.html?spm=5176.10695662.1996646101.searchclickresult.75dd5554zuYeuj)

需要得到

**bucket** (告诉 h5 前端)

**endpoint** 

**AccessKey ID**

**AccessKey Secret**

**role**

类似

```
{
	"roleSessionName": "normal-app",
	"roleArn": "acs:ram::1264888835193631:role/normal-app"
}
```

然后插入数据库表dtalk_oss_config

app=dtalk

oss_type=aliyun

duration_seconds=3600

policy 不用填

![](https://docimg6.docs.qq.com/image/6n0zM9_VhgDfBAV-HZaFSg?w=1520&h=55)

## 启动命令

```shell
# 解压 zb_otc 并移动到/opt 目录下

# 启动 kafka, redis, etcd
$ cd /opt/zb_otc
$ docker-compose -f docker-compose.yml build
$ docker-compose -f docker-compose.yml up -d

# 设置好 supervisor 更新配置
$ sudo supervisorctl update
$ sudo supervisorctl restart comet logic userauth generator oss offpush-line record

# 设置好 nginx 后更新 nginx 配置
$ nginx -s reload
```









