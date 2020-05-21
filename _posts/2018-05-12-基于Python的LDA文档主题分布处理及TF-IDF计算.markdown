---
layout:     post
title:      "基于Python的LDA文档主题分布处理及TF-IDF计算"
subtitle:   "这篇文章主要是讲述如何通过LDA处理文本内容TXT，并计算其文档主题分布及TF-IDF。"
date:       2018-05-12
author:     "Malcolm Suen"
header-img: "img/post-bg-2015.jpg"
header-mask:  0.3
catalog: true
tags:
    - Python
    - 机器学习
    - LDA
---

## 什么是LDA？

​	隐含狄利克雷分布（Latent Dirichlet Allocation）是一种文档主题生成模型，也称为一个三层贝叶斯概率模型，包含词、主题和文档三层结构。所谓生成模型，就是说，我们认为一篇文章的每个词都是通过“以一定概率选择了某个主题，并从这个主题中以一定概率选择某个词语”这样一个过程得到。文档到主题服从多项式分布，主题到词服从多项式分布。LDA是一种非监督机器学习技术，可以用来识别大规模文档集（document collection）或语料库（corpus）中潜藏的主题信息。

## 环境配置

下面核心代码里面需要的包都可以用`pip install`命令来安装，需要注意的是lda包安装需要10.0以上版本的pip。推荐使用Anaconda进行包管理，详情参见文章《[Anaconda入门指南](https://belikovvv.github.io/2018/01/14/Anaconda%E5%85%A5%E9%97%A8%E6%8C%87%E5%8D%97/)》。

## 参考文档

[官网文档](https://github.com/ariddell/lda)

[lda: Topic modeling with latent Dirichlet Allocation](http://pythonhosted.org/lda/)

[Getting started with Latent Dirichlet Allocation in Python - sandbox](http://chrisstrelioff.ws/sandbox/2014/11/13/getting_started_with_latent_dirichlet_allocation_in_python.html)

## 核心代码

```python
# coding=utf-8
import os
import sys
import numpy as np
import matplotlib
import scipy
import matplotlib.pyplot as plt
from sklearn import feature_extraction
from sklearn.feature_extraction.text import TfidfTransformer
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.feature_extraction.text import HashingVectorizer

if __name__ == "__main__":

    # 存储读取语料 一行预料为一个文档
    corpus = []
    for line in open('test.txt', 'r').readlines():
        # print line
        corpus.append(line.strip())
    # print corpus

    # 将文本中的词语转换为词频矩阵 矩阵元素a[i][j] 表示j词在i类文本下的词频
    vectorizer = CountVectorizer()
    print vectorizer

    X = vectorizer.fit_transform(corpus)
    analyze = vectorizer.build_analyzer()
    weight = X.toarray()

    print len(weight)
    print (weight[:5, :5])

    # LDA算法
    print 'LDA:'
    import numpy as np
    import lda
    import lda.datasets

    # 定义主题数量
    n_topics = 2
    # 定义迭代次数
    n_iter = 500
    model = lda.LDA(n_topics=n_topics, n_iter=n_iter, random_state=1)
    model.fit(np.asarray(weight))  # model.fit_transform(X) is also available
    # 主题-单词（topic-word）分布
    topic_word = model.topic_word_  # model.components_ also works

    # 文档-主题（Document-Topic）分布
    doc_topic = model.doc_topic_
    print("type(doc_topic): {}".format(type(doc_topic)))
    print("shape: {}".format(doc_topic.shape))

    # 输出前10篇文章最可能的Topic
    label = []
    for n in range(10):
        topic_most_pr = doc_topic[n].argmax()
        label.append(topic_most_pr)
        print("doc: {} topic: {}".format(n, topic_most_pr))

```

## 代码分析

### 输入输出

输入是test.txt文件，它是使用[Jieba分词](http://blog.csdn.net/eastmount/article/details/50256163)之后的文本内容，通常每行代表一篇文档。该文本内容原自博客：[文本分析之TFIDF/LDA/Word2vec实践](http://blog.csdn.net/vs412237401/article/details/50628239) 

```
新春 备 年货 ， 新年 联欢晚会  
新春 节目单 ， 春节 联欢晚会 红火  
大盘 下跌 股市 散户  
下跌 股市 赚钱  
金猴 新春 红火 新年  
新车 新年 年货 新春  
股市 反弹 下跌  
股市 散户 赚钱  
新年 , 看 春节 联欢晚会  
大盘 下跌 散户
```

输出则是这十篇文档的主题分布，shape: (10, 2)表示10篇文档，2个主题。
具体结果如下所示：

```
type(doc_topic): <type 'numpy.ndarray'>
shape: (10, 2)
doc: 0 topic: 0
doc: 1 topic: 0
doc: 2 topic: 1
doc: 3 topic: 1
doc: 4 topic: 0
doc: 5 topic: 0
doc: 6 topic: 1
doc: 7 topic: 1
doc: 8 topic: 0
doc: 9 topic: 1
```

同时可以调用`matplotlib.pyplot`输出了对应的文档主题分布图，实现代码如下：

```python
    #计算文档主题分布图  
    import matplotlib.pyplot as plt    
    f, ax= plt.subplots(6, 1, figsize=(8, 8), sharex=True)    
    for i, k in enumerate([0, 1, 2, 3, 8, 9]):    
        ax[i].stem(doc_topic[k,:], linefmt='r-',    
                   markerfmt='ro', basefmt='w-')    
        ax[i].set_xlim(-1, 2)     #x坐标下标  
        ax[i].set_ylim(0, 1.2)    #y坐标下标  
        ax[i].set_ylabel("Prob")    
        ax[i].set_title("Document {}".format(k))    
    ax[5].set_xlabel("Topic")  
    plt.tight_layout()  
    plt.show()    
```

可以看到主题Doc0、Doc1、Doc8分布于Topic0，它们主要描述主题新春；而Doc2、Doc3、Doc9分布于Topic1，主要描述股市。

![](https://ws1.sinaimg.cn/large/006MzNVDgy1fr8f62cg7nj318g18g405.jpg)LDA运行结果如下：

```
INFO:lda:n_documents: 10	#文档数
INFO:lda:vocab_size: 15		
INFO:lda:n_words: 36		#总词数
INFO:lda:n_topics: 2		#话题数
INFO:lda:n_iter: 500		#迭代次数
INFO:lda:<0> log likelihood: -192
INFO:lda:<10> log likelihood: -140
……
```

如果希望查询每个主题对应的问题词权重分布情况，代码实现如下：

```python
import matplotlib.pyplot as plt    
f, ax= plt.subplots(2, 1, figsize=(6, 6), sharex=True)    
for i, k in enumerate([0, 1]):         #两个主题  
    ax[i].stem(topic_word[k,:], linefmt='b-',    
               markerfmt='bo', basefmt='w-')    
    ax[i].set_xlim(-2,20)    
    ax[i].set_ylim(0, 1)    
    ax[i].set_ylabel("Prob")    
    ax[i].set_title("topic {}".format(k))    
    
ax[1].set_xlabel("word")    
    
plt.tight_layout()    
plt.show()  
```

运行结果如下图所示，共2个主题Topics，15个核心词汇。

![](https://ws1.sinaimg.cn/large/006MzNVDgy1fr8fqb5wj6j30xc0xcwfj.jpg)

### 计算主题TopN

通过word = vectorizer.get_feature_names()获取整个预料的词向量，其中TF-IDF对应的就是它的值。然后再获取其位置对应的关键词即可，代码中输出5个关键词，代码实现如下：

```python
    # 输出主题中的TopN关键词
    word = vectorizer.get_feature_names()
    for w in word:
        print w
    print topic_word[:, :3]
    # 获取每个topic下权重最高的n个单词
    n = 5
    for i, topic_dist in enumerate(topic_word):
        topic_words = np.array(word)[np.argsort(topic_dist)][:-(n + 1):-1]
        print(u'*Topic {}\n- {}'.format(i, ' '.join(topic_words)))
```

运行结果如下所示：

```
下跌 反弹 大盘 年货 散户 新年 新春 新车 春节 红火 联欢晚会 股市 节目单 赚钱 金猴
[[ 0.00049628  0.00049628  0.00049628]
 [ 0.24829721  0.0625387   0.1244582 ]]
*Topic 0
- 新春 新年 联欢晚会 红火 春节
*Topic 1
- 股市 下跌 散户 赚钱 大盘
```

### TF-IDF计算及词频TF计算

特征计算方法参考：[Feature Extraction - scikit-learn](http://scikit-learn.org/stable/modules/feature_extraction.html#text-feature-extraction)

代码实现如下：

```Python
    #计算TFIDF
    corpus = []
    
    #读取预料 一行预料为一个文档 
    for line in open('test.txt', 'r').readlines():
        #print line
        corpus.append(line.strip())
    #print corpus
    
    #将文本中的词语转换为词频矩阵 矩阵元素a[i][j] 表示j词在i类文本下的词频
    vectorizer = CountVectorizer()

    #该类会统计每个词语的tf-idf权值
    transformer = TfidfTransformer()

    #第一个fit_transform是计算tf-idf 第二个fit_transform是将文本转为词频矩阵
    tfidf = transformer.fit_transform(vectorizer.fit_transform(corpus))

    #获取词袋模型中的所有词语  
    word = vectorizer.get_feature_names()

    #将tf-idf矩阵抽取出来，元素w[i][j]表示j词在i类文本中的tf-idf权重
    weight = tfidf.toarray()

    #打印特征向量文本内容
    print 'Features length: ' + str(len(word))
    for j in range(len(word)):
        print word[j]

    #打印每类文本的tf-idf词语权重，第一个for遍历所有文本，第二个for便利某一类文本下的词语权重  
    for i in range(len(weight)):
        for j in range(len(word)):
            print weight[i][j],
        print '\n'
```

输出如下图所示，共统计处特征词15个，对应TF-IDF矩阵，共10行数据对应txt文件中的10个文档，每个文档15维数据，存储TF-IDF权重，这就可以通过10*15的矩阵表示整个文档权重信息。运行结果如下所示：

```
Features length: 15  
下跌 反弹 大盘 年货 散户 新年 新春 新车 春节 红火 联欢晚会 股市 节目单 赚钱 金猴  
  
0.0 0.0 0.0 0.579725686076 0.0 0.450929562568 0.450929562568 0.0 0.0 0.0 0.507191470855 0.0 0.0 0.0 0.0   
0.0 0.0 0.0 0.0 0.0 0.0 0.356735384792 0.0 0.458627428458 0.458627428458 0.401244805261 0.0 0.539503693426 0.0 0.0   
0.450929562568 0.0 0.579725686076 0.0 0.507191470855 0.0 0.0 0.0 0.0 0.0 0.0 0.450929562568 0.0 0.0 0.0   
0.523221265036 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.523221265036 0.0 0.672665604612 0.0   
0.0 0.0 0.0 0.0 0.0 0.410305398084 0.410305398084 0.0 0.0 0.52749830162 0.0 0.0 0.0 0.0 0.620519542315  
0.0 0.0 0.0 0.52749830162 0.0 0.410305398084 0.410305398084 0.620519542315 0.0 0.0 0.0 0.0 0.0 0.0 0.0  
0.482964462575 0.730404446714 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.482964462575 0.0 0.0 0.0   
0.0 0.0 0.0 0.0 0.568243852685 0.0 0.0 0.0 0.0 0.0 0.0 0.505209504985 0.0 0.649509260872 0.0   
0.0 0.0 0.0 0.0 0.0 0.505209504985 0.0 0.0 0.649509260872 0.0 0.568243852685 0.0 0.0 0.0 0.0   
0.505209504985 0.0 0.649509260872 0.0 0.568243852685 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0   
```



**感谢来自北京理工大学的杨秀璋同学，现任教于贵州财经大学信息学院；感谢陈炳宏同学的数据**。



## 附录

#### 微博热点话题检测运行结果

```
[[  1.02084567e-05   1.02084567e-05   1.02084567e-05]
 [  3.01539200e-03   1.50019503e-05   1.50019503e-05]
 [  1.52304365e-05   1.52304365e-05   1.52304365e-05]
 [  1.22013714e-05   1.22013714e-05   1.22013714e-05]
 [  1.49570732e-05   1.49570732e-05   1.49570732e-05]
 [  1.35947144e-05   1.35947144e-05   1.35947144e-05]
 [  1.30110073e-05   2.61521247e-03   1.30110073e-05]
 [  1.32173729e-05   1.32173729e-05   1.32173729e-05]
 [  1.27456728e-05   1.27456728e-05   1.27456728e-05]
 [  9.43770173e-06   9.43770173e-06   9.43770173e-06]
 [  1.49570732e-05   1.49570732e-05   1.49570732e-05]
 [  1.22013714e-05   1.22013714e-05   1.22013714e-05]
 [  1.16335885e-05   1.16335885e-05   1.16335885e-05]
 [  1.12539107e-05   1.12539107e-05   1.12539107e-05]
 [  1.41526791e-05   1.41526791e-05   1.41526791e-05]
 [  1.33230302e-05   1.33230302e-05   1.33230302e-05]
 [  1.55865208e-05   1.55865208e-05   1.55865208e-05]
 [  1.30790761e-05   1.30790761e-05   1.30790761e-05]
 [  9.89530765e-06   9.89530765e-06   9.89530765e-06]
 [  1.24909441e-05   1.24909441e-05   1.24909441e-05]
 [  1.28274199e-05   1.28274199e-05   1.28274199e-05]
 [  1.34665625e-05   1.34665625e-05   1.34665625e-05]
 [  1.22612129e-05   1.22612129e-05   1.22612129e-05]
 [  1.24134164e-05   1.24134164e-05   1.24134164e-05]
 [  1.39746779e-05   1.39746779e-05   1.39746779e-05]
 [  1.55865208e-05   1.55865208e-05   1.55865208e-05]
 [  1.53473096e-05   1.53473096e-05   1.53473096e-05]
 [  1.35211877e-05   1.35211877e-05   1.35211877e-05]
 [  1.35578514e-05   1.35578514e-05   1.35578514e-05]
 [  1.33586257e-05   1.33586257e-05   1.33586257e-05]
 [  1.42130248e-05   1.42130248e-05   1.42130248e-05]
 [  1.14079719e-05   1.14079719e-05   1.14079719e-05]
 [  1.13432700e-05   1.13432700e-05   1.13432700e-05]
 [  1.44805815e-05   1.44805815e-05   4.35865504e-03]
 [  1.53473096e-05   1.53473096e-05   1.53473096e-05]
 [  9.41105611e-06   9.41105611e-06   9.41105611e-06]
 [  1.32876239e-05   1.32876239e-05   1.32876239e-05]
 [  1.12286375e-05   1.12286375e-05   1.12286375e-05]
 [  1.40928437e-05   1.40928437e-05   1.40928437e-05]
 [  1.30962047e-05   1.30962047e-05   1.30962047e-05]]
*Topic 0
- 男子 打人 黑衣 发生 外卖 小哥
*Topic 1
- 医院 沙溢 城市 男孩 牙齿 心疼
*Topic 2
- 大风 孩子 女孩 小姐姐 一把 一点
*Topic 3
- 发布 购物 发现 实施 第一 卖场
*Topic 4
- 马克思 纪念 转发 求婚 收藏 大学生
*Topic 5
- 世界 生活 努力 可怕 患者 低于
*Topic 6
- 香港 日本 创新 疫苗 永远 刚果
*Topic 7
- 男子 山东 驾车 网友 路边 网络
*Topic 8
- 帕尔马 联赛 意甲 年月日 三年 恭喜
*Topic 9
- 篮球 赛季 季后赛 比赛 训练 勇士
*Topic 10
- 公司 男友 手机 分享 快递 图片
*Topic 11
- 一名 司机 货车 发现 女童 幸福
*Topic 12
- 药酒 道歉 秦东 女生 声明 医生
*Topic 13
- 顺丰 出售 教师 用户 宿舍 司机
*Topic 14
- 电影 复仇者 联盟 复联 灭霸 漫威
*Topic 15
- 孩子 父母 高温 朋友 转盘 十级
*Topic 16
- 租赁 警方 住房 记者 成都 市场
*Topic 17
- 书记 春风 事件 调查 广安 市委
*Topic 18
- 执法 警察 网友 教科书 操作 连续
*Topic 19
- 小时 生气 加班 事情 相当于 致命
*Topic 20
- 事故 武汉 身边 新闻 调查 死亡
*Topic 21
- 老人 司机 科技 网友 地方 刺激
*Topic 22
- 搞笑 幸福 一种 联想 红人周 喜欢
*Topic 23
- 美国 学生 高中 发生 国家 警方
*Topic 24
- 兰州 甘肃 惊喜 新区 集团 梦想
*Topic 25
- 真的 工作 教师资格 曝光 特别 舞蹈
*Topic 26
- 北京 猫咪 专辑 轮胎 这是 一张
*Topic 27
- 员工 老板 一场 哈哈哈 宝马 豪宅
*Topic 28
- 中国 小龙虾 德约 地区 科维奇 中方
*Topic 29
- 上海 明星 现身 粉丝 春天 颜值
*Topic 30
- 中国 规则 广州 无物 转发 维和
*Topic 31
- 保姆 野生 莫焕晶 纵火案 现场 一审
*Topic 32
- 足球 俱乐部 球员 世界杯 比赛 孩子
*Topic 33
- 王源 戛纳 电影节 主宰 礼物 喝水
*Topic 34
- 故事 链接 网页 流量 手机 发布
*Topic 35
- 古巴 乘客 坠毁 客机 遇难 时间
*Topic 36
- 母亲 女儿 儿子 赡养 支持 财产
*Topic 37
- 英雄 近日 烈士 暴走 孩子 中国
*Topic 38
- 活动 表情 真的 生活 警方 一部
*Topic 39
- 突发 丈夫 民警 急救 心梗 医生
type(doc_topic): <type 'numpy.ndarray'>
shape: (1943, 40)
doc: 0 topic: 37
doc: 1 topic: 31
doc: 2 topic: 25
doc: 3 topic: 30
doc: 4 topic: 38
doc: 5 topic: 17
doc: 6 topic: 18
doc: 7 topic: 6
doc: 8 topic: 29
doc: 9 topic: 24
```

#### 文档主题分布图

![](https://ws1.sinaimg.cn/large/006MzNVDly1frkgischgzj30m80m8t96.jpg)

#### 主题对应的问题词权重分布情况图

![](https://ws1.sinaimg.cn/large/006MzNVDgy1frkg1taeagj30go0goglv.jpg)