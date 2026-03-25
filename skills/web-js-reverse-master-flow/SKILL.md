---
name: web-js-reverse-master-flow
description: 用于复杂 Web/JS 网站还原的总控流程。适用于 sign/token/cookie/header/body/websocket 字段逆向、重混淆、花指令、控制流平坦化、JSVMP、worker/wasm、浏览器与 Node 差分、补环境、本地复现与回归，也覆盖 Akamai/Kasada/PX/reese84/同盾/a_bogus/腾讯滑块/阿里滑块/JSVMP/227/226/wasm/protobuf/rid/fuid/fs/bx-pp/run_js/storage.estimate/animationend 等案例线索。默认按 jshook + js-reverse + chrome-devtools-mcp 三 MCP 协同调试分析，并在 locate、recover、runtime、env-patch、replay 各阶段切换专项 skill。
---

# Web JS Reverse Master Flow

## 角色

这个 skill 是复杂 Web/JS 逆向项目的总控入口。

它不替代专项 skill，而是负责：

- 归一化当前任务阶段
- 选择下一步最合适的专项 skill
- 约束切换顺序，避免“还没坐实请求链就去补环境”这类误操作
- 输出当前阶段、阻塞点、下一份必须更新的工具

默认情况下，复杂网站还原一律从这个 skill 起手，而不是直接并行触发多个总控 skill。

职责边界：

- 这个 skill 始终是宏观总控，不下沉成某个厂商、某个站点或某种壳的案例库。
- 即使出现 `Akamai/Kasada/PX/reese84/同盾/a_bogus/腾讯滑块/阿里滑块/JSVMP/227/226/wasm/protobuf/rid/fuid/fs/bx-pp/run_js/storage.estimate/animationend` 这类强线索，也只是帮助预路由，不改变它“先判阶段、后选专项”的总控角色。
- 具体站点、厂商和技术经验，应交给 `$1997-pro-web-reverse-casebook` 或其他专项 skill 提供。

如果现场线索已经明显命中某类公开案例，应同步加载 `$1997-pro-web-reverse-casebook` 做案例预路由，但阶段选择仍由本 skill 决定。

## MCP 栈

这个 skill 默认采用三 MCP 协同，不把它们视作可选项：

- `chrome-devtools-mcp`
  负责浏览器接管、页面级调试、Network/DOM/Runtime/CDP 断点与调用栈观察，是最直接的现场取证面。
- `js-reverse`
  负责逆向工作流级别的源码搜索、Hook、断点、代码采集、运行时证据沉淀、WebSocket/Storage/Session 状态分析。
- `jshook`
  负责更重的 JS 逆向分析、反混淆、Stealth、Hook、浏览器自动化、调试器与代码理解增强。

使用规则：

- 先用 `chrome-devtools-mcp` 和 `js-reverse` 拿真实样本与现场证据。
- 进入重混淆、恢复语义层或高级 Hook/Stealth 阶段时，再显式引入 `jshook`。
- 不允许只凭静态阅读跳过浏览器现场证据。
- 不允许在没有浏览器基线样本时直接讨论 Node 迁移。

## 输入块

从以下输入开始：

```text
URL 或目标页面：
目标请求 / 字段 / cookie / 消息：
触发动作：
当前现象：
已有证据：
目标：
约束：
```

先补齐输入，再决定阶段。

## 总原则

- Hook-first, Breakpoint-second, Full-dump-last。
- 先证明真实请求链，再讨论纯算、补环境或重放。
- 先恢复语义层，再迁运行时，不要反过来。
- 运行时差分优先于盲补环境。
- 每次只扩大一个变量：一个 hook 点、一个样本、一个补丁、一个分支判断。
- 任何结论都要附带运行时证据或中间态检查点。
- 案例锚点只能帮助预路由，不能代替证据门与阶段判定。

## Anti-Spiral Protocol

这个 skill 的首要目标之一，是防止在难站点里陷入“长时间没有证据增量的深挖漩涡”。

以下任一情况出现时，视为已经开始打转：

- 连续多轮都在同一层做相似操作，但没有新增请求链、写入边界、状态载体或检查点
- 不断追加 Hook / 断点 / patch，却解释不了第一个真实分叉
- 花大量时间 beautify / AST 变换，但仍然没把逻辑绑回真实 sink
- 本地补环境越补越大，却说不清到底是哪一个对象或状态在影响结果
- 只盯最终值，不再比较中间态

强制规则：

