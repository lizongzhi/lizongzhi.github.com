---
layout: post
title: " 有限状态机"
category: Design Pattern
tags: [设计模式, C++]
description: |
  有限状态机在人工智能中有重要的地位，其典型的应用就是在游戏开发中用来控制状态的切换。使用if……else……,switch……case……同样也能控制状态的切换，但是当状态一多，觉得让你崩溃。有限状态机是最好的选择。
---

  上个学期的游戏作业快让我崩溃了，老师上课没讲什么，下课一堆的作业，花了我近一半的时间，实在无语。上学习在做游戏作业的时候，最大的感触就是作业越来越恐怖，出来那本身就令我很无语的界面外，就是作业越往后面状态也就越多
  ，if语句漫天飞，到了后面代码惨不忍睹。在查阅相关资料后发现有限状态机能够解决这个问题。
  
  在这个项目中本来是感觉状态很少，没必要杀鸡用牛刀，不过庆幸的是幸好用了有限状态机，虽然状态少，但是状态中要完成的任务并不简单，状态间的切换也很麻烦，再加上做出来一版后被要求修改成服务的形式，如果没用有限状态机真不知道会改到什么时候。
  
  下面列下我找到的学习资料。
  
  1、[A Simple C++ Finite State Machine](http://www.bigfatalien.com/?p=125)
  
  2、[有限状态机的C++实现(1)-epoll状态机](http://www.vimer.cn/2011/01/%E6%9C%89%E9%99%90%E7%8A%B6%E6%80%81%E6%9C%BA%E7%9A%84c%E5%AE%9E%E7%8E%B01-epoll%E7%8A%B6%E6%80%81%E6%9C%BA.html)
  
  3、[State-Driven Game Agent Design](http://www.ai-junkie.com/architecture/state_driven/tut_state1.html)
  
  
 