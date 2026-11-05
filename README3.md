# 손가락으로 LED 밝기 제어하기
웹캠을 통해 사용자의 손동작을 인식하고, 엄지와 검지 사이의 거리에 따라 LED의 밝기를 조절하는 프로그램입니다. HandPose 모델을 이용하여 손의 관절과 움직임을 실시간으로 인식하며, 손가락 사이의 거리를 계산하여 p5.js의 슬라이더 값을 변경합니다. 이후 변경된 밝기 값을 아두이노로 전송하여 LED의 밝기를 조절하도록 작동합니다.
<br>

## 1. 회로
<img width="1614" height="1180" alt="image" src="https://github.com/user-attachments/assets/28e36cbd-6513-4b58-9c2e-26724bef1642" />
<br>

## 2. 스케치
<br>
웹캠 영상 → ml5 handPose로 손 관절 추출 → 중지를 접었는지 확인 → 엄지·검지 사이 거리 계산 → 손바닥 길이로 정규화하여 0~100으로 변환 → 0.05 비율로 부드럽게 보정 → 슬라이더 값 갱신 → map()으로 0~255 PWM 값 변환 → 연결 상태와 값 변경 여부 확인 → Web Serial로 아두이노에 전송

### IDE
```javascript
const int ledPin = 9;
int brightness = 0;

void setup() {
  Serial.begin(9600);
  pinMode(ledPin, OUTPUT);
}

void loop() {
  if (Serial.available() > 0) {

    String data = Serial.readStringUntil('\n');

    if (data.startsWith("PWM:")) {

      brightness = data.substring(4).toInt();

      brightness = constrain(brightness, 0, 255);

      analogWrite(ledPin, brightness);
    }
  }
}
```

