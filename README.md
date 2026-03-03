# 色谱杂质自动分析工作流

一个自动化工作流系统，用于：
1. 识别分子结构（从图片转SMILES）
2. 预测合成杂质（调用ASKCOS API）
3. 预测杂质保留时间
4. 评估杂质分离度
5. 优化色谱分析条件

## 项目概述

本项目实现了一个由**6个子代理**串联运行的完整工作流：

```
Image → SMILES → Impurities → RT Prediction → Resolution Check → Chromo Optimization
 (A1)     (A2)      (A3)         (A4)           (A5)              (A6)
```

## 项目结构

```
chromatography-impurity-analyzer/
├── README.md                          # 项目说明
├── requirements.txt                   # Python依赖
├── config.yaml                        # 配置文件
├── main.py                            # 主入口脚本
│
├── agents/                            # 6个子代理模块
│   ├── agent1_image_to_smiles.py     # 代理1：图片转SMILES
│   ├── agent2_askcos_impurity.py     # 代理2：杂质预测
│   ├── agent3_smiles_processor.py    # 代理3：SMILES处理
│   ├── agent4_retention_time.py      # 代理4：保留时间预测
│   ├── agent5_resolution_check.py    # 代理5：分离度评估
│   └── agent6_chromo_optimize.py     # 代理6：色谱条件优化
│
├── utils/                             # 工具函数
│   ├── askcos_client.py              # ASKCOS API客户端
│   ├── retention_predictor.py        # 保留时间预测
│   ├── smiles_converter.py           # SMILES转换
│   └── chromatography_utils.py       # 色谱工具函数
│
├── tests/                             # 单元测试
│   └── test_agents.py
│
└── data/                              # 数据目录
    ├── input/                        # 输入图片
    └── output/                       # 输出结果
```

## 快速开始

### 环境要求
- Python 3.8+
- Git

### 安装依赖

```bash
# 克隆仓库
git clone https://github.com/yanxiaowei0222-jpg/chromatography-impurity-analyzer.git
cd chromatography-impurity-analyzer

# 创建虚拟环境
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 安装依赖
pip install -r requirements.txt
```

### 配置

1. 创建 `.env` 文件：
```bash
cp .env.example .env
```

2. 编辑 `.env` 并填入你的信息：
```
ASKCOS_USERNAME=chenlong
ASKCOS_PASSWORD=Aa123456
OPENAI_API_KEY=your_openai_key
```

3. 编辑 `config.yaml` 配置工作流参数

### 运行工作流

```bash
# 运行完整工作流
python main.py --image path/to/compound.png --output results.json

# 运行单个代理（调试用）
python -m agents.agent1_image_to_smiles --image path/to/compound.png
```

## 六个子代理详解

### Agent 1: 图片转SMILES
- **功能**: 识别分子结构图片，转换为SMILES格式
- **实现**: OpenAI Vision API 或 OCR + RDKit
- **输入**: 分子结构图片
- **输出**: SMILES字符串

### Agent 2: ASKCOS杂质预测
- **功能**: 调用ASKCOS API预测合成杂质
- **实现**: ASKCOS REST API集成
- **输入**: 目标化合物SMILES
- **输出**: 杂质SMILES列表

### Agent 3: SMILES处理器
- **功能**: 提取、规范化、验证杂质SMILES
- **实现**: RDKit SMILES规范化
- **输入**: 杂质信息
- **输出**: 标准化的SMILES列表

### Agent 4: 保留时间预测
- **功能**: 预测杂质在不同色谱条件下的保留时间
- **实现**: Neural Network��XGBoost模型
- **输入**: 杂质SMILES
- **输出**: 保留时间预测值

### Agent 5: 分离度评估
- **功能**: 计算杂质间的分离度，识别需要拆分的组
- **实现**: 分离度计算算法
- **输入**: 保留时间数据
- **输出**: 分组信息、分离度评估

### Agent 6: 色谱条件优化
- **功能**: 为每个分组优化色谱条件，确保分离度 > 1.5
- **实现**: 参数优化算法（遗传算法或贝叶斯优化）
- **输入**: 分组信息
- **输出**: 优化的色谱条件

## 工作流执行流程

```python
# 伪代码示例
workflow = ChromatographyWorkflow(image_path)

# Step 1: 图片 → SMILES
smiles = agent1.process(image)

# Step 2: 预测杂质
impurities = agent2.predict(smiles)

# Step 3: 提取SMILES
impurity_smiles = agent3.extract(impurities)

# Step 4: 预测保留时间
rt_results = agent4.predict(impurity_smiles)

# Step 5: 评估分离度
groups = agent5.evaluate(rt_results)

# Step 6: 优化条件
conditions = agent6.optimize(groups)

return conditions  # 最终输出优化的色谱条件
```

## 输出格式

工作流输出JSON格式的完整分析结果：

```json
{
  "original_smiles": "CC(C)Cc1ccc(cc1)C(C)C(O)=O",
  "impurities": [
    {
      "smiles": "CC(C)Cc1ccc(cc1)C(C)=O",
      "confidence": 0.85
    }
  ],
  "retention_times": {
    "RT": {"condition": "A", "time": 5.2}
  },
  "separation_groups": [
    {
      "group_id": 1,
      "compounds": ["smiles1", "smiles2"],
      "resolution": 1.8
    }
  ],
  "chromatography_conditions": [
    {
      "method_id": "M1",
      "temperature": 40,
      "ph": 2.5,
      "organic_modifier": "acetonitrile",
      "organic_percentage": 30
    }
  ]
}
```

## 依赖项说明

| 包名 | 用途 |
|------|------|
| langgraph, langchain | 代理框架 |
| rdkit-pypi | 分子结构处理 |
| openai | Vision API调用 |
| aiohttp | 异步HTTP请求 |
| pandas, numpy | 数据处理 |
| scikit-learn | 机器学习模型 |
| loguru | 日志记录 |

## 开发计划

- [ ] Phase 1: Agent 1 + Agent 2 (第1-2周)
- [ ] Phase 2: Agent 3 + Agent 4 (第3-4周)
- [ ] Phase 3: Agent 5 + Agent 6 (第5-6周)
- [ ] Phase 4: 集成测试 + 优化 (第7-8周)

## 许可证

MIT

## 贡献者

- yanxiaowei0222-jpg

## 联系方式

如有问题，请提交Issue或联系开发者。