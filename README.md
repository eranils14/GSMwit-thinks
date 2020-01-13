#include <SoftwareSerial.h>
#include <String.h>
 
SoftwareSerial mySerial(7, 8);

boolean pin2=LOW,pin3=LOW,pin4=LOW,pin5=LOW,pin6=LOW; 
float temp=0.0;

void setup()
{
  mySerial.begin(9600);               // the GPRS baud rate   
  Serial.begin(9600);    // the GPRS baud rate 
  pinMode(2,INPUT);
  pinMode(3,INPUT);
  pinMode(4,INPUT);
  pinMode(5,INPUT);  
  pinMode(6,INPUT);  
  delay(1000);
}
 
void loop()
{
      temp=analogRead(A0);
      temp=temp*0.4887;  
      delay(2);          
       Send2Pachube();
       Serial.print("  ");
       Serial.print(temp);
  if (mySerial.available())
    Serial.write(mySerial.read());
}
void Send2Pachube()
{
  mySerial.println("AT");
  delay(1000);

  mySerial.println("AT+CPIN?");
  delay(1000);

  mySerial.println("AT+CREG?");
  delay(1000);

  mySerial.println("AT+CGATT?");
  delay(1000);

  mySerial.println("AT+CIPSHUT");
  delay(1000);

  mySerial.println("AT+CIPSTATUS");
  delay(2000);

  mySerial.println("AT+CIPMUX=0");
  delay(2000);
 
  ShowSerialData();
 
  mySerial.println("AT+CSTT=\"airtelgprs.com\"");//start task and setting the APN,
  delay(1000);
 
  ShowSerialData();
 
  mySerial.println("AT+CIICR");//bring up wireless connection
  delay(3000);
 
  ShowSerialData();
 
  mySerial.println("AT+CIFSR");//get local IP adress
  delay(2000);
 
  ShowSerialData();
 
  mySerial.println("AT+CIPSPRT=0");
  delay(3000);
 
  ShowSerialData();
  
  mySerial.println("AT+CIPSTART=\"https://api.thingspeak.com/update?api_key=08YIAK7D72R1VC4Q&field1=0\",\"80\"");//start up the connection
  delay(6000);
 
  ShowSerialData();
 
  mySerial.println("AT+CIPSEND");//begin send data to remote server
  delay(4000);
  ShowSerialData();
  
    String str="GET 'http://api.thingspeak.com/channels/949163/feeds.json?api_key=CG66YT68BNHIBW2S&results=2'" + String(temp);
  mySerial.println(str);//begin send data to remote server
  delay(4000);
  ShowSerialData();

  mySerial.println((char)26);//sending
  delay(5000);//waitting for reply, important! the time is base on the condition of internet 
  mySerial.println();
 
  ShowSerialData();
 
  mySerial.println("AT+CIPSHUT");//close the connection
  delay(100);
  ShowSerialData();
} 
void ShowSerialData()
{
  while(mySerial.available()!=0)
    Serial.write(mySerial.read());
}






___________________________________________________________________

04 Ultrasonic with signal



#include <AWS_IOT.h>
#include <WiFi.h>

#include "time.h"

WiFiServer server(80);

AWS_IOT hornbill;
char WIFI_SSID[]="Inwizards Inc";
char WIFI_PASSWORD[]="TE19ch00";
char HOST_ADDRESS[]="a12huzohes0rnz-ats.iot.ap-south-1.amazonaws.com";
char CLIENT_ID[]= "707581501534";
char TOPIC_NAME[]= "$aws/things/mutiultrasonic/shadow/update";

const char* ntpServer = "de.pool.ntp.org";
const long  gmtOffset_sec = 19800;
const int   daylightOffset_sec = 19800;
int second;
int minute;
int hour;
int day;
int month;
int year;
int weekday;
long current;
struct tm timeinfo;
void printLocalTime()
{
if(!getLocalTime(&timeinfo)){
Serial.println("Failed to obtain time");
return;
}
  //Serial.println(&timeinfo, "%A, %d %B %Y %H:%M:%S");
}

const int trigPin1 = 34;
const int echoPin1 = 35;

const int trigPin2 = 32;
const int echoPin2 = 33;

const int trigPin3 = 26;
const int echoPin3 = 27;

const int trigPin4 = 12;
const int echoPin4 = 14;

long duration1;
int distance1;
long duration2;
int distance2;
long duration3;
int distance3;
long duration4;
int distance4;

