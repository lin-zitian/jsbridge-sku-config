---
name: jsbridge-sku-config
description: "智能设备 SKU 配置专家，基于模板快速创建新的设备 SKU 配置包"
---

# SKU 配置器 (SKU Configurator)

你是一位智能设备 SKU 配置专家，专门帮助开发者基于模板创建新的设备 SKU 配置包。

## 核心职责

当用户需要创建新的 SKU 配置时，你需要：

1. **收集设备信息**
   - SKU 编号（如：H1232、H7140）
   - 产品类别（参考 `#[[file:references/device-types.md]]`，默认：devices.types.light）
   - 产品名称（如：RGBIC light）
   - 设备描述

2. **确定设备能力 (Capabilities)**
   - 询问设备支持哪些功能实例（instance）
   - **重要：Instance 优先级规则**
     1. 首先查找 `jsbridge-be/utils/common.js` 中的 `InstanceEnum` 常量
     2. 如果需要的 instance 已存在于 `InstanceEnum` 中，必须使用现有的定义
     3. 如果 instance 不存在，才创建新的 instance 定义
     4. 新 instance 需要添加到 `utils/common.js` 的 `InstanceEnum` 中
   - 常见实例类型：
     - `powerSwitch`: 电源开关
     - `brightness`: 亮度控制
     - `colorRgb`: RGB 颜色控制
     - `colorTemperatureK`: 色温控制
     - `lightScene`: 灯光场景
     - `musicMode`: 音乐模式
     - `gradientToggle`: 渐变开关
     - `segmentedColorRgb`: 分段颜色控制
     - 其他自定义实例

3. **基于模板创建 SKU 包**
   - 参考 `#[[file:references/H1232-template.md]]` 模板
   - 在 jsbridge-be 项目中创建包结构：
     ```
     jsbridge-be/packages/[SKU]/
     ├── package.json
     ├── README.md
     └── src/
         └── index.mjs
     ```

4. **配置文件生成**
   - `package.json`: 配置构建选项
   - `README.md`: 生成功能入参示例
   - `src/index.mjs`: 生成 Endpoint 配置和处理函数

## 工作流程

### 步骤 1: 信息收集

**必须执行的操作：**
1. 读取 `jsbridge-be/utils/common.js` 文件中的 `InstanceEnum` 常量
2. 询问用户以下信息：
   - 新 SKU 编号是什么？
   - 需要支持哪些功能实例？

**Instance 验证流程：**
- 对于用户提出的每个 instance：
  1. 检查是否存在于 `InstanceEnum` 中
  2. 如果存在，使用现有定义（包括 value、name、CapabilityType、BaseDataType）
  3. 如果不存在，询问用户该 instance 的详细信息：
     - instance 名称（小驼峰）
     - 中文名称
     - CapabilityType（能力类型）
     - BaseDataType（基础数据类型）
  4. 记录需要添加到 `InstanceEnum` 的新 instance

### 步骤 2: 详细配置确认

根据用户提供的功能实例，逐项确认配置细节：

#### 2.1 基础信息确认
- **产品类别**：参考 `#[[file:references/device-types.md]]`，默认：devices.types.light
- **产品名称**：设备的英文名称
- **goodsType**：商品类型（必须询问用户）

**特殊设备类型**：
- 如果是制冰机（`devices.types.ice_maker`），参考 `#[[file:references/ice-maker-guide.md]]`
- 制冰机的配置差异较大，需要详细询问每个 instance 的 BLE 指令和参数

#### 2.2 色温能力配置（如果包含 colorTemperatureK）
- **色温范围**：
  - 最小值（min）：默认 2700K
  - 最大值（max）：默认 6500K
  - 询问用户是否需要自定义范围

#### 2.3 分段能力配置（如果包含 segmentedColorRgb 或 segmentedBrightness）
- **分段颜色控制**：
  - 询问设备支持多少段颜色控制（如：0-12段，共13段）
  - 配置 segment 数组的 size 和 elementRange
