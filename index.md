<img align = "right" src = "https://user-images.githubusercontent.com/93630610/180515604-3aa1e344-9047-42bb-81f5-785e7c99995c.png">
# 3-Joint Robotic Arm with Claw

I built a 3-jointed robotic arm that can pick up objects. Its movement can be controlled via potentiometers, but it can also enter a mode where it follows a set of pre-written instructions in order to pick up an object and set it down elsewhere.

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Cindy L. | BISV | Electrical Engineering | Incoming Junior |

![Headstone Image](https://user-images.githubusercontent.com/93630610/180076692-09a52651-ce25-47d0-995a-c32d05a88228.png)


  
# Final Milestone
My second and final milestone was finishing the code that allows the robot arm to move. The arm has two modes: potentiometer mode, where the arm's movement can be directly controlled via potentiometers, as well as auto mode, where the arm follows a pre-written set of instructions. Since during auto mode, the servo moved too quickly, I wrote a method that allows for the servos to move more smoothly. I also removed the support I added because it added too much weight to the robotic arm, causing for it to be unable to return to an upright posititon.

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/FdlcW692-zw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></center>

# First Milestone
  

My first milestone was finishing assembling the robot arm. I followed the instructions provided by the building set to assemble it, but I had to disassemble parts after I finished in order to align the robot arm properly. A servo was also improperly attached to the rest of the arm, so I fixed that. In addition, I added a support between the two sides of the robot arm to minimize the rattling of a servo.

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/aSILAIQVK5E" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></center>

# Final Code
` #include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#define OLED_RESET     4
Adafruit_SSD1306 display(128, 64, &Wire, OLED_RESET);

#define SER1 9
#define SER2 6
#define SER3 5
#define SER4 3
#define SER5 11

#define P1 0
#define P2 1
#define P3 2
#define P4 3
#define P5 6

#define button 4

#include <Servo.h>
Servo ser1;
Servo ser2;
Servo ser3;
Servo ser4;
Servo ser5;

int pos1 = 90;
int pos2 = 90;
int pos3 = 90;
int pos4 = 90;
int pos5 = 90;

double divNum = 5.683333333333;

boolean exited = false;
int pre = LOW;
int finCount = 0;

int s1Auto [] = {90, 125, 125, 125, 125, 125, 65, 65, 65, 65, 90};
int s2Auto [] = {90, 90, 30, 30, 30, 30, 40, 40, 35, 35, 90};
int s3Auto [] = {90, 90, 90, 165, 165, 105, 105, 105, 120, 120, 90};
int s5Auto [] = {60, 60, 60, 60, 90, 90, 90, 90, 90, 60, 60};

void setup() {
  ser1.attach(SER1);
  ser2.attach(SER2);
  ser3.attach(SER3);
  ser4.attach(SER4);
  ser5.attach(SER5);
  display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
  display.setTextColor(WHITE);
  display.clearDisplay();
  display.setTextSize(2);
  updateDisplay();
}

void loop() {
  if (checkButton()) {
    delay(10);
    autoMove();
    if (!exited) {
      shiftMode(sizeof(s1Auto) / sizeof(int) - 1);
      ++finCount;
      updateDisplay();
    }
  } else {
    exited = false;
    int read1 = analogRead(P1);
    int read2 = analogRead(P2);
    int read3 = analogRead(P3);
    int read4 = analogRead(P4);
    int read5 = analogRead(P5);
  
    // the range of analogRead of the potentiometer is 0-1023, so this converts the read to a range of 0-180 (or 25-90 for the 5th servo)
    pos1 = (int) (read1 / divNum);
    pos2 = (int) (read2 / divNum);
    pos3 = (int) (read3 / divNum);
    pos4 = (int) (read4 / divNum);
    pos5 = (int) (read5 / (divNum * 2.75)) + 25;
  
    ser1.write(pos1);
    ser2.write(pos2);
    ser3.write(pos3);
    ser4.write(pos4);
    ser5.write(pos5);
  }
  delay(100);
}

void autoMove() {
  shiftTo(pos1, s1Auto[0], ser1);
  shiftTo(pos2, s2Auto[0], ser2);
  shiftTo(pos3, s3Auto[0], ser3);
  shiftTo(pos5, s5Auto[0], ser5);
  for (int i = 1; i < sizeof(s1Auto) / sizeof(int); ++i) {
    if (checkButton()) {
      exited = true;
    }
    if (!exited) {
      shiftTo(s2Auto[i - 1], s2Auto[i], ser2);
      shiftTo(s3Auto[i - 1], s3Auto[i], ser3);
      shiftTo(s1Auto[i - 1], s1Auto[i], ser1);
      shiftTo(s5Auto[i - 1], s5Auto[i], ser5);
      for (int t = 0; t < 50; ++t) {
        if (checkButton()) {
          exited = true;
        }
        if (!exited) {
          delay(10);
        }
      }
    } else {
      shiftMode(i);
      break;
    }
  }
}

void shiftTo(int prev, int curr, Servo ser) {
  if (prev < curr) {
    for (int i = prev; i <= curr; i += 1) {
      if (!exited) {
        ser.write(i);
        if (checkButton()) {
          exited = true;
        }
      }
      delay(10);
    }
  } else {
    for (int i = prev; i >= curr; i -= 1) {
      if (!exited) {
        ser.write(i);
        if (checkButton()) {
          exited = true;
        }
      }
      delay(10);
    }
  }
  ser.write(curr);
}

boolean checkButton() {
  boolean val = digitalRead(button);
  boolean isOn = (val == LOW && pre == HIGH);
  pre = val;
  return isOn;
}

void shiftMode(int stepNum) {
  shiftTo(s2Auto[stepNum], pos2, ser2);
  delay(100);
  shiftTo(s3Auto[stepNum], pos3, ser3);
  delay(100);
  shiftTo(s5Auto[stepNum], pos5, ser5);
  delay(100);
  shiftTo(s1Auto[stepNum], pos1, ser1);
  delay(100);
}

void updateDisplay() {
  display.clearDisplay();
  display.setCursor(5, 20);
  String text = "# moved: " + String(finCount);
  display.print(text);
  display.display();
}'