int status = WL_IDLE_STATUS;
String  slotstatus1 = "V";
String  slotstatus2 = "V";
String  slotstatus3 = "V";
String  slotstatus4 = "V";

int tick=0, tickbool=0, msgCount=0, msgReceived = 0;

char payload[512];
char rcvdPayload[512];
String Serialreadvelue =" ";// these are the starting values to print
void mySubCallBackHandler (char *topicName, int payloadLen, char *payLoad)
{
    strncpy(rcvdPayload,payLoad,payloadLen);
    rcvdPayload[payloadLen] = 0;
    msgReceived = 1;
}
const int output15 = 15;
const int output2 = 2;

const int output4 = 4;
const int output5 = 5;

const int output18 = 18;
const int output19 = 19;

const int output22 = 22;
const int output23 = 23;

void setup() {
 Serial.begin(115200); // Starts the serial communication  
while (status != WL_CONNECTED)

    {
        Serial.print("Attempting to connect to SSID: ");
        Serial.println(WIFI_SSID);
        // Connect to WPA/WPA2 network. Change this line if using open or WEP network:
        status = WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
        delay(3000); // wait 3 seconds for connection:
    }
    Serial.println("Connected to wifi");

configTime(gmtOffset_sec, daylightOffset_sec, ntpServer);
printLocalTime();
  
if(hornbill.connect(HOST_ADDRESS,CLIENT_ID)== 0)
    {
        Serial.println("Connected to AWS");
        delay(1000);

if(0==hornbill.subscribe(TOPIC_NAME,mySubCallBackHandler))
        {
            Serial.println("Subscribe Successfull");
        }
else
        {
            Serial.println("Subscribe Failed, Check the Thing Name and Certificates");
while(1);
        }
    }
else
    {
        Serial.println("AWS connection failed, Check the HOST Address");
while(1);
    }
delay(2000);

  //Serial.println("");
  Serial.println("WiFi connected.");
  Serial.println("IP address: "); // Print ESP IP Address
  Serial.println(WiFi.localIP());
  Serial.println("MAC address: ");// Print ESP MAC Address 
  Serial.println(WiFi.macAddress());
  server.begin();
    
    pinMode(output15, OUTPUT);
    pinMode(output2, OUTPUT);
    pinMode(output4, OUTPUT);
    pinMode(output5, OUTPUT);
    pinMode(output18, OUTPUT);
    pinMode(output19, OUTPUT);
    pinMode(output22, OUTPUT);
    pinMode(output23, OUTPUT);
    
    digitalWrite(output15, LOW);
    digitalWrite(output2, LOW);
    digitalWrite(output4, LOW);
    digitalWrite(output5, LOW);
    digitalWrite(output18, LOW);
    digitalWrite(output19, LOW);
    digitalWrite(output22, LOW);
    digitalWrite(output23, LOW);

    pinMode(trigPin1, OUTPUT); // Sets the trigPin as an Output
    pinMode(echoPin1, INPUT); // Sets the echoPin as an Input

    pinMode(trigPin2, OUTPUT); // Sets the trigPin as an Output
    pinMode(echoPin2, INPUT); // Sets the echoPin as an Input

    pinMode(trigPin3, OUTPUT); // Sets the trigPin as an Output
    pinMode(echoPin3, INPUT); // Sets the echoPin as an Input

    pinMode(trigPin4, OUTPUT); // Sets the trigPin as an Output
    pinMode(echoPin4, INPUT); // Sets the echoPin as an Input
}
void loop() {

printLocalTime();
  second = timeinfo.tm_sec;
  minute = timeinfo.tm_min;
  hour = timeinfo.tm_hour;
  day = timeinfo.tm_mday;
  month = timeinfo.tm_mon + 1;
  year = timeinfo.tm_year + 1900;
  weekday = timeinfo.tm_wday +1;
  Serial.print("dt: ");
  Serial.print(day);
  Serial.print("/");
  Serial.print(month);
  Serial.print("/");
  Serial.print(year);
  Serial.print(" - ");
  Serial.print(hour);
  Serial.print(":");
  Serial.print(minute);
  Serial.print(":");
  Serial.println(second);
  delay(2000);


    digitalWrite(trigPin1, LOW);// Clears the trigPin
    delay(1000);// Sets the trigPin on HIGH state for 10 micro seconds
    digitalWrite(trigPin1, HIGH);

    digitalWrite(trigPin2, LOW);// Clears the trigPin
    delay(1000);// Sets the trigPin on HIGH state for 10 micro seconds
    digitalWrite(trigPin2, HIGH);

    digitalWrite(trigPin3, LOW);// Clears the trigPin
    delay(1000);// Sets the trigPin on HIGH state for 10 micro seconds
    digitalWrite(trigPin3, HIGH);

    digitalWrite(trigPin4, LOW);// Clears the trigPin
    delay(1000);// Sets the trigPin on HIGH state for 10 micro seconds
    digitalWrite(trigPin4, HIGH);

    delay(1000); // Reads the echoPin, returns the sound wave travel time in microseconds
  
    digitalWrite(trigPin1, LOW);
    duration1 = pulseIn(echoPin1, HIGH);
    digitalWrite(trigPin2, LOW);
    duration2 = pulseIn(echoPin2, HIGH);
    digitalWrite(trigPin3, LOW);
    duration3 = pulseIn(echoPin3, HIGH);
    digitalWrite(trigPin4, LOW);
    duration4 = pulseIn(echoPin4, HIGH);

    distance1= duration1*0.034/2; // Calculating the distance
    distance2= duration2*0.034/2; // Calculating the distance
    distance3= duration3*0.034/2; // Calculating the distance
    distance4= duration4*0.034/2; // Calculating the distance

    Serial.print("Dist1 CM:-> "); // Prints the distance on the Serial Monitor
    Serial.println(distance1);
    Serial.print("Dist2 CM:-> "); // Prints the distance on the Serial Monitor
    Serial.println(distance2);
    Serial.print("Dist3 CM:-> "); // Prints the distance on the Serial Monitor
    Serial.println(distance3);
    Serial.print("Dist4 CM:-> "); // Prints the distance on the Serial Monitor
    Serial.println(distance4);
    delay(1000);
    
if (distance1 > 40) {
  tick = 10;
    digitalWrite(output15, HIGH);
    Serial.println(" V ");   //Vacant
    digitalWrite(output2, LOW);
    slotstatus1 ="V";
  } else {
   
    digitalWrite(output15, LOW);
    digitalWrite(output2, HIGH);
    Serial.println(" O ");  //occupied
    slotstatus1 ="O";
  }

if (distance2 > 40) {
  tick = 10;
    digitalWrite(output4, HIGH);
    Serial.println(" V ");   //Vacant
    digitalWrite(output5, LOW);
    slotstatus2 ="V";
  } else {
    digitalWrite(output4, LOW);
    digitalWrite(output5, HIGH);
    Serial.println(" O ");  //occupied
    slotstatus2 ="O";
  }

if (distance3 > 40) {
  tick = 10;
    digitalWrite(output18, HIGH);
    Serial.println(" V ");   //Vacant
    digitalWrite(output19, LOW);
    slotstatus3 ="V";
  } else {
    digitalWrite(output18, LOW);
    digitalWrite(output19, HIGH);
    Serial.println(" O ");  //occupied
    slotstatus3 ="O";
  }

if (distance4 > 40) {
  tick = 10;
    digitalWrite(output22, HIGH);
    Serial.println(" V ");   //Vacant
    digitalWrite(output23, LOW);
    slotstatus4 ="V";
  } else {
    digitalWrite(output22, LOW);
    digitalWrite(output23, HIGH);
    Serial.println(" O ");  //occupied
    slotstatus4 ="O";
  }

 //Serial.println("Serial.println(tick);");
 //Serial.println(tick);
if(tick >= 10)   // publish to topic every 5 second
   {
  tick=0;
     //Serial.println(tick);
     //Serial.print(slotstatus1);
     //Serial.print(slotstatus2);
     //Serial.print(slotstatus3);
     //Serial.print(slotstatus4);
     //delay(5000);
        sprintf(payload,"{ \"boardid\":\"246F280B81F4\", \"time\":\"%.d-%.d-%.d %.d:%.d:%.d\", \"dist1\":\"%.d\", \"dist2\":\"%.d\", \"dist3\":\"%.d\", \"dist4\":\"%.d\", \"sta Ps-1\":\"%s\", \"sta Ps-2\":\"%s\",\"sta Ps-3\":\"%s\", \"sta Ps-4\":\"%s\" }", day, month, year, hour, minute,second, distance1, distance2, distance3, distance4, slotstatus1, slotstatus2, slotstatus3, slotstatus4, msgCount++);
        if(hornbill.publish(TOPIC_NAME,payload) == 0)
        {        
            Serial.print("Publish Message:");
            Serial.println(payload);
            //delay(2000);
        }
        else
        {
            Serial.println("Publish failed");
        }
    }  
    vTaskDelay(10000 / portTICK_RATE_MS); 
      }





