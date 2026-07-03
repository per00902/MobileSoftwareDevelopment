# 睡眠记录 - Sleep Tracker

GitHub 仓库地址：https://github.com/per00902/SleepTracker

## 1. 项目简介

- 应用名称：睡眠记录（Sleep Tracker）
- 目标用户：关注睡眠健康、希望改善睡眠质量的普通智能手机用户
- 核心功能：记录每日入睡/起床时间并自动计算时长；1-5星质量评级；自定义标签分类（带Emoji图标）；本周睡眠统计（记录天数、平均时长、平均质量、目标对比）；搜索与标签筛选联动；从网络获取睡眠建议；设置目标睡眠时长；深色/浅色模式跟随系统

## 2. 技术栈

- UI：Jetpack Compose + Material 3
- 数据库：Room（KSP编译处理器）
- 网络：Retrofit + OkHttp + Gson（接口来源：JSONPlaceholder Mock API，https://jsonplaceholder.typicode.com/）
- 状态管理：ViewModel + StateFlow
- 持久化偏好：DataStore Preferences
- 导航：Navigation Compose（导航版本 2.8.4）
- 异步处理：Kotlin Coroutines
- 其他依赖：Coil（网络图片加载）、Material Icons Extended

## 3. 功能清单

### 必做项完成情况

**UI 层**
- [x] Jetpack Compose 构建全部 UI
- [x] 至少 2 个主要页面（共 4 个页面：首页、编辑页、睡眠建议页、设置页）
- [x] Compose Navigation 导航
- [x] LazyColumn / LazyVerticalGrid 列表（首页使用 LazyColumn 展示睡眠记录，LazyRow 展示标签筛选）
- [x] Material 3 组件和主题（自定义夜间灵感配色，支持动态取色）
- [x] 浅色 / 深色模式支持（跟随系统自动切换）

**数据层**
- [x] Room 数据库，至少 2 张表（sleep_records 和 sleep_tags）
- [x] 完整 CRUD 操作（新增、查询、编辑、删除）
- [x] DAO 查询方法返回 Flow 类型（getAllRecords、searchRecords 等均返回 Flow）
- [x] 至少一种查询功能（多种：全量查询、按标签筛选、日期范围查询、模糊搜索、聚合统计查询）
- [x] DataStore 保存用户偏好或最近状态（保存目标睡眠时长、通知开关、上次搜索关键词等）

**网络层**
- [x] 声明并使用 Internet 权限（AndroidManifest.xml 中声明）
- [x] 使用网络请求获取真实 API 或 Mock API 数据（JSONPlaceholder /posts 接口）
- [x] 网络数据在核心页面中展示或参与主要功能流程（"睡眠建议"页面展示）
- [x] 处理 Loading / Success / Error 等网络状态（SleepTipsUiState 的 sealed interface）
- [x] Composable 不直接发起网络请求（通过 ViewModel → Repository → NetworkDataSource 调用）

**架构层**
- [x] ViewModel 状态管理（4 个 ViewModel 管理各页面状态）
- [x] Repository 模式（SleepRepository 封装所有数据访问）
- [x] StateFlow / Flow 数据流（ViewModel 暴露 StateFlow，Composable 通过 collectAsState 观察）
- [x] Kotlin 协程异步处理（viewModelScope 管理协程，Room 和网络请求均在协程中执行）
- [x] UiState 描述界面状态（SleepListUiState、EditRecordUiState、SleepTipsUiState、SettingsUiState）
- [x] Composable 不直接访问数据库或网络

**功能完整性**
- [x] 新增 / 编辑 / 删除 / 搜索等核心操作（全部实现）
- [x] 输入验证和错误提示（编辑页验证入睡/起床时间合法性、质量范围）
- [x] 状态展示（空 / 加载 / 错误中的至少一种）（三种全部实现）
- [x] 屏幕旋转后状态保持（ViewModel 不受屏幕旋转影响，状态自动保持）

