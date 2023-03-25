---
title: 'Use Kafka Consumer As Websocket Server'
date: '2023-03-25T13:06:38+08:00'
draft: false
description: 'integrate kafka consumer with websocket server'
author: 'Yan'
tags: ["python", "kafka", "websocket"]
theme: "dark"
---

# Introduction

In this post, I will show you how to integrate kafka consumer with websocket server. The websocket server will send message to client when it receives message from kafka consumer. [source code][https://github.com/claudezss/heizer/tree/main/samples/websockets]

# Steps

1. [Spin up a kafka cluster](https://docs.confluent.io/current/quickstart/ce-docker-quickstart.html#ce-docker-quickstart)
    
    ```yaml
   version: '2'

   services:
     zookeeper:
    
       image: confluentinc/cp-zookeeper:latest
       environment:
         ZOOKEEPER_CLIENT_PORT: 2181
         ZOOKEEPER_TICK_TIME: 2000
   
     kafka:
       image: confluentinc/cp-kafka:latest
       depends_on:
         - zookeeper
       ports:
         - 9092:9092
       environment:
         KAFKA_BROKER_ID: 1
         KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
         KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
         KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
         KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
         KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
   
     kafka-ui:
       image: provectuslabs/kafka-ui:latest
       ports:
         - 8080:8080
       depends_on:
         - kafka
       environment:
         KAFKA_CLUSTERS_0_NAME: local
         KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: "kafka:29092"
    ```

2. Install [heizer](https://github.com/claudezss/heizer) (a python library for creating kafka consumer/producer easily)

    ```bash
    pip install --pre heizer
    pip install websockets
    ```
3. Create a kafka topic `topic.test`
   you can create new topic in [kafka-ui] (http://localhost:8080/ui/clusters/local/topics)
   
   ![Create Topic](/kafka-websocket/create-topic.png)

4. Create websocket server `server.py` on `ws://localhost:8001`. [source code](https://github.com/claudezss/heizer/blob/main/samples/websockets/server.py)
   ```python
   import asyncio
   import websockets
   
   from heizer import HeizerConfig, HeizerTopic, consumer
   
   consumer_config = HeizerConfig(
       {
           "bootstrap.servers": "0.0.0.0:9092",
           "group.id": "test_group",
           "auto.offset.reset": "earliest",
       }
   )
   
   topics = [HeizerTopic(name="topic.test")]
   
   
   @consumer(topics=topics, config=consumer_config, is_async=True)
   async def handler(message, websocket, *args, **kwargs):
       await websocket.send(message.value)
   
   
   async def main():
       async with websockets.serve(handler, "", 8001):
           await asyncio.Future()
   
   
   if __name__ == "__main__":
       asyncio.run(main())

   ```
   and run `python server.py`
   ```bash
   python server.py
   `````

5. Create websocket client `client.html` and listen to `ws://localhost:8001`. [source code](https://github.com/claudezss/heizer/blob/main/samples/websockets/client.html)
   ```html
   <!DOCTYPE html>

   <html lang="en">
   
   <head>
   
       <meta charset="UTF-8">
   
       <meta http-equiv="X-UA-Compatible" content="IE=edge">
   
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
   
       <title>WebSocker Client</title>
   
   </head>
   
   <body>
   
   <div id="msgs"></div>
   
   </body>
   
   <script>
       var msgs = [];
   
       const socket = new WebSocket('ws://localhost:8001');
   
       socket.addEventListener('open', function (event) {
   
           socket.send('Connection Established');
   
       });
   
       const msg_box = document.getElementById("msgs");
   
       socket.addEventListener('message', function (event) {
           msgs.push(event.data);
           console.log(event.data);
   
           msg_box.innerHTML = msgs.join("<br>");
   
       });
   
   </script>
   </html>
   ```
    and open `client.html` in browser

6. Publish message to kafka topic `topic.test` and you will see the message in browser
   
   by [kafka-ui](http://localhost:8080/ui/clusters/local/all-topics/topic.test)

   ![Publish Message](/kafka-websocket/ublish-kafka-msg.png)


7. You will see the message in browser
   
   ![Message In Browser](/kafka-websocket/message-in-browser.png)
