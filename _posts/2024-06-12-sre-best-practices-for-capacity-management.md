---
title: SRE 容量管理的最佳实践
date: 2024-06-12 20:30:00 +0800
categories: [blog, SRE]
tags: [sre]     # TAG names should always be lowercase
---

本文内容从 [https://static.googleusercontent.com/media/sre.google/zh-CN//static/pdf/login\_winter20\_10\_torres.pdf](https://static.googleusercontent.com/media/sre.google/zh-CN//static/pdf/login_winter20_10_torres.pdf) 翻译而来，建议阅读原文。

作为SRE，你负责确定服务的初始资源需求，并确保服务在面临意外需求时也能合理运行。容量管理是确保你拥有适当数量的资源，以使你的服务具有可扩展性、高效性和可靠性的过程。面向用户和公司内部的服务必须能够应对预期和意外的增长。我们将利用率定义为正在使用的资源的百分比。确定初始资源利用率并预测未来需求是困难的。我们提供估算利用率和识别盲点的方法，并讨论构建冗余以避免故障的好处。你将使用这些信息来设计你的架构，以便通过增加每个服务组件的资源分配，能够有效地线性增加整个服务的容量。

## <span class="ez-toc-section" id="%E5%AE%B9%E9%87%8F%E7%AE%A1%E7%90%86%E5%8E%9F%E5%88%99"></span>容量管理原则<span class="ez-toc-section-end"></span>

在本文的上下文中，服务被定义为提供一组功能的所有二进制文件（服务堆栈）的集合。

成功的容量管理需要从两个复杂的角度分配资源：资源供应，即提供当前运行服务的初始容量，以及容量规划，即确保服务在未来的可靠性。

容量管理的核心必须遵循三个基本原则，以保持服务的可扩展性、可用性和可管理性：

1. 服务必须高效地使用其资源。需要大量资源的大型服务在部署和维护方面成本高昂。
2. 服务必须可靠运行。限制资源容量以提高服务效率可能会使服务面临故障和用户面临中断的风险。在服务效率和可靠性之间存在权衡。
3. 必须预见服务的增长。为服务添加资源可能需要很长时间，并且在部署方面存在现实世界的限制。这可能涉及购买和部署新设备或建设新数据中心。还可能需要增加服务依赖的其他软件系统和基础设施的容量。

## <span class="ez-toc-section" id="%E5%AE%B9%E9%87%8F%E7%AE%A1%E7%90%86%E7%9A%84%E5%A4%8D%E6%9D%82%E6%80%A7"></span>容量管理的复杂性<span class="ez-toc-section-end"></span>

一个大型服务就像一个复杂的有机体，有时其行为是无法预料的。在做出可能改变服务范围的工程决策时，需要考虑以下几个方面：

- **服务性能**：了解服务的不同组件在负载下的表现。
- **服务故障模式**：考虑已知的故障模式以及服务在遭遇这些故障时的表现。同时，考虑服务在遭遇未知故障模式时可能的表现。通过生成可能遇到的瓶颈和服务依赖列表来做好准备。
- **需求**：确定预期的用户数量和流量，用户群的位置，以及使用模式。
- **自然增长**：估算需求如何随时间增长。
- **非自然增长**：牢记添加新功能或服务比预期更成功对长期资源的影响。
- **扩展**：了解服务在增加资源分配时的扩展情况。
- **市场分析**：估算市场变化如何影响你获取额外资源的能力。研究可以提高服务性能、可靠性或效率的新技术及其实施成本。调查你多快能够采用新技术，例如用SSD替换HDD。

容量管理的目标是控制不确定性。在未知的环境中，服务必须现在可用，并且在未来继续运行。这是一个充满挑战但回报丰厚且微妙的权衡平衡：效率与可靠性，准确性与复杂性，以及努力与收益。

使用数据驱动容量决策。即使如此，你仍然会不可避免地犯错，并需要以创造性的方式解决问题。但最终结果是一个可靠的关键业务服务。

- **资源供应**解决的是战术问题：“我如何保持服务现在运行？”
- **容量规划**解决的是战略问题：“我如何保持服务在可预见的未来运行？”

以下章节将详细讨论这些主题。

## <span class="ez-toc-section" id="%E8%B5%84%E6%BA%90%E4%BE%9B%E5%BA%94"></span>资源供应<span class="ez-toc-section-end"></span>

我们的讨论集中在一个响应用户请求并查找数据的服务系统上。然而，你可以将这些原则同样应用于数据存储服务、数据转换服务以及几乎所有可以用计算机完成的事情。

资源供应涉及确定服务所需资源的目标利用率并分配这些资源。目标利用率被定义为允许服务可靠运行的特定资源类别的最高可能利用率。资源类别指的是特定类型的计算资产。CPU、RAM、存储等都是资源类别。

为了给你的服务供应资源，使用需求信号作为输入，并创建带有具体资源分配的生产布局，如图1所示。服务通常使用几种资源类别。

## <span class="ez-toc-section" id="%E8%B5%84%E6%BA%90%E7%9F%AD%E7%BC%BA%E7%9A%84%E5%BD%B1%E5%93%8D"></span>资源短缺的影响<span class="ez-toc-section-end"></span>

资源短缺会导致服务以不同方式失败，具体取决于资源类别。

当资源成为服务关键路径中的瓶颈时，用户会体验到延迟增加。在最坏的情况下，瓶颈会导致请求积压，导致延迟不断增加，最终使排队的请求超时。如果没有缓解计划，服务将无法处理请求，并发生中断。中断会持续到进入的流量减少，使服务能够赶上，或服务重启为止。

经常出现在关键路径中的资源包括：

- 处理能力（Processing power）
- 网络
- 存储吞吐量

当资源在非关键路径中成为瓶颈时，服务在一些非时间关键功能（如维护或异步处理）上会受到延迟。如果这些任务延迟时间足够长，可能会影响服务性能、功能、数据完整性，甚至在极端情况下导致服务中断。

当服务存储空间耗尽时，写入操作会失败。如果某些读取依赖于写入，也可能会失败。例如，如果服务或存储解决方案存储Paxos状态以进行一致性读取，或者存储解决方案跟踪所有访问的数据及其访问时间。

当内存或网络套接字等其他资源不足时，服务可能会崩溃、重启或挂起。资源不足的服务可能会因垃圾回收而频繁抖动，或以其他方式表现异常。这些故障会降低服务的容量，并可能触发需要人工干预解决的级联故障场景。

有关缓解策略，请参见下面的“减少中断影响”部分。

### <span class="ez-toc-section" id="%E4%BC%B0%E7%AE%97%E5%88%A9%E7%94%A8%E7%8E%87"></span>估算利用率<span class="ez-toc-section-end"></span>

由于性质不同，每个服务和每种资源类别的资源使用和目标利用率都是不同的。为了估算特定服务的目标利用率，需要考虑以下几个方面。

#### <span class="ez-toc-section" id="%E5%B3%B0%E5%80%BC%E4%BD%BF%E7%94%A8%E7%8E%87"></span>峰值使用率<span class="ez-toc-section-end"></span>

服务的峰值使用率是指在给定时间段内的最高使用率，这取决于服务的性质和用户群体。商业相关服务的高峰期可能在工作日的早晨，而社交相关服务的高峰期则可能在下午晚些时候、晚上、周末或社交活动（如音乐会、体育赛事等）期间。当发生意外事件时，使用率可能会下降或飙升。全球服务的用户群体分布在不同国家和时区，形成了更复杂的日常流量模式。

假设负载不是恒定的，资源利用率在高峰流量期间不应超过服务分配资源的100%。通过不使用所有资源，服务有足够的容量应对高峰期，并且不会因过度配置而浪费资源。

#### <span class="ez-toc-section" id="%E6%9C%80%E9%AB%98%E5%B3%B0%E5%80%BC%E5%88%A9%E7%94%A8%E7%8E%87"></span>最高峰值利用率<span class="ez-toc-section-end"></span>

即使在高峰期，将服务运行在100%利用率也是一个糟糕的主意。一些软件、语言或平台在CPU使用率达到100%之前会出现异常行为或垃圾回收频繁抖动。如果某个组件的内存利用率达到100%，服务会因内存不足（OOM）错误而崩溃。

将监控精细调整到足够精确，以在微秒甚至秒的时间框架内捕捉资源利用率是繁琐的。因此，难以确定低延迟应用程序的资源使用峰值。

## <span class="ez-toc-section" id="%E5%86%97%E4%BD%99"></span>冗余<span class="ez-toc-section-end"></span>

部署、硬件、软件甚至计划内维护的问题可能会导致服务组件故障或重启。这可能导致小到单个二进制实例崩溃，大到整个服务下线的故障。

冗余是一种系统设计原则，包括仅在替换故障组件时才激活的重复组件。冗余程度由N+x表示，其中N是活跃组件的总数，x是备份组件的数量。因此，N+3表示有三个系统组件可以失败，因为有三个重复组件可以替换它们。同时，无论组件总数（N）是多少，服务都能完全正常运行。

冗余可以在区域内或跨区域应用。区域是指位于与其他区域不同的物理站点的独立故障域，这样网络问题或自然灾害不会同时影响多个区域。

### <span class="ez-toc-section" id="%E5%8C%BA%E5%9F%9F%E5%86%85%E7%9A%84%E5%86%97%E4%BD%99"></span>区域内的冗余<span class="ez-toc-section-end"></span>

在区域内实现冗余相对简单。在区域内，你需要保护服务免受二进制文件或物理机器故障的影响。通常，你可以在每个区域内增加额外的服务二进制文件实例，并使用负载均衡解决方案在二进制文件或机器宕机时重定向流量。所需的冗余程度与基础设施的服务水平协议（SLA）相关。具体来说，SLA考虑了可以同时处于故障状态的机器总数以及在新机器上重新启动二进制文件实例的速度。

需要理解的是，区域内的冗余无法保护你的服务免受导致整个区域瘫痪的故障（如电力、网络、自然灾害等）的影响。

### <span class="ez-toc-section" id="%E8%B7%A8%E5%8C%BA%E5%9F%9F%E7%9A%84%E5%86%97%E4%BD%99"></span>跨区域的冗余<span class="ez-toc-section-end"></span>

跨区域的冗余要复杂得多。你需要防范整个区域的停机风险。通过在多个区域部署副本或服务堆栈的完整复制，可以实现跨区域的冗余，以应对服务在高峰时的负载。注意，每个副本必须有足够的容量来处理所有预期的负载，即使在某些副本宕机时也是如此。正如前文所述，无论副本数量（N）是多少，服务的区域冗余程度定义如下：

- **N+0**：服务正常运行，但不能容忍任何区域宕机
- **N+1**：服务可以承受一个区域宕机
- **N+2**：服务可以在两个区域宕机时仍然运行
- **等等**

虽然这些冗余涉及容量，但也涉及服务架构本身。例如，一致性存储服务通常需要大多数副本处于运行状态，以确保写入操作不会回滚。

为N+2冗余配置服务对可靠性有积极影响：可以计划对整个区域进行维护，但在维护期间冗余会降至N+1。服务仍然可以容忍另一个区域的意外事件。这将冗余降至N+0，但不会导致服务中断。需要注意的是，切换到另一个区域可能会对可见延迟产生影响。

在N+0冗余且无法容忍进一步故障的情况下，你的首要任务是尽快缓解或解决意外事件。一种选择是完成或回滚计划中的维护工作，使服务恢复到N+1状态。否则，任何其他区域发生事件都可能导致用户面临中断。

### <span class="ez-toc-section" id="%E5%86%97%E4%BD%99%E7%9A%84%E6%88%90%E6%9C%AC"></span>冗余的成本<span class="ez-toc-section-end"></span>

服务运行的区域越多，运行任何级别冗余的成本越低。考虑图2所描述的服务。它需要以N+2冗余运行。在第一个设置中，它运行三个副本（N=3），在第二个设置中，它运行五个副本（N=5）。两种配置都有两个备用副本（+2），因此可以承受两个副本的故障。

接下来，检查五副本设置。其副本较小，即使两个副本失败并且两个备用副本都在使用中，仍然有三个活跃副本来分担负载。这导致五副本N+2设置的成本为使用相同冗余程度的三副本服务成本的56.6%。见图2中的计算。

#### <span class="ez-toc-section" id="Figure_2_Example_comparison_of_the_cost_of_resource_provisioning_a_service_with_three_and_five_replicas"></span>Figure 2: Example comparison of the cost of resource provisioning a service with three and five replicas<span class="ez-toc-section-end"></span>

Expected load: 100 requests per second (rps)

**Running N+2 on 3 replicas**

2 replicas can go down (N+2)

3 - 2 = 1 replicas stay up to serve 100 rps

Each replica is provisioned to serve 100 rps/ 1 replica = 100 rps/replica

Total capacity provisioned for is 100 rps/replica x 3 replicas = 300 rps

At level flight, the maximum utilization for N+2 is 100 rps / 300 rps = 33%

**Running N+2 on 5 replicas**

2 replicas can go down (N+2)

5 - 2 = 3 replicas stay up to serve 100 rps

Each replica is provisioned to serve 100 rps / 3 replicas = 34 rps/replica

Total capacity provisioned for is 34 rps/replica x 5 replicas = 170 rps

At level flight, the maximum utilization for N+2 is 100 rps / 170 rps = 59%

The cost of running this service on 5 replicas is 170 / 300 = 56.6% of the cost of runing it on 3 replicas

### <span class="ez-toc-section" id="%E5%90%8C%E8%B4%A8%E5%92%8C%E5%BC%82%E8%B4%A8%E6%9C%8D%E5%8A%A1"></span>同质和异质服务<span class="ez-toc-section-end"></span>

对于同质副本大小的服务，实现冗余比对异质副本大小的服务更容易。

你的服务必须配置以应对最大区域的故障。如果各区域的容量不同（即异质），则每个区域需要不同的容量来应对其他最大区域的不可用性。结果是，较小的区域需要更多的资源，而服务相同负载所需的总体资源更高。

### <span class="ez-toc-section" id="%E5%A4%8D%E5%88%B6%E5%92%8C%E5%88%86%E5%B8%83%E5%BC%8F%E6%B5%81%E9%87%8F"></span>复制和分布式流量<span class="ez-toc-section-end"></span>

冗余配置还取决于服务流量的特征。

无状态服务，例如处理用户请求的Web服务器，接收的流量分布在多个副本之间。读取存储服务的请求也可以分布在不同区域的副本之间。为这些服务配置N+1或N+2是微不足道的，并遵循先前示例中的逻辑。

处理跨区域复制请求（例如写入）的服务行为不同。对实体的每次写入都需要最终写入到每个副本，以保持服务的数据在所有副本之间一致。

当一个副本不可用时，复制的写入请求不会给保持在线的副本造成额外的负载。然而，当不可用的副本恢复在线时会产生成本。这个副本需要追赶在其宕机期间错过的未完成的写入操作。这个操作会增加其负载。保持运行的副本提供恢复副本同步所需的数据，增加了恢复期间所有副本的负载。理想情况下，应将此操作限制在一定范围内，以避免影响整组副本的低延迟流量。

每个服务和每个组件可能会接收到不同比例的复制和分布式流量，在资源配置时需要考虑这一点。

### <span class="ez-toc-section" id="Latency-Insensitive_Processes"></span>Latency-Insensitive Processes<span class="ez-toc-section-end"></span>

一项服务通常会有对延迟不敏感的流程，比如批处理作业、异步请求、维护和实验。然而，这些流程在处理对延迟敏感的生产负载时会给服务增加额外负担。因此，服务需要额外的资源来容纳更高的峰值，增加了成本。你可以通过为这些延迟不敏感的请求分配较低的优先级，或者在低负载时段安排它们，以降低总体峰值来减少额外成本。请注意，这两种解决方案都需要经过适当的测试和谨慎地部署，以防止服务中断。

### <span class="ez-toc-section" id="Additional_Resources_for_the_Unknown"></span>Additional Resources for the Unknown<span class="ez-toc-section-end"></span>

另一个需要考虑的方面是未知因素。在为服务配置资源时，有许多理由增加额外资源：例如，底层库的性能退化由其他团队支持，或者实现团队外部要求（如加密所有RPC）。备用容量可以在出现问题时保持服务按预期性能运行，包括延迟和错误。然而，请记住，这种决定可能很昂贵，因此请确保在可靠性、可预测性和扩展性方面的权衡是值得的。

## <span class="ez-toc-section" id="Capacity_Planning"></span>Capacity Planning<span class="ez-toc-section-end"></span>

容量规划是指确定保证未来供应资源的需求量，而资源配置则是指确定保持服务正常运行所需资源的过程。

### <span class="ez-toc-section" id="Overview_of_Capacity_Planning"></span>Overview of Capacity Planning<span class="ez-toc-section-end"></span>

容量规划概述如下：

与资源配置类似，容量规划试图确定每种计算资产（资源类别）的数量，以维持服务的运行。然而，容量规划涉及在多个时间点上进行这些决策：例如，未来三个月、六个月或一年内的资源需求量。

对于现有的服务，容量规划利用历史需求来预测增长，以便在资源配置的基础上为服务的最大峰值利用率、冗余需求、延迟不敏感的流程和未知因素进行构建。通常，你会希望将计划中新的资源消耗者（包括新服务、营销活动、新功能等）的需求量加入到这个预测中。

对于你的服务中的每个组件，你需要不同数量的每个单独的资源类别。以RAM为例，Web服务器可能需要大量RAM，而代理服务器可能只需要很少。在规划未来容量时，确定单个资源的各种值时，请考虑以下因素：

- 服务中不同组件的数量（数据库、代理、应用）
- 每个组件的实例数量（例如，1个数据库、2个代理、2个应用）
- 你的服务运行的区域（例如，跨区域N+1或N+2）
- 你需要用于预测的数据点数量

容量规划的目标是为未来需求准确地提供资源，以确保服务的可用性、可靠性和性能。

这是一个复杂公式的简单例子，单个资源类别如RAM可能需要你考虑以下因素：  
（不同组件的数量）×（每个组件的实例数量）×（区域数量）×（数据点数量）×（其他影响因素）

正如你所见，当考虑所有服务器类型中的所有资源类别，并加入冗余时，你必须确定的容量值数量呈指数级增长。

### <span class="ez-toc-section" id="%E8%B5%84%E6%BA%90%E9%A2%84%E6%B5%8B"></span>资源预测<span class="ez-toc-section-end"></span>

容量规划是一个极其复杂的过程，因为涉及到许多因素，并且每个因素都可以独立变化。在上面的高级概述基础上，考虑以下因素：

#### <span class="ez-toc-section" id="%E7%BB%84%E4%BB%B6%E7%9A%84%E8%B5%84%E6%BA%90%E7%B1%BB%E5%88%AB"></span>组件的资源类别<span class="ez-toc-section-end"></span>

除了确定组件的总数之外，你还必须考虑每个组件使用的各种资源类别：RAM、CPU、存储、网络等。一个组件可能使用一组资源类别，而其他组件可能有完全不同的组合。如果你的服务由许多组件组成，那么你需要追踪的资源类别集合会迅速增加。

#### <span class="ez-toc-section" id="%E5%A4%9A%E4%B8%AA%E5%8C%BA%E5%9F%9F"></span>多个区域<span class="ez-toc-section-end"></span>

如果你需要在世界各地的许多区域运行，你可以想象一下，为各种机器（Web、数据库服务器、应用程序、代理等）预测单个资源类别（例如CPU）会变得更加困难。加上所有其他机器的所有资源类别，在一个给定时间段（六个月或一年后）内在所有区域内的冗余，以开始你的规划。

#### <span class="ez-toc-section" id="%E6%9C%8D%E5%8A%A1%E9%9C%80%E6%B1%82"></span>服务需求<span class="ez-toc-section-end"></span>

需求取决于新服务的成功和采用率，并且只有在服务启动后才会得知。你必须随着时间更新预测，并纠正长期的预测。要理解你正在准备应对突然的未计划的负载增加，如果忽视可能导致中断。

其他意外事件，如自然灾害、网络中断或停电等，可能会极大地改变你的流量模式。即使是计划中的情况，如社交活动或假期的开始或结束，也可能以意想不到的方式影响你的服务。随着新功能的推出或用户群的变化，很难推断这些事件的变化影响。

不同时区中用户分布的变化也会对服务产生影响。流量可能在一天中更加分散，意外地提高和降低峰值需求。

#### <span class="ez-toc-section" id="Growth"></span>Growth<span class="ez-toc-section-end"></span>

增长取决于你的服务的成功程度。用户可能需要一些时间（和营销活动）来了解你的服务并产生兴趣，兴趣可能会随着时间的推移而缓慢或迅速增长。互联网上的其他服务可能会依赖于你的服务，它们的成功或失败可以直接影响到你的服务。一个成功的外部服务可能会增加你的流量，反之亦然。

社会、经济、政治或其他因素可能会增加或减少你的用户流量。你必须确定你的增长率，并在容量规划会议中考虑这一点。

### <span class="ez-toc-section" id="%E9%A2%84%E6%B5%8B%E7%A4%BA%E4%BE%8B"></span>预测示例<span class="ez-toc-section-end"></span>

为了说明作为服务所有者，你必须正确预测的多个潜在独立资源类别值的多样性，让我们使用一个非常简单的例子：

#### <span class="ez-toc-section" id="%E4%B8%80%E4%B8%AA%E4%B8%A4%E4%B8%AA%E7%BB%84%E4%BB%B6%E6%9C%8D%E5%8A%A1%E7%9A%84%E8%B5%84%E6%BA%90%E7%B1%BB%E5%88%AB"></span>一个两个组件服务的资源类别<span class="ez-toc-section-end"></span>

假设你有一个小型服务，比如一个社交媒体应用。它由两台机器组成，一个是Web服务器，一个是数据库。Web服务器使用CPU和RAM，数据库使用CPU、RAM、HDD存储、HDD吞吐量和SSD存储。这总共是六个独特的资源类别值。请注意，这远远不及实际应用中完整的值集合。

通过拥有三个副本，你现在有18个值需要定义。如果你每季度预测12个月，这个数字将跳到72（每年四个季度 × 18）。

#### <span class="ez-toc-section" id="%E5%BD%B1%E5%93%8D%E4%BD%A0%E6%9C%8D%E5%8A%A1%E7%9A%84%E8%B6%8B%E5%8A%BF"></span>影响你服务的趋势<span class="ez-toc-section-end"></span>

你了解到你的社交媒体服务受季节性趋势的影响。在假期季节（11月-12月）开始时，你的流量会增加，春假期间又会增加一次，夏天开始时又有一个高峰期。你的预测不能只是资源线性增长，你必须考虑到一年中高峰期间的流量增加。

你可能还会在每个月或每周进行批处理任务（如数据清理或数据库整理）时经历类似的高峰期。负载可能每个月或每周都不同，进一步增加了准确估算资源利用率的难度。

## <span class="ez-toc-section" id="Best_Practices"></span>Best Practices<span class="ez-toc-section-end"></span>

我们提出了几种容量管理的最佳实践，以帮助您预见常见的问题和陷阱。

### <span class="ez-toc-section" id="%E8%B4%9F%E8%BD%BD%E6%B5%8B%E8%AF%95"></span>负载测试<span class="ez-toc-section-end"></span>

在目标利用率及以上运行服务的一个小副本，并进行故障转移、缓存故障、发布等测试。评估服务如何对超载做出反应并从中恢复，并通过经验验证资源分配是否足以满足定义的负载。在从数据中推断估计时要小心。如果一个分配了一个 CPU 的二进制实例可以处理每秒 100 个请求，通常可以假设每个分配了一个 CPU 的两个二进制实例总共可以处理每秒 200 个请求。但是，假设一个分配了两个 CPU 的二进制实例可以处理每秒 200 个请求是不合适的。除了处理能力之外，可能还存在其他瓶颈。

### <span class="ez-toc-section" id="%E5%85%A8%E9%9D%A2%E8%AF%84%E4%BC%B0%E5%AE%B9%E9%87%8F"></span>全面评估容量<span class="ez-toc-section-end"></span>

虽然你应该为未知情况增加额外容量，但要避免堆叠过多资源，无意中过度配置服务。然而，提供足够的备用资源，使服务能够承受问题。这可以在服务比预期更成功并且为之进行了配置时，购买一些额外时间来获取资源。

### <span class="ez-toc-section" id="%E5%87%8F%E5%B0%91%E5%81%9C%E6%9C%BA%E7%9A%84%E5%BD%B1%E5%93%8D"></span>减少停机的影响<span class="ez-toc-section-end"></span>

可以通过准备服务以使停机时的影响降低来实现。建议的预防措施包括：

- 优雅降级。当服务不堪重负时，服务会禁用一些非关键功能以减轻资源使用情况。
- 拒绝服务（DoS）攻击防护。提供给服务以防止增加的流量来自恶意方。
- 有效的超时。请求最终会超时，服务会丢弃请求而不会浪费更多资源。
- 负载削减。当服务不堪重负时，服务会快速拒绝请求，允许上层路由层重试请求或快速失败请求。这避免了服务落后和浪费资源在将会超时的请求上的问题。

### <span class="ez-toc-section" id="%E9%85%8D%E9%A2%9D%E7%AE%A1%E7%90%86%E5%92%8C%E9%99%90%E6%B5%81"></span>配额管理和限流<span class="ez-toc-section-end"></span>

部署配额系统有助于限制服务与后端之间的吞吐量，提供对使用同一后端的其他服务的隔离。每当一个服务发送的请求多于预期并达到配额限制时，后端会对服务进行限流，而不是使自己超负荷并影响使用该后端的其他服务。

### <span class="ez-toc-section" id="%E7%9B%91%E6%8E%A7"></span>监控<span class="ez-toc-section-end"></span>

从监控服务中收集的相关指标提供了指导资源配置和容量规划决策的数据。使用上述示例服务作为模型，以下指标非常有用：

#### <span class="ez-toc-section" id="%E8%B4%9F%E8%BD%BD%E6%8C%87%E6%A0%87"></span>负载指标<span class="ez-toc-section-end"></span>

- 每秒的进入请求
- 与延迟无关的负载
- 活跃用户数
- 总用户数

#### <span class="ez-toc-section" id="%E8%B5%84%E6%BA%90%E6%8C%87%E6%A0%87"></span>资源指标<span class="ez-toc-section-end"></span>

- 资源分配
- 实际资源使用情况
- 配额使用情况
- 被限流的请求数量

#### <span class="ez-toc-section" id="%E6%80%A7%E8%83%BD%E6%8C%87%E6%A0%87"></span>性能指标<span class="ez-toc-section-end"></span>

- 延迟
- 错误

#### <span class="ez-toc-section" id="%E9%AB%98%E7%BA%A7%E5%81%A5%E5%BA%B7%E6%8C%87%E6%A0%87%EF%BC%88%E5%8F%AF%E4%BB%A5%E5%B8%AE%E5%8A%A9%E8%BF%87%E6%BB%A4%E5%85%B6%E4%BB%96%E5%8F%97%E6%B1%A1%E6%9F%93%E7%9A%84%E6%8C%87%E6%A0%87%E6%95%B0%E6%8D%AE%EF%BC%89"></span>高级健康指标（可以帮助过滤其他受污染的指标数据）<span class="ez-toc-section-end"></span>

- 服务受到停机影响的时间
- 服务进行维护的时间

### <span class="ez-toc-section" id="%E6%8A%A5%E8%AD%A6"></span>报警<span class="ez-toc-section-end"></span>

使用报警进行资源配置和容量规划，以防止停机。一些有用的报警示例包括：当服务未达到预期的冗余水平，因此配置不足时触发的报警，指示根据预测服务缺乏未来资源的报警，当前性能问题等。

### <span class="ez-toc-section" id="%E8%B5%84%E6%BA%90%E6%B1%A0"></span>资源池<span class="ez-toc-section-end"></span>

资源池是将资源分组，使多个服务共享这些资源，而不是为每个服务提供单独的分配。资源池通常用于降低规划复杂性和减少资源碎片化，从而提高服务的效率。当您实施此策略时，大型服务的规划仍然详细而精确。然而，小型服务使用的资源池作为单个实体进行配置，大致和保守地进行规划。这降低了容量规划的工作量，但以隔离为代价。

## <span class="ez-toc-section" id="%E5%B8%B8%E8%A7%84SRE%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5"></span>常规SRE最佳实践<span class="ez-toc-section-end"></span>

遵循您为任何服务遵循的基本SRE原则。例如，将容量状态存储为版本控制系统中的配置，并要求对任何更改进行同行审查。自动执行，逐步推出所有更改，持续监视您的服务，并准备好回滚如果需要。

在发生故障或其他问题时，进行无过失的事后分析，诚实地从错误中学习，并致力于改进系统以避免重复。

## <span class="ez-toc-section" id="%E8%AF%84%E4%BC%B0%E6%9C%8D%E5%8A%A1%E7%9A%84%E5%AE%B9%E9%87%8F"></span>评估服务的容量<span class="ez-toc-section-end"></span>

在评估新服务或现有服务的容量时，我们建议按照以下步骤确定其资源需求：

1. 估计为满足预期负载所需的资源。使用表1中的模板，并填写不同资源类别的预期服务需求。
2. 计算并考虑服务不同组件的目标利用率。您可能需要进行负载测试来评估： 
  - 峰值使用率
  - 最大峰值利用率
  - 冗余
  - 不受延迟影响的进程
  - 未知因素的备用资源
3. 考虑诸如： 
  - 优先级
  - 地区
  - 服务组件
  - 特定时间点以及未来时间（每月、每季度、6个月、一年等）
4. 进行预测，考虑是否需要按以下方式规划容量： 
  - 优先级
  - 地区
  - 服务组件
  - 每年的时间点数
5. 持续学习容量管理： 
  - 观看视频“分布式服务容量管理的复杂性”以获取有关此主题的扩展技术讨论。
  - 阅读“Capacity Planning” \[2\]的";login:"文章。
  - 查阅谷歌的《Site Reliability Engineering》\[3\]中关于“软件工程在SRE中”，“管理关键状态”和“大规模可靠产品发布”的章节。

|---|
|-----|
|  | 处理器 |  | CPU 类型和数量（核心） |  |
|  | 图形处理单元 |  | GPU 类型和数量 |  |
|  | 存储 |  | HDD 容量（TB） |  |
|  |  | HDD 带宽 |  |
|  |  | HDD IOPS |  |
|  |  | SSD 容量（TB） |  |
|  |  | SSD 带宽 |  |
|  |  | SSD IOPS |  |
|  | 网络 |  | 数据中心内部延迟 |  |
|  |  | 数据中心内部带宽 |  |
|  |  | 数据中心间延迟 |  |
|  |  | 数据中心间带宽 |  |
|  |  | ISP 访问延迟 |  |
|  |  | ISP 访问带宽 |  |
|  | 后端服务和容量 |  | 所需容量 |  |
|  | 其他 |  | AI 加速器 |  |
|  |  | 其他特殊硬件 |  |

Table 1: Resource assessment template

## <span class="ez-toc-section" id="%E6%80%BB%E7%BB%93"></span>总结<span class="ez-toc-section-end"></span>

在这篇文章中，我们讨论了容量管理的组成部分和复杂性。我们将这个话题分成了两个部分：资源配置，它解决了战术性问题，“我如何让服务立即运行？”和容量规划，它解决了战略性问题，“我如何保持服务在可预见的未来运行？”回答这些问题并不是一件简单的事情，每个问题都需要审查您的服务的不同方面。

在资源配置时，要检查各种需求信号（输入）及其对资源分配（输出）的影响。了解服务可能面临的预期高峰需求以及您需要构建的冗余量很重要。您是否考虑过资源短缺和供应商供应的影响？

容量规划需要您尝试预测服务以及更重要的是其负载在不断变化的未来将会如何。为此，您必须完全了解您的服务，例如，您需要确定高峰周期及其发生时间，确定您必须运行的位置数量以及每个位置的不同能力，并且要预期可能影响您的服务的自然、社会甚至法律事件。当需要增加容量时，您是否有批准或资金来应对增长？

虽然我们提出的许多最佳实践都很重要，但遵循可靠的SRE原则有助于简化容量管理：进行适当的负载测试，实施全面的监控和警报，使用源代码控制系统，了解您的服务的优点和缺点，制定容量计划，并且随时准备好应对增长和扩展。

## <span class="ez-toc-section" id="Acknowledgments"></span>Acknowledgments<span class="ez-toc-section-end"></span>

The authors are grateful for the suggestions from JC van ­ Winkel,

Michal Kottman, Grant Bachman, Todd Underwood, Betsy Beyer,

and Salim Virji.

## <span class="ez-toc-section" id="References"></span>References<span class="ez-toc-section-end"></span>

\[1\] L. Quesada Torres, “Complexities of Capacity Management

for Distributed Services,” Google Tech Talk: <https://www>

.youtube.com/watch?v=pOo0oKNM9I8.

\[2\] D. Hixson and K. Guliani, “Capacity Planning,” ;login:,

vol. 40, no. 1 (February 2015): <https://www.usenix.org/system>

/files/login/articles/login\_feb15\_07\_hixson.pdf.

\[3\] B. Beyer, C. Jones, N. R. Murphy, and J. Petoff, eds., Site

Reliability Engineering, Chapters 18, 23, and 27: https://

landing.google.com/sre/sre-book/toc/index.html.