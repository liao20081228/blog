---
title: C数学库
tags: C
---




> 全部都有 `float`(后缀`f`) / `double`(无后缀)；部分有`long double`(后缀`l`)。 


[TOC]

以下按作用类型分类整理，包含功能详述、数学公式、示例、ARM64与x86实现方式（指令或libm）及大致CPU周期、单指令具体指令名、注意事项。指令周期数据主要来源于Arm官方优化指南及uops.info等公开基准测试，具体数值因微架构（如Cortex-A76、Neoverse N1、Intel Skylake等）而异，实际性能请以目标平台实测为准。

---

## 性能速查

| 类别 | 代表函数 | ARM64 | x86 | 优化建议 |
|------|---------|-------|-----|---------|
| 硬件单指令 | `fabs` `fma` `fmax` `fmin` `sqrt` | 2–12 cycle | 1–19 cycle | 直接用 |
| 取整 | `floor` `ceil` `trunc` `rint` | `FRINT*`，延迟 2 / 吞吐 2 | SSE4.1 `ROUND*`，延迟 ~3–6 | 用硬件指令 |
| 简单运算 | `copysign` `fdim` `nextafter` | 1–3 cycle | 1–3 cycle | 直接用 |
| 中等 | `exp` `log` `sin` `cos` | libm，20–50 cycle | libm，20–50 cycle | 批量用多项式近似 + NEON/SSE |
| 昂贵 | `pow` `atan2` `asin` `acos` | libm，50–100 cycle | libm，50–100 cycle | 避免热路径 |
| 极贵 | `fmod` `remainder` `tgamma` | libm，80–200 cycle | libm，80–200 cycle | 用 `floor`+`fma` 替代 |

---

## 1. 绝对值与符号

| 函数 | 功能 | 数学公式 | 示例 | ARM64 实现 / 周期 | x86 实现 / 周期 | 注意事项 |
|------|------|---------|------|------------------|----------------|---------|
| `fabs(x)` | 返回浮点数 x 的绝对值，去掉符号位。对 `-0.0` 返回 `+0.0`，对 `NaN` 返回 `NaN`，对 `±Inf` 返回 `+Inf` | \|x\| | `fabs(-3.5) = 3.5` | **硬件指令** `FABS`，延迟 2 / 吞吐 2 | **硬件指令** `ANDPS`/`ANDPD`（位与掩码），延迟 ~1 / 吞吐 ~0.5 | 仅用于浮点；整数用 `abs`/`labs`/`llabs`。x86 无专用浮点绝对值指令，用位与实现 |
| `copysign(x, y)` | 返回一个数，大小等于 x，符号等于 y。不受 x 原符号影响，正确处理 `±0.0`、`NaN`、`Inf` | \|x\|·sign(y) | `copysign(3, -1) = -3` | **硬件指令** 位操作组合（`FABS`+`FNEG`+`FCSEL`），延迟 ~2 | **硬件指令** 位操作（`ANDPS`/`ORPS`），延迟 ~1 | 不是简单相乘，编译器通常内联为几条基础指令。`-0.0` 的符号会被保留 |
| `signbit(x)` | 判断 x 的符号位是否为负，返回非零表示负。与 `x < 0` 不同，能区分 `-0.0` 和 `+0.0` | — | `signbit(-0.0) = 1` | **硬件指令** 位提取，延迟 ~1 | **硬件指令** 位操作（移位/与），延迟 ~1 | 不触发浮点异常 |

---

## 2. 取模与余数

