---
name: 1997-pro-web-reverse-casebook
description: 基于 1997.pro 全站 42 篇文章提炼的 Web/JS 逆向案例知识库。适用于 Akamai/Kasada/PX/reese84/同盾/a_bogus/腾讯滑块/阿里滑块/JSVMP/227/226/wasm/protobuf/rid/fuid/fs/bx-pp/run_js/storage.estimate/animationend 等案例线索，以及风控指纹、补环境、JSVMP/平坦流/WASM、验证码、协议参数链的场景判断、方法路由与案例对照；默认与 web-js-reverse-master-flow 及 jshook + js-reverse + chrome-devtools-mcp 三 MCP 协同使用。
---

# 1997.pro Web Reverse Casebook

## 角色

这个 skill 把 `1997.pro` 公开文章语料压成“案例知识库 + 场景路由器”。

它负责：

- 当任务出现厂商名、参数名、壳形态或风控线索时，快速判断最像哪一类案例
- 告诉你先读哪份 reference，先查哪类证据，先用哪组 MCP 动作
- 把 `1997.pro` 的案例经验桥接到现有执行 skill，而不是另起一条平行总控流程

它不替代：

- `$web-js-reverse-master-flow`
- `$jsr-reverse`
- `$mcp-js-reverse-playbook`

复杂任务仍然要回到这些总控或执行 skill。本 skill 只负责“像哪类问题、该从哪下刀”。

职责边界：

- 这个 skill 是专门站点 / 技术案例库，负责按 `1997.pro` 的文章体系提供案例参照、技术锚点和起刀建议。
- 它不接管宏观阶段判定，不重写 `证据门 -> locate -> recover -> runtime -> env-patch -> replay` 这条主流程。
- 如果当前任务已经进入正式阶段切换，必须把阶段裁决权交还给 `$web-js-reverse-master-flow`。

## 默认协同

默认执行栈不变：

- `chrome-devtools-mcp`
- `js-reverse`
- `jshook`

经验用法：

- 先用 `chrome-devtools-mcp` 与 `js-reverse` 拿真实请求、真实中间态、真实触发动作。
- 当线索已经像 `1997.pro` 某类案例，再用本 skill 选 reference 和专项 skill。
- 遇到重混淆、复杂 Hook、Stealth 或高级语义恢复时，再把 `jshook` 拉到前台。
- 如果当前任务已经进入正式阶段切换，最终阶段判断仍交给 `$web-js-reverse-master-flow`。

## 何时使用

当出现以下任一情况时使用本 skill：

- 用户明确提到 `1997.pro`、站内文章名、作者相关文章、或想“按这个博客的方法做”
- 线索指向具体厂商 / 产品 / 参数：
  - `Akamai`、`Kasada`、`PX3`、`reese84`、`Incapsula`、`同盾 BlackBox`、`a_bogus`
  - `x-s3-s4e`、`desc`、`bx-pp`、`rid`、`fuid`、`fs`
- 线索指向具体壳或运行时问题：
  - `JSVMP`、`while + switch`、`dispatcher`、`227/226`
  - `wasm`、`worker`、`postMessage`、`storage.estimate`
  - `animationend`、CSS 动画取值、`run_js` 检测、浏览器与 Node 不一致
- 用户要的是“案例参照 / 选型 / 速查表”，而不是纯流程编排

## 重点案例锚点

优先按这四组识别问题，不要把所有线索混成一个“大逆向题”：

### 1. 风控 / 厂商锚点

- `Akamai`
- `Kasada`
- `PX` / `PX3`
- `reese84` / `Incapsula`
- `同盾` / `BlackBox`
- `a_bogus`

默认读法：

- 先怀疑 `token lane`、`sign lane`、`fingerprint lane`、`challenge lane` 混杂
- 优先去读 `risk-fingerprint-and-vendors.md`
- 一般会反向驱动 `$web-js-reverse-master-flow` 的 `Phase 1 + Phase 3`

### 2. 验证码 / 业务字段锚点

- `腾讯滑块`
- `阿里滑块`
- `_rand`
- `fuid`
- `fs`
- `rid`

默认读法：

- 先拆“位置 / 图像 / 角度链”和“轨迹 / 加密 / 验签链”
- 腾讯 / 阿里滑块更容易同时命中浏览器特性通道与验证码双参数链
- 默认去读 `captcha-protocol-and-mobile.md`

### 3. 壳层 / 混淆锚点

- `JSVMP`
- `227`
- `226`
- `平坦流`
- `dispatcher`
- `while + switch`
- `opcode`
- `basearr`

默认读法：

- 不先补环境，先做壳层恢复
- 默认去读 `jsvmp-wasm-and-deobf.md`
- 反向驱动 `$web-js-reverse-master-flow` 的 `Phase 2`

### 4. 运行时 / 桥接锚点