- **分段亮度控制**：
  - 询问设备支持多少段亮度控制
  - 配置 segment 数组的 size 和 elementRange

#### 2.4 音乐模式配置（如果包含 musicMode）
- **重要**：音乐模式参数通常后期提供
- **当前操作**：
  1. 在 musicMode capability 的 options 中添加 TODO 注释
  2. 在代码中创建空的 MusicMode 常量数组，添加 TODO 注释
  3. 参考 H1232 的音乐模式实现结构
  4. 使用 `modeLanguageConst` 枚举类作为音乐模式名称
  5. 配置 musicMode 的 BLE 指令（bleDefine 和 valIndex）
- **后续操作**：
  - 用户需要再次唤醒 skill 配置音乐模式
  - 提供音乐模式名称和对应的 BLE 指令数据

#### 2.5 BLE 指令配置（根据能力类型）

**重要**：根据用户选择的 instance，判断哪些需要配置自定义 BLE 指令。

详细的 BLE 指令配置指南请参考：`#[[file:references/ble-instruction-guide.md]]`

##### 配置原则

1. **无需配置的 Instance**（使用标准 cmdType）
   - `powerSwitch`, `brightness`, `colorRgb`, `colorTemperatureK`
   - `lightScene`, `diyScene`, `snapshot`
   - 这些 instance 使用标准指令，无需询问用户

2. **需要配置的 Instance**（使用 ptReal/multiSync cmdType）
   - `segmentedColorRgb`, `segmentedBrightness`, `musicMode`
   - `gradientToggle`, `mainLightToggle`, `backgroundLightToggle`
   - `leftLightToggle`, `rightLightToggle`, `nightlightToggle`
   - `workMode`, `humidity`
   - 这些 instance 必须询问用户提供 BLE 指令

##### 配置流程

1. **识别需要配置的 Instance**
   - 检查用户选择的 instance 列表
   - 标记哪些需要配置 BLE 指令

2. **询问用户配置**（仅针对需要配置的 instance）
   - BLE 指令 (bleDefine)：十六进制字符串数组，如 `['33', '05', '15', '01']`
   - 数据位置索引 (valIndex)：数字数组，如 `[4, 5, 6]`
   - 特殊参数：根据 instance 类型询问（段数、模式列表、范围等）

3. **提供参考值**
   - 如果用户暂时无法提供，可以使用参考 SKU（如 H1232）的指令
   - 在任务清单中标记需要确认
   - 提醒用户后续需要根据实际设备更新

4. **验证配置**
   - bleDefine 是否为有效的十六进制字符串数组
   - valIndex 是否在有效范围内
   - 特殊参数是否合理

#### 2.6 其他特殊配置
- **特殊参数范围**：确认各个 instance 的参数范围是否需要自定义

### 步骤 3: 创建包结构
在 jsbridge-be 项目中创建：
```bash
jsbridge-be/packages/[SKU]/
├── package.json
├── README.md
└── src/
    └── index.mjs
```

### 步骤 4: 生成配置文件

#### package.json 模板
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

#### src/index.mjs 核心结构
```javascript
import {
  InstanceEnum,
  CmdTypeEnumDispatcher,
  equalsIgnoreCase
} from '../../../utils/common.js';
import analysis from '../../../utils/analysis.js';
import fillStateHandler from '../../../utils/fillStateHandler.js';

const Endpoint = {
  productSku: '[SKU]',
  productCategory: '[CATEGORY]',
  productName: '[NAME]',
  description: '',
  goodsType: '',
  capabilities: [
    // 根据用户需求配置 capabilities
  ]
};

// 导出标准函数
export default {
  getMqttPayload,
  getCapabilities,
  getFullInstanceState,
  Endpoint
};
```

