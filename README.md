🌱 Jetson Nano + Arduino 온·습도 기반 스마트팜 모니터링 시스템

본 프로젝트는 Jetson Nano와 Arduino를 연동하여
온·습도 센서(DHT11) 데이터 수집, 식물 이미지 자동 촬영, 이메일 전송,
부팅 시 자동 실행(systemd) 까지 구현한 임베디드 기반 스마트팜 모니터링 시스템이다.

센서 제어 · 카메라 처리 · 리눅스 자동화에 집중하여
실제 현장에서 운용 가능한 안정적인 시스템 구현을 목표로 한다.
📌 프로젝트 개요

Arduino + DHT11 센서를 이용한 온·습도 측정

Jetson Nano에서 카메라를 이용한 식물 이미지 촬영

정해진 시간에 자동 실행 (schedule)

촬영 즉시 Gmail SMTP를 통해 이메일 전송

systemd를 이용한 부팅 시 자동 실행

🧩 시스템 구성
🔧 하드웨어

Jetson Nano (JetPack 4.6)

Arduino (UNO / UNO R4)

DHT11 온습도 센서

USB 또는 CSI 카메라

💻 소프트웨어

Ubuntu (JetPack 4.6)

Python 3

OpenCV (apt 기반)

schedule

smtplib (Python 표준 라이브러리)

systemd

🔌 Arduino 코드 (DHT11 센서)

Arduino는 DHT11 센서에서 온도와 습도를 읽어
2초마다 시리얼 통신으로 Jetson Nano에 전송한다.

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


출력 형식: 온도,습도

통신 속도: 115200 baud

⚙️ Jetson Nano 환경 설정 (JetPack 4.6)
1️⃣ 시스템 업데이트
sudo apt update
sudo apt upgrade -y

2️⃣ 필수 패키지 설치
sudo apt install -y python3-pip python3-opencv v4l-utils

3️⃣ Python 라이브러리 설치
pip3 install schedule

4️⃣ 카메라 권한 설정
sudo usermod -a -G video $USER
sudo reboot

🛠 전체 설치 스크립트 (권장)
cat > install.sh << 'EOF'
#!/bin/bash

sudo apt update
sudo apt install -y python3-pip python3-opencv v4l-utils
pip3 install schedule
sudo usermod -a -G video $USER

echo "=== 설치 확인 ==="
python3 -c "import cv2; print('OpenCV:', cv2.__version__)"
python3 -c "import schedule; print('Schedule: OK')"

echo "설치 완료! 재부팅 필요"
EOF

chmod +x install.sh
./install.sh
sudo reboot

📸 메인 프로그램 (plant_monitor.py)
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

EMAIL = "your_email@gmail.com"
PASSWORD = "gmail_app_password"

TIMES = ["05:00", "12:00", "20:50"]

def take_and_send():
    cap = cv2.VideoCapture(0)
    time.sleep(1)
    ret, frame = cap.read()
    cap.release()

    if not ret:
        print("촬영 실패")
        return

    filename = datetime.now().strftime("%Y%m%d_%H%M%S.jpg")
    cv2.imwrite(filename, frame)

    msg = MIMEMultipart()
    msg["From"] = EMAIL
    msg["To"] = EMAIL
    msg["Subject"] = "식물 자동 촬영 이미지"
    msg.attach(MIMEText("자동 촬영된 식물 이미지입니다.", "plain"))

    with open(filename, "rb") as f:
        part = MIMEBase("application", "octet-stream")
        part.set_payload(f.read())
    encoders.encode_base64(part)
    part.add_header("Content-Disposition", f"attachment; filename={filename}")
    msg.attach(part)

    server = smtplib.SMTP_SSL("smtp.gmail.com", 465)
    server.login(EMAIL, PASSWORD)
    server.send_message(msg)
    server.quit()

if __name__ == "__main__":
    for t in TIMES:
        schedule.every().day.at(t).do(take_and_send)

    while True:
        schedule.run_pending()
        time.sleep(60)

▶️ 실행 방법
즉시 실행
python3 plant_monitor.py

백그라운드 실행
nohup python3 plant_monitor.py > plant.log 2>&1 &
tail -f plant.log

🔁 부팅 시 자동 실행 (systemd)
[Unit]
Description=Plant Monitoring Service
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=dli
WorkingDirectory=/home/dli
ExecStart=/usr/bin/python3 /home/dli/plant_monitor.py
Restart=always

[Install]
WantedBy=multi-user.target

sudo systemctl daemon-reload
sudo systemctl enable plant-monitor
sudo systemctl start plant-monitor

✅ 구현 결과

하루 3회 자동 촬영

촬영 즉시 이메일 전송

재부팅 후 자동 실행 유지

Jetson Nano 단독 운용

AI / GPT / 외부 API 미사용

❗ 자주 발생하는 오류 & 해결
OpenCV 오류
sudo apt install -y python3-opencv

카메라 인식 실패
v4l2-ctl --list-devices
ls /dev/video*

Gmail 로그인 실패

Gmail 2단계 인증 활성화

앱 비밀번호(16자리) 사용 필수
