# SKU 配置系统更新日志

## 2024-03-10 - 重大更新

### 1. goodsType 数据类型规范 ✅

**问题**: goodsType 数据类型不明确，可能被错误地配置为字符串

**解决方案**:
- 明确 goodsType 的数据类型规则
- 用户提供时必须使用数字类型（Number）
- 用户未提供时使用空字符串

**数据类型规则**:
- **用户提供时**: 使用数字类型
  - 示例: `goodsType: 69`
  - 示例: `goodsType: 321`
  - 示例: `goodsType: 184`
- **用户未提供时**: 使用空字符串
  - 示例: `goodsType: ''`

**错误示例**:
```javascript
// ❌ 错误：不要使用字符串格式的数字
goodsType: '69'
goodsType: "321"
```

**正确示例**:
```javascript
// ✅ 正确：用户提供时使用数字
goodsType: 69
goodsType: 321

// ✅ 正确：用户未提供时使用空字符串
goodsType: ''
```

**参考示例**:
- H8131: `goodsType: 321` (数字)
- H608D: `goodsType: 184` (数字)
- H713E: `goodsType: ''` (空字符串)

**更新文档**:
- `SKILL.md` - 更新基础信息确认、Endpoint 模板、任务清单
- `configuration-workflow.md` - 更新基础信息说明、配置检查清单

---

### 2. 外部填充数据规范 ✅

**问题**: AI 会编造 lightScene、diyScene、snapshot、musicMode 的 options 数据

**解决方案**:
- 明确这些字段的 options 必须为空数组 `[]`
- 添加详细的外部填充数据说明文档
- 在 SKILL.md 中添加明确的禁止编造数据规则

**影响的 instance**:
- `lightScene`: options 为空数组，由外部系统填充
- `diyScene`: options 为空数组，由外部系统填充
- `snapshot`: options 为空数组，由外部系统填充
- `musicMode`: musicMode 字段的 options 为空数组，后续填充

**新增文档**:
- `references/external-data-fields.md` - 外部填充数据字段详细说明

---

### 2. 外部填充数据规范 ✅

**问题**: musicMode 缺少 action 配置（BLE 指令）

**解决方案**:
- 明确 musicMode 必须包含完整的 action 配置
- 即使 musicMode 字段的 options 为空数组，也必须配置 action
- 提供标准的 musicMode action 配置模板

**正确配置**:
```javascript
{
  instance: InstanceEnum.musicMode.value,
  type: 'devices.capabilities.music_setting',
  ctrlPlatformSupported: ['openApi'],
  action: {  // ✅ 必须包含
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
        options: []  // ✅ 空数组，后续填充
      },
      // ... 其他字段
    ]
  }
}
```

---

### 3. musicMode 必须包含 action 配置 ✅

**问题**: 不清楚哪些 instance 需要配置 action，哪些使用固定指令

**解决方案**:
- 明确列出 7 个使用固定指令的 instance
- 规定所有其他 instance 都需要配置 action
- 如果用户暂时无法提供，配置空 action + TODO 注释

**固定指令能力（无需 action）**:
1. `powerSwitch` - cmdType: turn
2. `brightness` - cmdType: brightness
3. `colorRgb` - cmdType: colorwc/color
4. `colorTemperatureK` - cmdType: colorwc
5. `lightScene` - 外部填充
6. `diyScene` - 外部填充
7. `snapshot` - 外部填充

**需要 action 的能力**:
- 所有不在上述列表中的 instance
- 如果暂时无法提供，使用 TODO 模板：
  ```javascript
  action: {
    // TODO: 请提供 BLE 指令配置
    // write: [
    //   {
    //     bleDefine: ['33', '05', '...'],
    //     valIndex: [2, 3, ...]
    //   }
    // ]
  }
  ```

**新增文档**:
- `references/fixed-ble-instructions.md` - 固定 BLE 指令能力完整列表

---

### 4. 固定 BLE 指令能力规范 ✅

**新增功能**:
- 为每个设备类型创建专门的配置指南
- 从 7 种设备类型扩展到 10 种
- 每个指南包含详细的配置流程和注意事项

**新增设备类型配置指南**:
1. `light-guide.md` - 灯光设备配置指南
2. `humidifier-guide.md` - 加湿器配置指南
3. `fan-guide.md` - 风扇配置指南
4. `heater-guide.md` - 取暖器配置指南
5. `ice-maker-guide.md` - 制冰机配置指南（已有）
6. `thermometer-guide.md` - 温湿度计配置指南
7. `sensor-guide.md` - 传感器配置指南
8. `thermostat-guide.md` - 恒温器配置指南
9. `kettle-guide.md` - 水壶配置指南