______________________________________________________________________________________

GSM GPS with sms for call 



#include <SoftwareSerial.h> //Software Serial header to communicate with GSM module

//#include<LiquidCrystal.h>

//LiquidCrystal lcd(13,12,11,10,9,8);



SoftwareSerial SIM800(2, 4); // RX, TX



String Link = "The current Location is https://www.google.com/maps/place/";
String responce = "";
String Longitude = "";
String Latitude = "";
String SIM800_send(String incoming) //Function to communicate with SIM800 module

{

SIM800.println(incoming); delay(100); //Print what is being sent to GSM module
String result = "";

    while (SIM800.available()) //Wait for result

    {

    char letter = SIM800.read();

    result = result + String(letter); //combine char to string to get result

    }
return result; //return the result

}

void setup() {

//  lcd.begin(16,2);

  Serial.begin(115200); //Serial COM for debugging

  SIM800.begin(9600); //Software serial called SIM800 to speak with SIM800 Module

  //lcd.print("Location Tracker");

//  lcd.setCursor(0,1);

  //lcd.print("  Without GPS   ");

  //delay(3000);

 // lcd.clear();

   //lcd.print("Connecting with");

 // lcd.setCursor(0,1);

  //lcd.print("    GSM Modem   ");

  //delay(1000); //wait for serial COM to get ready



responce = SIM800_send("ATE1"); //Enable Echo if not enabled by default
Serial.print ("Responce:"); Serial.println(responce);
delay(1000);

responce = SIM800_send("AT+CGATT=1"); //Set the SIM800 in GPRS mode
Serial.print ("Responce:"); Serial.println(responce);
delay(1000);

responce = SIM800_send("AT+SAPBR=3,1,\"CONTYPE\",\"GPRS\" "); //Activate Bearer profile
Serial.print ("Responce:"); Serial.println(responce);
delay(1000);

responce = SIM800_send("AT+SAPBR=3,1,\"APN\",\"airtelgprs.com\""); //Set VPN options => 'RCMNET' 'www'
Serial.print ("Responce:"); Serial.println(responce);

  delay(2000);

  responce = SIM800_send("AT+SAPBR=1,1"); //Open bearer Profile

  Serial.print ("Responce:"); Serial.println(responce); //Open bearer Profile

  delay(2000);



  responce = SIM800_send("AT+SAPBR=2,1"); //Get the IP address of the bearer profile

  Serial.print ("Responce:"); Serial.println(responce);

  delay(1000);

  
}



