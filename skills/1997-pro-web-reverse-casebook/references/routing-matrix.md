# 1997.pro 场景路由矩阵

先用这张矩阵判断“像哪类题”，再决定要不要加载别的 reference。

| 线索 / 现象 | 第一判断 | 先读 | 下一 skill | 三 MCP 起手动作 |
|---|---|---|---|---|
| `Akamai`、`Kasada`、`abck`、`sensor_data`、`bm_sz`、`_px3`、`reese84`、`JA4`、`同盾`、`BlackBox`、`a_bogus`、`TLS` | 先分清是 token lane、sign lane、指纹 lane 还是挑战 lane | `risk-fingerprint-and-vendors.md` | `$web-js-reverse-master-flow` / `$jsr-locate` / `$jsr-runtime` | `chrome-devtools-mcp` 看请求与 cookie hop；`js-reverse` 看 storage / challenge / initiator；`jshook` 只在需要重 Hook 或 Stealth 时补上 |
| 浏览器能跑、Node 失败；`window/document/navigator` 缺失；`storage.estimate`、`postMessage`、`animationend`、`run_js` 检测 | 先怀疑隐藏状态载体、跨上下文通道、反自动化与最小环境缺口 | `browser-runtime-and-env.md` | `$jsr-runtime` / `$env-patch` / `$js-reverse-env-antidebug` | 先抓真实浏览器中间态，再看 worker/message/storage/computedStyle/样式终态，不要直接全量补浏览器 |
| `while(true)+switch`、`dispatcher`、`opcode`、`basearr`、`227/226`、`LL()[XX(XX)]()`、大数组解密、`JSVMP`、平坦流 | 这是壳层恢复问题，不是先补环境的问题 | `jsvmp-wasm-and-deobf.md` | `$jsr-recover` / `$js-controlflow-truth-sampling-prune` / `$js-ast-binding-alias-deobf` | 先坐实 sink，再记录 dispatcher 状态迁移、关键 opcode、真假分支 |
| `wasm`、`compileStreaming`、imports/exports、`bx-pp`、worker + wasm、VM 套 wasm | 先找 JS 与 wasm 的桥，不要只盯 `.wasm` 二进制 | `jsvmp-wasm-and-deobf.md` | `$jsr-recover` / `$js-wasm-vmp-ir-lifting` / `$js-webpack-runtime-node-reuse` | 记录 wasm 下载、实例化参数、导入对象、导出函数、调用点 |
| 滑块、旋转验证码、`腾讯滑块`、`阿里滑块`、`_rand`、`fuid`、`fs`、鼠标轨迹 | 先拆“图像 / 位置链”和“加密 / 轨迹 / 验签链” | `captcha-protocol-and-mobile.md` | `$captcha-parameter-chain-bisection` / `$js-reverse-sign-replay` | 先抓 style/log/verify 链路，确认位置字段、轨迹字段、加密字段分别从哪写入 |
| `jspb`、protobuf、`collect`、`rid`、`a_bogus`、`x-s3-s4e`、`desc`、小程序网关 | 先判断是协议封装题、header/签名题还是版本漂移题 | `captcha-protocol-and-mobile.md` | `$jsr-locate` / `$js-reverse-sign-replay` / `$protobuf-schema-backfill` | 找 json -> builder -> encode 边界；比较旧版本纯算与新版本差异；抓 header/body 的统一写入点 |
| App 抓包失败、`QUIC`、So 层、JNI、Hook 定位、Lazada/TikTok | 这是移动端协议 / 网络栈 / Hook 定位题，不要用纯 Web 经验硬套 | `captcha-protocol-and-mobile.md` | `$android-quic-downgrade-hook-capture` 或相关 Android skill | 先 `logcat`、再看网络栈选择、再看 So / JNI 边界与关键参数调用点 |
| 用户只想知道“这类题 1997.pro 有没有写过 / 哪篇最像” | 先用全量索引按题型找最近案例 | `article-index.md` | 保持当前 skill，必要时再 handoff | 先列相似文章，再说明相似点与不确定点 |

补充规则：

- 如果你还没证明真实请求链，不要因为案例相似就直接进入 purecalc。
- 如果你已经明确处于复杂网站还原流程中，优先让 `$web-js-reverse-master-flow` 做阶段判定，本 skill 只补案例经验。
- 如果用户点名某个厂商，但现场线索更像别的家，按现场证据走，不按厂商名走。