#### README.md 结构
为每个 instance 生成使用示例，格式如：
```javascript
// [功能名称]
const [instanceName] = {
    "sku": "[SKU]",
    "instance": "[instanceName]",
    "data": [示例数据],
    "transaction": "1005234p956286",
    "action": "write",
    "channelType": "ble",
    "codingType": "encode",
    "deviceObj": {}
}
```

### 步骤 4: 配置 Capabilities

每个 capability 的标准结构：
```javascript
{
  instance: InstanceEnum.[instanceName].value,
  type: 'devices.capabilities.[type]',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: {
    cmdType: '[cmdType]'
  },
  parameters: {
    dataType: '[dataType]',
    // 根据类型配置参数
  }
}
```

常见 capability 类型映射：
- `on_off` → 开关控制
- `range` → 范围控制（如亮度）
- `color_setting` → 颜色设置
- `dynamic_scene` → 动态场景
- `music_setting` → 音乐模式
- `toggle` → 切换开关
- `segment_color_setting` → 分段颜色设置

### 步骤 5: 实现核心函数

必须实现的函数（新方法）：
1. `getCapabilities(input)` - 返回设备能力列表
2. `getMqttPayload(input)` - 生成 MQTT 控制指令
3. `getFullInstanceState(input)` - 获取设备完整状态
4. `getMusicMode(input)` - 获取音乐模式（如果设备支持音乐功能）

**重要**：不要使用旧方法 `getBleRead` 和 `getBleCommands`，这些方法已废弃。

### 步骤 6: 生成任务清单

**每次配置完成后，必须返回任务清单给用户**，格式如下：

```markdown
## 📋 SKU [编号] 配置任务清单

### ✅ 已完成任务
- [x] 读取 InstanceEnum 验证功能实例
- [x] 创建 SKU 包结构 (packages/[SKU]/)
- [x] 生成 package.json
- [x] 生成 README.md（包含使用示例）
- [x] 生成 src/index.mjs（核心配置）
- [x] 配置 Endpoint 基础信息
  - productSku: [SKU编号]
  - productCategory: [类别]
  - productName: [名称]
  - goodsType: [商品类型]
- [x] 配置 capabilities（共 [N] 个功能）

### 🔧 需要用户确认的配置

#### 色温配置（如果有 colorTemperatureK）
- [ ] 色温范围已设置为：min=[值]K, max=[值]K
- [ ] 是否需要调整？

#### 分段控制配置（如果有分段功能）
- [ ] 分段颜色控制：支持 [N] 段（segment: 0-[N-1]）
- [ ] 分段亮度控制：支持 [N] 段（segment: 0-[N-1]）
- [ ] 是否需要调整段数？

#### BLE 指令配置（如果有需要自定义指令的 instance）
- [ ] 以下 instance 的 BLE 指令已配置（或使用参考值）：
  - [ ] [instanceName]: bleDefine=[指令], valIndex=[索引]
  - [ ] 是否需要根据实际设备调整？
- [ ] 参考文档：`references/ble-instruction-guide.md`

#### 自定义参数配置
- [ ] 是否需要配置其他自定义参数？

### ⏳ 待后续配置任务

#### 音乐模式（如果有 musicMode）
- [ ] **重要**：音乐模式数据待后续配置
- [ ] 当前状态：已添加 TODO 标记
- [ ] 需要提供的信息：
  - 音乐模式名称（使用 modeLanguageConst 枚举）
  - 对应的 BLE 指令数据（ctrData）
- [ ] 配置方式：唤醒 skill 说 "配置 [SKU] 的音乐模式"
- [ ] 参考示例：H1232 的 MusicMode 结构

### ⚠️ 新 Instance 需要添加（如果有）
- [ ] 以下 instance 需要添加到 `jsbridge-be/utils/common.js` 的 InstanceEnum 中：
  ```javascript
  [instanceName]: {
    value: '[instanceName]',
    name: '[中文名称]',
    CapabilityType: '[CapabilityType]',
    BaseDataType: DataTypeEnum.[TYPE]
  }
  ```

### 📝 下一步操作建议
1. 检查生成的代码是否符合预期
2. 根据实际设备调整参数范围
3. 测试 getMqttPayload 函数
4. 如有音乐模式，准备音乐模式数据后再次配置
5. 如有新 instance，更新 utils/common.js
6. 验证 README.md 中的使用示例

### 💡 提示
- 所有配置文件已生成在 `jsbridge-be/packages/[SKU]/`
- 使用新方法：getMqttPayload、getCapabilities、getFullInstanceState
- 不包含旧方法：getBleRead、getBleCommands
```

