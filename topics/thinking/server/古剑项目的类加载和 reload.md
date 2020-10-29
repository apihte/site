# 古剑项目的热加载

## 1.外部类加载

主要用于调用新加的外部 GM 命令，自定义了一个类加载器 ExtraCommandClassLoader，用于加载 extgm.jar 包中的 .class 文件。

在使用外部新加的 GM 命令时，会通过这个 ExtraCommandClassLoader 类加载器去 extgm.jar 包中查找相应的类，然后反射调用 exec 方法。

## 2.已有逻辑热加载

逻辑代码的热加载主要是使用 Instument 技术来替换和修改某些类的定义。