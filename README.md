# 🎱 HarmonyOS 桌球游戏

一个基于 HarmonyOS 6.0.2 (API Level 22) 开发的桌球游戏，支持单人训练、双人对战和人机对战模式。

## 📱 运行环境

- **操作系统**: HarmonyOS 6.0.2 (API Level 22)
- **开发工具**: DevEco Studio
- **模拟器**: Enjoy 90 Pro Max 或其他 HarmonyOS 设备

## ✨ 功能特性

### 游戏模式
- **单人训练模式**: 自由练习，无规则限制
- **双人对战**: 两人轮流击球，遵循标准桌球规则
- **人机对战**: 与 AI 对战，AI 会自动瞄准击球

### 游戏规则
- **单色球** (1-7号): 实心彩色球
- **双色球** (9-15号): 带白色条纹的彩色球
- **黑球** (8号): 最后击入获胜
- **首次进球决定球型**: 第一个进球决定玩家的球型（单色/双色）
- **犯规检测**: 母球落袋、未击中球等犯规行为
- **黑球规则**: 提前击入黑球判负

### 物理引擎
- 真实的碰撞检测和动量传递
- 球与球之间的碰撞
- 球与边框的反弹
- 摩擦力模拟
- 球袋检测

### 操作方式
- **拖动调整**: 在球桌上拖动同时调整角度和力度
- **角度**: 拖动方向决定击球角度
- **力度**: 拖动距离决定击球力度（距离越远力度越大）
- **自动发射**: 松开手指自动发射

## 🎮 游戏界面

### 球桌
- 标准绿色台面
- 棕色木质边框
- 6个黑色球袋（四角和两边中间）
- 球杆可视化显示

### 控制面板
- 游戏模式显示
- 玩家信息和比分
- 力度条可视化
- 操作说明
- 重新开始/返回菜单按钮

## 🛠️ 技术实现

### 核心技术
- **ArkTS**: HarmonyOS 的 TypeScript 扩展语言
- **声明式UI**: 使用 @Component 和 @State 装饰器
- **物理引擎**: 自定义 2D 物理模拟
- **触摸事件**: TouchEvent 处理用户交互

### 关键实现

#### 状态管理
```typescript
@State balls: Ball[] = [];           // 球数组
@State cueBallX: number = 160;       // 母球X坐标
@State cueBallY: number = 190;       // 母球Y坐标
@State aimAngle: number = 0;         // 瞄准角度
@State aimPower: number = 50;        // 发射力度
```

#### 碰撞检测
```typescript
// 母球与其他球碰撞
if (distance < this.ballRadius * 2 && distance > 0.1) {
  const impulse = dvn;
  this.cueBallVX -= impulse * nx;
  this.cueBallVY -= impulse * ny;
  ball.vx += impulse * nx;
  ball.vy += impulse * ny;
}
```

#### UI 更新
```typescript
// 强制触发状态更新
this.balls = [...this.balls];
```

#### ForEach 渲染
```typescript
ForEach(this.balls, (ball: Ball, index: number) => {
  // 渲染球
}, (ball: Ball, index: number) => ball.id.toString() + '_' + Math.round(ball.x) + '_' + Math.round(ball.y))
```

## 📁 项目结构

```
MyApplication/
├── entry/
│   └── src/
│       └── main/
│           ├── ets/
│           │   ├── entryability/
│           │   │   └── EntryAbility.ets    # 应用入口
│           │   └── pages/
│           │       └── PoolGame.ets        # 游戏主页面
│           ├── resources/
│           │   └── base/
│           │       └── profile/
│           │           └── main_pages.json # 页面配置
│           └── module.json5                # 模块配置
├── build-profile.json5                     # 构建配置
└── README.md                               # 说明文档
```

## 🚀 运行步骤

1. **打开项目**
   - 使用 DevEco Studio 打开 `MyApplication` 目录

2. **配置模拟器**
   - 确保已创建 HarmonyOS 6.0.2 (API Level 22) 模拟器
   - 推荐使用 Enjoy 90 Pro Max 模拟器

3. **运行项目**
   - 点击 DevEco Studio 的运行按钮
   - 选择目标模拟器
   - 等待编译和部署完成

4. **开始游戏**
   - 选择游戏模式（训练/双人对战/人机对战）
   - 在球桌上拖动调整角度和力度
   - 松开手指发射母球

## ⚠️ 常见问题

### 白屏问题
**原因**: 页面路径配置错误
**解决**: 检查 `EntryAbility.ets` 中的页面加载路径

### 球不动问题
**原因**: 状态更新未触发
**解决**: 使用 `this.balls = [...this.balls]` 强制更新

### 编译错误
**原因**: ArkTS 类型检查严格
**解决**: 为所有对象和数组添加明确的类型声明

## 📝 开发笔记

### HarmonyOS 开发要点

1. **状态管理**
   - 直接修改数组元素属性不会触发 UI 更新
   - 需要重新赋值数组来触发更新

2. **ForEach 渲染**
   - key 函数必须包含所有需要追踪的属性
   - 建议包含位置信息以实现动画效果

3. **类型安全**
   - ArkTS 要求所有类型必须明确
   - 使用接口定义数据结构

4. **触摸事件**
   - 使用 TouchType.Down/Move/Up 处理交互
   - 注意坐标系统的转换

## 🎯 未来优化方向

- [ ] 添加击球轨迹预测线
- [ ] 优化 AI 算法
- [ ] 添加音效和背景音乐
- [ ] 支持自定义球桌主题
- [ ] 添加回放功能
- [ ] 支持在线对战

## 📄 许可证

本项目仅供学习和研究使用。

## 👨‍💻 作者

开发于 2026年6月

---

**享受游戏！🎱**