### sketch.js
```javascript
let handPose;
let video;
let slider;
let hands = [];
let connections;

let sliderValue = 50;

// 웹캠 및 모델 상태
let cameraReady = false;
let modelReady = false;

// 시리얼 통신
let port;
let writer;
let connectBtn;
let lastPwmVal = -1;

function setup() {
  createCanvas(640, 480);

  // 아두이노 연결 버튼
  connectBtn = createButton('아두이노 연결');
  connectBtn.position(10, 490);
  connectBtn.mousePressed(connectSerial);

  // 슬라이더 생성
  slider = createSlider(0, 100, 50, 1);
  slider.position(10, 20);
  slider.style('width', '150px');
  slider.input(updateSliderValue);

  // 웹캠 실행
  video = createCapture(VIDEO, () => {
    cameraReady = true;
    console.log("웹캠 준비 완료");
    loadHandPose();
  });

  video.size(640, 480);
  video.hide();
}

function draw() {
  background(220);

  // 웹캠 출력
  if (cameraReady) {
    image(video, 0, 0, width, height);
  } else {
    fill(0);
    textSize(20);
    textAlign(CENTER);
    text("Camera Loading...", width / 2, height / 2);
  }

  // 손 인식
  if (modelReady && hands.length > 0) {
    drawSkeletons(hands);
    drawKeyPoints(hands);

    let adjustMode = isGrabMode(hands);

    if (adjustMode) {
      drawConnectedLine(hands);

      let currentValue = Number(slider.value());
      let distanceValue = getNormalizedDistance(hands);
      let nextValue = currentValue +
        0.05 * (distanceValue - currentValue);

      nextValue = constrain(nextValue, 0, 100);

      slider.value(nextValue);
      sliderValue = nextValue;
    }
  }

  // 아두이노로 밝기 값 전송
  sendPwmToArduino(sliderValue);

  // 상태 표시
  fill(255);
  noStroke();
  textSize(16);
  textAlign(LEFT);

  text("LED 밝기: " + round(sliderValue), 200, 30);

  if (!modelReady) {
    text("HandPose Loading...", 200, 55);
  }
}

// HandPose 모델 로딩
async function loadHandPose() {
  try {
    console.log("HandPose 모델 로딩 시작");

    handPose = ml5.handPose({
      flipped: false
    });

    await handPose.ready;

    connections = handPose.getConnections();

    console.log("손 인식 모델 준비 완료");

    handPose.detectStart(video, gotHands);

    modelReady = true;

  } catch (error) {
    console.error("손 인식 모델 오류:", error);
  }
}

// 아두이노 연결
async function connectSerial() {
  try {
    port = await navigator.serial.requestPort();

    await port.open({ baudRate: 9600 });

    const encoder = new TextEncoderStream();

    encoder.readable.pipeTo(port.writable).catch(console.error);

    writer = encoder.writable.getWriter();

    connectBtn.html("아두이노 연결됨");

    console.log("아두이노 연결 성공!");

  } catch (error) {
    console.error("시리얼 연결 실패:", error);
  }
}

// PWM 값 전송
async function sendPwmToArduino(value) {
  if (!writer) return;

  let pwmValue = floor(map(value, 0, 100, 0, 255));
  pwmValue = constrain(pwmValue, 0, 255);

  if (pwmValue !== lastPwmVal) {
    lastPwmVal = pwmValue;

    try {
      await writer.write("PWM:" + pwmValue + "\n");
    } catch (error) {
      console.error("전송 오류:", error);
    }
  }
}

// 손 인식 결과 저장
function gotHands(results) {
  hands = results;
}

// 손 관절 표시
function drawKeyPoints(hs) {
  for (let hand of hs) {
    for (let keypoint of hand.keypoints) {
      fill(255, 0, 0);
      noStroke();
      circle(keypoint.x, keypoint.y, 10);
    }
  }
}

// 손 뼈대 표시
function drawSkeletons(hs) {
  if (!connections) return;

  for (let hand of hs) {
    stroke(0, 255, 0);
    strokeWeight(2);

    for (const [a, b] of connections) {
      let start = hand.keypoints[a];
      let end = hand.keypoints[b];

      if (start && end) {
        line(start.x, start.y, end.x, end.y);
      }
    }
  }
}

// 손가락 조절 모드 확인
function isGrabMode(hs) {
  if (!hs || hs.length === 0) return false;

  let ks = hs[0].keypoints;

  if (ks.length !== 21) return false;

  let palmLength = dist(
    ks[0].x, ks[0].y,
    ks[9].x, ks[9].y
  );

  let grabDistance = dist(
    ks[0].x, ks[0].y,
    ks[12].x, ks[12].y
  );

  return grabDistance <= palmLength;
}

// 엄지와 검지 연결선
function drawConnectedLine(hs) {
  let hand = hs[0];

  let a = hand.keypoints[4];
  let b = hand.keypoints[8];

  stroke(255, 140, 0);
  strokeWeight(4);

  line(a.x, a.y, b.x, b.y);

  noFill();
  circle(a.x, a.y, 22);
  circle(b.x, b.y, 22);
}

// 엄지와 검지 사이 거리 계산
function getNormalizedDistance(hs) {
  let ks = hs[0].keypoints;

  let a = ks[4];
  let b = ks[8];
  let c = ks[0];
  let d = ks[9];

  let fingerDistance = dist(
    a.x, a.y, b.x, b.y
  );

  let palmDistance = dist(
    c.x, c.y, d.x, d.y
  );

  let value = round(
    100 * (fingerDistance / palmDistance)
  ) - 15;

  return constrain(value, 0, 100);
}

// 슬라이더 직접 조절
function updateSliderValue() {
  sliderValue = Number(slider.value());
}
```
### index.html
```javascript
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <title>LED Brightness Control</title>

  <!-- p5.js -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.9.0/p5.min.js"></script>

  <!-- ml5.js -->
  <script src="https://unpkg.com/ml5@1/dist/ml5.min.js"></script>

</head>

<body>

  <script src="sketch.js"></script>

</body>
</html>
```

## 3. 작동 영상

https://github.com/user-attachments/assets/0b66d79a-b4fb-410c-a635-0729744cdf94




