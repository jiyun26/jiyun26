# IFTTT 모바일 앱으로 알림 받기

## 1. 과제 목적
ThingSpeak의 채널 데이터가 설정한 조건을 만족하면 IFTTT Webhooks를 통해 휴대폰으로 알림을 전송하는 시스템을 구현하였습니다.

## 2. 시스템 구성
- **ThingSpeak** : 데이터 저장 및 조건 감지 (React)
- **ThingHTTP** : IFTTT Webhooks 호출
- **IFTTT** : 조건 충족 시 모바일 알림 전송

① ThingSpeak 채널 데이터 기록
센서값 또는 API를 통해 Field 값 저장

② ThingSpeak React
특정 조건이 충족되었는지 확인

③ ThingHTTP → IFTTT WebHooks
조건 충족 시 IFTTT에 이벤트 전달

④ IFTTT Notification
휴대폰에 알림 표시


## 3. 구현 과정

### ① IFTTT Applet 생성
Webhooks의 `Receive a web request`를 Trigger로 설정하고, Notifications를 Action으로 연결하여 모바일 알림을 구성하였습니다.

<img width="1226" height="750" alt="image" src="https://github.com/user-attachments/assets/35ec2845-69c2-414e-8027-1b6115da0352" />


### ② ThingHTTP 설정
IFTTT Webhooks URL을 ThingHTTP에 등록하여 ThingSpeak에서 IFTTT를 호출할 수 있도록 설정하였습니다.

<img width="1064" height="596" alt="image" src="https://github.com/user-attachments/assets/3b607673-01f0-45a1-b158-d942361e50c8" />


### ③ React 설정
채널의 Field 1 값이 30을 초과할 경우 ThingHTTP가 실행되도록 React를 설정하였습니다.

<img width="1360" height="598" alt="image" src="https://github.com/user-attachments/assets/615e6598-3b04-447a-a505-ed5a64be3797" />


### ④ 채널 데이터 입력 및 테스트
API Request를 통해 Field 1에 35,80,42 등 30을 초과하는 숫자를 입력하여 조건을 충족시켰으며, 휴대폰으로 알림이 전송되는 것을 확인하였습니다.

<img width="1310" height="922" alt="image" src="https://github.com/user-attachments/assets/b6f2f028-604d-43f2-a6b7-4d38a4fd0d8a" />
<img width="1206" height="501" alt="image" src="https://github.com/user-attachments/assets/63caac18-7faa-4611-a190-290c89408085" />


## 4. 결과
ThingSpeak에 입력된 데이터가 설정한 조건을 만족하면 ThingHTTP를 통해 IFTTT가 실행되고, 휴대폰으로 알림이 전송되는 것을 확인하였습니다.
<img width="1542" height="1140" alt="image" src="https://github.com/user-attachments/assets/616432b4-e298-4737-84a1-7e2d36757f43" />
<img width="1206" height="608" alt="image" src="https://github.com/user-attachments/assets/87ad63cd-8640-4fe5-ab66-cf2eca59ac58" />



