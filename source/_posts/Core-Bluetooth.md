---
title: Core Bluetooth
date: 2016-04-21 19:50:54
tags: iOS
---
本文介绍蓝牙4.0的一些基本知识。

## 基本概念、服务器、客户端
蓝牙LE是一个基于点对点的通信系统，其中一台设备作为服务器，另一台设备作为客户端。拥有数据的设备作为服务器，消费数据的设备作为客户端。

比如，心率监测器、温控器、手环等，都可以是服务器。服务器通过广播确定自己产生什么类型的数据并把数据发送给连接上的客户端。

客户端是对数据感兴趣的设备，客户端必须自己发现感兴趣的设备。客户端负责初始化对服务器的连接然后开始读取数据。

<!-- more -->

## 类与协议
在iOS中，服务器叫做外围设备（Peripheral），客户端叫做中心设备（Central）。iOS 系统允许iOS设备从一个蓝牙设备读取数据。在`CoreBluetooth.framework`中，读取设备对应`CBCentralManager`这个类，外围设备用`CBPeripheral`这个类来表示。`CBPeripheralManager`和`CBCentral`这两个类表示客户端设备。

类名 | 介绍
---- | ----
CBCentralManager | 该类的对象用于管理发现的或者连接的外围设备。在调用该类的方法前，你需要确定蓝牙设备是可用的。
CBPeripheral | 代表外围设备，每个外围设备通过 UUID 来标识。外围设备可以包含一个或者多个服务、或者提供关于连接的信号强度的信息。
CBCentral | 代表中心设备。


## How
### 通过扫描寻找服务
外围设备是提供数据的服务器，在提供数据之前，必须广播自己能够提供的服务。中心设备扫描这些包来探测附近的外围设备。一个服务器使用全局唯一的UUID来标识自己。

~~~objc
self.centralManager = [[CBCentralManager alloc] initWithDelegate:self queue:nil];
    if (self.centralManager.state == CBManagerStatePoweredOn) {
        [self.centralManager scanForPeripheralsWithServices:nil options:nil];
    }
~~~

找到之后会回调`- (void)centralManager:didDiscoverPeripheral:advertisementData:RSSI:`方法，这这个方法内部你可以做连接外围设备等操作。

### 连接设备
~~~objc
- (void)centralManager:(CBCentralManager *)central didDiscoverPeripheral:(CBPeripheral *)peripheral advertisementData:(NSDictionary<NSString *,id> *)advertisementData RSSI:(NSNumber *)RSSI {
    if (![self.peripherals containsObject:peripheral]) {
        peripheral.delegate = self;
        [self.peripherals addObject:peripheral];
        [self.centralManager connectPeripheral:peripheral options:nil];
    }
}
~~~

当然，如果你知道外围设备的 UUID ，那么根据 UUID 直接获取外围设备来连接也是可以的，我也建议你这么做（因为省电又快速）。

~~~objc
NSArray<CBPeripheral *> *peripheral = [self.centralManager retrievePeripheralsWithIdentifiers:@[[CBUUID UUIDWithString:@"设备的uuid"]]];
~~~

> 注意UUID的格式一定要写对，是这样的`XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX`，其中每一个X代表一个十六进制字符。在终端中输入`uuidgen`可以生成一个，你可以观察一下。

### 发现服务
如果连接成功外围设备，就会回调`- (void)centralManager:didConnectPeripheral:`方法，可以在这里发现外围设备设备提供的服务。如果发现服务，会回调`CBPeripheralDelegate`中的`- (void)peripheral:didDiscoverServices:`方法。

~~~objc
- (void)centralManager:(CBCentralManager *)central didConnectPeripheral:(CBPeripheral *)peripheral {
    [peripheral discoverServices:nil];
}
~~~

### 发现特性
~~~objc
- (void)peripheral:(CBPeripheral *)peripheral didDiscoverServices:(nullable NSError *)error {
    [peripheral.services enumerateObjectsUsingBlock:^(CBService * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        NSLog(@"%@", obj.peripheral.name);
        [peripheral discoverCharacteristics:nil forService:obj];
    }];
}
~~~

### 针对特性写数据
发现特性之后，就可以针对特性做数据处理了。

~~~objc
- (void)peripheral:(CBPeripheral *)peripheral
didDiscoverCharacteristicsForService:(CBService *)service
             error:(NSError *)error {
             
}
~~~

以上就是作为 Central 时，需要用到的基本 API。作为 Peripheral 时，需要时再总结。