
### 1 介绍


#### 1\.1 背景


越来越多的手机和平板电脑成为许多人的主要计算设备。这些设备上强大的传感器（包括摄像头、麦克风和GPS），加上它们经常被携带的事实，意味着它们可以访问前所未有的**大量数据**，其中大部分本质上是**私人的**。根据这些数据学习的模型持有承诺通过支持更智能的应用程序来大大提高可用性，但数据的敏感性意味着将其存储在集中位置存在风险和责任。


#### 1\.2 本文贡献


本文的主要贡献是


* 将来自移动设备的分散数据的训练问题（联邦学习）确定为一个重要的研究方向；
* 选择可以应用于该设置的简单实用的算法FedAvg；
* 对所提出的方法进行广泛的实证评估。


更具体地说，本文介绍了FedAvg算法，它将每个客户端上的局部随机梯度下降（SGD）与执行模型平均的服务器相结合。本文对该算法进行了广泛的实验，证明了它对**不平衡**和**非IID数据分布**具有鲁棒性，并且可以将在分散数据上训练深度网络所需的通信轮次减少几个数量级。


#### 1\.3 联邦学习的理想问题


* 对真实世界的移动设备上的数据进行训练 比 对数据中心可获得的代理数据进行训练，有明显的优势；
* 数据是隐私的或数据量很大；
* 对于监督任务，标签可以从用户交互中自然推断出来。


举例：


* 图像分类任务。预测哪些照片最有可能在未来被多次查看或分享。用户拍摄的照片是隐私的，但对于本地，用户对照片的删除、共享等行为就是推断出来的标签。
* 单词预测。用户在手机上输入时，输入法预测下一个单词。输入信息是隐私的，用户选择的下一个单词就是推断出来的标签。


#### 1\.4 联邦学习与分布式的对比


* 非独立同分布：不同用户对移动设备的使用是不同的，因此数据非独立同分布。
* 不平衡：一些用户会比其他人更频繁地使用服务或应用程序，从而导致本地训练数据的数量不同。
* 大规模分布式：预计参与优化的客户端数量将远远大于每个客户端的平均实例数量。
* 通信受限：移动设备有时候离线，或处于缓慢昂贵的连接中。


### 2 FedAvg


#### 2\.1 损失函数


对于机器学习问题，对于样本(xi,yi)的损失为fi(w)，那么全局损失定义为：


f(w)\=def1n∑ni\=1fi(w)在联邦学习问题中，假设有K个客户端，第k个客户端的数据集为Pk，数据集大小nk\=\|Pk\|。那么对于客户端k，该客户端数据的损失函数为：


Fk(w)\=1nk∑i∈Pkfi(w)全局的损失函数定义为客户端损失的加权平均：


f(w)\=∑Kk\=1nknFk(w)#### 2\.2 通信成本与计算成本


对于数据集中到中心的情况，由于数据量较大，通信成本相对较小，计算成本较大。


通信成本指客户端与中央服务器之间传输数据所需的成本。联邦学习中，会受到移动设备带宽限制，同时客户端通常仅在有电源和有WiFi等情况下愿意参与优化，因此通信成本较大。而设备数据量小、手机有GPU等特性使得计算成本较小。


为了减小通信成本，方法：


* 增加并行，每轮使用更多客户端（对应“客户端通常仅在有电源和有WiFi等情况下愿意参与优化”限制）。
* 每个客户端在每个通信轮之间执行更复杂的计算，而不是执行像梯度计算这样的简单计算。


#### 2\.3 相关工作


以往工作没有考虑不平衡和非独立同分布数据，以及客户端数量少。


#### 2\.4 FedSGD


根据当前的模型wt计算梯度gk\=∇Fk(wt)。由于：


∇f(wt)\=∇\[∑Kk\=1nknFk(wt)]\=∑Kk\=1nkngk那么中心服务器聚合**梯度**并进行更新的结果为：


wt\+1←wt−η∇f(wt)\=wt−η∑Kk\=1nkngk上式也等价于客户端先在本地做一次梯度更新，中心服务器再对**模型**进行加权平均：


wt\+1k←wt−ηgkwt\+1←∑Kk\=1nknwt\+1k#### 2\.5 FedAvg


写成上述第二种形式后，可以在做平均之前，**多次**迭代本地更新：


wk←wk−η∇Fk(wk)每个客户端可以多次计算上式得到本地在第t轮的最终模型，最后中心服务器将这些本地**模型**进行聚合得到wt\+1。


这就是FedAvg的思想，该算法主要有三个超参数：


* C：每次选择的客户端的比例
* B：本地训练时batchsize，当B\=∞，即全批量
* E：本地训练轮数


当B\=∞,E\=1时，FedAvg和FedSGD等价


这里还定义了每轮的本地更新次数：uk\=EnkB，由该公式也可以算出，FedSGD每轮本地更新次数为1。


完整的伪代码：


