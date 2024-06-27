# Prefix-Tuning
## 研究动机
文章观点：
1. 全参微调大模型成本巨大。
2. 不同的下游任务需要微调出不同的模型，因而需要存储多个大模型的参数文件。

个人补充：
1. 大模型泛化性能足够强。
2. 大模型有in-context learnig的能力。

## 相关工作：
1. 移除LM中的部分参数，如：训练一个binary mask。
2. 插入部分额外参数，如：加旁路、加task-specific layers。
3. AutoPrompt：搜索一系列离散的关键词并加在输入前面，以达到更好的效果；AutoPrompt是找到能够触发最佳答案的关键词，本文是通过训练优化一个task-specific的prefix向量，拼接在输入前面。
4. Controllable generation：训练时规定模型仅能根据某些metadata来生成结果，或解码阶段通过加权等方式控制生成结果。

## 具体方法
### 基础方法
1. 维护一个可训练的embedding矩阵$P$，形状为：prefix的长度×hidden_dim。
2. 在每一层transformer的输入中，查表($P$)，将每个prefix的embedding加到输入的token序列前面；后续流程与正常情况相同。
3. 反向传播时，冻结LLM参数，仅更新embedding矩阵$P$。

### 改进
作者发现直接更新$P$的参数容易导致训练不稳定，且效果不好，因此将$P$改为以下形式：
$P=MLP_\theta(P')$
其中，$P'$是一个维度更小的矩阵。
当训练完成后，可以直接舍弃MLP，仅保留算出来的$P$