| 函数 | 功能 | 数学公式 | 示例 | ARM64 实现 / 周期 | x86 实现 / 周期 | 注意事项 |
|------|------|---------|------|------------------|----------------|---------|
| `fmod(x, y)` | 计算 x 除以 y 的浮点余数，结果符号与 x 相同，大小小于 \|y\|。内部用 `trunc` 取整 | x - trunc(x/y)·y | `fmod(5.3, 2) = 1.3` | **libm**，~80–200 cycle | **libm**，~80–200 cycle | 两者均含除法+分支；ARM64 无浮点取模指令；x86 的 x87 `FPREM` 已淘汰；热路径避免 |
| `remainder(x, y)` | 计算 IEEE 754 定义的余数，结果符号可与 x 不同，用就近取整（round-to-nearest） | x - round(x/y)·y | `remainder(5.3, 2) = -0.7` | **libm**，~100–250 cycle | **libm**，~100–250 cycle | 比 `fmod` 更慢。结果在 `[-|y|/2, |y|/2]` 内 |
| `remquo(x, y, &q)` | 同时返回 `remainder(x,y)` 和商的低位（至少低 3 位） | 同 `remainder`，q 存商 | `remquo(5.3, 2, &q)` → q=3 | **libm**，~120–250 cycle | **libm**，~120–250 cycle | q 只保证低几位正确，不能当完整商用 |

---

## 3. 融合乘加

| 函数 | 功能 | 数学公式 | 示例 | ARM64 实现 / 周期 | x86 实现 / 周期 | 注意事项 |
|------|------|---------|------|------------------|----------------|---------|
| `fma(a, b, c)` | 计算 `a*b + c`，中间乘积不截断，以无限精度计算后只舍入一次。比 `a*b+c` 精度更高 | a·b + c | `fma(2, 3, 4) = 10` | **硬件指令** `FMADD`/`FMSUB`/`FNMADD`/`FNMSUB`，延迟 4（late-forwarding 后有效 2）/ 吞吐 2 | **硬件指令**（需 FMA3/FMA4）`VFMADD`/`VFMSUB` 系列，延迟 ~4 / 吞吐 ~2 | ARM64 基础指令，无需扩展；x86 需 CPU 支持 FMA3，否则回退 libm 软件实现 |

---

## 4. 最值与差值

| 函数 | 功能 | 数学公式 | 示例 | ARM64 实现 / 周期 | x86 实现 / 周期 | 注意事项 |
|------|------|---------|------|------------------|----------------|---------|
| `fmax(x, y)` | 返回 x、y 中较大者。若一个为 `NaN`，返回另一个（非 NaN） | max(x, y) | `fmax(3, 5) = 5` | **硬件指令** `FMAX`/`FMAXNM`，延迟 2 / 吞吐 2 | **硬件指令** `MAXSS`/`MAXSD`/`MAXPS`/`MAXPD`，延迟 ~2 / 吞吐 ~1 | **x86 NaN 语义与 C 标准不一致**：仅一个源为 NaN 时，返回第二个操作数（`src2`），需编译器额外处理 |
| `fmin(x, y)` | 返回 x、y 中较小者。NaN 处理同 `fmax` | min(x, y) | `fmin(3, 5) = 3` | **硬件指令** `FMIN`/`FMINNM`，延迟 2 / 吞吐 2 | **硬件指令** `MINSS`/`MINSD`/`MINPS`/`MINPD`，延迟 ~2 / 吞吐 ~1 | NaN 语义同 `fmax`，x86 需编译器修正 |
| `fdim(x, y)` | 返回 x - y 的正部分，即 x > y 时返回 x-y，否则返回 `+0.0` | max(x-y, 0) | `fdim(5, 3) = 2` | **硬件指令** `FMAX` + `FSUB` 组合，~4 cycle | **硬件指令** `MAXSS` + `SUBSS` 组合，~4 cycle | 结果是 `+0.0` 而非 `-0.0` |

---

## 5. 指数与对数