void prepare_message()

{

  //Sample Output for AT+CIPGSMLOC=1,1   ==> +CIPGSMLOC: 0,75.802460,26.848892,2019/04/23,08:32:35 //where 26.8488832 is Lattitude and 75.802460 is longitute

  int first_comma = responce.indexOf(','); //Find the position of 1st comma

  int second_comma = responce.indexOf(',', first_comma+1); //Find the position of 2nd comma

  int third_comma = responce.indexOf(',', second_comma+1); //Find the position of 3rd comma



  for(int i=first_comma+1; i<second_comma; i++) //Values form 1st comma to 2nd comma is Longitude

    Longitude = Longitude + responce.charAt(i);



  for(int i=second_comma+1; i<third_comma; i++) //Values form 2nd comma to 3rd comma is Latitude

    Latitude = Latitude + responce.charAt(i);



  Serial.println(Latitude); Serial.println(Longitude);

  Link = Link + Latitude + "," + Longitude; //Update the Link with latitude and Logitude values

  Serial.println(Link);

 // lcd.setCursor(0,0);

  //lcd.print("Lat: ");

  //lcd.print(Latitude);

  //lcd.setCursor(0,1);

  //lcd.print("Lon: ");

  //lcd.print(Longitude);

}



String incoming = "";



void loop() {



  if (SIM800.available()) { //Check if the SIM800 Module is telling anything

    char a = SIM800.read();

    Serial.write(a); //print what the module tells on serial monitor

    incoming = incoming + String(a);

    if (a == 13) //check for new line

    incoming =""; //clear the string if new line is detected

    incoming.trim(); //Remove /n or /r from the incomind data

 

    if (incoming=="RING") //If an incoming call is detected the SIM800 module will say "RING" check for it

    {

     Serial.println ("Sending sms"); delay(1000);

     responce = SIM800_send("ATH"); //Hand up the incoming call using ATH

     delay (1000);

     responce = SIM800_send("ATE0"); //Disable Echo

     delay (1000);



     responce = ""; Latitude=""; Longitude=""; //initialise all string to null

     SIM800.println("AT+CLBS=1,1"); delay(5000); //Request for location data

      while (SIM800.available())

      {

       char letter = SIM800.read();

       responce = responce + String(letter); //Store the location information in string responce

      }

      Serial.print("Result Obtained as:");   Serial.print(responce); Serial.println("*******");



     prepare_message(); delay(1000); //use prepare_message funtion to prepare the link with the obtained LAT and LONG co-ordinates



     SIM800.println("AT+CMGF=1"); //Set the module in SMS mode

     delay(1000);

   

     SIM800.println("AT+CMGS=\"+919993\""); //Send SMS to this number

     delay(1000);



     SIM800.println(Link); // we have send the string in variable Link

     delay(1000);



     SIM800.println((char)26);// ASCII code of CTRL+Z - used to terminate the text message

     delay(1000);

    }

  }



  if (Serial.available()) { //For debugging

    SIM800.write(Serial.read());

  }



}


