# Rotate SSH Keys between two host groups

SSH로 Remote 서버들을 관리할 때 느끼는거지만..
* yes/no 묻지 않게 known_host 자동 추가
* authorized_key에 허용 서버 키 추가

요거 꼭 하고 싶었다. 그래서 playbook으로 만들어봤다.

{% gist id="https://gist.github.com/kyungw00k/d56547b1e30b5d72316e7df0e9d99f92" %}{% endgist %}