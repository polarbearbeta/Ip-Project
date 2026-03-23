#include <DFRobotDFPlayerMini.h>
#include <Servo.h>

//--------Define Mini Player--------//
DFRobotDFPlayerMini myDFPlayer;

//--------Define TCS3200 Color Sensor--------//
const int S0 = 4;
const int S1 = 5;
const int S2 = 6;
const int S3 = 7;
const int sensorOut = 8;

//--------Calibration Light--------//
const int CALIBRATION_LED = 37;

int redFrequency = 0;
int greenFrequency = 0;
int blueFrequency = 0;

//--------Servo--------//
Servo servo1; //MOUTH
Servo servo2; //Esophagus
Servo servo3; //Stomach
Servo servo4; //SI
Servo servo5; //Rectum

const int SERVO1_PIN = 9; //epiglothis
const int SERVO2_PIN = 10; // Stomach
const int SERVO3_PIN = 11; // Stomach-duo
const int SERVO4_PIN = 12; // SI-LI
const int SERVO5_PIN = 13; //RECTUM

//--------LED--------//
const int LED1 = 22; //mouth
const int LED2 = 23; //esophagus
const int LED3 = 24; //liver
const int LED4 = 25; //stomach
const int LED5 = 26; //gallblader
const int LED6 = 27; //duodenum
const int LED7 = 28; //SI
const int LED8 = 29; //Pancrease
const int LED9 = 30; //LI

//--------State--------//
int lastDetected = 0; // 0 = none , 1 = carb, 2 = fat, 3 = protein

//--------Define Function--------//
void readColor();
int detectColor();
int classifyWithVoting();
void carb();
void fat();
void protein();

void setup() {
  Serial.begin(115200);
  Serial1.begin(9600); //DFPLAYER pins 18TX/19RX

  //--------Color sensor setup--------//
  pinMode(S0, OUTPUT);
  pinMode(S1, OUTPUT);
  pinMode(S2, OUTPUT);
  pinMode(S3, OUTPUT);
  pinMode(sensorOut, INPUT);

  // Frequency scaling = 20%
  digitalWrite(S0, HIGH);
  digitalWrite(S1, LOW);

  //--------Calibration LED--------//
  pinMode(CALIBRATION_LED, OUTPUT);
  digitalWrite(CALIBRATION_LED, LOW);

  //--------Servo Setup--------//
  servo1.attach(SERVO1_PIN);
  servo2.attach(SERVO2_PIN);
  servo3.attach(SERVO3_PIN);
  servo4.attach(SERVO4_PIN);
  servo5.attach(SERVO5_PIN);

  servo1.write(160);
  delay(100);
  servo2.write(180);
  delay(100);
  servo3.write(180);
  delay(100);
  servo4.write(160);
  delay(100);
  servo5.write(180);

  //--------LED setup--------//
  pinMode(LED1, OUTPUT);
  pinMode(LED2, OUTPUT);
  pinMode(LED3, OUTPUT);
  pinMode(LED4, OUTPUT);
  pinMode(LED5, OUTPUT);
  pinMode(LED6, OUTPUT);
  pinMode(LED7, OUTPUT);
  pinMode(LED8, OUTPUT);
  pinMode(LED9, OUTPUT);

  digitalWrite(LED1, LOW);
  digitalWrite(LED2, LOW);
  digitalWrite(LED3, LOW);
  digitalWrite(LED4, LOW);
  digitalWrite(LED5, LOW);
  digitalWrite(LED6, LOW);
  digitalWrite(LED7, LOW);
  digitalWrite(LED8, LOW);
  digitalWrite(LED9, LOW);

  //--------DFPlayer setup--------//
  Serial.println("Starting DFPlayer Mini...");

  if (!myDFPlayer.begin(Serial1)) {
    Serial.println("DFPlayer Mini not detected.");
    Serial.println("Check wiring, power, and SD card.");
    while (true) {
    }
  }

  Serial.println("DFPlayer Mini online.");
  myDFPlayer.volume(30);
}

void loop() {
  lastDetected = classifyWithVoting();

  if (lastDetected == 1) {
    carb();
    lastDetected = 0;
  }
  else if (lastDetected == 2) {
    fat();
    lastDetected = 0;
  }
  else if (lastDetected == 3) {
    protein();
    lastDetected = 0;
  }

  delay(500);
}

