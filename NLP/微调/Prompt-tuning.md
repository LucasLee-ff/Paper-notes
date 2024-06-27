# Prompt-Tuning
# 核心思想
1. 简化版本的prefix-tuning：仅在输入的地方加入多个虚拟的token，且token的参数是可学习的。
2. 相比prefix-tuning参数量更少

## Prompt Ensembling
- 借鉴model ensembling的思路（把基于不同任务训练的模型组合使用），prompt ensembling把基于不同任务训练出来的prompt组合使用。
- 由于prompt-tuning背后的LLM参数是固定的，仅在prompt上有区别，因此在推理$N$个不同任务的时候，可以把数据组成一个大小为$N$的batch；仅需在每条数据前面加上不同任务的prompt，就可以一次性推理出所有任务的结果。
