# RetroMAE
## 研究动机
1. 对于稠密检索（dense retrieval），目前仍未有非常好用的预训练策略。
2. 现有的向量模型通常都是在token-level的任务（mask language modeling）上预训练的，作者认为这样的训练策略使模型欠缺sentence-level的表示能力。
3. 其中一种常见方案是使用对比学习，但是会受限于数据增强的质量，并且需要构建大量负样本。
4. 另一种方案是auto-encoding，因此本文主要聚焦于如何进行auto-encoding。

## 主要贡献
提出了全新的、针对检索（Retrieval oriented）的向量模型预训练范式，并且使用的是MAE：
1. 训练时使用mask过的文本输入encoder，得到句子embedding；再次对文本输入mask后，与句子embedding拼接，共同输入decoder，得到重建后的句子。
2. 使用非对称模型结构：encoder是类似BERT的架构，decoder仅为单层transformer。
3. 非对称masking比例：mask掉原始输入文本的15~30%，输入decoder前mask掉原始输入文本的50~70%。

## 核心思想
- 针对检索场景，向量模型产生的向量必须能够表达足够的sentence-level的语义信息，因此decoding阶段应该足够困难，从而促使encoder能够学得更好。因此，decoding阶段，输入文本才会被非常激进地mask掉了50~70%。

## 具体方法
### Encoding
1. 随机用特殊token把输入文本mask掉，然后输入BERT（12层，hidden_dim=768）。
2. 最终输出的[CLS] token作为整句的embedding。
3. 像BERT一样预测被mask的token，计算损失，得到$L_{enc}$
### Decoding
1. 再次对输入文本进行mask，然后将输入文本的encoder得到的句子embedding与输入文本的embedding进行拼接，送进单层transformer。
2. 使用所有被mask的token与真实词的交叉熵的和作为decoding阶段的损失$L_{dec}$。
### Enhanced Decoding
个人认为这是本论文的最特殊的地方
作者认为原有的decoding策略得到的信息不够，主要体现在以下方面（讲故事）：
1. 交叉熵的计算仅仅基于被mask掉的那部分token。
2. 每个被mask的token在重建时都基于同一个上下文信息，也就是说在经过decoder得到一个hidden state后，所有token的重建都是基于这个hidden state的。

基于此，提出了新的enhanced decoding：
1. 将encoding阶段获得的句子embedding重复N份，变成长度为N的序列，并加上位置编码，作为decoder的其中一个输入$H_1$。
2. 将encoding阶段获得的句子embedding，和输入文本序列（没有mask）拼接，组成长度为N的序列，并加上位置编码，作为decoder的第二个输入$H_2$。
3. decoder做attention计算时，$H_1$作为$Q$，$H_2$作为$K$和$V$。
4. 使用mask矩阵把attention矩阵mask掉，逻辑如下：每个token能看到的token都是通过采样得到的，且都能看到首个token，因为首个token是encoder产出的句子embedding的信息。
5. 用enhanced decoding生成的向量做句子重建，使用所有被mask的token与真实词的交叉熵的和作为decoding阶段的损失$L_{dec}$。
### Loss
训练loss即为上述的$L_{enc} + L_{dec}$。