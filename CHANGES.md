# v2.5.1 发行说明

基于 [w4po/ExplorerTabUtility v2.5.0](https://github.com/w4po/ExplorerTabUtility) 的修复增强版。

## 新增

- **中英双语界面**：支持一键切换，自动检测系统语言，所有菜单、设置、提示均已汉化
- **深色/浅色/跟随系统主题**：支持三种主题模式，实时切换无需重启

## Bug 修复

- 修复 5 处 `Style.BasedOn` 非法使用 `DynamicResource` 导致的应用启动崩溃
- 修复 `Binding.Converter` 非法使用 `DynamicResource` 导致的偏好设置页闪退
- 修复控件初始化顺序错误导致的快捷键配置被悄悄重置为默认值
- 修复 4 处硬编码英文弹窗未跟随语言设置的问题

## 改进

- 设置文件保护：防抖保存 + 原子写入 + 自动备份，防止配置文件损坏
- 改进设置文件保存机制，防止配置文件损坏
- 字体优化：使用微软雅黑改善中文显示效果

## 变更

- 仅支持 .NET 9.0（移除 .NET Framework 4.8.1 支持）
- 移除自动更新功能（通过 GitHub Releases 手动更新）
