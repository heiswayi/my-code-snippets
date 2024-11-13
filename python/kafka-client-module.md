Kafka client module for Python app development. This module supports both for Kafka producer and consumer. This is part of my `utils` modules.

Prerequisite:
```bash
pip install confluent_kafka
```

kafka_client.py
```python
from confluent_kafka import Producer, Consumer, KafkaError
import json
import logging

class KafkaProducerClient:
    def __init__(
        self,
        bootstrap_servers,
        topic,
        logger=logging.getLogger("KafkaProducerClient")
    ):
        self.topic = topic
        self.producer = Producer({'bootstrap.servers': bootstrap_servers})
        self.logger = logger
        
    def delivery_report(self, err, msg):
        if err:
            self.logger.error(f"Message delivery failed: {err}")
        else:
            self.logger.info(f"Message delivered to {msg.topic()} [{msg.partition()}]")

    def produce(self, key, value):
        try:
            self.producer.produce(
                self.topic, 
                key=str(key), 
                value=json.dumps(value),
                callback=self.delivery_report
            )
            self.producer.flush()
        except Exception as e:
            self.logger.error(f"Failed to send message: {e}")

class KafkaConsumerClient:
    def __init__(
        self,
        bootstrap_servers,
        group_id,
        topics,
        message_callback=None,
        logger=logging.getLogger("KafkaConsumerClient")
    ):
        self.consumer = Consumer({
            'bootstrap.servers': bootstrap_servers,
            'group.id': group_id,
            'auto.offset.reset': 'earliest'
        })
        self.consumer.subscribe(topics)
        self.message_callback = message_callback
        self.logger = logger

    def consume_messages(self):
        try:
            while True:
                msg = self.consumer.poll(timeout=1.0)
                if msg is None:
                    continue
                if msg.error():
                    if msg.error().code() == KafkaError._PARTITION_EOF:
                        self.logger.info(f"End of partition reached {msg.topic()}/{msg.partition()}")
                    elif msg.error():
                        self.logger.error(msg.error())
                else:
                    key = msg.key().decode('utf-8') if msg.key() else None
                    value = json.loads(msg.value().decode('utf-8'))
                    self.process_message(key, value)
        except Exception as e:
            self.logger.error(f"Error in consuming messages: {e}")
        finally:
            self.consumer.close()

    def process_message(self, key, value):
        # Override this method to define how each message should be processed
        # logging.info(f"Received message: key={key}, value={value}")
        if self.message_callback:
            self.message_callback(key, value)
        
    def close(self):
        self.consumer.close()

# Usage example:
# producer = KafkaProducerClient(bootstrap_servers='localhost:9092', topic='your_topic')
# producer.produce(key="key1", value={"sample": "data"})

# consumer = KafkaConsumerClient(bootstrap_servers='localhost:9092', group_id='your_group', topics=['your_topic'], message_callback=None)
# consumer.consume_messages()
```
