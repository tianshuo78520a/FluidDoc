Hello Paddle: 从普通程序走向机器学习程序
========================================

这篇示例向你介绍普通的程序跟机器学习程序的区别，并带着你用飞桨框架，实现你的第一个机器学习程序。

普通程序跟机器学习程序的逻辑区别
--------------------------------

作为一名开发者，你最熟悉的开始学习一门编程语言，或者一个深度学习框架的方式，可能是通过一个hello,
world程序。

学习飞桨也可以这样，这篇小示例教程将会通过一个非常简单的示例来向你展示如何开始使用飞桨。

机器学习程序跟通常的程序最大的不同是，通常的程序是在给定输入的情况下，通过告诉计算机处理数据的规则，然后得到处理后的结果。而机器学习程序则是在并不知道这些规则的情况下，让机器来从数据当中\ **学习**\ 出来规则。

作为热身，我们先来看看通常的程序所做的事情。

我们现在面临这样一个任务：

我们乘坐出租车的时候，会有一个10元的起步价，只要上车就需要收取。出租车每行驶1公里，需要再支付每公里2元的行驶费用。当一个乘客坐完出租车之后，车上的计价器需要算出来该乘客需要支付的乘车费用。

如果用python来实现该功能，会如下所示：

.. code:: ipython3

    def calculate_fee(distance_travelled):
        return 10 + 2 * distance_travelled
    
    for x in [1.0, 3.0, 5.0, 9.0, 10.0, 20.0]:
        print(calculate_fee(x))


.. parsed-literal::

    12.0
    16.0
    20.0
    28.0
    30.0
    50.0


接下来，我们把问题稍微变换一下，现在我们知道乘客每次乘坐出租车的公里数，也知道乘客每次下车的时候支付给出租车司机的总费用。但是并不知道乘车的起步价，以及每公里行驶费用是多少。我们希望让机器从这些数据当中学习出来计算总费用的规则。

更具体的，我们想要让机器学习程序通过数据学习出来下面的公式当中的参数w和参数b（这是一个非常简单的示例，所以\ ``w``\ 和\ ``b``\ 都是浮点数，随着对深度学习了解的深入，你将会知道\ ``w``\ 和\ ``b``\ 通常情况下会是矩阵和向量）。这样，当下次乘车的时候，我们知道了行驶里程\ ``distance_travelled``\ 的时候，我们就可以估算出来用户的总费用\ ``total_fee``\ 了。

::

   total_fee = w * distance_travelled + b

接下来，我们看看用飞桨如何实现这个hello, world级别的机器学习程序。

导入飞桨
---------

为了能够使用飞桨，我们需要先用python的\ ``import``\ 语句导入飞桨\ ``paddle``\ 。
同时，为了能够更好的对数组进行计算和处理，我们也还需要导入\ ``numpy``\ 。

如果你是在本机运行这个notebook，而且还没有安装飞桨，可以去飞桨的官网查看如何安装：\ `飞桨官网 <https://www.paddlepaddle.org.cn/>`__\ 。并且请使用2.0beta或以上版本的飞桨。

.. code:: ipython3

    import paddle
    paddle.disable_static()
    print("paddle version " + paddle.__version__)


.. parsed-literal::

    paddle version 0.0.0


准备数据
---------

在这个机器学习任务中，我们已经知道了乘客的行驶里程\ ``distance_travelled``\ ，和对应的，这些乘客的总费用\ ``total_fee``\ 。
通常情况下，在机器学习任务中，像\ ``distance_travelled``\ 这样的输入值，一般被称为\ ``x``\ （或者特征\ ``feature``\ ），像\ ``total_fee``\ 这样的输出值，一般被称为\ ``y``\ （或者标签\ ``label``)。

我们用\ ``paddle.to_tensor``\ 把示例数据转换为paddle的Tensor数据。

.. code:: ipython3

    x_data = paddle.to_tensor([[1.], [3.0], [5.0], [9.0], [10.0], [20.0]])
    y_data = paddle.to_tensor([[12.], [16.0], [20.0], [28.0], [30.0], [50.0]])

用飞桨定义模型的计算
--------------------

使用飞桨定义模型的计算的过程，本质上，是我们用python，通过飞桨提供的API，来告诉飞桨我们的计算规则的过程。回顾一下，我们想要通过飞桨用机器学习方法，从数据当中学习出来如下公式当中的\ ``w``\ 和\ ``b``\ 。这样在未来，给定\ ``x``\ 时就可以估算出来\ ``y``\ 值（估算出来的\ ``y``\ 记为\ ``y_predict``\ ）

::

   y_predict = w * x + b

我们将会用飞桨的线性变换层：\ ``paddle.nn.Linear``\ 来实现这个计算过程，这个公式里的变量\ ``x, y, w, b, y_predict``\ ，对应着飞桨里面的\ `Tensor概念 <https://www.paddlepaddle.org.cn/documentation/docs/zh/beginners_guide/basic_concept/tensor.html>`__\ 。

稍微补充一下
~~~~~~~~~~~~

