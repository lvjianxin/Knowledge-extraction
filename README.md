### **基于中文的知识抽取，BaseLine：Bi-LSTM+CRF**

**Overview**

基于字的BiLSTM-CRF，使用Bakeoff-3评测中所采用的的BIO标注集，即B-PER、I-PER代表人名首字、人名非首字，B-LOC、I-LOC代表地名首字、地名非首字，B-ORG、I-ORG代表组织机构名首字、组织机构名非首字，O代表该字不属于命名实体的一部分，也可以采用更复杂的BIOSE标注集。

以句子为单位，将一个含有 n 个字的句子（字的序列）记作
	  
`x=(x1,x2,...,xn)`

其中 xi 表示句子的第 i 个字在字典中的id，进而可以得到每个字的one-hot向量，维数是字典大小。

**模型**

![Bi-LSTM标签预测原理图](https://github.com/lvjianxin/Knowledge-extraction/blob/master/img-folder/webp.webp.jpg)


第一层：look-up 层，利用预训练或随机初始化的embedding矩阵将句子中的每个字 xi 由one-hot向量映射为低维稠密的字向量（character embedding）xi∈Rd ，d 是embedding的维度。在输入下一层之前，设置dropout以缓解过拟合。

第二层：双向LSTM层，自动提取句子特征。将一个句子的各个字的char embedding序列 (x1,x2,...,xn) 作为双向LSTM各个时间步的输入，再将正向LSTM输出的隐状态序列 (h1⟶,h2⟶,...,hn⟶) 与反向LSTM的 (h1⟵,h2⟵,...,hn⟵) 在各个位置输出的隐状态进行按位置拼接 ht=[ht⟶;ht⟵]∈Rm，得到完整的隐状态序列。

第三层：是CRF层，进行句子级的序列标注。CRF层的参数是一个 (k+2)×(k+2) 的矩阵 A ，Aij 表示的是从第 i 个标签到第 j个标签的转移得分，进而在为一个位置进行标注的时候可以利用此前已经标注过的标签，之所以要加2是因为要为句子首部添加一个起始状态以及为句子尾部添加一个终止状态。

**CRF作用**

由于BiLSTM的输出为单元的每一个标签分值，我们可以挑选分值最高的一个作为该单元的标签。例如，对于单元w0,“B-Person”有最高分值—— 1.5，因此我们可以挑选“B-Person”作为w0的预测标签。同理，我们可以得到w1——“I-Person”，w2—— “O” ，w3——“B-Organization”，w4——“O”。

虽然我们可以得到句子x中每个单元的正确标签，但是我们不能保证标签每次都是预测正确的。例如，图2.中的例子，标签序列是“I-Organization I-Person” and “B-Organization I-Person”，很显然这是错误的。

![去除CRF层的BiLSTM模型](https://github.com/lvjianxin/Knowledge-extraction/blob/master/img-folder/2.png)


**CRF层能从训练数据中获得约束性的规则**

CRF层可以为最后预测的标签添加一些约束来保证预测的标签是合法的。在训练数据训练过程中，这些约束可以通过CRF层自动学习到。

这些约束可以是：

I：句子中第一个词总是以标签“B-“ 或 “O”开始，而不是“I-”

II：标签“B-label1 I-label2 I-label3 I-…”,label1, label2, label3应该属于同一类实体。例如，“B-Person I-Person” 是合法的序列, 但是“B-Person I-Organization” 是非法标签序列.

III：标签序列“O I-label” is 非法的.实体标签的首个标签应该是 “B-“ ，而非 “I-“, 换句话说,有效的标签序列应该是“O B-label”。

有了这些约束，标签序列预测中非法序列出现的概率将会大大降低。

