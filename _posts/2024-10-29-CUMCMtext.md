---
layout: single
title: "CUMCM State Prize Travelogues"
categories: [yoka]
last_modified_at: 2024-10-29
excerpt: " WE've touched the sky."
header:
  teaser: https://static.runoob.com/images/mix/life-code-typography-hd-wallpaper-1920x1080-7168.jpg
---


近日，我院建模团队在全国大学生数学建模竞赛中以出色的表现荣获今年学校建模比赛的最佳成绩，送评国家一等奖，代表学校赴广州参加现场答辩，充分展现了学院学子的综合实力和团队合作精神。  

#### 从零开始，逐梦国赛  

本次参赛队伍由Yae同学、N同学和Lilis同学组成，三人不仅是同班同学，更是一群志同道合的伙伴。团队从组建之初便定下了目标，力求在国赛中脱颖而出，冲刺国奖。备赛期间，三位同学以饱满的热情和高度的责任感投入到训练中。从数学理论的学习到算法优化的实践，再到实际问题的建模与分析，三位同学不畏艰难，逐步积累经验，为国赛的挑战做好了万全准备。  

Yae同学回忆道：“备赛的过程是艰苦的，但每次我们攻克一道难题，完成一篇论文初稿，心里都充满成就感。”L同学补充道：“与队友们共同奋斗的日子最难忘，大家相互鼓励、不断突破，真正感受到了团队的力量。”  

#### 沉浸比赛，三天奋战

国赛期间，队伍面临三天时间完成从选题到论文撰写的全流程挑战。团队选择的B题是一项基于企业生产决策的建模问题，考察参赛队伍在多状态转移、优化决策等方面的综合能力。参赛队伍需建立合理的生产决策模型，兼顾成本控制与收益最大化。

三位同学紧密协作，分工明确：YAE同学专注于编程实现与算法设计，N同学负责模型建立与理论推导，Li同学则承担论文撰写与结构优化的任务。

N同学感慨道：“这个过程充满挑战，但我们思路清晰，每次找到突破点都让人充满成就感。”

#### 全心投入，感恩成长

比赛结束后，三位同学表示，参赛不仅让他们收获了荣誉，更让他们在实践中得到了难以复制的成长。L同学说：“没有想到会取得这么好的成绩！所有的认真与努力都得到了认可。”  

#### 补充

本次优异成绩不仅是团队成员个人努力的体现，更是大数据学院培养创新型人才的重要成果。未来，学院将继续致力于为学生提供更多的实践机会和成长平台，期待更多学子在各类赛事中大放异彩，为学校争光添彩。  


> 大数据学院  

---

#### 通稿結束,以下是部分的英文化論文以及經驗分享

**A study of production strategies based on Markov decision processes**

**Abstract**

In response to problem one.In order to design a sampling and inspection scheme with as few inspections as possible, this paper is based on the **characteristics of the binary classification problem** of spare parts defective rate, which describes the defective sampling process using the binomial distribution, and approximates the binomial distribution by using the normal distribution, which is analyzed in two scenarios respectively. Model one, the **normal test model**, is developed. The sampling program metrics are determined by the minimum sample size and rejection/acceptance thresholds derived from the model, and the model effect is indexed by the correlation between sample size and rejection/acceptance capacity. Another model II, the **Ordinal Likelihood Ratio Test Model**, is developed. The dynamic sampling scheme is determined by the likelihood ratio, upper and lower thresholds, and the model effect is introduced into the **operating characteristic curve and average sample size curve** to assess and improve the decision-making reliability of the sampling and testing scheme.

For problem two, a decision-making model is established for the production process of the enterprise. This paper first establishes the **objective function** as the decision-making basis and indicators. Then, considering that different stages of the production process can be regarded as different state transitions, **Markov decision process** is selected for model construction and **Dynamic Programming** as the implementation algorithm. The implementation process is mainly through setting different stages of the production process as different state variables, adding state transfer equations and reward functions for the dynamic planning of the decision-making process, and constructing a dynamic planning table to make **bottom-up** optimization decisions to achieve the maximization of the enterprise's revenues and minimization of costs. After that, according to the specific information of the optimal solution for each of the six cases, and exhaustively enumerate the benefits and costs of all the decision-making solutions in each case to repeat the verification of the optimal decision-making solution, in order to prove that the resulting decision-making solution is optimal.