| 函数 | 功能 | 数学公式 | 示例 | ARM64 实现 / 周期 | x86 实现 / 周期 | 注意事项 |
|------|------|---------|------|------------------|----------------|---------|
| `exp(x)` | 计算 e 的 x 次幂。e ≈ 2.718281828... | eˣ | `exp(1) = 2.718...` | **libm**，~20–50 cycle | **libm**（glibc AVX 优化版），~20–50 cycle | x 大时溢出返回 `+Inf`，x 极负时下溢返回 `+0.0`；glibc 2.29 起 ARM64 有向量化实现 |
| `exp2(x)` | 计算 2 的 x 次幂 | 2ˣ | `exp2(3) = 8` | **libm**，~20–50 cycle | **libm**，~20–50 cycle | 比 `pow(2,x)` 快得多 |
| `expm1(x)` | 计算 eˣ - 1，在 x 接近 0 时比 `exp(x) - 1` 精度高 | eˣ - 1 | `expm1(1e-10) ≈ 1e-10` | **libm**，~20–50 cycle | **libm**，~20–50 cycle | 避免 `exp(x)-1` 在小 x 时丢失有效位 |
| `log(x)` | 计算自然对数 ln(x)，x > 0 | ln x | `log(e) = 1` | **libm**，~20–50 cycle | **libm**（glibc AVX），~20–50 cycle | x ≤ 0 返回 `NaN`/-Inf；ARM64 有向量化实现 |
| `log10(x)` | 计算以 10 为底的对数 | log₁₀ x | `log10(100) = 2` | **libm**，~20–50 cycle | **libm**（glibc AVX），~20–50 cycle | 卫星 CNR 计算常用；比 `log(x)/log(10)` 快且准 |
| `log2(x)` | 计算以 2 为底的对数 | log₂ x | `log2(8) = 3` | **libm**，~20–50 cycle | **libm**，~20–50 cycle | 比 `log(x)/log(2)` 快 |
| `log1p(x)` | 计算 ln(1+x)，在 x 接近 0 时比 `log(1+x)` 精度高 | ln(1+x) | `log1p(1e-10) ≈ 1e-10` | **libm**，~20–50 cycle | **libm**，~20–50 cycle | 避免 `1+x` 在小 x 时丢失精度 |

---

## 6. 幂与根

| 函数 | 功能 | 数学公式 | 示例 | ARM64 实现 / 周期 | x86 实现 / 周期 | 注意事项 |
|------|------|---------|------|------------------|----------------|---------|
| `pow(x, y)` | 计算 x 的 y 次幂，支持任意实数指数 | xʸ | `pow(2, 10) = 1024` | **libm**，~50–100 cycle | **libm**（glibc AVX），~50–100 cycle | 非常慢；整数幂用连乘或 `exp2`/`log2` 组合 |
| `sqrt(x)` | 计算平方根，x ≥ 0 | √x | `sqrt(16) = 4` | **硬件指令** `FSQRT`，H-form 延迟 5 / S-form 延迟 8 / D-form 延迟 12，吞吐均为 1 | **硬件指令** `SQRTSD`/`SQRTSS`，延迟 ~13–19（Skylake） | ARM64 延迟**随输入变化**（单精度 7–17，双精度 7–32）。确保 `-fno-math-errno` |
| `cbrt(x)` | 计算立方根，支持负数 | ∛x | `cbrt(27) = 3` | **libm**，~50–100 cycle | **libm**，~50–100 cycle | 比 `pow(x, 1.0/3)` 快且精度高；负数输入返回负根 |
| `hypot(x, y)` | 计算欧几里得范数 √(x²+y²)，避免中间溢出/下溢 | √(x²+y²) | `hypot(3, 4) = 5` | **libm**，~50–100 cycle | **libm**，~50–100 cycle | 直接 `sqrt(x*x+y*y)` 在 x、y 很大时会溢出 |

---

## 7. 三角函数

