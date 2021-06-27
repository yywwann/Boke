# Chat33pro 聊天部署文档



## Supervisor 配置文件

- supervisord.conf

```conf
; supervisor config file

[unix_http_server]
file=/var/run/supervisor.sock   ; (the path to the socket file)
chmod=0700                       ; sockef file mode (default 0700)

[supervisord]
logfile=/var/log/supervisor/supervisord.log ; (main log file;default $CWD/supervisord.log)
pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
childlogdir=/var/log/supervisor            ; ('AUTO' child log dir, default $TEMP)

minfds=65535                 ; (min. avail startup file descriptors;default 1024)
minprocs=65535                ; (min. avail process descriptors;default 200)

; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///var/run/supervisor.sock ; use a unix:// URL  for a unix socket

; The [include] section can just contain the "files" setting.  This
; setting can list multiple files (separated by whitespace or
; newlines).  It can also contain wildcards.  The filenames are
; interpreted as relative to this file.  Included files *cannot*
; include files themselves.

[include]
files = /etc/supervisor/conf.d/*.conf

```

- im.conf

```conf
[program:comet]
command=/opt/dtalk/srv/im/bin/comet -conf /opt/dtalk/srv/im/etc/comet-example.toml
#autorestart=true

[program:logic]
command=/opt/dtalk/srv/im/bin/logic -conf /opt/dtalk/srv/im/etc/logic-example.toml
#autorestart=true
```

- chain33.conf

```conf
[program:chain33]
command=/opt/chain33/chain33
#autorestart=true
```

- dtalk.conf

```conf
[program:disc]
command=/opt/dtalk/srv/app/bin/discovery -conf /opt/dtalk/srv/app/etc/discovery.toml
#autorestart=true

[program:user]
command=/opt/dtalk/srv/app/bin/user -conf /opt/dtalk/srv/app/etc/user.toml
#autorestart=true

[program:record]
command=/opt/dtalk/srv/app/bin/record -conf /opt/dtalk/srv/app/etc/record.toml
#autorestart=true

[program:generator]
command=/opt/dtalk/srv/app/bin/generator -conf /opt/dtalk/srv/app/etc/generator.toml
#autorestart=true

[program:backup]
command=/opt/dtalk/srv/app/bin/backup -conf /opt/dtalk/srv/app/etc/backup.toml
#autorestart=true

[program:backend]
command=/opt/dtalk/srv/app/bin/backend -conf /opt/dtalk/srv/app/etc/backend.toml
#autorestart=true

[program:oss]
command=/opt/dtalk/srv/app/bin/oss -conf /opt/dtalk/srv/app/etc/oss.toml
#autorestart=true

[program:offline-push]
command=/opt/dtalk/srv/app/bin/offline-push -conf /opt/dtalk/srv/app/etc/offline_push.toml
#autorestart=true

[program:group]
command=/opt/dtalk/srv/app/bin/group -conf /opt/dtalk/srv/app/etc/group.toml
#autorestart=true
```



## nginx配置

- dtalk.conf

```conf
upstream disc {
        server 127.0.0.1:18001;
}

upstream user {
        server 127.0.0.1:18002;
}

upstream backup {
        server 127.0.0.1:18004;
}

upstream record {
        server 127.0.0.1:18003;
}

upstream oss {
        server 127.0.0.1:18005;
}

upstream comet {
        server 127.0.0.1:3102;
}

upstream backend {
        server 127.0.0.1:18102;
}

upstream group {
        server 127.0.0.1:18011;
}

server {

	listen       8888;
        server_name  localhost;

        add_header Access-Control-Allow-Origin *;
	add_header Access-Control-Allow-Headers *;
	add_header Access-Control-Allow-Methods *;

	add_header Access-Control-Allow-Credentials 'true';

	#add_header Access-Control-Allow-Methods 'POST, GET, OPTIONS, PUT, PATCH, DELETE';
	#add_header Access-Control-Expose-Headers 'Content-Length, Access-Control-Allow-Origin, Access-Control-Allow-Headers, Content-Type';

	if ($request_method = 'OPTIONS'){
		return 204;
	}

	location /disc/ {
		proxy_pass http://disc/;
		proxy_redirect default;
		proxy_set_header Host $host;
		proxy_set_header X-real-ip $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_cookie_path / /;
	}

	location /user/ {
                proxy_pass http://user/user/;
                proxy_redirect default;
                proxy_set_header Host $host;
                proxy_set_header X-real-ip $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_cookie_path / /;
        }

	location /app/version/ {
                proxy_pass http://backend/app/version/;
                proxy_redirect default;
                proxy_set_header Host $host;
                proxy_set_header X-real-ip $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_cookie_path / /;
        }

	location /backend/ {
                proxy_pass http://backend/backend/;
                proxy_redirect default;
                proxy_set_header Host $host;
                proxy_set_header X-real-ip $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_cookie_path / /;
        }

	location /backup/ {
                proxy_pass http://backup/;
                proxy_redirect default;
                proxy_set_header Host $host;
                proxy_set_header X-real-ip $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_cookie_path / /;
        }

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

    location /group/ {
                proxy_pass http://group/;
                proxy_redirect default;
                proxy_set_header Host $host;
                proxy_set_header X-real-ip $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_cookie_path / /;
        }
}
```