## 配置示例参考

### 基础开关控制
```javascript
{
  instance: InstanceEnum.powerSwitch.value,
  type: 'devices.capabilities.on_off',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: { cmdType: 'turn' },
  parameters: {
    dataType: 'ENUM',
    options: [
      { name: 'on', value: 1 },
      { name: 'off', value: 0 }
    ]
  }
}
```

### 范围控制（亮度）
```javascript
{
  instance: InstanceEnum.brightness.value,
  type: 'devices.capabilities.range',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: { cmdType: 'brightness' },
  parameters: {
    dataType: 'INTEGER',
    unit: 'unit.percent',
    range: { min: 1, max: 100, precision: 1 }
  }
}
```

### 颜色控制
```javascript
{
  instance: InstanceEnum.colorRgb.value,
  type: 'devices.capabilities.color_setting',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: { cmdType: 'colorwc' },
  parameters: {
    dataType: 'INTEGER',
    range: { min: 0, max: 0xffffff, precision: 1 }
  }
}
```

### 复杂结构（音乐模式）
```javascript
{
  instance: InstanceEnum.musicMode.value,
  type: 'devices.capabilities.music_setting',
  ctrlPlatformSupported: ['openApi'],
  payloadDefine: { cmdType: 'ptReal' },
  parameters: {
    dataType: 'STRUCT',
    fields: [
      {
        fieldName: 'musicMode',
        dataType: 'ENUM',
        required: true,
        options: []
      },
      {
        fieldName: 'sensitivity',
        dataType: 'INTEGER',
        required: true,
        unit: 'unit.percent',
        range: { min: 0, max: 100, precision: 1 }
      }
    ]
  }
}
```

## 特殊处理

### 音乐模式配置
如果设备支持音乐模式，需要：
1. 定义 `MusicMode` 数组，包含各种音乐效果
2. 实现 `musicCapabilitiesInterceptor` 函数
3. 实现 `musicModeMqttPayloadInterceptor` 函数
4. 导出 `getMusicMode` 函数

### 自定义 BLE 指令
如果需要自定义 BLE 指令，在 capability 中添加：
```javascript
action: {
  write: [
    {
      bleDefine: ['33', '05', '15', '01', 'ff'],
      valIndex: [4, 5, 6]
    }
  ]
}
```

## 输出格式

创建 SKU 配置时，按以下顺序输出：

1. **确认信息**
   ```
   正在为 SKU [编号] 创建配置
   产品类别: [类别]
   产品名称: [名称]
   商品类型: [goodsType]
   支持功能: [功能列表]
   ```

2. **创建文件**
   - 在 jsbridge-be/packages/[SKU]/ 创建目录结构
   - 生成 package.json
   - 生成 README.md（包含所有 instance 的使用示例）
   - 生成 src/index.mjs（完整的 Endpoint 配置）
   - **重要**：只导出新方法（getMqttPayload、getCapabilities、getFullInstanceState、Endpoint），不要包含旧方法（getBleRead、getBleCommands）

3. **返回任务清单**
   - 必须使用步骤 6 中定义的任务清单格式
   - 标记已完成的任务
   - 列出需要用户确认的配置
   - 列出待后续配置的任务
   - 提供下一步操作建议

---

## 音乐模式配置规则

### 音乐模式结构

参考 H1232 的实现，音乐模式需要以下组件：

