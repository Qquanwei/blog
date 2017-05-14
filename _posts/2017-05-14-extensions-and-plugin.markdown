---
layout: post
title: extensions and plugin
---

事情是这样的。

经常我会看到extensions 和 plugin 在我的各种配置文件中使用, 而我，一直对其的界限不是很明确

但是不知从哪里看到了一句话让我认清了两者之间的区别。

## plugins

使用调用者自身提供的一组API完成的任务，在此基础上进行行为的扩充称之为plugins

## extensitions

当被加载后成为调用者自身的一部分，能够提供一组扩充的API供上层plugin使用或直接使用。

## 总结

A........

plugins的功能是开发者预留的一些接口来完成的功能，为了方便各种不同但是有共性的功能进行上层开发，
而extensitions往往更加偏向底层，提供一组原来不具备的能力。
