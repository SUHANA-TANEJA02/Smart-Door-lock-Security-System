#include <Keypad.h>
#include <LiquidCrystal.h>
#include <Servo.h>
#include <SoftwareSerial.h>

// LCD Pins
LiquidCrystal lcd(7, 6, 5, 4, 3, 2);

// Servo
Servo doorServo;

// GSM
SoftwareSerial gsm(10, 11); // RX, TX

// Buzzer
int buzzer = 8;

// Password
String correctPassword = "1234";
String enteredPassword = "";

// Keypad Setup
const byte ROWS = 4;
const byte COLS = 4;

char keys[ROWS][COLS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};

byte rowPins[ROWS] = {22, 24, 26, 28};
byte colPins[COLS] = {30, 32, 34, 36};

Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);

void setup() {
  lcd.begin(16, 2);
  doorServo.attach(9);
  doorServo.write(0);  // Door Locked
  pinMode(buzzer, OUTPUT);

  gsm.begin(9600);
  Serial.begin(9600);

  lcd.print("Door Locked");
}

void loop() {
  char key = keypad.getKey();

  if (key) {
    if (key == '#') {
      checkPassword();
    } 
    else if (key == '*') {
      enteredPassword = "";
      lcd.clear();
      lcd.print("Cleared");
      delay(1000);
      lcd.clear();
      lcd.print("Enter PIN:");
    } 
    else {
      enteredPassword += key;
      lcd.setCursor(enteredPassword.length() - 1, 1);
      lcd.print("*");
    }
  }
}

void checkPassword() {
  lcd.clear();

  if (enteredPassword == correctPassword) {
    lcd.print("Door opened");
    openDoor();
  } 
  else {
    lcd.print("Door closed");
    wrongAttempt();
  }

  enteredPassword = "";
  delay(2000);
  lcd.clear();
  lcd.print("Enter PIN:");
}

void openDoor() {
  doorServo.write(90);   // Door Open
  delay(3000);
  doorServo.write(0);    // Door Close
}

void wrongAttempt() {
  digitalWrite(buzzer, HIGH);
  sendSMS();
  delay(2000);
  digitalWrite(buzzer, LOW);
}

void sendSMS() {
  gsm.println("AT+CMGF=1");
  delay(1000);
  gsm.println("AT+CMGS=\"+91XXXXXXXXXX\""); // Replace number
  delay(1000);
  gsm.println("Alert! Incorrect Password Entered.");
  delay(1000);
  gsm.write(26); // CTRL+Z
}
