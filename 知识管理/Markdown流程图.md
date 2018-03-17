# Markdown流程图

## 1. 如何插入流程图

流程图的画法和代码段类似，也就是说，流程图是写在两个**```**之间的。

比如说Java代码，会是这样一种格式：

```txt
 ```java 
 代码段
 ```
```

那么流程图就是这样的：

```txt
 ```flow
 代码段
 ``` 
```

## 2. 画流程图的步骤

流程图的语法大体分为两段

1. 第一段用来定义元素
2. 第二段用来连接元素

### 定义元素阶段的语法是

```txt
tag=>type: content:>url
```

- tag是流程图中的标签，名称可以任意，一般为流程的英文缩写和数字的组合。
- type用来确定标签的类型，由于标签的名称可以任意指定，所以要依赖type来确定标签的类型
- content是流程图文本框中的描述内容，中英文均可。**特别注意**，type后的冒号与文本之间一定要有个空格
- url是一个连接，与框框中的文本相绑定，点击文本时可以通过链接跳转到url指定页面

type有6中类型，分别为：

- start         # 开始

- end           # 结束

- operation     # 操作

- subroutine    # 子程序

- condition     # 条件

- inputoutput   # 输入或产出

- content就是在框框中要写的内容，注意type后的冒号与文本之间一定要有个空格。

- url是一个连接，与框框中的文本相绑定

### 连接元素的语法

用**->**来连接两个元素，需要注意的是condition类型，因为他有yes和no两个分支，所以要写成

```txt
c2(yes)->io->e 
c2(no)->op2->e
```

## 3. 实例

```txt
st=>start: Start
op=>operation: Your Operation
sub=>subroutine: My Subroutine
cond=>condition: Yes or No?
io=>inputoutput: catch something...
e=>end: End

st->op->cond
cond(yes)->io->e
cond(no)->sub(right)->op
```

效果：

```flow
st=>start: Start
op=>operation: Your Operation
sub=>subroutine: My Subroutine
cond=>condition: Yes or No?
io=>inputoutput: catch something...
e=>end: End

st->op->cond
cond(yes)->io->e
cond(no)->sub(right)->op
```

### 实际应用

```txt
st=>start: Start|past:>http://www.google.com[blank]
e=>end: End:>http://www.google.com
op1=>operation: get_hotel_ids|past
op2=>operation: get_proxy|current
sub1=>subroutine: get_proxy|current
op3=>operation: save_comment|current
op4=>operation: set_sentiment|current
op5=>operation: set_record|current

cond1=>condition: ids_remain空?
cond2=>condition: proxy_list空?
cond3=>condition: ids_got空?
cond4=>condition: 爬取成功??
cond5=>condition: ids_remain空?

io1=>inputoutput: ids-remain
io2=>inputoutput: proxy_list
io3=>inputoutput: ids-got

st->op1(right)->io1->cond1
cond1(yes)->sub1->io2->cond2
cond2(no)->op3
cond2(yes)->sub1
cond1(no)->op3->cond4
cond4(yes)->io3->cond3
cond4(no)->io1
cond3(no)->op4
cond3(yes, right)->cond5
cond5(yes)->op5
cond5(no)->cond3
op5->e
```

效果

```flow
st=>start: Start|past:>http://www.google.com[blank]
e=>end: End:>http://www.google.com
op1=>operation: get_hotel_ids|past
op2=>operation: get_proxy|current
sub1=>subroutine: get_proxy|current
op3=>operation: save_comment|current
op4=>operation: set_sentiment|current
op5=>operation: set_record|current

cond1=>condition: ids_remain空?
cond2=>condition: proxy_list空?
cond3=>condition: ids_got空?
cond4=>condition: 爬取成功??
cond5=>condition: ids_remain空?

io1=>inputoutput: ids-remain
io2=>inputoutput: proxy_list
io3=>inputoutput: ids-got

st->op1(right)->io1->cond1
cond1(yes)->sub1->io2->cond2
cond2(no)->op3
cond2(yes)->sub1
cond1(no)->op3->cond4
cond4(yes)->io3->cond3
cond4(no)->io1
cond3(no)->op4
cond3(yes, right)->cond5
cond5(yes)->op5
cond5(no)->cond3
op5->e
```

