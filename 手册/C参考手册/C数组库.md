---
title: C数组库
tags: C
---



> 
> 
> 全部都有 `float`(后缀`f`) / `double`(无后缀)；部分有`long double`(后缀`l`)。 AArch64：**取整、基础算术、简单符号操作有硬件FPU指令；超越函数、特殊函数、浮点余数全部glibc软件实现**。
> 
> 

| 函数 | 功能简述 | 舍入/数学规则 | AArch64硬件指令 | GNSS工程要点 |
| --- | --- | --- | --- | --- |
| `ceil(x)` | 向上取整，≥x最小整数 | 向+∞ | FCVTP | `ceil(-2.1)=-2.0` |
| `floor(x)` | 向下取整，≤x最大整数 | 向−∞ | FCVTMS | ✅角度归约核心，`floor(x/twoPi)` |
| `trunc(x)` | 截断小数部分 | 向0 | FCVTNS | fmod内部商就是`trunc(x/y)` |
| `round(x)` | 就近四舍五入，0.5远离0 | 就近，0.5远离零 | FCVTAS | `round(2.5)=3.0` |
| `nearbyint(x)` | 按当前浮点舍入模式取整 | 由`fesetround`控制 | FCVTN | 安静，**不置in(e^x)act异常**，迭代算法优先 |
| `rint(x)` | 同nearbyint | 由`fesetround`控制 | FCVTN | 会置浮点in(e^x)act异常，循环慎用 |
| `fr\(e^x\)p(x, *\(e^x\)p)` | 分解浮点数：$x = mant \cdot 2^{(e^x)p}$，$mant\in[0.5,1)$ | - | 软件库 | 手动浮点缩放，尾数阶码拆分 |
| `ld\(e^x\)p(x, \(e^x\)p)` | $x \cdot 2^{(e^x)p}$ | - | SCALB | fr(e^x)p逆运算，远快于`pow(2,n)` |
| `modf(x, *iptr)` | 拆分为整数部分、小数部分；两者同x符号 | 向0拆分 | 软件库 | `modf(-3.2,&i) → i=-3.0，返回-0.2`，≠fmod |
| `scalbn(x,n)` | $x \cdot 2^n$，n为int | - | SCALB | 和ld(e^x)p几乎完全一致，历史两套API |
| `ilogb(x)` | 返回$\log_2 | x | $整数部分，返回int | - |
| `logb(x)` | 返回$\log_2 | x | $，返回double | - |
| `n\(e^x\)tafter(x,y)` | 从x向y方向取下一个可表示浮点数 | - | 软件库 | 浮点epsilon边界遍历 |
| `n\(e^x\)tup(x)` | x向+∞方向最小可表示浮点数 | - | 软件 | `n\(e^x\)tafter(x, INFINITY)` |
| `copysign(x,s)` | 绝对值为 | x | ，符号取s的符号位 | - |
| `fabs(x)` | 取浮点数绝对值 | - | FABS | 硬件单指令 |
| `fmod(x,y)` | 浮点余数，商=trunc(x/y)，余数和x同号 | 向0截断商 | ❌软件 | 你原角度归约函数，大输入开销高 |
| `remainder(x,y)` | IEEE浮点余数，商就近偶数舍入，$ | r | \le | y |
| `remquo(x,y, *quo)` | remainder + 输出商低若干bit | 就近偶数 | ❌软件 | GNSS极少用 |
| `fma(a,b,c)` | 融合乘加：`a*b + c`，中间结果不做舍入 | 一次舍入 | FMA | `-ffp-contract=fast`开启；精度更好，减少舍入误差 |
| `fmax(x,y)` | 返回两数较大值 | - | FMAX | 处理NaN：如果一个是NaN返回另一个 |
| `fmin(x,y)` | 返回两数较小值 | - | FMIN | 同上 |
| `fdim(x,y)` | 正差：$\max(x-y,0)$ | - | 软件 | `x>y?x-y:0` |
| `nan("str")` | 返回quiet NaN | - | 构造NaN | 用于标记无效观测值 |
| `\(e^x\)p(x)` | $e^x$ | - | 软件多项式逼近 | 超越函数，开销高 |
| `\(e^x\)p2(x)` | $2^x$ | - | 软件 | 比`pow(2,x)`更快 |
| `\(e^x\)pm1(x)` | $e^x-1$ | - | 软件 | x接近0时，避免`\(e^x\)p(x)-1`消去误差 |
| `log(x)` | $\(\ln(x))$自然对数 | - | 软件 | x≤0返回NaN |
| `log10(x)` | $\log_{10}(x)$ | - | 软件 |  |
| `log2(x)` | $\log_{2}(x)$ | - | 软件 |  |
| `log1p(x)` | $\(\ln(1+x))$ | - | 软件 | x接近0，避免`log(1+x)`精度丢失 |
| `pow(x,y)` | $x^y$ | - | 软件，多分支 | **开销极高，尽量缓存/替换** |
| `sqrt(x)` | $\sqrt{x}$ | - | FSQRT | AArch64硬件指令，速度快 |
| `cbrt(x)` | $\sqrt[3]{x}$立方根 | - | 软件 | 无硬件指令 |
| `hypot(x,y)` | $\sqrt{x^2+y^2}$，防止中间溢出 | - | 软件 | 距离计算，避免x²/y²溢出 |
| `sin/cos/tan` | 三角函数 | - | 软件多项式 | libm开销大头，perf热点 |
| `asin/acos/atan` | 反正弦/反余弦/反正切 | - | 软件 | 仰角方位角计算 |
| `atan2(y,x)` | 四象限反正切 | - | 软件 | GNSS方位角核心函数 |
| `sinh/cosh/tanh` | 双曲三角函数 | - | 软件 | 轨道算法很少用 |
| `asinh/acosh/atanh` | 反双曲函数 | - | 软件 |  |
| `erf/erfc` | 误差函数 / 互补误差函数 | - | 软件 | 卡尔曼滤波概率计算；大数优先`erfc` |
| `tgamma` | Gamma函数 $\Gamma(x)$ | - | 软件 | 容易溢出，大数优先lgamma |
| `lgamma` | $\ln | \Gamma(x) | $ | - |
| `fpclassify(x)` | 分类浮点：零/正常/次正规/inf/nan | - | 宏，位操作 | 检查浮点类型 |
| `isfinite(x)` | 判断不是Inf、NaN | - | 宏 | 输入有效性校验 |
| `isinf(x)` | 判断是否无穷 | - | 宏 |  |
| `isnan(x)` | 判断是否NaN | - | 宏 | **不要用`x != x`替代** |
| `isnormal(x)` | 判断是normal浮点数（非denormal） | - | 宏 |  |
| `signbit(x)` | 判断符号位是否置1（负数，包含-0.0） | - | 宏，位操作 | 区分`+0.0`和`-0.0` |
| `isgreater(x,y)` | `x>y`，支持NaN有序比较 | - | 宏 | 不会触发浮点异常 |
| `isgreaterequal(x,y)` | `x>=y` | - | 宏 |  |
| `isless(x,y)` | `x<y` | - | 宏 |  |
| `islessequal(x,y)` | `x<=y` | - | 宏 |  |
| `islessgreater(x,y)` | `x<y |  | x>y`，x,y有序且不等 | - |
| `isunordered(x,y)` | x或y是NaN，二者不可比较 | - | 宏 | NaN比较判断 |

