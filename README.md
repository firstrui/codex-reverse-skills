# Codex Reverse Skills

一组面向 Web/JS 逆向任务的 Codex skill。

当前包含：

- `skills/web-js-reverse-master-flow`
  - 复杂 Web/JS 逆向任务的宏观总控 skill
  - 默认按 `jshook + js-reverse + chrome-devtools-mcp` 三 MCP 协同
  - 内置 `Anti-Spiral Protocol`、困难站点升级梯子、阶段切换纪律
- `skills/1997-pro-web-reverse-casebook`
  - 基于 `热门站点` 提炼的专门技术案例库
  - 用于 `Akamai / Kasada / PX / reese84 / 同盾 / a_bogus / 腾讯滑块 / 阿里滑块 / JSVMP / wasm / protobuf / fs / bx-pp / animationend` 等案例线索的预路由与起刀建议

## 安装

把仓库中的 skill 目录复制到本机 Codex skill 目录：

```bash
cp -R skills/* ~/.codex/skills/
```

如果目标目录里已存在同名 skill，先自行备份或比对差异。

## 使用

复杂站点默认先用：

```text
$web-js-reverse-master-flow
```

命中具体站点 / 厂商 / 技术案例线索时，再叠加：

```text
$1997-pro-web-reverse-casebook
```

推荐分工：

- `web-js-reverse-master-flow` 负责宏观阶段判定、反卡死纪律、困难站点升级路径
- `1997-pro-web-reverse-casebook` 负责案例参照、技术锚点、起刀点建议

## 目录结构

```text
skills/
  web-js-reverse-master-flow/
  1997-pro-web-reverse-casebook/
```

## 说明

- `1997-pro-web-reverse-casebook` 不包含站点私有内容。
- 这些 skill 偏向研究、调试与方法路由，不保证适配所有目标，也不替代真实证据采集。