- 同一阶段连续两轮没有新增证据，必须切换动作，不允许第三次重复同类操作。
- 同一问题连续六轮仍无收敛，必须停止当前打法，升级到下一层策略。
- 任何升级都必须写明：为什么当前打法失败、下一步准备验证什么、放弃了哪些假设。

允许的升级方向只有这些：

- `locate` 卡住：扩大真实触发范围、改抓 initiator、改用断点栈或浏览器侧调用
- `recover` 卡住：降低恢复深度、退回 `locate` 重新坐实边界，或改 black-box reuse
- `runtime` 卡住：回退到第一个稳定浏览器快照，重建分叉表，再只 patch 一个状态面
- `env-patch` 卡住：停止继续补对象，改走浏览器辅助执行、远程调用或最小代理方案

禁止：

- 在没有新证据时继续加大 dump 范围
- 在没证明 builder / writer 边界前做全量反编译
- 在没证明首个分叉点前做全量浏览器环境迁移
- 用“这个站很难”代替阶段判断和停止深度判断

## 案例锚点预路由

以下线索出现时，不要先猜算法；先把它们映射到更可能的阶段：

- 厂商 / 产品锚点：
  - `Akamai`、`Kasada`、`PX`、`PX3`、`reese84`、`Incapsula`、`同盾`、`BlackBox`、`a_bogus`
  - 默认怀疑：`Phase 1` 的请求链 + `Phase 3` 的风控 / 指纹 / 挑战分叉
- 字段 / 协议锚点：
  - `rid`、`protobuf`、`x-s3-s4e`、`desc`、`bx-pp`
  - 默认怀疑：`Phase 1` 的 builder / writer 边界，必要时进入 `Phase 2` 看 `wasm` / 封装桥
- 验证码 / 业务锚点：
  - `腾讯滑块`、`阿里滑块`、`_rand`、`fuid`、`fs`
  - 默认怀疑：位置链与加密链双线并存，通常会同时触发 `Phase 1`、`Phase 3`
- 壳层锚点：
  - `JSVMP`、`227`、`226`、`while(true)+switch`、`dispatcher`、`basearr`
  - 默认直接怀疑 `Phase 2`
- 运行时锚点：
  - `run_js`、`storage.estimate`、`animationend`、`postMessage`、`worker`
  - 默认先做 `Phase 3` 的运行时差分，再决定是否进入 `Phase 4`

如果这些线索同时出现，优先顺序固定为：

`证据门 -> 写入边界 -> 壳层恢复(必要时) -> 运行时差分 -> Node 迁移`

## 主流程

### Phase 0: 证据门

目标：
- 证明目标请求、字段和触发动作都来自真实样本，而不是猜测。

要做：
- 先按 `$mcp-js-reverse-playbook` 跑最小链路。
- 强制使用 `chrome-devtools-mcp` 或 `$js-reverse` 建立浏览器基线样本。
- 创建或刷新 `reverse-records/总览.md`。
- 如果请求链、状态链、样本链还不完整，立即创建或刷新 `reverse-records/请求链路.md`。

退出条件：
- 至少有一条真实样本链。
- 目标字段或消息已由真实请求证明。

禁止：
- 目标请求还没坐实就开始补环境。
- 目标字段还没坐实就直接做算法还原。

卡住信号：

- 只能说“怀疑是某个参数 / 某个脚本”，但拿不出真实请求样本
- 触发动作不稳定，重复操作拿不到同一条链

必切动作：

- 回到最小触发动作，重新抓一条干净样本
- 若页面脚本太乱，优先用 `chrome-devtools-mcp` 的 Network / 调用栈现场，而不是先读源码

### Phase 1: 写入边界与请求链

主 skill：
- `$find-crypto-entry`
- `$jsr-locate`
- 搜不到时可加 `$js-reverse-trace-hook`

MCP 优先级：
- `chrome-devtools-mcp`：Network、DOM 触发点、调用栈与断点现场
- `js-reverse`：请求 initiator、Hook、代码采集、storage/session/websocket
- `jshook`：当普通搜索/断点不足以稳定定位时，补充更重的 Hook 与逆向分析

目标：
- 找到真实 sink、`entry -> builder -> writer` 关系和上游依赖链。

适用：
- 参数名明确。
- 请求链还没完全闭合。
- 怀疑有 `Set-Cookie`、挑战、session、response-driven 依赖。

退出条件：
- 最终写入边界已坐实。
- 当前阻塞已经从“找链”变成“代码读不懂/壳太重/运行时分叉”。

