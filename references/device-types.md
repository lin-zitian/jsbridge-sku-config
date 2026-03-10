# 设备类型参考文档

本文档记录项目中支持的所有设备类型（Product Category），用于创建新 SKU 时选择正确的设备类别。

## 设备类型概览

项目目前支持 10 种设备类型：

1. **灯具类** (Light) - 默认类型
2. **加湿器类** (Humidifier)
3. **风扇类** (Fan)
4. **加热器类** (Heater)
5. **制冰机类** (Ice Maker)
6. **温度计类** (Thermometer)
7. **传感器类** (Sensor)
8. **温控器类** (Thermostat)
9. **水壶类** (Kettle)
10. **冷冻甜品机类** (Frozen Treat Maker)

---

## 设备类型列表

### 1. 灯具类 (Light) - 默认类型

**类型标识**: `devices.types.light`

**配置指南**: `#[[file:references/light-guide.md]]`

**适用设备**:
- LED 灯带
- RGBIC 灯带
- 智能灯泡
- 台灯
- 氛围灯
- 灯串
- 投影灯
- 落地灯
- 吸顶灯

**常用功能实例**:
- `powerSwitch`: 电源开关
- `brightness`: 亮度控制
- `colorRgb`: RGB 颜色控制
- `colorTemperatureK`: 色温控制
- `segmentedColorRgb`: 分段颜色控制
- `segmentedBrightness`: 分段亮度控制
- `lightScene`: 灯光场景
- `musicMode`: 音乐模式
- `gradientToggle`: 渐变开关
- `mainLightToggle`: 主灯开关
- `backgroundLightToggle`: 背灯开关
- `nightlightToggle`: 夜灯开关

**示例 SKU**:
- H1232: RGBIC light
- H7140: Smart LED Strip

---

### 2. 加湿器类 (Humidifier)

**类型标识**: `devices.types.humidifier`

**配置指南**: `#[[file:references/humidifier-guide.md]]`

**适用设备**:
- 加湿器
- 香薰机
- 雾化器

**常用功能实例**:
- `powerSwitch`: 电源开关
- `workMode`: 工作模式(手动/自定义/自动)
- `humidity`: 湿度调节
- `nightlightToggle`: 夜灯开关
- `brightness`: 夜灯亮度
- `colorRgb`: 夜灯颜色
- `nightlightScene`: 夜灯场景
- `mistToggle`: 出雾开关
- `warmMistToggle`: 热雾开关
- `lackWaterEvent`: 缺水事件

**特点**:
- 湿度控制(通常40-80%)
- 可能支持夜灯功能
- 可能支持出雾控制

**示例 SKU**:
- H7140: 加湿器

---

### 3. 风扇类 (Fan)

**类型标识**: `devices.types.fan`

**配置指南**: `#[[file:references/fan-guide.md]]`

**适用设备**:
- 电风扇
- 循环扇
- 塔扇
- 落地扇

**常用功能实例**:
- `powerSwitch`: 电源开关
- `fanSpeed`: 风速调节
- `fanSpeedMode`: 风速档位模式
- `gearMode`: 档位调节
- `oscillationToggle`: 摇头开关
- `workMode`: 工作模式(Normal/Natural/Sleep/Auto)
- `swingLeafToggle`: 摆叶开关
- `airDeflectorToggle`: 导流板开关
- `powerOffDuration`: 延时关机

**特点**:
- 风速控制(连续或档位)
- 可能支持摇头功能
- 可能支持多种工作模式

**示例 SKU**:
- H7100: 风扇

---

### 4. 加热器类 (Heater)

**类型标识**: `devices.types.heater`

**配置指南**: `#[[file:references/heater-guide.md]]`

**适用设备**:
- 智能温控器
- 恒温器
- 温度调节器

**常用功能实例**:
- `powerSwitch`: 电源开关
- `temperature`: 温度调节
- `targetTemperature`: 目标温度
- `rangeTemperature`: 温度范围调节
- `thermostatToggle`: 恒温开关
- `sensorTemperature`: 传感器温度

**特点**:
- 支持摄氏度/华氏度切换
- 支持温度范围设置
- 支持自动恒温模式

---

