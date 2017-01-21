---
layout: post
title: ubuntu 16.04에서 iptables설정 save하기
tag: server, ubuntu
---

현재 서버의 포트가 외부로 연결된게 두 개 있다.  
하나는 Flask로 `HongikFood`학식알리미를 서빙하고 있는 5000번 포트  
다른 하나는 `supervisor`의 상태를 웹에서 확인하게끔 열어놓은 8017번 포트  
(`jupyter notebook`도 오픈해놔야 하는게 귀찮아서 미루고만 있다.)

근데 가끔가다 자기 멋대로 서버가 재부팅 될 때마다 (왜그러는지는 아직 모른다...) 열어두었던 포트가 다시 닫혀버리는 바람에 일일이 다시 열어줬었다.  
예전 서버에서는 안 그랬는데 왜그럴까 하던 와중에 `iptables`의 rule을 바꿔놓고 저장시키지 않았다는걸 깨달았다.  

일단 

```bash
iptables -I INPUT -p tcp --dport 5000 -j ACCEPT
iptables -I INPUT -p tcp --dport 8017 -j ACCEPT
```

명령어를 통해 5000번과 8017번 포트를 열어두고 `sudo service iptables save`명령어를 통해 저장시키려고 보니 잘못되었다한다.  
왜 맨날 저번엔 됐는데 이번엔 안되는 것 일까.  

하여튼 구글링 해보니 우분투 16.04에서는 `sudo apt install iptables-persistent`을 통해 패키지를 설치해주고  

```bash
sudo netfilter-persistent save
sudo netfilter-persistent reload
```

를 해줘야 한다 하더라.  
다음에는 까먹지 말자.