切换到 Phase 2 的信号：
- `while(true) + switch`
- 大量花指令、真假分支
- `JSVMP`
- `227/226/basearr/opcode/ip/g`
- `worker/wasm/webpack runtime`
- `_0x` 字符串表、动态 `eval/Function`
- `LL()[XX(XX)]()`、多套字符串解密器、动态回填字符串
- `Akamai/Kasada/PX` 类动态字符串与桥接壳
- `bx-pp`、`rid`、`protobuf builder` 的真实 builder 被 dispatcher 遮住
- 只能看到 dispatcher，看不到真实 builder

卡住信号：

- 能看到目标值，但无法说明是谁最后写入
- Hook 命中很多函数，但没有一个能稳定回到真实请求
- 反复在同一 bundle 搜关键词，没有把字段绑到请求边界

必切动作：

- 改抓请求 initiator 与调用栈，不继续盲搜源码
- 把问题收缩到 `entry -> builder -> writer` 三段，不再讨论全链路
- 若 builder 被壳遮蔽，立即切 `Phase 2`

### Phase 2: 壳层恢复与重混淆压缩

主 skill：
- `$jsr-recover`

补充 skill：
- `$js-controlflow-truth-sampling-prune`
- `$js-ast-binding-alias-deobf`

MCP 优先级：
- `js-reverse`：collect_code、trace、断点、Hook、源码级证据
- `jshook`：反混淆、Stealth、复杂 Hook、调试器控制、脚本级语义压缩
- `chrome-devtools-mcp`：保留浏览器现场，验证 dispatcher/bridge 在真实页面中的执行路径

目标：
- 把壳压到“足以继续 locate / runtime / replay”的程度，而不是一次性全还原。

要做：
- 识别 container、dispatcher、state carrier、bridge、core operator、write-back layer。
- 对平坦流或 dispatcher 做真实样本裁剪。
- 只恢复当前任务需要的最小 slice。
- 对 `JSVMP/227/226` 至少补齐：入口、状态变量、关键 `opcode` 家族、桥接合同。
- 对 `wasm` 至少补齐：下载点、实例化方式、imports、exports、JS 到 wasm 的调用桥。
- 对 `Akamai/Kasada/PX` 类重混淆，先冻结样本，再提字符串解密映射与最小执行片段。

退出条件：
- 下游工作不需要重新扒开同一层壳。
- 已能回到 `$jsr-locate` 坐实 sink，或进入 `$jsr-runtime` 处理运行时分叉。

铁律：
- 先 recover，再 runtime。
- 不要在 dispatcher 和 state carrier 还没搞清时就盲补 `window/document/navigator`。

卡住信号：

- 代码越来越可读，但仍然解释不了目标字段
- 一直在补 case / opcode，却没有把它们绑到目标写回路径
- 反混淆产物越来越大，但没有新增可验证检查点

必切动作：

- 降低恢复深度，先只保留关键 `opcode` 家族或 bridge
- 若 sink 关联性变弱，退回 `Phase 1`
- 若桥接合同已经清楚但结果仍不一致，切 `Phase 3`

停止纪律：

- 默认只允许做到“足以继续”的深度，不允许因为代码脏就自动升级到最小解释器
- `worker/wasm/JSVMP/227/226` 复合壳场景里，若 black-box reuse 已能支撑下游，不再继续全量 lifting

### Phase 3: 浏览器与本地差分诊断

主 skill：
- `$jsr-runtime`

补充 skill：
- `$js-runtime-diff-env-patching`
- 有反调试时加 `$js-reverse-env-antidebug`

MCP 优先级：
- `chrome-devtools-mcp`：建立浏览器正常路径与运行时现场
- `js-reverse`：抓 storage/session/network/websocket/hook 中间态
- `jshook`：反调试、Stealth、环境对抗、运行时行为补充验证

目标：
- 解释为什么浏览器能跑、本地跑不通，并输出最小运行时依赖清单。

优先分类：
- 缺对象
- 缺状态
- 反调试
- 指纹/风控分支
- 时间/随机数/seed 漂移
- `sign lane` 与 `token lane` 混杂
- 浏览器特性通道：`storage.estimate`、`animationend`、`postMessage`、`worker`
- 自动化摩擦：`run_js`、格式化检测、DevTools 开启即漂

退出条件：
- 已找到第一个真实失配检查点。
- 已说清哪些依赖必须迁、哪些依赖不能迁。

铁律：
- 先比中间态，再比最终值。
- 先 patch state，再 patch object。
- 先证明浏览器特性通道被真实消费，再 patch 浏览器对象表面。

卡住信号：

- 一直在补对象，但首个分叉点仍不清楚
- 最终值持续不一致，但没有中间态对照表
- 每次改动都影响很多面，无法知道是哪一项生效

