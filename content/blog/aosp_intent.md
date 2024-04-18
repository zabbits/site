+++
title = "Intent In AOSP"
date = 2024-04-17

[taxonomies]
tags = ["AOSP", "Android"]
+++
## Intent用途
Intent的主要是用于组件之间的通信. 像常见的APP内的分享到其他应用, 跳转其他APP都是通过Intent实现. Intent大致有以下几个用途:
- 启动activity
- 启动service
  - AOSP蓝牙代码方面大量使用bindService()的方式获取service
- 发送broadcast

{% tip(header="Tip") %}
也就是Android四大组件其中的三个, 所以Intent在源码中随处可见...
{% end %}

## Intent类型
Intent有两种类型:
- Explicit intent 显示Intent需要指定一个完整的组件名称来获取组件. 所以通常用于App内部的一些服务.
- Implicit intent 隐式Intent指定一个通用的Action, 其他组件通过监听这个Action来做出响应, 例如分享操作系统通常会显示许多APP供用户选择(这些APP都是指定了相应的Intent filter).

{% important(header="Important") %}
在Android5.0(API level 21)之后, 只能通过显示Intent调用bindService()
{% end %}

## Intent属性
Intent有几个比较重要的属性，通过这些属性构造不同的Indent
### Component name
指定组件名称是唯一的构建显示Intent的方式.
### Action
一个字符串，代表需要执行的操作.
在broadcast intent中，通常是表示一个操作已经发生了（例如广播蓝牙连接状态发生变化).
{% tip(header="Tip") %}
AOSP中蓝牙状态机的实现大量使用了Broadcast intent
{% end %}
通常一个Action也就决定了这个Intent的结构（包含哪些其他的Data，Extra等)
### Data
Data是操作数据的URI。指定Data的同时可以设置Type, Type可以标识数据的MIME类型。例如通过Intent查看图片：
```java
// 设定一个查看图片的Action
Intent intent = new Intent(Intent.ACTION_VIEW);
Uri data = Uri.parse("content://com.example.app/images/1");
String type = "image/jpg";
intent.setDataAndType(data, type);
```
### Category
表示Intent的类型。
### Extra
Intent附加的数据信息。
### Flags
控制Android如何启动Activity以及在启动和如何处理Activity. 这些Flags定义在`Intent`类中, 开发者不能自定义Flags。


接下来通过AOSP蓝牙的`A2dpStateMachine`具体看一下Intent的使用： 
{% codeblock(name="com.android.bluetooth.a2dp.A2dpStateMachine")%}
```java
private void broadcastConnectionState(int newState, int prevState) {
    log("Connection state " + mDevice + ": " + profileStateToString(prevState)
                + "->" + profileStateToString(newState));

    // 指定一个Action
    Intent intent = new Intent(BluetoothA2dp.ACTION_CONNECTION_STATE_CHANGED);
    // 写入Extras数据
    intent.putExtra(BluetoothProfile.EXTRA_PREVIOUS_STATE, prevState);
    intent.putExtra(BluetoothProfile.EXTRA_STATE, newState);
    intent.putExtra(BluetoothDevice.EXTRA_DEVICE, mDevice);
    // 指定Flags
    intent.addFlags(Intent.FLAG_RECEIVER_REGISTERED_ONLY_BEFORE_BOOT
                    | Intent.FLAG_RECEIVER_INCLUDE_BACKGROUND);
    // 通过Intent发起broadcast
    mA2dpService.sendBroadcast(intent, BLUETOOTH_CONNECT,
            Utils.getTempAllowlistBroadcastOptions());
}
```
{% end %}


## Intent filter
Intent filter用于组件过滤接收哪些隐式的Intent.
通过action, category, data来指定filter属性去匹配对应的Intent.
有两种方式定义Intent filter:
- 在APP的manifest文件中定义
{% codeblock(name="AndroidManifest.xml")%}
```xml
<activity android:name="ShareActivity" android:exported="false">
    <intent-filter>
        <action android:name="android.intent.action.SEND"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="text/plain"/>
    </intent-filter>
</activity>
```
{% end %}
- 通过broadcast receiver的方法registerReceiver()动态注册
