

```C++
#include <SPI.h>
#include <MFRC522.h>

#define SS_PIN    5
#define RST_PIN   4
#define RELAY_PIN 27
#define BEEP_PIN  26

MFRC522 mfrc522(SS_PIN, RST_PIN);

//合法卡片UID E9 81 F8 39
byte allowedUid[] = {0xE9,0x81,0xF8,0x39};

bool checkUid(byte *uid){
  for(int i = 0; i < 4; i++){
    if(uid[i] != allowedUid[i]) 
      return false;
  }
  return true;
}

void setup() {
  Serial.begin(115200);
  SPI.begin();
  mfrc522.PCD_Init();

  pinMode(RELAY_PIN, OUTPUT);
  digitalWrite(RELAY_PIN, LOW);

  Serial.println("=== RC522门禁就绪 ===");
}

void loop() {
  if ( ! mfrc522.PICC_IsNewCardPresent()) return;
  if ( ! mfrc522.PICC_ReadCardSerial())  return;

  //打印卡号
  Serial.print("刷卡UID：");
  for (byte i = 0; i < mfrc522.uid.size; i++) {
    Serial.print(mfrc522.uid.uidByte[i], HEX);
    Serial.print(" ");
  }
  Serial.println();

  if(checkUid(mfrc522.uid.uidByte)){
    Serial.println("✅合法卡，开门");
    tone(BEEP_PIN, 2000, 100);   //成功短嘀，2000Hz
    digitalWrite(RELAY_PIN, HIGH);
    delay(800);                   //开门保持时间800ms
    digitalWrite(RELAY_PIN, LOW);
  }else{
    Serial.println("❌非法卡");
    tone(BEEP_PIN, 2000, 400);   //错误长嘀
  }

  mfrc522.PICC_HaltA();
  delay(600);
}

```

