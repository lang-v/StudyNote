# 经典蓝牙

## 注意
- 蓝牙连接需要的 *UUID*,
客户端和服务端须保持一致，且如果此UUID被占用可能出现无法连接
- 在C#中 *UUID*就是 *GUID*

## 蓝牙套接字（BluetoothSocket）
### 服务端

```
//UUID
Guid guid = new Guid("");
//连接的客户端
BluetoothClient conn = null;
BluetoothListener bluetoothListener = new BluetoothListener(guid);
//开启监听
bluetoothListener.Start();
//System.Net.Sockets.NetworkStream
NetworkStream stream;
//等待客户端连接,同步阻塞
conn = bluetoothListener.AcceptBluetoothClient();
/**
* 异步方法
* BeginAcceptBluetoothClient()
* EndAcceptBluetoothClient()
*/
//从连接中获取流
stream = conn.GetStream();
byte[] received = new byte[1024];
//读取数据
stream.Read(received,0,received.Length);
byte[] sent = new byte[1024];
//发送数据
stream.Write(sent,0,sent.Length);
```
PS:当客户端断开连接时，服务端不会自动断开，如果服务端是循环读取数据，那么会不断的从流中读取到空数据 '\0'，很影响性能，建议手动中断