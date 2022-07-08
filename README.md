# MQTT in Python (Paho)

## 環境
```shell
 > python3 --version
 Python 3.9.13

> pip3 install paho-mqtt
...
Successfully installed paho-mqtt-1.6.1
```
Public MQTT 5 Broker | EMQ 
https://www.emqx.com/en/mqtt/public-mqtt5-broker

```yml
Access Information
Broker: broker.emqx.io
TCP Port: 1883
Websocket Port: 8083
TCP/TLS Port: 8883
Websocket/TLS Port: 8084
Certificate Authority: broker.emqx.io-ca.crt
```

## 手順

### Paho MQTT client を import
```python
from paho.mqtt import client as mqtt_client
```

### Brokerへの接続に必要なパラメータを設定
```python
broker = 'broker.emqx.io'
port = 1883
topic = 'saitakupon/test'
client_id = f'python-mqtt-{random.randint(0, 1000)}' # IDをランダム生成
username = '任意のユーザ名'
password = '任意のパスワード'
```

### 接続
```python
def connect_mqtt():
    def on_connect(client, userdata, flags, rc): # 接続確認
        if rc == 0:
            print("Connected to MQTT Broker!")
        else:
            print("Failed to connect, return code %d\n", rc)
    
    client = mqtt_client.Client(client_id)
    client.username_pw_set(username, password)
    client.on_connect = on_connect
    client.connect(broker, port)
    return client
```

### 送信（Publish）
指定のトピックへ1秒おきに`messages: {msg_count}`を送信
```python
def publish(client):
    msg_count = 0
    while True:
        time.sleep(1)
        msg = f"messages: {msg_count}"
        result = client.publish(topic, msg)
        # result: [0, 1]
        status = result[0]
        if status == 0:
            print(f"Send `{msg}` to topic `{topic}`")
        else:
            print(f"Failed to send message to topic {topic}")
        msg_count += 1
```

### 受信（Subscribe）
```python
def on_message(client, userdata, msg): # subscribe時に結果をprint
        print(f"Received `{msg.payload.decode()}` from `{msg.topic}` topic")

    client.subscribe(topic)
    client.on_message = on_message
```

### 全体像

### Publish
```python
import random
import time

from paho.mqtt import client as mqtt_client


broker = 'broker.emqx.io'
port = 1883
topic = 'saitakupon/test'
client_id = f'python-mqtt-{random.randint(0, 1000)}'  # IDをランダム生成
username = 'saitakupon_pub'
password = 'test'


def connect_mqtt():
    def on_connect(client, userdata, flags, rc):  # 接続確認
        if rc == 0:
            print("Connected to MQTT Broker!")
        else:
            print("Failed to connect, return code %d\n", rc)

    client = mqtt_client.Client(client_id)
    client.username_pw_set(username, password)
    client.on_connect = on_connect
    client.connect(broker, port)
    return client


def publish(client):
    msg_count = 0
    while True:
        time.sleep(1)
        msg = f"messages: {msg_count}"
        result = client.publish(topic, msg)
        # result: [0, 1]
        status = result[0]
        if status == 0:
            print(f"Send `{msg}` to topic `{topic}`")
        else:
            print(f"Failed to send message to topic {topic}")
        msg_count += 1


def run():
    client = connect_mqtt()
    client.loop_start()
    publish(client)


if __name__ == '__main__':
    run()
```

### Subscribe
```python
import random
import time

from paho.mqtt import client as mqtt_client


broker = 'broker.emqx.io'
port = 1883
topic = 'saitakupon/test'
client_id = f'python-mqtt-{random.randint(0, 1000)}'  # IDをランダム生成
username = 'saitakupon_sub'
password = 'test'


def connect_mqtt():
    def on_connect(client, userdata, flags, rc):  # 接続確認
        if rc == 0:
            print("Connected to MQTT Broker!")
        else:
            print("Failed to connect, return code %d\n", rc)

    client = mqtt_client.Client(client_id)
    client.username_pw_set(username, password)
    client.on_connect = on_connect
    client.connect(broker, port)
    return client


def subscribe(client: mqtt_client):  # subscribe時に結果をprint
    def on_message(client, userdata, msg):
        print(f"Received `{msg.payload.decode()}` from `{msg.topic}` topic")

    client.subscribe(topic)
    client.on_message = on_message


def run():
    client = connect_mqtt()
    subscribe(client)
    client.loop_forever()


if __name__ == '__main__':
    run()

```

## 実行結果

https://user-images.githubusercontent.com/58939922/177941179-da1f730d-92b5-4a30-9802-775c10df176e.mov

## 参考
How to use MQTT in Python (Paho) | EMQ 
https://www.emqx.com/en/blog/how-to-use-mqtt-in-python
