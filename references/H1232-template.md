# H1232 SKU 配置模板

这是一个完整的 SKU 配置示例，用于创建新的智能灯设备配置。

## 设备信息

- **SKU**: H1232
- **产品类别**: devices.types.light
- **产品名称**: RGBIC light
- **设备类型**: 智能 RGBIC 灯带

## 支持的功能实例 (Instances)

### 基础控制
- `powerSwitch`: 电源开关
- `brightness`: 整体亮度控制
- `gradientToggle`: 颜色渐变开关

### 颜色控制
- `colorRgb`: RGB 颜色控制
- `colorTemperatureK`: 色温控制
- `segmentedColorRgb`: 分段颜色控制
- `segmentedBrightness`: 分段亮度控制

### 场景与模式
- `lightScene`: 灯光场景切换
- `diyScene`: DIY 场景
- `musicMode`: 音乐模式
- `snapshot`: 快照

### 高级控制
- `mainLightToggle`: 主灯开关
- `backgroundLightToggle`: 背灯开关

## 文件结构

```
packages/H1232/
├── package.json          # 构建配置
├── README.md            # 使用示例文档
└── src/
    └── index.mjs        # 核心配置代码
```

## package.json

```json
{
  "main": "./dist/H1232.global.js",
  "license": "MIT",
  "buildOptions": {
    "name": "H1232",
    "formats": ["global"]
  }
}
```

## README.md 示例结构

每个功能实例都需要提供使用示例：

```javascript
// 电源开关 - 开
const powerSwitch = {
    "sku": "H1232",
    "instance": "powerSwitch",
    "data": 1,  // 0关 1开
    "transaction": "1005234p956286",
    "action": "write",
    "channelType": "ble",
    "codingType": "encode",
    "deviceObj": {}
}

// 亮度控制 - 50%
const brightness = {
    "sku": "H1232",
    "instance": "brightness",
    "data": 50,  // 1-100
    "transaction": "1005234p956286",
    "action": "write",
    "channelType": "ble",
    "codingType": "encode",
    "deviceObj": {}
}

// 颜色控制 - 白色
const colorRgb = {
    "sku": "H1232",
    "instance": "colorRgb",
    "data": 16777215,  // 0xFFFFFF
    "transaction": "1005234p956286",
    "action": "write",
    "channelType": "ble",
    "codingType": "encode",
    "deviceObj": {}
}

// 分段颜色控制
const segmentedColorRgb = {
    "sku": "H1232",
    "instance": "segmentedColorRgb",
    "data": {
        "segment": [1, 2, 3],  // 段号数组
        "rgb": 16777215        // RGB 值
    },
    "transaction": "1005234p956286",
    "action": "write",
    "channelType": "ble",
    "codingType": "encode",
    "deviceObj": {}
}

// 音乐模式
const musicMode = {
    "sku": "H1232",
    "instance": "musicMode",
    "data": {
        "musicMode": 5,      // 音乐模式编号
        "sensitivity": 50,   // 灵敏度 0-100
        "subEffect": 0,      // 子效果
        "autoColor": 0,      // 0开 1关
        "rgb": 0xffffff      // RGB 颜色
    },
    "transaction": "1005234p956286",
    "action": "write",
    "channelType": "ble",
    "codingType": "encode",
    "deviceObj": {}
}
```

## src/index.mjs 核心结构

### 1. 导入依赖

```javascript
import {
  InstanceEnum,
  CmdTypeEnumDispatcher,
  equalsIgnoreCase
} from '../../../utils/common.js';
import analysis from '../../../utils/analysis.js';
import fillStateHandler from '../../../utils/fillStateHandler.js';
import modeLanguageConst from '../../../utils/modeLanguageConst.js';
```

### 2. Endpoint 配置

```javascript
const Endpoint = {
  productSku: 'H1232',
  productCategory: 'devices.types.light',
  productName: 'RGBIC light',
  description: '',
  goodsType: '',
  capabilities: [
    // 所有功能能力配置
  ]
};
```

### 3. Capability 配置示例

