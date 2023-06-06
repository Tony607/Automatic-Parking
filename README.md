# 自动并排停车：路径规划、路径追踪与控制

这个仓库包含了一个在虚拟环境中实现自动并排停车系统的Python实现，包括路径规划、路径追踪和并排停车。代理(Agent)通过MPC控制器指导其在环境中的路线，并导向分配的停车位置。您可以在哔哩哔哩上找到视频教程 [控制的诗与远方——一种生活与科技的并行视角]()。

## 环境
开发自动停车系统的第一步是设计和开发一个能够利用`OpenCV`库进行视觉渲染的环境。这个环境在`environment.py`中以一个类的形式实现，并在开始时接收障碍物`env = Environment(obs)`。可以使用`env.render(x,y,angle)`来放置代理。下面显示了一个环境样本，您可以从1到24中选择停车位。

![开发环境](https://user-images.githubusercontent.com/56114938/127310550-745d7123-f02f-48ae-96a7-9f82089b9fd9.JPG)

## 路径规划
#### A* 算法
代理将使用A*找到从开始到目标的路径。[PythonRobotics](https://pythonrobotics.readthedocs.io/en/latest/modules/path_planning.html) 的这个A*实现考虑了障碍物和机器人半径等参数。

#### 用B样条插值路径
找到在离散的100\*100空间中的路径后，使用b样条将路径平滑并缩放到1000\*1000的环境空间。结果是一组引导我们的代理的点！

## 路径追踪
汽车的**运动模型**是:
```math
\left\{\begin{matrix}
\dot{x} = v . cos(ψ)\\
\dot{y} = v . sin(ψ)\\
\dot{v} = a\\
\dot{ψ} = v . tan(δ)/L
\end{matrix}\right.
```
```a: 加速度, δ: 转向角, ψ: 偏航角, L: 轴距, x: x位置, y: y位置, v: 速度```

**状态向量**是:
```math
z=[x,y,v,ψ]
```
```x: x位置, y: y位置, v: 速度, ψ: 偏航角```

**输入向量**是:
```math
u=[a,δ]
```
```a: 加速度, δ: 转向角```

#### 控制
MPC控制器根据模型控制车辆的速度和转向，车辆通过路径指导。这里有一个选项可以使用线性化模型进行MPC。在这种情况下，MPC

会在操作点周围线性化运动模型，然后进行优化。

## 并排停车
这部分包含4个规则，代理必须根据停车位置选择。首先，代理会找到一个到停车位置的路径，然后计算到达角度。根据到达角度，代理选择一个坐标作为ensure1。之后，使用下面提到的2个圆方程，规划从ensure1到ensure2的停车路径。MPC控制代理，车辆在ensure2坐标处停车。

![自动停车过程](https://user-images.githubusercontent.com/56114938/128083454-60f8ba82-00a8-43a2-b8ad-8d4ad09cc762.gif)

## 运行
使用以下命令运行代码：
```
$ pip install -r requirements.txt
$ cd CAR_kinematic_model
$ python main_autopark.py --x_start 0 --y_start 90 --psi_start 0 --parking 7
```
