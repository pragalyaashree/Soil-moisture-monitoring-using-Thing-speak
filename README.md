# Uploading Soil moisture sensor data in Thing Speak cloud

# AIM:
To upload the Soil moisture senso data in the Thing speak using an ESP32 controller.

# Apparatus required:
ESP32 Controller  </br>
HC-SR04 Ultrasonic sensor module </br>
Power supply </br>
Connecting wires </br>
Bread board </br>

# PROCEDURE:
## Arduino IDE
Step1:Open the Arduino IDE </br>
Step2: Go to sketch- include library – manage libraries file and install esp32 and thing speak library file </br>
Step3:Go to file and select new file option </br>
Step4:Type the program and update the thing speak channel ID, API key, wifi password and ID </br>
Step5:Go to file and select save option to save the program </br>
Step6:Go to sketch and select verify or compile options </br>
Step7:If no error Hex file will be generated in the temporary folder </br>
Step8: Connect all the components as per the circuit diagram </br>
Step9: Connect the programming cable with esp32 and PC.  </br>
Step10: Check the jumper position and connect 4 & 5 of P4.  </br>
Step11. Upload the program in the esp32. </br>
Step12 Press the boot button in ESP32 and then press and release the reset button after release the boot button </br>
Step13 Check the output in the cloud </br>

# THEORY:

### What is IoT?

Internet of Things (IoT) describes an emerging trend where a large number of embedded devices (things) are connected to the Internet. These connected devices communicate with people and other things and often provide sensor data to cloud storage and cloud computing resources where the data is processed and analyzed to gain important insights. Cheap cloud computing power and increased device connectivity is enabling this trend.IoT solutions are built for many vertical applications such as environmental monitoring and control, health monitoring, vehicle fleet monitoring, industrial monitoring and control, and home automation

