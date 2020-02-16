---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "新冠肺炎北京市数据分析 —— 2月2日 Data Analysis of COVID-19 (Coronavirus) Cases in Beijing - 2 Feb"
subtitle: ""
summary: "文章包括175例病例的特征分析和趋势分析，病例接触史以及各城区数据汇总分析。最后更新于2月3日 Last update on 3 Feb"
authors: [Yihui Fan]
tags: [COVID-19, 新冠肺炎]
categories: [中文博客]
date: 2020-02-02T23:27:25Z
lastmod: 2020-02-02T23:27:25Z
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

{{% alert note  %}}
2020年2月5日更新：

* 数据已传至[Github](https://github.com/presstofan/2019-nCov-data-collection)

{{% /alert %}}

新型肺炎疫情受到国内外重视已经有两周了，北京市卫健委也从21日起开始发布疫情防控的实时数据。无需赘述，这篇日志主要是对北京市已发布的疫情数据进行简单的分析，并希望能从中获得一些潜在信息。

## 数据说明

本次分析数据全部来自于北京市卫健委-新型冠状病毒感染的肺炎疫情防控专栏的[疫情通报](http://wjw.beijing.gov.cn/wjwh/ztzl/xxgzbd/gzbdyqtb/index_1.html)单元。数据主要是文字介绍和图片化的表格，经我录入整理后分析。整理好的数据我以上传到[Github](https://github.com/presstofan/2019-nCov-data-collection)。数据分为各区县统计和病例明细。其中病例明细一共175例，包括年龄，性别，发病时间，初次就诊时间，湖北接触史（不完全）和其他省份接触史（不完全）。各区县的数据包括确诊数量，湖北及其他省份接触史和密切接触史。病例明细截止到2月2日0时。2月3号查看并无更新，我将会继续关注数据发布。各区县数据目前（2月3日）还在以每天一次的速度持续更新。以下病例明细数据分析包括自1月21日到2月1日0时的全部数据。各区县数据分析则到2月2日0时。

## 性别

已获得的175例病例中，51.4%为女性，统计学不显著。没有证据证明男性或者女性更加易感。

## 年龄

感染病例平均年龄为46岁。年龄分布如下。观察年龄分布可发现30-45岁和55-65岁有两个高峰。我推测30-45岁感染病例多为流动人员，因出差，旅游，访亲等原因在外省市感染病毒回京。而55-65岁则为更加易感人群，并且多为前面年轻流动人员的密切接触者。虽然目前认为人群普遍易感，但是春节后返工时期，北京市病例还将以输入病例为主。所以我推测年龄分布在今后的一段时间还将呈双峰模式。

<div>
    <a href="https://plot.ly/~presstofan/36/?share_key=Pyz74oE2rGflAeQOogUu7j" target="_blank" title="2019nCov_beijing_age" style="display: block; text-align: center;"><img src="https://plot.ly/~presstofan/36.png?share_key=Pyz74oE2rGflAeQOogUu7j" alt="2019nCov_beijing_age" style="max-width: 100%;width: 600px;"  width="600" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="presstofan:36" sharekey-plotly="Pyz74oE2rGflAeQOogUu7j" src="https://plot.ly/embed.js" async></script>
</div>

观察下图可发现，病例年龄趋势（趋势用spline smoothing模拟）在40-50岁之间浮动。这也说明了目前北京的感染人群的年龄模式比较固定。随着时间的推移，我将继续观察报告。

<div>
    <a href="https://plot.ly/~presstofan/38/?share_key=EeNHB0yyQADJJ16x4mJgnj" target="_blank" title="2019nCov_beijing_age_trend" style="display: block; text-align: center;"><img src="https://plot.ly/~presstofan/38.png?share_key=EeNHB0yyQADJJ16x4mJgnj" alt="2019nCov_beijing_age_trend" style="max-width: 100%;width: 600px;"  width="600" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="presstofan:38" sharekey-plotly="EeNHB0yyQADJJ16x4mJgnj" src="https://plot.ly/embed.js" async></script>
</div>

## 发病到初次就诊时间

发病到初次就诊时间越短，病毒传播的风险则越小。在统计的175份病例中，超过半数在发病两天内就到医院就诊。而有一少部分病例则超过10天（其中一名患者超过20天）。由于理论传播期较长，找到这些病人的接触者难度将会很大，但这也正是病毒防控的关键。

<div>
    <a href="https://plot.ly/~presstofan/40/?share_key=YEDQUPcGe3Q3RNxzhZmH53" target="_blank" title="2019nCov_beijing_spread_time" style="display: block; text-align: center;"><img src="https://plot.ly/~presstofan/40.png?share_key=YEDQUPcGe3Q3RNxzhZmH53" alt="2019nCov_beijing_spread_time" style="max-width: 100%;width: 600px;"  width="600" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="presstofan:40" sharekey-plotly="YEDQUPcGe3Q3RNxzhZmH53" src="https://plot.ly/embed.js" async></script>
</div>

观察下方趋势图可发现，随着北京市民的病毒防控意识的增强，发病到初次就诊时间逐渐减短。目前发病人员在2天内，甚至是当天就会到医院就诊，减少了病毒传播的风险。这是一个好消息。

<div>
    <a href="https://plot.ly/~presstofan/42/?share_key=OFGQjm2p9clBUxuNOr0qo1" target="_blank" title="2019nCov_beijing_spread_time_trend" style="display: block; text-align: center;"><img src="https://plot.ly/~presstofan/42.png?share_key=OFGQjm2p9clBUxuNOr0qo1" alt="2019nCov_beijing_spread_time_trend" style="max-width: 100%;width: 600px;"  width="600" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="presstofan:42" sharekey-plotly="OFGQjm2p9clBUxuNOr0qo1" src="https://plot.ly/embed.js" async></script>
</div>

## 接触史

截止到2月2日0时，肺炎病毒传播呈现从外省市传入到密切接触传播过渡的趋势。由下图可发现，1月25日之前所有报告病例都有湖北或者外省市接触史。但从26日开始，陆续出现密切接触者发现被感染的病例。这体现了市政府所说的从输入期到传播期的转换。但目前数据表明，绝大多数病例都是由这两种途径感染。根据官方数据，2月2日12时至24时，有"4例否认湖北接触史"。这里的说法并不明确，因为我们不知道这4例是否是密切接触者，如果不是，则说明病毒开始在其他途径传播。监测既没有密切接触史又没有外省市接触史的病例数量成为关键，我会持续跟进这一数据。

<div>
    <a href="https://plot.ly/~presstofan/44/?share_key=m4FQwe3gzytg1eKXMeOzoA" target="_blank" title="2019nCov_beijing_infection_source_trend" style="display: block; text-align: center;"><img src="https://plot.ly/~presstofan/44.png?share_key=m4FQwe3gzytg1eKXMeOzoA" alt="2019nCov_beijing_infection_source_trend" style="max-width: 100%;width: 600px;"  width="600" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="presstofan:44" sharekey-plotly="m4FQwe3gzytg1eKXMeOzoA" src="https://plot.ly/embed.js" async></script>
</div>

## 各区县情况

最后，再来看看各区县的情况。截止到2月3日0时，各区县感染人数如下。病例人数前四位的是海淀，朝阳，大兴和西城。其中朝阳病例统计人数有所下降。这是因为根据国家有关规定，病例归属地原则按发病时的居住地确定。而之前北京市统计数据并不是按照这一标准，因此有小范围的出入，但不影响趋势统计。下图所示，目前大兴，西城病例增速较大。外来人员病例数量在春节期间停滞一段时间后再次出现快速增长，对应返工潮。东城，顺义，延庆，密云，怀柔，平谷，房山和石景山病例数量很低或为零感染，危险较小。

<div>
    <a href="https://plot.ly/~presstofan/46/?share_key=hrzngcLxDqhDMPTO0EefxM" target="_blank" title="2019nCov_beijing_infection_by_district" style="display: block; text-align: center;"><img src="https://plot.ly/~presstofan/46.png?share_key=hrzngcLxDqhDMPTO0EefxM" alt="2019nCov_beijing_infection_by_district" style="max-width: 100%;width: 600px;"  width="600" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="presstofan:46" sharekey-plotly="hrzngcLxDqhDMPTO0EefxM" src="https://plot.ly/embed.js" async></script>
</div>

## 总结

截止到2月2日0时，北京市新型冠状病毒感染的肺炎病例呈一下特点：

* 病毒感染暂无性别差别，男女感染概率相当
* 感染病例年龄呈双峰分布，中年人群流动性高，病例较多
* 由于病毒防控意识的增强，发病到初次就诊时间逐渐减短，有利于疫情控制
* 疫情由输入期向传播期过渡，监测既没有密切接触史又没有外省市接触史的病例数量成为关键
* 目前各区县中海淀，朝阳，大兴和西城病例较多，其中大兴和西城增速较大
* 外来人员病例数量在春节期间停滞一段时间后再次出现快速增长，对应返工潮。

虽然相比较其他城市北京市卫健委的数据发布已经做的很不错，但是还有些不足。以下是我个人的一些建议：

* 保持数据发布的频率和连贯性，保证至少一天一次
* 继续提供新增病例的具体信息，如年龄，性别，发病时间，初次就诊时间，有无其他省市接触史，有无密切接触史等（其中接触史对疫情传播的分析十分重要）
* 尽量避免使用图片发布数据（图片虽然不易被修改，但大大降低了数据的收集难度）

以上数据由我个人分析。我虽然有数据分析经验，但不具备流行病学知识。如有问题请留言指正。最后，祝大家平安！
