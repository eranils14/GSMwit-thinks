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




