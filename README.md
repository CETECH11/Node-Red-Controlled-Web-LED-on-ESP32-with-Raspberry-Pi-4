# Node-Red-Controlled-Web-LED-on-ESP32-with-Raspberry-Pi-4

![alt text](https://hackster.imgix.net/uploads/attachments/1628693/_kw9jefV2dw.blob?auto=compress%2Cformat&w=900&h=675&fit=min)

In this tutorial, you will learn how to use Node-RED, a visual programming tool for the Internet of Things (IoT), to control an LED on an ESP32 board with a Raspberry Pi as the MQTT broker. MQTT is a lightweight and simple messaging protocol that allows devices to communicate with each other over a network.

You will need the following components for this project:

ESP32 development board
USB cable to connect the ESP32 to your computer
Raspberry Pi with Node-RED
Computer with Arduino IDE and PubSubClient library installed

The first step is to create a device on the Qubitro platform. A device represents your physical device (Raspberry Pi) on the cloud. You need to create a device to obtain the MQTT credentials and topics for your Raspberry Pi.

To create a device on Qubitro, follow these steps:

1. Log in to your Qubitro account and create a new project

2. Then go to the Devices page, select MQTT as the communication protocol, and click Next.

3. Enter all the details.

4. Copy the Device ID, Device Token, Hostname, Port, Publish Topic, and Subscribe Topic. You will need these values later in the code. Click Finish.

You have successfully created a device on Qubitro. You can see your device on the Devices page.


![alt text](https://hackster.imgix.net/uploads/attachments/1628673/image_QMOxK4WHXg.png?auto=compress%2Cformat&w=740&h=555&fit=max)

The ESP32 is a powerful and versatile microcontroller that can run Arduino code. You will use the Arduino IDE to program the ESP32 and make it communicate with the MQTT broker using the PubSubClient library.

To install the ESP32 board in Arduino IDE, you can follow the instructions in this tutorial or use the steps below:

Open the preferences window from the Arduino IDE: File > Preferences.
Go to the “Additional Board Manager URLs” field and enter the following URL: https://dl.espressif.com/dl/package_esp32_index.json.
Open Boards Manager (Tools > Board > Boards Manager), search for ESP32, and click the install button for the “ESP32 by Espressif Systems”.
Select your ESP32 board from Tools > Board menu after installation.
Open the library manager from Sketch > Include Library > Manage Libraries.
Search for PubSubClient and click the install button for the “PubSubClient by Nick O’Leary”.
Restart your Arduino IDE after installation.

![alt text](https://hackster.imgix.net/uploads/attachments/1628689/whatsapp_image_2023-09-10_at_12_21_15_fY0EsCS3CV.jpg?auto=compress%2Cformat&w=740&h=555&fit=max)

The LED is a simple device that emits light when current flows through it. You will connect the LED to one of the GPIO pins of the ESP32 and control its state (on or off) with MQTT messages.

In my case I'm going to use the onboard LED in the ESP32 Dev board.
The code for the ESP32 will do the following tasks:

Connect to your Wi-Fi network
Connect to the Qubitro MQTT broker on Raspberry Pi
Receive messages from “output” and turn on or off the LED accordingly
You can copy and paste the code below into your Arduino IDE. Make sure to replace <your_ssid>, <your_password>, <your_Qubtro_Credientials> with your own values.

#include <WiFi.h>
#define DEBUG_SW 1
#include <PubSubClient.h>

//Relays for switching appliances
#define Relay1            2

int switch_ON_Flag1_previous_I = 0;

// Update these with values suitable for your network.

const char* ssid = "ELDRADO";
const char* password = "amazon123";
const char* mqtt_server = "broker.qubitro.com"; // Local IP address of Raspberry Pi

const char* username = "";
const char* pass = "";

// Subscribed Topics
#define sub1 "output"

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;


// Connecting to WiFi Router

void setup_wifi()
{

  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length)
{
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");


  if (strstr(topic, sub1))
  {
    for (int i = 0; i < length; i++)
    {
      Serial.print((char)payload[i]);
    }
    Serial.println();
    // Switch on the LED if an 1 was received as first character
    if ((char)payload[0] == 'f')
    {
      digitalWrite(Relay1, LOW);   // Turn the LED on (Note that LOW is the voltage level
      // but actually the LED is on; this is because
      // it is active low on the ESP-01)
    } else {
      digitalWrite(Relay1, HIGH);  // Turn the LED off by making the voltage HIGH
    }
  }
  else
  {
    Serial.println("unsubscribed topic");
  }

}


// Connecting to MQTT broker

void reconnect()
{
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str() , username, pass)) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("outTopic", "hello world");
      // ... and resubscribe
      client.subscribe(sub1);
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}



void setup()
{

  pinMode(Relay1, OUTPUT);
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
}

void loop()
{

  if (!client.connected())
  {
    reconnect();
  }
  client. Loop();
}
After writing the code, upload it to your ESP32 board by selecting the right board and port from the Tools menu and clicking the upload button.

![alt text](https://hackster.imgix.net/uploads/attachments/1518136/8_tJuwoRM3dI.JPG?auto=compress%2Cformat&w=740&h=555&fit=max)

You must check out [PCBWAY](https://www.pcbway.com/) for ordering PCBs online for cheap!

You get 10 good-quality PCBs manufactured and shipped to your doorstep for cheap. You will also get a discount on shipping on your first order. Upload your Gerber files onto PCBWAY to get them manufactured with good quality and quick turnaround time. [PCBWay](https://www.pcbway.com/) now could provide a complete product solution, from design to enclosure production. Check out their online Gerber viewer function. With reward points, you can get free stuff from their gift shop.
