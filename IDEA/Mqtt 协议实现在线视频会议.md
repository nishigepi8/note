简介
该教程介绍如何使用 Python 和 MQTT 实现一个简单的在线视频会议系统，其中包括推送端和订阅端。推送端通过摄像头捕获视频帧，使用 MQTT 将图像数据实时推送到指定主题。订阅端订阅相同的 MQTT 主题，接收图像并在本地显示，同时计算并显示帧率。

推送端
1. 准备环境
确保已经安装必要的 Python 库：

pip install opencv-python numpy paho-mqtt
1
2. 推送端代码
# pusher.py
import base64
import time
import multiprocessing
import paho.mqtt.client as mqtt
import numpy as np
import cv2

# MQTT配置
mqtt_broker = "localhost"
mqtt_port = 1883
mqtt_topic = "your_mqtt_topic"

# 初始化MQTT客户端
client = mqtt.Client()
client.connect(mqtt_broker, mqtt_port, 60)

def read_and_publish(output_queue):
    cap = cv2.VideoCapture(2)
    try:
        while True:
            ret, frame = cap.read()
            if ret:
                _, img_encoded = cv2.imencode('.jpg', frame)
                img_base64 = base64.b64encode(img_encoded.tobytes()).decode('utf-8')
                # 将base64图像放入队列
                output_queue.put(img_base64)
    except Exception as e:
        print(f"Error: {e}")
    finally:
        # Release the capture object when done
        cap.release()

def publish_to_mqtt(input_queue):
    while True:
        if not input_queue.empty():
            img_base64 = input_queue.get()
            # 发布到MQTT通道
            client.publish(mqtt_topic, img_base64)

def main():
    # 使用队列在主进程和子进程之间传递数据
    output_queue = multiprocessing.Queue()
    input_queue = multiprocessing.Queue()

    # 启动子进程
    process_read_publish = multiprocessing.Process(target=read_and_publish, args=(output_queue,))
    process_mqtt_publish = multiprocessing.Process(target=publish_to_mqtt, args=(input_queue,))

    # 启动子进程
    process_read_publish.start()
    process_mqtt_publish.start()

    while True:
        start = time.time()

        # 从队列中获取数据
        while not output_queue.empty():
            img_base64 = output_queue.get()
            # 将数据放入用于MQTT发布的队列
            input_queue.put(img_base64)

        end = time.time()
        print(end - start)

    # 不要忘记在结束时释放资源
    process_read_publish.terminate()
    process_mqtt_publish.terminate()
    process_read_publish.join()
    process_mqtt_publish.join()

if __name__ == "__main__":
    main()

订阅端
1. 准备环境
确保已经安装必要的 Python 库：

pip install opencv-python numpy paho-mqtt
1
2. 订阅端代码
# subscriber.py
import time
import cv2
import numpy as np
import base64
import paho.mqtt.client as mqtt
import concurrent.futures
from multiprocessing import Process, Queue

# MQTT配置
mqtt_broker = "localhost"
mqtt_port = 1883
mqtt_topic = "your_mqtt_topic"

# 共享变量，使用multiprocessing.Value来确保同步
global_fps = Value('f', 0)
start_time = time.time()
end_time = time.time()

def display_fps(frame):
    global start_time
    global end_time
    end_time = time.time()
    elapsed_time = end_time - start_time
    print(end_time)
    global_fps.value = 1 / elapsed_time

    # 在图像上显示帧率
    cv2.putText(frame, f"FPS: {global_fps.value:.2f}", (10, 30), cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 255, 0), 2, cv2.LINE_AA)
    start_time = end_time

def image_receiver(output_queue):
    # 初始化MQTT客户端
    client = mqtt.Client()
    client.connect(mqtt_broker, mqtt_port, 60)

    # 设置消息接收回调
    def on_message(client, userdata, msg):
        global start_time
        try:
            # 解码base64图像
            start_time = time.time()
            img_base64 = msg.payload.decode('utf-8')
            img_decoded = base64.b64decode(img_base64)
            img_np = np.frombuffer(img_decoded, dtype=np.uint8)
            img = cv2.imdecode(img_np, cv2.IMREAD_COLOR)

            display_fps(img, start_time)

            # 将图像放入队列
            output_queue.put(img)

        except Exception as e:
            print(f"Error: {e}")

    client.on_message = on_message

    # 订阅MQTT主题
    client.subscribe(mqtt_topic)

    # 开始循环，等待消息
    client.loop_forever()

def main():
    # 使用队列在主进程和子进程之间传递数据
    output_queue = Queue()

    # 创建子进程
    receiver_process = Process(target=image_receiver, args=(output_queue,))
    receiver_process.start()

    while True:
        start = time.time()

        # 从队列中获取图像
        while not output_queue.empty():
            img = output_queue.get()

            # 显示图像
            cv2.imshow('Received Image', img)
            cv2.waitKey(1)

        end = time.time()
        print(end - start)

    # 等待子进程结束
    receiver_process.join()

if __name__ == "__main__":
    main()

在推送端运行 pusher.py。
在订阅端运行 subscriber.py。
现在，您应该能够在订阅端实时接收并显示推送端的视频流，同时计算并显示帧率