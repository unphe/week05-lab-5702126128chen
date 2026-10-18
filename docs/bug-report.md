# 故障排查报告：单摆静止不动（ReferenceError）

> 任务四「闭环自愈 —— 控制台报错捕获与实操演练」与任务五「四段式自愈 Prompt」的记录。

## 一、四段式自愈 Prompt（任务五标准格式）

```
[故障现象]：打开 index.html 后，摆球静止不动，浏览器控制台输出红色 ReferenceError 报错。
[复现路径]：1. 打开 index.html；2. 按 F12 / ⌘⌥C 打开控制台；3. 刷新页面（⌘R）。
[预期结果]：单摆应从初始角度开始摆动，HUD 上的角度与角速度持续更新。
[环境信息]：Safari（WebKit）/ macOS；报错内容 ReferenceError: Can't find variable: LLm。
```

## 二、错误原因

单摆角加速度微分方程的实现中，摆长换算函数 `Lm()` 被误写为 `LLm`（字母 `L` 重复了一次）：

```
α = −(g / L) · sin(θ) − k · ω
```

- 错误位置：`index.html` 第 399 行，`angularAccel()` 函数内。
- 脚本中仅定义了 `const Lm = () => P.Lpx / PPM;`（第 388 行），`LLm` 不存在任何定义。
- JavaScript 运行时对该标识符抛出 `ReferenceError`，属于「未定义变量」类运行时错误。

## 三、影响

`angularAccel()` 是物理积分的核心：动画循环每帧调用 `integrate()` → `angularAccel()`。
该异常导致每帧积分中断、后续 `requestAnimationFrame` 不再调度，表现为**摆球完全静止**，但页面 UI 面板正常渲染——正是「画面有、动画死」的典型幻觉/笔误类故障。

## 四、修复方案

将未定义的 `LLm()` 恢复为正确的 `Lm()`：

```diff
   function angularAccel(theta, omega) {
-    return -(P.g / LLm()) * Math.sin(theta) - P.k * omega;
+    return -(P.g / Lm()) * Math.sin(theta) - P.k * omega;
   }
```

修复后经语法检查通过（`new Function` 校验脚本 20496 字符），且 `LLm` 残留计数为 0。

## 五、提交记录（证据链）

| 提交 | 说明 |
|---|---|
| `16fbbab` | experiment: inject typo (Lm -> LLm) for console-error debugging drill（故意引入） |
| `9126e04` | fix: resolve ReferenceError LLm -> Lm in angularAccel（定位修复） |

助教可用 `git diff 16fbbab 9126e04 -- index.html` 查看从引入到修复的完整过程。
