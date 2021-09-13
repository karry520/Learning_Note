### 什么是状态模式

允许一个对象在其内部状态发生改变时改变其行为，使这个对象看上去就像改变了它的类型一样

### 状态模式设计思想

状态模式的核心思想就是一个事物（对象）有多种状态，在不同的状态下所表现出来的行为和属性不一样

#### 设计要点：

1. 在实现状态模式的时候，实现的场景状态有时候会非常复杂，决定状态变化的因素也非常多，我们可以把决定状态变化的属性单独抽象成一个类StateInfo，这样判断状态是否符合当前的状态 isMatch时就可以传入更多的信息
2. 每一种状态应当只有唯一的实例

优点：

1. 封闭了状态的转换规则，在状态模式中可以将状态的转换代码封装在环境类中，对状态转换代码进行集中管理，而不是分散在一个个业务逻辑中
2. 将所有与某个状态有关的行为放到一个类中（称为状态类），使开发人员只专注于该状态下的逻辑开发
3. 允许状态转换逻辑与状态对象合为一体，使用时只需要注入一个不同的状态对象即可使环境对象拥有不同的行为

缺点：

1. 会增加系统类和对象的个数 
2. 状态模式的结构与实现都较为复杂，如果使用不当容易导致程序结构和代码的混乱

### 应用场景

1. 一个对象的行为取决于它的状态，并且它在运行时可能经常改变它的状态，从而改变它的行为
2. 一个操作中含有庞大的多分支条件语句，这些分支依赖于该对象的状态，且每一个分支的业务逻辑都非常复杂时，我们可以使用状态模式来拆分不同的分支逻辑，使程序有更好的可读性和可维护性

### 代码示例

```python
from abc import ABCMeta, abstractmethod
# 引入ABCMeta和abstractmethod来定义抽象类和抽象方法

class Context(metaclass=ABCMeta):
    """状态模式的上下文环境类"""

    def __init__(self):
        self.__states = []
        self.__curState = None
        # 状态发生变化依赖的属性, 当这一变量由多个变量共同决定时可以将其单独定义成一个类
        self.__stateInfo = 0

    def addState(self, state):
        if (state not in self.__states):
            self.__states.append(state)

    def changeState(self, state):
        if (state is None):
            return False
        if (self.__curState is None):
            print("初始化为", state.getName())
        else:
            print("由", self.__curState.getName(), "变为", state.getName())
        self.__curState = state
        self.addState(state)
        return True

    def getState(self):
        return self.__curState

    def _setStateInfo(self, stateInfo):
        self.__stateInfo = stateInfo
        for state in self.__states:
            if( state.isMatch(stateInfo) ):
                self.changeState(state)

    def _getStateInfo(self):
        return self.__stateInfo


class State:
    """状态的基类"""

    def __init__(self, name):
        self.__name = name

    def getName(self):
        return self.__name

    def isMatch(self, stateInfo):
        "状态的属性stateInfo是否在当前的状态范围内"
        return False

    @abstractmethod
    def behavior(self, context):
        pass



# Demo 实现

class Water(Context):
    """水(H2O)"""

    def __init__(self):
        super().__init__()
        self.addState(SolidState("固态"))
        self.addState(LiquidState("液态"))
        self.addState(GaseousState("气态"))
        self.setTemperature(25)

    def getTemperature(self):
        return self._getStateInfo()

    def setTemperature(self, temperature):
        self._setStateInfo(temperature)

    def riseTemperature(self, step):
        self.setTemperature(self.getTemperature() + step)

    def reduceTemperature(self, step):
        self.setTemperature(self.getTemperature() - step)

    def behavior(self):
        state = self.getState()
        if(isinstance(state, State)):
            state.behavior(self)


# 单例的装饰器
def singleton(cls, *args, **kwargs):
    "构造一个单例的装饰器"
    instance = {}

    def __singleton(*args, **kwargs):
        if cls not in instance:
            instance[cls] = cls(*args, **kwargs)
        return instance[cls]

    return __singleton


@singleton
class SolidState(State):
    """固态"""

    def __init__(self, name):
        super().__init__(name)

    def isMatch(self, stateInfo):
        return stateInfo < 0

    def behavior(self, context):
        print("我性格高冷，当前体温", context._getStateInfo(),
              "℃，我坚如钢铁，仿如一冷血动物，请用我砸人，嘿嘿……")


@singleton
class LiquidState(State):
    """液态"""

    def __init__(self, name):
        super().__init__(name)

    def isMatch(self, stateInfo):
        return (stateInfo >= 0 and stateInfo < 100)

    def behavior(self, context):
        print("我性格温和，当前体温", context._getStateInfo(),
              "℃，我可滋润万物，饮用我可让你活力倍增……")

@singleton
class GaseousState(State):
    """气态"""

    def __init__(self, name):
        super().__init__(name)

    def isMatch(self, stateInfo):
        return stateInfo >= 100

    def behavior(self, context):
        print("我性格热烈，当前体温", context._getStateInfo(),
              "℃，飞向天空是我毕生的梦想，在这你将看不到我的存在，我将达到无我的境界……")


# Test
########################################################################################################################
def testState():
    # water = Water(LiquidState("液态"))
    water = Water()
    water.behavior()
    water.setTemperature(-4)
    water.behavior()
    water.riseTemperature(18)
    water.behavior()
    water.riseTemperature(110)
    water.behavior()


testState()
```
