# source

speedynote按领域(domain)划分。

成熟项目的特征。

学。

## 核心业务

### 待定子节点

## 基础设施

### platform、compat

软件如何适应不同环境
qt5/qt6 兼容？windows/linux 差异？文件系统差异？

## 平台实现（android、……）

### .android

为什么speedynote根目录里已经有一个.android
这里还有一个？
大概率外面那个是Android工程
里面这个是Android平台相关代码
是职责的划分

## objects

看起来最特殊的一个
大概是作者这个项目的世界里，如何划分都有什么object？
link、ocrtext、highlight、textbox、image、inserted
为什么没有stroke、...？

### Link

LinkObject.h,LinkObject.cpp,LinkObjectTests.h