#### 开关控制 (on_off)
```javascript
{
  instance: InstanceEnum.powerSwitch.value,
  type: 'devices.capabilities.on_off',
  ctrlPlatformSupported: ['openApi', 'pad'],
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

#### 范围控制 (range) - 亮度
```javascript
{
  instance: InstanceEnum.brightness.value,
  type: 'devices.capabilities.range',
  ctrlPlatformSupported: ['openApi', 'pad'],
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

#### 颜色设置 (color_setting)
```javascript
{
  instance: InstanceEnum.colorRgb.value,
  type: 'devices.capabilities.color_setting',
  ctrlPlatformSupported: ['openApi', 'pad'],
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

#### 分段颜色设置 (segment_color_setting)
```javascript
{
  instance: InstanceEnum.segmentedColorRgb.value,
  type: 'devices.capabilities.segment_color_setting',
  ctrlPlatformSupported: ['openApi', 'pad'],
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

#### 音乐模式 (music_setting)
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
        options: []  // 动态填充
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

#### 自定义 BLE 指令 (toggle)
```javascript
{
  instance: InstanceEnum.gradientToggle.value,
  type: 'devices.capabilities.toggle',
  ctrlPlatformSupported: ['openApi', 'pad'],
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

### 4. 音乐模式数据（如果需要）

```javascript
const MusicMode = [
  {
    name: modeLanguageConst.Ripple,
    ctrData: ['owABAkFFB/8AAP9/AP//AAD/ACM=', '...']
  },
  {
    name: modeLanguageConst.Gridding,
    ctrData: ['owABAkFEB/8AAP9/AP//AAD/ACI=', '...']
  },
  // ... 更多音乐模式
];
```

### 5. 核心函数实现

#### getCapabilities
```javascript
function getCapabilities(input) {
  let capabilities = Endpoint.capabilities
    .filter(item => 
      item.ctrlPlatformSupported && 
      item.ctrlPlatformSupported.includes('openApi')
    )
    .map(originalItem => {
      const item = equalsIgnoreCase(
        originalItem.instance,
        InstanceEnum.musicMode.value
      ) ? JSON.parse(JSON.stringify(originalItem)) : originalItem;
      
      musicCapabilitiesInterceptor(item);
      
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

#### getMqttPayload
```javascript
function getMqttPayload(input) {
  try {
    if (input) {
      const instance = input.instance.toLocaleUpperCase();
      let capaInstance = Endpoint.capabilities.find(capa => {
        return (
          capa.instance.toLocaleUpperCase() === instance &&
          capa.ctrlPlatformSupported.includes(input.appid)
        );
      });
      
      const { sku, transaction, accountTopic = '' } = input;
      
      if (capaInstance) {
        const type = capaInstance.payloadDefine.cmdType;
        const cmd = analysis.cmdHandler(capaInstance, input);
        const value = CmdTypeEnumDispatcher[type].payloadBuilder(
          transaction,
          cmd,
          accountTopic
        );
        
        // 特殊处理（如音乐模式）
        musicModeMqttPayloadInterceptor(input, capaInstance, value);
        
        return value;
      } else {
        return {
          status: 400,
          transaction: transaction,
          message: `The instance of ${sku} not currently supported`
        };
      }
    } else {
      return {
        status: 400,
        message: 'Error: No arguments'
      };
    }
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

#### getFullInstanceState
```javascript
function getFullInstanceState(input) {
  let reportState = input.reportState;
  let isFull = input.isFull;
  let capabilities = [];
  
  Endpoint.capabilities.forEach((item, index, arr) => {
    if (
      item.ctrlPlatformSupported === undefined ||
      !item.ctrlPlatformSupported.includes('openApi')
    ) {
      return;
    }
    
    let instanceSate = fillStateHandler.fillInstanceSate(
      {
        instance: item.instance,
        type: item.type,
        category: Endpoint.productCategory,
        reportState: reportState
      },
      LocalInstanceStateValueMap
    );
    
    if (isFull === 1) {
      if (instanceSate === -1) instanceSate = '';
      if (instanceSate === -2) instanceSate = '';
    } else {
      if (instanceSate === -1) instanceSate = '';
      if (instanceSate === -2) return;
    }
    
    let capability = {
      type: item.type,
      instance: item.instance,
      state: { value: instanceSate }
    };
    capabilities.push(capability);
  });
  
  return {
    sku: Endpoint.productSku,
    type: Endpoint.productCategory,
    device: input.device,
    capabilities: capabilities
  };
}
```

### 6. 导出

```javascript
export default {
  getMusicMode,      // 如果有音乐模式
  getMqttPayload,
  getCapabilities,
  getFullInstanceState,
  Endpoint
};
```

## 关键配置说明

### cmdType 类型
- `turn`: 开关控制
- `brightness`: 亮度控制
- `colorwc`: 颜色/色温控制
- `ptReal`: 自定义 BLE 指令

### ctrlPlatformSupported
- `openApi`: 支持 Open API 控制
- `pad`: 支持 Pad 控制

### dataType 类型
- `ENUM`: 枚举类型（如开关）
- `INTEGER`: 整数类型（如亮度、颜色值）
- `STRUCT`: 结构体类型（如音乐模式的复杂参数）
- `Array`: 数组类型（如分段控制的段号）

### BLE 指令配置
- `bleDefine`: BLE 指令字节数组
- `valIndex`: 数据值在指令中的位置索引

## 使用此模板创建新 SKU

1. 复制 H1232 目录结构
2. 修改 SKU 编号和产品信息
3. 根据设备实际功能调整 capabilities
4. 更新 README.md 中的使用示例
5. 测试各个功能实例的控制指令
