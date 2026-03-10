# 制冰机 SKU 配置指南

本文档专门说明制冰机类型 SKU 的配置要点和注意事项。

## 制冰机特点

制冰机（`devices.types.ice_maker`）是一个功能复杂的设备类型，不同型号之间的 BLE 指令和参数配置存在较大差异。

## 常用功能实例

制冰机通常包含以下 instance：

### 核心控制
- `powerSwitch`: 电源开关 ⚠️ 指令可能不同
- `iceMakingToggle`: 制冰/清洗开关 ⚠️ options 数量不同
- `workMode`: 制冰模式 ⚠️ 指令和 options 差异最大

### 夜灯控制（指令基本一致）
- `nightlightToggle`: 夜灯开关 ✅ 指令一致
- `brightness`: 夜灯亮度 ✅ 指令一致
- `colorRgb`: 夜灯颜色 ✅ 指令一致
- `nightlightScene`: 夜灯场景 ⚠️ options 不同

### 其他功能
- `precoolToggle`: 预冷开关 ✅ 指令一致

### 事件通知
- `lackWaterEvent`: 缺水事件 ⚠️ alarmType 可能不同
- `iceFullEvent`: 冰满事件 ⚠️ alarmType 可能不同
- `cleaningCompletedEvent`: 清洁完成事件
- `runInterruptEvent`: 运行中断事件

---

## 指令差异对比

### 1. powerSwitch (电源开关)