| 函数 | 功能 | 数学公式 | 示例 | ARM64 实现 / 周期 | x86 实现 / 周期 | 注意事项 |
|------|------|---------|------|------------------|----------------|---------|
| `sin(x)` | 计算正弦，x 为弧度 | sin x | `sin(π/2) = 1` | **libm**，~20–50 cycle | **libm**（x87 `FSIN` 已淘汰，用 SSE/AVX），~20–50 cycle | 范围缩减是主要开销；ARM64 有向量化实现 |
| `cos(x)` | 计算余弦，x 为弧度 | cos x | `cos(0) = 1` | **libm**，~20–50 cycle | **libm**，~20–50 cycle | 与 sin 合用 `sincos` 省一次范围缩减 |
| `tan(x)` | 计算正切，x 为弧度 | sin/cos | `tan(π/4) = 1` | **libm**，~20–50 cycle | **libm**，~20–50 cycle | x 接近 π/2 + kπ 时结果趋于 ±Inf |
| `asin(x)` | 计算反正弦，返回 `[-π/2, π/2]` 弧度 | arcsin x | `asin(1) = π/2` | **libm**，~50–100 cycle | **libm**，~50–100 cycle | \|x\| > 1 返回 `NaN` |
| `acos(x)` | 计算反余弦，返回 `[0, π]` 弧度 | arccos x | `acos(1) = 0` | **libm**，~50–100 cycle | **libm**，~50–100 cycle | \|x\| > 1 返回 `NaN` |
| `atan(x)` | 计算反正切，返回 `(-π/2, π/2)` 弧度 | arctan x | `atan(1) = π/4` | **libm**，~50–100 cycle | **libm**，~50–100 cycle | 全域定义 |
| `atan2(y, x)` | 计算四象限反正切，根据 (x, y) 符号确定象限，返回 `(-π, π]` | arctan(y/x) + 象限 | `atan2(1, 1) = π/4` | **libm**，~50–100 cycle | **libm**，~50–100 cycle | 处理 x=0；比 `atan` 慢 |

---

## 8. 双曲函数

| 函数 | 功能 | 数学公式 | 示例 | ARM64 实现 / 周期 | x86 实现 / 周期 | 注意事项 |
|------|------|---------|------|------------------|----------------|---------|
| `sinh(x)` | 计算双曲正弦 | (eˣ-e⁻ˣ)/2 | `sinh(1) ≈ 1.175` | **libm**，~50–100 cycle | **libm**，~50–100 cycle | 大 \|x\| 时溢出；奇函数 |
| `cosh(x)` | 计算双曲余弦 | (eˣ+e⁻ˣ)/2 | `cosh(0) = 1` | **libm**，~50–100 cycle | **libm**，~50–100 cycle | 大 \|x\| 时溢出；偶函数 |
| `tanh(x)` | 计算双曲正切 | sinh/cosh | `tanh(0) = 0` | **libm**，~50–100 cycle | **libm**，~50–100 cycle | 大 \|x\| 时趋近 ±1，不溢出 |
| `asinh(x)` | 计算反双曲正弦 | ln(x+√(x²+1)) | `asinh(0) = 0` | **libm**，~50–100 cycle | **libm**，~50–100 cycle | 全域定义 |
| `acosh(x)` | 计算反双曲余弦 | ln(x+√(x²-1)) | `acosh(1) = 0` | **libm**，~50–100 cycle | **libm**，~50–100 cycle | x < 1 返回 `NaN` |
| `atanh(x)` | 计算反双曲正切 | ½ln((1+x)/(1-x)) | `atanh(0) = 0` | **libm**，~50–100 cycle | **libm**，~50–100 cycle | \|x\| ≥ 1 返回 `NaN`/`±Inf` |

---

## 9. 特殊函数

| 函数 | 功能 | 数学公式 | 示例 | ARM64 实现 / 周期 | x86 实现 / 周期 | 注意事项 |
|------|------|---------|------|------------------|----------------|---------|
| `erf(x)` | 误差函数，概率论、统计核心函数 | (2/√π)∫₀ˣ e⁻ᵗ² dt | `erf(0) = 0` | **libm**，~50–100 cycle | **libm**，~50–100 cycle | 奇函数 |
| `erfc(x)` | 互补误差函数 1 - erf(x)，大 x 时比 `1-erf(x)` 精度高 | 1 - erf(x) | `erfc(0) = 1` | **libm**，~50–100 cycle | **libm**，~50–100 cycle | 大 x 时精度高 |
| `tgamma(x)` | 伽马函数 Γ(x)，阶乘的连续推广 | ∫₀^∞ tˣ⁻¹e⁻ᵗ dt | `tgamma(5) = 24` | **libm**，~80–200 cycle | **libm**，~80–200 cycle | x 为非正整数时极值；大 x 溢出 |
| `lgamma(x)` | 计算 ln\|Γ(x)\|，避免 Γ(x) 溢出 | ln\|Γ(x)\| | `lgamma(5) = ln24` | **libm**，~80–200 cycle | **libm**，~80–200 cycle | 返回对数，符号需另查 |

