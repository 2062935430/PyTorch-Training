🤳PyTorch训练工程的内容🤳
----------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------
1️⃣模型定义  

1.必须继承nn.Module这个类，要让PyTorch知道这个类是一个Module  

2.在init(self)中设置好需要的"组件"(如conv,pooling,Linear,BatchNorm等)，第一行用super(Net,self).__init__()实现父类的初始化  

3.最后，在forward(self,x)中定义好的“组件”进行组装，就像搭积木，把网络结构搭建出来，这样一个模型就定义好了  

以下为一个简单的模型示例（来源于知乎）：  

    import torch
    import torch.nn as nn
    import torch.functional as F
    class Net(nn.Module):
        def __init__(self):
            super(Net,self).__init__()#实现父类的初始化
            self.conv1=nn.Conv2d(3,6,5)#定义卷积层组件
            self.pool1=nn.MaxPool2d(2,2)#定义池化层组件
            self.conv2=nn.Conv2dn(6,16,5)
            self.pool2=nn.MaxPool2d(2,2)
            self.fc1=nn.Linear(16*5*5,120)#定义线性连接
            self.fc2=nn.Linear(120,84)
            self.fc3=nn.Linear(84,10)
        def forward(self,x):#x模型的输入
            x=self.pool1(F.relu(self.conv1(x)))
            x=self.pool2(F.relu(self.conv2(x)))
            x=x.view(-1,16*5*5)#表示将x进行reshape，为后面做为全连接层的输入
            x=F.relu(self.fc1(x))
            x=F.relu(self.fc2(x))
            x=self.fc3(x)
        return x
            
 /还有其他的常用方法类似于：nn.Sequetial、nn.Sequetial、torch.nn/
 
 ----------------------------------------------------------------------------------------------------------------------------
 
 2️⃣数据处理和加载
 你可以使用torch.utils.data.Dataset类来定义自己的数据集，并实现__getitem__和__len__方法，以支持数据的索引和大小查询2。你也可以使用torch.utils.data.DataLoader类来分批次、随机、并行地加载数据2。如果你的数据是图片，你还可以使用torchvision.datasets.ImageFolder类来方便地读取图片和标签
 
