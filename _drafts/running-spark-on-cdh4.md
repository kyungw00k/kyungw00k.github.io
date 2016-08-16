---
layout: post
title: Running Spark on CDH4
date: 2015-10-21 14:51:52.000000000 +09:00
type: post
categories:
- Spark
- CDH
---
## Purpose
기존에 쓰고 있는 CDH4 구성에서 Spark를 쓰고자 하는 경우

## Installation

### Common
* `spark-1.5.1-bin-cdh4`를 내려받아 Master/Slave에 모두 설치한다.

## Configuration
먼저 Master에서 Slave로 페스워드 없이 SSH 접속하게 설정한다.

### Master
1. `$SPARK_HOME/conf/slave` 에 slave 노드의 IP(또는 hostname)을 적어준다.
2. `$SPARK_HOME/conf/spark-env.sh` 에 `HADOOP_HOME`을 지정한다.

### Slave
없음

## Run
1. `Master` 노드에서 `$SPARK_HOME/sbin/start-master.sh` 실행
2. `Master` 노드에서 `$SPARK_HOME/sbin/start-slaves.sh` 실행
3. `http://<master_node_ip>:8080` 으로 상태 확인


## References
* [Install Spark/Shark on CDH 4 | @sskaje](https://sskaje.me/2014/02/install-spark-shark-cdh-4/)
