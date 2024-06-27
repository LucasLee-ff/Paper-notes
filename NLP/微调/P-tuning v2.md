# P-Tuning v2
## 研究动机
1. 之前的prompt-tuning类型的方法在普通规模（100M~1B）的预训练模型中效果不佳。（注：prompt-tuning论文中也有提到，该方法在小参数量的模型中效果较差，随着模型参数的增加，prompt-tuning才逐渐追上全参微调的效果）。
2. 之前的prompt-tuning类型的方法在hard sequence labeling tasks（命名实体识别等需要对每个token进行分类的任务）上表现不佳。
3. 像P-tuning和Prompt-tuning这样仅在输入层加入embedding的方法，首先与序列长度的限制，可训练的参数并不多。
4. input token的embedding对模型输出的影响相对而言不够直接。

## 主要贡献
- 基于prefix-tuning等思路，找到了一个通用的类prompt-tuning方法，效果与全参微调相当，且在各个规模的预训练模型、各类NLU任务中都适用。

## 方法细节
- 与prefix-tuning类似，每一层的序列都拼接了可学习的prompt embedding。
- 与prefix-tuning不同，每一层拼接的prompt embedding都不一样。
- 与prefix-tuning不同，每一层拼接的prompt embedding都无需经过一个额外的MLP（实验发现这个MLP在不同任务上影响不同，有时候可能会有负增益）
- 不同任务拼接的prompt长度不同
- 因为主要针对sequence labeling任务，模型是用随机初始化的classification head而不是language modeling head
- 有监督学习