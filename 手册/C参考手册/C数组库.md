---
title: C数组库
tags: C
---




> 全部都有 `float`(后缀`f`) / `double`(无后缀)；部分有`long double`(后缀`l`)。 


[TOC]

# 取整函数

| 函数 | 数学规则 | 示例 | AArch64 指令 |
| --- | --- | --- | --- |
| `ceil(x)` | 向 +∞ 取整 | `ceil(-2.1)=-2.0` | FCVTP |
| `floor(x)` | 向 −∞ 取整 | `floor(-2.1)=-3.0` | FCVTMS |
| `trunc(x)` | 向 0 截断 | `trunc(-2.1)=-2.0` | FCVTNS |
| `round(x)` | 就近舍入，0.5 远离 0 | `round(2.5)=3.0` | FCVTAS |
| `nearbyint(x)` | 按当前 FP 舍入模式取整，**不抛 inexact** | 受 `fesetround()` 控制 | FCVTN |
| `rint(x)` | 同 nearbyint，但**会置 inexact 异常** | 受 `fesetround()` 控制 | FCVTN |

# 浮点分解/缩放

| 函数            | 功能                             | 备注                                       |
| --------------- | -------------------------------- | ------------------------------------------ |
| `frexp(x,*exp)` | 拆成 `mant·2^exp`，mant∈[0.5,1) | 软件库                                     |
| `ldexp(x,exp)`  | `x·2^exp`，frexp 逆运算          | SCALB，远快于 `pow(2,n)`                   |
| `modf(x,*iptr)` | 拆整数部分+小数部分，符号同 x    | `modf(-3.2,&i)→i=-3.0, 返回-0.2`，≠ fmod |
| `scalbn(x,n)`   | `x·2^n`，n 为 int                | 与 ldexp 等价，历史两套 API                |
| `ilogb(x)`      | 返回 ⌊log₂                     | x                                          |
| `logb(x)`       | 返回 log₂                       | x                                          |

# 相邻浮点数/符号操作

| 函数 | 功能 | 备注 |
| --- | --- | --- |
| `nextafter(x,y)` | x 往 y 方向下一个可表示浮点数 | 软件库 |
| `nextup(x)` | x 向 +∞ 方向最小浮点数 | = `nextafter(x,INF)` |
| `copysign(x,s)` | 取 |x|，符号取 s 的符号位 | 硬件符号位操作，替代 `if(x<0)` |
| `fabs(x)` | 绝对值 | FABS，硬件单指令 |
| `signbit(x)` | 符号位是否为 1（含 `-0.0`） | 宏，bit 操作；`x<0` 对 `-0.0` 为 false |

# 浮点余数

| 函数 | 商取整规则 | 余数范围 | 用途 |
| --- | --- | --- | --- |
| `fmod(x,y)` | `trunc(x/y)` 向 0 | (−|y|, |y|)，符号同 x | 你原角度归约用的就是它 |
| `remainder(x,y)` | 就近偶数舍入 | [−|y|/2, |y|/2] | IEEE 余数，不适合 0~2π 折叠 |
| `remquo(x,y,*q)` | remainder + 输出商低位 | 同上 | GNSS 极少用 |

# 基础算术

| 函数 | 功能 | 备注 |
| --- | --- | --- |
| `fma(a,b,c)` | `a*b+c` 一次舍入 | FMA 硬件，需 `-ffp-contract=fast` |
| `fmax(x,y)` | 取大（NaN 不传染） | FMAX |
| `fmin(x,y)` | 取小（NaN 不传染） | FMIN |
| `fdim(x,y)` | `max(x-y,0)` | 软件 |
| `nan("str")` | 构造 quiet NaN | 标记无效观测值 |

# 6 指数/对数

| 函数       | 功能       | 坑                           |
| ---------- | ---------- | ---------------------------- |
| `exp(x)`   | eˣ        | 多项式逼近                   |
| `exp2(x)`  | 2ˣ        | 比 `pow(2,x)` 快             |
| `expm1(x)` | eˣ−1     | x→0 必用它，别写 `exp(x)-1` |
| `log(x)`   | ln(x)      | x≤0 为 NaN                  |
| `log10(x)` | log₁₀(x) |                              |
| `log2(x)`  | log₂(x)   |                              |
| `log1p(x)` | ln(1+x)    | x→0 必用它，别写 `log(1+x)` |
| `pow(x,y)` | xʸ        | 最贵，多分支，尽量缓存       |

# 幂/根

| 函数 | 功能 | AArch64 |
| --- | --- | --- |
| `sqrt(x)` | √x | FSQRT 硬件，快 |
| `cbrt(x)` | ³√x | 软件 |
| `hypot(x,y)` | √(x²+y²)，防中间溢出 | 软件 |

# 三角函数

| 函数 | 功能 |
| --- | --- |
| `sin/cos/tan` | 正弦/余弦/正切 |
| `asin/acos/atan` | 反三角 |
| `atan2(y,x)` | 四象限反正切，方位角核心 |
| `sinh/cosh/tanh` | 双曲 |
| `asinh/acosh/atanh` | 反双曲 |

> 优化提示：优先用 `sincos` 一次算出 sin+cos，别分开调 `sin()+cos()`。


# 特殊函数

| 函数 | 功能 | 坑 |
| --- | --- | --- |
| `erf(x)` | 误差函数，值域 [−1,1] |  |
| `erfc(x)` | 互补误差 1−erf | x 大时用它，别写 `1-erf(x)` |
| `tgamma(x)` | Γ(x)，Γ(n)=(n−1)! | 易溢出，大数用 lgamma |
| `lgamma(x)` | ln|Γ(x)| | 多线程用 `lgamma_r(x,&sign)`，别用全局 `signgam` |

# 浮点分类/比较宏

| 宏 | 判断 |
| --- | --- |
| `fpclassify(x)` | 返回 FP_NAN/INFINITE/ZERO/NORMAL/SUBNORMAL |
| `isfinite(x)` | 非 Inf 非 NaN |
| `isinf(x)` | 是否无穷 |
| `isnan(x)` | 是否 NaN（**别用 `x!=x` 替代**） |
| `isnormal(x)` | 是否 normal（非 denormal） |
| `isgreater/isgreaterequal/isless/islessequal` | 有序比较，遇 NaN 不抛异常 |
| `islessgreater(x,y)` | x、y 有序且不等 |
| `isunordered(x,y)` | 任一为 NaN，不可比较 |

* * *

# 一句话分级

- **硬件便宜，随便用**：`ceil floor trunc round nearbyint rint fabs copysign fmax fmin sqrt fma`
- **glibc 软件，昂贵，重点优化**：`fmod remainder sin cos tan asin acos atan atan2 exp log pow hypot`
- **分类宏，零成本**：`isnan isfinite signbit` 等全部

需要的话，我可以再单独出一张**「你 perf 里出现过的函数 → 优化手段」对照表**（sincosf32x / fmod / sqrt / asin / atan2 / log10 逐个对应怎么改）。
