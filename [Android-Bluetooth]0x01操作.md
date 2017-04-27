
## 0x00 蓝牙（Bluetooth）

这个小硬件已经是Android机器的标配了，由于平时不怎么用，因此到现在都没有去研究过。现在有一个简单的小需求：通过蓝牙连接一个串口设备读取其上面的数据，即从已配对的设备列表中选择我们的串口蓝牙设备（从），连接，然后读取数据。遂写此文。


## 0x01 简单使用

蓝牙设备的详细使用，可以参考[Android关于蓝牙的官方文档](https://developer.android.google.cn/guide/topics/connectivity/bluetooth.html)。如果你和我一样，之前没有研究过蓝牙，估计看看完后也会有一堆问题存在：什么主设备、从设备、UUID是干嘛的，如何免密钥配对等等。不着急，我们慢慢来。

上面已经说到，我们的设备已经提前配对完成（怎么配对：网络设置中中找到蓝牙，然后搜索，找到你的设备，然后配对。这个时候可能会提示输入密码，默认密码比0000，1234等）。我们只需要连接即可。下面简述操作步骤：

1. 添加蓝牙权限

    ``` xml
    <uses-permission android:name="android.permission.BLUETOOTH" />
    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
    ```
    
2. 判断是否支持蓝牙

    ``` java
    private void initBluetooth() {
        mBluetoothAdapter = BluetoothAdapter.getDefaultAdapter();
        if (null == mBluetoothAdapter) {
            tvContent.setText("BluetoothAdapter is null");
            return;
        }
        if (!mBluetoothAdapter.isEnabled()) {
            tvContent.setText("BluetoothAdapter is disable, please open it");

            Intent intent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
            startActivityForResult(intent, REQUEST_ENABLE_BT);
        }

        tvContent.setText("Bluetooth init success");
    }
    ```
    
    首先是判断是否有蓝牙适配器，如果没有，`BluetoothAdapter.getDefaultAdapter()` 返回 `null`。然后判断蓝牙设备是否启用 `mBluetoothAdapter.isEnabled()` ，如果未启用，则发送一个 `Intent` 来让用户启用蓝牙，这个 `Intent` 是系统的，我们在 `onActivityResult` 中处理用户操作结果，如果用户顺利开启蓝牙，则会返回 `RESULT_OK`。
    
3. 获取设备列表

    ``` java
    Set<BluetoothDevice> pairedDevices = mBluetoothAdapter.getBondedDevices();
    if (null != pairedDevices && pairedDevices.size() > 0) {
      for (BluetoothDevice device : pairedDevices) {
          ......
          String msg = String.format("%s %s\n", device.getName(), device.getAddress());
          ......
      }
    }
    ```
    
    这个时候我们可以拿到 `BluetoothDevice` ，这个对象中保存了已配对蓝牙设备的信息，比如名称，MAC地址，状态，UUID等信息（但这些信息不一定都都），我们需要保存下来，在接下来连接的时候会使用。
    
4. 连接设备

    有了 `BluetoothDevice` 信息，我们就可以连接这个已经配对的蓝牙设备了。
    
    ``` java
    public static final UUID DEFAULT_UUID = UUID.fromString("00001101-0000-1000-8000-00805f9b34fb");

    try {
       if(null == mSocket){
           mSocket = mDevice.createRfcommSocketToServiceRecord(DEFAULT_UUID);
       }
       mSocket.connect(); // 阻塞的

       mIn = mSocket.getInputStream();
   } catch (IOException e) {
       e.printStackTrace();
       return false;
   }
    ```
    
    连接成功后，我们会得到一个 `BluetoothSocket` 对象，然后调用其阻塞的 `connect()` 方法，等待两台设备连接成功（所以这个时候必须在另外一个线程中进行）。当两台设备连接成功之后会继续向下执行。
    
5. 读取数据

    连接成功之后，我们可以通过 `mSocket.getInputStream()` 获得一个输入流，结下来的操作就是流的操作了，这个和普通 `socket` 中输入输出流的操作一样了。
    
    ``` java
    try {
       mBytes = mIn.read(readBuffer);
       System.arraycopy(readBuffer, 0, tempBuffer, mCount, mBytes);
       mCount += mBytes;
       if (mCount >= SIZE) {
           parseData(tempBuffer);
           mCount = 0;
       }
   } catch (IOException e) {
       e.printStackTrace();
   }
    ```


## 0x02 总结
   
Android蓝牙（主从）的操作：
   
- 检查是否支持，是否启用（包括是否可见等）
- 扫描设备
- 配对
- 连接
- 数据交换
- 等等

这里只讲了检查部分操作，涉及的扫描与配对可能是更复杂一些的，在接下来讲述。

最后，项目代码可参考这里[github/ttdevs/air](https://github.com/ttdevs/android/tree/master/apps/air)。


## 0x03 Java线程的封装

``` java
/**
 * Created by ttdevs
 * 2017-01-22 (android)
 * https://github.com/ttdevs
 */
public abstract class BaseWorkerThread extends Thread {

    private boolean isRunning = true;

    @Override
    public void run() {
        super.run();
        isRunning = workerBefore();

        while (isRunning) {
            workerCycle();
        }

        workerAfter();
    }

    /**
     * 提前执行 true: 继续 false: 结束
     *
     * @return
     */
    public boolean workerBefore() {
        return true;
    }

    /**
     * 工作方法，被循环调用
     *
     * @return true: 继续 false: 结束
     */
    public abstract void workerCycle();


    /**
     * 结束执行
     */
    public void workerAfter() {

    }

    /**
     * 开始线程
     */
    public void startThread() {
        isRunning = true;
        try {
            start();
        } catch (Exception e) {
            e.printStackTrace();
            isRunning = false;
        }
    }

    /**
     * 结束线程
     */
    public void stopThread() {
        isRunning = false;
    }
}
```


## 0x04 参考

- https://developer.android.google.cn/guide/topics/connectivity/bluetooth.html

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)


