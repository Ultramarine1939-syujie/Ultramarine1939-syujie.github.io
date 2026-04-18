---
layout: post
title: "Cell-Free MIMO功率分配优化：PSO算法实践"
date: 2026-03-20
categories: 研究
tags: CF-MIMO 功率分配 PSO 优化算法 资源调度
---

# Cell-Free MIMO功率分配优化：PSO算法实践

## 研究背景与动机

### 研究场景

本研究聚焦于 **Cell-Free Massive MIMO 下行链路**系统。与传统蜂窝网络不同，Cell-Free架构中 $L$ 个接入点（AP）无小区边界限制，协作服务 $K$ 个用户（UE），天然消除了小区边缘效应，具有更均匀的覆盖和更高的频谱效率潜力。

**系统关键参数**：

| 符号 | 含义 | 仿真取值 |
|------|------|---------|
| $L$ | AP数量 | 100 |
| $K$ | UE数量 | 20 |
| $N$ | 每AP天线数 | 1 |
| $\tau_c$ | 相干时间 | 200符号 |
| $\tau_p$ | 导频长度 | 10符号 |
| $\sigma_e$ | 信道估计误差标准差 | 0.3 |

### 核心问题

在Cell-Free系统中，下行**功率分配**是影响系统和速率（ESR）的关键因素。传统方案基于大尺度增益按比例分配功率，计算简单但无法针对实时干扰结构进行优化，在高SNR（干扰受限）区域会产生"干扰平台"，ESR不再随功率增大而增长。

**研究问题**：能否用智能优化算法（PSO）突破传统分配的干扰平台，提升系统ESR？

## 研究工作内容

### 系统架构与模块划分

本项目建立了完整的Cell-Free Massive MIMO下行链路仿真平台，在三个维度上进行全因子对比仿真：

| 维度 | 选项 | 说明 |
|------|------|------|
| 接入拓扑 | All-UE / DCC | 全连接 vs 动态小区簇 |
| 功率分配 | 基准 / PSO | 大尺度增益分配 vs PSO优化分配 |
| 预编码方向 | MR / L-MMSE / Robust-MMSE | 三种预编码方案 |

共计 **2 × 2 × 3 = 12条性能曲线**，进行系统对比分析。

### PSO功率分配方案设计

#### 优化变量与目标

引入用户级功率缩放系数向量 $\mathbf{d} = [d_1, \dots, d_K]^T$，PSO求解：

$$ \max_{\mathbf{d}} \quad \text{ESR}(\mathbf{d}) = \sum_{k=1}^{K} \left(1 - \frac{\tau_p}{\tau_c}\right) \log_2(1 + \text{SINR}_k(\mathbf{d})) $$

#### 约束条件

- 功率约束：$\sum_{k=1}^K d_k = K$，每个用户的功率缩放系数之和为 $K$
- 物理约束：$d_k \geq 0$，确保功率非负

#### 粒子群初始化

在可行域内随机初始化粒子位置与速度：
$$ d_k \sim \mathcal{U}(0, 2), \quad v_k \sim \mathcal{U}(-0.1, 0.1) $$

### 关键代码实现

```matlab
% 目标函数：系统和速率
function esr = calculateESR(d, SINR, tau_c, tau_p)
    prelogFactor = (tau_c - tau_p) / tau_c;
    esr = prelogFactor * sum(log2(1 + SINR .* (d.^2)));
end

% PSO参数设置
params = struct();
params.maxIter = 100;
params.nParticles = 30;
params.w = 0.729;  % 惯性权重
params.c1 = 1.49;  % 个体学习因子
params.c2 = 1.49;  % 群体学习因子
```

## 仿真结果分析

### All-UE拓扑下的ESR对比

在All-UE拓扑中，PSO功率优化相比基准方案：

- **MR预编码**：PSO在高SNR区域实现显著增益，突破干扰平台
- **L-MMSE预编码**：PSO优化效果明显，系统ESR提升约15-20%
- **Robust-MMSE预编码**：结合鲁棒设计，PSO进一步优化性能

### DCC拓扑下的ESR对比

在DCC动态簇拓扑中：

- DCC显著降低了系统复杂度（AP只服务部分UE）
- PSO优化在不同预编码方案下均展现出稳定增益
- DCC + PSO的组合在性能和复杂度间取得良好平衡

### 关键发现

1. **干扰平台的突破**：PSO优化能够有效应对高SNR区域的干扰受限问题
2. **预编码与功率的协同**：不同预编码方案下PSO优化效果存在差异
3. **DCC的有效性**：动态簇技术可显著降低复杂度，性能损失可接受

## 结论与展望

### 主要结论

1. PSO优化功率分配能够有效提升Cell-Free MIMO系统的和速率
2. 在高SNR（干扰受限）区域，PSO方案相比传统大尺度增益分配具有明显优势
3. 结合DCC动态簇技术，系统复杂度可大幅降低

### 未来工作方向

- 探索更高效的优化算法（如遗传算法、深度强化学习）
- 研究分布式场景下的功率分配方案
- 结合通感一体化需求设计联合优化框架

---

**相关研究**：
- [Cell-Free无蜂窝网络技术：研究现状与未来路径](/2026/03/20/cell-free-overview.html)
- [CF-MIMO预编码设计：原理、实现与性能分析](/2026/03/20/cf-mimo-precoding.html)
