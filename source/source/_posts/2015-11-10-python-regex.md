---
layout: post
title: Python正则表达式
date: 2015-11-10 15:09:20
tags: 
	- Python
	- re 
	- 正则表达式 
	- 模式匹配
category: Python
comments: true
---

许多语言处理任务都涉及模式匹配。例如,可以使用endswith('ed')找出以“ed”结尾的词。正则表达式提出了一个更加强大和灵活的方法描述感兴趣的字符模式。在Python中使用正则表达式，需要使用import re导入re函数库。

**下表为正则表达式基本元字符，其中包括通配符、范围和闭包**
<!-- more -->
![正则模式](http://7xo67b.com1.z0.glb.clouddn.com/regex1.png)



## 贪婪模式与非贪婪模式
Python里数量词默认是贪婪的（在少数语言里也可能是默认非贪婪），总是尝试匹配尽可能多的字符；非贪婪的则相反，总是尝试匹配尽可能少的字符。
例如：正则表达式 “ab*”，如果用于查找“abbbc”，将找到“abbb”。而如果使用非贪婪的数量词“ab\*?“，将找到”a“

1. \*? 是一个固定的搭配，.和\*代表可以匹配任意无限多个字符，加上？表示使用非贪婪模式进行匹配，也就是我们会尽可能短地做匹配。

2. (.\*?)代表一个分组，在这个正则表达式中我们匹配了五个分组，在后面的遍历item中，item[0]就代表第一个(.\*?)所指代的内容，item[1]就代表第二个(.*?)所指代的内容，以此类推。

3. re.S 标志代表在匹配时为点任意匹配模式，点 . 也可以代表换行符。


## re模块

### 一、re.search()

使用正则表达式<<ed$>>查找以ed结尾的词汇。使用函数re.search(p, s) 检查字符串s中是否有模式p。

	import re
	import nltk
	In[12]: wsj = sorted(set(nltk.corpus.treebank.word()))
	In[13]: ws = [w for w in wsj if re.search('ed$', w)]
	
	In[15]: ws[:10]
	Out[15]: 
	[u'62%-owned',u'Absorbed',u'Advanced',u'Alfred', u'Allied', u'Annualized', u'Arbitrage-related',
	 u'Asked',u'Atlanta-based', u'Bermuda-based']

通配符“.”可以用来匹配任何单个字符。假设有一个8个字母组成的字谜，j是第三个字母，t是第六个字母。每个空白单元格用句点隔开。


	In[16]: ws = [w for w in wsj if re.search('^..j..t..$', w)]
	In[18]: ws
	Out[18]: [u'adjusted', u'rejected']

匹配除元音字母之外的所有字母


	[^aeiouAEIOU]


**?:**
如果要使用括号来指定连接的范围，又不想选择要输出字符串，必须添加“?:”。


	In[20]: re.findall(r'^.*(?:ing|ly|ed|ies)$', 'processing')
	Out[20]: ['processing']


演示如何使用符号：\，{}，() 和 |


	In[20]: ws = [w for w in wsj if re.search('^[0-9]+\.[0-9]+$', w)]
	In[21]: ws[:5]
	Out[21]: [u'0.0085', u'0.05', u'0.1', u'0.16', u'0.2']
	
	In[22]: ws = [w for w in wsj if re.search('^[A-Z]+\$$', w)]
	In[23]: ws
	Out[23]: [u'C$', u'US$']
	
	In[24]: ws = [w for w in wsj if re.search('^[0-9]{4}$', w)]
	In[26]: ws[:5]
	Out[26]: [u'1614', u'1637', u'1787', u'1901', u'1903']
	
	In[27]: ws = [w for w in wsj if re.search('(ed|ing)$', w)]
	In[28]: ws[:5]
	Out[28]: [u'62%-owned', u'Absorbed', u'According', u'Adopting', u'Advanced']


### 二、re.split()
按照能够匹配的子串将string分割后返回列表。

#### re.split(pattern, string[,maxsplit])

	In[13]: raw = """'When I'M a Duchess,' she said to herself, (not in a very hopeful tone
	... though), 'I won't have any pepper in my kitchen AT ALL. Soup does very
	... well without--Maybe it's always pepper that makes people
	... hot-tempered,'..."""
	In[16]: re.split(r' ', raw)
	Out[16]: 
	["'When",
	 "I'M",
	 'a',
	 "Duchess,'",...]
	In[17]: re.split('[ \t\n]', raw)
	Out[17]: 
	["'When",
	 "I'M",
	 'a',
	 "Duchess,'",...]


#### split(string[, maxsplit])

	In [1]: import re
	
	In [2]: p = re.compile(r'\d+')
	
	In [3]: p.split('one1two2three3four4')
	Out[3]: ['one', 'two', 'three', 'four', '']


#### 切分字符串

Python自带的字符分割函数

	'a b   c'.split(' ')
	['a', 'b', '', '', 'c']

嗯，无法识别连续的空格

	import re
	>>> re.split(r'\s+', 'a b    c')
	['a', 'b', 'c']

使用re，无论多少个空格都可正常分割

### 三、findall()
findall函数返回的总是正则表达式在字符串中所有匹配结果的列表。


	In [2]: p = re.compile(r'\d+')
	
	In [4]: p.findall('one1two2three3four4')
	Out[4]: ['1', '2', '3', '4']
	
	In [5]: ss = "adfad asdfasdf asdfas asdfawef asd adsfas "
	
	In [6]: p = re.compile('((\w+)\s+\w+)')
	
	In [7]: p.findall(ss)
	Out[7]:
	[('adfad asdfasdf', 'adfad'),
	 ('asdfas asdfawef', 'asdfas'),
	 ('asd adsfas', 'asd')]
	 
	In [8]: p = re.compile('(\w+)\s+\w+')
	
	In [9]: p.findall(ss)
	Out[9]: ['adfad', 'asdfas', 'asd']


1. 当给出的正则表达式中不带括号时，列表的元素为字符串，此字符串为整个正则表达式匹配的内容。 

2. 当正则表达式中带有多个括号时，列表的元素为多个字符串组成的tuple，tuple中字符串个数与括号对数相同，字符串内容与每个括号内的正则表达式相对应，并且排放顺序是按括号出现的顺序。

3. 当给出的正则表达式中带有一个括号时，列表的元素为字符串，此字符串的内容与括号中的正则表达式相对应。


### 四、re.search()
re.search函数会在字符串内查找模式匹配，只要找到第一个匹配就返回，如果字符串没有匹配，则返回None。

	In [15]: text = "JGood is a handsome boy, he is cool, clever, and so on..."
	
	In [16]: m = re.search(r'(\w+)ome', text)
	
	In [20]: if m:
	   ....: print m.group(0), m.group(1)
	   ....: else:
	   ....: print 'not search'

其中 group(0）或group()匹配的是整个字符串，group(1)匹配的是第一个括号中内容。   

### 五、re.match()
re.match()和re.search()的区别：re.match只匹配字符串的开始，如果字符串开始不符合正则表达式，则匹配失败，函数返回None；而re.search匹配整个字符串，直到找到一个匹配。

	In [30]: s1 = "helloworld, i am 30!"
	
	In [31]: w1 = 'world'
	
	In [32]: m1 = re.match(w1, s1)
	
	In [33]: if m1:
	   ....: 	print m1.group()
	   ....: else:
	   ....: 	print "not find"
	   ....:
	not find


### 六、re.sub()
re.sub用于替换字符串中的匹配项。
下面的例子将字符串中的空格’ ‘替换成’-‘

	In [2]: text = "JGood is a handsome boy, he is cool, clever, and so on..."
	
	In [3]: re.sub(r'\s+', '-', text)
	Out[3]: 'JGood-is-a-handsome-boy,-he-is-cool,-clever,-and-so-on...'

**re.sub的函数原型为：re.sub(pattern, repl, string, count)**

	其中第二个参数时替换后的字符串；
	第四个参数为替换个数。默认为0，表示每个匹配项都替换。
re.sub还允许使用函数对匹配项的替换进行复杂的处理。如：

	re.sub(r'\s', lambda m : '[' + m.group(0) + ']', text, 0)

将字符串中的空格’‘替换为'[]'。

### 七、re.compile()
可以把正则表达式编译成一个正则表达式对象。对于经常要用的正则表达式，可以提高一定的效率。

	In [4]: text = "JGood is a handsome boy, he is cool, clever, and so on..."
	
	In [6]: regex = re.compile(r'\w*oo\w*')
	
	In [7]: regex.findall(text) #查找所有包含’oo‘的单词
	Out[7]: ['JGood', 'cool']
	
	In [8]: regex.sub(lambda m : '[' + m.group(0) + ']', text) # 将字符串中含有’oo‘的单词用[]括起来
	Out[8]: '[JGood] is a handsome boy, he is [cool], clever, and so on...'




	


