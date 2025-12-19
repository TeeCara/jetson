Jetson Nano + Arduino 온습도 감지 기기

개요

Jetson Nano와 Arduino를 연결하여 DHT11 온습도 센서의 값을 실시간으로 읽고, 사용자의 질문에 Gradio 창반 UI로 응답하는 프로젝트.

---

프로젝트 구조

- `Jetson Nano`에서 `Arduino`와 연결
- `DHT11` 센서를 통해 온도와 습도 수집
- `Function Calling` 기반 GPT 연동
- `Gradio` 기반 GUI 창반

---

사전 준비

소프트웨어

- Arduino IDE: [버전 1.8.19](https://www.arduino.cc/en/software) 또는 2.3.2
- Python 3.8
- Jupyter Notebook
- Gradio
- OpenAI API Key

하드웨어

- Jetson Nano
- Arduino (UNO 등)
- DHT11 센서

---

설치 가이드

1. Arduino 설치 (Jetson Nano 기준)

sudo apt update
sudo apt install openjdk-8-jdk
wget https://downloads.arduino.cc/arduino-1.8.19-linuxaarch64.tar.xz
tar -xf arduino-1.8.19-linuxaarch64.tar.xz
cd arduino-1.8.19
sudo ./install.sh
sudo usermod -aG dialout $USER
newgrp dialout


2. Python 3.8 설치 및 가상환경 구성

sudo apt install libbz2-dev libssl-dev libffi-dev python3-dev
wget https://www.python.org/ftp/python/3.8.12/Python-3.8.12.tar.xz
tar -xf Python-3.8.12.tar.xz
cd Python-3.8.12
./configure --enable-loadable-sqlite-extensions --with-bz2
make -j4
sudo make altinstall

python3.8 -m venv myenv
source myenv/bin/activate
pip install jupyter notebook gradio pandas ipykernel openai pyserial
python -m ipykernel install --user --name=myenv --display-name="Python (myenv)"

아두이노 코드 예제

#include <SimpleDHT.h>
int pinDHT11 = 2;
SimpleDHT11 dht11(pinDHT11);

void setup() {
  Serial.begin(115200);
}

void loop() {
  byte temperature = 0;
  byte humidity = 0;

  if (dht11.read(&temperature, &humidity, NULL) == SimpleDHTErrSuccess) {
    Serial.print((int)temperature);
    Serial.print(",");
    Serial.println((int)humidity);
  }

  delay(2000);
}

감지 실행 코드 (Python)

`DHT11Assistant` 클래스를 기반으로 온습도 창반을 실행함.

python chatbot_dht11.py

Gradio 인터페이스가 실행되며, "지금 온도는?", "습도는 얼마야?" 같은 질문을 할 수 있음.

Function Calling 예제

{
  "type": "function",
  "function": {
    "name": "get_temperature",
    "description": "현재 온도값을 가져옵니다",
    "parameters": {
      "type": "object",
      "properties": {},
      "required": []
    }
  }
}

온도/습도 데이터를 필요할 때 함수 형태로 호출하며, GPT는 호출 결과를 활용해 자연어로 응답한다.


날씨 API 창반 예제

- API: [OpenWeatherMap](https://openweathermap.org/)
- API Key 필요
- 지역 입력 시 현재 날씨, 온도, 습도 반환



관련 링크

- [Jetson & Arduino 연결 프로젝트 GitHub](https://github.com/ralralra/jetson_DLI)
- [Function Calling 설명](https://platform.openai.com/docs/guides/function-calling)
- [OpenWeatherMap API 한국어 설명 블로그](https://icedhotchoco.tistory.com/entry/OpenWeatherMap-날씨-API)

주의 사항

- `Jetson.GPIO` 패키지가 가상환경에서 동작하지 않을 경우 `/usr/lib/...` 디렉토리의 패키지를 수동으로 복사해야 한다다.
- `/dev/ttyACM0` 호시가 필요한 경우 권한 설정:

sudo chmod 666 /dev/ttyACM0




#include <WiFiS3.h>                // UNO R4 WiFi 전용
#include <WiFiUdp.h>
#include <NTPClient.h>

#include <SimpleDHT.h>
#include <Wire.h>

// LCD (UNO R4 WiFi 호환)
#include <hd44780.h>
#include <hd44780ioClass/hd44780_I2Cexp.h>

// ───────────────────────────────
// WiFi 설정
// ───────────────────────────────
const char* ssid     = "Ac";      
const char* password = "ldh12345";
//위의 이 코드들을 아두이노에 넣어야함
//이 과정에서 오류가 발생할 시에는 와이파이를 바꾸거나 기기 재시작

//사진 이메일로 보내기
//이 코드들은 젯슨 나노에 넣는 코드들임
smartfarm_angle
JetPack 4.6에서 이 코드를 실행하기 전에 필요한 라이브러리 설치 가이드입니다:

1. 시스템 업데이트
sudo apt update
sudo apt upgrade -y
2. Python3 및 pip 설치 확인
sudo apt install -y python3 python3-pip
python3 --version
pip3 --version
3. OpenCV 설치
JetPack 4.6에는 OpenCV가 포함되어 있지만, Python 바인딩 확인:

# 이미 설치된 OpenCV 확인
python3 -c "import cv2; print(cv2.__version__)"
dli@dli:~$ python3 -c "import cv2; print(cv2.version)" 4.1.1

만약 에러가 나면:

sudo apt install -y python3-opencv
또는 pip로 설치:

pip3 install opencv-python
4. Schedule 라이브러리 설치
pip3 install schedule
dli@dli:~$ pip3 install schedule Collecting schedule Downloading https://files.pythonhosted.org/packages/eb/3b/040bd180eaef427dd160562ee66adc9f4f67088185c272edcdb899c609c7/schedule-1.1.0-py2.py3-none-any.whl Installing collected packages: schedule Successfully installed schedule-1.1.0

5. V4L2 관련 패키지 설치 (카메라 지원)
sudo apt install -y v4l-utils
6. 카메라 권한 설정
sudo usermod -a -G video $USER
# 재부팅 필요
sudo reboot
7. 전체 설치 스크립트
한 번에 실행:

#!/bin/bash

# 시스템 업데이트
sudo apt update && sudo apt upgrade -y

# 필수 패키지
sudo apt install -y python3 python3-pip v4l-utils

# Python 라이브러리
pip3 install schedule opencv-python

# 카메라 권한
sudo usermod -a -G video $USER

echo "✅ 설치 완료! 재부팅 후 사용하세요."
echo "재부팅: sudo reboot"
8. 설치 확인
# OpenCV 확인
python3 -c "import cv2; print('OpenCV:', cv2.__version__)"

# Schedule 확인
python3 -c "import schedule; print('Schedule: OK')"

# 카메라 확인
v4l2-ctl --list-devices
9. Gmail 앱 비밀번호 설정
코드의 EMAIL_CONFIG에서 비밀번호 설정 필요:

Google 계정 → 보안 → 2단계 인증 활성화
"앱 비밀번호" 생성
생성된 16자리 비밀번호를 코드에 입력
10. 스크립트 실행
# 실행 권한 부여
chmod +x plant_monitor.py

# 실행
python3 plant_monitor.py

# 백그라운드 실행 (선택)
nohup python3 plant_monitor.py > /dev/null 2>&1 &
다시 정리하자면(에러가 나서 다시 정리)ㅅ

 전체 설치 및 실행 가이드
 설치 스크립트 실행
# 설치 스크립트 생성
cat > install.sh << 'EOF'
#!/bin/bash
# 시스템 업데이트
sudo apt update
# 필수 패키지
sudo apt install -y python3-pip v4l-utils python3-opencv
# Schedule만 pip로 설치
pip3 install schedule
# 카메라 권한
sudo usermod -a -G video $USER
# 설치 확인
echo "=== 설치 확인 ==="
python3 -c "import cv2; print('✅ OpenCV:', cv2.__version__)"
python3 -c "import schedule; print('✅ Schedule: OK')"
python3 -c "import smtplib; print('✅ Email: OK')"
echo ""
echo "✅ 설치 완료! 재부팅 후 사용하세요."
echo "재부팅: sudo reboot"
EOF

# 실행 권한 부여
chmod +x install.sh

# 설치 실행
./install.sh
재부팅
sudo reboot
 메인 프로그램 생성
재부팅 후:

cat > plant_monitor.py << 'EOF'
#!/usr/bin/env python3
import cv2
import time
import schedule
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.base import MIMEBase
from email.mime.text import MIMEText
from email import encoders
from datetime import datetime

# 이메일 설정
EMAIL = 'jmerrier0910@gmail.com'
PASSWORD = 'smvrcqoizxbxmyhy'

# 촬영 시간
TIMES = ["05:00", "12:00", "20:50"]

def take_and_send():
    """사진 찍고 바로 이메일 보내기"""
    print(f"📸 {datetime.now().strftime('%H:%M:%S')} 사진 촬영 시작...")
    
    # 1. 사진 촬영
    cap = cv2.VideoCapture(0)
    time.sleep(1)
    ret, frame = cap.read()
    cap.release()
    
    if not ret:
        print("❌ 촬영 실패")
        return
    
    # 2. 사진 저장
    filename = datetime.now().strftime("%Y%m%d_%H%M%S.jpg")
    cv2.imwrite(filename, frame)
    print(f"✅ 저장: {filename}")
    
    # 3. 이메일 전송
    try:
        msg = MIMEMultipart()
        msg['From'] = EMAIL
        msg['To'] = EMAIL
        msg['Subject'] = f"🌱 식물사진 {datetime.now().strftime('%m/%d %H:%M')}"
        
        msg.attach(MIMEText("식물 사진입니다 🌿", 'plain'))
        
        # 사진 첨부
        with open(filename, 'rb') as f:
            part = MIMEBase('application', 'octet-stream')
            part.set_payload(f.read())
        encoders.encode_base64(part)
        part.add_header('Content-Disposition', f'attachment; filename={filename}')
        msg.attach(part)
        
        # 전송
        server = smtplib.SMTP_SSL('smtp.gmail.com', 465)
        server.login(EMAIL, PASSWORD)
        server.send_message(msg)
        server.quit()
        
        print("✅ 이메일 전송 완료!")
        
    except Exception as e:
        print(f"❌ 전송 실패: {e}")

if __name__ == "__main__":
    print("🌱 식물 모니터링 시작")
    print(f"📅 촬영 시간: {', '.join(TIMES)}")
    
    # 스케줄 등록
    for t in TIMES:
        schedule.every().day.at(t).do(take_and_send)
        print(f"⏰ {t} 등록")
    
    print("🚀 실행 중... (Ctrl+C 종료)\n")
    
    while True:
        schedule.run_pending()
        time.sleep(60)
EOF

chmod +x plant_monitor.py
테스트 (즉시 촬영/전송)
cat > test_now.py << 'EOF'
#!/usr/bin/env python3
import cv2
import smtplib
import time
from email.mime.multipart import MIMEMultipart
from email.mime.base import MIMEBase
from email.mime.text import MIMEText
from email import encoders
from datetime import datetime

print("📸 테스트 촬영 시작...")

cap = cv2.VideoCapture(0)
time.sleep(1)
ret, frame = cap.read()
cap.release()

if not ret:
    print("❌ 카메라 오류")
    exit(1)

filename = 'test.jpg'
cv2.imwrite(filename, frame)
print('✅ 사진 저장')

msg = MIMEMultipart()
msg['From'] = 'jmerrier0910@gmail.com'
msg['To'] = 'jmerrier0910@gmail.com'
msg['Subject'] = '🌱 테스트'
msg.attach(MIMEText('테스트입니다', 'plain'))

with open(filename, 'rb') as f:
    part = MIMEBase('application', 'octet-stream')
    part.set_payload(f.read())
encoders.encode_base64(part)
part.add_header('Content-Disposition', f'attachment; filename={filename}')
msg.attach(part)

server = smtplib.SMTP_SSL('smtp.gmail.com', 465)
server.login('jmerrier0910@gmail.com', 'smvrcqoizxbxmyhy')
server.send_message(msg)
server.quit()
print('✅ 이메일 전송!')
EOF

chmod +x test_now.py
python3 test_now.py
result image

메인 프로그램 실행
# 포그라운드 실행 (로그 보면서)
python3 plant_monitor.py

# 또는 백그라운드 실행
nohup python3 plant_monitor.py > plant.log 2>&1 &

# 백그라운드 실행 확인
ps aux | grep plant_monitor

# 로그 확인
tail -f plant.log
자동 시작 설정 (부팅시 자동 실행)
# systemd 서비스 생성
sudo nano /etc/systemd/system/plant-monitor.service
내용:

[Unit]
Description=Plant Monitoring Service
After=network.target

[Service]
Type=simple
User=dli
WorkingDirectory=/home/dli
ExecStart=/usr/bin/python3 /home/dli/plant_monitor.py
Restart=always

[Install]
WantedBy=multi-user.target
활성화:

# 서비스 시작
sudo systemctl start plant-monitor

# 부팅시 자동 시작
sudo systemctl enable plant-monitor

# 상태 확인
sudo systemctl status plant-monitor

# 로그 확인
sudo journalctl -u plant-monitor -f
요약
설치: ./install.sh → 재부팅
테스트: python3 test_now.py (즉시 촬영/전송)
실행: python3 plant_monitor.py
자동시작: systemd 서비스 등록

매일 05:00, 12:00, 20:50에 자동 촬영
촬영 즉시 이메일 전송
부팅시 자동 시작

//자주 발생하는 오류들
❗ 자주 발생하는 오류 & 해결법 총정리


---

1️⃣ import cv2 에러

❌ 오류 예시

ModuleNotFoundError: No module named 'cv2'

📌 원인

OpenCV Python 바인딩이 설치되지 않음

pip opencv-python과 JetPack 기본 OpenCV 충돌


✅ 해결

JetPack 4.6에서는 apt 설치 방식 권장

sudo apt install -y python3-opencv

확인:

python3 -c "import cv2; print(cv2.__version__)"

⚠️ 주의
Jetson에서는 가급적:

pip3 install opencv-python ❌ (비권장)
apt install python3-opencv ✅


---

2️⃣ OpenCV 버전 출력 오류 (cv2.**version**)

❌ 오류 예시

AttributeError: module 'cv2' has no attribute 'version'

📌 원인

오타 (version → __version__)


✅ 해결

python3 -c "import cv2; print(cv2.__version__)"


---

3️⃣ 카메라 열기 실패 (VideoCapture(0))

❌ 오류 예시

❌ 촬영 실패

또는

ret == False

📌 원인

카메라 권한 없음

/dev/video0 접근 불가

다른 프로세스가 카메라 점유


✅ 해결 ① 권한 추가

sudo usermod -a -G video $USER
sudo reboot

✅ 해결 ② 카메라 인식 확인

v4l2-ctl --list-devices
ls /dev/video*

✅ 해결 ③ 인덱스 변경

cv2.VideoCapture(1)


---

4️⃣ Gmail 로그인 실패

❌ 오류 예시

SMTPAuthenticationError: 534-5.7.9 Application-specific password required

📌 원인

Gmail 일반 비밀번호 사용

2단계 인증 미설정


✅ 해결

1. Google 계정 → 보안


2. 2단계 인증 활성화


3. 앱 비밀번호 생성


4. 16자리 비밀번호만 코드에 사용



PASSWORD = 'xxxx xxxx xxxx xxxx'

⚠️ 공백 없이 입력


---

5️⃣ schedule 모듈 없음

❌ 오류 예시

ModuleNotFoundError: No module named 'schedule'

📌 원인

pip 패키지 미설치


✅ 해결

pip3 install schedule

확인:

python3 -c "import schedule; print('OK')"


---

6️⃣ systemd 서비스 실행 안 됨

❌ 오류 예시

Active: failed

📌 원인

경로 오류

실행 권한 없음

USER 계정 불일치


✅ 해결 체크리스트

[Service]
User=dli
WorkingDirectory=/home/dli
ExecStart=/usr/bin/python3 /home/dli/plant_monitor.py

권한 확인:

chmod +x /home/dli/plant_monitor.py

로그 확인:

sudo journalctl -u plant-monitor -f


---

7️⃣ 부팅 후 이메일 전송 안 됨

📌 원인

네트워크보다 서비스가 먼저 실행됨


✅ 해결

plant-monitor.service 수정:

[Unit]
After=network-online.target
Wants=network-online.target

적용:

sudo systemctl daemon-reload
sudo systemctl restart plant-monitor


---

8️⃣ 백그라운드 실행 후 멈춤

❌ 증상

nohup 실행했는데 동작 안 함


📌 원인

로그 확인 안 해서 원인 파악 불가


✅ 해결

nohup python3 plant_monitor.py > plant.log 2>&1 &
tail -f plant.log


---

9️⃣ 사진은 저장되는데 이메일만 실패

📌 원인

방화벽 / 네트워크 문제

Gmail SMTP 차단


✅ 해결

테스트:

ping smtp.gmail.com

SMTP 포트:

smtplib.SMTP_SSL('smtp.gmail.com', 465)


---

🔍 디버깅 필수 체크 명령어 모음

python3 -c "import cv2"
python3 -c "import schedule"
v4l2-ctl --list-devices
ls /dev/video*
ps aux | grep plant_monitor
tail -f plant.log


---

✅ 최종 안정화 팁

Jetson에서는 apt 기반 라이브러리 우선

카메라는 재부팅 후 테스트

Gmail은 반드시 앱 비밀번호

systemd는 로그 먼저 확인
