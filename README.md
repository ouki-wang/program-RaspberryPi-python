# python编程规范

python 封库编程规范

## 索引

* [参考库](#参考库)
* [文件结构与命名](#文件结构与命名)
* [类与注释](#类与注释)
* [常量命名](#常量命名)
* [变量命名](#变量命名)
* [函数命名与注释](#函数命名与注释)
* [高质量封库细节](#高质量封库细节)

## 参考库

https://github.com/DFRobot/DFRobot_RaspberryPi_Motor <br>
https://github.com/DFRobot/DFRobot_INA219<br>

## 文件结构与命名

库可以直接放在 Arduino 封库的根目录下，建立 python 目录用于存放树莓派的驱动和demo <br>
 <br>
文档结构应类似如下，如果不与 Arduino 一起使用，则直接将原本 python 文件夹下的内容放在根目录下: <br>
<pre>

--python:
| --raspberrypi:
| | --DFRobot_Module.py
| | --examples:
| | | --Modele_demo_xxxx.py
| | | --Modele_demo_yyy.py
| | ......
|
| --esp32:
| | --DFRobot_Module.py
| | --examples:
| | | --Modele_demo_xxxx.py
| | | --Modele_demo_yyy.py
| | ......
| |

</pre>

例程文件头部注释写法：<br>

<pre>
""" file 文件名
  #
  # 如何做这个实验，描述实验步骤（只需要下载程序就能肉眼观测到的简单小实验例如blink，这步可以不写）
  # 实验现象是什么
  #
  # @copyright   Copyright (c) 2010 DFRobot Co.Ltd (http://www.dfrobot.com)
  # @licence     The MIT License (MIT)
  # @author      Alexander(ouki.wang@dfrobot.com)
  # version  V1.0
  # date  2017-10-9
  # @get from https://www.dfrobot.com
  # @url https://github.com/DFRobot/DFRobot_PAJ7620U2
"""
</pre>

## 类与注释

类名应当以 DFRobot_ 作为开头, 模组型号作为结尾。<br>
在构造函数中说明参数:

```py
class DFRobot_Module:

  def __init__(i2cAddr):
    """ Class constructor

    :param i2cAddr    Module's i2c address
    """
    pass

```

## 常量命名

C 库中的宏，包括枚举变量在 python 中需要写在类的全局变量中，全部使用大写字母和下划线组合

C 库中的定义：
```cpp
#define MODULE_CONF   0x00

typedef enum {
  eModuleConfEnum1,
  eModuleConfEnum2
} eModuleConf_t;
```

对应 python 中的定义：
```py
class DFRobot_Module:

  _REG_CONF = 0x00

  CONF_ENUM1 = 0x00
  CONF_ENUM2 = 0x01

```

## 变量命名

受保护的变量（对应 cpp 里的 protected ）名前需要加 _, 私有的变量（对应 cpp 里的 private ）名前需要加 __ <br>
任何类型的变量和函数名都使用下划线加小写字母命名规则 <br>

例：
```py
class DFRobot_Module:

  _protect_var = 1  # 受保护的变量
  _protect_list = [0, 1]  # 变量注释
  _protect_dict = {
    "a": 1
  }

  __private_var = 1  # 私有的变量
  __private_list = [0, 1]  # 变量注释
  __private_dict = {
    "a": 1
  }

  public_var = 1  # 公有变量
  public_list = [0, 1]
  public_dict = {
    "a": 1
  }

```

## 函数命名与注释

受保护的函数（对应 cpp 里的 protected ）名前需要加 _, 私有的函数（对应 cpp 里的 private ）名前需要加 __ <br>
函数名使用下划线加小写字母命名规则 <br>

示例：
```py
class DFRobot_Module:

  def __init__(self, param1):
    """ Module init

    :param param1:int Set to ...
    """
    pass

  def _private_func(self, param1):
    """ func detail

    :param param1:int Set to ...
    """
    pass

  def public_func(self, param1):
    """ Check param1

    :param param1:int Parameter to check

    :return:bool Check result
      :retval True Check succeed
      :retval False Check falied
    """
    if param1:
      return True
    else:
      return False

  POWER_ON = 0x00
  POWER_OFF = 0x01

  def public_func2(self):
    """ Return power status

    :return Power status
    """
    return self._powerStatus

```

## 高质量封库细节

RaspberryPi库全部是由Arduino库移植过来的，所以完成Arduino库是基础。Arduino库没有问题了，Python库只是一个格式的转化
