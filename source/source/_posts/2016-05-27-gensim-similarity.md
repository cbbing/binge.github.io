---
layout: post
title: gensim文档相似度判断
date: 2016-05-27 17:55:25
tags:
	- gensim
	- 主题模型
	- 相似度
	- 文档去重
category: 机器学习
comments: true
mathjax: true
---


在文本处理中，比如商品评论挖掘，有时需要了解每个评论分别和商品的描述之间的相似度，以此衡量评论的客观性。

文本相似度计算的需求始于搜索引擎，搜索引擎需要计算“用户查询”和爬下来的众多“网页”之间的相似度，从而把最相似的排在最前，返回给用户。

# 一、基本概念

##  TF-IDF

- TF：term frequency，词频

$$ 词频(TF) = 某个词在文章中的出现次数 $$
<!-- more -->

$$ 词频(TF) = \frac{某个词在文章中的出现次数}{文章的总次数} $$

$$ 词频(TF) = \frac{某个词在文章中的出现次数}{该文出现次数最多的词的出现次数} $$

- IDF：inverse document frequency，逆文档频率

$$ IDF = log(\frac{语料库的文档总数}{包含该词的文档数+1}) $$

- TF-IDF

$$ TF-IDF = 词频(TF) \times逆文档频率(IDF) $$

主要思想是：如果某个词或短语在一篇文章中出现的频率高，并且在其他文章中很少出现，则认为此词或者短语具有很好的类别区分能力，适合用来分类。


## TF-IDF计算步骤

- 第一步：把每个网页文本分词，称为词包（bag of words）

- 第二步：统计网页（文档）总数M

- 第三步：统计第一个网页次数N，计算第一个网页第一个词在该网页中出现的次数n，再找出该词在所有文档中出现的次数m。

则该词的tf-idf为：

$$ \frac{\frac{n}{N}}{\frac{m}{M}} $$

- 第四步：重复第三步，计算出一个网页所有词的tf-idf值。

- 第五步：重复第四步，计算出所有网页每个词的tf-idf值。



## SVD，奇异值分解（Singular value decomposition）

奇异值分解是一个有着明显的物理意义的一种方法，它可以将一个比较复杂的矩阵用更小更简单的几个子矩阵的相乘来表示，这些小矩阵描述的是矩阵的重要的特性。就像是描述一个人一样，给别人描述说这个人长得浓眉大眼，方脸，络腮胡，而且带个黑框的眼镜，这样寥寥的几个特征，就让别人脑海里面就有一个较为清楚的认识，实际上，人脸上的特征是有着无数种的，之所以能这么描述，是因为人天生就有着非常好的抽取重要特征的能力，让机器学会抽取重要的特征，SVD是一个重要的方法。

## LSI，浅层语义索引（Latent Semantic Indexing）


潜在语义索引，指的是通过海量文献找出词汇之间的关系。当两个词或一组词大量出现在一个文档中时，这些词就可以被认为是语义相关的。

潜在语义索引是一种用SVD（Singular Value Decomposition）奇异值分解方法获得在文本中术语和概念之间关系的索引和获取方法。该方法的主要依据是在相同文章中的词语一般有类似的含义。该方法可以从一篇文章中提取术语关系，从而建立起主要概念内容。

## 余弦相似度 (cosine similiarity)

$$ cos\theta=\frac{a^2+b^2-c^2}{2ab} $$

<img src="http://www.forkosh.com/mathtex.cgi?cos\theta=\frac{x_{1}x_{2} + y_{1}y_{2}}{\sqrt{x_{1}^2+y_{1}^2} \times \sqrt{x_{2}^2+y_{2}^2}}"/>

<img src="http://www.forkosh.com/mathtex.cgi?cos\theta=\frac{\sum_{i=1}^{n}(A_{i}\times B_{i})}{\sqrt{\sum_{i=1}^{n}(A_{i})^2}\times \sqrt{\sum_{i=1}^{n}(B_{i})^2}}=\frac{A\cdot B}{|A|\times |B|}" />

