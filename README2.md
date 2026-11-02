#  p5 슬라이더를 이용하여 LED의 밝기 조절하기
웹 브라우저의 슬라이더로 아두이노에 연결된 LED의 밝기를 실시간으로 조절하는 프로젝트입니다.   
웹에서 슬라이더를 움직이면, 시리얼 통신을 통해 아두이노의 LED 밝기가 바뀌는 방식으로 작동합니다.
<br>

## 1. 회로 사진
<img width="2088" height="1356" alt="image" src="https://github.com/user-attachments/assets/3f12927a-b140-45dc-a4b4-218bbdfade85" />
아두이노의 9번 핀을 전원으로 사용하여 저항과 LED를 연결하고, LED의 반대쪽을 GND에 연결하였습니다.

---
## 2. 스케치
### Arduino IDE
p5.js로부터 시리얼 통신으로 전달받은 숫자를 읽어와 analogWrite()를 통해 9번 핀의 PWM 출력을 조절하여 LED 밝기를 바꿉니다.
```javascript
const int ledPin = 9; 

void setup() {
  pinMode(ledPin, OUTPUT);
  Serial.begin(9600); 
}

void loop() {
  if (Serial.available() > 0) {
    int brightness = Serial.parseInt();
    brightness = constrain(brightness, 0, 255);
    analogWrite(ledPin, brightness);
  }
}
```

### p5(index.html)
p5.js 코어 라이브러리를 웹페이지에 불러와 브라우저가 자바스크립트 스케치를 정상적으로 실행할 수 있도록 뼈대를 잡아줍니다.
```javascript
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>Arduino p5.js LED Dimmer</title>
  <!-- p5.js 코어 라이브러리 -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.6.0/p5.js"></script>
  <link rel="stylesheet" type="text/css" href="style.css">
</head>
<body>
  <script src="sketch.js"></script>
</body>
</html>
```
### p5(sketch.js)
화면에 연결 버튼과 슬라이더를 만들고, 슬라이더 값이 바뀔 때마다 웹 브라우저의 Web Serial API를 통해 아두이노로 실시간 값을 전송합니다.
```javascript
let portButton;
let brightnessSlider;
let port = null;
let writer = null;

function setup() {
  createCanvas(400, 300);
  background(220);

  // 1. 아두이노 연결 버튼 생성
  portButton = createButton('아두이노 연결');
  portButton.position(20, 20);
  portButton.mousePressed(openPort);

  // 2. LED 밝기 조절 슬라이더 생성 (0 ~ 255)
  createP('LED 밝기 조절').position(20, 50);
  brightnessSlider = createSlider(0, 255, 0);
  brightnessSlider.position(20, 90);
  brightnessSlider.style('width', '200px');
}

function draw() {
  background(240);
  
  // 현재 슬라이더 값 화면에 표시
  textSize(16);
  fill(0);
  text(`현재 밝기 값: ${brightnessSlider.value()}`, 20, 140);

  // 포트가 정상 연결되어 있으면 슬라이더 값을 아두이노로 전송
  if (writer) {
    sendData(brightnessSlider.value());
  }
}

// 비동기 방식으로 아두이노에 데이터 전송
async function sendData(val) {
  try {
    const dataString = val + '\n';
    await writer.write(new TextEncoder().encode(dataString));
  } catch (err) {
    console.error('데이터 전송 실패:', err);
  }
}

// 연결 버튼 클릭 시 시리얼 포트 요청 및 오픈
async function openPort() {
  if (!navigator.serial) {
    alert('이 브라우저는 Web Serial을 지원하지 않습니다. Chrome을 사용해주세요.');
    return;
  }

  try {
    // 사용자가 아두이노 포트 선택
    port = await navigator.serial.requestPort();
    // 9600 보레이트로 연결 오픈
    await port.open({ baudRate: 9600 });

    writer = port.writable.getWriter();
    
    portButton.html('연결됨');
    portButton.attribute('disabled', '');
  } catch (err) {
    console.error('포트 연결 에러:', err);
    alert('포트 연결에 실패했습니다. 아두이노 연결 상태를 확인해주세요.');
  }
}
```

<img width="870" height="456" alt="image" src="https://github.com/user-attachments/assets/403ccc16-a27c-4ee9-aa6a-f37642f54dcf" />

---
## 3. 작동 영상

https://github.com/user-attachments/assets/f0797224-3e70-4f49-9c9a-dad824b4ddb0



