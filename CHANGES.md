# 对比原版 (w4po/ExplorerTabUtility v2.5.0) 的修改说明

## 新增功能

### 中英双语界面
- 新增 `LocalizationService.cs`，通过 .resx 资源文件实现中英文切换
- 新增 `Properties/Resources.resx`（英文，75 条）和 `Resources.zh-CN.resx`（中文，75 条）
- 偏好设置页新增语言切换下拉框，选择后界面即时刷新
- 所有菜单、按钮、提示、快捷键操作说明均已汉化

### 深色/浅色/跟随系统主题
- 新增 `ThemeManager.cs`，支持三种主题模式实时切换，无需重启
- 新增 `Colors.Dark.xaml`（深色配色）和 `Colors.Light.xaml`（浅色配色）
- 偏好设置页新增主题下拉框
- 新增 `DisplayItem.cs`，用于 `ComboBox` 数据绑定

## 已修复的原版 Bug

### 启动崩溃
原版的 XAML 文件中有 5 处 `Style.BasedOn` 使用了 `DynamicResource`，这在 WPF 中不合法，导致应用启动时抛出 `XamlParseException` 崩溃。已改为 `StaticResource`。

涉及文件：`ButtonStyles.xaml`（2 处）、`ComboBoxStyles.xaml`（1 处）、`TextBoxStyles.xaml`（2 处）

### 偏好设置页闪退
`ComboBoxStyles.xaml` 中 `Binding.Converter` 使用了 `DynamicResource`，同样不合法。点击偏好设置页时触发渲染，导致 `NullReferenceException` 闪退。已改为 `StaticResource`。

### 配置被悄悄重置
`HotKeyProfileControl` 的初始化顺序问题：设 `CbScope.SelectedValue` 时触发 `SelectionChanged` 事件，此时 `CbAction.ItemsSource` 尚未设置，`SelectedValue` 为 null，退化为默认值 `Open`，覆盖了用户的配置。下次保存时这个错误值就持久化到了磁盘。已通过 `_isInitializing` 守卫标志修复。

### 硬编码英文弹窗未汉化
4 处代码中直接写死英文文本的对话框已接入 i18n 系统：
- "Another instance is already running"（重复启动提示）
- "Do you want to restore previously opened windows?"（恢复窗口确认）
- "Are you sure you want to clear the closed windows history?"（清除历史确认）

## 改进优化

### 设置文件保护
`SettingsManager` 重写为防抖保存 + 原子写入 + 自动备份：
- 修改设置后 500ms 防抖，避免频繁写盘
- 原子写入：先写 `.tmp`，备份旧文件为 `.bak`，再 `File.Move` 替换
- 正式文件损坏时自动从 `.bak` 恢复

### COM 互操作重构
从 MSBuild `COMReference`（依赖系统类型库）改为手动 C# 类型定义（`Interop/SHDocVw.cs`、`Interop/Shell32.cs`），配合 `ComEventSink.cs` 管理 COM 事件生命周期。更可控，不依赖构建环境。

### 界面细节
- 新增 `ComboBoxStyles.xaml`，统一下拉框样式
- 新增 `ThemeResources.xaml`，组织各控件主题文件
- 字体改为 `Microsoft YaHei`，优化中文显示
- `DefaultStyles.xaml` 增加了 `TextElement` 和 `ToolTip` 的全局样式
- 窗口标题栏、菜单导航栏、状态栏等 UI 元素重新布局

## 移除的功能

- **自动更新**：移除 `Autoupdater.NET` 包和 `UpdateManager`
- **.NET Framework 4.8.1 支持**：仅保留 `net9.0-windows`
- **Chocolatey/Winget 包**：安装脚本中移除

## CI/CD 变更

- `build-release.yml` 重构：`build → sign-artifacts → build-installer → create-release`
- 新增 SignPath 免费代码签名支持（需配置 token）
- 安装程序新增 .NET 9 可用性检测逻辑
- 移除 .NET Framework 4.8.1 构建矩阵

## 文件统计

| 类别 | 数量 |
|------|------|
| 新增文件 | 13 个 |
| 修改文件 | 31 个 |
| 新增代码 | 1877 行 |
| 删除代码 | 809 行 |
