🤳PyTorch训练工程的内容🤳
----------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------
1️⃣模型定义  
----------------------------------------------------------------------------------------------------------------------------

  首先必须继承nn.Module这个类，要让PyTorch知道这个类是一个Module。  

  然后在init(self)中设置好需要的"组件"(如conv,pooling,Linear,BatchNorm等)，第一行用super(Net,self).__init__()实现父类的初始化。  

  最后，在forward(self,x)中定义好的“组件”进行组装，就像搭积木，把网络结构搭建出来，这样一个模型就定义好了 。 

  以下为一个简单的模型示例（源于知乎）：  

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
            
   >模型定义还有其他的常用方法类似于：nn.Sequetial、nn.Sequetial、torch.nn。
 
----------------------------------------------------------------------------------------------------------------------------
2️⃣数据处理和加载
----------------------------------------------------------------------------------------------------------------------------
 
  我们可以使用torch.utils.data.Dataset类来定义自己的数据集，并实现__getitem__和__len__方法，以支持数据的索引和大小查询；  
  
  Dataset类是Pytorch中图像数据集中最为重要的一个类，也是Pytorch中所有数据集加载类中应该继承的父类。其中父类中的两个私有成员函数必须被重载，否则将会触发错误提示：  

    def getitem(self, index):
    def len(self):
  
  ***其中__len__应该返回数据集的大小，而__getitem__应该编写支持数据集索引的函数，例如通过dataset[i]可以得到数据集中的第i+1个数据。***  
  
  Dataset类是读入数据集数据并且对读入的数据进行了索引，但是光有这个功能是不够用的； 

  也可以使用torch.utils.data.DataLoader类来分批次、随机、并行地加载数据；  

  在实际的加载数据集的过程中，我们的数据量往往都很大，对此我们还需要一下几个功能：  

    可以分批次读取：batch-size  
    可以对数据进行随机读取，可以对数据进行洗牌操作(shuffling)，打乱数据集内数据分布的顺序  
    可以并行加载数据(利用多核处理器加快载入数据的效率)  
    
  如果数据是图片，还可以使用torchvision.datasets.ImageFolder类来方便地读取图片和标签:  
  
    imageFolder类:  
    
    ImageFolder(root, transform=None, target_transform=None, loader=default_loader)
    - root：在root指定的路径下寻找图片
    - transform：对PIL Image进行的转换操作，transform的输入是使用loader读取图片的返回对象
    - target_transform：对label的转换
    - loader：给定路径后如何读取图片，默认读取为RGB格式的PIL Image对象
  
