# 编程规范

python语言 RaspberryPi封库编程规范 <br>
## 大原则：
 * 看到github，就有购买欲望，并且知道怎么购买
 * 看到ino，就能玩通，尽量不用去查其他资料
 * 看到函数名称，就能基本猜到函数功能
 * 如果只有一个存取接口，不要写子类，全部在一个类里实现，比如DFRobot_Sensor，也不要有_IIC的后缀
## 细则
 * 寄存器命名和手册保持一致，可以加传感器型号头，例如 DS18B20_REG_CONFIG
 * 行业标准COLOR_RGB565_BLACK,写明编码标准，而不是直接用 COLOR_BLACK ，可以放在全局
 * 非行业标准，每个芯片可能都有差异，如果放在类内部不合适，放在全局要加芯片名称头，例如 DS18B20_SPEED_LEVEL1
 * 全局typedef类型可能多个库会冲突，放在class内部定义
 * API某一个参数高概率是一个值的画，我们再h文件中给它提供默认值
 
注意：所有的库严禁 copy , 所有的库首次编写时完全靠个人对技术手册的理解，遇到问题解决不了后才能参考别人的代码（必须理解人家的代码然后按照自己的理解解决问题）

## 索引

* [参考库](#参考库)
* [变量及函数命名](#变量及函数命名)
* [宏](#宏)
* [结构体](#结构体)
* [共用体](#共用体)
* [枚举](#枚举)
* [指针](#指针)
* [类](#类)
* [缩进](#缩进)
* [{}](#{})
* [if-else](#if-else)
* [switch-case](#switch-case)
* [运算符](#运算符)
* [Readme中多个参数函数写法](#readme中多个参数函数写法)
* [ino文件头部写法](#ino文件头部写法)
* [高质量封库细节](#高质量封库细节)

## 参考库

https://github.com/DFRobot/DFRobot_BMP280 <br>
https://github.com/DFRobot/DFRobot_BNO055 <br>

## 变量及函数命名

*首字母小写, 下个单词首字母大写 <br>
*专有名词首所有字母大写 <br>
*有两个专有名词连读的，专有名词首字母大写即可 <br>

正确的写法

```C++
int sensorValue;
int SDCard,IIS,I2C;
int SdSpiSetting;
```

错误的写法

```C++
int sensor_value;
float sensorvalue;
byte sensor_Value;
int SDSPISetting;
```

为了各个库之间的互相兼容，必须做到以下几点。<br>

  1. 库的 cpp 文件中除个别情况外不能有全局变量，所有相关变量应定义在 class 中
  2. 库相关的变量定义要么定义在 class 中，要么加上库名作为前缀
  3. 所有有特殊需求的外设通信（比如说要把IIC通信速率提高到800KHZ），在每次开始通信前必须进行相关配置，通信完成后必须把更改了的配置恢复到默认状态

例: <br>

.h 文件:

```cpp

#define MODULE_OK   0
#define MODULE_ERR  -1

typedef enum {
  eModuleStatusOK,
  eModuleStatusErr = -1
} eModuleStatus_t;

class DFRobot_Module {

  typedef enum {
    eOK,
    eERR
  } eStatus_t;

  eStatus_t writeReg(uint8_t reg, uint8_t *pBuf, uint16_t len);

}
```

.cpp 文件:

```cpp

DFRobot_Module::eStatus_t DFRobot_Module::writeReg(uint8_t reg, uint8_t *pBuf, uint16_t len)
{
  SPI.begin(SPISettings(32000000, MSBFIRST, SPI_MODE0));  // 特殊设定
  // do something
  SPI.begin(SPISettings(14000000, MSBFIRST, SPI_MODE0));  // 恢复默认设定
}

```

## 宏

作为简化记忆功能的宏所有字母都要大写
正确的写法
```C++
#define MAX 1000
#define SENSOR_REG1 1
```

错误的写法：
```C++
#define Max 1000
#define sensorReg1 1
```
带参数的宏,参数要加括号
```C++
#define ADD(x, y)   ((x) + (y))
```
类似函数用法的宏，采用函数的编程规范
```C++
#define swapInt(a, b)  { int c = b; a = b; b = c; }
```
## 结构体

使用typedef，前面要加s，后面首字母大写

```C++
typedef struct
{
  int age;
  char name[20];
}sTeacher_t;
```

```C++
如果使用结构体做寄存器配置，那么这个结构体要字节对其
  /*
   这里从数据手册上抄写关于这个寄存器的描述
     * -----------------------------------------------------------------------------------------
     * |    b7    |    b6    |    b5    |    b4    |    b3    |    b2   |    b1     |    b0    |
     * -----------------------------------------------------------------------------------------
     * |   ready  |         reversed               |      precision     | highspeed | lowpower |
     * -----------------------------------------------------------------------------------------
     *
     *上电后，ready位默认为1，不可更改
  */
  typedef struct {
    uint8_t   lowpower: 1; /*!< 上电为0，1：低功耗模式 0：正常功耗模式 */
    uint8_t   highspeed: 1; /*!< 上电为0，1：告诉模式 0：正常速度模式 */
    uint8_t   precision: 2; /*!< 上电为0，0：低精度模式，1：正常精度模式，2：高精度模式，3：超高精度模式 */
    uint8_t   reserved: 4; /*!< 上电为0，1：低功耗模式 0：正常功耗模式 */
    uint8_t   ready:1; /*!< 上电为0，1：低功耗模式 0：正常功耗模式 */
  } __attribute__ ((packed)) sMode_t;
```


## 共用体

使用typedef，前面要加u，后面首字母大写

```C++
typedef union {
  uint32_t value;
  uint8_t b[4];
} uValue_t;
```

## 枚举

前面要加e，后面首字母大写
```C++
typedef enum{
  eSunday=0,
  eMonday,
  eTuesday,
  eWednesday,
  eThursday,
  eFriday,
  eSaturday,
}eDay_t;
```
## 指针

指针前面要加 * 显式说明这是一个指针

```C++
int *pVar1, *pVar2;
int **ppVar3;
sTeacher_t *psTeacher,*psT1,*psT2;
uValue_t *puValue,*puV1,*puV2;
eDay_t *peDay, *peD, *peD1,*peD2;
```

指向函数的指针，使用pf做前缀

```C++
typedef void (*pfTeacherWork_t)();
```

## 类

以DFRobot_开头,后面接ClassName,ClassName首字母要大些
正确的写法
```C++
class DFRobot_SIM7000;
class DFRobot_BME280;
class DFRobot_Lora;
```
错误的写法
```C++
class DFRobot_sim7000;
class Lora;
class DFRobotLora;
```
除 public 成员变量外，其他成员变量的名字前要加 _ 前缀

```C++
class DFRobot_BME280
{
public:
  DFRobot_BME280(int addr)
    :_addr(addr)
    {}
private:
  int _addr;
}
```

## 缩进

全部使用空格缩进，不要使用tab缩进（除了特殊用途，比如keywords makefile里），请用notepad++检查

## {}
在函数中，{}顶格写，例如

```C++
void func()
{
  //do something
}
```

## if-else

{写在语句后面，}写在新行,这样很方便调试。例如
```C++
if(conditon1){
  //do something
}else if(conditon2){
  //do something
}else{
  //do something
}
```

## switch-case

case与switch对其，{跟在switch语句后面，例如
```C++
switch(value){
case x:
  //do something
case y:
  //do something
default:
  //do something
}
```

## 运算符

前后需要留空格(此条根据具体情况，不强制)

```C++
for(int x = 0; x <= 10; x++)
```

## Readme中多个参数函数写法
采用doxygen注释规则描述参数

```C++
    /*!
     *  @brief Set operational mode to VL53L0X
     *
     *  @param mode  Work mode settings
     *      Single : Single mode
     *      Continuous : Back-to-back mode
     *  @param precision  Set measurement precision
     *      High：High precision(0.25mm)
     *      Low: Low precision(1mm)
     *  @return  true if execute successfully, false otherwise.
     */
    bool setMode(uint8_t mode, uint8_t precision);
```

## ino文件头部写法
```C++
 /*!
  * file 文件名（不能使用中文）
  *
  * 如何做这个实验，描述实验步骤（只需要下载程序就能肉眼观测到的简单小实验例如blink，这步可以不写）（不能使用中文）
  * @n 实验现象是什么（不能使用中文）
  *
 * @copyright   Copyright (c) 2010 DFRobot Co.Ltd (http://www.dfrobot.com)
 * @licence     The MIT License (MIT)
 * @author      Alexander(ouki.wang@dfrobot.com)
 * @version  V1.0
 * @date  2019-07-16
 * @get from https://www.dfrobot.com
 * @url https://github.com/DFRobot/DFRobot_PAJ7620U2  */
```

## 高质量封库细节

1. 兼容性好（兼容 uno, leonardo, mega2560, esp8266, mpython/esp32，micro:bit)
2. 兼容旧版 arduino V1.0.5（不强制，但是要做这个测试，能兼容则一定要兼容）
3. 与其他库兼容好，参考[变量及函数命名](#变量及函数命名)
4. 库和 demo 中的便于理解的注释充足
5. 正常情况下不使用 malloc 函数和 new 关键字，库有使用大的 buffer 的情况依据平台定义大的静态 buffer 或想办法优化
6. 代码使用的 ram 尽量少，使用的 rom 不能太多
7. 对于传感器中寄存器的每一个位，必须使用结构体进行定义 <br>
例:
```cpp

// .h 文件定义在类中
typedef struct {
  uint8_t   power: 1;
  uint8_t   trig: 1;
  uint8_t   reversed: 2;
  uint8_t   mode: 2;
}__attribute__ ((packed)) sRegConfig_t;

// .cpp 文件中的使用

void DFRobot_Module::writeRegBits(uint8_t reg, uint8_t field, uint8_t val)
{
  uint8_t   temp;
  readReg(reg, (uint8_t*) &temp, sizeof(temp));
  temp &= ~field;
  temp |= val;
  writeReg(reg, (uint8_t*) &temp, sizeof(temp));
}

void DFRobot_Module::setPowerOn()
{
  sRegConfig_t    sField = {0}, sVal = {0};
  sField.power = 0xff; sVal.power = ePowerOn;
  writeRegBits(eRegPower, *(uint8_t*) &sField, *(uint8_t*) &sVal);
}

```
8. 代码功能设置合理，初级 demo（小白用户上手测试文件）用户体验好，中高级 demo （非小白用户）用户体验尽量好
9. 代码简洁精炼，易于升级维护，不是技术难度越高越好，而是以用户用起来简单功能容易上手、有深入功能、api用户体验设计非常好为核心
10. 所有的类必须逻辑与接口实现分离，接口的外设类需由用户传入，不允许直接使用 Wire，SPI 等类 <br>
例：
```cpp

class DFRobot_Module_I2C : public DFRobot_Module {
public:
  DFRobot_Module_I2C(TwoWire *pWire) { _pWire = pWire; }

  TwoWire   *_pWire;
}

```
12.  使用DBG宏，方便后期查找bug，release的时候屏蔽掉ENABLE_DBG
