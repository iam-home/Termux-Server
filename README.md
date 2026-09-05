# Termux-Server

- 이걸 왜 하려고 했느냐
마인크래프트를 플레이 하는 데 태블릿에서 하던 진행 상황 그대로 컴퓨터로 이어서 하고 싶음.
aternos를 쓰면 가장 쉬운 방법으로 할 수는 있는데 수동으로 직접 서버를 켜줘야 한다는 게 꽤 불편했었음.
오라클 클라우드를 쓰면 24시간 서버를 만들 수 있는 거로 알고 있는데 해외 결제가 가능한 카드가 없어서 시도도 못 해봄.

termux를 써서 24시간 서버를 만들까?

문제 1) 와이파이 연결

문제 2) 서브 기기에 할 시 항상 충전기 꽂아둬야됨

문제 3) 메인 기기에 할 시 실사용에 지장이 갈 수 있음?

-> termux를 써서 aternos보다만 더 편할 수 있게 만들자!

시작

termux에 우분투를 설치하고, 우분투에 Paper를 설치할 겁니다. GeyserMC 플러그인을 설치해서 베드락 친구들도 들어올 수 있도록 할 예정입니다.

포트포워딩은 하기 싫기도 하고 뭐랄까 좀 문제가 있을 거 같아서 안 할 겁니다. Tailscale은 약간 번거로워서 안 할 겁니다.
-> Playit을 쓰자!


# 1. 패키지 저장소 최신화
pkg update && pkg upgrade -y

# 2. 우분투 설치 도구 설치
pkg install proot-distro -y

# 3. 우분투 설치
proot-distro install ubuntu

# 4. 우분투 로그인
proot-distro login ubuntu

# 5. 우분투 패키지 최신화
apt update && apt upgrade -y

# 6. 자바 설치
apt install -y openjdk-25-jre-headless

버전 확인: java -version

# 7. 마인크래프트 서버 폴더 생성 및 이동
mkdir mc-server && cd mc-server

# 8. PaperMC 다운로드
우분투에서 설치하려 했는데 자꾸 에러나고 시간만 잡아먹길래 폰으로 먼저 PaperMC 사이트가서 최신 버전으로 다운받고,

cp /storage/emulated/0/Download/server/jar server.jar

해서 우분투로 가져옴. 파일이름은 server로 내가 바꾼 거


nano server.properties

해서  Ctrl + W 누르고 online-mode를 입력해서 해당 줄로 이동,

online-mode=true을 online-mode=false로 변경


Ctrl + O -> Enter (저장)

Ctrl + X (종료)


# 9. 플러그인 폴더 생성
cd

mkdir plugins

# 10. GeyserMC 및 Floodgate 설치
일단 폴더로 이동

cd plugins

PaperMC 떄랑 똑같이 우분투에서 바로 설치하려니까 에러가 너무 많이 나고 시간도 너무 많이 낭비함. PaperMC랑 플러그인 설치 시도한 시간한 해도 5시간 될 거 같음.

여기서도 똑같이 폰으로 먼저 GeyserMC랑 floodgate 설치하고 복사해주겠습니다.

cp /storage/emulated/0/Download/Geyser-Spigot.jar Geyser-Spigot.jar

cp /storage/emulated/0/Download/floodgate-spigot.jar floodgate-spigot.jar

이제 문제가 없는 줄 알았더니 geyser가 말을 안 들음. 이유는 모르겠는데 실행이 안되는 듯 보임.

Geyser-Standalone.jar로 다운 받아서 그냥 분리해줬음

rm -f Geyser-Spigot.jar 해서 원래 geyser 지우고 다시,

cp /storage/emulated/0/Download/Geyser-Standalone.jar Geyser-Standalone.jar


nano config.yml

해서 auth-type: online 부분을

auth-type: offline으로 변경.


Ctrl + O -> Enter (저장)

Ctrl + X (쫑료)

# 11. Playit 설치
여기는 어떻게 했는지 잘 모르겠어서 일단 그냥 썼던 명령어들 그대로 써보겠습니다. 막 에러 나다가 아래 명령어들을 써서 해결됐습니다.

