# 🌡️ Jetson Nano + Arduino 온습도 창반

## 📌 개요

Jetson Nano와 Arduino를 연결하여 DHT11 온습도 센서의 값을 실시간으로 읽고, 사용자의 질문에 Gradio 창반 UI로 응답하는 프로젝트입니다.

---

## 📁 프로젝트 구조

- `Jetson Nano`에서 `Arduino`와 연결
- `DHT11` 센서를 통해 온도와 습도 수집
- `Function Calling` 기반 GPT 연동
- `Gradio` 기반 GUI 창반

---

## 🧰 사전 준비

### 소프트웨어

- Arduino IDE: [버전 1.8.19](https://www.arduino.cc/en/software) 또는 2.3.2
- Python 3.8
- Jupyter Notebook
- Gradio
- OpenAI API Key

### 하드웨어

- Jetson Nano
- Arduino (UNO 등)
- DHT11 센서

---

## 🛠️ 설치 가이드

### 1. Arduino 설치 (Jetson Nano 기준)

```bash
sudo apt update
sudo apt install openjdk-8-jdk
wget https://downloads.arduino.cc/arduino-1.8.19-linuxaarch64.tar.xz
tar -xf arduino-1.8.19-linuxaarch64.tar.xz
cd arduino-1.8.19
sudo ./install.sh
sudo usermod -aG dialout $USER
newgrp dialout
```

### 2. Python 3.8 설치 및 가상환경 구성

```bash
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
```

---

## 🤖 아들이노 코드 예제

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
```

---

## 🤖 창반 실행 코드 (Python)

`DHT11Assistant` 클래스를 기반으로 온습도 창반을 실행합니다.

```bash
python chatbot_dht11.py
```

> Gradio 인터페이스가 실행되며, "지금 온도는?", "습도는 얼마야?" 같은 질문을 할 수 있습니다.

---

## 💡 Function Calling 예제

```json
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
```

> 온도/습도 데이터를 필요할 때 함수 형태로 호출하며, GPT는 호출 결과를 활용해 자연어로 응답합니다.

---

## ☁️ 날씨 API 창반 예제

- API: [OpenWeatherMap](https://openweathermap.org/)
- API Key 필요
- 지역 입력 시 현재 날씨, 온도, 습도 반환

---

## 🔗 관련 링크

- [Jetson & Arduino 연결 프로젝트 GitHub](https://github.com/ralralra/jetson_DLI)
- [Function Calling 설명](https://platform.openai.com/docs/guides/function-calling)
- [OpenWeatherMap API 한국어 설명 블로그](https://icedhotchoco.tistory.com/entry/OpenWeatherMap-날씨-API)

---

## ⚠️ 주의 사항

- `Jetson.GPIO` 패키지가 가상환경에서 동작하지 않을 경우 `/usr/lib/...` 디렉토리의 패키지를 수동으로 복사해야 합니다.
- `/dev/ttyACM0` 호시가 필요한 경우 권한 설정:

```bash
sudo chmod 666 /dev/ttyACM0
```

---