![](https://img2024.cnblogs.com/blog/3389949/202411/3389949-20241114213024892-1284313060.png)
至此我们可以简单比较FedSGD和FedAvg：






| 算法 | local | server |
| --- | --- | --- |
| FedSGD | 计算本轮梯度 | 收集local的梯度，加权平均后作为server要下降的梯度 |
| FedAvg | 多次梯度下降，得到本轮的本地模型 | 收集local的模型，加权平均后作为本轮得到的模型 |



### 3 实验


#### 3\.1 模型初始化


![](https://img2024.cnblogs.com/blog/3389949/202411/3389949-20241114214328398-152362950.png)
聚合参数θ：以θw\+(1−θ)w′对两个模型进行聚合，得到最终模型。


左图是使用两个初始模型w,w′训练不同数据得到的损失，右图是两模型使用同一个w初始化训练不同数据，可以看出右边损失较小，且当θ\=0\.5效果最好。因此在联邦学习实验中，每个客户端需要共享相同的初始化模型。


#### 3\.2 数据集和训练任务


选取大小适中的数据集，以便研究**超参数**。


第一个任务是MNIST数字识别，使用两个模型：


* 多层感知机。2个隐藏层，每个隐藏层有200个单元，使用ReLU激活。


199210个参数：图像为28×28，转为一维后是784。第一层偏置偏置784∗200\+偏置200，第二层偏置偏置200∗200\+偏置200，第三层偏置偏置200∗10\+偏置10
* 32∗5∗5卷积\+2∗2最大池化\+64∗5∗5卷积\+2∗2最大池化\+512单元全连接\+ReLU\+Softmax


数据集划分：


* iid：划分100个客户端，每个客户端接收600张图。
* 非iid：先按数字对图片进行排序，并划分成200个大小为300的碎片，给100个客户端每个分2个碎片，即每个客户端分到的数据只包含两个数字。


分出来的数据集有iid和非iid，但都是平衡的。


第二个任务是字符预测，使用LSTM，读取一行字符预测下一个字符。


数据集是莎士比亚全集，每个说话角色为一个客户端，共1146个。每个客户端，前80%的行是训练集，后20%行是测试集。


数据集划分：


* iid：将所有文字平均划分给每个客户端。
* 非iid：每个客户端仅有该角色说的话。


**学习率**设置在1013到1016区间。


#### 3\.3 增加并行性


C控制并行量，因此先改变C。


![](https://img2024.cnblogs.com/blog/3389949/202411/3389949-20241114214943695-875487816.png)
实验记录了MLP达到97测试集准确率和CNN达到99测试准确率所需要的通信轮数。


使用小批量，当C\=0\.1时效果就已经较好。为平衡计算效率和收敛速度，之后实验固定C\=0\.1。


#### 3\.4 增加每个客户端的计算量


在FedAvg算法部分，我们已经指出，每轮本地更新次数为uk\=EnkB。在实验中设置独立同分布的更新次数为期望更新次数，即u\=EnkB。


![](https://img2024.cnblogs.com/blog/3389949/202411/3389949-20241114215258988-457467499.png)
首先，对于两种任务，增加B都减少了通信轮数。


对于MNIST任务，iid效果比非iid更显著。实际生活我们设备上的数字也不会是规律性的，因此这种情况是该方法鲁棒的论证。


对于莎士比亚数据集，非iid效果很好，而这代表了我们在现实生活的数据分布（不同的人说话数量相差很大）。推测是某些客户端有较大的数据集，使本地训练更具有价值。


#### 3\.5 FedSGD vs FedAvg


![](https://img2024.cnblogs.com/blog/3389949/202411/3389949-20241114215613720-1483056182.png)
可以看出，FedAvg不仅减少通信轮数，还提高了测试精度（蓝色实线是FedSGD）。推测是模型平均会产生类似dropout正则化的收益。


#### 3\.6 是否能过度优化客户端


对于非常大的本地迭代次数，FedAvg可能会停滞或发散。这一结果表明，对于某些模型，尤其是在收敛的后期阶段，减少每轮的本地计算量（即减小E或增大B）可能是有益的，就像衰减学习率一样。


![](https://img2024.cnblogs.com/blog/3389949/202411/3389949-20241114215726891-288867943.png)
#### 3\.7 CIFAR实验


数据集包含50000个训练数据和10000个测试数据，将其平均划分给100个客户端，每个客户端包含500个训练数据和100个测试数据。


使用的模型为两个卷积层\+两个全连接层\+一个线性变换层。


图像会经过裁剪为24∗24、左右反转、调整对比度、亮度等预处理。


单机的SGD对比10个客户端的FedSGD和FedAvg：


![](https://img2024.cnblogs.com/blog/3389949/202411/3389949-20241114215845741-1022074463.png)
现有的模型，对CIFAR分类任务的测试精度已经很高，但这里只要达到80%左右即可，原因是本文的目标是评估FedAvg方法，而非提高CIFAR测试精度。


不同学习率的影响：


![](https://img2024.cnblogs.com/blog/3389949/202411/3389949-20241114220013521-595766826.png)
#### 3\.8 大规模LSTM实验


为了证明方法在现实世界问题上有效，还在大规模的预测下一个单词任务上进行了实验。


训练数据集由来自大型社交网络的1000万个公开帖子组成。按作者对帖子进行了分组，总共有超过500,000名客户。文中将每个客户端的数据集限制为最多5000个单词，并对10000个作者的数据进行了测试。


![](https://img2024.cnblogs.com/blog/3389949/202411/3389949-20241114215449138-1156320845.png)
**原文链接：**[https://arxiv.org/abs/1602\.05629](https://github.com)


 本博客参考[FlowerCloud机场](https://hanlianfangzhi.com)。转载请注明出处！