必切动作：

- 回到浏览器正常态，重建“首个分叉点对比表”
- 每次只 patch 一个状态面：`cookie/storage/channel/style/time/random`
- 如果六轮内还不能解释分叉，停止补环境，改走浏览器辅助执行或代理复用

### Phase 4: Node 最小迁移

主 skill：
- `$env-patch`

MCP 优先级：
- `chrome-devtools-mcp`：冻结浏览器基线样本
- `js-reverse`：提取可迁移入口、中间态与依赖
- `jshook`：在需要更重的 Hook/Stealth/分析时辅助本地迁移判断

目标：
- 把浏览器真实需要的依赖迁到 Node，只迁最小集合。

进入条件：
- 入口已知。
- dispatcher / builder / bridge 基本清楚。
- 浏览器与本地差分已经归因。

退出条件：
- 本地能稳定走到目标 builder。
- 中间态可与浏览器对齐。

铁律：
- `env-patch` 是迁运行时，不是替代 `recover`。
- 不要一次性搬完整浏览器环境。

卡住信号：

- 本地补丁清单越来越长，但中间态没有更接近浏览器
- 只能靠堆对象让代码不报错，却仍进不了真实 builder
- 每次修一个对象都会引出更多对象，但没有证据证明它们被消费

必切动作：

- 停止补对象，回到 `Phase 3` 只看状态与分叉
- 若入口已知但本地仍跑不稳，改用浏览器远程调用 / browser-assisted replay
- 若 Node 迁移成本明显高于收益，保留 browser-side black-box reuse，不强行本地化

### Phase 5: 稳定复现与回归

主 skill：
- `$js-reverse-sign-replay`

MCP 优先级：
- `chrome-devtools-mcp`：对照真实浏览器结果
- `js-reverse`：验证中间态与状态依赖是否闭合
- `jshook`：对复杂混淆或高级算法段做补充分析

目标：
- 把链路固化成可维护、可回归的本地复现脚本。

要做：
- 切分 `normalize -> concat -> hash/encrypt -> encode`
- 固化输入契约与阶段检查点
- 提供 baseline 与 regression

退出条件：
- 固定输入稳定一致
- 变体输入行为可解释
- 后续版本漂移时能快速定位断点

卡住信号：

- 最终值能对上，但无法解释哪些中间态必须一致
- 输入略变就漂，但没有记录哪个阶段先漂

必切动作：

- 把回归拆回阶段检查点，不再只看最终值
- 若漂移来自运行时依赖，退回 `Phase 3/4`

## 复杂场景分支

### 分支 A：普通参数站

特征：
- 参数名明确
- 没有明显 VM 或 dispatcher 壳
- 代码仍可直接追

流程：
- Phase 0
- Phase 1
- Phase 5

### 分支 B：重混淆 / 花指令 / 平坦流 / JSVMP

特征：
- 大 `switch`
- dispatcher
- `JSVMP`
- `227/226`
- `basearr/opcode/ip/g`
- 大量真假分支
- `worker/wasm/runtime` 壳
- 厂商壳：`Akamai/Kasada/PX/reese84`

流程：
- Phase 0
- Phase 1
- Phase 2
- 回到 Phase 1 坐实 sink
- Phase 3
- Phase 4
- Phase 5

### 分支 D：验证码 / 浏览器特性通道题

特征：
- `腾讯滑块`、`阿里滑块`
- `_rand`、`fuid`、`fs`
- `animationend`、CSS 动画、样式终态取值
- `run_js`、自动化检测、浏览器与本地行为差异

流程：
- Phase 0
- Phase 1
- Phase 3
- 必要时回 Phase 2 恢复桥接层
- Phase 4
- Phase 5

### 分支 E：风控指纹 / 厂商挑战题

特征：
- `Akamai`、`Kasada`、`PX`、`reese84`、`Incapsula`
- `同盾`、`BlackBox`、`a_bogus`
- `abck`、`sensor_data`、`bm_sz`、`TLS/JA4`
- 浏览器可过，本地 `403` 或 challenge 持续跳

流程：
- Phase 0
- Phase 1
- Phase 3
- 必要时 Phase 2
- Phase 4
- Phase 5

### 分支 F：协议封装 / Wasm / Builder 题

特征：
- `protobuf`、`jspb`
- `rid`、`x-s3-s4e`、`desc`、`bx-pp`
- `wasm`、`compileStreaming`、imports/exports
- builder / writer 被 VM 或 wasm 桥遮蔽

