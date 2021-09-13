### 什么是模式

在对象间定义一种一对多的依赖关系，当这个对象状态发生改变时，所有依赖它的对象都会被通知并自动更新

### 监听模式设计思想

观察者模式是对象的行为模式，又叫发布/订阅（Publish/Subscribe）模式、模型/视图（Model/View）模式、源/监听器（Source/Listener）模式或从属者（Dependents）模式。**监听者模式的核心思想就是在被观察者与观察者之间建立一种自动触发的关系**。

### 代码示例

```python
from abc import ABCMeta, abstractmethod
# 引入ABCMeta和abstractmethod来定义抽象类和抽象方法

class Observer(metaclass=ABCMeta):
    """观察者的基类"""

    @abstractmethod
    def update(self, observable, object):
        pass


class Observable:
    """被观察者的基类"""

    def __init__(self):
        self.__observers = []

    def addObserver(self, observer):
        self.__observers.append(observer)

    def removeObserver(self, observer):
        self.__observers.remove(observer)

    def notifyObservers(self, object=0):
        for o in self.__observers:
            o.update(self, object)


class WaterHeater(Observable):
    """热水器：战胜寒冬的有利武器"""

    def __init__(self):
        super().__init__()
        self.__temperature = 25

    def getTemperature(self):
        return self.__temperature

    def setTemperature(self, temperature):
        self.__temperature = temperature
        print("当前温度是：" + str(self.__temperature) + "℃")
        self.notifyObservers()


class WashingMode(Observer):
    """该模式用于洗澡用"""

    def update(self, observable, object):
        if isinstance(observable, WaterHeater) \
                and observable.getTemperature() >= 50 and observable.getTemperature() < 70:
            print("水已烧好！温度正好，可以用来洗澡了。")


class DrinkingMode(Observer):
    "该模式用于饮用"

    def update(self, observable, object):
        if isinstance(observable, WaterHeater) and observable.getTemperature() >= 100:
            print("水已烧开！可以用来饮用了。")
            
            
 def testWaterHeater():
    heater = WaterHeater()
    washingObser = WashingMode()
    drinkingObser = DrinkingMode()
    heater.addObserver(washingObser)
    heater.addObserver(drinkingObser)
    heater.setTemperature(40)
    heater.setTemperature(60)
    heater.setTemperature(100)
```

### 应用场景

1. 对一个对象状态或数据的更新需要其他对象同步更新，或者一个对象的更新需要依赖另一个对象的更新。
2. 对象仅需要将自己的更新通知给其他对象而不需要知道其他对象的细节，如消息推送。