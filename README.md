Data L.ogger (and using cool sensors!)
A lab report by Tony Liebel

In The Report
Include your responses to the bold questions on your own fork of this lab report template. Include snippets of code that explain what you did. Deliverables are due next Tuesday. Post your lab reports as README.md pages on your GitHub, and post a link to that on your main class hub page.
For this lab, we will be experimenting with a variety of sensors, sending the data to the Arduino serial monitor, writing data to the EEPROM of the Arduino, and then playing the data back.

Part A. Writing to the Serial Monitor
a. Based on the readings from the serial monitor, what is the range of the analog values being read?
	0v-5v. Showing the values of 0-1023, however.
b. How many bits of resolution does the analog to digital converter (ADC) on the Arduino have?
	10 bits
 
Part B. RGB LED

How might you use this with only the parts in your kit? Show us your solution.
	I followed along with the code, and got the RGB to cycle through it’s many colors. 
1.	The common anode must be connected to power.
2.	Resistors must then be placed to each of the other LED pins, then to PWM pins
I would like to have the RGB change colors as a response to the accelerometer – maybe will figure this out later in the lab.
This is the code that I used:
int redPin = 6;
int greenPin = 5;
int bluePin = 3;
 
//uncomment this line if using a Common Anode LED
//#define COMMON_ANODE;
 
void setup()
{
  pinMode(redPin, OUTPUT);
  pinMode(greenPin, OUTPUT);
  pinMode(bluePin, OUTPUT);  
}
 
void loop()
{
  setColor(255, 0, 0);  // red
  delay(1000);
  setColor(0, 255, 0);  // green
  delay(1000);
  setColor(0, 0, 255);  // blue
  delay(1000);
  setColor(255, 255, 0);  // yellow
  delay(1000);  
  setColor(80, 0, 80);  // purple
  delay(1000);
  setColor(0, 255, 255);  // aqua
  delay(1000);
}
 
void setColor(int red, int green, int blue)
{
  #ifdef COMMON_ANODE
    red = 255 - red;
    green = 255 - green;
    blue = 255 - blue;
  #endif
  analogWrite(redPin, red);
  analogWrite(greenPin, green);
  analogWrite(bluePin, blue);  
}

Part C. Voltage Varying Sensors

1. FSR, Flex Sensor, Photo cell, Softpot
a. What voltage values do you see from your force sensor?
	I am able to read 0v to approximately 4.5 volts.
 
b. What kind of relationship does the voltage have as a function of the force applied? (e.g., linear?)
	It seems like it may be exponential, but it is not entirely possible to tell because I can’t confirm that I am applying force linearly.
 
c. Can you change the LED fading code values so that you get the full range of output voltages from the LED when using your FSR?
	Yes, it would be possible to do this by mapping the analog input from the force sensor to the full range of LED brightness (0-255)
 
d. What resistance do you need to have in series to get a reasonable range of voltages from each sensor?
	You would need to use 10k resistors.
 
e. What kind of relationship does the resistance have as a function of stimulus? (e.g., linear?)
	While the flex & FSR are not linear, the photosensor seems like it might be linear.
 
2. Accelerometer
a. Include your accelerometer read-out code in your write-up.
	#include <Wire.h>
#include <SPI.h>
#include <Adafruit_LIS3DH.h>
#include <Adafruit_Sensor.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

// Used for software SPI
#define LIS3DH_CLK 13
#define LIS3DH_MISO 12
#define LIS3DH_MOSI 11
// Used for hardware & software SPI
#define LIS3DH_CS 10
#define SCREEN_WIDTH 128 // OLED display width, in pixels
#define SCREEN_HEIGHT 32 // OLED display height, in pixels
// Declaration for an SSD1306 display connected to I2C (SDA, SCL pins)
#define OLED_RESET     4 // Reset pin # (or -1 if sharing Arduino reset pin)
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);