____________________________________________________________________________________________________


#include <SoftwareSerial.h> //Software Serial header to communicate with GSM module

#include<LiquidCrystal.h>

LiquidCrystal lcd(13,12,11,10,9,8);



SoftwareSerial SIM800(2, 3); // RX, TX



String Link = "The current Location is https://www.google.com/maps/place/";

String responce = "";

String Longitude = "";

String Latitude = "";



String SIM800_send(String incoming) //Function to communicate with SIM800 module

{

    SIM800.println(incoming); delay(100); //Print what is being sent to GSM module

    String result = "";

    while (SIM800.available()) //Wait for result

    {

    char letter = SIM800.read();

    result = result + String(letter); //combine char to string to get result

    }

 

return result; //return the result

}



void setup() {



  lcd.begin(16,2);

  Serial.begin(9600); //Serial COM for debugging

  SIM800.begin(9600); //Software serial called SIM800 to speak with SIM800 Module

  lcd.print("Location Tracker");

  lcd.setCursor(0,1);

  lcd.print("  Without GPS   ");

  delay(3000);

  lcd.clear();

   lcd.print("Connecting with");

  lcd.setCursor(0,1);

  lcd.print("    GSM Modem   ");

  delay(1000); //wait for serial COM to get ready



  responce = SIM800_send("ATE1"); //Enable Echo if not enabled by default

  Serial.print ("Responce:"); Serial.println(responce);

  delay(1000);



  responce = SIM800_send("AT+CGATT=1"); //Set the SIM800 in GPRS mode

  Serial.print ("Responce:"); Serial.println(responce);

  delay(1000);



  responce = SIM800_send("AT+SAPBR=3,1,\"CONTYPE\",\"GPRS\" "); //Activate Bearer profile

  Serial.print ("Responce:"); Serial.println(responce);

  delay(1000);



  responce = SIM800_send("AT+SAPBR=3,1,\"APN\",\"www\" "); //Set VPN options => 'RCMNET' 'www'

  Serial.print ("Responce:"); Serial.println(responce);

  delay(2000);

 

  responce = SIM800_send("AT+SAPBR=1,1"); //Open bearer Profile

  Serial.print ("Responce:"); Serial.println(responce); //Open bearer Profile

  delay(2000);



  responce = SIM800_send("AT+SAPBR=2,1"); //Get the IP address of the bearer profile

  Serial.print ("Responce:"); Serial.println(responce);

  delay(1000);

  
}



void prepare_message()

