# born2beroot

### 1. Project Overview

- VM 동작 원리
- 선택한 OS에 대한 설명, 이유
- CentOS와 Debian 차이
- VM 사용 목적
- aptitude와 apt 차이, AppArmor

### 2. Simple Setup

- GUI가 아니어야함
- root가 아닌 사용자로 로그인
- password 사용해서 machine에 로그인 해야함
- UFW 동작 상태 확인 `sudo ufw status verbose`
- ssh service 동작(UFW 동작 상태 확인) `systemctl status ssh`
-![image](https://user-images.githubusercontent.com/69064310/168206900-394afa9e-d319-478d-ae67-094341f4c351.png)
- OS 확인 hostnamectl
![image](https://user-images.githubusercontent.com/69064310/168206955-f6e22096-f425-4da2-b976-f6293406ad18.png)

### 3. User

- 현재 사용자의 group status 확인 `id [사용자명]`
- 현재 password 정책 설명
`sudo vi /etc/login.defs`
![image](https://user-images.githubusercontent.com/69064310/168207004-e1ec26d0-d1d2-469a-86eb-d85a497a8813.png)
  -PASS_MAX_DAYS 30 : password는 30일 마다 만료되어야 함  
  -PASS_MIN_DAYS 2 : 비밀번호는 최소 2일을 사용해야 함. 즉, 2일 이후에 비밀 번호 변경 가능  
  -PASS_WARN_AGE 7 : 유저는 비밀번호가 만료되기 7일 전부터 경고 메세지를 받아야 함  
  `sudo apt install libpam-pwquality`  
  `sudo vi /etc/pam.d/common-password`  
![image](https://user-images.githubusercontent.com/69064310/168207063-22fce339-a13a-4d19-bc78-631b473651ce.png)
  -`retry=3` : 암호 입력 3회 까지  
  -`minlen=10` : 암호 최소 길이 10   
  -`difok=7` : 기존 암호와 달라야 하는 문자수 7  
  -`ucredit=-1` : 대문자 1개 이상  
  -`lcredit=-1` : 소문자 1개 이상  
  -`dcredit=-1` : 숫자 1개 이상  
  -`maxrepeat=3` : 최대 3글자까지 중복된 문자 연속 사용 가능  
  -`reject_username` : username이 그대로 또는 뒤집혀서 패스워드에 들어있는지 검사, 들어있으면 거부  
  -`enforce_for_root` : 해당 비밀번호 정책을 root에도 적용  
- 비밀번호 정책의 경우 정책 설정 이후에 생성된 유저나 비밀번호에 대해서만 적용된다. 기존 유저와 root에 대해서 강화된 비밀번호로 변경하고, 비밀번호 기간 적용을 위해서 다음 명령어들을 실행한다.  
    - `$ passwd -e [user_name]` - 다음 로그인 시 비밀번호를 강제적으로 변경해야 한다.  
    - `$ chage -m 2 -M 30 -W 7 [user_name]` - 비밀번호 기간 정책을 설정한다.  
- 사용자별 비밀번호 관련 설정 확인  
    - `$ sudo vi /etc/shadow`  
- 새로운 그룹 생성 (그룹명 : evaluating)  
su -  
`adduser [newuser]`  
`groupadd evaluating`  
`usermod -aG {그룹명1, 그룹명2, ...} [사용자명]` 사용자를 그룹에 추가  
`id [사용자명]` 사용자의 그룹 확인  
- password 정책의 장점  
- password 정책을 이행하는 것의 단점  
-장점 : 보안향상   
-단점 : 지나치게 복잡할 경우 메모 위험, 일부 특수문자 키보드 호환 안 될 가능성  

### 4. hostname and Partitions  

- hostname 확인 `hostnamectl`  
- hostname 변경 `sudo hostnamectl set-hostname [새로운 호스트명]`-변경 후 restart  
- 파티셔닝 상태 확인 `lsblk`  
- LVM 무엇인지 설명  

### 5. Sudo

- sudo 설치 여부 확인 `dpkg -l sudo`
- 새로운 사용자 sudo 그룹에 추가 `sudo usermod -aG sudo [새로운 사용자명]`
- sudo에 대한 엄격한 규칙, sudo 사용하는 가치, 작동 원리
- /var/log/sudo를 통해 확인 sudo visudo

![image](https://user-images.githubusercontent.com/69064310/168207566-ae24fe6a-3551-4018-ab3a-e8aed33ce6f0.png)
  -`passwd_tries=3` : sudo 비밀번호 인증 횟수 지정 (기본값은 3)  
  -`authfail_message=` : 권한 획득 실패 시 출력하는 메시지 (sudo 인증 실패)  
  -`baspass_message` : 비밀번호를 틀렸을 시 출력하는 메시지  
  -`log_input` : sudo 명령어 실행 시 입력된 명령어를 log로 저장  
  -`log_output` : sudo 명령어 실행 시 출력 결과를 log로 저장  
  -`iolog_dir="/var/log/sudo/"` : sudo log 저장 디렉토를 지정  
  -`requiretty` : sudo 명령어 실행 시 TTY에서만 실행 가능  
  *TTY : teletypewrite, 콘솔의 한 종류로 os에서 제공하는 가상 콘솔. 실제 물리적인 장치가 연결된 것이 아닌 커널에서 터미널을 에뮬레이팅하는 것이다.  
  `secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"`  
  -보안상 이유로 sudo 실행 시 사용할 수 있는 명령어의 경로는 제한되어야 함 

### 6. UFW
- UFW 설치 여부, 동작 여부 확인 `sudo ufw status verbose`  
![image](https://user-images.githubusercontent.com/69064310/168207706-10ea91b0-1a6a-4401-aad8-8a6128e7f623.png)
- UFW 설명   
- 현재 rules 확인 `sudo ufw status numbered`  
- 새로운 rule 추가 sudo ufw allow 8080  
`sudo vim /etc/ssh/sshd_config` Port 8080  
- rule 삭제 `sudo ufw  delete allow 8080`  

### 7. ssh
- ssh 서버 설치  `apt search openssh-server`  
- ssh 서버 실행 중인지 여부 확인 `systemctl status ssh`  
- - 4242 포트 허용 `sudo ufw allow 4242/tcp`
- `vi /etc/ssh/sshd_config`

```bash
#Port 22 --- 제거
port 4242 --- 추가

PermitRootLogin no --- no로 변경
```

- 새로운 사용자로 terminal 에서 접근  
-host IP, guest IP 비움  
-ssh 연결 방법 : ssh mkim2@127.0.0.1  
-ssh 기본 포트가 22이므로, host port에 22를 넣으면 포트 지정 안해도 연결 가능  
![image](https://user-images.githubusercontent.com/69064310/168207905-b4a9fb24-7c11-4529-b761-37c97acaa454.png)
![image](https://user-images.githubusercontent.com/69064310/168207931-b433e657-51d5-4f99-9e17-a2abf941868f.png)

  -ip 확인방법 `ip addr show`
  ![image](https://user-images.githubusercontent.com/69064310/168208038-681b9331-0e01-49a7-a204-5a1bddfa583f.png)


### 8. Script Monitoring
- cron: 작업 예약 스케줄러,  cron은 유닉스 계열 운영 체제의 시간 기반 Job Scheduler  
`crontab -e`  cron 편집  
*/10 * * * * /monitoring.sh | wall  
-분, 시, 일, 월, 요일, 실행할 명령  
-10분 마다 [monitoring.sh](http://monitoring.sh) 실행  
`systemctl status cron`  

- script 실행  
`sudo /etc/init.d/cron start`  
`sudo /etc/init.d/cron restart`  
`sudo /etc/init.d/cron stop`  
`./monitoring.sh`  

- 초 단위 실행 (30초 마다)  
```bash
* * * * * bash monitoring.sh
* * * * * sleep 30; bash monitoring.sh
```

- Monitoring Script `vi /monitoring.sh`  

### dhcp 끄기

`apt-get install ifupdown2`  
[VirtualBox Debian - dhclient 없애기, 고정 IP 설정](https://nostressdev.tistory.com/3)

`sudo ifdown -a`  
`sudo ifup -a`  
reboot  

![image](https://user-images.githubusercontent.com/69064310/168208504-e064bb40-b8d5-49d1-be7d-26b93ae04fba.png)

## submit
![image](https://user-images.githubusercontent.com/69064310/168208097-5be089ee-fdcd-4ccd-931c-559611decd76.png)
