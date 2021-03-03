---
title: Ubuntu에 Tinc VPN 설치하기
date: 2021-03-03 22:03:33
category: dev-etc
thumbnail: { thumbnailSrc }
draft: false
---

Tinc VPN을 설치할 일이 있어 조사한 내용을 정리하고자 한다.<br />
이 내용은 Tinc VPN 1.0.26 버전을 기준으로 작성했다.<br />
따라서 가장 최근 Stable 버전인 Tinc VPN 1.0.36 과 최신베타버전 Tinc VPN 1.1pre17 과는 조금 상이할 수 있다.

## 모든 노드 공통 설정

### tinc 설치

```
sudo apt update
sudo apt upgrade
sudo apt install tinc
```

### hosts 경로 생성

```sh
sudo mkdir -p /etc/tinc/노드이름/hosts
```

### 현재 사용중인 사설 ip 확인

`ifconfig` or `ip addr | grep inet`

### 방화벽 정책 추가

```sh
sudo ufw enable
sudo ufw allow 22 ## ssh port
sudo ufw allow 655 ## tinc vpn default port
```

## 서버 노드 설정

### tinc.conf 설정 파일 생성 및 수정

```sh
sudo vi /etc/tinc/노드이름/tinc.conf
```
```sh
Name = 호스트이름
Device = /dev/net/tun
Interface = tun0

BindToAddress = 사용중인 사설IP
AddressFamily = ipv4
```

### tinc 호스트 public key, private key 생성 및 수정
```sh
sudo tincd -n 노드이름 -K4096
```
```sh
sudo vi /etc/tinc/노드이름/hosts/호스트이름
```
```sh
Address = 사용중인 사설IP
Subnet = x.x.x.1/32
Port = 655

-----BEGIN RSA **PUBLIC** KEY-----
.....
...
..
```
### tinc 시작시 명령어 입력
```sh
sudo vi /etc/tinc/노드이름/tinc-up
```
```sh
#!/bin/sh
/sbin/ip link set $INTERFACE up
/sbin/ip addr add x.x.x.1/32 dev $INTERFACE
/sbin/ip route add x.x.x.0/24 dev $INTERFACE
```

### tinc 종료시 명령어 입력
```sh
sudo vi /etc/tinc/노드이름/tinc-down
```
```sh
#!/bin/sh
/sbin/ip route del x.x.x.0/24 dev $INTERFACE
/sbin/ip addr del x.x.x.1/32 dev $INTERFACE
/sbin/ip link set $INTERFACE down
```

### tinc-up tinc-down 실행 가능하도록 권한 수정
```sh
sudo chmod 755 /etc/tinc/노드이름/tinc-*
```

## 클라이언트 노드 설정

### tinc.conf 파일 생성 및 수정
```sh
sudo vi /etc/tinc/노드이름/tinc.conf
```
```sh
Name = 호스트이름
Device = /dev/net/tun
Interface = tun0

ConnectTo = 서버 호스트이름
BindToAddress = 사용중인 사설IP
AddressFamily = ipv4
```

### tinc 호스트 public key, private key 생성 및 수정
```sh
sudo tincd -n 노드이름 -K4096
```
```sh
Generating 4096 bits keys:
....................++++ p
......................................................................++++
q
Done.
```
```sh
sudo vi /etc/tinc/노드이름/hosts/호스트이름
```
```sh
Subnet = x.x.x.n/32
Port = 655

-----BEGIN RSA PUBLIC KEY-----
MIICC..........................................................0
...
..
....
............................................................==
-----END RSA PUBLIC KEY-----
```
### tinc 시작시 명령어 입력
```sh
sudo vi /etc/tinc/노드이름/tinc-up
```
```sh
#!/bin/sh

/sbin/ip link set $INTERFACE up
/sbin/ip addr add x.x.x.n/32 dev $INTERFACE
/sbin/ip route add x.x.x.0/24 dev $INTERFACE
```

### tinc 종료시 명령어 입력
```sh
sudo vi /etc/tinc/노드이름/tinc-down
```
```sh
#!/bin/sh

/sbin/ip route del x.x.x.0/24 dev $INTERFACE
/sbin/ip addr del x.x.x.n/32 dev $INTERFACE
/sbin/ip link set $INTERFACE down
```

### tinc-up tinc-down 실행 가능하도록 권한 수정
```sh
sudo chmod 755 /etc/tinc/노드이름/tinc-*
```

## 호스트 공개 키 교환

서버는 클라이언트들 모든 키를 가지고 있어야함.<br />
각 클라이언트는 서버 공개 키만 추가로 가지고 있으면 됨.

### openssh-server 설치
```sh
sudo apt-get install openssh-server
```

### scp 명령어로 호스트 파일 전송
```sh
sudo scp /etc/tinc/노드이름/hosts/전송할호스트이름 PC ID\@PC IP:~
```

### 전송받은 호스트 파일 /hosts로 이동
```sh
sudo mv ~/전송받은호스트파일이름
/etc/tinc/노드이름/hosts/전송받은호스트파일이름
```

## 부팅시 tinc 자동시작
```sh
sudo vi /etc/tinc/nets.boot
```
```sh
노드이름
```

## tinc 시작
```sh
sudo systemctl enable tinc
sudo systemctl start tinc
````

## 연결 테스트

각 노드에서 설정한 ip로 ping 테스트 진행
```sh
ping x.x.x.n
```

### 참고
>[Tinc VPN](https://www.tinc-vpn.org)
>
>[Ubuntu Install Tinc and Set Up a Basic VPN - nixCraft](https://www.cyberciti.biz/faq/ubuntu-install-tinc-and-set-up-a-basic-vpn/)
>
>[ubuntu에서 tinc vpn 설정하기](https://blog.hanaoto.me/tinc_vpn_setting_guide/)
