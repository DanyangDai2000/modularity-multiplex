# 基于模块度的多层网络社区发现

Modularity-based Community Detection for Multilayer / Multiplex Networks

## 项目简介

在多层（multiplex）网络上做社区发现，比较三种**基于模块度**的方法，并在合成基准与真实多层网络上用模块度 Q 与 NMI 评估。

- 输入：多层网络，每一层是一个邻接矩阵，整体用 cell 数组 `{A_1, A_2, …, A_T}` 表示
- 输出：每个节点的社区划分 `idx`
- 角色：项目组长，负责方法整合、评估与整体实现

## 方法

### 1. GenLouvain —— 多层 Louvain（基线）
- 多层模块度最大化的 Louvain 实现，来自 Mucha / Porter / Onnela 等人的工作
- 本仓库含 Jutla, Jeub, Mucha 的 GenLouvain 实现（预编译 MEX 二进制）
- 位置：`GenLouvain/`

### 2. PMM —— Principal Modularity Maximization
- 实现 Lei Tang et al. 的「主成分模块度最大化」方法
- 接口：`[idx] = PMM(network, k, ell, maxKmeans)`
  - `network`：每层一个稀疏矩阵的 cell 数组
  - `k`：社区数；`ell`：每层提取的结构特征数；`maxKmeans`：k-means 重复次数（默认 1）
- 位置：`PMM/PMM.m`（同目录 `modularitymaximization.m`）

### 3. m-spec —— 谱方法
- 多层模块度的谱方法：用 `multiord` 构造模块度矩阵，用 `eigs_max` 取最大特征对，用 `compute_delta_Q` 贪心提取社区
- 接口：`[Q, X] = mSpec(A)`，`A` 为三层张量（节点 × 节点 × 层数）
- 位置：`m-spec/`

## 评估

| 指标 | 文件 | 说明 |
|------|------|------|
| 模块度 Q | `eval/QFModul.m` | Newman–Girvan 经典模块度（E. le Martelot） |
| NMI | `eval/nmi.m` | 标准化互信息（Mo Chen），另含 `rand_index`（ARI） |

## 目录结构

```
code_mmm/
├── GenLouvain/    # 第三方：多层 Louvain 基线（含 License.txt、预编译 MEX）
├── PMM/           # PMM：主成分模块度最大化（自研实现）
├── m-spec/        # 多层谱方法（multiord / eigs_max / compute_delta_Q）
├── eval/          # 评估：QFModul.m（Q）、nmi.m（NMI/ARI）
├── function/      # 数据读取与变换（transform_datanet / getn / transmat / SLDS）
├── data/          # 真实多层网络 + 合成基准数据
└── test_in_real_data.m        # 真实数据示例（TailorShop）
└── test_in_stimulate_data.m   # 合成数据示例（SLDS）
```

## 运行

需要 **MATLAB**。在 MATLAB 中把各子目录加入路径后运行示例脚本：

```matlab
addpath('data'); addpath('eval'); addpath('function');
addpath('function/SLDS'); addpath('PMM'); addpath('m-spec'); addpath('GenLouvain');

run('test_in_stimulate_data.m');   % 合成基准 SLDS：生成数据 → PMM 划分 → 算 NMI/ARI
run('test_in_real_data.m');        % 真实多层数据 TailorShop：PMM 划分 → 逐层算模块度 Q
```

> 注意：示例脚本里的部分数据路径为历史绝对路径（原 Windows 环境），运行前请改为本机相对路径；合成基准生成依赖外部 `MultilayerGM-MATLAB` 工具，需单独获取。

## 数据

- **真实多层网络**（在 `data/`）：TailorShop（Kapferer）、Drosophila、Euair、Fao、Florentine、London_Transport、Pierreauger、Ckmpims 等
- **合成基准**（SLDS）：由 `DirichletDCSBMBenchmark` 按多层度修正随机块模型生成（`function/SLDS/`）

## 依赖与许可

- 运行环境：MATLAB
- **GenLouvain**：Copyright (c) 2012–2014 Inderjit S. Jutla, Lucas G. S. Jeub, Peter J. Mucha —— 见 `GenLouvain/License.txt`（BSD 类许可，再分发需保留版权声明）
- **QFModul.m**：Newman–Girvan 模块度，代码由 E. le Martelot 编写
- **nmi.m**：标准化互信息，由 Mo Chen 编写
- **PMM**：基于 Lei Tang（Arizona State University, L.Tang@asu.edu, 2011）的 Principal Modularity Maximization
- 多层合成基准生成器 `MultilayerGM-MATLAB`（uuinfolab）为外部依赖，非本仓库内容

## 备注

- GenLouvain 的 MEX 二进制（`genlouvain` 等）为预编译文件，跨平台使用时需按官方仓库重新编译
- 本项目以「方法对比与工程实现」为目的，三种方法均为已有算法的实现/集成，非原创算法
