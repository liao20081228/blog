---
title: C数学库
tags: C
---




> 全部都有 `float`(后缀`f`) / `double`(无后缀)；部分有`long double`(后缀`l`)。 


[TOC]

# 取整函数

| 函数         | 功能                 | 数学规则                           | 示例                                | 实现          | 工程注意事项                                    |
| ------------ | -------------------- | ---------------------------------- | ----------------------------------- | ------------- | ----------------------------------------------- |
| ceil(x)      | 向上取整，向正无穷   | ⌈x⌉ 返回≥x 的最小整数           | `ceil(2.1)=3.0; ceil(-2.1)=‑2.0`   | 硬件 `FCVTPS` | 返回 double，不是 int；负数行为易错；溢出无告警 |
| floor(x)     | 向下取整，向负无穷   | ⌊x⌋ 返回≤x 的最大整数           | `floor(2.9)=2.0; floor(-2.1)=‑3.0` | 硬件 `FCVTMS` | **角度规约高频**；负数行为与 C 整数除法不同     |
| trunc(x)     | 截断取整，向 0       | 舍弃小数部分                       | `trunc(2.9)=2.0; trunc(-2.9)=‑2.0` | 硬件 `FCVTZS` | 等价 double 强制转 int 的截断行为               |
| round(x)     | 四舍五入，0.5 远离 0 | 0.5 向远离 0 舍入                  | `round(2.5)=3.0; round(-2.5)=‑3.0` | 硬件 `FCVTAS` | 不是银行家舍入；GNSS 时间转换常用               |
| nearbyint(x) | 按 FPU 舍入模式取整  | 遵从 FPU CSR，不产生 FE_INEXACT    | `nearbyint(2.5)=2.0`(默认偶舍入)    | 硬件 `FCVTNS` | 默认就近偶舍入；不置浮点异常标志，UT 友好       |
| rint(x)      | 按 FPU 舍入模式取整  | 同 nearbyint，舍入时置`FE_INEXACT` | `rint(2.5)=2.0`                     | 硬件 `FCVTNS` | 会修改浮点异常状态；单元测试需清除异常标志      |



# 浮点数拆解与指数缩放

| 函数               | 功能                        | 数学规则                              | 示例                                  | 实现         | 工程注意事项                                |
| ------------------ | --------------------------- | ------------------------------------- | ------------------------------------- | ------------ | ------------------------------------------- |
| frexp(x, &y)       | 拆分为尾数与 2 的指数       | x=mantissa×2^exp^,0.5≤∥mantissa∥<1 | `frexp(8.0, &e)` → mantissa=0.5, e=4 | 软件 libm    | 处理 0、inf、NaN 边界；尾数范围`[0.5,1)`    |
| ldexp(x, y)        | 乘以 2 的 y次幂             | res=x⋅2^y^                           | `ldexp(1.0,3)=8.0`                    | 硬件 `SCALB` | exp 越界输出 inf/0；不要用 pow(2, y)        |
| modf(x, &int_part) | 拆分整数、小数部分          | x=int_part+frac，符号与 x 一致        | `modf(-2.3, &i)` → i=-2.0，frac=-0.3 | 软件 libm    | 常用于拆分整数秒 / 小数秒；结果通过指针输出 |
| scalbn(x,n)        | 乘以2^n^，n 为 int          | res=x⋅2^n^                           | `scalbn(1.0,3)=8.0`                   | 硬件 `SCALB` | 性能远优于 x \*pow(2,n) 优先使用            |
| ilogb(x)           | 获取二进制指数，返回 int    | 提取浮点数二进制指数                  | `ilogb(8.0)=3`                        | 硬件 `FLOGB` | 0/inf/NaN 返回特殊魔数，输入必须合法性判断  |
| logb(x)            | 获取二进制指数，返回 double | 同 ilogb，返回 double                 | `logb(8.0)=3.0`                       | 硬件 `FLOGB` | 只看幅值，允许负数输入，不会报错            |

# 一句话分级

- **硬件便宜，随便用**：`ceil floor trunc round nearbyint rint fabs copysign fmax fmin sqrt fma`
- **glibc 软件，昂贵，重点优化**：`fmod remainder sin cos tan asin acos atan atan2 exp log pow hypot`
- **分类宏，零成本**：`isnan isfinite signbit` 等全部

需要的话，我可以再单独出一张**「你 perf 里出现过的函数 → 优化手段」对照表**（sincosf32x / fmod / sqrt / asin / atan2 / log10 逐个对应怎么改）。