## docker-compose

```yaml
version: '3'

services:
    dtalk_redis:
        image: redis
        container_name: dtalk_redis
        command: redis-server
        ports:
            - "6379:6379"

    dtalk_zookeeper:
        image: wurstmeister/zookeeper
        hostname: dtalk_zookeeper
        container_name: dtalk_zookeeper
        ports:
            - "2181:2181"
    
    dtalk_kafka:
        image: wurstmeister/kafka
        hostname: dtalk_kafka
        container_name: dtalk_kafka
        ports:
            - "9092:9092"
        environment:
            - KAFKA_BROKER_ID=1
            - KAFKA_ZOOKEEPER_CONNECT=dtalk_zookeeper:2181
            - KAFKA_ADVERTISED_HOST_NAME=127.0.0.1
            - KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://127.0.0.1:9092
            - KAFKA_ADVERTISED_PORT=9092
            - KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1
        depends_on:
            - dtalk_zookeeper

    dtalk_etcd:
        image: quay.io/coreos/etcd
        container_name: dtalk_etcd
        environment:
            - ETCDCTL_API=3
        volumes:
            - ./data/etcd/etcd-data:/etcd-data
        command: etcd -name etcd1 -advertise-client-urls http://0.0.0.0:2379 -listen-client-urls http://0.0.0.0:2379 -listen-peer-urls http://0.0.0.0:2380 -initial-cluster-token tkn -initial-cluster-state new
        ports:
            - "2379:2379"
            - "2380:2380"
```



## MySQL 数据库

- 数据库

> 数据库名: dtalk
>
> 字符集:utf8mb4
>
> 排序规则:utf8mb4_general_ci

- 数据库表

