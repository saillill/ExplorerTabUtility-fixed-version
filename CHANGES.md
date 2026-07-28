# Explorer Tab Utility 修复版 v2.5.1

基于 [w4po/ExplorerTabUtility v2.5.0](https://github.com/w4po/ExplorerTabUtility/tree/v2.5.0) 修复增强。

## 新增

- **中英双语界面**：自动检测系统语言，偏好设置中可随时切换
- **主题切换**：深色、浅色、跟随系统三种模式，实时生效无需重启

## 修复

- 启动时因 XAML 写法错误导致的崩溃
- 打开偏好设置页面的闪退问题
- 快捷键配置偶尔被重置为默认值的 Bug
- 窗口打开关闭过快时未能合并为标签页的问题
- 几处弹窗未跟随语言设置仍显示英文的问题

## 改进

- 设置文件保存更可靠，意外断电后可以从备份恢复
- 优化了中文显示字体

## 其他

- 同时提供 .NET 9.0 和 .NET Framework 4.8.1 版本
- 支持 x64、x86、arm64 三种架构
- 通过 GitHub Actions 自动构建发布