### 选做项完成情况

- [x] 复杂数据库查询：SQL 聚合查询（AVG 平均时长、AVG 平均质量、COUNT 记录数）、日期范围查询、LIKE 模糊搜索
- [x] 搜索防抖或搜索历史：使用 StateFlow + combine + flatMapLatest 实现搜索与标签筛选的联动防抖
- [x] Coil 图片加载：睡眠建议页面使用 AsyncImage 加载 Unsplash 网络配图
- [x] 标签管理：支持用户自定义标签名称和 emoji 图标，增删操作实时更新
- [ ] 数据导出：未实现
- [ ] 图表统计：未实现
- [ ] 就寝提醒：通知开关已保留但未实现定时提醒逻辑

## 4. 数据库设计

### 表 1：sleep_records（睡眠记录表）

| 字段名 | 类型 | 说明 |
|--------|------|------|
| id | Long (INTEGER) | 主键，自增 |
| sleep_time | Long (INTEGER) | 入睡时间戳（毫秒） |
| wake_time | Long (INTEGER) | 起床时间戳（毫秒） |
| quality | Int (INTEGER) | 睡眠质量评级，范围 1-5 |
| note | String (TEXT) | 备注/日记 |
| created_at | Long (INTEGER) | 记录创建时间戳 |
| tag_id | Long? (INTEGER) | 关联标签ID（外键，可空） |

### 表 2：sleep_tags（睡眠标签表）

| 字段名 | 类型 | 说明 |
|--------|------|------|
| id | Long (INTEGER) | 主键，自增 |
| name | String (TEXT) | 标签名称（如"午睡"、"失眠"） |
| emoji | String (TEXT) | 标签图标 emoji（如 🌙、💤） |
| created_at | Long (INTEGER) | 标签创建时间戳 |

**表关系**：sleep_records.tag_id 外键关联 sleep_tags.id，一个标签可对应多条记录（一对多关系）。

**主要 DAO 查询方法**：
- `getAllRecords()` → `Flow<List<SleepRecordEntity>>`：按入睡时间倒序获取全部
- `getRecordById(id)` → `SleepRecordEntity?`：按 ID 查询单条
- `getRecordsByDateRange(start, end)` → `Flow<List<SleepRecordEntity>>`：日期范围查询
- `getRecordsByTagId(tagId)` → `Flow<List<SleepRecordEntity>>`：按标签筛选
- `searchRecords(keyword)` → `Flow<List<SleepRecordEntity>>`：备注 LIKE 模糊搜索
- `getAverageSleepDuration(sinceTime)` → `Double?`：SQL AVG 聚合计算平均睡眠时长
- `getAverageSleepQuality(sinceTime)` → `Double?`：SQL AVG 聚合计算平均质量
- `getRecordCount(sinceTime)` → `Int`：SQL COUNT 统计记录数量

## 5. 网络功能设计

- API 来源：JSONPlaceholder（免费 Mock API）
- 接口地址：`https://jsonplaceholder.typicode.com/posts`
- 请求方式：GET
- 主要返回字段：id（编号）、title（标题）、body（正文）、userId（用户ID）
- App 中使用这些网络数据的页面或功能：在"睡眠建议"页面（Sleep Tips Screen）展示为卡片列表，每张卡片包含配图、分类标签、标题和描述。数据在 Repository 层通过 `mapToSleepTip()` 将 Posts DTO 映射为 `SleepTip` 业务模型（保留标题和内容，按 id 取模赋予 5 类睡眠分类，并配 Unsplash 睡眠主题配图 URL）
- 网络失败时的处理方式：显示 WifiOff 图标 + 错误信息文本 + "重试"按钮，用户点击重试按钮可重新发起请求

## 6. 架构设计

