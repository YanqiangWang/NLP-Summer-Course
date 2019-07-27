
### 作业任务：

使用98年人民日报语料库进行中文分词训练及测试。

### 作业输入：

98年人民日报语料库（1998-01-105-带音.txt），用80%的数据作为训练集，20%的数据作为验证集。

### 运行环境：

Jupyter Notebook, Python3

### 作业方法：

实现了前向匹配算法的分词功能。

### 作业步骤：

1.处理语料库: 删除段前标号，以及词性标注。


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
#删除段前标号,[],{},词性标注(最短匹配)
for sentence in in_data:
    words = sentence.strip().split(' ')
    words.pop(0)
    
    for word in words:
        if word.strip() != '':
            if word.startswith('['):
                word = word[1:]
            elif ']' in word:
                word = word[0:word.index(']')]
                
            if '{' in word:
                word = word[0:word.index('{')]

            w_c = word.split('/')
            # 生成语料库
            curpusfile.write(w_c[0] + ' ')
            
    curpusfile.write('\n')
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

    22787
    0.7999736691973494
    0.20002633080265064
    

3.前向匹配算法FMM的实现。


```python
# 生成词典
from tqdm import tqdm_notebook

dic = []

for sentence in tqdm_notebook(train_data):
    words = sentence.strip().split(' ')
    for word in words:
        if word.strip() != '':
            if word not in dic:
                dic.append(word)
```


    HBox(children=(IntProgress(value=0, max=18229), HTML(value='')))


    
    


```python
# 设置单词最大长度
max_dic_len = 5
```


```python
# 生成分词测试文本
test_text = []
for sentence in test_data:
    words = sentence.strip().split(' ')
    test_text.append(''.join(words))
```


```python
# 保存验证集
test_path = 'test.txt'
testfile = open(test_path, 'w', encoding='utf-8')
for sentence in test_data:
    testfile.write(sentence)
```


```python
# 保存分词结果
result_path = 'result.txt'
resultfile = open(result_path, 'w', encoding='utf-8')
```


```python
# 前向匹配
for sentence in tqdm_notebook(test_text):
    sent = sentence
    words = []
    max_len = max_dic_len
    while(len(sent) > 0):
        word_len = max_len
        for i in range(0, max_len):
            word = sent[0:word_len]
            if word_len == 1 or word in dic:
                sent = sent[word_len:]
                words.append(word)
                word = []
                break
            else:
                word_len -= 1
                word = []
    resultfile.write(' '.join(words) + '\n')
```


    HBox(children=(IntProgress(value=0, max=4558), HTML(value='')))


    
    

### 性能评价

查准率，查全率，F度量

Precision = (Number of words correctly segmented) / (Number of words segmented) * 100%

Recall = (Number of words correctly segmented) / (Number of words in the reference) * 100%

F measure = 2 * P * R / (P + R)


```python
def get_word(path):
    f = open(path, 'r', encoding='utf-8')
    lines = f.readlines()
    return lines

result_lines = get_word(result_path)
test_lines = get_word(test_path)

list_num = len(test_lines) if len(test_lines) < len(result_lines) else len(result_lines)
right_num = 0
result_cnt = 0
test_cnt = 0

for i in tqdm_notebook(range(list_num)):
    result_sent = list(result_lines[i].split())
    test_sent = list(test_lines[i].split())
    
    result_cnt += len(result_sent)
    test_cnt += len(test_sent)
    
    str_result = ''
    str_test = ''
    
    i_result = 0
    i_test = 0
    
    while i_result < len(result_sent) and i_test < len(test_sent):
        word_result = result_sent[i_result]
        word_test = test_sent[i_test]
        
        str_result += word_result
        str_test += word_test
        
        if word_result == word_test:
            right_num += 1
            i_result += 1
            i_test += 1
        
        else:
            while len(str_result) > len(str_test):
                i_test += 1
                if i_test >= len(test_sent):
                    break
                str_test += test_sent[i_test]
            
            while len(str_result) < len(str_test):
                i_result += 1
                if i_result >= len(result_sent):
                    break
                str_result += result_sent[i_result]
            
            i_test += 1
            i_result += 1
```


    HBox(children=(IntProgress(value=0, max=4527), HTML(value='')))


    
    


```python
print("生成结果词的个数：", result_cnt)
print("验证集结果词个数：", test_cnt)

p = right_num / result_cnt
r = right_num / test_cnt
f = 2 * p * r / (p + r)

print("查准率：", p)
print("查全率：", r)
print("F度量：", f)
```

    生成结果词的个数： 227640
    验证集结果词个数： 219680
    查准率： 0.8301748374626603
    查全率： 0.8602558266569555
    F度量： 0.8449476884556917
    
