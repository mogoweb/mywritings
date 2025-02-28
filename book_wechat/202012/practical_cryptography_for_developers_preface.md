# 写给开发者的实用密码学 - 前言

各位朋友，非常惭愧这半年来疏于更新公众号，不是因为太忙，而是太闲。

过去的2020年发生了太多的事情，2020年1月23日武汉封城历历在目，仿佛就在昨天。多少人经历了悲欢离合、多少店铺关张，又有多少企业倒下？所以这段时间我一直在思索人生的意义。人生的意义就是追求幸福的人生，可什么是幸福呢？

财务自由是很多人的梦想，这意味着我们不需要看老板的脸色，随心所欲，做自己想做的事情。虽然我还未做到财务自由，但一定程度的随心所欲还是可以达到。所以这段时间我就随心的练练书法，跑步也非常佛系，不追求配速、不追求跑量，闲暇时间玩玩摄影(拍照)、围棋。工作上的事情，也是随心所欲，完成就好。但半年时间下来，我发现并没有体会到快乐。以前跑步都是朝着跑马的目标训练，看着一点点完成跑马目标，那份喜悦之情是不跑步的人体验不到的。在没有具体目标之后，人反而懒散下来，现在最多跑个十公里，就无以为继。练习书法，尝试了很多帖子，依然感觉不到写字的乐趣。

问题在哪？乔丹.彼得森说过：

> 人类只要一适应幸福的状态，就又会开始觉得不幸福。

缺少目标感，缺少进步，即使衣食无忧，我仍然感受不到幸福。当然，如何找到幸福，是一个宏达的主题。而当下最重要的是，赶快行动起来，找一些具体的事情做一做。一点点进步也比没有进步强，所以我又捡起荒废很久的公众号，还是以我擅长的技术入手，写一写技术相关的文章。以今天为起点，总结一下前一段时间所做的工作。

计算机安全在学校课堂有学过，但毕业之后并没有涉及这一领域，所以回想起来一片空白。前一段时间做了一些浏览器的开发工作，为浏览器增加国密支持，不得不啃了密码学相关的知识。有两本书对我帮助很大，一本是《深入浅出HTTPS：从原理到实战》，另一本是《Practical Cryptography for Developers》。

一直以来，密码学算法一直被认为是专家和数学家的专有技术。现在依然如此，在阅读国密相关的标准文档，特别是讲到密码算法的原理，如读天书。当这并没有妨碍我完成这项工作，成功的给浏览器增加了国密算法的支持。因为我们只是密码算法的使用者，其内部为什么要那么设计，其原理是什么，如果能够理解更好，但理解不了也不影响我们使用。更为重要的是，这些国密算法已经有开源实现，虽然其实现方式和语言，但一些通用的东西还是可以借鉴。当然，密码学仍然是一门非常复杂的学科，即使不用理解算法，但一些密码学的概念，如何使用密码组合成一套安全的体系，也需要深入学习。

在开发国密浏览器的同时，我也对密码学有了一定的理解，所以尝试着将我所理解的密码学总结一下，算是对这段时间的开发工作的一个小结。其实关于密码学的一些知识，上面的两本书已经讲得很好。但怎么说呢？书本上的知识仅仅是知识，只有在消化吸收后才能变为自己的。我希望在梳理的过程中，加深对密码学的理解。在后面的文章中，我会按照《Practical Cryptography for Developers》的组织结构来梳理在 HTTPS 中涉及到的密码学知识。

因为我的数学知识比较薄弱，所以在这里我不会涉及到密码算法的内部，有关算法也只会一笔带过。更多的是通过代码示例和动手实践，就像我们学习一门编程语言一样，更多的是学习一些概念，然后就是掌握 API 和工具的使用，最后完成一个实际的应用。

在后续文章中，将涉及到哈希、MAC 码和密钥推导函数（KFD）、随机生成器、密钥交换协议、对称密码、加密方案、非对称密码系统、公共密钥密码学、椭圆曲线、数字签名，在谈到各项技术时，会加入国密算法的相关内容。当然，由于学识有限，量子安全加密算法、现代加密工具和库就不会涉及。

这些技术都是一些冷门的技术，如果在工作中没有用到，肯定也没有兴趣看，不过这不重要了，即使有一个人看到了文章，对他有用，那就值得了。

这里也要谢谢各位朋友的不离不弃，你们的支持是我最大的动力。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)