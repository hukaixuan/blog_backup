---
title: "Python Cookbook -- 数据结构与算法"
tags:
 - Python
---

### 记录最后N项（历史记录功能）**collections.deque(maxlen=N)**
``` python
from collections import deque
def search(lines, pattern, history=5):
    previous_lines = deque(maxlen=history)
    for line in lines:
        if pattern in line:
            yield line, previous_lines
        previous_lines.append(line)
if __name__ == '__main__':
    with open('somefile.txt') as f:
        for line, previous_lines in search(f, 'Apple', 5):
            for pline in previous_lines:
                print(pline, end='')
            print(line, end='')
            print('-'*20)
```

### 找到最大或最小的N个项： **heapq.nlargest(n, nums) / heapq.hsmallest(n, nums)**

### 实现优先队列：
heapq默认是从小到大的，所以用 -priority 来做依据push，结果是从大到小

``` python
import heapq

class PriorityQueue:
    def __init__(self):
        self._queue = []
        self._index = 0

    def push(self, item, priority):
        heapq.heappush(self._queue, (-priority, self._index, item)) 
        self._index += 1

    def pop(self):
        return heapq.heappop(self._queue)[-1]

class Item:
    def __init__(self, name):
        self.name = name

    def __repr__(self):
        return 'item({!r})'.format(self.name)

q = PriorityQueue()
q.push(Item('foo'), 1)
q.push(Item('bar'), 5)
q.push(Item('spam'), 4)
q.push(Item('grok'), 1)

print(q.pop())
print(q.pop())
print(q.pop())
print(q.pop())
```

### 一键对多值的映射： **collections.defaultdict**
``` python
from collections import defaultdict

d = defaultdict(list) 
d['a'].append(1)
d['a'].append(2)
d['b'].append(3)

d = defaultdict(set)
d['a'].append(1)
d['a'].append(2)
d['b'].append(3)
```

### 字典保留插入顺序：**collections.OrderedDict**
```python
from collections import OrderedDict

d = OrderedDict()
d['foo'] = 1
d['bar'] = 2
d['spam'] = 3
d['grok'] = 4
```
- OrderedDict占用的空间比较大
- 可以用来序列化或编码的时候保持顺序：
```python
import json
json.dumps(d)
```

### 字典的并交等运算：
```python
a = {'x':1, 'y':2, 'z':3}
b = {'w':10, 'x':11, 'y':12}
a.keys() & b.keys()
a.keys() - b.keys()
a.items() & b.items()
```
keys / items 可以执行 set 的一些操作，
values 不可以，因为 values 没有唯一性。

### 保持序列的顺序不变删除重复项：
```python
def dedupe(items, key=None):
	seen = set()
	for item in items:
		val = item if key is None else key(item)
		if val not in seen:
			yield item
			seen.add(val)
			
a = [{'x':1, 'y':2}, {'x':1, 'y':3}, {'x':1, 'y':2}, {'x':2, 'y':4}]
list(dedupe(a, key=lambda d: (d['x'], d['y'])))
```

### 命名切片：**name = slice(start, stop, step)**

### 计算某序列中最常出现的项：**collections.Counter.most_common**
```python
words = 'hello world world, and hello you and me and bla bla, gds'.split()
word_counts = Counter(words)
top_three = word_counts.most_common(3)
word_counts['world']
word_counts.update(morewords)
```
Counter 支持算数运算：
```python
a = Counter(words)
b = Counter(morewords)

c = a + b
c = a - b
```

### 对一个列表中的字典进行排序：用到**operator.itemgetter**
```python
rows = [{'fname': 'Hu', 'lname': 'dd', 'uid': 001}, {...}, {...}, ...]
rows_by_lfname = sorted(rows, key=itemgetter('lname', 'fname'))

rows_by_lfname = sorted(rows, key=lambda r: (r['lname'], r['fname']))
```

### 根据对象的某属性对对象进行排序：用到**operator.attrgetter**
```python
from operator import attrgetter
sorted(users, key=attrgetter('user_id'))
```

### 根据某属性进行Group： 用到 **itertools.groupby**
`groupby(rows, key=itemgetter('a_filed')`

### 过滤序列的元素
```
[n for n in mylist if n>0]
(n for n in mylist if n>0)
filter(lambda x: x>0, a_list)
[n if n>0 else 0 for n in mylist]
```
**itertools.compress**

```
from itertools import compress
more5 = [n>5 for n in counts]
list(compress(addressses, more5))
```
compress 第一个参数是第二个参数序列对应的布尔值的序列。

### **collections.namedtuple**
```
from collections import namedtuple
Subscriber = namedtuple('Subscriber', ['addr', 'joined'])
sub = Subscriber('jonesy@example.com', '2014-10-19')
print(sub.addr)
print(sub.joined)
```

### **collections.ChainMap**