---
3️⃣训练模型
---
  
  你需要定义损失函数和优化器，如torch.nn.MSELoss和torch.optim.SGD1。  
  
    criterion = torch.nn.MSELoss(reduction='sum') 
    optimizer = torch.optim.SGD(model.parameters(), lr=1e-4)
    
  然后，你需要在一个循环中进行前向传播、计算损失、反向传播和更新参数的操作。  
  
    //前向传播，后向传播
    
    for t in range(epoch):
        for step, (x, y) in enumerate(train_loader):
            # Forward pass: Compute predicted y by passing x to the model
            y_pred = model(x)
 
            # Compute and print loss
            loss = criterion(y_pred, y) # 计算损失函数
  
            # Zero gradients, perform a backward pass, and update the weights.
            optimizer.zero_grad() # 梯度置零，因为反向传播过程中梯度会累加上一次循环的梯度
            loss.backward() # loss反向传播
            optimizer.step() # 反向传播后参数更新 
   
 4️⃣更新参数
 ---
   
 在训练模型的过程中。  
 
 我们可以使用torch.optim模块中的优化器，如torch.optim.SGD，  
 
 来实现***梯度下降算法***，并在每个训练循环中调用optimizer.step()方法来更新参数。  

 >梯度下降的基本思想：最优化算法，用于求解损失函数（loss function，即预测值f(x)于真实值Y差异程度的运算函数）的最小值。  
 >
 >其思想为在当前位置求偏导，即梯度，正常梯度的方向类似于上山方向，  
 >
 >是使值函数增大的，下山最快则需使其最小，从负梯度求最小值，此为梯度下降。  
 
 1、预测函数  
 
 机器学习通过学习算法，自动发现数据背后的规律，不断改进模型，然后做出预测。
 
 ![函数](https://user-images.githubusercontent.com/128795948/235164403-16b56755-61bc-4663-bc64-f75baf8abef9.jpg)
 
 其中函数y=wx为预测函数，w即为该函数图像的斜率，样本点垂直到直线的偏离程度为损失函数，样本点和函数垂直x轴的偏离程度为偏差。

2、代价函数

-量化数据偏离程度（误差），常见方法为**均方误差**。  

-通过定义预测函数，后根据误差公式推导代价函数，将样本点拟合过程映射到一个函数图像上。  

![函数](https://user-images.githubusercontent.com/128795948/235165253-6aecd002-61e0-41e6-8a41-9d2053acf902.jpg)

3、梯度计算

机器学习的目标为拟合出最接近训练数据分布的直线，  

即找到误差代价最小的参数w，对应函数最低点位置，其过程为梯度下降。

![函数1](https://user-images.githubusercontent.com/128795948/235170383-25d992fd-d951-48b9-bada-b1560a41d1a1.png)
![函数1](https://user-images.githubusercontent.com/128795948/235171914-1395d89b-e920-464c-9572-116fdf53b8e1.png)

4、学习率

算法收敛时，令斜率乘一个非常小的值，使梯度下降优化，该值为学习率。  

通过学习率调整权重的方式就是***新w=旧w-斜率×学习率***。  

5、循环迭代

实际中，代价函数因样本多样而千变万化，不一定仅为一条抛物线，如预测函数为y=wx+b，代价函数变为误差e关于参数w和b的曲面；

同时还可能出现多个最小点存在时，机器学习目标将是找到最低点那个点，即全局最优，而非局部最优。  

![image](https://user-images.githubusercontent.com/128795948/235173754-46b96fda-61a2-4e54-b471-f13e7da61b0b.png)

代价函数甚至可以因元素多样变为更复杂更多维的图像而使可视化困难，但维度多少不影响梯度下降法的使用来寻找最小误差点。  

>批量梯度下降法（BGD）：全部训练样本参与计算，是梯度下降最原始的形式。优点在能保证算法精准度，找到全局最优解，但训练搜索过程变慢。  
>
>随机梯度下降法（SGD）：每下降一步仅需一个样本进行计算，优点在提升了计算的速度，但牺牲了一定的精准度。  
>
>小批量梯度下降法（MBGD）：每下降一步选用一小批样本参与计算，优点在简洁、高效，属于前二者的居中类型。  

**梯度下降法简单有效，适用范围广，但若学习率取值太大，可能反复横跳取不到最低点，太小又浪费计算量；**    

**除BGD外，SGD、MBGD无法保证找到全局最低点，更可能陷入全局最低点。**  

所以如今已经有其他的更优法可供选择：  

![1](https://user-images.githubusercontent.com/128795948/235175240-700e6644-2fe6-452d-b782-0d55aa1487e0.jpg)

---
5️⃣可视化训练过程
---

你可以使用tensorboardX这个第三方库来记录和展示训练过程中的各种指标，如损失、准确率、梯度等。

>1.创建一个 SummaryWriter 的示例
     from tensorboardX import SummaryWriter

    # Creates writer1 object.
    # The log will be saved in 'runs/exp'
    writer1 = SummaryWriter('runs/exp')

    # Creates writer2 object with auto generated file name
    # The log directory will be something like 'runs/Aug20-17-20-33'
    writer2 = SummaryWriter()

    # Creates writer3 object with auto generated file name, the comment will be appended to the filename.
    # The log directory will be something like 'runs/Aug20-17-20-33-resnet'
    writer3 = SummaryWriter(comment='resnet')
    
>2.接下来就可以调用 SummaryWriter 实例的各种add_something方法向日志中写入不同类型的数据了，  
>
>想要在浏览器中查看可视化这些数据，只要在命令行中开启 tensorboard 即可。

    tensorboard --logdir=<your_log_dir>
    
-add_scalar(tag, scalar_value, global_step=None, walltime=None)  

    tag (string): 数据名称，不同名称的数据使用不同曲线展示
    scalar_value (float): 数字常量值
    global_step (int, optional): 训练的 step
    walltime (float, optional): 记录发生的时间，默认为 time.time() 

***需要注意，这里的scalar_value一定是 float 类型，如果是 PyTorch scalar tensor，则需要调用.item()方法获取其数值。***  
  
  
-add_histogram(tag, values, global_step=None, bins='tensorflow', walltime=None, max_bins=None)  

    tag (string): 数据名称
    values (torch.Tensor, numpy.array, or string/blobname): 用来构建直方图的数据
    global_step (int, optional): 训练的 step
    bins (string, optional): 取值有 ‘tensorflow’、‘auto’、‘fd’ 等, 该参数决定了分桶的方式，详见这里。
    walltime (float, optional): 记录发生的时间，默认为 time.time()
    max_bins (int, optional): 最大分桶数
    
-add_graph(model, input_to_model=None, verbose=False, **kwargs)  

    model (torch.nn.Module): 待可视化的网络模型
    input_to_model (torch.Tensor or list of torch.Tensor, optional): 待输入神经网络的变量或一组变量

完成准备工作后，就可以开始自己的训练工程了。
