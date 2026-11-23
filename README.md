# 碧蓝航线CD计算工具

原项目演示: [GithubPages](https://azurlanetools.github.io/az-cd/)

该工具的数据和计算公式主要来源于[碧蓝航线Wiki](https://wiki.biligame.com/blhx/%E9%A6%96%E9%A1%B5)和[碧蓝航线](https://game.bilibili.com/blhx/)的游戏内数据.

## 项目结构与数据维护

这是一个 Vue + TypeScript + Vite 的纯静态网页，没有应用后端或数据库。舰船计算在浏览器中完成，舰队和个人设置保存在当前浏览器的 localStorage 中，不会自动同步到其他设备。

- 前端源码位于 `vue/src/`。
- 舰船、装备数据分别位于 `vue/public/data/ships.json` 和 `vue/public/data/equips.json`，由网页直接读取。
- 数据在仓库中手动维护。提交数据修改后，构建会将这些文件原样复制到静态站点中。
- `load_azcd_data.py` 和 `utils/` 是独立的数据准备工具，不参与网页构建或运行。

当前数据以国服 `9.6.657` 为基线，另于 2026-11-24 补入 9 艘舰船和 2 件九期科研装备。舰船装填与 Wiki 满级面板核对，游戏 ID 和槽位编码使用国服客户端配置。两件装备的 `cd` 由 Wiki 的 `+10` 标准射速（23.85 和 12.20 秒/轮）换算。新增技能为图鉴资料，技能对时间轴的影响待建模。[Wiki 舰船装填参数](https://wiki.biligame.com/blhx/MediaWiki:PN.js) · [九期科研专题](https://wiki.biligame.com/blhx/碧蓝海事局科研九期活动专题) · [国服客户端配置](https://github.com/AzurLaneTools/AzurLaneLuaScripts/tree/main/CN)

舰船头像随站点静态发布，资源位于 `vue/public/img/squareicon/`。这次补齐的头像来自碧蓝航线 Wiki。

## 本地开发与构建

使用 Node.js 24；`vue/.nvmrc` 记录了与 CI 一致的版本。以下命令通过 npx 使用固定的 Yarn 1.22.22，无需预先全局安装 Yarn。

```sh
cd vue
npx --yes yarn@1.22.22 install --frozen-lockfile --non-interactive
npx --yes yarn@1.22.22 dev
```

构建生产文件：

```sh
cd vue
npx --yes yarn@1.22.22 build
```

构建结果位于 `vue/dist/`，可交给 GitHub Pages 或其他静态文件服务器托管。Node.js 只用于开发和构建，部署后的网页不需要常驻 Node.js 或 Python 服务。可在 `vue/` 中运行 `npx --yes yarn@1.22.22 serve` 预览构建结果。

## GitHub Pages 部署

工作流直接使用仓库中的 JSON 数据，不再自动抓取游戏数据，也不需要 Docker 镜像或 `GHCR_TOKEN`。

- 向 `master` 推送代码或数据修改：构建成功后发布到 `gh-pages` 分支。
- 向 `master` 提交 PR：只检查构建，不部署。
- 在 Actions 中手动运行工作流：始终构建，只有选择 `master` 分支时才部署。

在 GitHub 仓库的 **Settings → Pages** 中选择 **Deploy from a branch**，分支选择 **gh-pages**，目录选择 **/(root)**。发布使用 GitHub Actions 自动提供的 `GITHUB_TOKEN`，无需创建个人 Token。

## TODO
### CD计算器
- [x] 简单CD展示
- [x] 绑定技能CD展示
- [x] 交互界面优化
- [x] 自定义对轴项解析和展示
- [x] 基于舰娘和装备信息的CD计算
- [x] 记录保存和加载(基于localStorage实现)
- [ ] 记录分享
- [x] 界面优化: 移动端适配
- [x] 基本功能
- [x] 舰载机筛选: 支持对各位置的舰载机进行进一步的筛选
- [ ] 排序方式优化: 支持按Buff覆盖率排序
- [x] 界面优化: 移动端适配
- [x] 界面优化: 舰船以名称和ID展示，支持按ID或名称搜索
- [ ] 界面优化: 显示装备图标

## 开源协议

[CC BY-NC-SA 3.0](https://creativecommons.org/licenses/by-nc-sa/3.0/legalcode)

## 已知问题

- 航战只支持主炮时间轴和战列相关设备

目前所有的舰船只支持一条武器时间轴，这导致航战目前只有主炮时间轴、而没有舰载机时间轴。相应的，第4、5装备槽也只列出了战列相关设备

- 战列公共CD问题 #2

因公共CD的处理逻辑, 战列实际开炮时间在部分情况下会和游戏内不一致.
由于实际机制尚不明确而本人暂时没空进行详细测试, 并且这种情况出现概率较小, 所以暂时不会进行修复.

- 约克城II的'翱翔天际之鹰'技能

由于机制较复杂, 暂时无法模拟
