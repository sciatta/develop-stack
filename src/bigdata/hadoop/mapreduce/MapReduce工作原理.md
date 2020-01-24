# 概述

MapReduce的核心思想是“<font color=red>分而治之</font>”，把大任务分解为小任务并行计算。Map负责“分”，即把复杂的任务分解为若干个“简单的任务”来并行处理。<font color=red>可以进行拆分的前提是这些小任务可以并行计算，彼此间几乎没有依赖关系</font>。Reduce负责“合”，即对Map阶段的结果进行全局汇总。



# 编程模型

## Map阶段

**第一步：InputFormat**

设置InputFormat类，将我们的数据切分成 (key, value) 输出。本质是把大任务拆分为互相独立的小任务。如TextInputFormat，输入的是block，输出的key是行偏移位置，value是每行的内容。

| InputFormat             | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| TextInputFormat         | 1、默认将每个block作为一个split<br />2、输出的key是行偏移位置，value是每行的内容 |
| CombineTextInputFormat  | 1、解决小文件导致过多split的问题。涉及到虚拟存储和切片过程，可以自定义split大小<br />2、输出的key是行偏移位置，value是每行的内容 |
| KeyValueTextInputFormat | 1、默认将每个block作为一个split<br />2、以自定义分隔符进行分割，输出相应的key和value |
| NLineInputFormat        | 1、以输入文件的N行作为一个split<br />2、输出的key是行偏移位置，value是每行的内容 |

**第二步：MapTask**

自定义map逻辑，对输入的 (key, value) 进行处理，转换成新的 (key, value) 进行输出。本质是把小任务拆分为最小可计算任务。例如需求是统计单词数量，第一步是TextInputFormat，则MapTask输入的key是行偏移位置，value是每行的内容。因此，可以进一步拆分单行的内容，输出的key是一个单词，value是1。后面把所有相同单词的value累加，即为一个单词出现的数量。最后汇总所有单词即可满足需求。



## Shuffle阶段

**第三步：Partition**

对输入的 (key, value) 进行分区。<font color ="red">满足条件的key</font>划分到一个分区中，一个分区发送到一个ReduceTask

1. Order：对不同分区的数据按照key进行排序
2. Combine：对分组后的数据进行规约(combine操作)，降低数据的网络拷贝（可选步骤）
3. Group：对排序后的数据进行分组，将<font color="red">相同key</font>的value放到一个集合当中



## Reduce阶段

1. ReduceTask：自定义Reduce逻辑，对输入的key，value对进行处理，转换成新的key，value对进行输出
2. OutputFormat：设置Outputformat将输出的key，value对数据进行保存到文件中



# MapTask工作机制

## MapTask个数

**MapTask的并行度是由什么决定的？**

其中，block是hdfs系统存储文件的单位。而每个MapTask处理一个切片split的数据量，split是逻辑上对输入进行切片。

切片大小的计算公式：`Math.max(minSize, Math.min(maxSize, blockSize));` 

其中，`mapreduce.input.fileinputformat.split.minsize=1` ，`mapreduce.input.fileinputformat.split.maxsize= Long.MAXValue` 。block的默认大小是128M，所以split的默认大小是128M，同block的大小一致。

如果要控制split的数量，则只需要改变minsize或maxsize就可以改变切片的大小。如果自定义split大于128M，minsize要大于128M；如果自定义split小于128M，maxsize要小于128M。



**如果有1000个小文件，每个文件在1K-100M之间，默认情况下有1000个block，1000个split，1000个MapTask并行处理？**

通过CombineTextInputFormat来控制小文件的切片数量，从而控制MapTask的数量。

切片机制：

`CombineTextInputFormat.setMaxInputSplitSize(job, 4194304);` // 4m

切片过程包括虚拟存储和切片两步：

- 虚拟存储：将所有小文件依次同`MaxInputSplitSize`参数作比较

1. 小于参数，则直接作为一个逻辑块
2. 大于参数，且不大于2倍参数，则均分作为两个逻辑块（防止出现太小的切片）
3. 大于2倍参数，则先按参数切割一个逻辑块，剩下的继续1、2、3步判断

如：`MaxInputSplitSize`设置为4M，输入文件大小为8.02M，满足3，则先划分一个4M的逻辑块，剩下人4.02继续判断；4.02满足2，则均分为两个2.01M的逻辑块。

- 切片：将所有虚拟存储逻辑块依次同`MaxInputSplitSize`参数作比较

1. 如果大于等于参数，则作为一个切片
2. 如果小于参数，则同下一个虚拟存储逻辑块一起作为一个切片

如：`MaxInputSplitSize`设置为4M，有4个小文件大小：1.7M、5.1M、3.4M以及6.8M

虚拟存储为6个逻辑块：1.7M 2.55M 2.55M 3.4M 3.4M 3.4M

切片为3个切片：（1.7M+2.55M=4.25M）（2.55M+3.4M=5.95M）（3.4M+3.4M=6.8M）



测试场景：

1. 10个文件：0.1K



1. 2个文件：8.1M 8K

虚拟存储：4M 2.05M 2.05M 8K

切片：4M 4.1M 8K



# ReduceTask工作机制

