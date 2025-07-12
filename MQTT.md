# 建立MQTT Broker


這裡使用mosquitto  
sudo apt update  
sudo apt install mosquitto mosquitto-clients  
檢查mosquitto 有沒有運行  
sudo systemctl status mosquitto  

## 修改配置文件

Mosquitto 的配置文件位於 /etc/mosquitto/mosquitto.conf。你可以編輯這個文件來更改設置，如開放端口、設置密碼等。  
cd /  
cd etc/mosquitto  

使用nano 編輯  用完 按crtl+x 按Y儲存 再按enter  
sudo nano /etc/mosquitto/mosquitto.conf   

使用vim 編輯 在 vim 中，按 i 進入插入模式進行編輯。編輯完成後，按 Esc 鍵，然後輸入 :wq 並按 Enter 保存並退出。  
sudo vim /etc/mosquitto/mosquitto.conf  

把裡面的文字 改成這樣  
persistence true  
persistence_location /var/lib/mosquitto/  

改完  
sudo systemctl restart mosquitto  

## 安裝python 環境  
cd /  
cd /home/timmy10289  反正隨便找一個方便尋找的地方  

sudo apt update  
sudo apt install python3-venv  

創建一個虛擬環境  
python3 -m venv myenv  

激活他 每次要使用都要激活  
source myenv/bin/activate  

要退出的話  
deactivate  

## 安裝MQTT並測試  
使用 localhost：如果你的應用程序（如發布者和訂閱者）與 MQTT Broker 在同一台設備上運行，可以將 Broker 設定為 localhost 或 127.0.0.1  
pip install paho-mqtt  

建立一個發佈消息的程式  
touch publisher.py  
sudo nano publisher.py  

```python  
import paho.mqtt.client as mqtt

broker = "localhost"  # 或者替換為你的 MQTT broker IP
topic = "test/topic"

client = mqtt.Client()
client.connect(broker)

client.publish(topic, "Hello, MQTT!")
client.disconnect()

```  
建立一個訂閱消息的程式  
touch subscriber.py   
sudo nano subscribe.py   

```python  
import paho.mqtt.client as mqtt

broker = "localhost"  # 或者替換為你的 MQTT broker IP
topic = "test/topic"

def on_message(client, userdata, message):
    print(f"Received message '{message.payload.decode()}' on topic '{message.topic}'")

client = mqtt.Client()
client.connect(broker)
client.subscribe(topic)
client.on_message = on_message

client.loop_start()

```  