---

## 10. 取整

| 函数 | 功能 | 数学公式 | 示例 | ARM64 实现 / 周期 | x86 实现 / 周期 | 注意事项 |
|------|------|---------|------|------------------|----------------|---------|
| `ceil(x)` | 向上取整，返回不小于 x 的最小整数 | ⌈x⌉ | `ceil(1.2) = 2` | **硬件指令** `FRINTP`，延迟 2 / 吞吐 2 | **硬件指令** SSE4.1 `ROUNDPS`/`ROUNDSD`（mode=2），延迟 ~6 / 吞吐 ~0.5 | 结果仍为浮点 |
| `floor(x)` | 向下取整，返回不大于 x 的最大整数 | ⌊x⌋ | `floor(1.8) = 1` | **硬件指令** `FRINTM`，延迟 2 / 吞吐 2 | **硬件指令** SSE4.1 `ROUNDSD`（mode=1），延迟 ~6 / 吞吐 ~0.5 | 负数向 -∞，与 `trunc` 不同 |
| `trunc(x)` | 向零取整，去掉小数部分 | trunc(x) | `trunc(-1.8) = -1` | **硬件指令** `FRINTZ`，延迟 2 / 吞吐 2 | **硬件指令** SSE4.1 `ROUNDSD`（mode=3），延迟 ~6 / 吞吐 ~0.5 | 与 `floor` 在负数时不同 |
| `round(x)` | 四舍五入到最近整数，半整数远离 0 | round(x) | `round(1.5) = 2` | **硬件指令** `FRINTA`，延迟 2 / 吞吐 2 | **libm** 或 SSE4.1 自定义 | `ROUND` 不支持 ties-away，x86 需额外处理 |
| `nearbyint(x)` | 就近取整到最近整数，使用当前舍入模式（默认 ties-to-even） | 就近整数 | `nearbyint(1.5) = 2` | **硬件指令** `FRINTN`，延迟 2 / 吞吐 2 | **硬件指令** SSE4.1 `ROUNDSD`（mode=0） | 不触发 inexact 异常 |
| `rint(x)` | 同 `nearbyint`，但可能触发 inexact 异常 | 就近整数 | `rint(1.5) = 2` | **硬件指令** `FRINTX`，延迟 2 / 吞吐 2 | **硬件指令** SSE4.1 `ROUNDSD`（mode=4） | 可能触发 inexact 异常 |

---

## 11. 浮点分解与操作

| 函数 | 功能 | 数学公式 | 示例 | ARM64 实现 / 周期 | x86 实现 / 周期 | 注意事项 |
|------|------|---------|------|------------------|----------------|---------|
| `frexp(x, &e)` | 把 x 分解为尾数和指数：x = m·2ᵉ，m ∈ [0.5, 1) | x = m·2ᵉ | `frexp(8, &e)` → m=0.5, e=4 | **libm** 或位操作，~1–5 cycle | **libm** 或位操作，~1–5 cycle | 与 `ldexp` 互逆 |
| `ldexp(x, e)` | 计算 x·2ᵉ，快速缩放 | x·2ᵉ | `ldexp(0.5, 4) = 8` | **libm** 或位操作，~1–5 cycle | **libm** 或位操作，~1–5 cycle | 比 `x * pow(2, e)` 快 |
| `modf(x, &ip)` | 把 x 拆成整数部分和小数部分，整数部分存入 `*ip` | x = ip + fp | `modf(3.7, &ip)` → ip=3.0 | **libm**，~1–5 cycle | **libm**，~1–5 cycle | ip 和返回值符号同 x |
| `scalbn(x, n)` | 计算 x·2ⁿ，n 为 int | x·2ⁿ | `scalbn(1, 10) = 1024` | **libm** 或位操作，~1–5 cycle | **libm** 或位操作，~1–5 cycle | 与 `ldexp` 类似 |
| `ilogb(x)` | 提取浮点指数，返回 `⌊log₂\|x\|⌋` 的整数 | ⌊log₂\|x\|⌋ | `ilogb(8) = 3` | **libm** 或位操作，~1–5 cycle | **libm** 或位操作，~1–5 cycle | 返回 `int`；x=0 返回 `FP_ILOGB0` |
| `logb(x)` | 同 `ilogb`，但返回浮点数 | ⌊log₂\|x\|⌋ | `logb(8) = 3.0` | **libm**，~1–5 cycle | **libm**，~1–5 cycle | 返回 `double` |