# 二、相似度计算步骤

1，处理用户查询

- 第一步：对用户查询进行分词

- 第二步： 根据网页库（文档）的数据，  计算用户查询中每个词的tf-idf值。



2，相似度的计算

使用余弦相似度来计算用户查询和每个网页之间的夹角。夹角越小，越相似。



# 三、gensim介绍

Gensim是一个相当专业的主题模型Python工具包。是一个用于主题建模、文档索引以及使用大规模语料数据的相似性检索。相比RAM，它能处理更多的输入数据。作者称它是“根据纯文本进行非监督性建模最健壮、最有效的、最让人放心的软件。”

*gensim安装： pip install gensim*



# 四、实现步骤

## 1，中文分词

以数据库中关于美联储的新闻6000条，为例。
![](http://7xo67b.com1.z0.glb.clouddn.com/2016-05-27/1.png)


首先对标题和内容进行分词。
标题分词的结果如下：
![](http://7xo67b.com1.z0.glb.clouddn.com/2016-05-27/2.png)


### Python代码

```
self.df 为pandas 的DataFrame结构
#self.df = pd.read_sql(sql, engine)
#df.columns: title, content, href, etc..

import re
regex = re.compile(ur"[^\u4e00-\u9f5aa-zA-Z0-9]") # 中英文和数字

def jieba_cut(self):
    """
    对标题和内容分词
    :return:
    """
    import jieba
    jieba.load_userdict('Data/userdict.txt')  # 自己准备用户词典，也可不指定

    # 1，去掉标点和特殊字符
    # 2，分词
    self.df['title_fenci'] = self.df['title'].apply(lambda x : '|'.join(jieba.cut(regex.sub('',x))))

```
## 2，去掉频率为1的词

```
def remove_low_freq_word(self, texts, times=1):
    """
    去掉低频词
    :param times:出现次数
    :return:
    """
    all_tokens = sum(texts, [])
    title_token_once = set(word for word in set(all_tokens) if all_tokens.count(word) == times)

    texts_result = [[word for word in text if word not in title_token_once] for text in texts]
    return texts_result

texts = []
for ix, row in self.df.iterrows():
    texts_cuts = row['title_fenci'].split('|')
    texts.append(texts_cuts)

texts = self.remove_low_freq_word(texts)
```



## 3，建立LSI模型

通过上一步的texts抽取一个“词袋（bag of words），将文档的token映射为id。

```
dictionary = corpora.Dictionary(texts)
print dictionary
print dictionary.token2id

Dictionary(179 unique tokens: [u'\u8868\u793a', u'', u'\u53bb\u5e74', u'\u4ee5\u6765', u'\u800c']...)
{u'\u8868\u793a': 0, u'': 124, u'\u53bb\u5e74': 127, u'\u4ee5\u6765': 1, u'\u800c': 113, u'\u5219': 121, u'\u7f57\u68ee\u683c\u4f26': 175, ...}
```

接下来用字符串表示的文档转换为用id表示的文档向量.

```
corpus = [dictionary.doc2bow(text) for text in texts]
print corpus

[[(0, 3), (1, 5), (2, 1), (3, 2), (4, 2), (5, 1), (6, 1), (7, 1), (8, 2), (9, 1), (10, 1), (11, 2), (12, 3), (13, 2), (14, 1), (15, 1), (16, 1), (17, 5), (18, 2), (19, 1), (20, 1), (21, 8), (22, 5), (23, 1), (24, 2), (25, 1), (26, 4), (27, 1), (28, 1), (29, 2), (30, 1), (31, 2), (32, 1), (33, 2), (34, 3), (35, 8), (36, 7), (37, 1), (38, 1), (39, 1), (40, 1), (41, 1), (42, 5), (43, 4), (44, 3), (45, 5), (46, 9), (47, 1), (48, 2), (49, 1), (50, 2), (51, 4), (52, 2), (53, 3), (54, 2), (55, 2), (56, 2), (57, 1), (58, 1), (59, 1), (60, 3), (61, 6), (62, 3), (63, 2), (64, 3), (65, 1), (66, 2), (67, 1), (68, 2), (69, 1), (70, 1), (71, 1), (72, 1), (73, 5), (74, 1), (75, 1), (76, 1), (77, 1), (78, 1), (79, 1), (80, 1), (81, 3), (82, 3), (83, 2), (84, 2), (85, 1), (86, 3), (87, 1), (88, 1), (89, 1), (90, 1), (91, 2), (92, 1), (93, 3), (94, 1), (95, 13), (96, 1), (97, 1), (98, 2), (99, 1), (100, 1), (101, 1), (102, 1)], [(0, 1), (3, 2), (6, 1), (7, 1), (8, 4), (9, 2), (12, 2), (13, 1), (15, 1), (16, 1), (17, 2), (18, 1), (19, 1), (21, 10), (23, 2), (25, 1), (26, 1), (34, 1), (35, 2), (38, 1), (39, 1), (42, 3), (43, 1), (44, 2), (45, 2), (46, 1), (48, 1), (50, 1), (52, 1), (58, 1), (61, 1), (63, 1), (64, 2), (65, 1), (68, 3), (69, 1), (73, 2), (75, 1), (79, 1), (85, 1), (86, 1), (89, 1), (91, 1), (94, 1), (95, 3), (96, 4), (99, 2), (103, 1), (104, 2), (105, 1), (106, 1), (107, 1), (108, 1), (109, 2), (110, 1), (111, 2), (112, 4), (113, 1), (114, 1), (115, 1), (116, 2), (117, 2), (118, 1), (119, 1), (120, 2), (121, 1), (122, 3), (123, 1)],
```

例如，最后一列的(123,1)表示第二篇文档中id为123的单词出现了1次。

接下来基于这个“训练文集”计算TF-IDF模型：

```
tfidf = models.TfidfModel(corpus)
corpus_tfidf = tfidf[corpus]
```

有了tf-idf值的文档向量，接下来开始训练LSI模型：

```
lsi = models.LsiModel(corpus_tfidf, id2word=self.dictionary, num_topics=10)

2016-05-21 22:27:45,601 : INFO : topic #0(2.646): 1.000*"" + 0.000*"柯薛拉柯塔" + 0.000*"应该" + 0.000*"加息" + 0.000*"不" + 0.000*"美联储" + 0.000*"今年" + 0.000*"下降" + -0.000*"有" + 0.000*"2014"
2016-05-21 22:27:45,601 : INFO : topic #1(1.594): 0.375*"的" + 0.266*"美联储" + 0.244*"美国" + 0.232*"br" + 0.200*"加息" + 0.191*"柯薛拉柯塔" + 0.163*"在" + 0.162*"罗森格伦" + 0.138*"应该" + 0.131*"是"
2016-05-21 22:27:45,602 : INFO : topic #2(1.055): 0.320*"柯薛拉柯塔" + 0.257*"美联储" + -0.245*"埃文斯" + 0.216*"罗森格伦" + -0.214*"增速" + 0.213*"加息" + -0.205*"到" + -0.205*"了" + -0.197*"都" + -0.188*"美国"
2016-05-21 22:27:45,602 : INFO : topic #3(0.989): 0.420*"柯薛拉柯塔" + -0.355*"罗森格伦" + 0.304*"应该" + -0.270*"鉴于" + -0.270*"处于" + -0.245*"利率" + 0.233*"今年" + 0.222*"不" + -0.216*"目前" + -0.194*"很"
2016-05-21 22:27:45,604 : INFO : topic #4(0.908): -0.347*"到" + -0.347*"了" + -0.346*"都" + -0.331*"增速" + -0.244*"美联储" + -0.203*"埃文斯" + -0.176*"就业" + -0.158*"柯薛拉柯塔" + -0.151*"市场" + 0.131*"月"
```
lsi最核心的意义是将训练文档向量组成的矩阵SVD分解，并做了一个秩为2的近似SVD分解。
有了LSI模型，建立索引:

```
index = similarities.MatrixSimilarity(lsi[corpus])
2016-05-21 22:32:20,877 : INFO : creating matrix with 4 documents and 4 features
```

## 4，计算相似度

```
# 计算某一篇文档的相似度
tl_bow = dictionary.doc2bow(df.ix[10, 'title_fenci'].split('|'))
tl_lsi = lsi[tl_bow]
# print tl_lsi
sims = index[tl_lsi]
# print sims

sort_sims = sorted(enumerate(sims), key=lambda item: -item[1]) #
print sort_sims[0:10]


# Output:

[(10, 1.0), (12, 0.74251199), (0, 0.62192106), (1, 0.61248362), (13, 0.45317733), (11, 0.44128361), (8, 0.40342486), (2, 0.0), (3, 0.0), (4, 0.0)]

```

第10篇的为它自己，相似度为1，完全相似；与第12篇的相似度为0.74等等。



# 五、进阶

计算出文章的相似度，就可以对相似度设定一个阈值，高于阈值的文章算是重复文章。这样就引出了另外一个用途，文章去重！

本文的应用背景是将多个资讯平台的文章汇总，很有可能出现不同的平台报道同样的内容，这是大概率事件，所以，为了保证内容的唯一性，需要对汇总的文章去重处理。

## Python实现代码

```
#coding:utf-8
"""
文章去重
"""

import pandas as pd
import os
import cPickle
from gensim import corpora, models, similarities
from db_config import engine

import re
regex = re.compile(ur"[^\u4e00-\u9f5aa-zA-Z0-9]")

import logging
logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s', level=logging.INFO)

class NewsCorpus():

    def __init__(self, column):
        self.dictionary = []
        self.column = column
        self.similarity = 0.85
        self.num_topics = 10

        self.filename = 'Data/pkl/pkl_news'
        if not os.path.exists(self.filename):
            sql = 'select id, title, content from article_news order by published_date desc limit 1000'

            self.df = pd.read_sql(sql, engine)
            with open(self.filename, 'wb') as f:
                cPickle.dump(self.df, f)

        else:
            with open(self.filename, 'rb') as f:
                self.df = cPickle.load(f)

        print '标题去重前:{}'.format(len(self.df))
        self.df = self.df.drop_duplicates(['title'])
        print '标题去重后:{}'.format(len(self.df))

    def jieba_cut(self):
        """
        对标题和内容分词
        :return:
        """

        if self.column+"_fenci" in self.df.columns:
            return

        import jieba
        jieba.load_userdict('userdict.txt')   # 自己准备用户词典，也可不指定

        # 去掉标点和特殊字符 然后分词
        self.df[self.column+'_fenci'] = self.df[self.column].apply(lambda x : '|'.join(jieba.cut(regex.sub('',x))) if x and len(x) else '')

        with open(self.filename, 'wb') as f:
            cPickle.dump(self.df, f)

    def remove_low_freq_word(self, texts, times=1):
        """
        去掉低频词
        :param times:出现次数
        :return:
        """
        all_tokens = sum(texts, [])
        title_token_once = set(word for word in set(all_tokens) if all_tokens.count(word) == times)

        texts_result = [[word for word in text if word not in title_token_once] for text in texts]

        return texts_result


    def create_dictionary(self, df):
        """
        创建词典
        :return:
        """

        try:
            texts = []
            for ix, row in df.iterrows():
                texts_cuts = row[self.column+'_fenci'].split('|')
                texts.append(texts_cuts)

            texts = self.remove_low_freq_word(texts)
            self.dictionary = corpora.Dictionary(texts)
            print self.dictionary
            print self.dictionary.token2id

            corpus = [self.dictionary.doc2bow(text) for text in texts]
            print corpus

            tfidf = models.TfidfModel(corpus)
            corpus_tfidf = tfidf[corpus]

            # 训练topic数量为10的LSI模型
            lsi = models.LsiModel(corpus_tfidf, id2word=self.dictionary, num_topics=self.num_topics)

            print lsi.print_topic(3)
            # 建立索引
            index = similarities.MatrixSimilarity(lsi[corpus])

            # 计算相似度
            sims_all = []
            for ix, row in df.iterrows():
                tl_bow = self.dictionary.doc2bow(row[self.column+'_fenci'].split('|'))
                tl_lsi = lsi[tl_bow]
                # print tl_lsi
                sims = index[tl_lsi]
                # print sims
                sims_all.append(sims)

                sort_sims = sorted(enumerate(sims), key=lambda item: -item[1])

                print sort_sims[0:10]

            return sims_all
        except Exception,e:
            print 'create_dictionary:', e
            return []


    def get_results(self, df, sims_all):

        df_sims = pd.DataFrame(sims_all)
        df_sims[df_sims < self.similarity] = 0
        print df_sims

        indexs_dict = {}
        df_sims = df_sims[df_sims>0]

        ixs = [] # 记录已跑过的index
        for ix, row in df_sims.iterrows():
            row0 = row[row > 0]
            ixs.append(ix)
            if len(row0) == 0:
                continue

            # print row0
            index = row0.index.tolist()
            index = set(index) - set(ixs)
            if len(index) > 0:
                indexs_dict[ix] = list(index)

        file_name = 'Data/{}_{}.xlsx'.format(self.column, df.ix[0, 'date'])

        print file_name
        if indexs_dict:
            with pd.ExcelWriter(file_name) as writer:
                for key in indexs_dict.keys():
                    index_list = []
                    index_list.append(key)
                    index_list.extend(indexs_dict[key])

                    print df['title'].head()
                    df_ouput = df[df['id'].apply(lambda x : x in index_list)]
                    print df_ouput.head()
                    if len(df_ouput) == 0:
                        continue

                    # dfs_ouput.append(df_ouput)
                    df_ouput.to_excel(writer, sheet_name=str(key))


        indexs_all = sum(indexs_dict.values(), [])
        print indexs_all[:5]
        indexs_all = list(set(indexs_all))

        se_result = df['id'].apply(lambda x : x not in indexs_all)
        df_result = df[se_result==True]
        print len(df_result)
        print df_result['title'].head()
        return df_result


    def run(self):
        self.jieba_cut()
        # 按周期(天)分隔
        df = self.df
        if len(df) <= 1:
            continue


        df.index = pd.Int64Index(range(len(df)))
        df['id'] = df.index
        print df['title'].head()
        print "len:", len(df)

        sims_all = self.create_dictionary(df)
        if len(sims_all) == 0:
            continue
        df_result = self.get_results(df, sims_all)
        if len(df_result):
            df_results.append(df_result)
        df_o = pd.concat(df_results, ignore_index=True)
        df_o.to_excel('Data/sim_{}.xlsx'.format(self.column))
        # df_o.to_csv('Data/sim_{}.csv'.format(self.column))


if __name__ == '__main__':

    news = NewsCorpus('content')
    news.similarity = 0.95
    news.num_topics = 10
    news.run()
```




*参考*

> 1. [机器学习中的数学(5)-强大的矩阵奇异值分解(SVD)及其应用](http://www.cnblogs.com/LeftNotEasy/archive/2011/01/19/svd-and-applications.html)

> 2. [如何计算两个文档的相似度](http://www.52nlp.cn/如何计算两个文档的相似度)



