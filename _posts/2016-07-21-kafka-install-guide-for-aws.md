---
layout: post
fbcomments: false
published: true
title: Kafka Install Guide for AWS
category: linux
tags:
  - Kafka
  - aws
---
# Kafka Guide

## Instance Type
- ec2 : m4.xlarge
- ebs : gp2 (20GB) , st1 (2TB)
- link1 : https://aws.amazon.com/ko/blogs/korea/amazon-ebs-update-new-cold-storage-and-throughput-options/
- link2 : https://aws.amazon.com/ko/ebs/details/
- link3 : https://aws.amazon.com/ko/ebs/details/#ebsoptimized

## Before Installing
```
sudo yum update
sudo vim /etc/security/limits.conf
/////////////////////////////
root soft nofile 65536
root hard nofile 65536
* soft nofile 65536
* hard nofile 65536
/////////////////////////////

sudo reboot
```

#### ebs 볼륨 마운트
- http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/ebs-using-volumes.html

```
[ec2-user@ip-172-31-4-59 ~]$ lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0  20G  0 disk
└─xvda1 202:1    0  20G  0 part /
xvdb    202:16   0   2T  0 disk
[ec2-user@ip-172-31-4-59 ~]$ sudo file -s /dev/xvdb
/dev/xvdb: data
[ec2-user@ip-172-31-4-59 ~]$ sudo mkfs -t ext4 /dev/xvdb
mke2fs 1.42.12 (29-Aug-2014)
Creating filesystem with 524288000 4k blocks and 131072000 inodes
Filesystem UUID: eeafe700-f067-4934-9469-bb03b1516386
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
	4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
	102400000, 214990848, 512000000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done

[ec2-user@ip-172-31-4-59 ~]$ sudo mkdir /data
[ec2-user@ip-172-31-4-59 ~]$ sudo mount /dev/xvdb /data

[ec2-user@ip-172-31-4-59 ~]$ sudo vi /etc/fstab
/dev/xvdb       /data   ext4    defaults,nofail        0       2
[ec2-user@ip-172-31-4-59 ~]$ sudo mount -a

```

## Install
```
sudo chown -R ec2-user.ec2-user /data

mkdir ~/app
```

### zookeeper Install
```
cd ~/app
wget http://apache.mirror.cdnetworks.com/zookeeper/stable/zookeeper-3.4.8.tar.gz
tar xvfz zookeeper-3.4.8.tar.gz
cd zookeeper-3.4.8/conf
cat <<EOF > zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data/zookeeper
clientPort=2181
EOF

~/app/zookeeper-3.4.8/bin/zkServer.sh start
```

### Kafka Install
```
cd ~/app
wget http://apache.tt.co.kr/kafka/0.8.2.2/kafka_2.10-0.8.2.2.tgz
tar xvfz kafka_2.10-0.8.2.2.tgz

sudo vim /etc/profile
/////////////////////////////
export KAFKA_HOME=/home/ec2-user/app/kafka_2.10-0.8.2.2
export PATH=$PATH:$KAFKA_HOME/bin
/////////////////////////////

source /etc/profile

vim $KAFKA_HOME/config/server.properties
/////////////////////////////
broker.id=0
host.name=kafka1
log.dirs=/data/kafka-logs
zookeeper.connect=zookeeper1:2181
/////////////////////////////

sudo vim /etc/hosts
/////////////////////////////
172.31.4.59 zookeeper1 ip-172-31-4-59
172.31.4.59 kafka1 ip-172-31-4-59
/////////////////////////////


nohup $KAFKA_HOME/bin/kafka-server-start.sh $KAFKA_HOME/config/server.properties &

cd $KAFKA_HOME
bin/kafka-topics.sh --list --zookeeper zookeeper1:2181
bin/kafka-topics.sh --zookeeper zookeeper1:2181 --create --replication-factor 1 --partitions 1 --topic topic_bid_us1
```

#### Fluent::Plugin::Kafka Install
```
https://github.com/htgc/fluent-plugin-kafka
sudo yum install patch
sudo yum groupinstall "Development Tools"

td-agent-gem install fluent-plugin-kafka
/////////////////////////////
<match>
  <store>
    @type               kafka_buffered

    brokers             kafka1:9092
    zookeeper           zookeeper1:2181

    default_topic       topic_bid_us1
    client_id           td-agent
    required_acks       0
    flush_interval      1s

    output_data_type    json
    output_include_tag  true
    output_include_time true
    time_format %Y-%m-%dT%H:%M:%S
    buffer_type         file
    buffer_path /data/fluent/kafka_buffered/topic_bid_us
  </store>
</match>
/////////////////////////////
```
