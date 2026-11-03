This project aims to improve the safety and well-being of elderly individuals by providing an automated fall detection and emergency alert system. It uses the MPU6050 accelerometer and gyroscope sensor to continuously monitor body movements, detect sudden changes in acceleration, and identify unusual body orientations.

Key Features
Real-Time Monitoring: Continuously tracks body movement, acceleration, and tilt using the MPU6050 sensor.

Fall Detection: Identifies potential falls based on sudden acceleration changes and abnormal orientation.

Alert Notification: Activates an LED and buzzer when a possible fall is detected.

10-Second Cancellation: Provides a countdown period for the user to cancel a false alarm using a keypad.

Emergency Alert: If the user does not cancel the alert, the system activates an emergency alarm.

User-Friendly Design: Simple interface that makes the system easy for elderly people to operate.

Improved Safety: Helps reduce response delays by providing an immediate indication when a potential fall occurs.

Working Principle
The MPU6050 sensor continuously collects acceleration and gyroscope data from the user's body movements. The microcontroller processes this data to identify sudden movements and abnormal tilting conditions. When a possible fall is detected, the buzzer and LED are activated, and a 10-second countdown begins. During this period, the user can enter a cancellation input through the keypad. If no cancellation is received, the system triggers the emergency alarm to indicate that assistance may be needed
