# hand_controlled_robot
Hand Controlled Robot
  Materials:
    Arduino
    NRF24L01 //wireless communication platform
    MPU6050 //Gyro sensor
    DC motors
    Battery
    
Transmitter Code:

#include <SPI.h>
#include <RF24.h>
#include <Wire.h>
#include <MPU6050.h> //Mpu6050 kütüphanesi 

MPU6050 ivme_sensor; 
int x, y, z; //ivme tanımlama

RF24 radio(7, 8); // CE, CSN
int data[2]; // X ve Y düzlemi için dizi tanımlama

void setup() 
{
Wire.begin();
ivme_sensor.initialize();

radio.begin();
radio.openWritingPipe(1234);
Serial.begin(9600); 

}

void loop() 
{

ivme_sensor.getAcceleration(&x, &y, &z); // ivme ve gyro değerlerini okuma
    
data[0] = map(x, -17000, 17000, -255, 255 ); //X düzleminin verisi (ileri-geri)
data[1] = map(y, -17000, 17000, -255, 255); //Y düzeleminin verisi (sağ-sol)
radio.write(data, sizeof(data));
Serial.print("x axis: ");Serial.print(data[0]);Serial.print("   y axis: ");Serial.println(data[1]);
}




Receiver Code:


#include <SPI.h>
#include <RF24.h>

RF24 radio(7, 8); // CE, CSN

int data[2];// X ve Y düzlemi için dizi tanımlama


#define sol_motor_hiz 3 // Sol motor hız pini
#define sag_motor_hiz 10 // Sağ motor hız pini
#define sol_motor_1 4 // Sol motor ileri
#define sol_motor_2 9 // Sol motor geri
#define sag_motor_1 5 //Sağ motor ileri
#define sag_motor_2 6 // Sağ motor geri

int sag_sensor_echo = A0;// sağ sensör
int sag_sensor_trigger = A1;// sağ sensör
int sol_sensor_echo = A2;// sol sensör
int sol_sensor_trigger = A3;// sol sensör
int on_sensor_echo = A4; // Ön sensör
int on_sensor_trigger = A5; // Ön sensör

int PWM=255;

void setup()
{
  radio.begin();
radio.openReadingPipe(1,1234);
radio.startListening();
pinMode(on_sensor_trigger, OUTPUT);
pinMode(on_sensor_echo, INPUT);
pinMode(sol_sensor_trigger, OUTPUT);
pinMode(sol_sensor_echo, INPUT);
pinMode(sag_sensor_trigger, OUTPUT);
pinMode(sag_sensor_echo, INPUT);
pinMode(sol_motor_hiz, OUTPUT);
pinMode(sag_motor_hiz, OUTPUT);
pinMode(sol_motor_1, OUTPUT);
pinMode(sol_motor_2, OUTPUT);
pinMode(sag_motor_1, OUTPUT);
pinMode(sag_motor_2, OUTPUT);
Serial.begin(9600);
delay(5000);
}

void loop()
{ 
  long on_sensor_zaman, sol_sensor_zaman, sag_sensor_zaman, sag_mesafe, sol_mesafe, on_mesafe;
  digitalWrite(on_sensor_trigger, LOW);
  delayMicroseconds(2);
  digitalWrite(on_sensor_trigger, HIGH);
  delayMicroseconds(5);
  digitalWrite(on_sensor_trigger, LOW);
  on_sensor_zaman = pulseIn(on_sensor_echo, HIGH);
  on_mesafe = on_sensor_zaman/29/2;
  digitalWrite(sol_sensor_trigger, LOW);
  delayMicroseconds(2);
  digitalWrite(sol_sensor_trigger, HIGH);
  delayMicroseconds(5);
  digitalWrite(sol_sensor_trigger, LOW);
  sol_sensor_zaman = pulseIn(sol_sensor_echo, HIGH);
  sol_mesafe = sol_sensor_zaman/29/2;
  digitalWrite(sag_sensor_trigger, LOW);
  delayMicroseconds(2);
  digitalWrite(sag_sensor_trigger, HIGH);
  delayMicroseconds(5);
  digitalWrite(sag_sensor_trigger, LOW);
  sag_sensor_zaman = pulseIn(sag_sensor_echo, HIGH);
  sag_mesafe = sag_sensor_zaman/29/2;
  analogWrite(sol_motor_hiz, 0);
  analogWrite(sag_motor_hiz, 0);
  analogWrite(sol_motor_1, 0);
  analogWrite(sol_motor_2, 0);
  analogWrite(sag_motor_1, 0);
  analogWrite(sag_motor_2, 0);

 /* Serial.print("ön=");Serial.print(on_mesafe);
  Serial.print("    sağ=");Serial.print(sag_mesafe);
  Serial.print("    sol=");Serial.print(sol_mesafe);
  Serial.println("");  */
  
Serial.print("x axis: ");Serial.print(data[0]);Serial.print("y axis: ");Serial.println(data[1]);

if (radio.available()) {
  radio.read(data, sizeof(data));
  Serial.print("x axis: ");Serial.print(data[0]);Serial.print("y axis: ");Serial.println(data[1]);
  
  if(data[0]> 50  && on_mesafe > 10)//ileri
  {
    analogWrite(sol_motor_hiz, PWM);
    analogWrite(sag_motor_hiz, PWM);
    analogWrite(sol_motor_1, data[0]);
    analogWrite(sol_motor_2, 0);
    analogWrite(sag_motor_1, data[0]);
    analogWrite(sag_motor_2, 0);
    }
    if(data[0]< -50)//geri
  {
    analogWrite(sol_motor_hiz, PWM);
    analogWrite(sag_motor_hiz, PWM);
    analogWrite(sol_motor_1, 0);
    analogWrite(sol_motor_2, data[0]);
    analogWrite(sag_motor_1, 0);
    analogWrite(sag_motor_2, data[0]);
    }
    if(data[1] > 50)//sol
  {
    analogWrite(sol_motor_hiz, PWM);
    analogWrite(sag_motor_hiz, PWM);
    analogWrite(sol_motor_1, 0);
    analogWrite(sol_motor_2, 0);
    analogWrite(sag_motor_1, data[1]);
    analogWrite(sag_motor_2, 0);
    }
    if(data[1] < -50)//sağ
  {
    analogWrite(sol_motor_hiz, PWM);
    analogWrite(sag_motor_hiz, PWM);
    analogWrite(sol_motor_1, -data[1]);
    analogWrite(sol_motor_2, data[1]);
    analogWrite(sag_motor_1, 0);
    analogWrite(sag_motor_2, 0);
    }
    if(data[0] > -50 && data[0] < 50 && data[1] > -50 && data[1] < 50)//dur
    {
      analogWrite(sol_motor_hiz, 0);
    analogWrite(sag_motor_hiz, 0);
    analogWrite(sol_motor_1, 0);
    analogWrite(sol_motor_2, 0);
    analogWrite(sag_motor_1, 0);
    analogWrite(sag_motor_2, 0);
    }
}
}
