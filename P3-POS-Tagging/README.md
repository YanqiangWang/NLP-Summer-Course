
### 作业任务：

使用98年人民日报语料库进行词性标注训练及测试。

### 作业输入：

98年人民日报语料库（1998-01-105-带音.txt），用80%的数据作为训练集，20%的数据作为验证集。

### 运行环境：

Jupyter Notebook, Python3

### 作业方法：

使用简单的统计词频的方法，对于单词的词性做出预测。暂未使用N-gram语言规则。

### 博客地址：

https://www.cnblogs.com/yanqiang/p/11259468.html

### 作业步骤：

1.处理语料库：删除段前标号。


```python
# 读取原始语料文件
in_path = '1998-01-105-带音.txt'
file = open(in_path, encoding='gbk')
in_data = file.readlines()
```


```python
# 预处理后的语料库
curpus_path = 'curpus.txt'
curpusfile = open(curpus_path, 'w', encoding='utf-8')
```


```python
#删除段前标号,[],{}
for sentence in in_data:
    words = sentence.strip().split(' ')
    words.pop(0)
    
    for word in words:
        if word.strip() != '':
            if word.startswith('['):
                word = word[1:]
            elif ']' in word:
                word = word[0:word.index(']')]

            w_c = word.split('/')
            # 生成语料库
            if(len(w_c) > 1):
                curpusfile.write(w_c[0] + ' ' + w_c[1] + '\n')
```

2.随机划分训练集80%和验证集20%。


```python
from sklearn.model_selection import train_test_split

# 随机划分
curpus = open(curpus_path, encoding='utf-8').readlines()
train_data, test_data = train_test_split(
    curpus, test_size=0.2, random_state=10)
```


```python
# 查看划分后的数据大小
print(len(curpus))
print(len(train_data) / len(curpus))
print(len(test_data) / len(curpus))
```

    1114419
    0.7999998205342874
    0.20000017946571264
    

3.统计训练集的词频。


```python
# 生成词频记录文件
from tqdm import tqdm_notebook

doc = []

for sentence in tqdm_notebook(train_data):
    words = sentence.strip().split(' ')
    if len(words) > 1:
        temp = []
        temp.append(words[0])
        temp.append(words[1])
        flag = False
        for line in doc:
            if line[0] == temp[0] and line[1] == temp[1]:
                line[2] += 1
                flag = True
                break
        if not flag:
            temp.append(1)
            doc.append(temp)
```


4.选择概率最大的词性。


```python
# 保存验证集
test_path = 'test.txt'
testfile = open(test_path, 'w', encoding='utf-8')
for sentence in test_data:
    words = sentence.strip().split(' ')
    if len(words) > 1:
        testfile.write(sentence)
```


```python
# 保存标注结果
result_path = 'result.txt'
resultfile = open(result_path, 'w', encoding='utf-8')
```


```python
# 选择概率最大的词性进行标注
for sentence in tqdm_notebook(test_data):
    words = sentence.strip().split(' ')
    if len(words) > 1:
        words[1] = 'n'
        max = 0
        for line in doc:
            if line[0] == words[0] and line[2] > max:
                max = line[2]
                words[1] = line[1]
        resultfile.write(words[0] + ' ' + word[1] + '\n')
```


### 性能评价：准确率


```python
def get_word(path):
    f = open(path, 'r', encoding='utf-8')
    lines = f.readlines()
    return lines

result_lines = get_word(result_path)
test_lines = get_word(test_path)

list_num = len(test_lines)
right_num = 0

for i in range(0, list_num):
    if result_lines[i][1] == test_lines[i][1]:
        right_num += 1

print("准确率为：", right_num / list_num)
```

    准确率为： 0.23189316857201872
    
