# Role 만들때는 ansible-galaxy를 사용하자

## 1. 뼈대 만들기
예컨데, `ansible-airflow` Role을 만든다고 할 때, 다음과 같이 실행한다.

```sh
$ ansible-galaxy ansible-airflow
- ansible-airflow was created successfully
$ # done
```

그러면 다음과 같은 파일들이 생성된다.

```
ansible-airflow
├── README.md
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml
```

## 2. 진짜로 만들기
적당히 살릴껀 살리고 지울껀 지우면 되겠다.

## 3. 공유하기
Ansible Galaxy 사이트에 Github 계정을 통해서 로그인 하면 GitHub Repo에서 Ansible Project를 찾는다. 해당 프로젝트를 활성화 하려면 Travis 설정만 점검하면 되겠다.

(다른 계정으로는 안해봐서 잘 몰라요)