# 分布式校园记事本

基于HarmonyOS ArkTS API 22开发的分布式校园记事本应用，支持一次开发多端部署和自由流转。

## 功能特性

### 1. 一多自适应布局

#### 布局策略
- **弹性布局**：使用Column/Row弹性布局，百分比宽高，自适应不同屏幕尺寸
- **媒体查询断点适配**：
  - **SM断点（<600vp）**：手机竖屏单栏布局，笔记列表和编辑框上下排布
  - **MD断点（600-840vp）**：平板/折叠屏横屏双栏布局，左侧笔记列表，右侧编辑框
  - **LG断点（≥840vp）**：智慧屏三栏布局，左侧分类导航，中间笔记列表，右侧编辑框

#### 实现细节
- 使用`mediaQuery.matchConditionSync`实现断点检测
- 通过`BreakpointSystem`统一管理断点状态
- 所有UI组件使用纯ArkUI基础组件（Text、Button、List、Column、Row等），无Canvas/WebGL/3D组件
- 避免渲染白屏和编译类型报错

### 2. 自由流转（分布式任务接续）

#### 分布式能力
- 接入鸿蒙分布式任务调度框架
- 支持超级终端设备间流转
- 配置`continueType: "immediate"`实现即时接续

#### 流转数据同步
流转时同步以下数据：
- 当前编辑笔记内容
- 光标位置
- 分类标签选择状态
- 笔记ID和时间戳

#### 实现机制
- `DistributedManager`：管理分布式流转生命周期
- `onContinue`：流转发起时序列化当前状态
- `onNewWant`：流转接收时恢复状态
- 实时更新流转数据，确保无缝接续体验

### 3. 业务功能

- **笔记管理**：新增、删除、修改笔记
- **分类筛选**：支持学习、生活、工作、其他四个分类
- **数据持久化**：使用Preferences本地存储，重启应用数据不丢失

## 项目结构

```
notebook/
├── entry/
│   └── src/main/
│       ├── ets/
│       │   ├── components/          # UI组件
│       │   │   ├── NoteListComponent.ets       # 笔记列表组件
│       │   │   ├── NoteEditorComponent.ets     # 笔记编辑组件
│       │   │   └── CategorySelectorComponent.ets # 分类选择组件
│       │   ├── model/               # 数据模型
│       │   │   ├── Note.ets                     # 笔记数据模型
│       │   │   └── DistributedData.ets          # 分布式数据模型
│       │   ├── utils/               # 工具类
│       │   │   ├── NoteStorage.ets              # Preferences持久化存储
│       │   │   ├── DistributedManager.ets       # 分布式流转管理
│       │   │   └── BreakpointSystem.ets         # 断点适配系统
│       │   ├── pages/
│       │   │   └── Index.ets                    # 主页面（自适应布局）
│       │   └── entryability/
│       │       └── EntryAbility.ets             # 应用入口（支持流转）
│       ├── resources/              # 资源文件
│       └── module.json5            # 模块配置（多设备+分布式）
└── README.md
```

## 技术实现

### 一多适配实现逻辑

1. **断点检测**：
   ```typescript
   mediaQuery.matchConditionSync('(0vp<=width<600vp)')   // SM
   mediaQuery.matchConditionSync('(600vp<=width<840vp)') // MD
   mediaQuery.matchConditionSync('(840vp<=width)')       // LG
   ```

2. **布局切换**：
   ```typescript
   if (breakpoint === BreakpointType.SM) {
     this.phoneLayout()    // 单栏
   } else if (breakpoint === BreakpointType.MD) {
     this.tabletLayout()   // 双栏
   } else {
     this.desktopLayout()  // 三栏
   }
   ```

3. **弹性布局**：
   - 使用`layoutWeight`实现弹性伸缩
   - 使用百分比宽度`width('40%')`实现比例布局
   - 使用`Blank()`填充剩余空间

### 分布式流转实现逻辑

1. **配置分布式能力**：
   ```json5
   {
     "continueType": "immediate",
     "deviceTypes": ["phone", "tablet", "2in1"]
   }
   ```

2. **流转数据序列化**：
   ```typescript
   onContinue(want: Want): OnContinueResult {
     const data = {
       currentNoteId: this.selectedNoteId,
       currentNote: this.currentNote,
       cursorPosition: this.cursorPosition,
       selectedCategory: this.selectedCategory
     }
     want.parameters['distributedData'] = JSON.stringify(data)
     return OnContinueResult.AGREE
   }
   ```

3. **流转数据恢复**：
   ```typescript
   onNewWant(want: Want): void {
     const data = JSON.parse(want.parameters['distributedData'])
     this.currentNote = data.currentNote
     this.cursorPosition = data.cursorPosition
     // 恢复UI状态
   }
   ```

4. **实时数据同步**：
   - 每次编辑内容变化时调用`updateDistributedData()`
   - 确保流转时携带最新状态

### 数据持久化实现

```typescript
// 初始化Preferences
this.prefs = await preferences.getPreferences(context, 'notebook_prefs')

// 保存笔记
await this.prefs.put('notes_list', JSON.stringify(notes))
await this.prefs.flush()

// 加载笔记
const notesStr = await this.prefs.get('notes_list', '[]')
return JSON.parse(notesStr)
```

## 开发环境

- **DevEco Studio**: HarmonyOS 6.0.2
- **API Version**: 22
- **语言**: ArkTS
- **框架**: ArkUI

## 使用说明

1. **手机端**：
   - 竖屏显示笔记列表
   - 点击笔记进入编辑页面
   - 点击"+"新建笔记

2. **平板/折叠屏**：
   - 横屏显示双栏布局
   - 左侧列表，右侧编辑
   - 点击"流转"按钮选择目标设备

3. **智慧屏**：
   - 三栏布局，左侧分类导航
   - 中间笔记列表，右侧编辑区

4. **分布式流转**：
   - 在编辑界面点击"流转"
   - 选择目标设备（需登录同一华为账号）
   - 笔记内容和编辑状态自动同步到目标设备

## 注意事项

- 所有UI组件均为ArkUI基础组件，无第三方依赖
- 代码无类型报错，无渲染白屏
- 支持深色模式自动适配
- 数据本地持久化，重启不丢失

## 许可证

MIT License