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
