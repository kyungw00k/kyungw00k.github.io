---
layout: post
title: Windows 10 Vagrant Box 만들기
date: 2015-09-09 13:40:53.000000000 +09:00
type: post
published: true
status: publish
categories:
- Tutorial
tags:
- vagrant
- vitualbox
- windows10
meta: {}
---
이 글은 [Creating a Windows 10 Base Box for Vagrant with VirtualBox](http://huestones.co.uk/node/305) 가이드를 기반으로 하고, 중간에 안되는 부분은 다른 글을 참고로 작업했다.

## 나 그냥 쓰면 안되니?
그냥 [최종본](https://atlas.hashicorp.com/kyungw00k/boxes/windows-10-pro-kn-x64) 쓰고 싶은 분들은... 아래 처럼 하시면 되겠다.

```sh
$ vagrant init kyungw00k/windows-10-pro-kn-x64; vagrant up --provider virtualbox
$ vagrant rdp
```

만들어보고 싶으신 분은 계속 읽어나가면 되겠다.

## 뭐가 필요한가?
* [VirtualBox 5](https://www.virtualbox.org/wiki/Downloads), [Extension Pack](https://www.virtualbox.org/wiki/Downloads)
* [Vagrant](https://docs.vagrantup.com/v2/getting-started/index.html) 1.7.4
* [Windows 10 ISO](https://www.microsoft.com/en-us/software-download/windows10ISO)
* Microsoft Remote Desktop

작업하기 전에 설치할 건 설치하고, 다운 받을건 다운 받아놓자.

## 가상 머신 만들기
이름은 (영어로) 아무렇게 짓되, 몇 가지 수정할 필요가 있다.

* 불필요한 하드웨어는 Disable 시키자. 예를 들면 사운드 카드!(필요하면 그냥 냅둬도 된다.)
* 메모리는 1024MB로!(늘리고 싶으면 Vagrantfile에서 변경 할 수 있을테니)

## 윈도우를 설치하기
Vagrant Box니 ID/PASSWORD를 모두 `vagrant`로 설정 해 주면 되겠다.

## 추가 설정


### VirtualBox Guest Addition 설치하기
![스크린샷 2015-09-09 오후 1.19.23](/images/2015/09/09/ec8aa4ed81aceba6b0ec83b7-2015-09-09-ec98a4ed9b84-1-19-23.png)

설치하고 리부팅하자.

###사용자 계정 컨트롤 설정을 "알리지 않음"으로 변경
![스크린샷 2015-09-09 오후 1.17.04](/images/2015/09/09/ec8aa4ed81aceba6b0ec83b7-2015-09-09-ec98a4ed9b84-1-17-04.png)

이후에 레지스트리 작업을 하라는데(`EnableLUA=0`)이 있는데, 바꾸면 Edge 등 몇몇 앱들이 동작하지 않게 된다. 필요하면 하면 됨.

### WinRM 서비스 활성화
관리자 모드로 명령프롬프트를 띄운 다음 아래 항목을 한 행씩 실행하자.

```
winrm quickconfig -q
winrm set winrm/config/winrs @{MaxMemoryPerShellMB="512"}
winrm set winrm/config @{MaxTimeoutms="1800000"}
winrm set winrm/config/service @{AllowUnencrypted="true"}
winrm set winrm/config/service/auth @{Basic="true"}
sc config WinRM start= auto
```

방화벽 상태가 게스트 또는 공용 네트워크로 연결된 경우, 첫 행에서 방화벽 관련 이상한 에러가 뜬다.

이 때는 PowerShell을 관리자 모드로 실행한 후, 아래 코드를 넣어보자.

```
$nlm = [Activator]::CreateInstance([Type]::GetTypeFromCLSID([Guid]"{DCB00C01-570F-4A9B-8D69-199FDBA5723B}"))
$connections = $nlm.getnetworkconnections()
$connections |foreach {
    if ($_.getnetwork().getcategory() -eq 0) {
        $_.getnetwork().setcategory(1)
    }
}
```

위 스크립트를 실행하면 방화벽 상태가 개인 네트워크로 바뀌게 된다.

![스크린샷 2015-09-09 오후 1.12.52](/images/2015/09/09/ec8aa4ed81aceba6b0ec83b7-2015-09-09-ec98a4ed9b84-1-12-52.png)

### PowerShell Execution Policy 변경

![스크린샷 2015-09-09 오후 1.08.32](/images/2015/09/09/ec8aa4ed81aceba6b0ec83b7-2015-09-09-ec98a4ed9b84-1-08-32.png)

관리자 모드로 PowerShell을 실행시킨 다음 아래 항목을 실행하자.

```
Set-ExecutionPolicy -ExecutionPolicy Unrestricted
```

A를 눌러 일괄 승인한다.

### 원격 데스트톱 연결 허용
연결을 허용하면 `vagrant rdp` 로 접속할 수 있게 된다.

![스크린샷 2015-09-09 오후 1.06.48](/images/2015/09/09/ec8aa4ed81aceba6b0ec83b7-2015-09-09-ec98a4ed9b84-1-06-48.png)

### (optional) Tweak!
* 시각 효과를 변경한다든지
* 가상 메모리 설정을 변경한다든지

기타 등등 해주심 될듯

## 자, 이제 Box로 Exporting 해보자

디렉토리 하나 만들어 `Vagrantfile`을 하나 만들자.

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
config.vm.guest = :windows
config.vm.communicator = "winrm"
config.vm.boot_timeout = 600
config.vm.graceful_halt_timeout = 600

# Create a forwarded port mapping which allows access to a specific port
# within the machine from a port on the host machine. In the example below,
# accessing "localhost:8080" will access port 80 on the guest machine.
# config.vm.network "forwarded_port", guest: 80, host: 8080
config.vm.network :forwarded_port, guest: 3389, host: 3389
config.vm.network :forwarded_port, guest: 5985, host: 5985, id: "winrm", auto_correct: true

# config.vm.provider "virtualbox" do |vb|
# # Customize the name of VM in VirtualBox manager UI:
# vb.name = "yourcompany-yourbox"
# end
end
```

그리고 아래 명령을 수행해 Box를 만든다.

```sh
$ vagrant package --base 가상머신이름 --output `pwd`/windows.box --vagrantfile `pwd`/Vagrantfile
```

## 사용해보기
앞에서 만들어진 windows.box를 추가해서 사용해보자. 이미지 이름은 `windows-10-pro-kn-korean-x64` 라고 가정한다.

```sh
$ vagrant box add /path/to/output/windows.box --name windows-10-pro-kn-korean-x64
```

이상 없이 박스가 추가 되었다면, 이제 정말 띄워보자.

```sh
$ vagrant init windows-10-pro-kn-korean-x64
$ vagrant up
```

별다른 에러 문구가 없다면, 이제 원격 데스크톱으로 접속해보자.

```sh
$ vagrant rdp
```

패스워드가 틀리다고 나오는데 이때, vagrant 입력해주면 접속 된다.
(패스워드를 plaintext로 줘서 그런거 같은데... 해싱하면 해결된다는듯. 아몰랑~)

## 배포하기
만든 [Vagrant Box를 배포할 수 있는 곳](https://atlas.hashicorp.com/)이 있더라.
여기에 계정 파고 지금까지 만든 box를 올려놨다.
(링크 : https://atlas.hashicorp.com/kyungw00k/boxes/windows-10-pro-kn-x64)

## 마지막으로...
실제 로컬에서 box를 만드는 시간도 그렇고, Provider를 여러개 추가해야 하는 이슈가 있는데, 이런걸 한방에 해결하기 위해서는 [Packer를 활용하는게 좋을 듯](https://atlas.hashicorp.com/tutorial/packer-vagrant) 하다.