```sql
CREATE TABLE `dtalk_addr_backup` (
  `address` varchar(255) NOT NULL COMMENT '用户地址',
  `area` varchar(4) DEFAULT NULL COMMENT '区号',
  `phone` varchar(11) DEFAULT NULL COMMENT '手机号',
  `email` varchar(30) DEFAULT NULL COMMENT '邮箱',
  `mnemonic` varchar(1020) DEFAULT NULL COMMENT '助记词',
  `private_key` varchar(1020) DEFAULT NULL COMMENT '加密私钥',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`address`),
  KEY `idx_phone` (`phone`) USING HASH,
  KEY `idx_email` (`email`) USING HASH
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `dtalk_group_info` (
  `group_id` bigint(20) NOT NULL COMMENT '群id',
  `group_mark_id` varchar(40) NOT NULL COMMENT '群编号',
  `group_name` varchar(40) NOT NULL COMMENT '群名称',
  `group_avatar` varchar(1000) NOT NULL DEFAULT '' COMMENT '群头像 url',
  `group_member_num` int(11) NOT NULL COMMENT '群成员人数',
  `group_maximum` int(11) NOT NULL DEFAULT '200' COMMENT '群成员人数上限， 默认 200 人',
  `group_introduce` longtext NOT NULL COMMENT '群简介',
  `group_status` tinyint(4) NOT NULL COMMENT '群状态，0=正常 1=封禁 2=解散',
  `group_owner_id` varchar(40) NOT NULL COMMENT '群主 id',
  `group_create_time` bigint(20) NOT NULL COMMENT '创建时间',
  `group_update_time` bigint(20) NOT NULL COMMENT '更新时间',
  `group_join_type` tinyint(4) NOT NULL COMMENT '加群方式，0=无需审批（默认），1=禁止加群，群主和管理员邀请加群',
  `group_mute_type` tinyint(4) NOT NULL COMMENT '禁言， 0=全员可发言， 1=全员禁言(除群主和管理员)',
  `group_friend_type` tinyint(4) NOT NULL COMMENT '加好友限制， 0=群内可加好友，1=群内禁止加好友',
  PRIMARY KEY (`group_id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `dtalk_group_member` (
  `group_id` bigint(20) NOT NULL COMMENT '群 id',
  `group_member_id` varchar(40) NOT NULL COMMENT '用户 id',
  `group_member_name` varchar(40) NOT NULL COMMENT '用户群昵称',
  `group_member_type` tinyint(4) NOT NULL COMMENT '用户角色，2=群主，1=管理员，0=群员，3=退群',
  `group_member_join_time` bigint(20) NOT NULL COMMENT '用户加群时间',
  `group_member_update_time` bigint(20) NOT NULL COMMENT '用户更新时间',
  PRIMARY KEY (`group_id`,`group_member_id`),
  KEY `idx_userid_type` (`group_member_id`,`group_member_type`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `dtalk_group_member_mute` (
  `group_id` bigint(20) NOT NULL COMMENT '群 id',
  `group_member_id` varchar(40) NOT NULL COMMENT '用户 id',
  `group_member_mute_time` bigint(20) NOT NULL COMMENT '用户禁言结束时间',
  `group_member_mute_update_time` bigint(20) NOT NULL COMMENT '用户上一次被禁言的时间',
  PRIMARY KEY (`group_id`,`group_member_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `dtalk_oss_config` (
  `app` varchar(20) NOT NULL COMMENT '应用类型',
  `oss_type` varchar(20) NOT NULL COMMENT '存储服务类型',
  `endpoint` varchar(255) DEFAULT NULL COMMENT '服务节点',
  `access_key_id` varchar(255) DEFAULT NULL,
  `access_key_secret` varchar(255) DEFAULT NULL,
  `role` varchar(255) DEFAULT NULL,
  `policy` varchar(255) DEFAULT NULL COMMENT '角色权限控制',
  `duration_seconds` int(11) DEFAULT NULL COMMENT '最大会话时间',
  PRIMARY KEY (`app`,`oss_type`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `dtalk_group_msg_content` (
  `mid` bigint(20) unsigned NOT NULL COMMENT '\n\n消息id\n',
  `seq` varchar(40) NOT NULL COMMENT '消息序列号',
  `sender_id` varchar(40) NOT NULL COMMENT '发送者',
  `receiver_id` varchar(40) NOT NULL COMMENT '接收者',
  `msg_type` tinyint(3) unsigned NOT NULL COMMENT '消息类型',
  `content` longtext NOT NULL COMMENT '消息内容',
  `create_time` bigint(20) NOT NULL COMMENT '创建时间',
  `source` varchar(1024) DEFAULT NULL COMMENT '转发来源',
  PRIMARY KEY (`mid`) USING BTREE,
  KEY `idx_sender_id_seq` (`sender_id`,`seq`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC;

CREATE TABLE `dtalk_group_msg_relation` (
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

CREATE TABLE `dtalk_msg_content` (
  `mid` bigint(20) unsigned NOT NULL COMMENT '\n\n消息id\n',
  `seq` varchar(40) NOT NULL COMMENT '消息序列号',
  `sender_id` varchar(40) NOT NULL COMMENT '发送者',
  `receiver_id` varchar(40) NOT NULL COMMENT '接收者',
  `msg_type` tinyint(3) unsigned NOT NULL COMMENT '消息类型',
  `content` longtext NOT NULL COMMENT '消息内容',
  `create_time` bigint(20) NOT NULL COMMENT '创建时间',
  `source` varchar(1024) DEFAULT NULL COMMENT '转发来源',
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

CREATE TABLE `dtalk_notice_content` (
  `id` bigint(20) NOT NULL COMMENT '消息id',
  `uid` varchar(40) NOT NULL COMMENT '接收者',
  `type` tinyint(3) DEFAULT NULL COMMENT '通知类型',
  `state` tinyint(3) DEFAULT NULL COMMENT '0->未接收；1->已接收',
  `content` varchar(255) DEFAULT NULL COMMENT '通知内容',
  `create_time` bigint(20) DEFAULT NULL COMMENT '创建时间',
  `update_time` bigint(20) DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`,`uid`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

CREATE TABLE `dtalk_ver_backend` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '版本编号',
  `platform` varchar(40) NOT NULL COMMENT '平台',
  `state` tinyint(4) NOT NULL COMMENT '线上状态',
  `device_type` varchar(40) NOT NULL COMMENT '终端',
  `version_code` bigint(20) NOT NULL COMMENT '版本号',
  `version_name` varchar(40) NOT NULL COMMENT '版本名字',
  `download_url` varchar(2083) NOT NULL COMMENT '下载地址',
  `size` bigint(20) NOT NULL COMMENT '包大小',
  `md5` varchar(40) NOT NULL COMMENT 'MD5',
  `force_update` tinyint(4) NOT NULL COMMENT '强制更新',
  `description` text NOT NULL COMMENT '描述信息',
  `ope_user` varchar(40) DEFAULT NULL COMMENT '操作者',
  `update_time` bigint(20) NOT NULL COMMENT '更新时间',
  `create_time` bigint(20) NOT NULL COMMENT '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=13 DEFAULT CHARSET=utf8mb4;
```



## 配置文件修改

- backend.toml

```toml
[MySQL]
Host = "127.0.0.1"()
Port = 3306
User = "root"()
Pwd = "123456"()
Db = "dtalk"
```

- backup.toml

```toml
[MySQL]
Host= "127.0.0.1"()
Port= 3306
User= "root"()
Pwd=  "123456"()
Db=   "dtalk"
```

- discovery.toml(待确认)
- backup.toml(待确认)
- group.toml

```toml
[MySQL]
Host= "127.0.0.1"()
Port= 3306
User= "root"()
Pwd=  "123456"()
Db=   "dtalk"
```

- oss.toml

```toml
[MySQL]
Host= "127.0.0.1"()
Port= 3306
User= "root"()
Pwd=  "123456"()
Db=   "dtalk"
```

- store.toml

```toml
[MySQL]
Host= "127.0.0.1"()
Port= 3306
User= "root"()
Pwd=  "123456"()
Db=   "dtalk"
```





## 华为云 OBS 配置

[华为云 OBS 临时权限配置 - 即时通讯系统 - Confluence (33.cn)](https://confluence.33.cn/pages/viewpage.action?pageId=25891791)

**bucket** (桶名), **endpoint** (终端链接)需要告诉客户端

```sql
INSERT INTO dtalk_oss_config ( app, oss_type, endpoint, access_key_id, access_key_secret, role, policy, duration_seconds)
                        VALUES (
                        'dtalk',
                        'huaweiyun',
                        '+https://obs.cn-east-3.myhuaweicloud.com',   (各地域接入地址)
                        '+VS1SNCU6SM7NSRLFT0HF',
                        '+pRI6OjPTrc0atS3Do4PFgSOyx7IOnaXXbDf5ZBd2',
                        '',
                        '{
	"Version": "1.1",
	"Statement": [
		{
			"Action": [
				"obs:object:*"
			],
			"Effect": "Allow"
		}
	]
}',
                        3600 )
```



## 启动命令

```shell
# 解压 dtalk 并移动到/opt 目录下

# 启动 zookeeper, kafka, redis, etcd
$ cd /opt/dtalk
$ docker-compose -f docker-compose.yml up -d

# 设置好 supervisor 更新配置
$ sudo supervisorctl update
$ sudo supervisorctl restart backend disc backup generator oss user offline-push
$ sudo supervisorctl restart comet logic
$ sudo supervisorctl restart answer pusher group store

# 设置好 nginx 后更新 nginx 配置
$ nginx -s reload
```

