# 🌱 Jetson Nano + Arduino 스마트팜 모니터링 시스템

Jetson Nano와 Arduino를 연동하여  
**온·습도 측정(DHT11) + 식물 사진 자동 촬영 + 이메일 전송**을 수행하는  
임베디드 기반 스마트팜 모니터링 프로젝트입니다.

AI 및 GPT 기능 없이,  
**센서·카메라·리눅스 자동화(systemd)**에 초점을 둔 실사용 가능한 시스템입니다.

---

## 📌 프로젝트 개요

- Arduino + DHT11 센서를 이용한 온·습도 측정
- Jetson Nano에서 카메라로 식물 사진 촬영
- 정해진 시간에 자동 실행 (schedule)
- 촬영 즉시 Gmail로 사진 전송
- systemd를 통한 부팅 시 자동 실행

---

## 🧩 시스템 구성

### 하드웨어
- Jetson Nano (JetPack 4.6)
- Arduino (UNO / UNO R4 등)
- DHT11 온습도 센서
- USB 또는 CSI 카메라

### 소프트웨어
- Ubuntu (JetPack 4.6)
- Python 3
- OpenCV (apt 기반)
- schedule
- smtplib (Python 내장)
- systemd

---

## 🔌 Arduino 코드 (DHT11)

```cpp
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
2초마다 온도,습도 형식으로 시리얼 출력

⚙️ Jetson Nano 설치 가이드 (JetPack 4.6)
1️⃣ 시스템 업데이트
bash
코드 복사
sudo apt update
sudo apt upgrade -y
2️⃣ 필수 패키지 설치
bash
코드 복사
sudo apt install -y python3-pip python3-opencv v4l-utils
3️⃣ Python 라이브러리 설치
bash
코드 복사
pip3 install schedule
4️⃣ 카메라 권한 설정
bash
코드 복사
sudo usermod -a -G video $USER
sudo reboot
🛠 전체 설치 스크립트 (권장)
bash
코드 복사
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
python
코드 복사
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
        print("❌ 촬영 실패")
        return

    filename = datetime.now().strftime("%Y%m%d_%H%M%S.jpg")
    cv2.imwrite(filename, frame)

    msg = MIMEMultipart()
    msg["From"] = EMAIL
    msg["To"] = EMAIL
    msg["Subject"] = "🌱 식물 자동 촬영 사진"
    msg.attach(MIMEText("자동 촬영된 식물 사진입니다.", "plain"))

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

    print("✅ 이메일 전송 완료")

if __name__ == "__main__":
    for t in TIMES:
        schedule.every().day.at(t).do(take_and_send)

    while True:
        schedule.run_pending()
        time.sleep(60)
▶️ 실행 방법
즉시 실행
bash
코드 복사
python3 plant_monitor.py
백그라운드 실행
bash
코드 복사
nohup python3 plant_monitor.py > plant.log 2>&1 &
로그 확인:

bash
코드 복사
tail -f plant.log
🔁 부팅 시 자동 실행 (systemd)
bash
코드 복사
sudo nano /etc/systemd/system/plant-monitor.service
ini
코드 복사
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
bash
코드 복사
sudo systemctl daemon-reload
sudo systemctl start plant-monitor
sudo systemctl enable plant-monitor
🧪 디버깅 체크 명령어
bash
코드 복사
python3 -c "import cv2"
python3 -c "import schedule"
v4l2-ctl --list-devices
ls /dev/video*
tail -f plant.log
✅ 최종 기능 요약
하루 3회 자동 촬영

촬영 즉시 이메일 전송

재부팅 후 자동 실행

Jetson Nano 단독 운용

AI / GPT / 외부 API 미사용

📎 참고
Jetson Nano JetPack 4.6 환경 기준

OpenCV는 apt 기반 설치 권장

Gmail은 앱 비밀번호 필수
