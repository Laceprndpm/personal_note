---
title: 
date: 2025-08-02
last_clear: 2025-08-02
tags:
  - tag/图论/树上问题
grasp: 
difficulty: 
source: 
article:
---
tiger:
![[Pasted image 20250802214503.png]]
SS:
G是折半报警器
就是我让较长的那一半
做i的父亲
只看有效长度
维护每跳一次左右两边丢失的长度
树上倍增
然后每次跳出界 就暴力一下
唯一会让长度不折半的情况
就是跳到r了
但是r因为被截断
实际上比一半长
但是这样
x所在的位置一定不能超过inf/2了
如果超过说明上一次r是比较长的 不会跳出界
jiangly:
https://oi-wiki.org/ds/persistent-balanced/
学习一下兄弟
别写 2log 了
写个可持久化平衡树不比你这个好写
![[Pasted image 20250802215722.png]]
这个和折半警报器也没啥关系
就一个树上倍增
SS:
折半报警就是掉出大的那一半吧
可能和传统的折半报警器不太一样 **但是核心思想相同**
jiangly:
完全不一样吧
折半警报器是两边一起在动
然后找最后一个动完的时间
这个只是每次减半然后用树上倍增优化一下
#TODO/abcG