// software SPI
//Adafruit_LIS3DH lis = Adafruit_LIS3DH(LIS3DH_CS, LIS3DH_MOSI, LIS3DH_MISO, LIS3DH_CLK);
// hardware SPI
//Adafruit_LIS3DH lis = Adafruit_LIS3DH(LIS3DH_CS);
// I2C
Adafruit_LIS3DH lis = Adafruit_LIS3DH();

void setup(void) {
#ifndef ESP8266
  while (!Serial);     // will pause Zero, Leonardo, etc until serial console opens
#endif

  Serial.begin(9600);
  Serial.println("Hi Tony!");
    display.begin(SSD1306_SWITCHCAPVCC, 0x3C);// Address 0x3C for 128x32

  
  if (! lis.begin(0x18)) {   // change this to 0x19 for alternative i2c address
    Serial.println("Couldnt start");
    while (1);
  }
  Serial.println("LIS3DH found!");
  
  lis.setRange(LIS3DH_RANGE_4_G);   // 2, 4, 8 or 16 G!
  
  Serial.print("Range = "); Serial.print(2 << lis.getRange());  
  Serial.println("G");
}

void loop() {
  lis.read();      // get X Y and Z data at once
  // Then print out raw data
  Serial.print("X:  "); Serial.print(lis.x); 
  Serial.print("  \tY:  "); Serial.print(lis.y); 
  Serial.print("  \tZ:  "); Serial.print(lis.z); 

  /* Or....get a new sensor event, normalized */ 
  sensors_event_t event; 
  lis.getEvent(&event);
  
  /* Display the results (acceleration is measured in m/s^2) */
  Serial.print("\t\tX: "); Serial.print(event.acceleration.x);
  Serial.print(" \tY: "); Serial.print(event.acceleration.y); 
  Serial.print(" \tZ: "); Serial.print(event.acceleration.z); 
  Serial.println(" m/s^2 ");

  Serial.println();
 
  delay(200); 
  //print accelormeter values onto LED
  display.clearDisplay();
  display.setTextSize(1);             // Normal 1:1 pixel scale
  display.setTextColor(WHITE);        // Draw white text
  display.setCursor(0,0);  
  display.print("X:  "); display.print(lis.x);
  display.display();

  display.setCursor(0,10);  
  display.print("Y:  "); display.print(lis.y);
  display.display();

  display.setCursor(0,20);  
  display.print("Z:  "); display.print(lis.z);
  display.display();
}
Optional. Graphic Display
Take a picture of your screen working insert it here!
https://youtu.be/ySUjxvRYS2M 

Part D. Logging values to the EEPROM and reading them back

1. Reading and writing values to the Arduino EEPROM
a. Does it matter what actions are assigned to which state? Why?
	Yes, it matters because there are 3 separate states, all with different abilities. State 0 is used for clearing the memory, State 1 is used for reading what is stored in the memory, and State 2 is what is needed to write to the memory. These are needed in a specific order.
 
b. Why is the code here all in the setup() functions and not in the loop() functions?
	Because it really only needs to run on startup, and we wouldn’t want it to run continuously
 
c. How many byte-sized data samples can you store on the Atmega328?
	1024
 
d. How would you get analog data from the Arduino analog pins to be byte-sized? How about analog data from the I2C devices?
	EEPROM can only hold bytes with 8 bits, while analog signals on the Arduino are 10 bits. You would need to divide the signal from the analog pins
 
e. Alternately, how would we store the data if it were bigger than a byte? (hint: take a look at the EEPROMPut example)
	The EEPROMPut code uses update semantics, only writing to the memory if something has changed. 
Upload your modified code that takes in analog values from your sensors and prints them back out to the Arduino Serial Monitor.

2. Design your logger
a. Insert here a copy of your final state diagram.
 https://github.com/tonyliebel/Interactive-Lab-Hub/blob/master/Final%20State%20Data%20Logger.PNG