---

## 12. 次一可表示值

| 函数 | 功能 | 数学公式 | 示例 | ARM64 实现 / 周期 | x86 实现 / 周期 | 注意事项 |
|------|------|---------|------|------------------|----------------|---------|
| `nextafter(x, y)` | 返回 x 朝 y 方向的下一个可表示浮点数 | — | `nextafter(1, 2) > 1` | **libm**，~1–5 cycle | **libm**，~1–5 cycle | 用于 ULP 步进、浮点精度测试 |
| `nextup(x)` | 返回 x 朝 +∞ 方向的下一个可表示浮点数 | — | `nextup(1) > 1` | **libm**，~1–5 cycle | **libm**，~1–5 cycle | C99 起；比 `nextafter(x, +Inf)` 直观 |

---

## 13. NaN 生成

| 函数 | 功能 | 数学公式 | 示例 | ARM64 实现 / 周期 | x86 实现 / 周期 | 注意事项 |
|------|------|---------|------|------------------|----------------|---------|
| `nan(s)` | 生成一个安静 NaN（quiet NaN），字符串 s 可指定 payload | — | `nan("")` | **libm** 或位操作，~1–2 cycle | **libm** 或位操作，~1–2 cycle | payload 实现定义；用于测试 NaN 传播 |

---

## 14. 分类与判断

| 函数 | 功能 | 数学公式 | 示例 | ARM64 实现 / 周期 | x86 实现 / 周期 | 注意事项 |
|------|------|---------|------|------------------|----------------|---------|
| `fpclassify(x)` | 返回 x 的浮点分类：`FP_NAN`、`FP_INFINITE`、`FP_ZERO`、`FP_SUBNORMAL`、`FP_NORMAL` | — | `fpclassify(0.0) = FP_ZERO` | **硬件指令** `FCMP` + 位检查，~2 cycle | **硬件指令** `UCOMISS`/`UCOMISD` + 位检查，~2 cycle | 返回宏常量 |
| `isfinite(x)` | 判断 x 是否有限（非 Inf、非 NaN） | — | `isfinite(1.0) = 1` | **硬件指令** `FCMP`，~2 cycle | **硬件指令** `UCOMISS`/`UCOMISD`，~2 cycle | 排除 Inf/NaN |
| `isinf(x)` | 判断 x 是否 ±Inf | — | `isinf(1.0/0.0) = 1` | **硬件指令** `FCMP` + 位检查，~2 cycle | **硬件指令** `UCOMISS`/`UCOMISD` + 位检查，~2 cycle | 返回非零，不区分正负 |
| `isnan(x)` | 判断 x 是否 NaN | — | `isnan(0.0/0.0) = 1` | **硬件指令** `FCMP`（NaN 比较特殊），~2 cycle | **硬件指令** `UCOMISS`/`UCOMISD`（无序标志），~2 cycle | 返回非零 |
| `isnormal(x)` | 判断 x 是否规格化浮点数（非 0、非次正规、非 Inf、非 NaN） | — | `isnormal(1.0) = 1` | **libm** 或位操作，~2 cycle | **libm** 或位操作，~2 cycle | 次正规数返回 0 |
| `signbit(x)` | 判断 x 的符号位是否为负 | — | `signbit(-0.0) = 1` | **硬件指令** 位操作，~1 cycle | **硬件指令** 位操作，~1 cycle | 含 -0 |

---

## 15. 比较（不触发异常）

