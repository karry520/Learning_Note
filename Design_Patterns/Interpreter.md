### 什么是中介模式

用一个中介对象来封装一系列的对象交互，中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互

### 中介模式设计思想

在很多系统中，多个类很容易相互耦合，形成网状结构。中介模式的作用就是将这种网状结构分离成星型结构。这样调整之后，使得对象间的结构更加简洁，交互更加顺畅。

#### 设计要点：

中介模式主要有以下三个角色，在设计中介模式时要找到并区分这些角色：

1. 交互对象（InteractiveObject）：要进行交互的一系列对象
2. 中介者（Mediator）：负责协调各个对象之间的交互
3. 具体中介者（Mediator）：中介的具体实现

优点：

1. Mediator将原本分布于多个对象间的行为集中在一起，作为一个独立的概念并将其封装在一个对象中，简化了对象之间的交互。
2. 将多个调用者与多个实现者之间多对多的交互关系，转换为一对多的交互关系，一对多的交互关系更易于理解、维护和扩展，大大减少了多个对象之间相互交叉引用的情况。

缺点：

1. 中介者承接了所有的交互逻辑，交互的复杂度转变成了中介者的复杂度，中介者类会变得越来越庞大和复杂，以至于难以维护
2. 中介者出现问题会导致多个使用者同时出问题

### 应用场景

1. 一组对象以定义良好但复杂的方式进行通信。产生的相互依赖关系结构混乱且难以理解
2. 一个对象引用其他很多对象并且直接与这些对象通信，导致难以复用该对象
3. 想通过一个中间类来封装多个类中的行为，同时又不想生成太多的子类

### 代码示例

```python
from abc import ABCMeta, abstractmethod
# 引入ABCMeta和abstractmethod来定义抽象类和抽象方法
from enum import Enum
# Python3.4 之后支持枚举Enum的语法

class DeviceType(Enum):
    "设备类型"
    TypeSpeaker = 1
    TypeMicrophone = 2
    TypeCamera = 3

class DeviceItem:
    """设备项"""

    def __init__(self, id, name, type, isDefault = False):
        self.__id = id
        self.__name = name
        self.__type = type
        self.__isDefault = isDefault

    def __str__(self):
        return "type:" + str(self.__type) + " id:" + str(self.__id) \
               + " name:" + str(self.__name) + " isDefault:" + str(self.__isDefault)

    def getId(self):
        return self.__id

    def getName(self):
        return self.__name

    def getType(self):
        return self.__type

    def isDefault(self):
        return self.__isDefault


class DeviceList:
    """设备列表"""

    def __init__(self):
        self.__devices = []

    def add(self, deviceItem):
        self.__devices.append(deviceItem)

    def getCount(self):
        return len(self.__devices)

    def getByIdx(self, idx):
        if idx < 0 or idx >= self.getCount():
            return None
        return self.__devices[idx]

    def getById(self, id):
        for item in self.__devices:
            if( item.getId() == id):
                return item
        return None

class DeviceMgr(metaclass=ABCMeta):

    @abstractmethod
    def enumerate(self):
        """枚举设备列表
        (在程序初始化时，有设备插拔时都要重新获取设备列表)"""
        pass

    @abstractmethod
    def active(self, deviceId):
        """选择要使用的设备"""
        pass

    @abstractmethod
    def getCurDeviceId(self):
        """获取当前正在使用的设计ID"""
        pass


class SpeakerMgr(DeviceMgr):
    """扬声器设备管理类"""

    def __init__(self):
        self.__curDeviceId = None

    def enumerate(self):
        """枚举设备列表
        (真实的项目应该通过驱动程序去读取设备信息，这里只用初始化来模拟)"""
        devices = DeviceList()
        devices.add(DeviceItem("369dd760-893b-4fe0-89b1-671eca0f0224", "Realtek High Definition Audio", DeviceType.TypeSpeaker))
        devices.add(DeviceItem("59357639-6a43-4b79-8184-f79aed9a0dfc", "NVIDIA High Definition Audio", DeviceType.TypeSpeaker, True))
        return devices

    def active(self, deviceId):
        """激活指定的设备作为当前要用的设备"""
        self.__curDeviceId = deviceId

    def getCurDeviceId(self):
        return self.__curDeviceId


class DeviceUtil:
    """设备工具类"""

    def __init__(self):
        self.__mgrs = {}
        self.__mgrs[DeviceType.TypeSpeaker] = SpeakerMgr()
        # 为节省篇幅，MicrophoneMgr和CameraMgr不再实现
        # self.__microphoneMgr = MicrophoneMgr()
        # self.__cameraMgr = CameraMgr

    def __getDeviceMgr(self, type):
        return self.__mgrs[type]

    def getDeviceList(self, type):
        return self.__getDeviceMgr(type).enumerate()

    def active(self, type, deviceId):
        self.__getDeviceMgr(type).active(deviceId)

    def getCurDeviceId(self, type):
        return self.__getDeviceMgr(type).getCurDeviceId()


# Test
#=======================================================================================================================

def testDevices():
    deviceUtil = DeviceUtil()
    deviceList = deviceUtil.getDeviceList(DeviceType.TypeSpeaker)
    print("麦克风设备列表：")
    if deviceList.getCount() > 0:
        # 设置第一个设备为要用的设备
        deviceUtil.active(DeviceType.TypeSpeaker, deviceList.getByIdx(0).getId())
    for idx in range(0, deviceList.getCount()):
        device = deviceList.getByIdx(idx)
        print(device)
    print("当前使用的设备："
          + deviceList.getById(deviceUtil.getCurDeviceId(DeviceType.TypeSpeaker)).getName())


# testRenting()
testDevices()
```