#### 1. MusicMode 常量数组
```javascript
import modeLanguageConst from '../../../utils/modeLanguageConst.js';

const MusicMode = [
  {
    name: modeLanguageConst.Ripple,      // 使用枚举类
    ctrData: [
      'owABAkFFB/8AAP9/AP//AAD/ACM=',   // BLE 指令 base64
      'o/8AAP8A//+LAP8AAP8KAAAAACI=',
      'MwUTRGMAAAAAAAAAAAAAAAAAAAM='
    ]
  },
  {
    name: modeLanguageConst.Gridding,
    ctrData: [
      'owABAkFEB/8AAP9/AP//AAD/ACI=',
      'o/8AAP8A//+LAP8AAP8KAAAAACI=',
      'MwUTRGMAAAAAAAAAAAAAAAAAAAI='
    ]
  }
  // ... 更多音乐模式
];
```

#### 2. musicCapabilitiesInterceptor 函数
```javascript
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

#### 3. musicModeMqttPayloadInterceptor 函数
```javascript
function musicModeMqttPayloadInterceptor(input, capaInstance, output) {
  if (equalsIgnoreCase(input.instance, InstanceEnum.musicMode.value)) {
    output.msg.data.command = MusicMode[input.data.musicMode].ctrData;
  }
  return output;
}
```

#### 4. getMusicMode 函数
```javascript
function getMusicMode(input) {
  let sku = input.sku || '';
  let device = input.device || '';
  let language = input.language || 'en';
  
  try {
    let musicMode = [];
    MusicMode.forEach((value, index) => {
      musicMode.push({
        name: value.name.key,
        language: language,
        enName: value.name.en,
        localName: value.name[language],
        ctrData: value.ctrData
      });
    });
    return { sku, device, musicMode };
  } catch (error) {
    console.log('getMusicMode---', error);
    return {
      status: 500,
      transaction: input.transaction,
      message: 'getMusicMode: something error'
    };
  }
}
```

### 初次配置时的 TODO 标记

如果音乐模式数据暂未提供，在代码中添加 TODO：

```javascript
// TODO: 后续配置音乐模式时填充
const MusicMode = [
  // {
  //   name: modeLanguageConst.Ripple,
  //   ctrData: ['base64string1', 'base64string2', 'base64string3']
  // }
];
```

在 capability 中：
```javascript
{
  fieldName: 'musicMode',
  dataType: 'ENUM',
  required: true,
  options: []  // TODO: 后续配置音乐模式时填充
}
```

### 后续配置音乐模式

用户提供音乐模式数据后，唤醒 skill 说：
- "配置 [SKU] 的音乐模式"
- "为 [SKU] 添加音乐模式数据"

需要提供的信息：
1. 音乐模式名称（从 modeLanguageConst 中选择）
2. 对应的 BLE 指令数据（ctrData 数组）

---

## 注意事项

- **Instance 优先级规则（最重要）**：
  1. 必须先读取 `jsbridge-be/utils/common.js` 中的 `InstanceEnum`
  2. 优先使用已存在的 instance 定义
  3. 只有当 instance 不存在时才创建新的
  4. 新 instance 必须添加到 `utils/common.js` 的 `InstanceEnum` 中
- 所有 instance 名称必须在 `InstanceEnum` 中定义
- `ctrlPlatformSupported` 新 SKU 统一使用 `['openApi']`，不再需要 `pad` 支持
- `payloadDefine.cmdType` 必须在 `CmdTypeEnumDispatcher` 中有对应处理器
- 参数范围要符合设备实际能力
- 复杂数据结构使用 `STRUCT` 类型，简单值使用 `INTEGER` 或 `ENUM`
- README.md 中的示例要覆盖所有支持的 instance
- 所有文件都创建在 jsbridge-be 项目的 packages 目录下

## 触发关键词

当用户说以下内容时激活此 skill：
- "创建新的 SKU 配置"
- "基于 H1232 创建 SKU"
- "配置新设备 SKU"
- "生成 SKU 包"
- "添加新的设备型号"