流程：
- Phase 0
- Phase 1
- 必要时 Phase 2
- Phase 3
- Phase 4
- Phase 5

### 分支 G：Hard Site 复合场景

特征：

- 同时出现 `JSVMP/平坦流/worker/wasm`
- 同时存在 `run_js/storage.estimate/animationend` 一类浏览器特性通道
- 同时有 `Akamai/Kasada/PX/reese84/同盾/a_bogus` 一类风控分支
- Node 与浏览器差异巨大，且本地 patch 快速发散

流程：

- Phase 0
- Phase 1
- Phase 2，只恢复最小桥接切片
- 回到 Phase 1 重新坐实 sink
- Phase 3，先找第一个分叉点
- Phase 4，只在依赖清单已稳定时进入
- Phase 5

额外纪律：

- 这种场景禁止直接从 `Phase 2` 跳到“全量本地还原”
- 任何时候一旦失去边界锚点，优先退回 `Phase 1`

## Hard-Site Escalation Ladder

困难站点统一按这个升级梯子推进，不允许乱跳：

1. 浏览器最小样本
   - 坐实真实请求、触发动作、目标字段
2. 写入边界
   - 坐实 `entry -> builder -> writer`
3. 最小壳层恢复
   - 只恢复挡路的 dispatcher / bridge / imports / opcode
4. 首个分叉点诊断
   - 做浏览器 vs 本地对比表
5. 最小运行时迁移
   - 只迁被证明会影响结果的依赖
6. 浏览器辅助执行或 black-box reuse
   - 当本地化成本过高时接受 browser-side 方案
7. 稳定回归
   - 再做纯算、代理或回放固化

升级条件：

- 上一层已经产出稳定工具，但仍不能支撑下一步

禁止跨级：

- 没有第 2 步就做第 5 步
- 没有第 4 步就宣称本地环境已经足够
- 没有第 6 步评估就强行追求“必须纯本地”

### 分支 C：浏览器与 Node 差异很大

特征：
- 浏览器正常，本地失败
- 本地能生成值，但服务端持续 `403`
- 调试一开就漂
- 指纹、时间、随机数、风控敏感

流程：
- Phase 0
- Phase 1
- Phase 3
- Phase 4
- Phase 5

## 切换规则速查

- 参数名明确且未重混淆：先走 Phase 1。
- 看到 `JSVMP/227/226/平坦流/basearr/opcode`：先切 Phase 2。
- 看到 `wasm/bx-pp/protobuf builder/动态字符串壳`：通常先走 Phase 1，再视遮蔽程度切 Phase 2。
- 看到 `run_js/storage.estimate/animationend/postMessage/worker`：优先做 Phase 3。
- 看到 `Akamai/Kasada/PX/reese84/同盾/a_bogus`：优先检查 Phase 1 与 Phase 3 的混合阻塞。
- 看到 `腾讯滑块/阿里滑块/_rand/fuid/fs`：优先按验证码双链与浏览器特性通道处理。
- 浏览器和 Node 出现首个真实分叉：切 Phase 3。
- 差分已归因、准备本地执行：切 Phase 4。
- 链路稳定、准备交付：切 Phase 5。

## MCP 执行纪律

- `chrome-devtools-mcp` 负责“真实页面现场”，没有它就没有可靠浏览器基线。
- `js-reverse` 负责“逆向工作流证据”，没有它就没有稳定的 Hook/代码/状态记录。
- `jshook` 负责“高级逆向与对抗补强”，在重混淆、反调试、Stealth、复杂 Hook 时默认加入。
- 任何阶段如果三者之一没有提供增量证据，要明确说明为什么此阶段暂不需要它，而不是静默跳过。

## 交付物

每轮至少维护这些文件：
- `reverse-records/总览.md`
- `reverse-records/请求链路.md`
- `reverse-records/恢复记录.md`
- `reverse-records/运行时依赖.md`
- `reverse-records/验证记录.md`
- `reverse-records/状态载体卡.md`
- `reverse-records/环境依赖清单.md`
- `reverse-records/升级记录.md`

没有这些工具时，不要长时间继续深挖。

最少要写清：

- 当前 phase
- 当前阻塞
- 当前最强锚点
- 最近一轮新增证据
- 若无新增证据，下一步准备切换到哪一层

## 一句话版本

复杂网站还原统一流程：

`证据门 -> locate -> recover(必要时) -> runtime diff -> env-patch -> sign-replay`

其中：
- 壳重就提前 recover
- 分叉明显就先 runtime diff
- Node 迁移只用 env-patch 收尾
- 最终交付再进 sign-replay