//--------Read Color--------//
void readColor() {
  // Read RED RAW
  digitalWrite(S2, LOW);
  digitalWrite(S3, LOW);
  redFrequency = pulseIn(sensorOut, LOW);

  // Read GREEN RAW
  digitalWrite(S2, HIGH);
  digitalWrite(S3, HIGH);
  greenFrequency = pulseIn(sensorOut, LOW);

  // Read BLUE RAW
  digitalWrite(S2, LOW);
  digitalWrite(S3, HIGH);
  blueFrequency = pulseIn(sensorOut, LOW);
}

//--------Single Detection Logic--------//
int detectColor() {

  // -------- YELLOW = fat --------
  // B highest, G middle, R lowest
  if (blueFrequency > greenFrequency &&
      greenFrequency > redFrequency &&
      redFrequency >= 20 && redFrequency <= 30 &&
      greenFrequency >= 34 && greenFrequency <= 42 &&
      blueFrequency >= 44 && blueFrequency <= 55) {

    return 2;
  }

  // -------- RED = protein --------
  // G ≈ B and both much larger than R
  else if (greenFrequency >= 60 && greenFrequency <= 70 &&
           blueFrequency >= 58 && blueFrequency <= 68 &&
           redFrequency >= 30 && redFrequency <= 40 &&
           abs(greenFrequency - blueFrequency) <= 5 &&
           (greenFrequency - redFrequency > 20)) {

    return 3;
  }

  // -------- BLUE = carb --------
  // G highest or equal, R mid, B lowest
  else if (greenFrequency >= 45 && greenFrequency <= 52 &&
           redFrequency >= 40 && redFrequency <= 50 &&
           blueFrequency >= 38 && blueFrequency <= 45 &&
           greenFrequency >= redFrequency &&
           redFrequency >= blueFrequency) {

    return 1;
  }

  else {
    return 0;
  }
}
//--------Voting System--------//
int classifyWithVoting() {
  int countNone = 0;
  int countBlue = 0;
  int countYellow = 0;
  int countRed = 0;

  digitalWrite(CALIBRATION_LED, HIGH);

  for (int i = 0; i < 10; i++) {
    readColor();
    int detected = detectColor();

    Serial.print("Sample ");
    Serial.print(i + 1);
    Serial.print(" -> R: ");
    Serial.print(redFrequency);
    Serial.print("  G: ");
    Serial.print(greenFrequency);
    Serial.print("  B: ");
    Serial.print(blueFrequency);
    Serial.print("  |  ");

    if (detected == 1) {
      Serial.println("BLUE / CARB");
      countBlue++;
    }
    else if (detected == 2) {
      Serial.println("YELLOW / FAT");
      countYellow++;
    }
    else if (detected == 3) {
      Serial.println("RED / PROTEIN");
      countRed++;
    }
    else {
      Serial.println("NONE");
      countNone++;
    }

    delay(50);
  }

  digitalWrite(CALIBRATION_LED, LOW);

  Serial.print("Counts -> None: ");
  Serial.print(countNone);
  Serial.print("  Blue: ");
  Serial.print(countBlue);
  Serial.print("  Yellow: ");
  Serial.print(countYellow);
  Serial.print("  Red: ");
  Serial.println(countRed);

  int maxCount = countNone;
  int finalResult = 0;

  if (countBlue > maxCount) {
    maxCount = countBlue;
    finalResult = 1;
  }

  if (countYellow > maxCount) {
    maxCount = countYellow;
    finalResult = 2;
  }

  if (countRed > maxCount) {
    maxCount = countRed;
    finalResult = 3;
  }

  Serial.print("Final voted result: ");
  if (finalResult == 1) {
    Serial.println("CARB");
  }
  else if (finalResult == 2) {
    Serial.println("FAT");
  }
  else if (finalResult == 3) {
    Serial.println("PROTEIN");
  }
  else {
    Serial.println("NONE");
  }

  return finalResult;
}