在这里的示例中，我们根据经验，已经事先知道了\ ``distance_travelled``\ 和\ ``total_fee``\ 之间是线性的关系，而在更实际的问题当中，\ ``x``\ 和\ ``y``\ 的关系通常是非线性的，因此也就需要使用更多类型，也更复杂的神经网络。(比如，BMI指数跟你的身高就不是线性关系，一张图片里的某个像素值跟这个图片是猫还是狗也不是线性关系。）

.. code:: ipython3

    linear = paddle.nn.Linear(in_features=1, out_features=1)

准备好运行飞桨
----------------

机器（计算机）在一开始的时候会随便猜\ ``w``\ 和\ ``b``\ ，我们先看看机器猜的怎么样。你应该可以看到，这时候的\ ``w``\ 是一个随机值，\ ``b``\ 是0.0，这是飞桨的初始化策略，也是这个领域常用的初始化策略。（如果你愿意，也可以采用其他的初始化的方式，今后你也会看到，选择不同的初始化策略也是对于做好深度学习任务来说很重要的一点）。

.. code:: ipython3

    w_before_opt = linear.weight.numpy().item()
    b_before_opt = linear.bias.numpy().item()
    
    print("w before optimize: {}".format(w_before_opt))
    print("b before optimize: {}".format(b_before_opt))


.. parsed-literal::

    w before optimize: -1.7107375860214233
    b before optimize: 0.0


告诉飞桨怎么样学习
--------------------

前面我们定义好了神经网络（尽管是一个最简单的神经网络），我们还需要告诉飞桨，怎么样去\ **学习**\ ，从而能得到参数\ ``w``\ 和\ ``b``\ 。

这个过程简单的来陈述一下，你应该就会大致明白了（尽管背后的理论和知识还需要逐步的去学习）。在机器学习/深度学习当中，机器（计算机）在最开始的时候，得到参数\ ``w``\ 和\ ``b``\ 的方式是随便猜一下，用这种随便猜测得到的参数值，去进行计算（预测）的时候，得到的\ ``y_predict``\ ，跟实际的\ ``y``\ 值一定是有\ **差距**\ 的。接下来，机器会根据这个差距来\ **调整\ ``w``\ 和\ ``b``**\ ，随着这样的逐步的调整，\ ``w``\ 和\ ``b``\ 会越来越正确，\ ``y_predict``\ 跟\ ``y``\ 之间的差距也会越来越小，从而最终能得到好用的\ ``w``\ 和\ ``b``\ 。这个过程就是机器\ **学习**\ 的过程。

用更加技术的语言来说，衡量\ **差距**\ 的函数（一个公式）就是损失函数，用来\ **调整**\ 参数的方法就是优化算法。

在本示例当中，我们用最简单的均方误差(mean square
error)作为损失函数(``paddle.nn.MSELoss``)；和最常见的优化算法SGD（stocastic
gradient
descent)作为优化算法（传给\ ``paddle.optimizer.SGD``\ 的参数\ ``learning_rate``\ ，你可以理解为控制每次调整的步子大小的参数）。

.. code:: ipython3

    mse_loss = paddle.nn.MSELoss()
    sgd_optimizer = paddle.optimizer.SGD(learning_rate=0.001, parameters = linear.parameters())

运行优化算法
---------------

接下来，我们让飞桨运行一下这个优化算法，这会是一个前面介绍过的逐步调整参数的过程，你应该可以看到loss值（衡量\ ``y``\ 和\ ``y_predict``\ 的差距的\ ``loss``)在不断的降低。

.. code:: ipython3

    total_epoch = 5000
    for i in range(total_epoch):
        y_predict = linear(x_data)
        loss = mse_loss(y_predict, y_data)
        loss.backward()
        sgd_optimizer.minimize(loss)
        linear.clear_gradients()
        
        if i%1000 == 0:
            print("epoch {} loss {}".format(i, loss.numpy()))
            
    print("finished training， loss {}".format(loss.numpy()))


.. parsed-literal::

    epoch 0 loss [2107.3943]
    epoch 1000 loss [7.8432994]
    epoch 2000 loss [1.7537074]
    epoch 3000 loss [0.39211753]
    epoch 4000 loss [0.08767726]
    finished training， loss [0.01963376]


机器学习出来的参数
-------------------

经过了这样的对参数\ ``w``\ 和\ ``b``\ 的调整（\ **学习**)，我们再通过下面的程序，来看看现在的参数变成了多少。你应该会发现\ ``w``\ 变成了很接近2.0的一个值，\ ``b``\ 变成了接近10.0的一个值。虽然并不是正好的2和10，但却是从数据当中学习出来的还不错的模型的参数，可以在未来的时候，用从这批数据当中学习到的参数来预估了。（如果你愿意，也可以通过让机器多学习一段时间，从而得到更加接近2.0和10.0的参数值。)

.. code:: ipython3

    w_after_opt = linear.weight.numpy().item()
    b_after_opt = linear.bias.numpy().item()
    
    print("w after optimize: {}".format(w_after_opt))
    print("b after optimize: {}".format(b_after_opt))



.. parsed-literal::

    w after optimize: 2.017843246459961
    b after optimize: 9.771851539611816


hello paddle
---------------

通过这个小示例，希望你已经初步了解了飞桨，能在接下来随着对飞桨的更多学习，来解决实际遇到的问题。

.. code:: ipython3

    print("hello paddle")


.. parsed-literal::

    hello paddle

