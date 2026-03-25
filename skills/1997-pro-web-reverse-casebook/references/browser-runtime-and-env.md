# 浏览器通道 / 运行时 / 补环境

主要对应文章：

- `利用浏览器特性进行数据加密与传输的N种方式`
- `从阿里滑块_rand参数谈基于CSS动画特性的参数传递机制`
- `DrissonPage替换js或指定请求的响应内容`
- `DrissionPage检测点与修改DP源码通过所有run_js检测`
- `同盾环境分析与补环境`
- `TLS&&补环境理解`

## 先问“值从哪条浏览器通道走”

`1997.pro` 的一条核心经验是：

- 很多值不是直接在同一函数里算出来的
- 而是被浏览器原生能力搬运、隐藏、延迟读取

高频通道包括：

- `localStorage` / `sessionStorage` / `cookie`
- `IndexedDB` / `Cache API`
- `iframe` / `worker` / `postMessage`
- `MessageChannel` / `BroadcastChannel`
- DOM 属性、`img.src`
- `canvas` / SVG / CSS 动画

因此，浏览器能跑、本地跑不通时，不要只列缺失对象；先证明：

- 是否有跨上下文消息
- 是否有上一次会话残留状态
- 是否有动画结束、样式计算、异步能力检测这类生命周期依赖

## 最小补环境的真正入口

同盾相关文章的高价值经验不是“补了哪些字段”，而是：

- 先断到环境数组最终采样点
- 看哪些检测值真的进入最终数组
- 再补缺失项

这比一开始就铺满 `window/document/navigator` 更稳。

特别注意：

- `navigator.storage.estimate()` 这种异步能力接口
- 时间相关值
- 代码格式化 / 自动化痕迹检测

如果这些值被最终数组消费，才值得补；否则只是噪音。

## CSS 动画与样式链是独立运行时

阿里 `_rand` 一文说明：

- 参数可以被编码进 CSS 规则筛选与动画最终样式
- 关键触发点可能是 `animationend`
- 真正的“中间态”不是 JS 变量，而是计算后的 `className` / `color` / `opacity` / `border` 等样式结果

实战含义：

- 只在 Node 里抠 JS，往往拿不到这条链
- 若要本地还原，必须把规则过滤、关键帧执行、样式采样逻辑一起迁走
- 更稳的做法通常是先在真实浏览器里把这条样式链抓明白

## 自动化对抗要做定向实验

DrissionPage 两篇文章给出的经验：

- 优先用 CDP 做“替换 JS / 替换响应 / 局部改检测点”的定向实验
- 不要一上来改一大坨环境，先验证某个检测点是否真影响结果

推荐实验顺序：

1. 用 `chrome-devtools-mcp` 替换或拦截可疑脚本 / 响应
2. 对照正常态与修改态的分支差异
3. 只在确定检测点后，才做自动化工具补丁或源码 patch

## 浏览器与 Node 分叉的诊断顺序

遵循下面的顺序：

1. 浏览器里固定一份正常样本
2. 比较 `storage/cookie/channel/style` 中间态
3. 比较时间源、随机源、异步返回值
4. 最后才比较最终参数

不要反过来。

## 三个优先 probe

- 看 `storage`、`cookie`、`IndexedDB`、`Cache API` 是否真的被读取
- 看 `worker/message/postMessage` 是否承载关键状态
- 看 `computedStyle`、事件回调、能力检测是否决定了最终分支

## 推荐 handoff

- 先诊断分叉：`$jsr-runtime`
- 已知要最小迁移：`$env-patch`
- 已经能跑但中间态漂移：`$js-runtime-diff-env-patching`
- 明显存在 DevTools / 自动化摩擦：`$js-reverse-env-antidebug`
- 复杂站总控：`$web-js-reverse-master-flow`