{

  //Sample Output for AT+CIPGSMLOC=1,1   ==> +CIPGSMLOC: 0,75.802460,26.848892,2019/04/23,08:32:35 //where 26.8488832 is Lattitude and 75.802460 is longitute

  int first_comma = responce.indexOf(','); //Find the position of 1st comma

  int second_comma = responce.indexOf(',', first_comma+1); //Find the position of 2nd comma

  int third_comma = responce.indexOf(',', second_comma+1); //Find the position of 3rd comma



  for(int i=first_comma+1; i<second_comma; i++) //Values form 1st comma to 2nd comma is Longitude

    Longitude = Longitude + responce.charAt(i);



  for(int i=second_comma+1; i<third_comma; i++) //Values form 2nd comma to 3rd comma is Latitude

    Latitude = Latitude + responce.charAt(i);



  Serial.println(Latitude); Serial.println(Longitude);

  Link = Link + Latitude + "," + Longitude; //Update the Link with latitude and Logitude values

  Serial.println(Link);

  lcd.setCursor(0,0);

  lcd.print("Lat: ");

  lcd.print(Latitude);

  lcd.setCursor(0,1);

  lcd.print("Lon: ");

  lcd.print(Longitude);

}



String incoming = "";



void loop() {



  if (SIM800.available()) { //Check if the SIM800 Module is telling anything

    char a = SIM800.read();

    Serial.write(a); //print what the module tells on serial monitor

    incoming = incoming + String(a);

    if (a == 13) //check for new line

    incoming =""; //clear the string if new line is detected

    incoming.trim(); //Remove /n or /r from the incomind data

 

    if (incoming=="RING") //If an incoming call is detected the SIM800 module will say "RING" check for it

    {

     Serial.println ("Sending sms"); delay(1000);

     responce = SIM800_send("ATH"); //Hand up the incoming call using ATH

     delay (1000);

     responce = SIM800_send("ATE0"); //Disable Echo

     delay (1000);



     responce = ""; Latitude=""; Longitude=""; //initialise all string to null

     SIM800.println("AT+CLBS=1,1"); delay(5000); //Request for location data

      while (SIM800.available())

      {

       char letter = SIM800.read();

       responce = responce + String(letter); //Store the location information in string responce

      }

      Serial.print("Result Obtained as:");   Serial.print(responce); Serial.println("*******");



     prepare_message(); delay(1000); //use prepare_message funtion to prepare the link with the obtained LAT and LONG co-ordinates



     SIM800.println("AT+CMGF=1"); //Set the module in SMS mode

     delay(1000);

   

     SIM800.println("AT+CMGS=\"+917986771492\""); //Send SMS to this number

     delay(1000);



     SIM800.println(Link); // we have send the string in variable Link

     delay(1000);



     SIM800.println((char)26);// ASCII code of CTRL+Z - used to terminate the text message

     delay(1000);

    }

  }



  if (Serial.available()) { //For debugging

    SIM800.write(Serial.read());

  }



}

______________________________________________________________________________________________

#include <SoftwareSerial.h> //Software Serial header to communicate with GSM module

#include<LiquidCrystal.h>

LiquidCrystal lcd(13,12,11,10,9,8);



SoftwareSerial SIM800(2, 3); // RX, TX



String Link = "The current Location is https://www.google.com/maps/place/";

String responce = "";

String Longitude = "";

String Latitude = "";



String SIM800_send(String incoming) //Function to communicate with SIM800 module

{

    SIM800.println(incoming); delay(100); //Print what is being sent to GSM module

    String result = "";

    while (SIM800.available()) //Wait for result

    {

    char letter = SIM800.read();

    result = result + String(letter); //combine char to string to get result

    }

 

return result; //return the result

}



void setup() {



  lcd.begin(16,2);

  Serial.begin(9600); //Serial COM for debugging

  SIM800.begin(9600); //Software serial called SIM800 to speak with SIM800 Module

  lcd.print("Location Tracker");

  lcd.setCursor(0,1);

  lcd.print("  Without GPS   ");

  delay(3000);

  lcd.clear();

   lcd.print("Connecting with");

  lcd.setCursor(0,1);

  lcd.print("    GSM Modem   ");

  delay(1000); //wait for serial COM to get ready



  responce = SIM800_send("ATE1"); //Enable Echo if not enabled by default

  Serial.print ("Responce:"); Serial.println(responce);

  delay(1000);



  responce = SIM800_send("AT+CGATT=1"); //Set the SIM800 in GPRS mode

  Serial.print ("Responce:"); Serial.println(responce);

  delay(1000);



  responce = SIM800_send("AT+SAPBR=3,1,\"CONTYPE\",\"GPRS\" "); //Activate Bearer profile

  Serial.print ("Responce:"); Serial.println(responce);

  delay(1000);



  responce = SIM800_send("AT+SAPBR=3,1,\"APN\",\"www\" "); //Set VPN options => 'RCMNET' 'www'

  Serial.print ("Responce:"); Serial.println(responce);

  delay(2000);

 

  responce = SIM800_send("AT+SAPBR=1,1"); //Open bearer Profile

  Serial.print ("Responce:"); Serial.println(responce); //Open bearer Profile

  delay(2000);



  responce = SIM800_send("AT+SAPBR=2,1"); //Get the IP address of the bearer profile

  Serial.print ("Responce:"); Serial.println(responce);

  delay(1000);

  
}



