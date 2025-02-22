# サービス通信

今回は以下のようなノードをもつ`hello_service`パッケージを作ってみましょう。

- `server_node.py`
    - `hello_msgs/Order`型の`/order`サービスからメッセージを受け取る
    - サービスをリクエストしたクライアントにレスポンスを返す
- `client_node.py`
    - `hello_msgs/Order`型のリクエストを`/order`サービスに送って、サーバからレスポンスを受け取る


## パッケージの作成

```none
ros2 pkg create --build-type ament_python hello_service
```

## server_node.pyのコード

`hello_service/hello_service/server_node.py`を以下の内容で書き込む。

```py
import rclpy
from rclpy.node import Node
from hello_msgs.srv import Order

class Service(Node):
    def __init__(self):
        super().__init__("service_node")
        self.order_service = self.create_service(Order, "order", self.order_service_callback)
        self.get_logger().info("service is ready")

    def order_service_callback(self, request, response):
        self.get_logger().info("Recieved: {}".format(request.menu))
        response.message = "へい!{}、お待ち!".format(request.menu)
        self.get_logger().info("Sending {}".format(response.message))
        return response

def main():
    rclpy.init()

    node = Service()
    rclpy.spin(node)

    rclpy.shutdown()

if __name__ == "__main__":
    main()
```

## client_node.pyのコード

`hello_service/hello_service/client_node.py`を以下の内容で書き込む。

```py
import rclpy
from rclpy.node import Node
from hello_msgs.srv import Order

class Client(Node):
    def __init__(self):
        super().__init__("client_node")
        self.order_client = self.create_client(Order, "order")

        while not self.order_client.wait_for_service(timeout_sec=1.0):
            self.get_logger().info("service not avilable, waiting...")

        self.request = Order.Request()

    def send_request(self, menu):
        self.request.menu = menu
        self.future = self.order_client.call_async(self.request)
        rclpy.spin_until_future_complete(self, self.future)
        return self.future.result()


def main():
    rclpy.init()

    node = Client()
    menu = "ラーメン"

    node.get_logger().info("Request: {}".format(menu))

    response = node.send_request(menu)
    node.get_logger().info("Response: {}".format(response.message))

    node.destroy_node()
    rclpy.shutdown()

if __name__ == "__main__":
    main()
```

## ビルドと実行

```none
cd ~/my_ws
colcon build --symlink-install
source install/setup.bash
```

```none
ros2 run hello_service server_node
ros2 run hello_service client_node
```

## 参照

- [https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Service-And-Client.html](https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Service-And-Client.html)
