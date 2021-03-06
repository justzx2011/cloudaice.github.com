---
layout: post
title: "对实时数据可视化的讨论"
description: ""
category: realtime
published: true

---
{% include JB/setup %}

实时性在大数据处理也被广泛讨论。对现有得数据进行实时的可视化是一件非常有趣，也是非常有用的一件事情。在这套系统的构建中往往需要这样几个步骤：

1. 数据收集
2. 数据分析和计算
3. 数据存储
4. 数据的可视化

数据收集
-------
大部分情况下，要收集的信息都是存储在文件里面的，首先就是要从文件里面读取数据。在这里如何实时的读取文件的数据很重要。常用的方法往往是linux中得tailf命令。这时需要考虑tailf命令是否支持文件切割的跟踪，tailf命令对于大量数据写入的情况下是否会有瓶颈。收集之后的数据需要存储，现在来看，比较好的办法就是写入到hadoop中。这就是涉及到写入格式问题。是先对原始数据进行初步的解析（比如日志文件），还是原文存入，在后续需要的时候再做解析。我的看法是先存入数据，然后在已经存储的数据上进行解析和计算。原因很简单，在数据高并发写入的情况下，CPU，I/O，网路都是很大的考验。数据量可能达到上万，几十万每秒的级别。如果是在存储之前做过多的解析工作，不但导致整个收集流程复杂度增加，而且对于整套收集系统的稳定性也是很大的考验。当然这里说的存到hadoop，是为离线计算做准备的。而利用hadoop的map reduce去进一步分析显得更加有意义。

数据分析和计算
------------
在实时环境中，收集的数据是需要实时进行计算的。收集数据的工具和实时计算的工具如何对接是一个比较重要的问题。如果使用scribed作为数据收集的工具的话，那么其中一种方式就是配置为写本地文件，然后按照各个分类对本地文件进行读取放入到队列中，例如redis，这种方式看起来比较土。但是也算是一种可行的方法，实时计算应用比较广泛的就是storm了。可以使用redis的subpub功能作为storm的spout。然后在storm上编写拓扑进行实时计算。

数据存储
-------
这里主要这对实时计算的数据存储，通用的方法有redis, mongodb，hbase。hbase比较庞大，也不是很了解，但是个人觉得redis和mongodb很不错，如果有条件可以使用redis作为数据可视化的后台，使用mongodb作为持久化存储。

数据可视化
---------
说道数据可视化就不得不提github上著名的javascript库[d3](https://github.com/mbostock/d3?source=cc)。而实时性，据我所知，nodejs+socket.io是最佳选择。

总结
---
虽然有这样的一套方法可以用来做这件事情，但是还是有其瓶颈和难点，首先实时性，严格要求的话，是应该越实时越好。但是在实际操作中这个精度越高就越难达到。而这个时候准确性的要求就上来了。我的观点是尽量实时，少延迟，但是一定要准确。就是说五秒前的数据现在刚刚可视化出来，没有关系。但是不可以把5秒前的数据当做是当前的数据。这就是错误，而不叫延迟。而在这个过程中，准确性我们是可以做到的，但是实时性受到资源和实际环境影响只能尽量做到。最后就是一句话，离线数据要保数据的完整性，实时数据要保证数据的准确性。