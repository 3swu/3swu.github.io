---
author: wulei
comments: true
date: 2018-07-20 18:01:00+00:00
link: http://wuleiaty.github.io/2018/07/20/Homebridge-homekit/
slug: Homebridge-homekit
title: 智能家具DIY之让Siri帮你开灯
tags:
- 智能家居
- HomeKit
- HomeBridge
---

### 简述

HomeKit平台是苹果的智能家居平台，将支持HomeKit的产品接入这个平台，就可以在诸如iPhone和iPad等平台上控制这些产品，但是由于苹果HomeKit“严格的性能标准”要求，支持HomeKit的设备比较少，也比较昂贵。出于理工男的DIY精神:-)，昨天决定试一试通过开源的Homebridge项目将一个台灯接入HomeKit，这样就能让Siri控制灯。

通过查阅资料发现，网络上大多数资料都介绍通过现成的插件将一些设备接入HomeKit，比较难实现自己的想法，所以今天重点聊聊编写自己的插件，来完美的解决自己的方案。

#### Homebridge

Homebridge是一个开源的HomeKit桥接器，项目地址[nfarina/Homebridge](https://github.com/nfarina/Homebridge)。Homebridge实现了HomeKit的协议，使得它可以作为设备和HomeKit之间的桥接器。Homebridge收到HomeKit的指令然后通过支持不同设备的插件来实现相应的动作。简单来说，Homebridge就是一个服务器，工作方式如下图：

![](https://wuleiaty.github.io/assets/images/homebridge-1.JPG)

Homebridge支持在Windows/Linux/mac os平台上运行，所以最好的场景是将Homebridge安装在树莓派或者家中的路由器上（使用OpenWRT/LEDE系统的路由器），由于我的树莓派有其他用，所以这次讲讲在我的电脑（Ubuntu 18.04）使用Homebridge，在树莓派上的操作和我这次基本无异。

#### Homebridge插件

Homebridge负责作为设备与HomeKit之间的桥梁，而具体的动作则由相应设备的插件来实现。有许多典型场景的插件，比如有插件可以将米家网关接入HomeKit，这样米家设备也可以通过HomeKit来控制。通过实现不同的插件，可以实现各种不同的动作，比如通过HTTP请求来控制远程的设备，也可以编写插件来实现树莓派GPIO口的输入输出等，这次的DIY重点就是编写适合自己的插件。

#### 此次场景

我将以前的小灯开关拆掉，接上继电器，用继电器代替手动的开关，继电器接在Arduino上，Arduino通过串口与电脑通信。HomeKit发出指令后，Homebridge接收到指令调用自己编写的插件，这个插件的作用就是向串口写控制指令，Arduino通过串口接收到指令后就可以实现灯的开关。

下面依次介绍Homebridge环境的安装，Homebridge插件的编写，Arduino程序的编写和硬件的调试。

### Homebridge环境安装

环境安装的资料可以在网络上轻松的找到，所以这里只简述在Linux上安装Homebridge。特别的，如果要在树莓派上安装，有一点区别，具体参考这个[wiki](https://github.com/nfarina/Homebridge/wiki/Running-Homebridge-on-a-Raspberry-Pi)。

1. Homebridge基于Node.js开发，所以Node环境是必须的(v4.3.2以上)，Ubuntu上可以通过`sudo apt install nodejs`命令安装，通过`node --version`命令查看当前安装的Node版本。然后通过`sudo apt install npm`安装npm，npm是Node的软件包管理器。
2. 安装libavahi-compat-libdnssd-dev包：`sudo apt install libavahi-compat-libdnssd-dev`。
3. 安装Homebridge：`sudo npm install -g --unsage-perm homebridge`。
4. 安装插件，但我们需要自己编写插件进行调试，所以这步跳过。
5. 创建Homebridge的配置文件，目录为`~/.homebridge/config.json`，编写完插件进行调试时会编辑这个文件。

### 编写插件

#### 编写之前的环境准备

1. 建立一个文件夹，文件夹名自取，我将这个插件命名为serialportlight，所以这个文件夹也叫serialportlight。

2. 进入这个文件夹，使用`npm init`命令初始化为一个node项目，初始化过程中会有一些配置，跟着提示即可。

3. 初始化完成之后目录中会生成一个`package.json`文件，这是项目的配置文件。初始化完成之后需要手动添加一些字段，不然后续会报错。我的文件如下：
``` json
   {
     "name": "homebridge-serialportlight",
     "version": "1.0.0",
     "description": "",
     "main": "index.js",
     "scripts": {
       "test": "echo \"Error: no test specified\" && exit 1"
     },
     "author": "wulei",
     "license": "ISC",
     "keywords":["homebridge-plugin"],
     "engines": {
         "node": ">=0.12.0",
         "homebridge": ">=0.2.0"
     }
   }
```
4. 在目录先建立`index.js`文件，插件的逻辑将全部写在这个文件中。
5. 安装Node串口操作需要的库`node-serialport`：`sudo npm install node-serialport`

#### 编写插件逻辑

我们编写的插件在Homebridge中抽象为一个Accessory，Homebridge中还有一个Platform的概念，这个例子很简单，作为一个Accessory就可以完成。所以我们先注册：

``` javascript
var Service, Characteristic;

module.exports = function(homebridge) {
    Service = homebridge.hap.Service;
    Characteristic = homebridge.hap.Characteristic;
    homebridge.registerAccessory('homebridge-serialportlight', 'serialportlight', light);
}
```

将我们的设备用`light`对象表示，接下来我们编写light的具体属性和动作：

``` javascript
function light(log, config){
    this.log = log;
    this.name = config['name'];
    this.service = new Service.Switch(this.name);
    this.informationService = new Service.AccessoryInformation();

    this.service
        .getCharacteristic(Characteristic.On)
        .on('get', this.getOn.bind(this))
        .on('set', this.setOn.bind(this));
}
```

这个构造函数初始化了`light`对象的属性，其中`this.name = config['name']`表示对象的`name`属性由上文提到的Homebridge的`config.json`配置文件中制定。

下一行说明这个设备是一个Switch类型的设备，因为Homebridge不仅支持这种离散指令的设备，还支持温度计等各种复杂的具有连续数据的设备。

最后将设备的状态设置为内置的On状态，这个状态的值只有true和false两种。然后将`set`和`get`两种指令分别绑定到两个函数，`get`指令用于HomeKit获取当前的设备状态，是开还是关，`set`指令用于改变当前的状态。

接下来编写这两个具体的函数：

``` javascript
var SerialPort = require('serialport');
var port = new SerialPort('/dev/ttyACM0',{baudRate:9600});
var lightStatus = false;

light.prototype.getOn = function(next){
    next(null, lightStatus);
}

light.prototype.setOn = function(on, next){
    // console.log("Set on function");
    if(on == true)
        port.write('1', function(err){
            if(err)
                return console.log("Error on write: ", err.message);
            lightStatus = true;
            console.log("[+] Message written: 1");
        });
    else if (on == false)
        port.write('0', function(err){
            if(err)
                return console.log("Error on write: ", err.message);
            lightStatus = false;
            console.log("[+] Message written: 0");
        });
    next(null);
}
```

`lightStatus`变量用于存储设备当前的状态。

首先导入串口操作的库`serialport`，然后打开串口，设备地址在`/dev/*`中，具体的地址可以在里面查看，波特率设置为9600。

在`getOn`函数中，使用回调函数返回当前的设备状态，由于`lightStatus`变量中直接存储了设备状态的布尔值并且在改变设备状态的时候这个变量也相应的改变了，所以直接传回这个变量即可。

在setOn函数中，传入一个`on`参数表示需要设备改变到什么状态，为一个布尔型变量。使用`port.write()`函数向串口中写入数据，因为我们需要的指令只有简单的两个，所以用0和1就可以表示两条指令。在这个函数的回调函数中向屏幕输出日志并且改变变量中的设备状态。

至此，插件的编写到此结束，插件和配置文件的详细代码请移步[wuleiaty/homebridge-arduino-plugin](https://github.com/wuleiaty/homebridge-arduino-plugin)

### 编写Arduino程序

使用Arduino IDE或者其它安装了Arduino插件的编辑器编写Arduino程序。这个程序接收串口的指令并且根据指令向数字I/O口写高低电平以控制灯的开关。程序很简单，如下：

``` c
int light_pin = 8;

void setup() {
    pinMode(light_pin, OUTPUT);
    digitalWrite(light_pin, LOW);
    Serial.begin(9600);
}

void loop() {
    while(Serial.available()){
        char data = Serial.read();
        if(data == '1'){
            digitalWrite(light_pin, LOW);
        }
        else if(data == '0'){
            digitalWrite(light_pin, HIGH);
        }
    }
}
```



这里假如将继电器的信号引脚接到Arduino的8号引脚，将串口的波特率设置为和上文插件中的波特率一样。`Serial.available()`表示串口缓冲区中有数据可以读出，然后根据指令向引脚写高低电平就行。

需要注意的是，在Ubuntu上将程序写入Arduino时，可能会因为设备权限的原因报错，假如设备地址为`/dev/ttyACM0`，这时使用以下命令解决：`sudo chmod 777 /dev/ttyACM0`

### 调试

所有的程序都已经编写完成，将Arduino接入电脑并且确保串口通信和Arduino逻辑正常工作，可以使用Arduino IDE自带的串口调试器。还需要编写Homebridge的配置文件，上文提到地址为`~/.homebridge/config.json`，我的配置文件如下：

``` json
{
    "bridge": {
        "name": "Homebridge",
        "username": "CC:22:3D:E3:CE:52",
        "port": 55555,
        "pin": "033-73-876"
    },

    "accessories": [
        {
            "accessory": "serialportlight",
            "name": "灯"
        }
    ],

    "platforms": []
}
```

这个文件配置Homebridge的地址，端口和pin码等信息还有插件的信息，使用命令`DEBUG=* homebridge -D -U ~/.homebridge/ -P ~/serialportlight/`即可启动Homebridge。使用iPhone或者iPad上的“家庭“App，扫描二维码或者手动输入PIN码，确保你的iOS设备和Homebridge服务器在同一局域网下，即可添加成功。

现在，你就可以试试”嘿 Siri，开灯！”

![](https://wuleiaty.github.io/assets/images/homebridge-2.png)
