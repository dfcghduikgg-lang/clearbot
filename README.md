# Kobe 群组管理机器人配置文档

本文档详细说明 Kobe 群组管理机器人的所有可配置项及其作用。

## 目录

- [配置方式](#配置方式)
- [显示模式配置](#显示模式配置)
- [榜单配置](#榜单配置)
- [图像识别配置](#图像识别配置)
- [配置示例](#配置示例)

---

## 配置方式

使用 `/kobe_config` 命令配置群组参数：

```
/kobe_config 键名 值
```

### 配置说明

- **权限要求**：仅管理员可以使用配置命令
- **嵌套配置**：使用点（`.`）分隔路径来配置嵌套项，例如：`leaderboards.activity.enabled`
- **值类型**：
  - 布尔值：`true`/`false`、`1`/`0`、`yes`/`no`、`on`/`off`
  - JSON：支持数组和对象，例如：`[{"name":"链接榜","regex":"https?://"}]`
  - 字符串和数字：直接输入

---

## 显示模式配置

### stats_display_mode

**用途**：控制统计命令中用户名的显示方式

**配置命令**：
```
/kobe_config stats_display_mode <mode>
```

**可选值**：
- `mention`（默认）：显示为 @用户名 或高亮显示（可点击）
- `name_id`：显示为 `名字 (ID: 123456)`
- `name`：只显示用户名字

**使用场景**：
- 使用 `/kobe_id` 命令时
- 其他统计类命令的用户信息展示

---

### inactive_display_mode

**用途**：控制 `/kobe_inactive` 命令中不活跃用户名的显示方式

**配置命令**：
```
/kobe_config inactive_display_mode <mode>
```

**可选值**：
- `mention`（默认）：显示为 @用户名 或高亮显示
- `name_id`：显示为 `名字 (ID: 123456)`
- `name`：只显示用户名字

**使用场景**：
- 使用 `/kobe_inactive` 查看未发言用户时

---

### leaderboard_display_mode

**用途**：控制榜单命令中用户名的显示方式

**配置命令**：
```
/kobe_config leaderboard_display_mode <mode>
```

**可选值**：
- `mention`（默认）：显示为 @用户名 或高亮显示
- `name_id`：显示为 `名字 (ID: 123456)`
- `name`：只显示用户名字

**使用场景**：
- 所有榜单的用户名显示
- 如果未配置，会回退到使用 `stats_display_mode` 的值

---

## 榜单配置

榜单功能提供多种维度的用户活跃度统计。所有榜单配置都在 `leaderboards` 命名空间下。

### 1. 发言榜 (Activity Leaderboard)

**用途**：统计指定天数内用户的发言次数

**配置命令**：
```
/kobe_config leaderboards.activity.enabled true
```

**特性**：
- 图标：💬
- 统计维度：发言次数
- 支持时间范围：1天、7天、30天
- 只显示发言数 > 0 的用户

**显示信息**：
- 用户排名
- 发言次数
- 最后发言时间

---

### 2. 值班榜 (Night Shift Leaderboard)

**用途**：统计深夜值班用户（凌晨 1:00-5:30 在线且持续至少2小时）

**配置命令**：
```
/kobe_config leaderboards.night_shift.enabled true
```

**特性**：
- 图标：🌙
- 时间范围：固定为最近一次完成的值班时段
  - 当前时间 < 5:30：显示昨天的 1:00-5:30
  - 当前时间 ≥ 5:30：显示今天的 1:00-5:30
- 筛选条件：消息时间跨度 ≥ 2小时
- 排序方式：按最后发言时间降序

**显示信息**：
- 用户排名
- 最后发言时间（HH:MM）
- 值班期间消息数

**时区**：北京时间（UTC+8）

---

### 3. DONE榜 (Done Leaderboard)

**用途**：统计发送"完成"图片（带绿色勾选标记）的用户

**配置命令**：
```
/kobe_config leaderboards.done.enabled true
```

**特性**：
- 图标：💯
- 统计维度：DONE图片数量
- 依赖：图像识别服务（自动检测DONE标记）
- 支持时间范围：1天、7天、30天

**显示信息**：
- 用户排名
- DONE次数
- 最后打卡时间

**技术说明**：
- 图片必须包含类型为 `is_done_image` 的识别标记
- 识别结果存储在消息的 `extra_data` 字段中

---

### 4. 关键字榜 (Keyword Leaderboard)

**用途**：统计消息匹配特定正则表达式的用户

**配置命令**：
```
# 启用关键字榜
/kobe_config leaderboards.keyword.enabled true

# 配置关键字模式（JSON数组）
/kobe_config leaderboards.keyword.patterns [{"name":"链接榜","regex":"https?://"}]
```

**特性**：
- 图标：🔑
- 支持多个关键字榜（每个模式一个榜单）
- 使用 PostgreSQL 正则表达式语法
- 支持时间范围：1天、7天、30天

**配置格式**：
```json
[
  {
    "name": "链接榜",
    "regex": "https?://"
  },
  {
    "name": "问候榜",
    "regex": "(早上好|晚安|你好)"
  }
]
```

**显示信息**：
- 用户排名
- 匹配次数
- 最后匹配时间

**注意事项**：
- 每个模式会创建一个独立的榜单
- 模式必须包含 `name`（显示名称）和 `regex`（正则表达式）
- 正则表达式区分大小写

---

### 5. 活跃榜 (Time Activity Leaderboard)

**用途**：统计用户在不同30分钟时间段内的发言覆盖率

**配置命令**：
```
/kobe_config leaderboards.time_activity.enabled true
```

**特性**：
- 图标：⏰
- 统计维度：不同30分钟时间段数
- 时间段定义：每天48个时间段（24小时 × 2）
- 支持时间范围：1天、7天、30天

**评分机制**：
- 分数 = 用户在不同时间段内发言的段数
- 分数越高表示全天活跃度越高
- 例如：7天内在100个不同时间段发言，得分100

**显示信息**：
- 用户排名
- 活跃时间段数
- 总消息数
- 最后发言时间

---

### 6. NSFW榜 (NSFW Leaderboard)

**用途**：统计发送 NSFW（色情/性感）图片的用户

**配置命令**：
```
/kobe_config leaderboards.nsfw.enabled true
```

**特性**：
- 图标：🔞
- 统计维度：NSFW图片数量（按类型分组）
- 依赖：NSFW图像识别服务
- 支持时间范围：1天、7天、30天

**NSFW类型**：
- 🍌 porn（色情）
- ❤️‍🔥 hentai（色情动漫）
- 💋 sexy（性感）

**显示信息**：
- 用户排名
- 总计NSFW图片数
- 各类型图片数（只显示非零类型）
- 最后发送时间

**技术说明**：
- 识别结果存储在消息的 `extra_data->>'nsfw_type'` 字段中
- 只统计 `message_type = 'photo'` 的消息

---

## 图像识别配置

### image_detection.min_confidence

**用途**：设置图像识别的最小置信度阈值

**配置命令**：
```
/kobe_config image_detection.min_confidence <value>
```

**取值范围**：
- 最小值：`0.1`
- 最大值：`0.99`
- 默认值：`0.1`

**工作原理**：
- 只有识别置信度 ≥ 阈值的图片才会被标记
- 阈值越高，误报越少，但可能漏检
- 阈值越低，检测更灵敏，但可能误报

**使用场景**：
- DONE图片检测
- NSFW图片检测
- 相似图片检测

**建议值**：
- 严格检测：`0.8` - `0.95`
- 平衡检测：`0.5` - `0.7`
- 宽松检测：`0.1` - `0.4`

---

## 配置示例

### 基础配置示例

```bash
# 配置显示模式为 名字+ID
/kobe_config stats_display_mode name_id

# 配置不活跃用户显示为纯名字
/kobe_config inactive_display_mode name
```

### 榜单配置示例

```bash
# 启用发言榜
/kobe_config leaderboards.activity.enabled true

# 启用值班榜
/kobe_config leaderboards.night_shift.enabled true

# 启用DONE榜
/kobe_config leaderboards.done.enabled true

# 启用活跃榜
/kobe_config leaderboards.time_activity.enabled true

# 启用NSFW榜
/kobe_config leaderboards.nsfw.enabled true

# 配置关键字榜 - 链接统计
/kobe_config leaderboards.keyword.enabled true
/kobe_config leaderboards.keyword.patterns [{"name":"链接榜","regex":"https?://"}]

# 配置关键字榜 - 多个模式
/kobe_config leaderboards.keyword.patterns [{"name":"链接榜","regex":"https?://"},{"name":"问候榜","regex":"(早上好|晚安|你好)"}]
```

### 图像识别配置示例

```bash
# 设置较高的置信度阈值（严格检测）
/kobe_config image_detection.min_confidence 0.8

# 设置较低的置信度阈值（宽松检测）
/kobe_config image_detection.min_confidence 0.3
```

### 完整配置流程示例

```bash
# 1. 配置显示模式
/kobe_config stats_display_mode mention
/kobe_config leaderboard_display_mode mention

# 2. 启用所有榜单
/kobe_config leaderboards.activity.enabled true
/kobe_config leaderboards.night_shift.enabled true
/kobe_config leaderboards.done.enabled true
/kobe_config leaderboards.time_activity.enabled true
/kobe_config leaderboards.nsfw.enabled false

# 3. 配置关键字榜
/kobe_config leaderboards.keyword.enabled true
/kobe_config leaderboards.keyword.patterns [{"name":"链接榜","regex":"https?://"},{"name":"表情榜","regex":"😀|😁|😂|🤣|😃"}]

# 4. 配置图像识别
/kobe_config image_detection.min_confidence 0.6
```

---

## 数据结构说明

### config 字段结构

群组的 `config` 字段是一个 JSON 对象，结构如下：

```json
{
  "stats_display_mode": "mention",
  "inactive_display_mode": "mention",
  "leaderboard_display_mode": "mention",
  "image_detection": {
    "min_confidence": 0.1
  },
  "leaderboards": {
    "activity": {
      "enabled": true
    },
    "night_shift": {
      "enabled": true
    },
    "done": {
      "enabled": true
    },
    "time_activity": {
      "enabled": true
    },
    "nsfw": {
      "enabled": false
    },
    "keyword": {
      "enabled": true,
      "patterns": [
        {
          "name": "链接榜",
          "regex": "https?://"
        }
      ]
    }
  }
}
```

---

## 注意事项

1. **权限要求**：所有配置命令都需要管理员权限
2. **配置持久化**：配置保存在数据库中，重启后仍然有效
3. **配置生效**：大部分配置立即生效，无需重启机器人
4. **缓存刷新**：配置修改后会自动清除相关缓存
5. **JSON格式**：配置 JSON 值时注意正确的格式，避免语法错误
6. **正则表达式**：使用 PostgreSQL 正则表达式语法，不是 Python 或 JavaScript 语法
7. **时区处理**：所有时间显示使用北京时间（UTC+8）

---

## 常见问题

### Q: 如何查看当前的配置？

A: 目前没有直接命令查看配置，建议通过数据库查询或查看命令效果来确认。

### Q: 榜单启用后没有数据怎么办？

A: 可能原因：
- 时间范围内没有符合条件的数据
- 图像识别服务未启动（DONE榜、NSFW榜）
- 正则表达式配置错误（关键字榜）

### Q: 如何禁用某个榜单？

A: 将对应的 `enabled` 设置为 `false`：
```bash
/kobe_config leaderboards.nsfw.enabled false
```

### Q: 关键字榜的正则表达式如何编写？

A: 使用 PostgreSQL 的 POSIX 正则表达式语法：
- 基础匹配：`hello` 匹配包含 "hello" 的消息
- 或运算：`hello|hi` 匹配 "hello" 或 "hi"
- 开头匹配：`^hello` 匹配以 "hello" 开头的消息
- 结尾匹配：`hello$` 匹配以 "hello" 结尾的消息
- 字符类：`[0-9]+` 匹配一个或多个数字

### Q: 图像识别的置信度阈值如何选择？

A: 建议策略：
- 初始使用默认值 `0.1` 观察效果
- 如果误报过多，逐步提高到 `0.5`、`0.7`
- 如果漏检过多，降低到 `0.3`
- 通常 `0.5-0.7` 是较好的平衡点

---

## 更新日志

- 2025-01-13：初始版本，包含所有基础配置项和榜单配置