void prepare_message()

{

  //Sample Output for AT+CIPGSMLOC=1,1   ==> +CIPGSMLOC: 0,75.802460,26.848892,2019/04/23,08:32:35 //where 26.8488832 is Lattitude and 75.802460 is longitute

  int first_comma = responce.indexOf(','); //Find the position of 1st comma

  int second_comma = responce.indexOf(',', first_comma+1); //Find the position of 2nd comma

  int third_comma = responce.indexOf(',', second_comma+1); //Find the position of 3rd comma



  for(int i=first_comma+1; i<second_comma; i++) //Values form 1st comma to 2nd comma is Longitude

    Longitude = Longitude + responce.charAt(i);



  for(int i=second_comma+1; i<third_comma; i++) //Values form 2nd comma to 3rd comma is Latitude

    Latitude = Latitude + responce.charAt(i);



  Serial.println(Latitude); Serial.println(Longitude);

  Link = Link + Latitude + "," + Longitude; //Update the Link with latitude and Logitude values

  Serial.println(Link);

  lcd.setCursor(0,0);

  lcd.print("Lat: ");

  lcd.print(Latitude);

  lcd.setCursor(0,1);

  lcd.print("Lon: ");

  lcd.print(Longitude);

}



String incoming = "";



void loop() {



  if (SIM800.available()) { //Check if the SIM800 Module is telling anything

    char a = SIM800.read();

    Serial.write(a); //print what the module tells on serial monitor

    incoming = incoming + String(a);

    if (a == 13) //check for new line

    incoming =""; //clear the string if new line is detected

    incoming.trim(); //Remove /n or /r from the incomind data

 

    if (incoming=="RING") //If an incoming call is detected the SIM800 module will say "RING" check for it

    {

     Serial.println ("Sending sms"); delay(1000);

     responce = SIM800_send("ATH"); //Hand up the incoming call using ATH

     delay (1000);

     responce = SIM800_send("ATE0"); //Disable Echo

     delay (1000);



     responce = ""; Latitude=""; Longitude=""; //initialise all string to null

     SIM800.println("AT+CLBS=1,1"); delay(5000); //Request for location data

      while (SIM800.available())

      {

       char letter = SIM800.read();

       responce = responce + String(letter); //Store the location information in string responce

      }

      Serial.print("Result Obtained as:");   Serial.print(responce); Serial.println("*******");



     prepare_message(); delay(1000); //use prepare_message funtion to prepare the link with the obtained LAT and LONG co-ordinates



     SIM800.println("AT+CMGF=1"); //Set the module in SMS mode

     delay(1000);

   

     SIM800.println("AT+CMGS=\"+917986771492\""); //Send SMS to this number

     delay(1000);



     SIM800.println(Link); // we have send the string in variable Link

     delay(1000);



     SIM800.println((char)26);// ASCII code of CTRL+Z - used to terminate the text message

     delay(1000);

    }

  }



  if (Serial.available()) { //For debugging

    SIM800.write(Serial.read());

  }



}
____________________________________________________________________________________________

GSM WITH DHT SENSOR


#include <SoftwareSerial.h>
#include <String.h>

SoftwareSerial mySerial(7, 8);



void setup()
{
  mySerial.begin(19200);               // the GPRS baud rate   
  Serial.begin(19200);    // the GPRS baud rate
  delay(500);
  SIM900power();
 
}

void loop()
{

  if (Serial.available())
    switch(Serial.read())
   {
     case 'h':
       SubmitHttpRequest();
       break;

   }
  if (mySerial.available())
    Serial.write(mySerial.read());
}


