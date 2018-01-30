#include <Keypad.h>
#include <EEPROM.h>
#include <Wire.h>
#include <LCD.h>
#include <LiquidCrystal_I2C.h>
#define I2C_ADDR 0x27 // <<- Add your address here.
#define Rs_pin 0
#define Rw_pin 1
#define En_pin 2
#define BACKLIGHT_PIN 3
#define D4_pin 4
#define D5_pin 5
#define D6_pin 6
#define D7_pin 7

LiquidCrystal_I2C lcd(I2C_ADDR,En_pin,Rw_pin,Rs_pin,D4_pin,D5_pin,D6_pin,D7_pin);


const byte ROWS = 4; // Four rows
const byte COLS = 3; // Three columns
// Define the Keymap
char keys[ROWS][COLS] = {
  {'1','2','3'},
  {'4','5','6'},
  {'7','8','9'},
  {'*','0','#'}
};
// Connect keypad ROW0, ROW1, ROW2 and ROW3 to these Arduino pins.
//byte rowPins[ROWS] = { 9, 8, 7, 6 };
byte rowPins[ROWS] = { 2, 3, 4, 5 };
//byte rowPins[ROWS] = { 5, 4, 3, 2 };
// Connect keypad COL0, COL1 and COL2 to these Arduino pins.
//byte colPins[COLS] = { 12, 11, 10 }; 
byte colPins[COLS] = { 6, 7, 8 };
//byte colPins[COLS] = { 8, 7, 6 }; 

// Create the Keypad
Keypad kpd = Keypad( makeKeymap(keys), rowPins, colPins, ROWS, COLS );

#define ledpin 13
#define lockpin 11
int inpt[8];
short ep1=0;
int curnt=0;
long num=0;
long code=0;
long saved_code=0;
unsigned long last_key_time;
void setup()
{

  lcd.begin (16,2); // <<-- our LCD is a 20x4, change for your LCD if neededLCD Backlight ON
  lcd.setBacklightPin(BACKLIGHT_PIN,POSITIVE);
  lcd.setBacklight(HIGH);
  lcd.home();
  lcd.setCursor(5, 0);
  lcd.print("WELCOME");
  delay(500);
   
  pinMode(ledpin,OUTPUT);
  pinMode(lockpin,OUTPUT);
  digitalWrite(ledpin, HIGH);
  digitalWrite(lockpin, LOW);
  Serial.begin(9600);
  int val = EEPROM.read(0);
  Serial.print("Starting; current EEPROM value is ");
  Serial.println(val);
  if(val != 1){
    Serial.println("EEPROM byte not set yet; Writing...");
    EEPROM.write(0, 1);
    EEPROMWritelong(1, 10000);
    saved_code = 10000;
  } else if (val == 1000){
    Serial.println("EEPROM byte was set!");
  }
  saved_code = EEPROMReadlong(1);
  Serial.println("Done.");
  last_key_time = millis();
  Serial.println(saved_code);
  
 
}