```
UI Layer (Composable 函数)
    ↑ collectAsState() 观察 StateFlow
ViewModel Layer（4 个 ViewModel）
    ↑ 调用 Repository 方法，通过 StateFlow 发布 UiState
Repository Layer（SleepRepository）
    ↗           ↖
Room DAO     NetworkDataSource
(SleepRecordDao, SleepTagDao)    (Retrofit API)
    ↑                ↑
Room DB (SQLite)    JSONPlaceholder API
```

各层职责：
- **Data Layer**：Room 数据库负责本地数据持久化；NetworkDataSource 封装 Retrofit 实例和网络请求
- **Repository Layer**：SleepRepository 作为唯一数据入口，隔离本地 DAO 和网络调用，对外提供统一的数据访问接口
- **ViewModel Layer**：每个页面拥有对应的 ViewModel，管理业务逻辑和页面状态，通过 `StateFlow` 向 UI 层暴露 `UiState`
- **UI Layer**：Composable 函数通过 `viewModel(factory=...)` 获取 ViewModel，通过 `collectAsState()` 收集状态，只负责渲染和事件触发

## 7. 核心功能截图

> 请运行应用后截取实际图片替换下方占位。

### 首页
![alt text](image.png)
说明：展示睡眠记录列表和本周统计卡片（记录天数、平均时长、平均质量、目标对比），顶部有搜索按钮、睡眠建议按钮和设置按钮，底部 FAB 添加按钮。

### 编辑记录页
![alt text](image-1.png)
说明：新增/编辑睡眠记录表单，包含入睡时间选择（DatePicker + TimePicker）、起床时间选择、预计时长预览、睡眠质量滑动条（1-5星，实时显示星星数量）、标签选择、备注输入和保存按钮。

### 睡眠建议页
![alt text](image-2.png)
说明：从网络获取睡眠建议，每张卡片包含 Unsplash 配图、分类标签（睡眠环境/作息规律/饮食建议等）、标题和描述。顶部有刷新按钮，加载中显示进度条，加载失败显示错误信息和重试按钮。

### 设置页
![alt text](image-3.png)
说明：包含目标睡眠时长设置（小时+分钟滑动条）、就寝提醒开关、自定义标签管理（添加标签弹窗、删除标签）。

## 8. 技术难点与解决方案

### 难点 1：搜索与标签筛选的联动

- 问题描述：需要同时支持搜索关键词和标签筛选，且两者需要联动（有搜索词时按搜索，无搜索词但选了标签时按标签筛选，都为空时显示全部），同时避免重复查询。
- 原因分析：搜索和筛选是两种不同的查询条件，如果各自独立触发查询，会导致竞态条件和数据不一致。
- 解决方案：使用两个 `MutableStateFlow`（`_searchQuery` 和 `_selectedTagId`）管理输入状态，通过 Flow 的 `combine` 操作符合并为一个数据流，再使用 `flatMapLatest` 根据条件自动切换查询方法：搜索词非空 → `searchRecords()`；选了标签 → `getRecordsByTagId()`；都为空 → `getAllRecords()`。`flatMapLatest` 确保新查询发出时自动取消上一个查询，避免重复。

### 难点 2：Mock API 数据到业务模型的映射

- 问题描述：JSONPlaceholder 的 `/posts` 接口返回通用帖子数据（id、title、body、userId），没有睡眠相关的字段名和配图 URL，不能直接用于展示睡眠建议。
- 原因分析：选择的免费 Mock API 数据结构与业务需求不匹配，需要额外的数据映射层。
- 解决方案：在 `NetworkDataSource` 中实现 `mapToSleepTip()` 映射函数：
  1. 将 `title` 截取前 30 字符作为睡眠建议标题
  2. 将 `body` 截取前 200 字符作为建议描述
  3. 按帖子 ID 使用 `id % 5` 分配到 5 个睡眠分类（"睡眠环境"、"作息规律"、"饮食建议"、"运动建议"、"放松技巧"）
  4. 使用 Unsplash 睡眠主题图片 URL 列表轮换作为配图
- 参考资料：JSONPlaceholder 官方文档 https://jsonplaceholder.typicode.com/

