# JSVMP / 平坦流 / WASM / 反混淆

主要对应文章：

- `akamai3反混淆`
- `225fireyejs三层转一层`
- `从227谈控制流平坦化的还原(一)`
- `从227谈控制流平坦化的还原(二)`
- `实现一个伪asm的jsvmp`
- `vmp简单实现以及快速sig3源码解析`
- `腾讯滑块VMP反编译(一)`
- `腾讯滑块VMP反编译(二)`
- `akamai反混淆思路`
- `kasada代码分析&&插桩点&&反编译`
- `同盾AST思路与插件`
- `PerimeterX盾-PX3按压反混淆&&逆向分析`
- `wasm加载流程与逆向实战`
- `227最新请求头bx-pp的wasm调用`

## 总原则

这批文章反复说明一件事：

- 壳层恢复必须围绕已证明的目标边界做最小切片
- 不是看到 `while + switch` 或 `JSVMP` 就直接全量反编译

优先级应该是：

1. 先证明 sink / builder / 写回边界
2. 再恢复与它直接相关的 dispatcher、opcode、状态载体
3. 只有在下游必须重放多条路径时，才继续扩深

## 227 / 平坦流恢复

从 `227` 两篇可以抽出这条顺序：

1. 先剔除虚假条件
2. 再把节点理解成状态机里的 `case num`
3. 先恢复顺序执行、再恢复分支、最后恢复循环

特别注意：

- 假条件的错分支不一定是“死代码”，它可能在别的路径上是真分支
- `while break` / `while continue` 需要显式构造对应循环节点，不能只展开为顺序语句

## 腾讯 VMP 的关键经验

腾讯两篇的价值，在于把“解释执行”改造成“生成 AST”：

- 先记录执行过的指令与堆栈行为
- 把关键 opcode 映射成 AST 节点
- 给 VM 增加 `funcBody` 之类的承载数组，让不同函数的 block 可以被回填
- 为不同函数保存各自的常量、参数、起止 IP、判断条件

这使得：

- 函数节点可以被还原
- `if/else` 可以按 true/false 分支分别填充 block
- 交汇点可以通过执行路径与 `g/ip` 变化来判断

这套思路比“单纯打印日志”更进一步，适合需要最小反编译的 VM。

## Akamai / Kasada 类反混淆

高价值经验：

- 动态字符串解密函数要在“全部生成完成后”再调用
- 断点优先放在最终打包 / 最终加密附近，再向上一个栈取原始入参
- 脚本版本轮换快时，先固定一份样本再做替换

因此：

- 先提取运行中的字符串解密映射
- 再做 AST 替换回原 JS
- 不要在半初始化状态就尝试静态还原全部字符串

## 伪 asm JSVMP 与 Wasm 的桥接认识

`实现一个伪asm的jsvmp` 与 `wasm加载流程与逆向实战` 两篇给出一个重要视角：

- 不要只盯 VM 字节码或 `.wasm`
- 真正关键的是“导入对象 / 外部环境函数 / JS 与 VM 的桥”

实战里优先记录：

- wasm 从哪下载
- 是 `compileStreaming` 还是 `instantiate`
- 传入了哪些 imports
- 哪个 JS 函数把浏览器环境塞给了 VM
- 哪个 export 最终回写目标字段

如果 VM 套了 wasm，也先抓桥，不要先做全量 lifting。

## 三个优先 probe

- 记录 dispatcher 状态变量、`ip/g` 变化、关键分支条件
- 记录 wasm 下载、实例化、imports/exports、调用点
- 记录字符串解密函数何时才真正可用

## 推荐 handoff

- 壳层总控：`$jsr-recover`
- 控制流真假分支裁剪：`$js-controlflow-truth-sampling-prune`
- 字符串池 / 多层别名：`$js-ast-binding-alias-deobf`
- Wasm VM：`$js-wasm-vmp-ir-lifting`
- 打包运行时复用：`$js-webpack-runtime-node-reuse`
