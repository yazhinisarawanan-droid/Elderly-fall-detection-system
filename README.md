#include <Wire.h>
#include <Adafruit_MPU6050.h>
#include <Adafruit_Sensor.h>
#include <LiquidCrystal_I2C.h>
#include <Keypad.h>
#include <math.h>

// ================================
// MODULES
// ================================

Adafruit_MPU6050 mpu;
LiquidCrystal_I2C lcd(0x27, 16, 2);

// ================================
// PINS
// ================================

const int LED_PIN = 8;
const int BUZZER_PIN = 9;

// ================================
// KEYPAD
// ================================

const byte ROWS = 4;
const byte COLS = 4;

char keys[ROWS][COLS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};

byte rowPins[ROWS] = {2, 3, 4, 6};
byte colPins[COLS] = {10, 11, 12, 13};

Keypad keypad = Keypad(
  makeKeymap(keys),
  rowPins,
  colPins,
  ROWS,
  COLS
);

// ================================
// FALL THRESHOLDS
// ================================

const float ACCELERATION_THRESHOLD = 25.0;
const float TILT_THRESHOLD = 45.0;

// ================================
// SETUP
// ================================

void setup() {

  Serial.begin(9600);
  Wire.begin();

  pinMode(LED_PIN, OUTPUT);
  pinMode(BUZZER_PIN, OUTPUT);

  digitalWrite(LED_PIN, LOW);
  noTone(BUZZER_PIN);

  // LCD
  lcd.init();
  lcd.backlight();

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Fall Detection");
  lcd.setCursor(0, 1);
  lcd.print("Starting...");
  delay(2000);

  // MPU6050
  if (!mpu.begin()) {

    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("MPU ERROR");

    Serial.println("MPU6050 NOT FOUND!");

    while (1) {
      delay(100);
    }
  }

  mpu.setAccelerometerRange(MPU6050_RANGE_8_G);
  mpu.setGyroRange(MPU6050_RANGE_500_DEG);
  mpu.setFilterBandwidth(MPU6050_BAND_21_HZ);

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("System Ready");
  lcd.setCursor(0, 1);
  lcd.print("Monitoring...");

  Serial.println("============================");
  Serial.println("ELDERLY FALL DETECTION");
  Serial.println("============================");
  Serial.println("MPU6050: OK");
  Serial.println("LCD: OK");
  Serial.println("LED: OK");
  Serial.println("BUZZER: OK");
  Serial.println("KEYPAD: OK");
  Serial.println("System Ready");

  delay(2000);
}

// ================================
// MAIN LOOP
// ================================

void loop() {

  sensors_event_t acceleration;
  sensors_event_t gyro;
  sensors_event_t temperature;

  mpu.getEvent(
    &acceleration,
    &gyro,
    &temperature
  );

  float ax = acceleration.acceleration.x;
  float ay = acceleration.acceleration.y;
  float az = acceleration.acceleration.z;

  // Total acceleration
  float totalAcceleration = sqrt(
    ax * ax +
    ay * ay +
    az * az
  );

  // Tilt angle
  float tiltAngle = atan2(
    sqrt(ax * ax + ay * ay),
    az
  ) * 180.0 / PI;

  // Serial Monitor
  Serial.print("Acceleration: ");
  Serial.print(totalAcceleration, 2);

  Serial.print(" m/s2 | Tilt: ");
  Serial.print(tiltAngle, 1);

  Serial.println(" deg");

  // LCD
  lcd.clear();

  lcd.setCursor(0, 0);
  lcd.print("A:");
  lcd.print(totalAcceleration, 1);

  lcd.print(" m/s2");

  lcd.setCursor(0, 1);
  lcd.print("Tilt:");
  lcd.print(tiltAngle, 1);
  lcd.write(byte(223));

  // ================================
  // FALL DETECTION
  // ================================

  if (
    totalAcceleration > ACCELERATION_THRESHOLD &&
    tiltAngle > TILT_THRESHOLD
  ) {

    fallDetected();
  }

  delay(200);
}

// ================================
// FALL DETECTED
// ================================

void fallDetected() {

  Serial.println();
  Serial.println("!!! POSSIBLE FALL DETECTED !!!");

  digitalWrite(LED_PIN, HIGH);
  tone(BUZZER_PIN, 2000);

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("FALL DETECTED!");
  lcd.setCursor(0, 1);
  lcd.print("CHECK PERSON");

  delay(1000);

  // Stop continuous buzzer before countdown
  noTone(BUZZER_PIN);

  bool cancelled = false;

  // ================================
  // 10 SECOND COUNTDOWN
  // ================================

  for (int count = 10; count >= 1; count--) {

    lcd.clear();

    lcd.setCursor(0, 0);
    lcd.print("FALL DETECTED!");

    lcd.setCursor(0, 1);
    lcd.print("Press C: ");
    lcd.print(count);

    Serial.print("Countdown: ");
    Serial.print(count);
    Serial.println(" seconds");

    // Check keypad
    char key = keypad.getKey();

    if (key != NO_KEY) {

      Serial.print("Key pressed: ");
      Serial.println(key);

      // C = cancel
      if (key == 'C') {

        cancelled = true;
        break;
      }
    }

    // Short beeps during countdown
    tone(BUZZER_PIN, 2000);
    delay(200);
    noTone(BUZZER_PIN);

    delay(800);
  }

  // ================================
  // CANCELLED
  // ================================

  if (cancelled) {

    digitalWrite(LED_PIN, LOW);
    noTone(BUZZER_PIN);

    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("ALERT CANCELLED");

    lcd.setCursor(0, 1);
    lcd.print("Person Safe");

    Serial.println("ALERT CANCELLED");
    Serial.println("Person confirmed safe.");

    delay(3000);
  }

  // ================================
  // EMERGENCY
  // ================================

  else {

    lcd.clear();

    lcd.setCursor(0, 0);
    lcd.print("EMERGENCY!");

    lcd.setCursor(0, 1);
    lcd.print("HELP REQUIRED");

    Serial.println("!!! EMERGENCY !!!");
    Serial.println("No cancellation received.");
    Serial.println("GSM NOT CONNECTED");

    // Emergency alarm
    digitalWrite(LED_PIN, HIGH);

    tone(BUZZER_PIN, 2000);

    delay(5000);

    noTone(BUZZER_PIN);
    digitalWrite(LED_PIN, LOW);

    delay(1000);
  }

  // ================================
  // RETURN TO MONITORING
  // ================================

  lcd.clear();

  lcd.setCursor(0, 0);
  lcd.print("System Ready");

  lcd.setCursor(0, 1);
  lcd.print("Monitoring...");

  Serial.println("Back to monitoring.");
  Serial.println("----------------------------");

  delay(2000);
}