| 函数 | 功能 | 数学公式 | 示例 | ARM64 实现 / 周期 | x86 实现 / 周期 | 注意事项 |
|------|------|---------|------|------------------|----------------|---------|
| `isgreater(x, y)` | 判断 x > y，NaN 参与时返回 0，不触发浮点异常 | x > y | `isgreater(2, 1) = 1` | **硬件指令** `FCMP` + `CSET`，~2 cycle | **硬件指令** `UCOMISS`/`UCOMISD` + `SET`，~2 cycle | 不触发浮点异常 |
| `isgreaterequal(x, y)` | 判断 x ≥ y，NaN 返回 0 | x ≥ y | `isgreaterequal(1, 1) = 1` | **硬件指令** `FCMP` + `CSET`，~2 cycle | **硬件指令** `UCOMISS`/`UCOMISD` + `SET`，~2 cycle | 同上 |
| `isless(x, y)` | 判断 x < y，NaN 返回 0 | x < y | `isless(1, 2) = 1` | **硬件指令** `FCMP` + `CSET`，~2 cycle | **硬件指令** `UCOMISS`/`UCOMISD` + `SET`，~2 cycle | 同上 |
| `islessequal(x, y)` | 判断 x ≤ y，NaN 返回 0 | x ≤ y | `islessequal(1, 1) = 1` | **硬件指令** `FCMP` + `CSET`，~2 cycle | **硬件指令** `UCOMISS`/`UCOMISD` + `SET`，~2 cycle | 同上 |
| `islessgreater(x, y)` | 判断 x ≠ y，NaN 返回 0 | x ≠ y | `islessgreater(1, 2) = 1` | **硬件指令** `FCMP` + `CSET`，~2 cycle | **硬件指令** `UCOMISS`/`UCOMISD` + `SET`，~2 cycle | 等价于 `x < y \|\| x > y` |
| `isunordered(x, y)` | 判断 x、y 是否无序（任一为 NaN） | — | `isunordered(NaN, 1) = 1` | **硬件指令** `FCMP`（无序标志），~2 cycle | **硬件指令** `UCOMISS`/`UCOMISD`（PF 标志），~2 cycle | 用于检测 NaN 参与的比较 |

---

## 关键差异总结

| 函数 | ARM64 | x86 | 差异说明 |
|------|-------|-----|---------|
| `fabs` | `FABS` 指令，延迟 2 | `ANDPS`/`ANDPD` 位操作 | x86 无专用 FABS，用位与 |
| `fmax`/`fmin` | `FMAX`/`FMIN` 单指令，NaN 语义符合 C | `MAXSS`/`MINSS` 单指令，但 NaN 语义不符合 C，需编译器额外处理 | ARM64 更符合标准 |
| `fma` | `FMADD` 基础指令 | 需 FMA3/FMA4 扩展 | x86 需 CPU 支持 |
| `sqrt` | `FSQRT`，延迟随输入变化（7–32） | `SQRTSD`，延迟 ~13–19（Skylake） | ARM64 延迟更低 |
| 取整 | `FRINT*`，延迟 2 / 吞吐 2 | SSE4.1 `ROUND*`，延迟 ~6 / 吞吐 ~0.5 | ARM64 略快 |
| `fmod` | libm（glibc 有 ARM64 优化） | libm（x87 `FPREM` 已淘汰） | 两者都慢，热路径避免 |
| `sin`/`cos` | libm，ARM64 有向量化实现 | libm，x87 `FSIN` 已淘汰 | 都走 libm，x86 的 x87 版本更慢 |

**一句话总结**：ARM64 在基础数学运算（`fabs`、`fmax`、`fmin`、取整）上普遍有硬件指令支持，且 NaN 语义更符合 C 标准；x86 也有对应 SSE 指令，但 `fmax`/`fmin` 的 NaN 语义需编译器额外处理，`fma` 需 FMA3 扩展。两者的超越函数（`sin`/`cos`/`exp`/`log`）都走 libm，ARM64 可通过 SLEEF 库或 glibc 向量化实现优化，x86 有 glibc 的 AVX 优化版。