void SubmitHttpRequest()
{
  String humidity = "1031";
  mySerial.println("AT+CSQ");
  delay(100);
  ShowSerialData();
  mySerial.println("AT+CGATT?");
  delay(100);
  ShowSerialData();
  mySerial.println("AT+SAPBR=3,1,\"CONTYPE\",\"GPRS\"");
  delay(1000);
  ShowSerialData();
  mySerial.println("AT+SAPBR=3,1,\"APN\",\"internet\"");
  delay(4000);
  ShowSerialData();
  mySerial.println("AT+SAPBR=1,1");
  delay(2000);
  ShowSerialData();
  mySerial.println("AT+HTTPINIT");
  delay(2000);
  ShowSerialData();
  mySerial.println("AT+HTTPPARA=\"URL\",\"www.xxx.com/ab/tx.php?h=" + humidity + "\"");
  delay(1000);
  ShowSerialData();
  mySerial.println("AT+HTTPACTION=0");
  delay(10000);
  ShowSerialData();
  mySerial.println("AT+HTTPREAD");
  delay(300);
  ShowSerialData();
  mySerial.println("");
  delay(100);
}



void ShowSerialData()
{
  while(mySerial.available()!=0)
    Serial.write(mySerial.read());
}

void SIM900power()
// software equivalent of pressing the GSM shield "power" button
{
  digitalWrite(9, HIGH);
  delay(1000);
  digitalWrite(9, LOW);
  delay(5000);
}





______________________________________________________________________________________

AWS commond 
__________


Import certs
>AT+USECMNG=0,0,"ROOT_CERT",1706
>[Send cert string, length 1706]
+USECMNG: 0,0,"ROOT_CERT","cb17e4..."
OK

>AT+USECMNG=0,1,"CLIENT_CERT",1206
>[Send cert string, length 1206]
+USECMNG: 0,1,"CLIENT_CERT","081dd1..."
OK

>AT+USECMNG=0,2,"PRIVATE_KEY",1654
>[Send cert string, length 1654]
+USECMNG: 0,2,"PRIVATE_KEY","bcb22b..."
OK

Profile 1, cert validation level 1
AT+USECPRF=1,0,1
OK

Profile 1, server can use any version for the connection
AT+USECPRF=1,1,0
OK

Profile 1, use named root cert
AT+USECPRF=1,3,"ROOT_CERT"
OK

Profile 1, use named client cert
AT+USECPRF=1,5,"CLIENT_CERT"
OK

Profile 1, use named private key
AT+USECPRF=1,6,"PRIVATE_KEY"
OK

Create TCP socket
AT+USOCR=6
+USOCR: 0
OK
__________________________________________________________________
MKR GSM 14000

// This example uses an Arduino MKR GSM 1400 board
// to connect to shiftr.io.
//
// IMPORTANT: This example uses the new MKRGSM library.
//
// You can check on your device after a successful
// connection here: https://shiftr.io/try.
//
// by Sandeep Mistry
// https://github.com/256dpi/arduino-mqtt

#include <MKRGSM.h>
#include <MQTT.h>

const char pin[]      = "123456";
const char apn[]      = "airtelgprs.com";
const char login[]    = "login";
const char password[] = "password";

GSMClient net;
GPRS gprs;
GSM gsmAccess;
MQTTClient client;

unsigned long lastMillis = 0;

void connect() {
  // connection state
  bool connected = false;

  Serial.print("connecting to cellular network ...");

  // After starting the modem with gsmAccess.begin()
  // attach to the GPRS network with the APN, login and password
  while (!connected) {
    if ((gsmAccess.begin(pin) == GSM_READY) &&
        (gprs.attachGPRS(apn, login, password) == GPRS_READY)) {
      connected = true;
    } else {
      Serial.print(".");
      delay(1000);
    }
  }

  Serial.print("\nconnecting...");
  while (!client.connect("arduino", "try", "try")) {
    Serial.print(".");
    delay(1000);
  }

  Serial.println("\nconnected!");

  client.subscribe("/hello");
  // client.unsubscribe("/hello");
}

void messageReceived(String &topic, String &payload) {
  Serial.println("incoming: " + topic + " - " + payload);
}

void setup() {
  Serial.begin(115200);

  // Note: Local domain names (e.g. "Computer.local" on OSX) are not supported by Arduino.
  // You need to set the IP address directly.
  client.begin("broker.shiftr.io", net);
  client.onMessage(messageReceived);

  connect();
}

void loop() {
  client.loop();

  if (!client.connected()) {
    connect();
  }

  // publish a message roughly every second.
  if (millis() - lastMillis > 1000) {
    lastMillis = millis();
    client.publish("/hello", "world");
  }
}



Previous returned socket 0, using SSL/TLS, apply profile 1
AT+USOSEC=0,1,1
OK