**更新文档**:
- `device-types.md` - 更新为 10 种设备类型，添加配置指南链接

---

### 5. 设备类型配置指南系统 ✅

```
jsbridge-sku-config/
├── SKILL.md                              # 主 Skill 文档
├── CHANGELOG.md                          # 更新日志（本文件）
└── references/
    ├── H1232-template.md                 # H1232 完整示例
    ├── standard-sku-template.md          # 标准 SKU 模板
    ├── device-types.md                   # 设备类型总览（10种）
    ├── configuration-workflow.md         # 配置流程
    ├── ble-instruction-guide.md          # BLE 指令配置指南
    ├── fixed-ble-instructions.md         # 固定 BLE 指令列表（新增）
    ├── external-data-fields.md           # 外部填充数据说明（新增）
    │
    ├── light-guide.md                    # 灯光配置指南（新增）
    ├── humidifier-guide.md               # 加湿器配置指南（新增）
    ├── fan-guide.md                      # 风扇配置指南（新增）
    ├── heater-guide.md                   # 取暖器配置指南（新增）
    ├── ice-maker-guide.md                # 制冰机配置指南
    ├── thermometer-guide.md              # 温湿度计配置指南（新增）
    ├── sensor-guide.md                   # 传感器配置指南（新增）
    ├── thermostat-guide.md               # 恒温器配置指南（新增）
    └── kettle-guide.md                   # 水壶配置指南（新增）
```

---

## 配置检查清单

使用 Skill 配置 SKU 时，请检查：

### 基础信息
- [ ] goodsType 数据类型是否正确（数字或空字符串 ''）
- [ ] 产品类别是否正确
- [ ] 产品名称是否正确

### 外部填充数据
- [ ] lightScene 的 options 是否为空数组 `[]`
- [ ] diyScene 的 options 是否为空数组 `[]`
- [ ] snapshot 的 options 是否为空数组 `[]`
- [ ] musicMode 的 musicMode 字段 options 是否为空数组 `[]`
- [ ] 没有编造上述字段的数据

### action 配置
- [ ] musicMode 是否包含完整的 action 配置
- [ ] 所有非固定指令的 instance 是否配置了 action
- [ ] 如果暂时无法提供，是否添加了 TODO 注释
- [ ] 在任务清单中是否标记了需要配置的 instance

### 固定指令能力
- [ ] powerSwitch、brightness、colorRgb、colorTemperatureK 是否只配置了 cmdType
- [ ] lightScene、diyScene、snapshot 是否没有配置 action

---

## 使用建议

1. **配置前**: 阅读对应设备类型的配置指南
2. **配置中**: 按照 SKILL.md 的流程逐步确认
3. **配置后**: 使用检查清单验证配置是否正确
4. **测试**: 验证生成的代码是否符合预期

---

## 参考文档优先级

1. **设备类型配置指南** - 针对特定设备类型的详细指导
2. **fixed-ble-instructions.md** - 判断是否需要配置 action
3. **external-data-fields.md** - 了解哪些字段由外部填充
4. **ble-instruction-guide.md** - BLE 指令配置详细说明
5. **H1232-template.md** - 完整实现示例参考

---

## 常见问题

### Q: goodsType 应该使用什么数据类型？
A: 如果用户提供了 goodsType，使用数字类型（如：69, 321）；如果用户未提供，使用空字符串 ''。

### Q: 如何判断一个 instance 是否需要配置 action？
A: 查看 `fixed-ble-instructions.md`，如果不在固定指令列表中，就需要配置 action。

### Q: 用户暂时无法提供 BLE 指令怎么办？
A: 配置空 action + TODO 注释，在任务清单中标记，提醒用户后续补充。

### Q: lightScene 的 options 应该填什么？
A: 必须为空数组 `[]`，由外部系统填充，不要编造数据。

### Q: musicMode 的 options 为空，还需要配置 action 吗？
A: 是的，musicMode 必须包含完整的 action 配置，即使 musicMode 字段的 options 为空。

### Q: 如何选择设备类型？
A: 参考 `device-types.md`，根据设备主要功能选择，默认灯光设备使用 `devices.types.light`。

---

## 版本历史

- **v2.1.0** (2024-03-10): 添加 goodsType 数据类型规范
- **v2.0.0** (2024-03-10): 添加外部填充数据规范、固定 BLE 指令规范、设备类型配置指南系统
- **v1.1.0** (2024-03-10): 添加制冰机配置指南、Instance 优先级规则
- **v1.0.0** (2024-03-10): 初始版本，基础 SKU 配置功能
