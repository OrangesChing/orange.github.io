---
title: 强化学习实践——程序建模模板及Gym使用
date: 2019-12-05 12:10:41
tags:
---

# 强化学习解决问题的设计流程

1. 将实际问题建模成马尔可夫决策过程，抽象出五元组（状态集、动作集、状态转移概率、奖励函数、折扣因子），  其中奖励与实际目标相关联
2. 根据动作是否连续选择对应的算法

# 强化学习的两类对象



## 环境

需要存储的信息

1. 所有可能状态集
2. 智能体的行动集
3. 智能体当前状态

需要做的事

1. 响应智能体行为，更新环境，给予智能体即时奖励
2. 给予个体观测值
3. 终止交互的条件



环境模板就不给出了，gym库在设计环境和个体交互时有规范的接口，可以借鉴他们的规范和接口



## 个体

需要存储的信息

1. 环境对象信息
2. 状态信息

需要做的事：

1. 观察功能：获得环境信息，哪些行为是允许的，获得的奖励
2. 决策功能：根据当前观测来判断下一时刻该采取什么行为，按照一个策略产生一个行动
3. 执行行动功能

```python
class Agent():
    def __init__(self, env: Env， ):
        self.env = env      # 个体持有环境的引用
        self.state = None   # 个体当前的观测，最好写成obs.

    def choose_action(self, state): pass # 执行一个策略，选择一个行为

    def act(self, a):       # 执行一个行为

    def learn(self， s, a, r, s_): pass   # 学习过程
    
    def check_state_exist(self, state)
```



# Gym

gym主要用于生成常见的强化学习环境，方便学习，即相当于给你写好了上面的环境类，你只需要写智能体类了。

