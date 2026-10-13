讨论LinkObject前，注意到：
LinkObject.h; LinkObject.cpp,
LinkObjectTests.h
Tests放单独一行的原因：猜测它不一样（待读源码验证）

## 一个LinkObject由什么组成？


>1 × LinkObject = 3 × LinkSlot>

（最多3个LinkSlot）

所以，可以先写LinkObject的核心代码，部分概念留着，后面再补（LinkObject先放到前面，再补写LinkObject可能用到的LinkSlot相关概念）

不过作者这里是先写的LinkSlot。

## 0. 文件结构观察

LinkObject.h，    LinkObject.cpp

LinkObjectTests.h

观察：

- .h：定义对象结构、成员变量、函数接口
- .cpp：应该负责具体实现
- Tests 单独存在：
    - 猜测：测试代码，与正式功能代码分离
    - 待读源码验证

---

## 1. LinkSlot 基础结构

### 1.1 LinkSlot 是什么？

```
struct LinkSlot
{
    ...
};
```

> LinkSlot 是一个“链接槽”的数据结构。

一个 LinkObject 有多个 LinkSlot：

```
LinkObject
    |
    ├── LinkSlot 1
    ├── LinkSlot 2
    └── LinkSlot 3
```

---

## 2. Type：LinkSlot 的状态标签

代码：

```
enum class Type
{
    Empty,
    Position,
    Url,
    Markdown
};
```

理解：Type 是一个标签（状态分类）。

作用：告诉 LinkSlot：

> 当前这个槽里面存储的链接是什么类型。

四种状态：

|Type|含义|
|---|---|
|Empty|空槽|
|Position|跳转到某个位置|
|Url|外部网页|
|Markdown|Markdown笔记|

然后：

```
Type type = Type::Empty;
```
创建一个 LinkSlot 时，默认状态：

```
LinkSlot
    |
    type = Empty
```

> 默认创建一个没有链接的槽。

---

## 3. LinkSlot 内的数据分类

LinkSlot 里面的数据不是全部同时有意义。

大致：

```
LinkSlot

├── type
│
├── Position 类型数据
│
├── Url 类型数据
│
└── Markdown 类型数据
```

Type 决定当前应该看哪部分数据。

---

## 4. Position 类型

这是LinkSlot最复杂的一类。

核心问题：

> “链接要跳到哪里？”

目标可能有几种：

### 4.1 页面中的位置（paged mode）

目标：跳转到

>"某个page.某position"

```
targetPageUuid // 某个page（页数）
targetPosition // page.具体坐标(position)
```
例如：

targetPageUuid : PDF第5页

targetPosition : (x=100,y=200)

---

### 4.2 无边界文档位置（edgeless mode）

注释：

> page-local coordinates for paged, document coordinates for edgeless

说的是两种坐标体系：

Paged mode : 坐标属于某一页

Edgeless mode : 坐标属于整个无限画布(document)

所以需要额外信息：

```
isEdgelessTarget

edgelessTileX
edgelessTileY
```

用于描述无边界画布中的位置。

---

### 4.3 targetObjectId

这个比较重要。

之前理解：位置 = 坐标。(处理了坐标型的跳转目标点)

>但是，“跳转target是一个对象Object？”

于是有了:

```
targetObjectId
```


>target 也可能不是一个裸坐标。


```
LinkSlot
       |
       ↓
另一个Object
```

除了“去某个位置”，LinkSlot 的功能还有：

> 去某个对象。

---

### 4.4 targetSlotIndex

暂时未理解。

已知：

```
targetObjectId
+
targetSlotIndex
```

应该用于：

精确找到目标对象中的某个slot。

待继续阅读注释/实现验证。

---

## 5. Url 类型

被定义的数据类型：

```
QString url;
```

比较简单；不用像position那样处理很多情况，网页地址所见即所得。

```
LinkSlot
    |
    URL
```

保存网页地址。

---

## 6. Markdown 类型

被定义的数据类型：Markdown笔记的id

```
QString markdownNoteId;
```

LinkSlot 存放某个 目标Markdown笔记 的 ID。

作用：

```
LinkSlot
      |
      ↓
某个Markdown文档
```

---

## 7. Serialization（序列化）

代码：

```
QJsonObject toJson() const;

static LinkSlot fromJson(...)
```

解决：

> C++对象 ↔ 文件保存格式

转换。

例如：

内存：

```
LinkSlot
{
 type = Url
 url = xxx
}
```

保存：

```
JSON
{
 type:"Url",
 url:"xxx"
}
```

重新打开：

```
JSON

↓

LinkSlot对象。
```

---

## 当前对 LinkSlot 的整体理解

可以总结为：

```
LinkSlot

一个“链接槽”

负责保存：
    1. 当前链接类型(Type)
    2. 对应类型的数据

类型包括：

Empty
    空

Position
    跳转到：
        页面坐标
        无限画布坐标
        某个Object

Url
    网页地址

Markdown
    Markdown笔记ID

另外负责：
    JSON序列化
    基础状态操作
```
