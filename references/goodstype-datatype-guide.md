# goodsType 数据类型规范

本文档详细说明 SKU 配置中 `goodsType` 字段的数据类型规则。

---

## 数据类型规则

### 规则概述

`goodsType` 字段的数据类型取决于用户是否提供该值：

1. **用户提供时**：使用数字类型（Number）
2. **用户未提供时**：使用空字符串（Empty String）

---

## 正确示例

### 用户提供 goodsType

```javascript
// ✅ 正确：使用数字类型
const Endpoint = {
  productSku: 'H8131',
  productCategory: 'devices.types.ice_maker',
  productName: 'Ice Maker',
  description: '',
  goodsType: 321,  // 数字类型
  capabilities: [...]
};
```

```javascript
// ✅ 正确：使用数字类型
const Endpoint = {
  productSku: 'H608D',
  productCategory: 'devices.types.light',
  productName: 'RGBIC Light',
  description: '',
  goodsType: 184,  // 数字类型
  capabilities: [...]
};
```

### 用户未提供 goodsType

```javascript
// ✅ 正确：使用空字符串
const Endpoint = {
  productSku: 'H713E',
  productCategory: 'devices.types.heater',
  productName: 'Heater',
  description: '',
  goodsType: '',  // 空字符串
  capabilities: [...]
};
```

---

## 错误示例

### 不要使用字符串格式的数字

```javascript
// ❌ 错误：不要使用字符串格式的数字
const Endpoint = {
  productSku: 'H8131',
  productCategory: 'devices.types.ice_maker',
  productName: 'Ice Maker',
  description: '',
  goodsType: '321',  // ❌ 错误：字符串格式
  capabilities: [...]
};
```

```javascript
// ❌ 错误：不要使用字符串格式的数字
const Endpoint = {
  productSku: 'H608D',
  productCategory: 'devices.types.light',
  productName: 'RGBIC Light',
  description: '',
  goodsType: "184",  // ❌ 错误：字符串格式
  capabilities: [...]
};
```

---

## 配置流程

### 步骤 1: 询问用户

在配置 SKU 时，必须询问用户是否有 goodsType：

```
请问这个设备有 goodsType 吗？如果有，请提供数字值。
```

### 步骤 2: 根据用户回答配置

**情况 A：用户提供了 goodsType**
```javascript
// 用户回答：goodsType 是 321
goodsType: 321  // 使用数字类型
```

**情况 B：用户未提供 goodsType**
```javascript
// 用户回答：没有 goodsType 或不知道
goodsType: ''  // 使用空字符串
```

---

## 代码库示例

### H8131 (制冰机)

```javascript
const Endpoint = {
  productSku: 'H8131',
  productCategory: 'devices.types.ice_maker',
  productName: 'Ice Maker',
  description: '',
  goodsType: 321,  // ✅ 数字类型
  capabilities: [...]
};
```

### H608D (灯光)

```javascript
const Endpoint = {
  productSku: 'H608D',
  productCategory: 'devices.types.light',
  productName: 'RGBIC Light',
  description: '',
  goodsType: 184,  // ✅ 数字类型
  capabilities: [...]
};
```

### H713E (取暖器)

```javascript
const Endpoint = {
  productSku: 'H713E',
  productCategory: 'devices.types.heater',
  productName: 'Heater',
  description: '',
  goodsType: '',  // ✅ 空字符串（用户未提供）
  capabilities: [...]
};
```

---

## 任务清单中的表示

在任务清单中，应该明确显示 goodsType 的数据类型：

```markdown
### ✅ 已完成任务
- [x] 配置 Endpoint 基础信息
  - productSku: H8131
  - productCategory: devices.types.ice_maker
  - productName: Ice Maker
  - goodsType: 321（数字类型）
```

或者：

```markdown
### ✅ 已完成任务
- [x] 配置 Endpoint 基础信息
  - productSku: H713E
  - productCategory: devices.types.heater
  - productName: Heater
  - goodsType: ''（空字符串，用户未提供）
```

---

## 验证检查清单

配置完成后，检查：

- [ ] 如果用户提供了 goodsType，是否使用了数字类型？
- [ ] 如果用户未提供 goodsType，是否使用了空字符串 ''？
- [ ] 没有使用字符串格式的数字（如 '321'、"184"）？
- [ ] 任务清单中是否明确标注了数据类型？

---

## 常见错误

### 错误 1: 使用字符串格式的数字

```javascript
// ❌ 错误
goodsType: '69'
goodsType: "321"

// ✅ 正确
goodsType: 69
goodsType: 321
```

### 错误 2: 用户未提供时使用 null 或 undefined

```javascript
// ❌ 错误
goodsType: null
goodsType: undefined

// ✅ 正确
goodsType: ''
```

### 错误 3: 混淆数字和字符串

```javascript
// ❌ 错误：有时用数字，有时用字符串
const config1 = { goodsType: 69 };
const config2 = { goodsType: '321' };

// ✅ 正确：统一使用数字类型
const config1 = { goodsType: 69 };
const config2 = { goodsType: 321 };
```

---

## 总结

**记住这个简单规则**：

- 用户提供 → 数字类型（如：69, 321, 184）
- 用户未提供 → 空字符串（''）

**永远不要使用字符串格式的数字**（如：'69', "321"）
