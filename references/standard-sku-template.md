# SKU 配置标准模板

这是一个通用的 SKU 配置标准模板，列出了所有可用的控制类型、数据类型和常用方法。

## 目录

1. [基础数据类型](#基础数据类型)
2. [产品类别](#产品类别)
3. [功能实例类型 (Instance)](#功能实例类型)
4. [能力类型 (Capability Type)](#能力类型)
5. [命令类型 (Command Type)](#命令类型)
6. [标准 SKU 结构](#标准-sku-结构)
7. [常用工具函数](#常用工具函数)

---

## 基础数据类型

```javascript
export const DataTypeEnum = {
  INTEGER: 'INTEGER',    // 整数类型
  ENUM: 'ENUM',         // 枚举类型
  BOOLEAN: 'BOOLEAN',   // 布尔类型
  STRING: 'STRING',     // 字符串类型
  FLOAT: 'FLOAT',       // 浮点类型
  STRUCT: 'STRUCT',     // 结构体类型
  ARRAY: 'Array'        // 数组类型
};
```

---

## 产品类别

```javascript
export const ProductCategoryEnum = {
  'devices.types.thermostat': '温控器',
  'devices.types.heater': '加热器',
  'devices.types.kettle': '水壶',
  'devices.types.light': '灯具',
  'devices.types.thermometer': '温度计',
  'devices.types.ice_maker': '制冰机',
  'devices.types.frozen_treat_maker': '冷冻甜品机'
};
```

**注意**：虽然 `utils/common.js` 中只定义了前5种类型，但项目中实际使用了 7 种设备类型。新 SKU 可以使用任何一种类型。

---

## Instance 优先级规则（重要）

在创建新的 SKU 配置时，必须遵循以下 Instance 使用优先级：

### 规则说明

1. **优先使用现有 Instance**
   - 首先查找 `jsbridge-be/utils/common.js` 中的 `InstanceEnum` 常量
   - 如果需要的 instance 已存在，必须使用现有定义
   - 不要重复创建已存在的 instance

2. **创建新 Instance 的条件**
   - 只有当 `InstanceEnum` 中不存在所需的 instance 时才创建新的
   - 新 instance 必须添加到 `utils/common.js` 的 `InstanceEnum` 中

3. **新 Instance 定义格式**
   ```javascript
   [instanceName]: {
     value: '[instanceName]',
     name: '[中文名称]',
     CapabilityType: '[devices.capabilities.xxx]',
     BaseDataType: DataTypeEnum.[TYPE]
   }
   ```

### 操作流程

1. **读取现有 Instance**
   ```bash
   # 查看 utils/common.js 中的 InstanceEnum
   cat jsbridge-be/utils/common.js | grep -A 5 "export const InstanceEnum"
   ```

2. **检查 Instance 是否存在**
   - 在 `InstanceEnum` 中搜索需要的 instance 名称
   - 确认其 CapabilityType 和 BaseDataType 是否符合需求

3. **使用现有 Instance**
   ```javascript
   {
     instance: InstanceEnum.powerSwitch.value,  // 使用现有的
     type: 'devices.capabilities.on_off',
     // ...
   }
   ```

4. **添加新 Instance（如果需要）**
   ```javascript
   // 在 utils/common.js 的 InstanceEnum 中添加
   export const InstanceEnum = {
     // ... 现有 instance
     newInstanceName: {
       value: 'newInstanceName',
       name: '新功能名称',
       CapabilityType: 'devices.capabilities.xxx',
       BaseDataType: DataTypeEnum.INTEGER
     }
   };
   ```

### 示例

假设要创建一个支持"呼吸灯"功能的 SKU：

1. **检查 InstanceEnum**
   - 搜索是否有 `breathingLight` 或类似的 instance
   - 如果有 `breathingLightToggle`，直接使用

2. **如果不存在**
   - 创建新的 instance 定义
   - 添加到 `InstanceEnum`：
   ```javascript
   breathingLightToggle: {
     value: 'breathingLightToggle',
     name: '呼吸灯开关',
     CapabilityType: 'devices.capabilities.toggle',
     BaseDataType: DataTypeEnum.ENUM
   }
   ```

3. **在 SKU 配置中使用**
   ```javascript
   {
     instance: InstanceEnum.breathingLightToggle.value,
     type: 'devices.capabilities.toggle',
     ctrlPlatformSupported: ['openApi'],
     // ...
   }
   ```

---

## SKU 方法版本说明（重要）

### 新方法 vs 旧方法

项目中的 SKU 配置方法经历了版本演进，目前分为新旧两种方法：

#### ✅ 新方法（必须使用）

所有新创建的 SKU 必须使用以下方法：

1. **getMqttPayload(input)**
   - 用途：生成 MQTT 控制指令
   - 参数：包含 sku、instance、data、transaction 等的输入对象
   - 返回：MQTT payload 对象

2. **getCapabilities(input)**
   - 用途：返回设备能力列表
   - 参数：包含 device 信息的输入对象
   - 返回：包含 capabilities 数组的对象

3. **getFullInstanceState(input)**
   - 用途：获取设备完整状态
   - 参数：包含 reportState、isFull 的输入对象
   - 返回：包含所有 instance 状态的对象

4. **Endpoint**
   - 用途：设备配置对象
   - 包含：productSku、productCategory、productName、capabilities

5. **getMusicMode(input)** （可选）
   - 用途：获取音乐模式数据
   - 仅音乐设备需要
   - 返回：音乐模式列表

#### ❌ 旧方法（已废弃）

以下方法仅存在于历史 SKU 中，新 SKU 不要使用：

1. **getBleRead** - 已废弃，不要使用
2. **getBleCommands** - 已废弃，不要使用

### 为什么要使用新方法？

1. **统一接口**：新方法提供了统一的接口规范
2. **更好的抽象**：通过 `getMqttPayload` 统一处理所有控制指令
3. **易于维护**：减少重复代码，提高可维护性
4. **平台兼容**：更好地支持 OpenAPI 平台

### 导出示例

**正确的导出（新方法）**：
```javascript
export default {
  getMqttPayload,
  getCapabilities,
  getFullInstanceState,
  Endpoint,
  getMusicMode  // 可选
};
```

**错误的导出（包含旧方法）**：
```javascript
// ❌ 不要这样做
export default {
  getBleRead,      // 旧方法，不要使用
  getBleCommands,  // 旧方法，不要使用
  getMqttPayload,
  getCapabilities,
  getFullInstanceState,
  Endpoint
};
```

### 检查清单

创建新 SKU 时，确保：
- ✅ 实现了 `getMqttPayload` 方法
- ✅ 实现了 `getCapabilities` 方法
- ✅ 实现了 `getFullInstanceState` 方法
- ✅ 导出了 `Endpoint` 对象
- ✅ 如果有音乐模式，实现了 `getMusicMode` 方法
- ❌ 没有使用 `getBleRead` 方法
- ❌ 没有使用 `getBleCommands` 方法

---

## 功能实例类型

### 开关类 (Toggle/On-Off)

#### 基础开关
- `powerSwitch`: 电源开关 (devices.capabilities.on_off)
- `fanToggle`: 风扇开关 (devices.capabilities.toggle)

#### 灯光开关
- `mainLightToggle`: 主灯开关
- `backgroundLightToggle`: 背灯开关
- `nightlightToggle`: 夜灯开关
- `leftLightToggle`: 左灯柱
- `rightLightToggle`: 右灯柱
- `backLightToggle`: 背灯
- `pillarLightToggle`: 灯柱
- `baseLightToggle`: 底座
- `light1Toggle`: 第一灯头
- `light2Toggle`: 第二灯头
- `light3Toggle`: 第三灯头

#### 功能开关
- `oscillationToggle`: 摇头开关
- `swingLeafToggle`: 摆叶开关
- `airDeflectorToggle`: 导流板开关
- `reverseAirflowToggle`: 反向出风开关
- `fanOscillateToggle`: 风扇摇头开关
- `mistToggle`: 出雾开关
- `whiteNoiseToggle`: 白噪音开关
- `gradientToggle`: 渐变开关
- `thermostatToggle`: 恒温开关
- `warmMistToggle`: 热雾开关
- `dreamViewToggle`: 盛宴开关
- `precoolToggle`: 预冷开关
- `iceMakingToggle`: 制冰开关
- `socketToggle1`: 电源插口开关1
- `socketToggle2`: 电源插口开关2

### 范围控制类 (Range)

- `brightness`: 亮度调节 (1-100%)
- `humidity`: 湿度调节
- `fanSpeed`: 风速调节
- `mistDuration`: 出雾延时关
- `nightlightDuration`: 夜灯延时关
- `powerOffDuration`: 电源延时关

### 温度控制类 (Temperature Setting)

- `temperature`: 温度调节
- `sliderTemperature`: 滑动温度调节
- `targetTemperature`: 目标温度
- `rangeTemperature`: 温度范围调节

### 颜色控制类 (Color Setting)

- `colorRgb`: RGB 颜色控制
- `colorTemperatureK`: 色温控制 (K值)
- `colorHsb`: HSB 颜色控制
- `segmentedColorRgb`: 分段颜色控制
- `segmentedBrightness`: 分段亮度控制
- `colorForAll`: 全段颜色控制
- `klvForAll`: 全段色温控制

### 场景模式类 (Scene/Mode)

- `lightScene`: 灯光场景
- `diyScene`: DIY 场景
- `musicMode`: 音乐模式
- `movieMode`: 视频模式
- `workMode`: 工作模式
- `gearMode`: 档位模式
- `fanSpeedMode`: 风速档位
- `deflectorSpeedMode`: 导流板速度
- `deflectorRange`: 导板摆动范围
- `nightlightStatus`: 夜灯状态模式
- `nightlightScene`: 夜灯场景
- `hdmiSource`: HDMI 切换
- `snapshot`: 快照

### 传感器类 (Sensor/Property)

- `sensorTemperature`: 传感器温度
- `sensorHumidity`: 传感器湿度
- `carbonDioxideConcentration`: 二氧化碳浓度
- `airQuality`: 空气质量
- `filterLifeTime`: 滤芯寿命

### 事件类 (Event)

- `iceFullEvent`: 冰满事件
- `waterFullEvent`: 水满事件
- `lackWaterEvent`: 缺水事件
- `cleaningCompletedEvent`: 清洁完成事件
- `runInterruptEvent`: 运行中断事件
- `iceMakingCompletedEvent`: 制冰完成事件
- `keepColdEndingSoonEvent`: 保冷即将结束事件
- `keepColdEndedEvent`: 保冷结束事件
- `longRunProtectionEvent`: 长时间运行保护事件
- `bodyAppearedEvent`: 人体出现事件

### 系统功能类 (System)

- `heartbeat`: 心跳
- `bindDevice`: 绑定设备
- `abnormalState`: 异常状态
- `temCalibration`: 温度校准
- `humCalibration`: 湿度校准
- `sensorQuery`: 传感器查询
- `broadcastParser`: 广播包解析
- `feastPreview`: 盛宴预览


---

## 能力类型

```javascript
export const CapabilityTypeEnum = {
  'devices.capabilities.on_off': '开关控制',
  'devices.capabilities.toggle': '切换开关',
  'devices.capabilities.range': '范围控制',
  'devices.capabilities.temperature_setting': '温度设置',
  'devices.capabilities.temperature_range': '温度范围',
  'devices.capabilities.temperature_auto': '自动温度',
  'devices.capabilities.mode': '模式控制',
  'devices.capabilities.music_setting': '音乐设置',
  'devices.capabilities.work_mode': '工作模式',
  'devices.capabilities.color_setting': '颜色设置',
  'devices.capabilities.scene_color_setting': '场景颜色',
  'devices.capabilities.diy_color_setting': 'DIY 颜色',
  'devices.capabilities.segment_color_setting': '分段颜色',
  'devices.capabilities.hsb_color_setting': 'HSB 颜色',
  'devices.capabilities.snapshot_color_setting': '快照颜色',
  'devices.capabilities.broadcast_parser': '广播解析',
  'devices.capabilities.color_for_all': '全段颜色',
  'devices.capabilities.klv_for_all': '全段色温',
  'devices.capabilities.property': '属性',
  'devices.capabilities.event': '事件',
  'devices.capabilities.sys_read': '系统读取',
  'devices.capabilities.abnormal_state': '异常状态',
  'devices.capabilities.bind_device': '绑定设备',
  'devices.capabilities.tem_calibration': '温度校准',
  'devices.capabilities.hum_calibration': '湿度校准',
  'devices.capabilities.sensor_query': '传感器查询',
  'devices.capabilities.feast_preview': '盛宴预览'
};
```

---

## 命令类型

```javascript
export const CmdTypeEnumDispatcher = {
  turn: {
    value: 'turn',
    payloadBuilder: turnPayloadBuilder,
    description: '开关控制指令'
  },
  brightness: {
    value: 'brightness',
    payloadBuilder: brightnessPayloadBuilder,
    description: '亮度控制指令'
  },
  colorwc: {
    value: 'colorwc',
    payloadBuilder: colorwcPayloadBuilder,
    description: '颜色/色温控制指令（新设备）'
  },
  color: {
    value: 'color',
    payloadBuilder: colorPayloadBuilder,
    description: '颜色控制指令'
  },
  colorTem: {
    value: 'colorTem',
    payloadBuilder: colorTemPayloadBuilder,
    description: '色温控制指令（旧设备）'
  },
  multiSync: {
    value: 'multiSync',
    payloadBuilder: multiSyncPayloadBuilder,
    description: '多包同步指令'
  },
  ptReal: {
    value: 'ptReal',
    payloadBuilder: ptRealPayloadBuilder,
    description: '透传实时指令'
  },
  ptIot: {
    value: 'ptIot',
    payloadBuilder: ptIotPayloadBuilder,
    description: '透传 IoT 指令'
  },
  bulb: {
    value: 'bulb',
    payloadBuilder: bulbPayloadBuilder,
    description: '灯泡控制指令'
  },
  status: {
    value: 'status',
    payloadBuilder: statusPayloadBuilder,
    description: '状态查询指令'
  }
};
```

### 命令类型使用场景

- **turn**: 用于简单的开关控制（powerSwitch）
- **brightness**: 用于亮度调节
- **colorwc**: 用于颜色和色温控制（推荐新设备使用）
- **color**: 用于纯颜色控制
- **colorTem**: 用于色温控制（旧设备兼容）
- **ptReal**: 用于自定义 BLE 指令（最常用）
- **multiSync**: 用于需要多条指令同步的场景
- **ptIot**: 用于 IoT 平台透传
- **bulb**: 用于灯泡类设备
- **status**: 用于状态查询

---

## 标准 SKU 结构

### 文件结构

```
packages/[SKU]/
├── package.json          # 构建配置
├── README.md            # 使用示例文档
└── src/
    └── index.mjs        # 核心配置代码
```

### package.json 模板

```json
{
  "main": "./dist/[SKU].global.js",
  "license": "MIT",
  "buildOptions": {
    "name": "[SKU]",
    "formats": ["global"]
  }
}
```

### src/index.mjs 标准结构

```javascript
import {
  InstanceEnum,
  CmdTypeEnumDispatcher,
  equalsIgnoreCase
} from '../../../utils/common.js';
import analysis from '../../../utils/analysis.js';
import fillStateHandler from '../../../utils/fillStateHandler.js';

// 1. Endpoint 配置
const Endpoint = {
  productSku: '[SKU]',
  productCategory: '[CATEGORY]',
  productName: '[NAME]',
  description: '',
  goodsType: '',
  capabilities: [
    // capabilities 配置
  ]
};

// 2. 特殊数据配置（如音乐模式、场景等）
const MusicMode = []; // 如果需要

// 3. 拦截器函数（如果需要）
function capabilitiesInterceptor(item) {
  // 动态修改 capabilities
}

function mqttPayloadInterceptor(input, capaInstance, output) {
  // 特殊处理 MQTT payload
  return output;
}

// 4. 核心函数
function getCapabilities(input) {
  // 返回设备能力列表
}

function getMqttPayload(input) {
  // 生成 MQTT 控制指令
}

function getFullInstanceState(input) {
  // 获取设备完整状态
}

// 5. 状态映射（如果需要）
const LocalInstanceStateValueMap = new Map([
  // 状态解析映射
]);

// 6. 导出
export default {
  getMqttPayload,
  getCapabilities,
  getFullInstanceState,
  Endpoint
};
```

### 导出方法说明

**新方法（必须使用）**：
- `getMqttPayload`: 生成 MQTT 控制指令
- `getCapabilities`: 返回设备能力列表
- `getFullInstanceState`: 获取设备完整状态
- `Endpoint`: 设备配置对象
- `getMusicMode`: 音乐模式数据（可选，仅音乐设备需要）

**旧方法（已废弃）**：
- ❌ `getBleRead` - 不要使用
- ❌ `getBleCommands` - 不要使用

**重要提示**：
- 新创建的 SKU 只使用新方法
- 旧方法仅存在于历史 SKU 中，用于兼容
- 所有新 SKU 必须实现 `getMqttPayload`、`getCapabilities`、`getFullInstanceState` 三个核心方法


---

## Capability 配置模板

### 1. 开关控制 (on_off)

```javascript
{
  instance: InstanceEnum.powerSwitch.value,
  type: 'devices.capabilities.on_off',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: {
    cmdType: 'turn'
  },
  parameters: {
    dataType: 'ENUM',
    options: [
      { name: 'on', value: 1 },
      { name: 'off', value: 0 }
    ]
  }
}
```

### 2. 切换开关 (toggle)

```javascript
{
  instance: InstanceEnum.gradientToggle.value,
  type: 'devices.capabilities.toggle',
  ctrlPlatformSupported: ['openApi'],
  action: {
    write: [{ bleDefine: ['33', 'a3', '01'], valIndex: [2] }],
    read: []
  },
  payloadDefine: {
    cmdType: 'ptReal'
  },
  parameters: {
    dataType: 'ENUM',
    options: [
      { name: 'on', value: 1 },
      { name: 'off', value: 0 }
    ]
  }
}
```

### 3. 范围控制 (range)

```javascript
{
  instance: InstanceEnum.brightness.value,
  type: 'devices.capabilities.range',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: {
    cmdType: 'brightness'
  },
  parameters: {
    dataType: 'INTEGER',
    unit: 'unit.percent',
    range: {
      min: 1,
      max: 100,
      precision: 1
    }
  }
}
```

### 4. 颜色设置 (color_setting)

#### RGB 颜色
```javascript
{
  instance: InstanceEnum.colorRgb.value,
  type: 'devices.capabilities.color_setting',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: {
    cmdType: 'colorwc'
  },
  parameters: {
    dataType: 'INTEGER',
    range: {
      min: 0,
      max: 0xffffff,
      precision: 1
    }
  }
}
```

#### 色温
```javascript
{
  instance: InstanceEnum.colorTemperatureK.value,
  type: 'devices.capabilities.color_setting',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: {
    cmdType: 'colorwc'
  },
  parameters: {
    dataType: 'INTEGER',
    range: {
      max: 6500,
      min: 2700,
      precision: 1
    }
  }
}
```

### 5. 分段颜色设置 (segment_color_setting)

```javascript
{
  instance: InstanceEnum.segmentedColorRgb.value,
  type: 'devices.capabilities.segment_color_setting',
  ctrlPlatformSupported: ['openApi'],
  action: {
    write: [
      {
        bleDefine: ['33', '05', '15', '01', 'ff', 'ff', 'ff', '00', '00', '00', '00', '00', 'ff', 'ff'],
        valIndex: [4, 5, 6, 12, 13]
      }
    ]
  },
  payloadDefine: {
    cmdType: 'ptReal'
  },
  parameters: {
    dataType: 'STRUCT',
    fields: [
      {
        fieldName: 'segment',
        dataType: 'Array',
        required: true,
        elementType: 'INTEGER',
        elementRange: { max: 12, min: 0 },
        size: { max: 13, min: 1 }
      },
      {
        fieldName: 'rgb',
        dataType: 'INTEGER',
        required: true,
        range: {
          min: 0,
          max: 0xffffff,
          precision: 1
        }
      }
    ]
  }
}
```

### 6. 温度设置 (temperature_setting)

```javascript
{
  instance: InstanceEnum.temperature.value,
  type: 'devices.capabilities.temperature_setting',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: {
    cmdType: 'ptReal'
  },
  action: {
    write: [
      {
        bleDefine: ['33', '02', '00', '00', '00'],
        valIndex: [2, 3, 4]
      }
    ]
  },
  parameters: {
    dataType: 'STRUCT',
    fields: [
      {
        fieldName: 'temperature',
        dataType: 'INTEGER',
        required: true,
        range: { min: 40, max: 100, precision: 1 }
      },
      {
        fieldName: 'unit',
        dataType: 'ENUM',
        required: false,
        options: [
          { name: 'Celsius', value: 'Celsius' },
          { name: 'Fahrenheit', value: 'Fahrenheit' }
        ]
      }
    ]
  }
}
```

### 7. 模式控制 (mode)

```javascript
{
  instance: InstanceEnum.gearMode.value,
  type: 'devices.capabilities.mode',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: {
    cmdType: 'ptReal'
  },
  action: {
    write: [
      {
        bleDefine: ['33', '05', '00'],
        valIndex: [2]
      }
    ]
  },
  parameters: {
    dataType: 'ENUM',
    options: [
      { name: 'low', value: 1 },
      { name: 'medium', value: 2 },
      { name: 'high', value: 3 }
    ]
  }
}
```

### 8. 音乐模式 (music_setting)

```javascript
{
  instance: InstanceEnum.musicMode.value,
  type: 'devices.capabilities.music_setting',
  ctrlPlatformSupported: ['openApi'],
  action: {
    write: [
      {
        bleDefine: ['33', '05', '13', '05', '32', '00', '01', 'ff', 'ff', 'ff'],
        valIndex: [3, 4, 5, 6, 7, 8, 9]
      }
    ]
  },
  payloadDefine: {
    cmdType: 'ptReal'
  },
  parameters: {
    dataType: 'STRUCT',
    fields: [
      {
        fieldName: 'musicMode',
        dataType: 'ENUM',
        required: true,
        options: [] // 动态填充
      },
      {
        fieldName: 'sensitivity',
        dataType: 'INTEGER',
        required: true,
        unit: 'unit.percent',
        range: { min: 0, max: 100, precision: 1 }
      },
      {
        fieldName: 'autoColor',
        dataType: 'ENUM',
        required: false,
        options: [
          { name: 'on', value: 1 },
          { name: 'off', value: 0 }
        ]
      },
      {
        fieldName: 'rgb',
        dataType: 'INTEGER',
        required: false,
        range: { min: 0, max: 0xffffff, precision: 1 }
      }
    ]
  }
}
```

### 9. 场景设置 (scene_color_setting)

```javascript
{
  instance: InstanceEnum.lightScene.value,
  type: 'devices.capabilities.scene_color_setting',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: {
    cmdType: 'ptReal'
  },
  parameters: {
    dataType: 'ENUM',
    options: [
      { name: 'scene1', value: 1 },
      { name: 'scene2', value: 2 },
      { name: 'scene3', value: 3 }
    ]
  }
}
```

### 10. 属性读取 (property)

```javascript
{
  instance: InstanceEnum.sensorTemperature.value,
  type: 'devices.capabilities.property',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: {
    cmdType: 'ptReal'
  },
  parameters: {
    dataType: 'STRUCT',
    fields: [
      {
        fieldName: 'temperature',
        dataType: 'INTEGER',
        range: { min: -40, max: 125, precision: 0.1 }
      },
      {
        fieldName: 'unit',
        dataType: 'ENUM',
        options: [
          { name: 'Celsius', value: 'Celsius' },
          { name: 'Fahrenheit', value: 'Fahrenheit' }
        ]
      }
    ]
  }
}
```


---

## 常用工具函数

### 1. 颜色转换

```javascript
// RGB 数字转 RGB 对象
numberToRGB(16777215) // { r: 255, g: 255, b: 255 }

// RGB 对象转数字
rgbToNumber(255, 255, 255) // 16777215

// 色温转 RGB
TempK2RGB(3000) // { r: 255, g: 180, b: 107 }
```

### 2. 温度转换

```javascript
// 摄氏度转华氏度
celsiusToFahrenheit(25) // 77

// 华氏度转摄氏度
fahrenheitToCelsius(77) // 25

// 温度值转 BLE 字节
temperatureToBleValue({
  autoStop: 1,
  temperature: 25,
  unit: 'Celsius',
  protocol: 1
}, capaInstance)
```

### 3. 分段控制

```javascript
// 分段数组转 BLE 值
segmentToBleValue(
  { segment: [1, 2, 3], rgb: 0xffffff },
  [1, 2, 3],
  'segmentedColorRgb'
)
```

### 4. BLE 指令处理

```javascript
// BLE 指令填充
bleCommandPadding('write', [0xff, 0x00, 0x00], commandDefineArr)

// Base64 转十六进制数组
base64ToHexArray('MwUTQGMAAAAAAAAAAAAAAAAAAAY=')

// 添加校验和
addCheckSum(uint8Array)

// Uint8Array 转 Base64
uint8ArrayToBase64(uint8Array, 'base64')
```

### 5. 字节操作

```javascript
// 获取字节特定位的值
getByteValueByIndex(0b10101010, 3) // 1

// 多字节转整数（无符号）
getUnSignedBytesToInt([0xff, 0x00], true)

// 多字节转整数（有符号）
getSignedInt([0xff, 0x00], true)

// 获取无符号字节
getUnsignedByte(-1) // 255
```

### 6. 字符串比较

```javascript
// 忽略大小写比较
equalsIgnoreCase('PowerSwitch', 'powerswitch') // true
```

### 7. 亮度映射

```javascript
// 映射亮度范围（1-100 映射到自定义范围）
mapBrightnessRange(50, 10, 255) // 132
```

---

## 核心函数实现模板

### 1. getCapabilities

```javascript
function getCapabilities(input) {
  let capabilities = Endpoint.capabilities
    .filter(item => 
      item.ctrlPlatformSupported && 
      item.ctrlPlatformSupported.includes('openApi')
    )
    .map(originalItem => {
      // 如果需要动态修改，进行深拷贝
      const item = needsModification(originalItem)
        ? JSON.parse(JSON.stringify(originalItem))
        : originalItem;
      
      // 调用拦截器
      capabilitiesInterceptor(item);
      
      return {
        type: item.type,
        instance: item.instance,
        parameters: item.parameters
      };
    });
    
  return {
    sku: Endpoint.productSku,
    type: Endpoint.productCategory,
    device: input.device,
    capabilities: capabilities
  };
}
```

### 2. getMqttPayload

```javascript
function getMqttPayload(input) {
  try {
    if (!input) {
      return {
        status: 400,
        message: 'Error: No arguments'
      };
    }
    
    const instance = input.instance.toLocaleUpperCase();
    const capaInstance = Endpoint.capabilities.find(capa => {
      return (
        capa.instance.toLocaleUpperCase() === instance &&
        capa.ctrlPlatformSupported.includes(input.appid)
      );
    });
    
    if (!capaInstance) {
      return {
        status: 400,
        transaction: input.transaction,
        message: `The instance of ${input.sku} not currently supported`
      };
    }
    
    const { sku, transaction, accountTopic = '' } = input;
    const type = capaInstance.payloadDefine.cmdType;
    const cmd = analysis.cmdHandler(capaInstance, input);
    const value = CmdTypeEnumDispatcher[type].payloadBuilder(
      transaction,
      cmd,
      accountTopic
    );
    
    // 特殊处理拦截器
    mqttPayloadInterceptor(input, capaInstance, value);
    
    return value;
  } catch (error) {
    console.log('getMqttPayload---', error);
    return {
      status: 500,
      transaction: input.transaction,
      message: 'getMqttPayload: something error'
    };
  }
}
```

### 3. getFullInstanceState

```javascript
function getFullInstanceState(input) {
  const { reportState, isFull } = input;
  let capabilities = [];
  
  Endpoint.capabilities.forEach((item) => {
    // 跳过不支持 openApi 的
    if (
      item.ctrlPlatformSupported === undefined ||
      !item.ctrlPlatformSupported.includes('openApi')
    ) {
      return;
    }
    
    // 填充状态值
    let instanceState = fillStateHandler.fillInstanceSate(
      {
        instance: item.instance,
        type: item.type,
        category: Endpoint.productCategory,
        reportState: reportState
      },
      LocalInstanceStateValueMap
    );
    
    // 处理全状态和非全状态
    if (isFull === 1) {
      if (instanceState === -1 || instanceState === -2) {
        instanceState = '';
      }
    } else {
      if (instanceState === -1) {
        instanceState = '';
      }
      if (instanceState === -2) {
        return; // 跳过
      }
    }
    
    capabilities.push({
      type: item.type,
      instance: item.instance,
      state: { value: instanceState }
    });
  });
  
  return {
    sku: Endpoint.productSku,
    type: Endpoint.productCategory,
    device: input.device,
    capabilities: capabilities
  };
}
```

### 4. 状态映射 (LocalInstanceStateValueMap)

```javascript
const LocalInstanceStateValueMap = new Map([
  [
    InstanceEnum.powerSwitch.value,
    {
      analysis: function (reportState, categoryType) {
        if (Object.keys(reportState).includes('power')) {
          return reportState.power;
        }
        return '';
      }
    }
  ],
  [
    InstanceEnum.brightness.value,
    {
      analysis: function (reportState, categoryType) {
        if (Object.keys(reportState).includes('brightness')) {
          return reportState.brightness;
        }
        return '';
      }
    }
  ]
]);
```

---

## BLE 指令配置说明

### action 配置结构

```javascript
action: {
  write: [
    {
      bleDefine: ['33', '05', '15', '01', 'ff'],  // BLE 指令字节数组
      valIndex: [4]                                // 数据值位置索引
    }
  ],
  read: [
    {
      bleDefine: ['aa', '05', '00'],
      valIndex: []
    }
  ]
}
```

### BLE 指令格式

- 第1字节：命令类型
  - `0x33`: 单包写
  - `0xaa`: 单包读
  - `0xa3`: 多包写读
  - `0x3a`: 单包写读

- 第2字节：功能码
- 第3-N字节：数据内容
- 最后1字节：校验和（自动计算）

### valIndex 说明

指定数据值在 BLE 指令中的位置，从0开始计数。

例如：
```javascript
bleDefine: ['33', '05', '15', 'ff', 'ff', 'ff']
valIndex: [3, 4, 5]
```
表示将数据的3个字节分别填充到指令的第3、4、5位置。

---

## 常见问题与解决方案

### 1. 如何处理音乐模式？

需要定义 MusicMode 数组并实现拦截器：

```javascript
const MusicMode = [
  {
    name: modeLanguageConst.Ripple,
    ctrData: ['base64string1', 'base64string2']
  }
];

function musicCapabilitiesInterceptor(item) {
  if (equalsIgnoreCase(item.instance, InstanceEnum.musicMode.value)) {
    item.parameters.fields = item.parameters.fields.map(field => {
      if (equalsIgnoreCase(field.fieldName, 'musicMode')) {
        return {
          ...field,
          options: MusicMode.map((curr, index) => ({
            name: curr.name.en,
            value: index
          }))
        };
      }
      return field;
    });
  }
}
```

### 2. 如何处理特殊的 MQTT Payload？

实现 mqttPayloadInterceptor：

```javascript
function mqttPayloadInterceptor(input, capaInstance, output) {
  if (equalsIgnoreCase(input.instance, InstanceEnum.musicMode.value)) {
    output.msg.data.command = MusicMode[input.data.musicMode].ctrData;
  }
  return output;
}
```

### 3. 如何支持多平台控制？

在 ctrlPlatformSupported 中配置：

```javascript
ctrlPlatformSupported: ['openApi']
```

**注意**：新 SKU 只需要支持 `openApi`，不再需要 `pad` 平台支持。

### 4. 如何处理温度单位转换？

使用工具函数：

```javascript
import { celsiusToFahrenheit, fahrenheitToCelsius } from '../../../utils/common.js';

// 在处理温度数据时自动转换
if (unit === 'Fahrenheit') {
  temperature = celsiusToFahrenheit(temperature);
}
```

---

## 最佳实践

1. **命名规范**
   - Instance 名称使用小驼峰：`powerSwitch`, `colorRgb`
   - 常量使用大写下划线：`PROTOCOL_BYTES_LEN`

2. **错误处理**
   - 所有核心函数都应有 try-catch
   - 返回统一的错误格式

3. **代码复用**
   - 相同逻辑提取为工具函数
   - 使用拦截器模式处理特殊情况

4. **性能优化**
   - 避免不必要的深拷贝
   - 使用 Map 而非对象进行状态映射

5. **可维护性**
   - 添加必要的注释
   - 保持函数职责单一
   - 遵循 SOLID 原则

---

## 完整示例导出

**新方法（推荐）**：
```javascript
export default {
  getMqttPayload,
  getCapabilities,
  getFullInstanceState,
  Endpoint,
  getMusicMode          // 如果有音乐模式
};
```

**旧方法（已废弃，不要使用）**：
```javascript
// ❌ 不要使用以下方法
export default {
  getBleRead,           // 已废弃
  getBleCommands,       // 已废弃
  getMqttPayload,
  getCapabilities,
  getFullInstanceState,
  Endpoint
};
```

---

## 参考资料

- H1232 完整示例：`references/H1232-template.md`
- utils/common.js：所有工具函数和枚举定义
- utils/analysis.js：指令分析处理
- utils/fillStateHandler.js：状态填充处理
