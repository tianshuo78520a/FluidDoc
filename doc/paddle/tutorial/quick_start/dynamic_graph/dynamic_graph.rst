动态图
======

从飞桨开源框架2.0beta版本开始，飞桨默认为用户开启了动态图模式。在这种模式下，每次执行一个运算，可以立即得到结果（而不是事先定义好网络结构，然后再执行）。

在动态图模式下，您可以更加方便的组织代码，更容易的调试程序，本示例教程将向你介绍飞桨的动态图的使用。

设置环境
--------

我们将使用飞桨2.0beta版本，并确认已经开启了动态图模式。

.. code:: ipython3

    import paddle
    import paddle.nn.functional as F
    import numpy as np
    
    paddle.disable_static()
    print(paddle.__version__)
    print(paddle.__git_commit__)



.. parsed-literal::

    0.0.0
    89af2088b6e74bdfeef2d4d78e08461ed2aafee5


基本用法
--------

在动态图模式下，您可以直接运行一个飞桨提供的API，它会立刻返回结果到python。不再需要首先创建一个计算图，然后再给定数据去运行。

.. code:: ipython3

    a = paddle.randn([4, 2])
    b = paddle.arange(1, 3, dtype='float32')
    
    print(a.numpy())
    print(b.numpy())
    
    c = a + b
    print(c.numpy())
    
    d = paddle.matmul(a, b)
    print(d.numpy())


.. parsed-literal::

    [[-0.49341336 -0.8112665 ]
     [ 0.8929015   0.24661176]
     [-0.64440054 -0.7945008 ]
     [-0.07345356  1.3641853 ]]
    [1. 2.]
    [[0.5065867  1.1887336 ]
     [1.8929014  2.2466118 ]
     [0.35559946 1.2054992 ]
     [0.92654645 3.3641853 ]]
    [-2.1159463  1.386125  -2.2334023  2.654917 ]


使用python的控制流
------------------

动态图模式下，您可以使用python的条件判断和循环，这类控制语句来执行神经网络的计算。（不再需要\ ``cond``,
``loop``\ 这类OP)

.. code:: ipython3

    a = paddle.to_tensor(np.array([1, 2, 3]))
    b = paddle.to_tensor(np.array([4, 5, 6]))
    
    for i in range(10):
        r = paddle.rand([1,])
        if r > 0.5:
            c = paddle.pow(a, i) + b
            print("{} +> {}".format(i, c.numpy()))
        else:
            c = paddle.pow(a, i) - b
            print("{} -> {}".format(i, c.numpy()))



.. parsed-literal::

    0 +> [5 6 7]
    1 +> [5 7 9]
    2 +> [ 5  9 15]
    3 -> [-3  3 21]
    4 -> [-3 11 75]
    5 +> [  5  37 249]
    6 +> [  5  69 735]
    7 -> [  -3  123 2181]
    8 +> [   5  261 6567]
    9 +> [    5   517 19689]


构建更加灵活的网络：控制流
-------------------------------

-  使用动态图可以用来创建更加灵活的网络，比如根据控制流选择不同的分支网络，和方便的构建权重共享的网络。接下来我们来看一个具体的例子，在这个例子中，第二个线性变换只有0.5的可能性会运行。
-  在sequence to sequence with
   attention的机器翻译的示例中，你会看到更实际的使用动态图构建RNN类的网络带来的灵活性。

.. code:: ipython3

    class MyModel(paddle.nn.Layer):
        def __init__(self, input_size, hidden_size):
            super(MyModel, self).__init__()
            self.linear1 = paddle.nn.Linear(input_size, hidden_size)
            self.linear2 = paddle.nn.Linear(hidden_size, hidden_size)
            self.linear3 = paddle.nn.Linear(hidden_size, 1)
    
        def forward(self, inputs):
            x = self.linear1(inputs)
            x = F.relu(x)
    
            if paddle.rand([1,]) > 0.5: 
                x = self.linear2(x)
                x = F.relu(x)
    
            x = self.linear3(x)
            
            return x     

.. code:: ipython3

    total_data, batch_size, input_size, hidden_size = 1000, 64, 128, 256
    
    x_data = np.random.randn(total_data, input_size).astype(np.float32)
    y_data = np.random.randn(total_data, 1).astype(np.float32)
    
    model = MyModel(input_size, hidden_size)
    
    loss_fn = paddle.nn.MSELoss(reduction='mean')
    optimizer = paddle.optimizer.SGD(learning_rate=0.01, 
                                     parameters=model.parameters())
    
    for t in range(200 * (total_data // batch_size)):
        idx = np.random.choice(total_data, batch_size, replace=False)
        x = paddle.to_tensor(x_data[idx,:])
        y = paddle.to_tensor(y_data[idx,:])
        y_pred = model(x)
    
        loss = loss_fn(y_pred, y)
        if t % 200 == 0:
            print(t, loss.numpy())
    
        loss.backward()
        optimizer.minimize(loss)
        model.clear_gradients()


.. parsed-literal::

    0 [2.0915627]
    200 [0.67530334]
    400 [0.52042854]
    600 [0.28010666]
    800 [0.09739777]
    1000 [0.09307177]
    1200 [0.04252927]
    1400 [0.03095707]
    1600 [0.03022156]
    1800 [0.01616007]
    2000 [0.01069116]
    2200 [0.0055158]
    2400 [0.00195092]
    2600 [0.00101116]
    2800 [0.00192219]


构建更加灵活的网络：共享权重
---------------------------------

-  使用动态图还可以更加方便的创建共享权重的网络，下面的示例展示了一个共享了权重的简单的AutoEncoder的示例。
-  你也可以参考图像搜索的示例看到共享参数权重的更实际的使用。

.. code:: ipython3

    inputs = paddle.rand((256, 64))
    
    linear = paddle.nn.Linear(64, 8, bias_attr=False)
    loss_fn = paddle.nn.MSELoss()
    optimizer = paddle.optimizer.Adam(0.01, parameters=linear.parameters())
    
    for i in range(10):
        hidden = linear(inputs)
        # weight from input to hidden is shared with the linear mapping from hidden to output
        outputs = paddle.matmul(hidden, linear.weight, transpose_y=True) 
        loss = loss_fn(outputs, inputs)
        loss.backward()
        print("step: {}, loss: {}".format(i, loss.numpy()))
        optimizer.minimize(loss)
        linear.clear_gradients()


.. parsed-literal::

    step: 0, loss: [0.37666085]
    step: 1, loss: [0.3063845]
    step: 2, loss: [0.2647248]
    step: 3, loss: [0.23831272]
    step: 4, loss: [0.21714918]
    step: 5, loss: [0.1955545]
    step: 6, loss: [0.17261818]
    step: 7, loss: [0.15009595]
    step: 8, loss: [0.13051331]
    step: 9, loss: [0.11537809]


The end
--------

可以看到使用动态图带来了更灵活易用的方式来组网和训练。
