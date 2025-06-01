---
title: T_Engine_TODO
date: 2024-01-03 22:41:05
tags:
---
- 重写渲染架构，现在是model在导入的时候就把纹理保存在mesh里面了，看了Unity meshrender，meshrender保存material每个material对应一个mesh，把纹理保存在material里而不是模型中，把模型分为mesh和材质球 修改文件modle，mesh entity ，<font color=red>不着急写meshrender 但是要先把mesh和material分开</font>