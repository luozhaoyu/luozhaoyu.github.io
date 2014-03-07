# 数据挖掘中的推荐系统

zhaoyu.luo@guohead.com

---

# 大纲
* 数据挖掘与推荐系统是什么
* 如何推荐
* 数据挖掘的主要方法
* 协同过滤
* 存在的问题与对策
* 如何衡量推荐效果
* 具体业务实现

---
= data-y="400"
# 数据挖掘与推荐系统是什么
* 数据挖掘是[计算机科学](http://zh.wikipedia.org/wiki/%E6%95%B0%E6%8D%AE%E6%8C%96%E6%8E%98)：
    - 统计
    - 在线分析处理OLAP
    - 信息检索IR
    - 机器学习
    - 专家系统

---
= data-rotate="45"
# 如何推荐
* 向朋友咨询
* 自己的兴趣爱好
* 排行榜
* ???

---
# 主要方法
## 常见算法
* 基于邻域的算法（用户、物品）
* 隐语义模型（机器学习）（兴趣分类）[Latent Class Model](http://en.wikipedia.org/wiki/Latent_class_model)
* 图模型

## 视角
* **用户**
* **物品**

---
= data-scale="0.5"
# 效能
* 准确率 召回率 覆盖率 流行度
* 基于邻域的算法 22.28% 10.76% 18.84% 7.2545
* 隐语义模型 26.97% 13.01% 44.25% 6.7899
* 图模型 16.45% 7.95% 3.42% 7.6928

---
# 基于物品的协同过滤
* 推荐用户所喜欢物品中最相似的物品
* 业界应用最多[Amazon](http://www.amazon.cn/) Netflix Hulu YouTube
* 物品间的相似度（基于用户行为）
* 长尾物品丰富
* 用户的新行为一定会导致推荐结果实时变化

1. 物品间相似度
- 计算用户u对j的兴趣: Puj = Wji * Rui

---
= data-scale="0.5"
![Matrix](http://i.imgur.com/H5xpjnf.png)

---
= data-y="400"
# 存在的问题与对策
* 冷启动（额外信息、[TF-IDF](http://zh.wikipedia.org/wiki/TF-IDF)）
* 无法加载全部数据（[基于模型](http://en.wikipedia.org/wiki/Bayesian_networks)）


---
#  如何衡量推荐效果
* 用户满意度（反馈）
* 准确率 Precision Rate
* 召回率 Recall Rate
* 覆盖度（每个物品至少被展示一次）
* 多样性（兴趣间的差异）
* 新颖行（未知物品）
* *惊喜度、信任度、实时性、健壮性、商业目标……*

---
# 具体业务实现
* AB测试系统
* 数据的测试与训练

---
= data-scale="0.7"
# 个性化展示广告

* 研究最多的公司 Yahoo
* 最成功的公司 Facebook
* [ACM Recommender System](http://recsys.acm.org/)
