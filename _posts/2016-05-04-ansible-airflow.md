---
layout: post
title: ansible-airflow로 Airflow 설치하기
date: 2016-05-04 16:25:38.000000000 +09:00
type: post
categories:
- Ansible
- Airflow
---

ansible로 airflow 환경을 구성하고 싶어서 [`role`로 한 번 만들어 봤다](https://github.com/kyungw00k/ansible-airflow).

가장 힘들었던건
python2.7 환경을 맞추는건데 이를 위해 [`ansible-python27`](https://github.com/kyungw00k/ansible-python27) 이라는 또 하나의 `role`인 만들게 된다.

## Install
```sh
$ ansible-galaxy install kyungw00k.airflow
```

## Usage
기본(?) 환경을 그대로 사용하려면 다음과 같이 하면 되겠다.
```yaml
- hosts: servers
  roles:
     - kyungw00k.airflow
```

하지만 실제로 Airflow를 기본 설정 그대로 사용하진 않더라.

```yaml
- name : Airflow 설치
  hosts: airflow-manager
  remote_user : "{{ user }}"
  roles:
    - role: repo-mysql-community
    - role: kyungw00k.airflow
      package_extras:
        - libffi
        - libffi-devel
        - postgresql-devel
        - mysql-community-devel
        - cyrus-sasl
        - cyrus-sasl-devel
      airflow_extra_packages:
        - pyhs2
        - MySQL-python
        - cryptography
        - future
        - numpy
        - thrift
        - thrift_sasl
        - impyla
        - redis
        - unicodecsv
        - airflow[celery]
        - airflow[rabbitmq]
        - airflow[crypto]
        - airflow[jdbc]
        - airflow[hdfs]
        - airflow[hive]
        - airflow[mysql]
        - airflow[postgres]
```

위 설정은...
* Job 관리 Celery(Broker로 RabbitMQ)로
* MySQL를 공통(Airflow 및 Celery의) DB로
* Operator에서 Hive/JDBC/MySQL/Postgres 를 쓰고 싶어서

구성했습니다.


## TODO
막상 만들고 보니, 더 추가로 해줘야하는 것들이 생기더군요.

크게 생각 나는게..
* webserver
* worker
* scheduler

위 모드로 구성하고, 데몬을 재시작 할 수 있게 추가로 role을 만들어야 쓸만할 것 같다는 생각이 듭니다.

현재 버전으로 쓰다가 데몬 관리가 절실해지면 그때 위 Role를 Dependency 걸어서 작업할 것 같네요(아직은 절실하지 않다?).