gym的官方网址在：[这里](https://link.zhihu.com/?target=https%3A//gym.openai.com/)，其库代码托管地址在：[这里](https://link.zhihu.com/?target=https%3A//github.com/openai/gym/tree/master/gym)

## Gym的安装

### Windows

OpenAI官网没有说支持Windows，最好不要在Window上跑，因为我看视频写博客都在Windows了，就硬着头皮装了 ，没想到居然好像能用

找到Anaconda3的开始菜单目录，找到指令终端：**Anaconda Prompt**。

较老版本的Anaconda3这里没有这一项，无妨，可以去Anaconda3的安装文件夹下去找。运行这个cmd.exe，切记以管理员身份运行。

在指令终端输入：

```shell
pip install gym
```

### Linux

Anaconda创建虚拟环境 conda create –-name 你要创建的名字 python=版本号，例如

```
conda create –-name gymlab python=3.5
```

操作完此步之后，会在anaconda3/envs文件夹下多一个gymlab。Python3.5就在gymlab下得lib文件夹中。

开一个新的终端激活虚拟环境

```
source activate gymlab
```

下载gym

```
git clone [openai/gym](https://github.com/openai/gym.git)
```

安装gym，会装一系列的库，如果报错可以先安装依赖项，键入命令sudo apt-get install -y python-numpy python-dev cmake zlib1g-dev libjpeg-dev xvfb libav-tools xorg-dev python-opengl libboost-all-dev libsdl2-dev swig

```
cd gym
pip install –e '.[all]'
```

装完后可以将你的gym安装文件的目录写到环境变量中，打开.bashrc文件，在末尾加入语句：

```
export PYTHONPATH= [your gymPath]：$PYTHONPATH
```

### 验证安装成功

```python
import gym env = gym.make('CartPole-v0') 
env.reset() 
for _ in range(1000):    
    env.render()    
    env.step(env.action_space.sample())
```



## Gym的使用

使用gym编写自己的Agent代码，需要在在Agent类中声明一个env变量，指向对应的环境类，个体使用自己的代码产生一个行为，将该行为送入env的step方法中，可通过 render()显示图像

重要的四个函数

- env = gym.make('环境名');
- env.reset()
- env.render()
- env.step()

### gym.make('环境名')：建立环境类对象

在自己的代码中建立环境类对象呢

1. 在gym库里注册了的对象，你只要使用下面的语句：

```python3
import gym
env = gym.make("registered_env_name")
```

其中不同的环境类有不同的注册名，只要把make方法内的字符串改成对应的环境名就可以了。

2. 使用自己编写的未注册的环境类，这种很简单，同一般的建立对象的语句没什么区别：

```python3
env = MyEnvClassName()
```

### reset()：重新初始化函数

在强化学习算法中，智能体需要一次次地尝试，累积经验，然后从经验中学到好的动作。一次尝试我们称之为一条轨迹或一个episode. 每次尝试都要到达终止状态. 一次尝试结束后，智能体需要从头开始，这就需要智能体具有重新初始化的功能。函数reset()就是这个作用。

reset()的源代码为：

```python
def _reset()
    self.state = self.np_random.uniform(low=-0.05, high=0.05, size=(4,))  #利用均匀随机分布初试化环境的状态
    self.steps_beyond_done = None  #设置当前步数为None
    return np.array(self.state)  #返回环境的初始化状态
```

### render()：图像引擎

render()函数在这里扮演图像引擎的角色。一个仿真环境必不可少的两部分是物理引擎和图像引擎。物理引擎模拟环境中物体的运动规律；图像引擎用来显示环境中的物体图像。其实，对于强化学习算法，该函数可以没有。但是，为了便于直观显示当前环境中物体的状态，图像引擎还是有必要的。另外，加入图像引擎可以方便我们调试代码。

### step()：物理引擎

该函数在仿真器中扮演物理引擎的角色。它描述了智能体与环境交互的所有信息，是环境文件中最重要的函数。在该函数中，一般利用智能体的运动学模型和动力学模型计算下一步的状态和立即回报，并判断是否达到终止状态。

- 输入：动作a

- 输出：下一步状态，立即回报，是否终止，调试项。

```python3
state, reward, is_done, info = env.step(a)
```

state 是一个元组或numpy数组，其提供的信息维度应与观测空间的维度一样、每一个维度的具体指在制定的low与high之间，保证state信息符合这些条件是env类的_step方法负责的事情。

reward 则是根据环境的动力学给出的即时奖励，它就是一个数值。

is_done 是一个布尔变量，True或False，你可以根据具体的值来安排个体的后续动作。

info 提供的数据因环境的不同差异很大，通常它的结构是一个字典：

```python3
{"key1":data1,"key2":data2,...}
```



### 例

```python
import gym import time 
env = gym.make('CartPole-v0')   #创造环境 
observation = env.reset()   #初始化环境，observation为环境状态
count = 0 
for t in range(100):    
    action = env.action_space.sample()  #随机采样动作    
    observation, reward, done, info = env.step(action)  #与环境交互，获得下一步的时刻的观察值，奖励    
    if done:                     
        break    
    env.render()         #绘制场景    
    count+=1    
    time.sleep(0.2)      #每次等待0.2s 
print(count)             #打印该次尝试的步数
```



## 自定义环境类

gym库的核心在文件core.py里，里面定义了两个最基本的类Env和Space。前者是所有环境类的基类，后者是所有空间类的基类。

### Env类

自定义类的时候继承这个Env基类，并重写里面的一些方法，就可以实现自己的环境类了

要实现的方法有：

- **\_reset(self)**：初始化，开启个体与环境交互前调用该方法，确定个体的初始状态以及其他可能的一些初始化设置
- **\_step(self, action)**：物理引擎（重要部分），确定个体的下一个状态、奖励信息、是否Episode终止，以及一些额外的信息
- **\_render(self, mode='human', close=False)**：图像引擎，如果需要将个体与环境的交互以动画的形式展示出来的话，需要重写该方法。简单的UI设计可以用gym包装好了的pyglet方法来实现，这些方法在rendering.py文件里定义。具体使用这些方法进行UI绘制需要了解基本的OpenGL编程思想和接口
- **\_seed(self, seed=None)** ：设置随机数种子
- **\_close（可选）**：可以不实现

要设置的参数有：

- **action_space 动作空间**：一个描述所有有效动作的Space对象
- **observation_space 观察空间**：一个描述所有有效观察的Space对象
- **reward_range 奖励范围**：一个包含最大最小可能奖励的元组

环境基类的一段解释：

```python3
class Env(object):
    """The main OpenAI Gym class. It encapsulates an environment with
    arbitrary behind-the-scenes dynamics. An environment can be
    partially or fully observed.
    The main API methods that users of this class need to know are:
        step
        reset
        render
        close
        seed
    When implementing an environment, override the following methods
    in your subclass:
        _step
        _reset
        _render
        _close
        _seed
    And set the following attributes:
        action_space: The Space object corresponding to valid actions
        observation_space: The Space object corresponding to valid observations
        reward_range: A tuple corresponding to the min and max possible rewards
    Note: a default reward range set to [-inf,+inf] already exists. Set it if you want a narrower range.
    The methods are accessed publicly as "step", "reset", etc.. The
    non-underscored versions are wrapper methods to which we may add
    functionality over time.
    """

    # Override in SOME subclasses
    def _close(self):
        pass

    # Set these in ALL subclasses
    action_space = None
    observation_space = None

    # Override in ALL subclasses
    def _step(self, action): raise NotImplementedError
    def _reset(self): raise NotImplementedError
    def _render(self, mode='human', close=False): return
    def _seed(self, seed=None): return []
```

### Space类

用来描述空间的，比如行为空间，状态空间等。从Space基类衍生出几个常用的空间类，其中最主要的是**Discrete类**和**Box**类。前者对应于一维离散空间，后者对应于多维连续空间。它们既可以应用在行为空间中，也可以用来描述状态空间，具体怎么用看问题本身。

例如：

如果我要描述上篇提到的一个4*4的格子世界，其一共有16个状态，每一个状态只需要用一个数字来描述，这样我可以把这个问题的状态空间用Discrete(16)对象来描述就可以了。

对于另外一个经典的小车爬山的问题，小车的状态是用两个变量来描述的，一个是小车对应目标旗杆的水平距离，另一个是小车的速度（是沿坡度切线方向的速率还是速度在水平方向的分量这个没仔细研究），因此环境要描述小车的状态需要2个连续的变量。由于描述小车的状态数据对个体完全可见，因此小车的状态空间即是小车的观测空间，此时再用Discrete来描述就不行了，要用Box类，Box空间可以定义多维空间，每一个维度可以用一个最低值和最大值来约束。同时小车作为个体可以执行的行为只有3个：左侧加速、不加速、右侧加速。因此行为空间可以用Discrete来描述。最终，该环境类的观测空间和行为空间描述如下：

```python
self.min_position = -1.2
self.max_position = 0.6
self.max_speed = 0.07
self.goal_position = 0.5 
self.low = np.array([self.min_position, -self.max_speed])
self.high = np.array([self.max_position, self.max_speed])
self.action_space = spaces.Discrete(3)  #一个参数n
self.observation_space = spaces.Box(self.low, self.high)  #每个维度的最小最大值
```