**H8120/H8121/H8122**:
```javascript
{
  instance: InstanceEnum.powerSwitch.value,
  type: 'devices.capabilities.on_off',
  ctrlPlatformSupported: ['openApi', 'pad'],
  action: {
    write: [
      {
        bleDefine: ['33', '01'],  // 使用 '33' 开头
        valIndex: [2]
      }
    ],
    read: ['aa', '01']
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

**H8131**:
```javascript
{
  instance: InstanceEnum.powerSwitch.value,
  type: 'devices.capabilities.on_off',
  ctrlPlatformSupported: ['openApi', 'pad'],
  action: {
    write: [
      {
        bleDefine: ['3a', '01'],  // 使用 '3a' 开头 ⚠️
        valIndex: [2]
      }
    ],
    read: ['aa', '01']
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

**配置建议**：询问用户 BLE 指令的第一个字节是 '33' 还是 '3a'

---

### 2. iceMakingToggle (制冰/清洗开关)

**H8120/H8121** (2个选项):
```javascript
{
  instance: InstanceEnum.iceMakingToggle.value,
  type: 'devices.capabilities.toggle',
  ctrlPlatformSupported: ['openApi', 'pad'],
  action: {
    write: [
      {
        bleDefine: ['33', '01', '01', '00'],
        valIndex: [3]
      }
    ],
    read: ['aa', '01']
  },
  payloadDefine: {
    cmdType: 'multiSync'
  },
  parameters: {
    dataType: 'ENUM',
    options: [
      { name: 'iceMaking', value: 0 },
      { name: 'Clean', value: 1 }
    ]
  }
}
```

**H8131** (4个选项):
```javascript
{
  instance: InstanceEnum.iceMakingToggle.value,
  type: 'devices.capabilities.toggle',
  ctrlPlatformSupported: ['openApi', 'pad'],
  action: {
    write: [
      {
        bleDefine: ['3a', '01', '01', '00'],  // '3a' 开头
        valIndex: [3]
      }
    ],
    read: ['aa', '01']
  },
  payloadDefine: {
    cmdType: 'multiSync'
  },
  parameters: {
    dataType: 'ENUM',
    options: [
      { name: 'Ice Making', value: 0 },
      { name: 'Deep Cleaning', value: 1 },
      { name: 'Defrosting', value: 2 },
      { name: 'Cleaning', value: 3 }
    ]
  }
}
```

**配置建议**：
1. 询问用户 BLE 指令的第一个字节（'33' 或 '3a'）
2. 询问用户支持几种模式（2种或4种）
3. 询问用户每种模式的名称和值

---

### 3. workMode (制冰模式) ⚠️ 差异最大

**H8120/H8121** (嵌套结构):
```javascript
{
  instance: InstanceEnum.workMode.value,
  type: 'devices.capabilities.work_mode',
  ctrlPlatformSupported: ['openApi', 'pad'],
  action: {
    write: [
      {
        bleDefine: ['3A', '05', '01', '01'],
        valIndex: [2, 3]
      }
    ],
    writeStart: [
      {
        bleDefine: ['3A', '05', '01', '01'],
        valIndex: [2, 3]
      },
      {
        bleDefine: ['3A', '01', '01', '01']
      }
    ],
    read: ['aa', '05']
  },
  payloadDefine: {
    cmdType: 'multiSync'
  },
  parameters: {
    dataType: 'STRUCT',
    fields: [
      {
        fieldName: 'workMode',
        dataType: 'ENUM',
        required: true,
        options: [{ name: 'IceMakingMode', value: 1 }]
      },
      {
        fieldName: 'modeValue',
        dataType: 'ENUM',
        required: true,
        options: [
          {
            name: 'IceMakingMode',
            options: [
              { name: 'Small Nugget', value: 1 },
              { name: 'Medium Nugget', value: 2 },
              { name: 'Large Nugget', value: 3 }
            ]
          }
        ]
      }
    ]
  }
}
```

**H8122** (平铺结构):
```javascript
{
  instance: InstanceEnum.workMode.value,
  type: 'devices.capabilities.work_mode',
  ctrlPlatformSupported: ['openApi', 'pad'],
  action: {
    write: [
      {
        bleDefine: ['3A', '05', '01', '00'],  // 最后一个字节不同
        valIndex: [2, 3]
      },
      {
        bleDefine: ['3A', '01', '01', '01']
      }
    ],
    writeStart: [
      {
        bleDefine: ['3A', '05', '01', '00'],
        valIndex: [2, 3]
      },
      {
        bleDefine: ['3A', '01', '01', '01']
      }
    ],
    read: ['aa', '05']
  },
  payloadDefine: {
    cmdType: 'multiSync'
  },
  parameters: {
    dataType: 'STRUCT',
    fields: [
      {
        fieldName: 'workMode',
        dataType: 'ENUM',
        required: true,
        options: [
          { name: 'Small Nugget', value: 1 },
          { name: 'Medium Nugget', value: 2 },
          { name: 'Large Nugget', value: 3 },
          { name: 'Clean', value: 1 }
        ]
      },
      {
        fieldName: 'modeValue',
        dataType: 'ENUM',
        required: false,
        options: [
          { name: 'Small Nugget', value: 0 },
          { name: 'Medium Nugget', value: 0 },
          { name: 'Large Nugget', value: 0 },
          { name: 'Clean', value: 1 }
        ]
      }
    ]
  }
}
```

**H8131** (完全不同):
```javascript
{
  instance: InstanceEnum.workMode.value,
  type: 'devices.capabilities.work_mode',
  ctrlPlatformSupported: ['openApi', 'pad'],
  action: {
    write: [
      {
        bleDefine: ['3a', '05'],  // 更简短
        valIndex: [2, 3]
      }
    ],
    read: ['aa', '05']
  },
  payloadDefine: {
    cmdType: 'multiSync'
  },
  parameters: {
    dataType: 'STRUCT',
    fields: [
      {
        fieldName: 'workMode',
        dataType: 'ENUM',
        required: true,
        options: [
          { name: 'Thin', value: 1 },
          { name: 'Medium', value: 2 },
          { name: 'Thick', value: 3 },
          { name: 'SuperThick', value: 4 }
        ]
      },
      {
        fieldName: 'modeValue',
        dataType: 'ENUM',
        required: true,
        options: [
          { name: 'Small', value: 1 },
          { name: 'Medium', value: 2 },
          { name: 'Large', value: 3 }
        ]
      }
    ]
  }
}
```

**配置建议**：
1. 询问用户 BLE 指令（完整的 bleDefine 数组）
2. 询问用户是否需要 writeStart（H8131 不需要）
3. 询问用户 workMode 的 options（模式名称和值）
4. 询问用户 modeValue 的 options（子模式名称和值）
5. 询问用户参数结构（嵌套或平铺）

---

### 4. nightlightScene (夜灯场景)

**H8120/H8121** (15个场景):
```javascript
{
  instance: InstanceEnum.nightlightScene.value,
  type: 'devices.capabilities.mode',
  ctrlPlatformSupported: ['openApi'],
  action: {
    write: [
      {
        bleDefine: ['3a', '1b', '05', '13', '01'],
        valIndex: [4]
      }
    ]
  },
  payloadDefine: {
    cmdType: 'multiSync'
  },
  parameters: {
    dataType: 'ENUM',
    options: [
      { name: 'Party', value: 1 },
      { name: 'Gathering', value: 2 },
      { name: 'Activity', value: 3 },
      { name: 'Dinner', value: 4 },
      { name: 'Wine bureau', value: 5 },
      { name: 'Celebration', value: 6 },
      { name: 'Morning', value: 7 },
      { name: 'Afternoon', value: 8 },
      { name: 'Night', value: 9 },
      { name: 'Leisure', value: 0xa },
      { name: 'Music', value: 0xb },
      { name: 'Christmas', value: 0xc },
      { name: 'Halloween', value: 0xd },
      { name: 'Easter', value: 0xe },
      { name: 'Thanksgiving Day', value: 0xf }
    ]
  }
}
```

**H8122** (5个场景，指令不同):
```javascript
{
  instance: InstanceEnum.nightlightScene.value,
  type: 'devices.capabilities.mode',
  ctrlPlatformSupported: ['openApi'],
  action: {
    write: [
      {
        bleDefine: ['3a', '1b', '05', '13', '00', '01'],  // 多一个字节
        valIndex: [4, 5]  // valIndex 也不同
      }
    ]
  },
  payloadDefine: {
    cmdType: 'multiSync'
  },
  parameters: {
    dataType: 'ENUM',
    options: [
      { name: 'Gathering', value: 1 },
      { name: 'Party', value: 2 },
      { name: 'Ecology', value: 3 },
      { name: 'Night Light', value: 4 },
      { name: 'Food Waste', value: 5 }
    ]
  }
}
```

**H8131** (5个场景):
```javascript
{
  instance: InstanceEnum.nightlightScene.value,
  type: 'devices.capabilities.mode',
  ctrlPlatformSupported: ['openApi'],
  action: {
    write: [
      {
        bleDefine: ['3a', '1b', '05', '13', '01'],
        valIndex: [4]
      }
    ]
  },
  payloadDefine: {
    cmdType: 'multiSync'
  },
  parameters: {
    dataType: 'ENUM',
    options: [
      { name: 'Forest', value: 1 },
      { name: 'Ocean', value: 2 },
      { name: 'Wetlands', value: 3 },
      { name: 'Leisurely', value: 4 },
      { name: 'Sleep', value: 5 }
    ]
  }
}
```

**配置建议**：
1. 询问用户 BLE 指令（bleDefine 和 valIndex）
2. 询问用户支持多少个场景
3. 询问用户每个场景的名称和值

---

### 5. 事件 alarmType

**H8122**:
- `lackWaterEvent`: alarmType = 51
- `iceFullEvent`: alarmType = 58

**H8120/H8121/H8131**:
- `lackWaterEvent`: alarmType = 17406
- `iceFullEvent`: alarmType = 17603
- `cleaningCompletedEvent`: alarmType = 17609
- `runInterruptEvent`: alarmType = 17410

**配置建议**：询问用户每个事件的 alarmType 值

---

## 指令一致的 Instance

以下 instance 在所有制冰机中指令基本一致，可以使用默认值：

### precoolToggle (预冷开关)
```javascript
{
  instance: InstanceEnum.precoolToggle.value,
  type: 'devices.capabilities.toggle',
  ctrlPlatformSupported: ['openApi'],
  action: {
    write: [
      {
        bleDefine: ['3a', '1f', '16'],
        valIndex: [3]
      }
    ]
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

### nightlightToggle (夜灯开关)
```javascript
{
  instance: InstanceEnum.nightlightToggle.value,
  type: 'devices.capabilities.toggle',
  ctrlPlatformSupported: ['openApi'],
  action: {
    write: [
      {
        bleDefine: ['3a', '1b', '01', '01', '01'],
        valIndex: [4]
      }
    ]
  },
  payloadDefine: {
    cmdType: 'multiSync'
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

### brightness (夜灯亮度)
```javascript
{
  instance: InstanceEnum.brightness.value,
  type: 'devices.capabilities.range',
  ctrlPlatformSupported: ['openApi'],
  action: {
    write: [
      {
        bleDefine: ['3a', '1b', '01', '02', '32'],
        valIndex: [4]
      }
    ]
  },
  payloadDefine: {
    cmdType: 'multiSync'
  },
  parameters: {
    dataType: 'INTEGER',
    range: {
      min: 1,
      max: 100,
      precision: 1
    }
  }
}
```

### colorRgb (夜灯颜色)
```javascript
{
  instance: InstanceEnum.colorRgb.value,
  type: 'devices.capabilities.color_setting',
  ctrlPlatformSupported: ['openApi'],
  action: {
    write: [
      {
        bleDefine: ['3a', '1b', '05', '0d', '32', '32', '32'],
        valIndex: [4, 5, 6]
      }
    ],
    read: []
  },
  payloadDefine: {
    cmdType: 'multiSync'
  },
  parameters: {
    dataType: DataTypeEnum.INTEGER,
    range: {
      min: 0,
      max: 0xffffff,
      precision: 1
    }
  }
}
```

---

## 配置流程建议

### 步骤 1: 确认基础信息
- SKU 编号
- 产品类别：`devices.types.ice_maker`
- goodsType

### 步骤 2: 确认功能实例
询问用户设备支持哪些功能：
- [ ] powerSwitch (必选)
- [ ] iceMakingToggle (必选)
- [ ] workMode (必选)
- [ ] precoolToggle (可选)
- [ ] nightlightToggle (可选)
- [ ] brightness (可选，需要 nightlightToggle)
- [ ] colorRgb (可选，需要 nightlightToggle)
- [ ] nightlightScene (可选，需要 nightlightToggle)
- [ ] 事件通知（lackWaterEvent, iceFullEvent 等）

### 步骤 3: 配置差异化指令

对于每个需要配置的 instance，按优先级询问：

#### 高优先级（必须询问）
1. **powerSwitch**
   - BLE 指令第一个字节：'33' 或 '3a'？

2. **iceMakingToggle**
   - BLE 指令第一个字节：'33' 或 '3a'？
   - 支持几种模式：2种或4种？
   - 每种模式的名称和值

3. **workMode**（最复杂）
   - 完整的 BLE 指令（bleDefine 数组）
   - 是否需要 writeStart？
   - workMode 的 options（模式列表）
   - modeValue 的 options（子模式列表）
   - 参数结构：嵌套或平铺？

4. **nightlightScene**（如果有）
   - BLE 指令（bleDefine 和 valIndex）
   - 支持多少个场景？
   - 每个场景的名称和值

5. **事件 alarmType**
   - 每个事件的 alarmType 值

#### 低优先级（可使用默认值）
- `precoolToggle`, `nightlightToggle`, `brightness`, `colorRgb` 使用标准指令

### 步骤 4: 生成配置文件

根据用户提供的信息生成完整的配置文件。

### 步骤 5: 返回任务清单

特别标注制冰机的配置差异项，提醒用户确认。

---

## 参考 SKU

- **H8120**: 110V 制冰机，15个夜灯场景
- **H8121**: 110V 制冰机，15个夜灯场景（与 H8120 几乎相同）
- **H8122**: 制冰机，5个夜灯场景，workMode 结构不同
- **H8131**: 110V 制冰机，5个夜灯场景，指令差异最大

---

## 常见问题

### Q: 为什么制冰机的指令差异这么大？

A: 制冰机是功能复杂的设备，不同硬件版本、不同供应商的设备使用不同的通信协议和指令集。

### Q: 如何判断使用哪种指令？

A: 需要根据实际设备的硬件规格和通信协议文档来确定。如果不确定，可以先参考相似型号的 SKU。

### Q: workMode 的嵌套结构和平铺结构有什么区别？

A: 
- **嵌套结构**（H8120/H8121）：workMode 只有一个选项，modeValue 根据 workMode 的值有不同的子选项
- **平铺结构**（H8122）：workMode 直接列出所有模式，modeValue 对应每个模式的值

### Q: 是否所有制冰机都需要夜灯功能？

A: 不一定。夜灯功能是可选的，根据实际设备配置决定。

### Q: 事件的 alarmType 如何确定？

A: alarmType 是事件通知的唯一标识，需要根据设备的事件定义文档来确定。不同设备可能使用不同的 alarmType 值。

---

## 最佳实践

1. **详细询问**：制冰机配置差异大，必须详细询问用户每个配置项
2. **提供参考**：为用户提供参考 SKU（H8120/H8131）的配置示例
3. **分步确认**：先配置核心功能（powerSwitch, iceMakingToggle, workMode），再配置可选功能
4. **标记差异**：在任务清单中明确标记哪些配置使用了默认值，哪些需要用户确认
5. **测试验证**：配置完成后，建议用户测试每个功能是否正常工作

---

## 参考文档

- 标准 SKU 模板：`standard-sku-template.md`
- BLE 指令配置指南：`ble-instruction-guide.md`
- 设备类型参考：`device-types.md`
