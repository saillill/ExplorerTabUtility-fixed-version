# CLAUDE.md — Explorer Tab Utility 修复版

## 项目概述

基于 [w4po/ExplorerTabUtility](https://github.com/w4po/ExplorerTabUtility) 的修复和增强版。强制 Windows 11 的新文件资源管理器窗口以标签页方式打开。

- **技术栈**: .NET 9.0, WPF, C# 12
- **运行环境**: Windows 11 22H2+ (Build 22621+)
- **仓库**: `saillill/ExplorerTabUtility-fixed-version`

## 与原版的主要差异

| 方面 | 原版 | 这个版本 |
|------|------|------|
| 界面语言 | 仅英文 | 中英双语切换 |
| 主题 | 仅深色 | 深色/浅色/跟随系统 |
| COM 互操作 | MSBuild COMReference | 手动 C# 类型定义 |
| 自动更新 | 有 | 已移除 |
| .NET 目标 | net9 + net481 | 仅 net9.0 |
| 设置保存 | 普通 File.WriteAllText | 防抖保存 + 原子写入 + 自动备份 |

## 项目架构

```
ExplorerTabUtility/
├── App.xaml(.cs)          # 入口：单实例 Mutex、加载主题、初始化 i18n
├── Helpers/
│   ├── Constants.cs       # 全局常量（默认快捷键配置等）
│   ├── Helper.cs          # Win32 辅助方法
│   └── LocalizationService.cs  # i18n 中英切换，通过 .resx 实现
├── Hooks/
│   ├── ExplorerWatcher.cs # 核心：COM 事件监听、窗口转标签页逻辑
│   ├── Keyboard.cs        # 键盘钩子管理
│   └── Mouse.cs           # 鼠标钩子管理
├── Interop/               # 手动 COM 类型定义（替代 MSBuild COMReference）
│   ├── ComEventSink.cs    # COM 事件订阅/取消（使用 ComEventsHelper）
│   ├── SHDocVw.cs         # 浏览器/Shell 窗口 COM 接口
│   ├── Shell32.cs         # Shell 文件操作 COM 接口
│   └── ShellPathComparer  # PIDL 路径比较
├── Managers/
│   ├── ThemeManager.cs    # 主题管理：深色/浅色/跟随系统，实时换肤
│   ├── SettingsManager.cs # 设置持久化：防抖保存 + 原子写入 + .bak 备份
│   ├── ProfileManager.cs  # 快捷键配置方案管理
│   ├── HookManager.cs     # 钩子总控
│   └── UpdateManager.cs   # 自动更新（已废弃，保留空壳）
├── Models/
│   ├── HotKeyProfile.cs   # 快捷键配置数据模型
│   ├── HotKeyAction.cs    # 操作类型枚举（Open, ToggleVisibility 等）
│   ├── HotkeyScope.cs     # 作用域枚举（Global, FileExplorer）
│   ├── DisplayItem.cs     # ComboBox 绑定用的通用包装类
│   └── WindowInfo.cs      # 窗口状态跟踪
├── UI/
│   ├── Themes/            # XAML 主题文件
│   │   ├── Colors.xaml         # 基础颜色（编译时加载，深色）
│   │   ├── Colors.Dark.xaml    # 深色主题颜色
│   │   ├── Colors.Light.xaml   # 浅色主题颜色
│   │   ├── DefaultStyles.xaml  # 全局默认样式 & 字体
│   │   ├── ThemeResources.xaml # 引用各控件样式文件
│   │   ├── ButtonStyles.xaml   # 按钮样式（含 BasedOn 引用）
│   │   ├── ComboBoxStyles.xaml # 下拉框样式
│   │   └── ...                 # 其他控件样式
│   ├── Views/
│   │   ├── MainWindow.xaml(.cs)     # 主窗口，偏好设置/快捷键配置/关于
│   │   ├── HotKeyProfileControl    # 单个快捷键配置卡片控件
│   │   ├── SystemTrayIcon          # 系统托盘图标和菜单
│   │   ├── TabSearchPopup          # 标签页搜索弹出窗口
│   │   └── CustomMessageBox        # 自定义消息弹窗
│   ├── Behaviors/          # WPF 附加行为（HoverBackground 等）
│   ├── Converters/         # 值转换器（EnumDescriptionConverter 等）
│   └── Commands/           # RelayCommand
├── WinAPI/                 # Win32 P/Invoke 声明
├── Properties/
│   ├── Resources.resx          # 英文资源文件（75 条字符串）
│   └── Resources.zh-CN.resx    # 中文资源文件（75 条字符串）
└── installers/
    └── installer.iss           # Inno Setup 安装脚本
```

## 关键流程

### 启动流程
```
App.OnStartup()
  → 创建单实例 Mutex
  → 从 settings.json 读取语言设置 → LocalizationService.SetLanguage()
  → ThemeManager.Initialize() → 监听系统主题变化
  → ThemeManager.ApplyTheme() → 加载 Colors.Dark/Light.xaml
  → new MainWindow() → 初始化系统托盘、启动钩子
```

### 主题切换
```
用户选主题 → CbTheme_SelectionChanged
  → ThemeManager.CurrentTheme = AppTheme.Light
  → SettingsManager.ThemeMode = (int)value
  → ThemeManager.ApplyTheme()
    → 从 MergedDictionaries 移除旧颜色字典
    → 添加新颜色字典 → DynamicResource 自动更新所有控件
```

### 设置保存
```
属性变更 → DebounceSave() → 500ms 防抖 → DoSave()
  → WriteAtomic():
    1. 序列化为 JSON 写入 .tmp 文件
    2. 复制当前文件到 .bak 备份
    3. 删除原文件
    4. File.Move(.tmp → 正式文件)
    
恢复: 正式文件损坏时自动从 .bak 恢复
```

## 已知陷阱

### 1. XAML 中 DynamicResource 的限制
`DynamicResource` 只能用于 `DependencyProperty`，以下位置**必须**用 `StaticResource`：
- ❌ `Style.BasedOn="{DynamicResource ...}"` → ✅ `{StaticResource ...}`
- ❌ `Binding.Converter="{DynamicResource ...}"` → ✅ `{StaticResource ...}`
- ✅ `Control.Foreground="{DynamicResource ...}"` 正常
- ✅ `Setter Value="{DynamicResource ...}"` 正常

### 2. HotKeyProfileControl 初始化顺序
**绝对不能**在设 `CbAction.ItemsSource` 之前设 `CbScope.SelectedValue`。
设 scope 会触发 `SelectionChanged` → `UpdateActionComboBox()`，
此时若 `CbAction.ItemsSource` 为空，`SelectedValue = null`，
退化为 `default(HotKeyAction) = Open`，覆盖 `_profile.Action`。
已通过 `_isInitializing` 守卫标志解决，新代码中修改初始化逻辑时注意。

### 3. 设置保存的竞态
`DebounceSave()` 用 500ms 防抖，`ForceSave()` 立即保存。
多处快速修改会不断重置计时器，确保退出时调用 `ForceSave()`。

### 4. COM 事件生命周期
`ComEventSink.Connect()` 需要 `AllowUnsafeBlocks`。
必须在使用后调用 `.Dispose()` 取消 COM 事件订阅，否则内存泄漏。
`ExplorerWatcher.cs` 中通过 `windowInfo.OnQuitSink` 等字段管理。

### 5. 单实例 Mutex
`App.OnStartup` 用命名 Mutex 确保单实例。
主题切换或语言切换不需要重启（实时生效），但如果有需要重启的功能：
必须先调用 `app.ReleaseMutex()` 再 `Process.Start(exe)`。

## i18n 国际化

- 所有 UI 字符串通过 `LocalizationService.Get("Key")` 获取
- 英文: `Properties/Resources.resx`
- 中文: `Properties/Resources.zh-CN.resx`
- XAML 中通过 `{Binding Source={StaticResource Loc}, Path=[Key]}` 绑定
- 语言切换: `LocalizationService.Instance.SetLanguage("zh-CN"|"en")`
- 语言变更后需调用各控件 `RefreshLocalization()` 刷新下拉框文字

## 构建和发布

```bash
# 本地构建
dotnet build

# 发布框架依赖版
dotnet publish -c Release -o ./publish-fd

# 推送到 GitHub（标签推送触发 CI）
git push fork master
git tag v2.5.1
git push fork v2.5.1
```

CI 工作流 (`.github/workflows/build-release.yml`)：
- 标签 `v*` 推送自动触发
- 构建 x64/x86/arm64 框架依赖包
- 可选 SignPath 代码签名（需配置 `SIGNPATH_API_TOKEN`）
- 生成 Inno Setup 安装程序
- 创建 GitHub Release（draft）

## 项目历史

本分支基于 fork `391f1ac`，仅做了以下最小改动：
1. 修复 5 处 `BasedOn="{DynamicResource}"` → `StaticResource`（XamlParseException 崩溃）
2. 修复 1 处 `Binding.Converter="{DynamicResource}"` → `StaticResource`（偏好设置闪退）
3. 补全 4 条硬编码英文弹窗的中文翻译
4. 修复配置被初始化顺序悄悄重置的 Bug
5. 清理重复 using 指令
6. 版本号 2.5.1
