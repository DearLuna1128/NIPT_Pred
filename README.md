# NIPT-T13-18-21 无创产前检测临床判读系统

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![AutoGluon](https://img.shields.io/badge/AutoGluon-Latest-green.svg)](https://auto.gluon.ai/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## 📋 项目概述

本项目是一个基于机器学习的无创产前检测（NIPT）临床判读系统，专门用于预测胎儿染色体13、18、21三体异常。系统采用**FF估计模型**与**异常判读模型**的标准化临床判读流程，为产前筛查提供高精度的预测服务。

### 🎯 核心功能

- **胎儿游离DNA浓度（FF）估计**：基于染色体Z值、GC含量、测序质量等特征预测胎儿DNA浓度
- **染色体异常检测**：预测13、18、21号染色体三体异常风险
- **标准化临床判读流程**：4步骤质量控制与风险分级判断
- **临床决策支持**：提供高风险、中等风险、低风险三级分类

## 🏗️ 系统架构

```
NIPT-T13-18-21/
├── 📄 q4_analysis.py                   # 训练代码（核心）
├── 📁 outputs/                         # 训练结果与模型权重
│   └── Q4/                            # 问题4相关输出
│       ├── improved_ff_estimator/      # FF估计模型
│       │   ├── learner.pkl             # 模型权重
│       │   ├── predictor.pkl           # 预测器
│       │   ├── metadata.json           # 模型元数据
│       │   └── models/                 # 模型文件目录
│       └── q4_improved_pipeline/       # 改进管道结果
│           ├── improved_ff_estimator/  # FF估计模型
│           └── 问题4_女胎异常判定方案_完整技术报告.md
├── 📄 requirements.txt                 # 依赖包列表
└── 📄 README.md                        # 本文件
```

## 🚀 快速开始

### 环境要求

- Python 3.8+
- 推荐使用conda或virtualenv创建虚拟环境

### 安装依赖

```bash
# 克隆仓库
git clone https://github.com/DearLuna1128/NIPT-T13-18-21-.git
cd NIPT-T13-18-21-

# 安装依赖
pip install -r requirements.txt
```

### 基础使用

#### 1. 运行训练代码

```bash
# 运行完整的训练流程
python q4_analysis.py
```

#### 2. 使用预训练模型

```python
from q4_analysis import ImprovedClinicalPipeline

# 初始化管道
pipeline = ImprovedClinicalPipeline()

# 加载预训练模型
pipeline.load_models("outputs/Q4/improved_ff_estimator/")

# 进行预测
results = pipeline.predict(sample_data)
```

## 🔬 临床判读流程模型

### 4步骤标准化流程

#### 步骤1：质量控制（QC）
- **测序数据检查**：若测序数据 < 300万条或比对率 < 70%，判定测序质量不通过
- **输出结果**：直接输出"无效检测"
- **建议措施**：重新测序

#### 步骤2：FF估计与有效性判断
- **模型输入**：合格数据输入AutoGluon FF估计模型
- **FF预测**：得到胎儿游离DNA浓度预测值 `FF_hat`
- **有效性判断**：若 `FF_hat < 4%`，输出"无效检测"
- **建议措施**：建议复检

#### 步骤3：异常风险预测
- **模型输入**：将 `FF_hat ≥ 4%` 的样本特征输入XGBoost模型
- **风险预测**：得到三类染色体风险概率 `P_T21`, `P_T18`, `P_T13`
- **模型特征**：57个特征，包括Z值、GC含量、FF交互项等

#### 步骤4：风险分级判断
- **高风险（阳性）**：`P > 风险阈值`
  - 建议：遗传咨询与羊水穿刺确证
- **中等风险（需复核）**：`风险阈值 × 50% < P ≤ 风险阈值`
  - 建议：二次上机测序复核
- **低风险（阴性）**：`P ≤ 风险阈值 × 50%`
  - 建议：常规产检

## 📊 模型性能

### FF估计模型
- **特征数量**：44个特征
- **性能指标**：
  - 皮尔逊相关系数：**0.9941**（接近完美）
  - 平均绝对误差：**0.0045**（极低误差）

## 🔧 核心特征

### FF估计器特征（44个）
- **染色体Z值**：z_chr13, z_chr18, z_chr21, z_chrX
- **GC含量**：gc_content_total, gc_content_chr13/18/21
- **测序质量**：mapping_rate, duplicate_rate, raw_reads, unique_reads
- **临床信息**：孕周、BMI、年龄及其非线性变换
- **交互特征**：孕周×BMI、年龄×BMI等

### 异常分类器特征（56个）
- **基础特征**：所有FF估计器特征
- **FF预测值**：ff_hat
- **性别标识**：is_male
- **Z×FF交互**：z_chr13_x_ff_hat, z_chr18_x_ff_hat, z_chr21_x_ff_hat, z_chrX_x_ff_hat
- **GC×FF交互**：gc_content_total_x_ff_hat, gc_content_chr13_x_ff_hat等

## 📈 技术特点

### 1. 生物学原理
- **FF估计**：基于染色体计数一致性和技术偏倚校正
- **异常检测**：Z值分析结合FF校正，捕捉异常与FF的复合效应
- **交互建模**：Z×FF交互特征建模生物学交互

### 2. 技术创新
- **无Y染色体依赖**：完全基于常染色体信息估计FF
- **严格质控体系**：多维度QC确保数据质量
- **可解释性**：SHAP分析提供特征重要性解释

### 3. 临床适用性
- **标准化流程**：4步骤质量控制与风险分级
- **三级风险分类**：高风险、中等风险、低风险
- **明确建议**：遗传咨询、复检、常规产检

## 📋 使用指南

### 数据格式要求

输入数据应为包含以下列的DataFrame：
- **检测孕周**：格式为"Xw+Y"（如"12w+3"）
- **孕妇BMI**：数值型
- **年龄**：数值型
- **染色体Z值**：z_chr13, z_chr18, z_chr21, z_chrX
- **GC含量**：gc_content_total, gc_content_chr13/18/21
- **测序质量**：mapping_rate, duplicate_rate, raw_reads, unique_reads

### 预测示例

```python
import pandas as pd
from q4_analysis import ImprovedClinicalPipeline

# 准备样本数据
sample_data = pd.DataFrame({
    'gestational_age': [12.5],
    'bmi': [22.5],
    'maternal_age': [28],
    'z_chr13': [0.5],
    'z_chr18': [-0.3],
    'z_chr21': [1.2],
    'z_chrX': [0.8],
    'gc_content_total': [0.42],
    'mapping_rate': [0.85],
    'duplicate_rate': [0.15],
    'raw_reads': [5000000],
    'unique_reads': [4000000]
})

# 初始化管道
pipeline = ImprovedClinicalPipeline()

# 加载模型
pipeline.load_models("outputs/Q4/improved_ff_estimator/")

# 进行预测
results = pipeline.predict(sample_data)

# 输出结果
print(f"FF预测值: {results['ff_hat']:.4f}")
print(f"T21风险概率: {results['t21_prob']:.4f}")
print(f"T18风险概率: {results['t18_prob']:.4f}")
print(f"T13风险概率: {results['t13_prob']:.4f}")
print(f"临床建议: {results['clinical_decision']}")
```

## 📚 文件说明

### 训练代码
- **q4_analysis.py**：完整的训练代码，包含数据预处理、模型训练、性能评估

### 模型权重
- **outputs/Q4/improved_ff_estimator/**：FF估计模型文件
  - `learner.pkl`：AutoGluon学习器
  - `predictor.pkl`：预测器
  - `metadata.json`：模型元数据
  - `models/`：模型文件目录

### 技术报告
- **outputs/Q4/q4_improved_pipeline/问题4_女胎异常判定方案_完整技术报告.md**：完整技术报告

## 🤝 贡献指南

欢迎贡献代码和建议！请遵循以下步骤：

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 开启 Pull Request

## 📄 许可证

本项目采用 MIT 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情。

## 📞 联系方式

- 项目链接：[https://github.com/DearLuna1128/NIPT_Pred](https://github.com/DearLuna1128/NIPT_Pred)
- 问题反馈：[Issues](https://github.com/DearLuna1128/NIPT-Pred/issues)

## 🙏 致谢

感谢所有为NIPT技术发展做出贡献的研究者和开发者。

---

**注意**：本项目仅用于学术研究目的，不应用于临床诊断。任何临床应用都需要经过严格的临床试验验证和监管审批。

## 📝 更新日志

### v1.0.0 (2025-09)
- 初始版本发布


---

**重要声明**：本项目将在比赛结束后完全开源，包括所有训练代码、模型权重和使用指南。目前仅提供README文档作为项目概览。

## 🔒 代码开源计划

- **当前状态**：仅提供项目概览和技术文档
- **开源时间**：比赛结束后
- **开源内容**：
  - 完整的训练代码（q4_analysis.py）
  - 预训练模型权重
  - 详细的使用指南
  - 技术报告和性能分析
- **开源承诺**：确保所有代码、模型和文档的完整性和可重现性