- `wasm`
- `protobuf`
- `bx-pp`
- `run_js`
- `storage.estimate`
- `animationend`
- `worker`
- `postMessage`

默认读法：

- 先判断是 builder / writer 遮蔽，还是浏览器特性通道与运行时分叉
- `wasm/protobuf/bx-pp` 更偏 `recover`
- `run_js/storage.estimate/animationend` 更偏 `runtime`

## 使用顺序

1. 先读 [routing-matrix.md](references/routing-matrix.md) 做场景归类。
2. 若用户提到具体文章、具体厂商或想确认覆盖面，再读 [article-index.md](references/article-index.md)。
3. 按当前阻塞只加载一份专题 reference：
   - 风控、指纹、厂商产品：`risk-fingerprint-and-vendors.md`
   - 浏览器通道、补环境、自动化对抗：`browser-runtime-and-env.md`
   - `JSVMP`、平坦流、`wasm`、反混淆：`jsvmp-wasm-and-deobf.md`
   - 验证码、协议参数、移动端案例：`captcha-protocol-and-mobile.md`
4. 然后把任务 handoff 到最合适的执行 skill。

强化规则：

- 命中 `Akamai/Kasada/PX/reese84/同盾/a_bogus` 时，除非证据非常充分，否则不要直接谈 purecalc。
- 命中 `JSVMP/227/226/平坦流/混淆` 时，默认只恢复最小切片，不做全量 beautify。
- 命中 `wasm/protobuf/rid/bx-pp` 时，先找 builder、桥接层和写回边界。
- 命中 `run_js/storage.estimate/animationend` 时，先把它当运行时分叉证据，不要直接把它归入“缺对象”。

## Handoff 规则

### 总控 / 取证

- 复杂 Web 任务默认回 `$web-js-reverse-master-flow`
- 需要阶段判定与工具化时回 `$jsr-reverse`
- 需要最小请求链 / sink 定位时回 `$jsr-locate` 或 `$js-reverse-trace-hook`

### 壳层恢复

- `JSVMP` / dispatcher / 平坦流：`$jsr-recover`
- 控制流虚假分支裁剪：`$js-controlflow-truth-sampling-prune`
- 别名、字符串池、多层赋值：`$js-ast-binding-alias-deobf`
- Wasm 型 VM：`$js-wasm-vmp-ir-lifting`
- Webpack 运行时 / 打包桥：`$js-webpack-runtime-node-reuse`

### 运行时与补环境

- 浏览器与 Node 分叉：`$jsr-runtime`
- 最小环境迁移：`$env-patch`
- 已能跑但结果仍漂移：`$js-runtime-diff-env-patching`
- 反调试 / DevTools 摩擦：`$js-reverse-env-antidebug`

### 稳定复现 / 协议

- 参数链回放与回归：`$js-reverse-sign-replay`
- 验证码双参数链：`$captcha-parameter-chain-bisection`
- 无 schema 的 protobuf 二进制：`$protobuf-schema-backfill`
- Android 侧 QUIC / So 抓包降级：`$android-quic-downgrade-hook-capture`

## 你应该输出什么

使用本 skill 时，优先输出以下内容，而不是泛泛讲案例：

- 当前目标最像哪类 `1997.pro` 案例
- 该类案例的常见“隐藏状态载体 / 隐藏通道 / 壳形态”
- 三个最先验证的现场证据点
- 该读哪份 reference
- 下一步应该切到哪个执行 skill，为什么
- 当前仍未证明的假设

如果当前任务命中 `JSVMP / 平坦流 / wasm / 混淆`，必须额外输出：

- 当前更像 `dispatcher` 壳、字符串解密壳、`wasm` 桥，还是浏览器特性通道
- 关键状态载体是什么：`ip/g`、寄存器数组、栈对象、常量池、样式终态、storage、message 通道
- 最小需要恢复到多深：只要入口 / builder，还是需要关键 `opcode` 家族 / imports / exports
- 为什么当前应该先 `recover`、先 `runtime`，还是先回 `locate`

如果当前任务命中 `Akamai/Kasada/PX/reese84/同盾/a_bogus/腾讯滑块/阿里滑块/rid/fuid/fs/bx-pp/run_js/storage.estimate/animationend`，必须额外输出：

- 这一类问题最常见的隐藏状态载体
- 最可能的第一分叉点
- 不该先做的误操作
- 该回到 `$web-js-reverse-master-flow` 的哪个 phase

## 不要这样用

- 不要把“厂商名相似”当成已经证明算法相同。
- 不要跳过浏览器真实证据，直接套历史案例做纯算。
- 不要默认加载全部 references；只读当前最相关的一份或两份。
- 不要让案例经验盖过真实现场。这个 skill 提供的是“起刀位置”，不是证据本身。
- 不要因为看到了 `wasm` 或 `JSVMP` 就默认全量 lifting / 全量反编译。
