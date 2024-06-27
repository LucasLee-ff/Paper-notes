# P-Tuning
## 研究动机
1. 现有的预训练LLM需要在不同的任务下使用不同的prompt。
2. 人工编写的prompt非常不稳定，有时只是几个词语或表述发生了变化，性能就会有明显的下降，在few-shot learning中尤甚。
3. 有相关工作曾尝试在给定任务中搜索最优的prompt，但仍改变不了离散的prompt（个人理解即基于自然语言编写的prompt）非常不稳定这一事实。

## 核心思路
- 与prompt-tuning类似，在输入token的前面拼接可学习的embedding以优化模型表现。
- 与prompt-tuning的主要区别：prompt-tuning直接学习embedding，P-tuning训练了一个由LSTM/MLP构成的prompt encoder，embedding会先经过encoder再与输入进行拼接。
- 个人理解，在prompt encoder这里，思路和prefix-tuning类似，都用了一个神经网络对prompt的embedding进行编码；在拼接的环节上，和prompt-tuning类似，仅在输入的地方拼接。

## 消融实验
### Prompt Encoder
- 用LSTM和MLP差别不大，但都比直接优化embedding好。
- 这个实验结果和prefix-tuning的说法一致，因此prefix-tuning也是用了一个MLP来映射prefix。
### Prompt token的位置
- 插入的prompt token最好不要切断原有的句子（也符合直觉）。
- 在不切断的前提下，放在输入的边缘或中间没有明显区别。
- 实践中视情况而定
### Prompt token的数量
- 数量对效果有明显影响，但不是越多越好，实践中同样需要视情况而定。

## 其他发现
P-tuning有助于稳定模型在不同的离散prompt下的表现：
- 加入P-tuning前，同一个模型，输入不同的离散prompt，表现差别巨大。
- 加入P-tuning后，不同的离散prompt的表现差距缩小。