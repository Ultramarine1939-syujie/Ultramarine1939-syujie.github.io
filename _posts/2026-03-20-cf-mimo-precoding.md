---
layout: post
title: "CF-MIMO预编码设计：原理、实现与性能分析"
date: 2026-03-20
categories: [研究, 无线通信, 6G]
tags: [CF-MIMO, 预编码, MR, L-MMSE, LP-MMSE, 频谱效率]
---

# CF-MIMO预编码设计：原理、实现与性能分析

## 网络结构

### ALL (All APs serve all UEs)

理论模型常做此假设【所有AP服务所有UE（理论上界）】，但真实系统有其他问题：

- fronthaul负载爆炸
- 计算复杂度 ∝ AP数 × UE数
- pilot reuse困难

### DCC (Dynamic Cooperation Clustering)

每个UE只选一部分AP服务（实际系统），使用DCC后，复杂度和fronthaul负载可以显著降低：

- 不同UE的AP集合不同
- AP集合可以重叠
- 集合会随UE位置动态变化

> **结论**：如果CPU centralized precoding，但UE只由一部分AP服务（DCC），性能不会明显变差，但系统复杂度会大幅降低

## 信号模型

### AP的发射信号

$$ \mathbf{x}_l = \sum_{i \in \mathcal{D}_l} \mathbf{w}_{il} s_i $$

可见AP不是直接给UE发数据 $s_i$，而是先用预编码矩阵 $w_{il}$ 把数据"变成空间向量"再发射。

AP $l$ 只给他"负责的UE集合"发信号：
$$ \mathcal{D}_l = \{i: AP_l \text{服务} UE_i\} $$

### UE的接收信号

$$ y_k = \sum_{l=1}^L \mathbf{h}_{kl}^H \mathbf{x}_l + n_k $$

展开后：
$$ y_k = \sum_{i=1}^K \sum_{l=1}^L \mathbf{h}_{kl}^H \mathbf{w}_{il} s_i + n_k $$

其中：
- **期望信号**：$\sum_l \mathbf{h}_{kl}^H \mathbf{w}_{kl} \cdot s_k$
- **干扰**：$\sum_{i \neq k} \sum_l \mathbf{h}_{kl}^H \mathbf{w}_{il} s_i$
- **噪声**：$n_k$

## 预编码的作用

预编码 = 把"标量数据"变成"空间定向发射"

1. **增强信号**：使多个AP的信号**同相叠加（coherent combining）**
   $$ \left|\sum_l \mathbf{h}_{kl}^H \mathbf{w}_{kl}\right|^2 $$

2. **抑制干扰**：发给别人时，对UE $k$ **尽量正交/抵消**
   $$ \sum_{i \neq k} \left|\sum_l \mathbf{h}_{kl}^H \mathbf{w}_{il}\right|^2 $$

## 经典预编码方法

| 方法 | 数学结构 | 是否抑制干扰 | 信息范围 | 复杂度 |
|------|---------|------------|---------|--------|
| MR | $(\mathbf{h})$ | ❌ 不抑制 | 本地 | ⭐ |
| L-MMSE | $(\mathbf{R}^{-1}\mathbf{h})$ | ✔ 全局抑制 | 全局UE | ❌❌❌ |
| LP-MMSE | $(\mathbf{R}_{local}^{-1}\mathbf{h})$ | ✔ 局部抑制 | 局部UE | ⭐⭐ |

### 统一形式

$$ \mathbf{w}_{kl} = \sqrt{\rho_{kl}} \times \frac{1}{\sqrt{\mathbb{E}\{\|\overline{\mathbf{w}}_{kl}\|^2\}}} \times \overline{\mathbf{w}}_{kl} $$

其中：
- $\mathbf{w}_{kl}$：最终预编码向量（AP $l$→ UE $k$），维度 = 天线数 $N$
- $\overline{\mathbf{w}}_{kl}$：未归一化的"方向向量"，决定波束"往哪打"
- $\rho_{kl}$：AP $l$ 分配给 UE $k$ 的发射功率

---

### MR (Maximum Ratio)

$$ \overline{\mathbf{w}}_{kl}^{MR} = \mathbf{D}_{kl} \hat{\mathbf{h}}_{kl} $$

其中 $\mathbf{D}_{kl}$ 是服务选择矩阵：
$$ \mathbf{D}_{kl} = \begin{cases} \mathbf{I}_N, & l \in \mathcal{M}_k \\ \mathbf{0}, & \text{else} \end{cases} $$

**结构本质**：只用"信道方向"，没有任何干扰信息【信号增强，干扰不管】

> MR = 用"信道估计本身"当作预编码方向：$\mathbf{w} \sim \mathbf{h}$

**优点**：实现简单、复杂度低、只需要本地信道估计、可以得到闭式SINR表达式

**缺点**：完全不考虑多用户干扰，当UE较多时干扰严重

**适用场景**：
- 天线数量远大于用户数
- favorable propagation（信道近似正交）
- 在CF网络中这种AP少UE多的情况下，编码性能很差

---

### L-MMSE (Global-MMSE)

