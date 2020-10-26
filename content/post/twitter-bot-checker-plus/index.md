---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Twitter Bot Checker App Powered By Botometer"
subtitle: ""
summary: "A Streamlit app for checking whether your Twitter followers are bots"
authors: [Yihui Fan]
tags: [Python, Twitter, Streamlit]
categories: [Social Network]
date: 2020-09-15T21:30:50+01:00
lastmod: 2020-09-15T21:30:50+01:00
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

<small><span>Photo by <a href="https://unsplash.com/@morningbrew?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Morning Brew</a> on <a href="https://unsplash.com/?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span></small>

A couple years back, I have written [a short Python code]({{< ref "/post/twitter-bot-checker/index.md" >}}) that retrieves all the followers for a specific account and check for bots using [Botometer](https://botometer.osome.iu.edu/). However, the Botometer has gone through some [major changes]() which renders the original code obsolete. It is time to create something new!

Many new tools have become available since, including [Streamlit](https://www.streamlit.io/). According to Streamlit's developers:

> Streamlit’s open-source app framework is the easiest way for data scientists and machine learning engineers to create beautiful, performant apps in only a few hours!  All in pure Python. All for free.

Although it might be an overkill, I have decided to create a Streamlit app that serves as a wrapper over the new Botometer API. It essentially provides two simple functions: 1) Collect all the followers for a given account and store them in an SQLite database and 2) Loop through each account and post the data to the Botometer API and receive bot check data. SQLite database is natively supported by Python so no need to run a separate service.

## How to use

### Download and set up the Streamlit app

The app is built with Python 3.7.4 (although other Python versions such as 3.7.x and 3.6.x should also work). **It is easier to start fresh with a new virtual environment** (e.g. "conda create --name botcheck python=3.7" if you are using conda), then running:

```{sh}
git clone https://github.com/presstofan/twitter-bot-checker
cd twitter-bot-checker
pip install -r requirements.txt
streamlit run app.py
```

Now, you should be able to see the app at: [http://localhost:8501](http://localhost:8501/)

### Get the Twitter and Botometer Rapid API credentials

Please see [this page](https://github.com/IUNetSci/botometer-python#rapidapi-and-twitter-access-details) for setting up the credentials. There is a free plan for testing that allows us to check up to 500 accounts per day. If you need more quota, their other paid plans would be ideal.

### Use the app to retrieve followers and check for bots

1. Select the JSON file that contains the Twitter and Botometer credentials. `credentials.json` is a template of the credential file, which can be found with the app. Credential loaded will be cached for later use.
2. Provide an existing database. If not provided, the app will look for database in the temp folder. If no cached database available, a new database will be created.
3. Specify the account whose followers need to be checked, and set the timeframe you want to keep the bot check data.
4. Run "Retrieve Twitter followers", which uses Tweepy API to retrieve all the followers of a specific account. If a database is provided or cached in the temp folder, the app will only add new followers that are not already in the database. If no database is available, the app will create a new database from scratch.
5. Run "Check bot", which call the Botometer Rapid API to check for bots in the database. It will not re-check the followers unless the bot data is expired. A download link will be available at the bottom of the sidebar when the process is completed. Alternatively, you can click the "Download bot check results" button to get the download link.

![Demo Animation](https://github.com/presstofan/twitter-bot-checker/blob/master/app-demo.gif?raw=true "Demo Animation")

### Results

Please check [this section](https://github.com/IUNetSci/botometer-python#botometer-v4) for the details of the output from bot check. You may also find this [blog post](https://cnets.indiana.edu/blog/2020/09/01/botometer-v4/) and [this paper](https://arxiv.org/abs/2006.06867) from the developer of Botometer useful.