//--------Carb--------//
void carb() {
  Serial.println("CARB detected");
  digitalWrite(CALIBRATION_LED,LOW);
  //
  myDFPlayer.play(1);
  digitalWrite(LED1, HIGH);
  delay(55000);
  servo1.write(90);
  digitalWrite(LED1,LOW);
  delay(500);
  servo1.write(160);
  //
  myDFPlayer.play(2);
  digitalWrite(LED2, HIGH);
  delay(55000);
  servo2.write(90);
  digitalWrite(LED2,LOW);
  delay(500);
  servo2.write(180);
  //
  myDFPlayer.play(3);
  digitalWrite(LED4, HIGH);
  delay(116000);
  servo3.write(90);
  digitalWrite(LED4, LOW);
  delay(500);
  servo3.write(180);
  //
  myDFPlayer.play(4);
  digitalWrite(LED3, HIGH);
  digitalWrite(LED5, HIGH);
  digitalWrite(LED6, HIGH);
  digitalWrite(LED7, HIGH);
  digitalWrite(LED8, HIGH);
  delay(95000);
  servo4.write(90);
  digitalWrite(LED3, LOW);
  digitalWrite(LED5, LOW);
  digitalWrite(LED6, LOW);
  digitalWrite(LED7, LOW);
  digitalWrite(LED8, LOW);
  delay(500);
  servo4.write(160);
  //
  myDFPlayer.play(5);
  digitalWrite(LED9, HIGH);
  delay(87000);
  digitalWrite(LED9,LOW);
  delay(500);
  //
  myDFPlayer.play(6);
  delay(35000);
  servo5.write(90);
  delay(500);
  servo5.write(180);
  //
  myDFPlayer.play(7);
  delay(25000);
  digitalWrite(CALIBRATION_LED,HIGH);
}


//--------Fat--------//
void fat() {
  Serial.println("FAT detected");
  digitalWrite(CALIBRATION_LED,LOW);
  //
  myDFPlayer.play(8);
  digitalWrite(LED1, HIGH);
  delay(119000);
  servo1.write(90);
  digitalWrite(LED1,LOW);
  delay(500);
  servo1.write(160);
  //
  myDFPlayer.play(9);
  digitalWrite(LED2, HIGH);
  delay(55000);
  servo2.write(90);
  digitalWrite(LED2,LOW);
  delay(500);
  servo2.write(180);
  //
  myDFPlayer.play(10);
  digitalWrite(LED4, HIGH);
  delay(79000);
  servo3.write(90);
  digitalWrite(LED4, LOW);
  delay(500);
  servo3.write(180);
  //
  myDFPlayer.play(11);
  digitalWrite(LED3, HIGH);
  digitalWrite(LED5, HIGH);
  digitalWrite(LED6, HIGH);
  digitalWrite(LED7, HIGH);
  digitalWrite(LED8, HIGH);
  delay(185000);
  servo4.write(90);
  digitalWrite(LED3, LOW);
  digitalWrite(LED5, LOW);
  digitalWrite(LED6, LOW);
  digitalWrite(LED7, LOW);
  digitalWrite(LED8, LOW);
  delay(500);
  servo4.write(160);
  //
  myDFPlayer.play(12);
  digitalWrite(LED9, HIGH);
  delay(38000);
  digitalWrite(LED9,LOW);
  delay(500);
  //
  myDFPlayer.play(13);
  delay(35000);
  servo5.write(90);
  delay(500);
  servo5.write(180);
  //
  myDFPlayer.play(14);
  delay(25000);
  digitalWrite(CALIBRATION_LED,HIGH);
}

//--------Protein--------//
void protein() {
  Serial.println("PROTEIN detected");
  digitalWrite(CALIBRATION_LED,LOW);
  //
  myDFPlayer.play(15);
  digitalWrite(LED1, HIGH);
  delay(22000);
  servo1.write(90);
  digitalWrite(LED1,LOW);
  delay(500);
  servo1.write(160);
  //
  myDFPlayer.play(16);
  digitalWrite(LED2, HIGH);
  delay(55000);
  servo2.write(90);
  digitalWrite(LED2,LOW);
  delay(500);
  servo2.write(180);
  //
  myDFPlayer.play(17);
  digitalWrite(LED4, HIGH);
  delay(105000);
  servo3.write(90);
  digitalWrite(LED4, LOW);
  delay(500);
  servo3.write(180);
  //
  myDFPlayer.play(18);
  digitalWrite(LED3, HIGH);
  digitalWrite(LED5, HIGH);
  digitalWrite(LED6, HIGH);
  digitalWrite(LED7, HIGH);
  digitalWrite(LED8, HIGH);
  delay(137000);
  servo4.write(90);
  digitalWrite(LED3, LOW);
  digitalWrite(LED5, LOW);
  digitalWrite(LED6, LOW);
  digitalWrite(LED7, LOW);
  digitalWrite(LED8, LOW);
  delay(500);
  servo4.write(160);
  //
  myDFPlayer.play(19);
  digitalWrite(LED9, HIGH);
  delay(57000);
  digitalWrite(LED9,LOW);
  delay(500);
  //
  myDFPlayer.play(20);
  delay(35000);
  servo5.write(90);
  delay(500);
  servo5.write(180);
  //
  myDFPlayer.play(21);
  delay(25000);
  digitalWrite(CALIBRATION_LED,HIGH);
}