![image](https://user-images.githubusercontent.com/71547910/235334044-c01d4261-d46f-4f62-b07f-72a7b6fce5d5.png)

### Sending Data to Cloud with ESP32 and ThingSpeak

ThingSpeak is an Internet of Things (IoT) analytics platform that allows users to collect, analyze, and visualize data from sensors or devices connected to the Internet. It is a cloud-based platform that provides APIs for storing and retrieving data, as well as tools for data analysis and visualization.The Internet of Things ( or IoT) is a network of interconnected computing devices such as digital machines, automobiles with built-in sensors, or humans with unique identifiers and the ability to communicate data over a network without human intervention.Hello readers, I hope you all are doing great. In this tutorial, we will learn how to send sensor readings from ESP32 to the ThingSpeak cloud. Here we will use the ESP32’s internal sensor like hall-effect sensor and temperature sensor to observe the data and then will share that data cloud.

### What is ThingSpeak?

![image](https://user-images.githubusercontent.com/71547910/235333909-29d2e831-9fe5-4afd-b18d-f1e5d2e32518.png)

It is an open data platform for IoT (Internet of Things). ThingSpeak is a web service operated by MathWorks where we can send sensor readings/data to the cloud. We can also visualize and act on the data (calculate the data) posted by the devices to ThingSpeak. The data can be stored in either private or public channels.ThingSpeak is frequently used for internet of things prototyping and proof of concept systems that require analytics.

### Features Of ThingSpeak

ThingSpeak service enables users to share analyzed data through public channels: </br>
ThingSpeak allows professionals to prepare and analyze data for their businesses: </br>
ThingSpeak updates various ThingSpeak channels using MQTT and REST APIs: </br>
Easily configure devices to send data to ThingSpeak using popular IoT protocols. </br>
Visualize your sensor data in real-time. </br>
Aggregate data on-demand from third-party sources. </br>
Use the power of MATLAB to make sense of your IoT data. </br>
Run your IoT analytics automatically based on schedules or events. </br>
Prototype and build IoT systems without setting up servers or developing web software.</br>
Automatically act on your data and communicate using third-party services like Twilio® or Twitter®</br>

![image](https://user-images.githubusercontent.com/71547910/235334056-3ba9579f-2f62-43b1-a714-8fde6cf9ef32.png)


# PROGRAM:
     #include <SoftwareSerial.h>
      #include <Adafruit_Sensor.h>
    /*
      */
        #define triggerpin 8                 // trigger pin connected to the ultrosonic sensor 
           #define echopin 9                   // techo pin connected to the ultrosonic sensor 
          int duration, inches, cm;
            String inputString = "";         // a String to hold incoming data
            bool stringComplete = false;     // whether the string is complete
             long old_time=millis();
             long new_time;
             long uplink_interval=30000;      //ms
             bool time_to_at_recvb=false;
             bool get_LA66_data_status=false;
             bool network_joined_status=false;
             SoftwareSerial ss(10, 11);       // Arduino RX, TX ,
           char rxbuff[128];
             uint8_t rxbuff_index=0;
           void setup() {
            // initialize serial
            pinMode(triggerpin,OUTPUT);
            pinMode(echopin,INPUT);
            Serial.begin(9600);
           ss.begin(9600);
            ss.listen();
            // reserve 200 bytes for the inputString:
           inputString.reserve(200);
           dht.begin();
            sensor_t sensor;
           dht.temperature().getSensor(&sensor);
           dht.humidity().getSensor(&sensor);
          ss.println("ATZ");//reset LA66
           }
        void loop() {
        new_time = millis();
        if((new_time-old_time>=uplink_interval)&&(network_joined_status==1)){
        old_time = new_time;
        get_LA66_data_status=false;
        //ultrasonic sensor
       HC04();    
          char sensor_data_buff[128]="\0";
          //confirm status,Fport,payload length,payload(HEX)
          //--------------perfectly worked for dht11 and ultrasonic sensor-------------
             //-------------for 0,2,2 payload length with data 
               snprintf(sensor_data_buff,128,"AT+SENDB=%d,%d,%d,%02X%02X",0,2,2,(short)(inches),(short)(cm));
             ss.println(sensor_data_buff);
            }
         if(time_to_at_recvb==true){
            time_to_at_recvb=false;
         get_LA66_data_status=true;
           delay(1000);
    ss.println("AT+CFG");    
      }
    while ( ss.available()) {
    // get the new byte:
    char inChar = (char) ss.read();
    // add it to the inputString:
    inputString += inChar;
    rxbuff[rxbuff_index++]=inChar;
    if(rxbuff_index>128)
    break;
    // if the incoming character is a newline, set a flag so the main loop can
    // do something about it:
    if (inChar == '\n' || inChar == '\r') {
      stringComplete = true;
      rxbuff[rxbuff_index]='\0';
      if(strncmp(rxbuff,"JOINED",6)==0){
        network_joined_status=1;
      }
      if(strncmp(rxbuff,"Dragino LA66 Device",19)==0){
        network_joined_status=0;
      }
      if(strncmp(rxbuff,"Run AT+RECVB=? to see detail",28)==0){
        time_to_at_recvb=true;
        stringComplete=false;
        inputString = "\0";
      }
      if(strncmp(rxbuff,"AT+RECVB=",9)==0){       
        stringComplete=false;
        inputString = "\0";
        Serial.print("\r\nGet downlink data(FPort & Payload) ");
        Serial.println(&rxbuff[9]);
      }
      rxbuff_index=0;
      if(get_LA66_data_status==true){
        stringComplete=false;
        inputString = "\0";
      }
      }
      }
       while ( Serial.available()) {
    // get the new byte:
    char inChar = (char) Serial.read();
    // add it to the inputString:
    inputString += inChar;
    // if the incoming character is a newline, set a flag so the main loop can
    // do something about it:
    if (inChar == '\n' || inChar == '\r') {
      ss.print(inputString);
      inputString = "\0";
       }
     }
     // print the string when a newline arrives:
     if (stringComplete) {
    Serial.print(inputString);
    // clear the string:
    inputString = "\0";
    stringComplete = false;
     }
    }
      void HC04()
     {
        digitalWrite(triggerpin, LOW);
         delayMicroseconds(2);
         digitalWrite(triggerpin, HIGH);
          delayMicroseconds(10);
         digitalWrite(triggerpin, LOW);
         duration = pulseIn(echopin, HIGH);
         inches = microsecondsToInches(duration);
         cm = microsecondsToCentimeters(duration.Serial.print(inches);
        Serial.print("in, ");
        Serial.print(cm);
        Serial.print("cm");
        Serial.println();
      //  delay(1000); //Delay of 1 second for ease of viewing
       }
    long microsecondsToInches(long microseconds) {
      return microseconds / 74 / 2;
      }
    long microsecondsToCentimeters(long microseconds) {
       return microseconds / 29 / 2;
        }
# CIRCUIT DIAGRAM:

![cd2](https://github.com/Yuvakrishna0/Soil-moisture-monitoring-using-Thing-speak/assets/117915037/71fe41c9-d1c3-4dde-a01d-daa70cd2aaff)

# OUTPUT:
## OUTPUT1:
![OUT1](https://github.com/Yuvakrishna0/Soil-moisture-monitoring-using-Thing-speak/assets/117915037/e6d3c333-3b99-40a4-a14a-aabbfd515329)

## OUTPUT2:
![OUT2](https://github.com/Yuvakrishna0/Soil-moisture-monitoring-using-Thing-speak/assets/117915037/bfce47a5-7e8b-459f-9ca8-10008ba64b58)


# RESULT:
Thus the soil moisture sensor values are uploaded in the Thing speak using ESP32 controller.

