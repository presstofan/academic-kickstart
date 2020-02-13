---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "新冠肺炎深圳市数据分析 —— 2月9日 Data Analysis of Novel Coronavirus Cases in Shenzhen - 9 Feb"
subtitle: ""
summary: "文章包括"
authors: [Yihui Fan]
tags: [2019-nCov, 新冠肺炎]
categories: [中文博客]
date: 2020-02-08T17:33:09Z
lastmod: 2020-02-08T17:33:09Z
featured: false
draft: true

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

## 太长不看版 (TL;DR)

## 性别

已获得的375例病例中，52.3%为女性，统计学不显著。没有证据证明男性或者女性更加易感。

## 年龄

目前深圳新冠肺炎病例（共375例）年龄分布如下。30-40岁以及60-65岁人数较多，同北京数据一样呈现双峰分布。30-40年龄段人员流动性大，更可能表现为输入性病例。而中老年人病例数量较多则有可能是因为该人群更加易感。随着样本数量增大，曲线将更加接近病毒感染的真是状况，以上猜测也将得到证明（证伪）。

Below chart shows the age distribution of the first 375 cases in Shenzhen. It tentatively shows two peaks, one at age 30-40 and one at age 60-65. This is similar to what we have observed for the first 175 cases in Beijing. The peak consists of younger people could be linked to their mobility while the older group might be more susceptible to infection. With a bigger sample size, the noise can be reduced and we can take another look to confirm this.

<div>
    <a href="https://plot.ly/~presstofan/48/?share_key=dXfJlkEOlUitdzBxJBYh3g" target="_blank" title="2019nCov_shenzhen_age" style="display: block; text-align: center;"><img src="https://plot.ly/~presstofan/48.png?share_key=dXfJlkEOlUitdzBxJBYh3g" alt="2019nCov_shenzhen_age" style="max-width: 100%;width: 600px;"  width="600" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="presstofan:48" sharekey-plotly="dXfJlkEOlUitdzBxJBYh3g" src="https://plot.ly/embed.js" async></script>
</div>

随着时间的推移，更多年轻人被感染（尤其是有很多儿童）。这导致了病例的平均年龄有所下降。这些年轻病例有可能是输入性病例的密切接触者（家人）。另一种可能是，病毒在年轻人体内潜伏期更长，出现症状较晚。我们需要研究个体病例之间的联系而解决这一问题。

Over time, we saw more young people being infected (particularly those younger than 10 years old), dragging down the average age. Those young people could be the close contact of the imported cases. But it is also likely to be the case that young people have longer incubation period, delaying the onset of the symptoms. Understanding the relation between cases would help us to test these hypotheses.

<div>
    <a href="https://plot.ly/~presstofan/50/?share_key=vQXJPFx4GArPXto0esxrwm" target="_blank" title="2019nCov_shenzhen_age_trend" style="display: block; text-align: center;"><img src="https://plot.ly/~presstofan/50.png?share_key=vQXJPFx4GArPXto0esxrwm" alt="2019nCov_shenzhen_age_trend" style="max-width: 100%;width: 600px;"  width="600" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="presstofan:50" sharekey-plotly="vQXJPFx4GArPXto0esxrwm" src="https://plot.ly/embed.js" async></script>
</div>

## 发病到初次就诊时间

发病到初次就诊时间越短，病毒传播的风险则越小。

<div>
    <a href="https://plot.ly/~presstofan/52/?share_key=rC4AdxBFUklqwp0hrLnNuW" target="_blank" title="2019nCov_shenzhen_spread_time" style="display: block; text-align: center;"><img src="https://plot.ly/~presstofan/52.png?share_key=rC4AdxBFUklqwp0hrLnNuW" alt="2019nCov_shenzhen_spread_time" style="max-width: 100%;width: 600px;"  width="600" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="presstofan:52" sharekey-plotly="rC4AdxBFUklqwp0hrLnNuW" src="https://plot.ly/embed.js" async></script>
</div>

## Contact History


## 各城区情况 (Cases by District)

<div>
    <a href="https://plot.ly/~presstofan/60/?share_key=hpd7a2IWyczsHaaG4JsFH7" target="_blank" title="2019nCov_shenzhen_infection_by_district" style="display: block; text-align: center;"><img src="https://plot.ly/~presstofan/60.png?share_key=hpd7a2IWyczsHaaG4JsFH7" alt="2019nCov_shenzhen_infection_by_district" style="max-width: 100%;width: 600px;"  width="600" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="presstofan:60" sharekey-plotly="hpd7a2IWyczsHaaG4JsFH7" src="https://plot.ly/embed.js" async></script>
</div>



<a href="/2019nCov_map/shenzhen.html">深圳新冠肺炎疫情地图 Map of Novel Coronavirus Cases in Shenzhen</a>