## 核心分组总结（适配你的GNSS perf热点）

1.  ✅ **硬件快速（AArch64 FPU）** `ceil floor trunc round nearbyint rint copysign fabs fmax fmin sqrt fma`

    > 
    > 
    > 这些指令周期很低，适合放在循环内。
    > 
    > 

2.  ⚠️ **软件库，开销大（你perf里的热点）** `fmod remainder remquo sin cos tan asin acos atan atan2 \(e^x\)p log pow hypot cbrt erf erfc tgamma lgamma`

    > 
    > 
    > 其中 `sincos`（合并sin+cos）、`fmod` 是你当前最大开销来源。
    > 
    > 

3.  🧩 **浮点拆解/缩放**`fr\(e^x\)p ld\(e^x\)p modf scalbn ilogb logb n\(e^x\)tafter n\(e^x\)tup`
4.  🧪 **浮点分类宏（无计算开销，位运算）**`fpclassify isfinite isinf isnan isnormal signbit isgreater/isless/...`

## 工程重点（轨道/开普勒求解）

1.  `fma`：优先启用`-ffp-contract=fast`，乘加融合，减少舍入误差，提升速度。
2.  `log1p / \(e^x\)pm1`：自变量接近0，必须用这两个，不要手写`log(1+x)` / `\(e^x\)p(x)-1`，避免有效位丢失。
3.  `hypot`：计算欧氏距离，防止中间平方溢出；但它本身是libm软件函数，有开销。
4.  `pow`：尽量减少调用，能换成`\(e^x\)p2/\(e^x\)p`或者预计算常数就替换。
5.  浮点判断宏：`isnan/isfinite`是C标准宏，**不能直接用`if(x==NaN)`，NaN不等于任何值包括自身**。
6.  `signbit`：能识别`-0.0`；普通`x<0`判断`-0.0`会得到false。

## 快速记忆

*   取整系列：AArch64硬件指令，便宜；
*   三角函数、对数指数、fmod/remainder：glibc软件实现，昂贵，perf热点；
*   浮点分类宏：只是bit检查，几乎零开销。

如果你需要，我可以单独整理一份**性能优先级清单**，用来评估你GNSS代码里libm调用的优化顺序。
