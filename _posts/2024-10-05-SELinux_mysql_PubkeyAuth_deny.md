---
title: "[MySQL] SElinux 로 인해 mysql 계정의 공개키 접속이 차단되는 현상"
excerpt: "SElinux로 인해 mysql 계정의 공개키 접속이 차단되는 현상이 있어 공유합니다."
#layout: archive
categories:
 - Mysql
tags:
  - [mysql, mariadb]
#permalink: mysql-architecture
toc: true
toc_sticky: true
date: 2024-10-05
last_modified_at: 2024-10-05
comments: true
---

### ✏️mysql 서버를 패스워드 인증없이 SSH 키로 접근하기
--- 
mysql 서버의 접근을 용이하게 하기 위해 보통 우리는 SSH 키 인증 방식을 설정하여 접근합니다. 대강의 루틴을 그림으로 보면 이렇습니다.

![SSH 키 인증 방식 설정](https://github.com/user-attachments/assets/37af50d5-1717-4781-8332-1ee42f3f8ed4)
[그림1] SSH 키 인증 방식 설정

#### 1. SSH 키 생성

먼저, 클라이언트에서 ssh-keygen 명령을 통해 공개키와 개인키를 생성합니다. 이때 공개키는 .pub 파일로, 개인키는 id_rsa라는 파일로 생성됩니다.

{% include codeHeader.html name="SSH 키 생성" %}
```bash
ssh-keygen -t rsa
```
<br>

#### 2. 공개키를 서버에 복사

클라이언트에서 생성된 공개키를 서버에 전달해야 합니다. 이를 위해 ssh-copy-id 명령을 사용하여 클라이언트의 공개키를 서버에 복사합니다. 공개키는 서버의 ~/.ssh/authorized_keys 파일에 저장되며, 이 파일을 통해 서버는 클라이언트의 인증을 처리합니다.

{% include codeHeader.html name="ssh-copy-id 로 공개키를 서버에 복사" %}
```bash
ssh-copy-id 사용자명@서버_IP
```
<br>

그런데 위의 명령어를 사용하기 위해서는 패스워드가 필요하기 때문에 보안상의 이유로 패스워드 없이 계정과 홈디렉토리만 있어야 한다면 클라이언트의 공개키 파일 내용을 그대로 복사해 서버의 authorized_keys 파일에 붙여넣어도 됩니다.

{% include codeHeader.html name="id_rsa.pub 로 공개키를 서버에 복사" %}
```bash
#클라이언트에서 작업
RESULT=`cat ~/.ssh/id_rsa.pub`
echo ${RESULT}

#위의 결과를 복사해서 서버에서 작업
#${RESULT} 값을 붙여넣고 저장(:wq)
vi ~/.ssh/authorized_keys

#authorized_keys 권한조정
chmod 600 ~/.ssh/authorized_keys
```
<br>

#### 3. 서버에서 공개키 인증 설정

서버에서 할 작업으로 /etc/ssh/sshd_config 파일에서 공개키 인증을 활성화해야 합니다. 해당 설정을 통해 서버는 공개키를 기반으로 연결할 수 있게 됩니다. 설정이 완료되면 SSH 서버를 재시작합니다.

{% include codeHeader.html name="서버에서 공개키 인증 설정" %}
```bash
#서버에서 작업
sudo vi /etc/ssh/sshd_config

#아래 두항목을 찾아 설정
PasswordAuthentication no #패스워드 접근을 허용할건지 설정(Optional)
PubkeyAuthentication yes

#SSH 서비스를 재시작
sudo systemctl restart sshd

```
<br>

#### 4. 패스워드 없이 접속

이제 클라이언트는 패스워드 없이 SSH 접속할 수 있습니다. 클라이언트는 개인키를 사용하여 서버에 인증 요청을 보내고, 서버는 미리 저장된 공개키로 이 요청을 검증하여 접속을 허용합니다.

{% include codeHeader.html name="패스워드 없이 접속" %}
```bash
#클라이언트에서 작업
ssh 사용자명@서버_IP
```

<br>

### ❓mysql 유저만 SSH 키 접근이 안된다?
---

Rocky8.8 환경에서 mysql 계정에 쉽게 접근하기 위해 공개키를 각 서버들의 authorized_keys 에 넣고 SSH 키 인증으로 연결을 시도했지만 계속해서 패스워드를 인증하라는 문구가 발생하였습니다. 그리고 신기한 것은 유독 mysql 유저로 연결을 맺으려고 할때에만 SSH 키 접근이 불가했습니다. 조금더 자세히 살펴보기 위해 ssh 접속 시 -v(verbose) 옵션을 주어 다른 계정과의 차이를 확인해보았습니다.




<br/>




### 📚 참고자료
---
- [~/.ssh/authorized_keys 에 public key 를 추가했으나 자동 로그인이 안 됨](https://www.lesstif.com/system-admin/ssh-authorized_keys-public-key-17105307.html)

<br/>
---

{% assign posts = site.categories.Mysql %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}