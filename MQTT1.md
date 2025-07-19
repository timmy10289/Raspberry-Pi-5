# 使用Raspberry-Pi-5 跟esp8266 跟三線性RGB燈泡 做MQTT測試    
## 將三線性 RGB 燈泡的引腳連接到 ESP8266 的 GPIO 引腳上   
紅色引腳連接到 D1，綠色引腳連接到 D2，藍色引腳連接到 D3 接地連接到G  
## 在 Raspberry Pi 上設置 MQTT 代理
sudo apt update  
sudo apt install mosquitto mosquitto-clients  
sudo systemctl start mosquitto  
sudo systemctl enable mosquitto  
## 在 ESP8266 上編寫控制代碼  

```python
#include <ESP8266WiFi.h>
#include <PubSubClient.h>

// WiFi 設定
const char* ssid = "asd"; // 替換為您的 WiFi 名稱
const char* password = "bnm85246"; // 替換為您的 WiFi 密碼

// MQTT 設定
const char* mqtt_server = "192.168.137.182"; // 替換為您的 Raspberry Pi 的 IP 地址
const char* mqtt_topic = "rgb/control";

WiFiClient espClient;
PubSubClient client(espClient);

// RGB 引腳設定
const int redPin = D1;
const int greenPin = D2;
const int bluePin = D3;

void setup() {
  pinMode(redPin, OUTPUT);
  pinMode(greenPin, OUTPUT);
  pinMode(bluePin, OUTPUT);
  
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
}

void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);
  
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  
  Serial.println(" connected");
  Serial.print("ESP8266 IP address: ");
  Serial.println(WiFi.localIP()); // 顯示 ESP8266 的 IP 地址
}

void callback(char* topic, byte* payload, unsigned int length) {
  String message;
  for (int i = 0; i < length; i++) {
    message += (char)payload[i];
  }
  
  Serial.print("Message received: "); // 顯示接收到的消息
  Serial.println(message);
  
  if (message == "RED") {
    setColor(255, 0, 0);
  } else if (message == "GREEN") {
    setColor(0, 255, 0);
  } else if (message == "BLUE") {
    setColor(0, 0, 255);
  } else if (message == "OFF") {
    setColor(0, 0, 0);
  }
}

void setColor(int red, int green, int blue) {
  analogWrite(redPin, red);
  analogWrite(greenPin, green);
  analogWrite(bluePin, blue);
}

void reconnect() {
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    if (client.connect("ESP8266Client")) {
      Serial.println("connected");
      client.subscribe(mqtt_topic);
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      delay(2000);
    }
  }
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();
}

```
## 在 Raspberry Pi 上發送 MQTT 消息  
mosquitto_pub -h localhost -t rgb/control -m "RED"  
mosquitto_pub -h localhost -t rgb/control -m "GREEN"  
mosquitto_pub -h localhost -t rgb/control -m "BLUE"  
mosquitto_pub -h localhost -t rgb/control -m "OFF"  

## 使用 Python 腳本(可選)  
```python
import paho.mqtt.client as mqtt

broker = "YOUR_RASPBERRY_PI_IP"
topic = "rgb/control"

client = mqtt.Client()
client.connect(broker)

client.publish(topic, "RED")   # 發送紅色
client.publish(topic, "GREEN") # 發送綠色
client.publish(topic, "BLUE")  # 發送藍色
client.publish(topic, "OFF")   # 關閉燈泡
```
