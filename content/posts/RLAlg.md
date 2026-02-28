+++
date = '2026-02-27T16:00:00+08:00'
draft = false
title = '强化学习原理与核心算法'
description = '强化学习原理及其算法介绍'
tags = ['强化学习', '教程']
type = "posts"
+++

## 1. 问题的本质

强化学习（RL）解决的核心问题是：**一个智能体（agent）在与环境交互的过程中，如何学会做出一系列决策，使得长期累积回报最大化。**

这和监督学习有本质区别——没有人告诉你"正确答案"是什么，你只能通过试错（trial and error）获得奖励信号，而且当前的决策会影响未来的状态。

---

## 2. 马尔可夫决策过程（MDP）

RL 的数学框架是 **MDP**，定义为一个五元组 $(S, A, P, R, \gamma)$：

- $S$：状态空间（所有可能的状态）
- $A$：动作空间（所有可能的动作）
- $P(s'|s,a)$：状态转移概率——在状态 $s$ 下执行动作 $a$，转移到状态 $s'$ 的概率
- $R(s,a,s')$：奖励函数——执行转移后获得的即时奖励
- $\gamma \in [0,1)$：折扣因子

**"马尔可夫"的含义**：未来只取决于当前状态，与历史无关。即 $P(s_{t+1}|s_t, a_t, s_{t-1}, a_{t-1}, \ldots) = P(s_{t+1}|s_t, a_t)$。这个假设极其重要——它意味着"状态"必须包含做决策所需的全部信息。

### 折扣因子 $\gamma$ 的直觉

$\gamma$ 控制智能体多"远视"。$\gamma=0$ 意味着只看眼前利益，$\gamma \to 1$ 意味着非常在意长远回报。同时 $\gamma < 1$ 保证了无限时间步的累积回报是有界的（数学上需要这个来保证收敛）。

---

## 3. 策略、回报、价值函数

**策略（Policy）** $\pi(a|s)$ 是一个从状态到动作的映射（可以是确定性的，也可以是随机的）。它完整定义了智能体的行为。

**累积折扣回报（Return）**：从时刻 $t$ 开始的总回报为

$$ G_t = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \cdots = \sum_{k=0}^{\infty} \gamma^k R_{t+k+1} $$

这就是我们想最大化的东西。注意 $G_t$ 是一个**随机变量**——因为未来的状态转移和动作选择都有随机性。

### 状态价值函数（State Value Function）

$$V^\pi(s) = \mathbb{E}_\pi\left[G_t \mid s_t = s\right]$$

含义：从状态 $s$ 出发，**按照策略 $\pi$ 行动**，期望能获得多少累积回报。

### 动作价值函数（Action Value Function / Q 函数）

$$Q^\pi(s, a) = \mathbb{E}_\pi\left[G_t \mid s_t = s, a_t = a\right]$$

含义：在状态 $s$ 下执行动作 $a$，**之后再按照策略 $\pi$**，期望获得多少累积回报。

两者的关系：

$$V^\pi(s) = \sum_{a} \pi(a|s) \, Q^\pi(s, a)$$

（对所有可能的动作，按策略概率加权求和）

---

## 4. Bellman 方程——RL 的基石

### Bellman 期望方程

对 $V^\pi$，通过将 $G_t$ 展开一步：

$$ V^\pi(s) = \sum_{a} \pi(a|s) \sum_{s'} P(s'|s,a) \left[ R(s,a,s') + \gamma \, V^\pi(s') \right] $$

**逐项解读**：
- 对所有动作 $a$ 按策略 $\pi(a|s)$ 加权
- 对每个动作，考虑所有可能的下一状态 $s'$（按转移概率 $P$ 加权）
- 到达 $s'$ 时获得即时奖励 $R$，再加上从 $s'$ 出发的未来价值（打了 $\gamma$ 折扣）

这个方程说的是：**当前状态的价值 = 即时奖励 + 折扣后的下一状态价值**。看似简单，但这个递归结构是几乎所有 RL 算法的根基。

对 $Q^\pi$ 也有类似的 Bellman 方程：

$$ Q^\pi(s,a) = \sum_{s'} P(s'|s,a) \left[ R(s,a,s') + \gamma \sum_{a'} \pi(a'|s') Q^\pi(s', a') \right] $$

### Bellman 最优方程

我们的目标是找到**最优策略** ${\pi}^*$，对应的最优价值函数 $V^*$ 和 $Q^*$：

$$ V^*(s) = \max_{a} \sum_{s'} P(s'|s,a) \left[ R(s,a,s') + \gamma \, V^*(s') \right] $$

$$ Q^*(s,a) = \sum_{s'} P(s'|s,a) \left[ R(s,a,s') + \gamma \max_{a'} Q^*(s', a') \right] $$

区别在于：期望方程里是"按策略 $\pi$ 加权"，最优方程里是"取 max"。一旦你有了 $Q^*$，最优策略就是 $\pi^*(s) = \arg\max_a Q^*(s,a)$——贪心地选 Q 值最大的动作。

---

## 5. 动态规划方法（已知模型时）

如果你知道 $P$ 和 $R$（即"模型已知"），可以直接用动态规划求解。

**策略迭代（Policy Iteration）**：
1. **策略评估**：给定固定策略 $\pi$，反复用 Bellman 期望方程更新 $V^\pi$，直到收敛
2. **策略改进**：对每个状态，贪心地选 $\pi'(s) = \arg\max_a Q^\pi(s,a)$
3. 重复直到策略不再变化

**价值迭代（Value Iteration）**：直接反复应用 Bellman 最优方程

$$V(s) \leftarrow \max_a \sum_{s'} P(s'|s,a) \left[ R(s,a,s') + \gamma \, V(s') \right]$$

直到收敛，然后从 $V^*$ 提取最优策略。

但在现实问题中，状态空间巨大（或连续）、转移概率未知，动态规划不可行。这引出了 RL 的核心方法。

---

## 6. 无模型方法：从采样中学习

### 6.1 蒙特卡洛方法（Monte Carlo）

核心思想：不用方程求期望，而是**直接采样大量轨迹，用平均回报来估计价值**。

1. 按策略 $\pi$ 走完一整条轨迹 $(s_0, a_0, r_1, s_1, a_1, r_2, \ldots)$
2. 对轨迹中每个状态-动作对，计算从该处到轨迹结束的实际回报 $G_t$
3. 更新：$Q(s,a) \leftarrow Q(s,a) + \alpha \left[ G_t - Q(s,a) \right]$

这里 $\alpha$ 是学习率，$G_t - Q(s,a)$ 是"误差"——实际回报和当前估计之间的差。

**优点**：不需要模型，无偏估计。
**缺点**：方差大（一条轨迹的回报波动很大），而且必须等整个 episode 结束才能更新。

### 6.2 时序差分学习（Temporal Difference, TD）

TD 方法是 RL 的关键突破。核心思想是**不等到 episode 结束，每走一步就更新**：

$$V(s_t) \leftarrow V(s_t) + \alpha \left[ r_{t+1} + \gamma V(s_{t+1}) - V(s_t) \right]$$

方括号内称为 **TD 误差** $\delta_t = r_{t+1} + \gamma V(s_{t+1}) - V(s_t)$。

**关键洞察**：用 $r_{t+1} + \gamma V(s_{t+1})$ 作为 $G_t$ 的估计（这叫 **bootstrapping**——用自己的估计来更新自己的估计）。MC 用真实回报，方差大但无偏；TD 用 bootstrap 估计，方差小但有偏。实践中 TD 通常更好。

### 6.3 Q-Learning（Off-Policy TD）

Q-Learning 是最经典的 RL 算法之一：

$$Q(s_t, a_t) \leftarrow Q(s_t, a_t) + \alpha \left[ r_{t+1} + \gamma \max_{a'} Q(s_{t+1}, a') - Q(s_t, a_t) \right]$$

注意这里用的是 $\max_{a'}$——这意味着**无论实际采用什么策略来探索**（behavior policy），更新时总是朝着最优策略的方向。这就是 **off-policy** 的含义：探索用的策略和要学习的目标策略可以不同。

探索时一般用 **$\epsilon$-greedy**：以 $1-\epsilon$ 概率选当前最优动作，$\epsilon$ 概率随机探索。

### 6.4 SARSA（On-Policy TD）

$$Q(s_t, a_t) \leftarrow Q(s_t, a_t) + \alpha \left[ r_{t+1} + \gamma Q(s_{t+1}, a_{t+1}) - Q(s_t, a_t) \right]$$

和 Q-Learning 的区别：这里用的是**实际执行的下一动作** $a_{t+1}$（而非 $\max$）。名字 SARSA 就来自 $(s_t, a_t, r_{t+1}, s_{t+1}, a_{t+1})$。SARSA 是 on-policy 的——学习的就是自己正在执行的策略。

---

## 7. 函数逼近与深度 RL

当状态空间巨大（如图像像素），不可能给每个 $(s,a)$ 存一个 Q 值表。解决方案：**用参数化函数来近似**。

### 7.1 Deep Q-Network（DQN）

用神经网络 $Q(s,a;\theta)$ 来近似 $Q^*(s,a)$。损失函数为：

$$L(\theta) = \mathbb{E}\left[ \left( r + \gamma \max_{a'} Q(s', a'; \theta^-) - Q(s, a; \theta) \right)^2 \right]$$

其中 $\theta^-$ 是 **目标网络**（target network）的参数——它是 $\theta$ 的一个延迟拷贝，定期同步。

DQN 的两个关键技巧：
1. **经验回放（Experience Replay）**：将 $(s,a,r,s')$ 存入缓冲区，训练时随机采样 mini-batch。打破了时间相关性，使训练更稳定。
2. **目标网络**：避免"用自己更新自己"造成的不稳定振荡。

### 7.2 为什么需要策略梯度

DQN 有根本性限制：它只能处理**离散动作空间**（因为需要对动作取 max）。对于连续动作（机器人关节角度、油门大小），需要另一条路——直接优化策略。

---

## 8. 策略梯度方法

### 8.1 核心定理

直接参数化策略 $\pi_\theta(a|s)$（比如用神经网络输出动作概率），优化目标：

$$ J(\theta) = \mathbb{E}_{\tau\sim \pi_{\theta}} \left[ \sum_{t=0}^{T} \gamma^{t} r_t \right] $$


**策略梯度定理**（这是 RL 最重要的定理之一）：

$$ \nabla_\theta J(\theta) = \mathbb{E}_{\pi_\theta} \left[ \nabla_\theta \log \pi_\theta(a_t|s_t) \cdot G_t \right] $$

**逐项拆解**：
- $\nabla_\theta \log \pi_\theta(a_t|s_t)$：策略关于参数的梯度方向——"参数怎么调，才能让动作 $a_t$ 在状态 $s_t$ 下更可能被选中"
- $G_t$：从该时刻起的实际回报——作为"权重"
- 两者相乘的直觉：**如果一个动作带来了高回报，就增大它被选中的概率；如果回报差，就减小**

这就是 **REINFORCE** 算法。问题是 $G_t$ 方差很大。

### 8.2 用基线降低方差

一个关键改进是减去一个**基线** $b(s_t)$（通常取 $V(s_t)$）：

$$\nabla_\theta J(\theta) = \mathbb{E}_{\pi_\theta} \left[ \nabla_\theta \log \pi_\theta(a_t|s_t) \cdot (G_t - b(s_t)) \right]$$

可以证明，减去只与状态相关的基线**不改变梯度的期望**（无偏），但能大幅降低方差。

当 $b(s_t) = V(s_t)$ 时，$G_t - V(s_t)$ 称为**优势函数（Advantage）**$A(s_t, a_t)$——衡量"这个动作比平均水平好多少"。这就引出了 Actor-Critic 方法。

---

## 9. Actor-Critic 方法

同时维护两个网络：
- **Actor**（策略网络 $\pi_\theta$）：决定采取什么动作
- **Critic**（价值网络 $V_\phi$ 或 $Q_\phi$）：评估当前状态/动作的好坏

Critic 用 TD 方法更新：

$$\phi \leftarrow \phi - \alpha_\phi \nabla_\phi \left( r + \gamma V_\phi(s') - V_\phi(s) \right)^2$$

Actor 用策略梯度更新，以 TD 误差 $\delta = r + \gamma V_\phi(s') - V_\phi(s)$ 作为优势的近似：

$$\theta \leftarrow \theta + \alpha_\theta \nabla_\theta \log \pi_\theta(a|s) \cdot \delta$$

### A2C/A3C

**A2C（Advantage Actor-Critic）**：用多个并行环境收集数据，同步更新。
**A3C**：异步版本，每个 worker 独立更新共享参数。

### GAE（Generalized Advantage Estimation）

实践中优势函数的估计可以在偏差和方差之间权衡。定义 TD 误差 $\delta_t = r_t + \gamma V(s_{t+1}) - V(s_t)$，则 GAE 为：

$$\hat{A}_t^{\text{GAE}(\gamma,\lambda)} = \sum_{l=0}^{\infty} (\gamma\lambda)^l \delta_{t+l}$$

$\lambda = 0$ 时退化为单步 TD（低方差高偏差），$\lambda = 1$ 时退化为 MC（高方差低偏差）。$\lambda \approx 0.95$ 通常效果很好。

---

## 10. PPO（Proximal Policy Optimization）

PPO 是目前实践中最常用的策略梯度算法（OpenAI 训练 ChatGPT 的 RLHF 用的就是 PPO）。

核心问题：策略梯度的步长很难控制——步子太大策略可能崩溃且不可逆。PPO 的解决方案是**限制每次更新的幅度**。

定义概率比：

$$r_t(\theta) = \frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{\text{old}}}(a_t|s_t)}$$

PPO-Clip 的目标函数：

$$L^{\text{CLIP}}(\theta) = \mathbb{E}_t \left[ \min \left( r_t(\theta) \hat{A}_t, \; \text{clip}(r_t(\theta), 1-\epsilon, 1+\epsilon) \hat{A}_t \right) \right]$$

**逐项解读**：
- $r_t(\theta)$：新旧策略之间的概率比。如果 $r_t = 1$，策略没变
- $\hat{A}_t$：优势估计（通常用 GAE）
- $\text{clip}(r_t, 1-\epsilon, 1+\epsilon)$：将概率比截断在 $[1-\epsilon, 1+\epsilon]$ 内（$\epsilon$ 通常取 0.1~0.2）
- 取 $\min$ 的意图：
  - 如果 $\hat{A}_t > 0$（好动作），$r_t$ 增大是好的，但超过 $1+\epsilon$ 后被截断——**防止过度增大好动作的概率**
  - 如果 $\hat{A}_t < 0$（差动作），$r_t$ 减小是好的，但低于 $1-\epsilon$ 后被截断——**防止过度压缩差动作的概率**

本质上这是一个"信赖域"的简化实现：每次更新保证策略不会变化太大。

---

## 11. 连续动作空间的处理

对于连续动作（如机器人控制），策略通常参数化为高斯分布：

$$\pi_\theta(a|s) = \mathcal{N}(\mu_\theta(s), \sigma_\theta^2(s))$$

神经网络输出均值 $\mu$ 和标准差 $\sigma$，动作从中采样。训练时 $\sigma$ 较大（多探索），随训练推进逐渐减小。

### DDPG（Deep Deterministic Policy Gradient）

针对连续动作的 off-policy 方法，结合了 DQN 和 Actor-Critic：
- Critic $Q_\phi(s,a)$ 的目标：$y = r + \gamma Q_{\phi^-}(s', \mu_{\theta^-}(s'))$
- Actor $\mu_\theta(s)$ 的目标：$\max_\theta \mathbb{E}[Q_\phi(s, \mu_\theta(s))]$
- 使用目标网络做软更新：$\theta^- \leftarrow \tau\theta + (1-\tau)\theta^-$

### SAC（Soft Actor-Critic）

SAC 引入了**最大熵**目标——不仅最大化回报，还最大化策略的熵：

$$J(\pi) = \sum_t \mathbb{E}\left[ r_t + \alpha \mathcal{H}(\pi(\cdot|s_t)) \right]$$

其中 $\mathcal{H}(\pi) = -\sum_a \pi(a|s)\log\pi(a|s)$ 是熵，$\alpha$ 是温度系数。

这鼓励智能体在回报相近时选择**更随机的策略**，带来更好的探索性和鲁棒性。SAC 是目前连续控制领域最常用的算法之一。

---

## 12. 探索-利用困境

这是 RL 的根本性挑战。一些重要方法：

- **$\epsilon$-greedy**：最简单，但不考虑不确定性
- **UCB（Upper Confidence Bound）**：选择 $a = \arg\max_a \left[ Q(s,a) + c\sqrt{\frac{\ln t}{N(s,a)}} \right]$——越少被尝试的动作，bonus 越大
- **好奇心驱动（Intrinsic Motivation）**：给预测误差高的状态额外奖励，鼓励探索"新奇"区域
- **后验采样/Thompson Sampling**：维护 Q 值的不确定性分布，从中采样决策

---

## 13. 从 RL 到 RLHF（与大语言模型的联系）

RLHF 将上述框架应用于语言模型微调：
- **状态**：对话上下文
- **动作**：生成的 token 序列
- **奖励**：由人类偏好训练的奖励模型给出
- **策略**：语言模型本身 $\pi_\theta$
- 用 PPO 优化，同时加一个 KL 惩罚防止偏离原始预训练模型太远：

$$\text{reward} = R_\phi(\text{response}) - \beta \, D_{\text{KL}}(\pi_\theta \| \pi_{\text{ref}})$$

---

## 学习路线建议

1. **先吃透 Bellman 方程和 TD 学习**——这是一切的基础
2. **实现 Q-Learning 和 SARSA**（在 GridWorld 等简单环境上）
3. **理解策略梯度定理的推导**——搞清楚 $\nabla \log \pi$ 这个"log-derivative trick"为什么成立
4. **实现 PPO**（在 CartPole/MuJoCo 上）
5. 推荐资源：Sutton & Barto 的 *Reinforcement Learning: An Introduction*（免费在线版），以及 David Silver 的 UCL 课程