void loop()
{
  char key = kpd.getKey();  

  lcd.setCursor(0,0);
  lcd.print("Enter Unlockcode");
  //delay(2000);
  
    if(key)  // Check for a valid key.
    {

    // for(int j=1;j<8;j++){
     lcd.setCursor(0, 1); 
     lcd.print("Key = ");
     //lcd.print(key);
    // }
     //delay(2000);
     
     
    switch (key)
    {
      case '*':
        //digitalWrite(ledpin, LOW);
        curnt=0;
        num=0;
        lcd.clear();
        break;
      case '#':
        //digitalWrite(ledpin, HIGH);
        curnt=0;
        num=0;
        lcd.clear();
        break;
      default:
      
      if((millis()-last_key_time)>10000){
        curnt=0;
        num=0;
        Serial.println("old key");
        //lcd.setCursor(8,1);
       // lcd.print("Old Key");
        //lcd.home();
        //delay(1000);
        lcd.clear();
        //lcd.setCursor(1,1);
        //lcd.print("Press * or #");
        //delay(600);
        //lcd.clear();    
        }
        
        last_key_time = millis();
      
      num = num * 10 + (key - '0');
        inpt[curnt]=key-'0';
        curnt++;
        
        //lcd.clear();
        //lcd.print(inpt);
        for(int i=0;i<8;i++){
          Serial.println(inpt[i]);
          }
          
          for(int j=0;j<curnt;j++){
                  lcd.print(inpt[j]);        
          }
          
          if(curnt==8){
            code = ((num/1000) ^ (num%1000));
            curnt=0;
            Serial.println("decrypting");
            if(num == 22200071){
              delay(1000);
              lcd.clear();
              lcd.setCursor(5, 1);
              lcd.print("Correct");
              delay(2000);
              lcd.clear(); //clear LCD, since we are still on 2nd line...
              lcd.setCursor(0,1);
              lcd.print("Door is Unlocked");
              lcd.home();
              delay(600);
              Serial.println("correct master code!!");
              digitalWrite(ledpin, LOW);
              digitalWrite(lockpin, HIGH);
              delay(2000);
              digitalWrite(ledpin, HIGH);
              digitalWrite(lockpin, LOW);
              }
              if(num == 92200071){
              Serial.println("correct master code!!");
              EEPROMWritelong(1, 10000);
             }
              
            num=0;
            if( ( (code > saved_code) &&
                  (code < (saved_code+50))
                ) || 
                ( (code < saved_code) &&
                  (code > (saved_code-3))
                )
               ){
              delay(1000);
              lcd.clear();
              delay(500);  
              Serial.println("correct code!!");
              
              lcd.setCursor(5, 1);
              lcd.print("Correct");
              delay(2000);
              lcd.clear(); //clear LCD, since we are still on 2nd line...
              lcd.setCursor(0,1);
              lcd.print("Door is Unlocked");
              lcd.home();
             // delay(600);
              
              EEPROMWritelong(1, code);
              saved_code = code;
              digitalWrite(ledpin, LOW);
              digitalWrite(lockpin, HIGH);
              delay(2000);
              digitalWrite(ledpin, HIGH);
              digitalWrite(lockpin, LOW);
              lcd.clear();
              }else{
                delay(1000);
                Serial.println("incorrect code!!");
                lcd.clear();
                delay(500);
                lcd.setCursor(4, 1);
                lcd.print("Incorrect");
                delay(2000);
                lcd.clear(); //clear LCD, since we are still on 2nd line...
                lcd.setCursor(1,1);
                lcd.print("Door is Locked");
                delay(2000);
                lcd.clear();
                lcd.home();
                delay(600);
                }
            Serial.println(code);
            Serial.println(saved_code);
            }
        Serial.println(num);
        
    }    
  }
}



void EEPROMWritelong(int address, long value)
      {
      //Decomposition from a long to 4 bytes by using bitshift.
      //One = Most significant -> Four = Least significant byte
      byte four = (value & 0xFF);
      byte three = ((value >> 8) & 0xFF);
      byte two = ((value >> 16) & 0xFF);
      byte one = ((value >> 24) & 0xFF);

      //Write the 4 bytes into the eeprom memory.
      EEPROM.write(address, four);
      EEPROM.write(address + 1, three);
      EEPROM.write(address + 2, two);
      EEPROM.write(address + 3, one);
      }
long EEPROMReadlong(long address)
      {
      //Read the 4 bytes from the eeprom memory.
      long four = EEPROM.read(address);
      long three = EEPROM.read(address + 1);
      long two = EEPROM.read(address + 2);
      long one = EEPROM.read(address + 3);

      //Return the recomposed long by using bitshift.
      return ((four << 0) & 0xFF) + ((three << 8) & 0xFFFF) + ((two << 16) & 0xFFFFFF) + ((one << 24) & 0xFFFFFFFF);
      }