$$ \overline{\mathbf{w}}_{kl}^{L-MMSE} = p_k \left( \sum_{i=1}^K p_i (\hat{\mathbf{h}}_{il} \hat{\mathbf{h}}_{il}^H + \mathbf{C}_{il}) + \sigma^2 \mathbf{I} \right)^{-1} \mathbf{D}_{kl} \hat{\mathbf{h}}_{kl} $$

**编码目标**：最小化接收信号的均方误差【信号增强，压制干扰】

$$ \mathbf{w} \sim \left( \sum_{\text{all UE}} \hat{\mathbf{h}} \hat{\mathbf{h}}^H + \mathbf{C} + \sigma^2 \mathbf{I} \right)^{-1} \hat{\mathbf{h}} $$

**优点**：最大化SINR、能同时考虑信号干扰噪声三者的影响、SE最高

**缺点**：复杂度高、需要大量CSI信息、需要大规模矩阵求逆

---

### LP-MMSE (Local-MMSE)

$$ \overline{\mathbf{w}}_{kl}^{LP-MMSE} = p_k \left( \sum_{i \in \mathcal{D}_l} p_i (\hat{\mathbf{h}}_{il} \hat{\mathbf{h}}_{il}^H + \mathbf{C}_{il}) + \sigma^2 \mathbf{I} \right)^{-1} \mathbf{D}_{kl} \hat{\mathbf{h}}_{kl} $$

**与L-MMSE的区别**：

| 项 | L-MMSE | LP-MMSE |
|----|--------|---------|
| 求和范围 | $(i=1\sim K)$ | $(i \in \mathcal{D}_l)$ |
| 信息范围 | 全局 | 局部 |
| 复杂度 | 高 | 低 |

**结构本质**：在局部范围内抑制干扰【信号增强，压制干扰】
$$ \mathbf{w} \sim \left( \sum_{\text{local UE}} \hat{\mathbf{h}} \hat{\mathbf{h}}^H + \mathbf{C} + \sigma^2 \mathbf{I} \right)^{-1} \hat{\mathbf{h}} $$

## SE计算推导

### 接收信号结构

在"分布式下行架构"下，每个UE的可达频谱效率（SE）表达式：

$$ y_k^{dl} = \sum_{l=1}^L \mathbf{h}_{kl}^H \mathbf{x}_l + n_k $$

$$ \mathbf{x}_l = \sum_i \mathbf{D}_{il} \mathbf{w}_{il} \varsigma_i $$

### SINR表达式

$$ SINR_k = \frac{ \left| \mathbb{E} \left\{ \sum_l \mathbf{h}_{kl}^H \mathbf{D}_{kl} \mathbf{w}_{kl} \right\} \right|^2 }{ \sum_{i=1}^K \mathbb{E} \left\{ \left| \sum_l \mathbf{h}_{kl}^H \mathbf{D}_{il} \mathbf{w}_{il} \right|^2 \right\} - |\mathbb{E}\{\cdot\}|^2 + \sigma_{dl}^2 } $$

> **UatF (Use-and-then-Forget) 核心**：用信道估计来设计预编码，但在检测时"忘掉瞬时信道"，只用统计平均。因为在distributed情况下，UE不知道瞬时信道，AP也没有全局CSI。

### SE表达式

$$ SE = \log_2(1+SINR) $$

$$ SE_k = \frac{\tau_d}{\tau_c} \log_2(1+SINR_k) $$

其中 $\frac{\tau_d}{\tau_c}$ 是时间因子（下行符号数/信道使用周期）。

## 实验平台实现

### MATLAB代码示例

**MR预编码的SE计算**：
```matlab
SE_MR = prelogFactor * real(log2(1 + (abs(signal_MR).^2) ./ (interf_MR + sum(abs(cont_MR).^2,2) - abs(signal_MR).^2 + 1)));
```

**L-MMSE预编码的SE计算**：
```matlab
SE_L_MMSE = prelogFactor * real(log2(1 + (abs(signal_L_MMSE).^2) ./ (interf_L_MMSE - abs(signal_L_MMSE).^2 + 1)));
```

**LP-MMSE预编码的SE计算**：
```matlab
SE_LP_MMSE = prelogFactor * real(log2(1 + (abs(signal_LP_MMSE).^2) ./ (interf_LP_MMSE - abs(signal_LP_MMSE).^2 + 1)));
```

## 总结

CF-MIMO预编码设计的核心挑战在于**信号增强**与**干扰抑制**的平衡：

- **MR**方法简单但干扰抑制能力弱，适用于理想信道条件
- **L-MMSE**性能最优但复杂度高，适用于小规模用户场景
- **LP-MMSE**在性能和复杂度间取得平衡，是大规模系统的实用选择

在实际系统设计中，需要根据：
1. 用户规模
2. 信道条件
3. 计算资源
4. 延迟要求

综合选择合适的预编码方案。

---

**相关研究**：
- [Cell-Free无蜂窝网络技术：研究现状与未来路径](/2026/03/20/cell-free-overview.html)
- [CF-MIMO同步问题探究](/2026/03/20/cell-free-sync.html)