For problem three, in order to get the general case of the production situation and give a special case of the decision-making program, this paper is based on problem two used Markov decision process based dynamic programming algorithm, adding **value iteration algorithm** for optimal decision-making. Firstly, we introduce the factors of multiple processes, spare parts and defective rate of semi-finished products **update the objective function**; then **expand the state space, actions, and state transfer equations with the reward function**, calculate the optimal action for each state through value iteration repeatedly, and adjust the strategy according to the future value of the state and the instantaneous rewards until we find the optimal strategy; finally, we carry out the model sensitivity analysis to evaluate the decision scheme of the model.

For problem four, based on the sampling test method in problem one, problem two and three are re-solved for the decision scheme. In this paper, the ratio of the minimum sample size obtained from the normality test model in Problem I to the rejection/acceptance threshold is fitted by **linear regression** , which yields the fitted subprime rate, and the model is updated to re-obtain the optimal decision scheme in Problem II and III.

**Keywords:** Markov Decision Process Dynamic Planning Value Iteration Algorithm Sequential Probability Ratio Testing Model

---

```python

p0 = 0.10  
alpha_reject = 0.05 
alpha_accept = 0.10  
z_alpha_reject = stats.norm.ppf(1 - alpha_reject)
z_alpha_accept = stats.norm.ppf(1 - alpha_accept)

def calculate_threshold(n, p0, z_alpha):
    return math.ceil((z_alpha * np.sqrt(p0 * (1 - p0) / n) + p0) * n)

results_reject = []
results_accept = []

sample_sizes_reject = range(100, 300, 20)
for n in sample_sizes_reject:
    threshold = calculate_threshold(n, p0, z_alpha_reject)
    results_reject.append(threshold)

sample_sizes_accept = range(100, 300, 20)
for n in sample_sizes_accept:
    threshold = calculate_threshold(n, p0, z_alpha_accept)
    results_accept.append(threshold)

df_reject = pd.DataFrame({
    '样本量 n': sample_sizes_reject,
    '拒收阈值': results_reject
})

df_accept = pd.DataFrame({
    '样本量 n': sample_sizes_accept,
    '接收阈值': results_accept
})

df_reject = df_reject.style.set_caption("拒收时阈值")
df_accept = df_accept.style.set_caption("接收时阈值")
display(df_reject)
display(df_accept)

p0 = 0.10  
p1 = 0.15  
alpha = 0.05 
beta = 0.10 
A = np.log((1 - beta) / alpha)
B = np.log(beta / (1 - alpha))
def sp_simu(p0, p1, lnA, lnB, true_p):
    n = 0
    sum_x = 0
    while True:
        n += 1
        x = np.random.rand() < true_p  
        sum_x += x
        lnLR = sum_x * np.log(p1 / p0) + (n - sum_x) * np.log((1 - p1) / (1 - p0))
        if lnLR >= lnA:
            decision = "Reject H0"
        elif lnLR <= lnB:
            decision = "Accept H0"
        else:
            continue
        return decision, n

T_p_ran = np.arange(0.05, 0.26, 0.02)
num_simulations = 10000
oc_curve = np.zeros_like(T_p_ran)
asn_curve = np.zeros_like(T_p_ran)

for i, true_p in enumerate(T_p_ran):
    decisions = []
    sample_sizes = np.zeros(num_simulations)
    for j in range(num_simulations):
        decision, n = sp_simu(p0, p1, A, B, true_p)
        decisions.append(decision)
        sample_sizes[j] = n
    oc_curve[i] = decisions.count("Accept H0") / num_simulations
    asn_curve[i] = np.mean(sample_sizes)

# 绘制OC/ASN
# 略

```

---

I was going to drink a Coke with the trophy, isn't that bold? CUMCM, enjoy Coke with the trophy.

It's all about the competition, ENJOY Coke.

When we first formed the team, we said we'd take the cup and drink it.

Wow, the team is excited. Everyone prepares for this, writing a question inside, writing a question at night, and finishing it in three days.

But in the end, we gave up hahahahahahaha, mainly because we didn't have the strength.
