# Mini Tank Robot
My project is the Mini Tank Robot named Austin. It is a robot that resembles a tank that can be controlled by a remote bluetooth controller. I chose this project mainly because I think the robot's design itself looks cool and fun to build. I built the robot and made it ultrasonic following and able to follow my flashlight which is my modification. My plan to complete my project is divided into 3 milestones. My 1st milestone will be to simply assemble the physical body of the robot. My second milestone is to get the robot moving by the mobile app through bluetooth. My last milestone will be to program the robot to be ultrasonic following where it can follow my hand wherever it goes. For my modification, I plan to make my robot light following meaning that if I shine a flashlight, the robot will follow.

|Austin H| |Homestead High School| |Engineering| |Incoming Sophomore|


**Replace the BlueStamp logo below with an image of yourself and your completed project. Follow the guide [here](https://tomcam.github.io/least-github-pages/adding-images-github-pages-site.html) if you need help.**

![Headstone Image](logo.svg)
  
# First Milestone

<iframe width="560" height="315" src="[https://www.youtube.com/watch?v=-zEXSaoK4Kc](https://www.youtube.com/embed/-zEXSaoK4Kc)" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

My first milestone will be assembling the physical body of the robot using all the components such as the motors, battery, and wheels. One of the biggest challenges in completing this milestone was that the instructions were very unclear and the images were not detailed enough. Additionally, there are many small screws and parts where it is really hard to wire up.

# Second Milestonez

<iframe width="560" height="315" src="https://www.youtube.com/watch?v=royftR4s4Yk" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

In the second milestone, my goal was to be able to make two functions work at the same time. I was able to get the robot moving by the bluetooth controller and I made the LED board start flashing. The project was not as difficult as I thought it would be before coming into this summer program. One of my biggest challenges was fixing errors within the code. For my final milestone, I plan to make three functions work simultaneously and add on my modification. My modification will be to

# Final Milestone

<iframe width="560" height="315" src="https://www.youtube.com/watch?v=okEF07TcLl8" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

The goal for my third milestone was to be able to get the robot to follow my hand. My biggest challenges throughout this program was mainly getting to know engineering and robotics. I have learned multiple things such as soldering, engineering, and it was a good way to get an introduction to engineering. I hope that I can continue to gain more experience working with robotics and engineering whether it be at school or a program like this.

# Schematics 
Here's where you'll put images of your schematics. [Tinkercad](https://www.tinkercad.com/blog/official-guide-to-tinkercad-circuits) and [Fritzing](https://fritzing.org/learning/) are both great resources to create professional schematic diagrams, though BSE recommends Tinkercad becuase it can be done easily and for free in the browser. 

# Code
Here's where you'll put your code. The syntax below places it into a block of code. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize it to your project needs. 

/*
 Keyestudio Mini Tank Robot v2.0
 Final Merged Sketch: Light Following (Override) + Ultrasonic Hand Following + LED Panel
*/

// LED Matrix Icons
unsigned char start01[] = {0x01,0x02,0x04,0x08,0x10,0x20,0x40,0x80,0x80,0x40,0x20,0x10,0x08,0x04,0x02,0x01};
unsigned char front[]   = {0x00,0x00,0x00,0x00,0x00,0x24,0x12,0x09,0x12,0x24,0x00,0x00,0x00,0x00,0x00,0x00};
unsigned char back[]    = {0x00,0x00,0x00,0x00,0x00,0x24,0x48,0x90,0x48,0x24,0x00,0x00,0x00,0x00,0x00,0x00};
unsigned char left[]    = {0x00,0x00,0x00,0x00,0x00,0x00,0x44,0x28,0x10,0x44,0x28,0x10,0x44,0x28,0x10,0x00};
unsigned char right[]   = {0x00,0x10,0x28,0x44,0x10,0x28,0x44,0x10,0x28,0x44,0x00,0x00,0x00,0x00,0x00,0x00};
unsigned char STOP01[]  = {0x2E,0x2A,0x3A,0x00,0x02,0x3E,0x02,0x00,0x3E,0x22,0x3E,0x00,0x3E,0x0A,0x0E,0x00};
unsigned char clear[]   = {0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00};

// LED Matrix Pins
#define SCL_Pin A5
#define SDA_Pin A4

// Motor Pins
#define ML_Ctrl 13
#define ML_PWM 11
#define MR_Ctrl 12
#define MR_PWM 3

// Photoresistor Pins
#define leftLDR A0
#define rightLDR A1

// Ultrasonic & Servo Pins
#define Trig 5
#define Echo 4
#define servoPin 9

// Ambient Light Variables
int leftAmbient = 0;
int rightAmbient = 0;
int pulsewidth;
float distance;

// Tuning Parameters
#define FLASHLIGHT_MARGIN 70  // How much brighter than room light flashlight must be
#define STEER_THRESHOLD 40    // Steering deadband

void setup()
{
  Serial.begin(9600);

  // Matrix display setup
  pinMode(SCL_Pin, OUTPUT);
  pinMode(SDA_Pin, OUTPUT);
  matrix_display(clear);
  matrix_display(start01);

  // Servo setup (Center to 90 degrees)
  pinMode(servoPin, OUTPUT);
  procedure(90); 

  // Ultrasonic setup
  pinMode(Trig, OUTPUT);
  pinMode(Echo, INPUT);

  // Motor setup
  pinMode(ML_Ctrl, OUTPUT);
  pinMode(ML_PWM, OUTPUT);
  pinMode(MR_Ctrl, OUTPUT);
  pinMode(MR_PWM, OUTPUT);

  Car_Stop();

  // CALIBRATION: Sample ambient room light for 2 seconds
  Serial.println("Calibrating ambient light... Keep flashlight OFF!");
  long sumLeft = 0;
  long sumRight = 0;
  int samples = 50;

  for (int i = 0; i < samples; i++) {
    sumLeft += analogRead(leftLDR);
    sumRight += analogRead(rightLDR);
    delay(40);
  }

  leftAmbient = sumLeft / samples;
  rightAmbient = sumRight / samples;

  Serial.println("Calibration complete!");
}

void loop()
{
  // 1. Measure Distance
  distance = checkdistance();

  // 2. Measure Light
  int leftLight = analogRead(leftLDR);
  int rightLight = analogRead(rightLDR);

  // Check if light on either sensor exceeds room baseline (prevents shadow false-positives)
  bool leftActive  = (leftLight > (leftAmbient + FLASHLIGHT_MARGIN));
  bool rightActive = (rightLight > (rightAmbient + FLASHLIGHT_MARGIN));
  bool flashlightPresent = (leftActive || rightActive);

  int difference = leftLight - rightLight;

  Serial.print("Dist: ");
  Serial.print(distance);
  Serial.print("cm | L: ");
  Serial.print(leftLight);
  Serial.print(" | R: ");
  Serial.print(rightLight);
  Serial.print(" | Flashlight: ");
  Serial.println(flashlightPresent ? "YES" : "NO");

  // --- DECISION LOGIC (PRORITY ORDER) ---

  // CRITICAL SAFETY 1: Too Close (< 10 cm) -> Reverse
  if (distance > 0 && distance <= 10)
  {
    matrix_display(back);
    Car_back();
  }
  // CRITICAL SAFETY 2: Stop Zone (10 cm to 20 cm) -> Stop
  else if (distance > 10 && distance <= 20)
  {
    matrix_display(STOP01);
    Car_Stop();
  }
  // PRIORITY 1 OVERRIDE: Flashlight Detected -> Steering Follow Mode
  else if (flashlightPresent)
  {
    if (difference > STEER_THRESHOLD)
    {
      matrix_display(right);
      Car_right();
    }
    else if (difference < -STEER_THRESHOLD)
    {
      matrix_display(left);
      Car_left();
    }
    else
    {
      matrix_display(front);
      Car_front();
    }
  }
  // PRIORITY 2: Ultrasonic Hand Following (20 cm to 50 cm)
  else if (distance > 20 && distance <= 50)
  {
    matrix_display(front);
    Car_front();
  }
  // IDLE: No flashlight active and nothing within range -> Stop
  else
  {
    matrix_display(STOP01);
    Car_Stop();
  }

  delay(50);
}

/*********** MOTOR FUNCTIONS ****************/

void Car_front()
{
  digitalWrite(MR_Ctrl, LOW);
  analogWrite(MR_PWM, 200);
  digitalWrite(ML_Ctrl, LOW);
  analogWrite(ML_PWM, 200);
}

void Car_back()
{
  digitalWrite(MR_Ctrl, HIGH);
  analogWrite(MR_PWM, 200);
  digitalWrite(ML_Ctrl, HIGH);
  analogWrite(ML_PWM, 200);
}

void Car_left()
{
  digitalWrite(MR_Ctrl, LOW);
  analogWrite(MR_PWM, 200);
  digitalWrite(ML_Ctrl, HIGH);
  analogWrite(ML_PWM, 200);
}

void Car_right()
{
  digitalWrite(MR_Ctrl, HIGH);
  analogWrite(MR_PWM, 200);
  digitalWrite(ML_Ctrl, LOW);
  analogWrite(ML_PWM, 200);
}

void Car_Stop()
{
  digitalWrite(MR_Ctrl, LOW);
  analogWrite(MR_PWM, 0);
  digitalWrite(ML_Ctrl, LOW);
  analogWrite(ML_PWM, 0);
}

/**************** ULTRASONIC & SERVO ****************/

float checkdistance() {
  digitalWrite(Trig, LOW);
  delayMicroseconds(2);
  digitalWrite(Trig, HIGH);
  delayMicroseconds(10);
  digitalWrite(Trig, LOW);
  
  float dist = pulseIn(Echo, HIGH, 30000) / 58.20; 
  
  // FIX: If pulseIn times out (returns 0), set to 999 so code doesn't freeze or lock up
  if (dist == 0) {
    dist = 999;
  }
  
  delay(10);
  return dist;
}

void procedure(int myangle) {
  for (int i = 0; i <= 50; i++) {
    pulsewidth = myangle * 11 + 500;
    digitalWrite(servoPin, HIGH);
    delayMicroseconds(pulsewidth);
    digitalWrite(servoPin, LOW);
    delay((20 - pulsewidth / 1000));
  }
}

/**************** DOT MATRIX FUNCTIONS ****************/

void matrix_display(unsigned char matrix_value[])
{
  IIC_start();
  IIC_send(0xc0);
  
  for(int i = 0; i < 16; i++)
  {
    IIC_send(matrix_value[i]);
  }

  IIC_end();
  
  IIC_start();
  IIC_send(0x8A);
  IIC_end();
}

void IIC_start()
{
  digitalWrite(SCL_Pin, HIGH);
  delayMicroseconds(3);
  digitalWrite(SDA_Pin, HIGH);
  delayMicroseconds(3);
  digitalWrite(SDA_Pin, LOW);
  delayMicroseconds(3);
}

void IIC_send(unsigned char send_data)
{
  for(char i = 0; i < 8; i++)
  {
    digitalWrite(SCL_Pin, LOW);
    delayMicroseconds(3);
    if(send_data & 0x01)
      digitalWrite(SDA_Pin, HIGH);
    else
      digitalWrite(SDA_Pin, LOW);
      
    delayMicroseconds(3);
    digitalWrite(SCL_Pin, HIGH);
    delayMicroseconds(3);
    send_data = send_data >> 1;
  }
}

void IIC_end()
{
  digitalWrite(SCL_Pin, LOW);
  delayMicroseconds(3);
  digitalWrite(SDA_Pin, LOW);
  delayMicroseconds(3);
  digitalWrite(SCL_Pin, HIGH);
  delayMicroseconds(3);
  digitalWrite(SDA_Pin, HIGH);
  delayMicroseconds(3);
}

# Bill of Materials
Here's where you'll list the parts in your project. To add more rows, just copy and paste the example rows below.
Don't forget to place the link of where to buy each component inside the quotation marks in the corresponding row after href =. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize this to your project needs. 

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Item Name | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Item Name | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Item Name | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |

# Other Resources/Examples
One of the best parts about Github is that you can view how other people set up their own work. Here are some past BSE portfolios that are awesome examples. You can view how they set up their portfolio, and you can view their index.md files to understand how they implemented different portfolio components.
- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)

To watch the BSE tutorial on how to create a portfolio, click here.
