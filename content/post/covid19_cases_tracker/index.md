---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Global COVID-19 Deaths Tracker"
subtitle: ""
summary: "Updated daily"
authors: [databentobox]
tags: [COVID-19, 新冠肺炎]
categories: [COVID-19]
date: 2020-03-16T23:44:31Z
lastmod: 2020-03-16T23:44:31Z
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

It has become a consensus in the data science community that the number of reported COVID-19 cases is not a reliable metric to track the pandemic. It all boils down to the point that there are not enough tests done and many of the people who are infected are asymptomatic. FiveThirtyEight published an [article](https://fivethirtyeight.com/features/coronavirus-case-counts-are-meaningless/) explain this in details.

The number of COVID-19 deaths, however, is more telling. Although it is a bit delayed (i.e. it probably reflects the situation two weeks ago rather than right now), it should be largely proportional to the overall cases unless a mutation creates a more deadly virus or the health care system is overwhelmed.

Mostly for my own peace of mind, I made the plot below, summarising the number of COVID-19 deaths by country. It has been plotted on the log10 scale with reference lines to contextualise the magnitudes. I aim to update this daily using the [dataset](https://github.com/CSSEGISandData/COVID-19) consolidated by [Johns Hopkins CSSE](https://systems.jhu.edu/).

<div>
    <a href="https://plotly.com/~presstofan/92/" target="_blank" title="2019nCov_deaths_by_country_log" style="display: block; text-align: center;"><img src="https://plotly.com/~presstofan/92.png" alt="2019nCov_deaths_by_country_log" style="max-width: 100%;width: 600px;"  width="600" onerror="this.onerror=null;this.src='https://plotly.com/404.png';" /></a>
    <script data-plotly="presstofan:92" src="https://plotly.com/embed.js" async></script>
</div>