rm -rf /root/.config/playit_gg /usr/local/bin/playit

curl -SsL -o /usr/local/bin/playit https://github.com/playit-cloud/playit-agent/releases/download/v0.15.26/playit-linux-aarch64

chmod +x /usr/local/bin/playit

playit

해서 나온 링크를 복사해서 폰 브라우저로 붙여넣고 접속,

계정 만들고,

터널 생성,

Minecraft Java 선택 후 추가,

여기서 나온 주소가 자바 서버의 주소.

같은 방법으로 Minecraft Bedrock 선택 후 추가,

여기서 나온 주소가 베드락 서버의 주소, 주소 옆에 포트.

# 12. 서버 최초 실행
server.jar 있는 폴더가서

java -Djava.awt.headless=true -Xms1G -Xmx1G -jar server.jar nogui

# 13. EULA 동의 수락
echo "eula=true" > eula.txt

cat eula.txt 했을 때 eula=true 한 줄 나오면 된 거심.

# 14. 서버 실행
java -Djava.awt.headless=true -Xms1G -Xmx1G -jar server.jar nogui

서버 끄는 건 stop

# 15. 단축키 만들기
- 서버 시작
  
  nano start.sh 해서 스크립트 작성


// 아래 스크립트 내용

    #!/bin/bash
    
    tmux new-session -d -s mc 'cd /root/mc-server && java -Djava.awt.headless=true -Xms1G -Xmx1G -jar server.jar nogui'
    
    tmux new-session -d -s geyser 'cd /root/plugins && java -jar Geyser-Standalone.jar'
    
    tmux new-session -d -s play 'playit'




Ctrl + O -> Enter

Ctrl + X

-------------------------

- 서버 끄기
  
  nano stop.sh 해서 스크립트 작성

// 아래 스크립트 내용

    #!/bin/bash
    
    tmux send-keys -t mc "stop" C-m
    
    tmux send-keys -t geyser "geyser stop" C-m
    
    tmux kill-session -t play




Ctrl + O -> Enter

Ctrl + X

# 16. 스크립트 실행 권한 부여
chmod +x start.sh

chmod +x stop.sh


시작하려면 ./start.sh

끄려면 ./stop.sh

# 17. 자동화 및 더 간략하게
exit 해서 우분투에서 나가기

nano ~/.bashrc

proot-distro login ubuntu --shared-tmp -- bash -c "cd /root && ./start.sh; bash"


Ctrl + O -> Enter

Ctrl + X


다시 우분투로 가야됨

proot-distro login ubuntu


nano ~/.bashrc

alias s='/root/stop.sh'


Ctrl + O -> Enter

Ctrl + X


nano ~/.bash_profile


// 아래 이거 쓰기

    if [ -f ~/.bashrc ]; then
    
        . ~/.bashrc
        
    fi




Ctrl + O -> Enter

Ctrl + X


* alias s='./stop.sh' 여기서 s가 아니라 다른 걸 쓰면 그 다른 걸 입력했을 때 서버가 꺼집니다. ex) alias mcstop='./stop.sh'이면 mcstop을 썼을 때 서버가 꺼짐


---------------------------------------------------------------------------------------

기억 더듬어가면서 쓴 거라 혹시 안되는 게 있을 수 있습니다. 안되는 건 알아서 찾아서 해보시길~ 근데 아마 되지 않을까 기대해봅니다

아 근데 하나 불편한 게 서버 켜는 속도가 좀 느립니다. Playit이랑 GeyserMC는 켜졌는데 서버가 아직 켜지기 전이더라고요.
그래도 aternos보다는 빠르고 편하긴 한데 어쩌다 한 번 정도는 불편함을 느낄 가능성이 있다? 정도는 되는 거 같습니다.


-사용기기-

샤오미 홍미노트 11 프로 5G

AP: SD695

제가 전문가가 아니라 다른 기기에서는 어떻게 되는지 모르겠네요,,