**常用功能实例**:
- `powerSwitch`: 电源开关
- `targetTemperature`: 目标温度
- `rangeTemperature`: 温度范围控制
- `workMode`: 工作模式(档位/风扇/自动/睡眠)
- `gearMode`: 档位调节
- `sensorTemperature`: 传感器温度
- `thermostatToggle`: 恒温开关
- `oscillationToggle`: 摇头开关
- `powerOffDuration`: 延时关机

**特点**:
- 温度控制(单点或范围)
- 支持多种工作模式
- 可能支持档位调节

**示例 SKU**:
- H7138: 取暖器
- H7137: 取暖器

---

### 5. 制冰机类 (Ice Maker)

**类型标识**: `devices.types.kettle`

**适用设备**:
- 智能水壶
- 电热水壶
- 保温壶

**常用功能实例**:
- `powerSwitch`: 电源开关
- `temperature`: 温度设置
- `targetTemperature`: 目标温度
- `sensorTemperature`: 当前温度

**特点**:
- 支持精确温度控制
- 支持保温功能
- 支持温度单位切换

---

### 6. 温度计类 (Thermometer)

**类型标识**: `devices.types.thermometer`

**配置指南**: `#[[file:references/thermometer-guide.md]]`

**适用设备**:
- 温湿度计
- 温度传感器
- 环境监测仪

**常用功能实例**:
- `sensorTemperature`: 温度读取
- `sensorHumidity`: 湿度读取
- `temCalibration`: 温度校准
- `humCalibration`: 湿度校准

**特点**:
- 只读设备,无控制功能
- 可能支持校准功能

**示例 SKU**:
- H5110: 温湿度计
- H5111: 温湿度计

---

### 7. 传感器类 (Sensor)

**类型标识**: `devices.types.sensor`

**配置指南**: `#[[file:references/sensor-guide.md]]`

**适用设备**:
- 人体感应传感器
- 空气质量传感器
- CO2传感器

**常用功能实例**:
- `bodyAppearedEvent`: 人体出现事件
- `sensorTemperature`: 传感器温度(只读)
- `sensorHumidity`: 传感器湿度(只读)
- `carbonDioxideConcentration`: 二氧化碳浓度(只读)
- `airQuality`: 空气质量(只读)

**特点**:
- 只读设备,无控制功能
- 可能支持事件告警

**示例 SKU**:
- H5127: 人体感应传感器

---

### 8. 温控器类 (Thermostat)

**类型标识**: `devices.types.thermostat`

**配置指南**: `#[[file:references/thermostat-guide.md]]`

**适用设备**:

**类型标识**: `devices.types.ice_maker`

**适用设备**:
- 制冰机
- 冰块制造机
- 商用制冰设备

**常用功能实例**:
- `powerSwitch`: 电源开关
- `iceMakingToggle`: 制冰/清洗开关
- `workMode`: 制冰模式（薄冰、中冰、厚冰、超厚冰）
- `precoolToggle`: 预冷开关
- `temperature`: 温度设置
- `sensorTemperature`: 传感器温度
- `nightlightToggle`: 夜灯开关
- `brightness`: 夜灯亮度
- `colorRgb`: 夜灯颜色
- `nightlightScene`: 夜灯场景
- `iceFullEvent`: 冰满事件
- `waterFullEvent`: 水满事件
- `lackWaterEvent`: 缺水事件
- `cleaningCompletedEvent`: 清洁完成事件
- `runInterruptEvent`: 运行中断事件
- `iceMakingCompletedEvent`: 制冰完成事件

**特点**:
- 支持多种制冰模式
- 支持清洗和除霜功能
- 丰富的事件通知
- 支持温度监控
- 可选夜灯氛围灯功能

**配置注意事项**:
- ⚠️ 制冰机的 BLE 指令差异较大，不同型号的指令可能完全不同
- ⚠️ 核心 instance（powerSwitch, iceMakingToggle, workMode）需要详细询问用户配置
- ⚠️ 事件的 alarmType 值可能不同，需要确认
- 详细配置指南：`references/ice-maker-guide.md`

**示例 SKU**:
- H8131: 制冰机（110V，4种清洗模式）
- H8120: 制冰机（110V，2种清洗模式，15个夜灯场景）
- H8121: 制冰机（110V，2种清洗模式，15个夜灯场景）
- H8122: 制冰机（5个夜灯场景）
- H7172: 制冰机