### 难点 3：Room 数据库版本升级

- 问题描述：开发过程中数据库表结构可能发生变化（如新增字段），Room 需要处理版本迁移。
- 解决方案：当前使用 `fallbackToDestructiveMigration()` 策略——版本升级时直接销毁旧表重建。此方案仅在开发阶段适用，生产环境应编写具体的 `Migration` 对象（如 `Migration(1, 2){...}`）以保留用户数据。

## 9. AI 使用说明

请在以下选项中勾选，可多选：

- [ ] 未使用 AI
- [ ] 网页版 AI（如 ChatGPT、Claude、Kimi、豆包等）
- [x] AI Agent / 编程代理（如 Claude Code、Codex、OpenCode、Cursor Agent 等）
- [ ] 国产大模型服务（如 DeepSeek、GLM、通义千问、文心一言等）
- [x] IDE 插件或代码补全工具（如 GitHub Copilot、Cursor、CodeGeeX 等）
- [ ] 其他：

具体工具名称：CodeBuddy（AI 编程助手）

AI 主要用于哪些环节：项目架构设计和数据库设计、Jetpack Compose UI 代码生成、Room Entity/DAO/Repository 代码编写、Retrofit 网络层封装与 DTO 映射、技术难点调试与问题排查、项目文档和报告整理

说明：是否使用 AI 以及使用了什么 AI 工具不会影响分值，请如实填写。

## 10. 运行说明

- 最低 Android 版本：API 24（Android 7.0 Nougat）
- 推荐 Android 版本：API 35（Android 15）
- 特殊权限：网络权限（`android.permission.INTERNET`）；如使用相机、通知等功能，也需要说明对应权限
- 运行步骤：
  1. 克隆仓库：`git clone https://github.com/你的用户名/仓库名`
  2. 使用 Android Studio 打开项目
  3. 等待 Gradle 同步完成（国内网络已配置阿里云镜像加速依赖下载）
  4. 连接模拟器（API 24+）或真机（开启 USB 调试），点击 Run

## 11. 项目亮点（可选）

- **完整的 MVVM + Repository 架构**：代码分层清晰，ViewModel 不直接依赖数据源，Composable 不直接访问数据库或网络，各层职责明确
- **响应式数据流**：Room DAO 返回 Flow，ViewModel 通过 StateFlow 发布状态，数据变更实时同步到 UI 界面
- **Flow 组合操作**：使用 `combine` + `flatMapLatest` 优雅实现搜索防抖和筛选联动，代码简洁高效
- **全面的网络状态处理**：通过 `sealed interface SleepTipsUiState` 明确定义 Loading / Success / Error / Empty 四种状态，UI 层根据状态渲染不同视图
- **Material 3 完整主题**：自定义夜间灵感配色方案，浅色/深色模式跟随系统自动切换，圆角形状统一（8dp / 12dp / 16dp）
- **标签 + Emoji 系统**：支持用户自定义标签名称和 emoji 图标，提供 10 个预置图标供选择，提升分类可视化体验
- **SQL 聚合统计**：使用 Room 原生 SQL 聚合函数（AVG、COUNT）计算统计数据，高效准确

## 12. 未来改进方向（可选）

- **图表统计**：使用 MPAndroidChart 或 Compose Canvas 绘制睡眠趋势折线图、质量分布饼图
- **数据导出**：支持导出为 CSV / JSON 文件，方便用户备份和分析
- **就寝提醒**：实现定时推送通知功能，提醒用户按时就寝
- **平板适配**：实现 Master-Detail 双栏布局，优化平板和横屏体验
- **Room Migration**：为后续数据库版本升级编写 Migration 对象，避免 `fallbackToDestructiveMigration` 丢失用户数据
- **Hilt 依赖注入**：引入 Hilt 替代手动 ViewModelFactory，简化依赖管理
- **单元测试**：为 ViewModel 和 Repository 添加 JUnit 和 MockK 单元测试覆盖
