# 동작중임을 나타내는 캐릭터 모션 만들기
LCD 첫 번째 칸의 픽셀을 단계적으로 채우고 비우는 방식을 통해,  
동작중임을 심플하게 나타내려 하였습니다.
<br><br>

## 1. 회로 

<img width="1508" height="750" alt="image" src="https://github.com/user-attachments/assets/07b3dbd2-3300-4d1c-a0ca-2a29a245a3b3" />

<br> <br> 

## 2. 코드
```
#include <Wire.h>                
#include <LiquidCrystal_I2C.h>    

LiquidCrystal_I2C lcd(0x27, 16, 2); // LCD 주소 0x27, 16칸 2줄 설정

// LCD에 표시할 8가지 모양 정의 
byte c0[] = {0x1F, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x1F};
byte c1[] = {0x1F, 0x1F, 0x00, 0x00, 0x00, 0x00, 0x1F, 0x1F};
byte c2[] = {0x1F, 0x1F, 0x1F, 0x00, 0x00, 0x1F, 0x1F, 0x1F};
byte c3[] = {0x1F, 0x1F, 0x1F, 0x1F, 0x1F, 0x1F, 0x1F, 0x1F};
byte c4[] = {0x00, 0x1F, 0x1F, 0x1F, 0x1F, 0x1F, 0x1F, 0x00};
byte c5[] = {0x00, 0x00, 0x1F, 0x1F, 0x1F, 0x1F, 0x00, 0x00};
byte c6[] = {0x00, 0x00, 0x00, 0x1F, 0x1F, 0x00, 0x00, 0x00};
byte c7[] = {0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00};

// 정의한 모양들을 배열에 저장
byte* frames[] = {c0, c1, c2, c3, c4, c5, c6, c7};

void setup() {
  lcd.init();       // LCD 초기화
  lcd.backlight();  // LCD 백라이트 켜기
}

void loop() {
  // 0번부터 7번까지 모양을 차례대로 표시
  for (int i = 0; i < 8; i++) {
    lcd.createChar(0, frames[i]); // 0번 문자에 새로운 모양 저장
    lcd.setCursor(0, 0);          // LCD 첫 번째 줄, 첫 번째 칸 선택
    lcd.write(0);                 // 저장된 모양 출력
    delay(500);                   // 0.5초 대기
  }
}
```

<br><br>
I2C LCD 화면의 픽셀을 이용해 8가지 모양을 0.5초마다 순서대로 교체 출력하도록 작성하였습니다. 이를 통해 LCD의 첫 번째 칸에서 모양이 계속 바뀌는 간단한 애니메이션 효과를 연출하였습니다.
<br><br>
## 3. 영상

https://github.com/user-attachments/assets/9e55bdc3-b9e6-4af7-9849-b4b5d004c6b1

<br> <br> 