---

### 7. 冷冻甜品机类 (Frozen Treat Maker)

**类型标识**: `devices.types.frozen_treat_maker`

**适用设备**:
- 冷冻甜品机
- 冰淇淋机
- 雪糕机

**常用功能实例**:
- `powerSwitch`: 电源开关
- `iceMakingCompletedEvent`: 制作完成事件
- `keepColdEndingSoonEvent`: 保冷即将结束事件
- `keepColdEndedEvent`: 保冷结束事件

**特点**:
- 专注于冷冻甜品制作
- 支持保冷功能
- 制作完成通知

**示例 SKU**:
- H8102: Frozen Treat Maker

---

## 设备类型选择指南

### 如何选择设备类型？

1. **根据设备主要功能**
   - 照明功能 → `devices.types.light`
   - 加湿功能 → `devices.types.humidifier`
   - 风扇功能 → `devices.types.fan`
   - 加热功能 → `devices.types.heater`
   - 制冰功能 → `devices.types.ice_maker`
   - 环境监测 → `devices.types.thermometer`
   - 传感器检测 → `devices.types.sensor`
   - 温度控制 → `devices.types.thermostat`
   - 烧水功能 → `devices.types.kettle`
   - 冷冻甜品 → `devices.types.frozen_treat_maker`

2. **根据设备控制方式**
   - 主要控制颜色/亮度 → `devices.types.light`
   - 主要控制湿度 → `devices.types.humidifier`
   - 主要控制风速 → `devices.types.fan`
   - 主要控制温度 → `devices.types.thermostat` 或 `devices.types.heater`
   - 只读传感器数据 → `devices.types.thermometer` 或 `devices.types.sensor`
   - 制冰相关控制 → `devices.types.ice_maker`

3. **默认选择**
   - 如果不确定，灯具类设备默认使用 `devices.types.light`

### 配置示例

```javascript
const Endpoint = {
  productSku: 'H1232',
  productCategory: 'devices.types.light',  // 设备类型
  productName: 'RGBIC light',
  description: '',
  goodsType: '',
  capabilities: [
    // ...
  ]
};
```

---

## 扩展新设备类型

如果需要添加新的设备类型：

1. **在 utils/common.js 中添加**
   ```javascript
   export const ProductCategoryEnum = {
     'devices.types.thermostat': 'devices.types.thermostat',
     'devices.types.heater': 'devices.types.heater',
     'devices.types.kettle': 'devices.types.kettle',
     'devices.types.light': 'devices.types.light',
     'devices.types.thermometer': 'devices.types.thermometer',
     'devices.types.newType': 'devices.types.newType'  // 新类型
   };
   ```

2. **更新此文档**
   - 添加新类型的详细说明
   - 列出适用设备
   - 列出常用功能实例
   - 添加配置示例

3. **更新 SKILL.md**
   - 在产品类别选项中添加新类型

---

## 常见问题

### Q: 一个设备可以属于多个类型吗？
A: 不可以。每个 SKU 只能有一个 `productCategory`。选择最能代表设备主要功能的类型。

### Q: 如果设备既有照明又有加热功能怎么办？
A: 选择主要功能作为设备类型。例如，带加热功能的氛围灯应该选择 `devices.types.light`。

### Q: 新设备类型不在列表中怎么办？
A: 参考"扩展新设备类型"章节，先在 `ProductCategoryEnum` 中添加，然后更新文档。

### Q: 默认使用哪个类型？
A: 如果是灯具类设备，默认使用 `devices.types.light`。

---

## 更新记录

| 日期 | 版本 | 更新内容 |
|------|------|----------|
| 2024-03-10 | 1.0.0 | 初始版本，包含5种基础设备类型 |
| 2024-03-10 | 1.1.0 | 添加制冰机类和冷冻甜品机类，共7种设备类型 |
| 2024-03-10 | 2.0.0 | 添加加湿器、风扇、传感器类型，共10种设备类型，为每种类型添加配置指南链接 |

---

## 参考资料

- `jsbridge-be/utils/common.js` - ProductCategoryEnum 定义
- `standard-sku-template.md` - 标准 SKU 配置模板
- `H1232-template.md` - 灯具类设备完整示例
