---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Simulations"
subtitle: "Simulations of COVID-19 outbreak in central Tokyo area"
summary: ""
authors: [databentobox]
tags: [COVID-19, 新冠肺炎]
categories: [中文博客]
date: 2020-03-01T20:30:43Z
lastmod: 2020-03-01T20:30:43Z
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

In dealing with the outbreak of the coronavirus, China has slapped extraordinary lockdowns on Wuhan and nearby cities, effectively curbing all movement of over 50 million people. Many cities around the world have cancelled school and advising people to work from home, hoping to contain the virus by reducing mobility. But what role transportation plays in spreading the disease in modern megacities? Are those measures effective?

In this post, I will try to answer these questions by running a couple of simulations based on real world data. I will use the simple SIR model as a framework, together with the population density and traffic flow data to simulate a potential outbreak of coronavirus in Central Tokyo area. **Please note that this is by no means an accurate epidemic model as it relies on simple models and strong assumptions.** For example the current model does not consider incubation period and the measures people may take to stop the virus. As such the simulations are only for demonstration.

## Simulations

### Scenario 1: All traffic as usual

### Scenario 2: Schools cancelled

### Scenario 3: People working from home

## Model

The model I used in the simulation is the simple SIR model, which computes the theoretical number of people infected with a contagious illness in a closed population over time. I first encountered this model in one episode of the American crime drama Numb3rs, in which the math genius Charlie attempts to use the SIR model to locate the point of origin of a bioterrorist attack. The model assumes there are three compartments in outbreaks of infectious diseases, **S** for the number **s**usceptible, **I** for the number of **i**nfectious, and **R** for the number **r**ecovered (or immune). Susceptible people catch the disease and become infectious. He/she may pass the disease to other people before recovering. Once recovered,  he/she become immune and can no longer catch the disease again. The model is dynamic, meaning that people can move from one compartment to another following the order of S, I and R. This can be expressed by the following equations:

$$\frac{dS}{dt} = -\frac{\beta IS}{N}$$
$$\frac{dI}{dt} = \frac{\beta IS}{N} - \gamma I$$
$$\frac{dR}{dt} = \gamma I,$$

where *S* is the stock of susceptible population, *I* is the stock of infected, and *R* is the stock of recovered population. $\beta$ and $\gamma$ are the transition rates between different compartments. Between S and I, the transition rate is $\beta$I, where $\beta$ is the average number of contacts per person times the probability of disease transmission in a contact between a susceptible and an infectious subject. Between I and R, the transition rate is $\gamma$ (simply the rate of recovery or death). If the duration of the infection is denoted D, then $\gamma$ = 1/D, since an individual experiences one recovery in D units of time.

$R_{0}$, the basic reproduction number of the virus that has been wildly quoted, can actually be derived from $\beta$ and $\gamma$.

$$R_{0} = \frac{\beta }{\gamma }$$

$R_{0}$ is defined as the number of secondary infections caused by a single primary infection; in other words, it determines the number of people infected by contact with a single infected person before his death or recovery.

Now we have the basic model of the spreading of the infectious disease. But how can we link this to the transportation system of a city? For this, Urban Data Scientist [Gevorg Yeghikyan](https://lexparsimon.github.io/coronavirus/) found a good solution that is based on a [Nature article](https://www.nature.com/articles/s41467-017-02064-4). The modified model can be summarised by the following:

$$
p(t,j)=\frac{\beta_{t}S\_{j,t}(1-\exp(-\sum\_{k}m^{\_{j,k}^{t}}x\_{k,t}y\_{j,t}))}{1+\beta\_{t}y\_{j,t}},
$$

where $\beta_{t}$ is the S-I transition rate at time $t$, $m^{\_{j,k}^{t}}$ is the probability of people moving from location $k$ to location $j$ at time $t$, $x\_{k,t}$ and $y\_{j,t}$ denote the proportion of infected and susceptible population at location $k$ and $j$ at time $t$. $x\_{k,t}$ and $y\_{j,t}$ are defined as follow:

$$x\_{k,t}=\frac{I\_{k,t}}{N\_{k}}$$
$$y\_{j,t}=\frac{S\_{j,t}}{N\_{j}},$$

where $I\_{k,t}$ and $S\_{j,t}$ are the total infected and susceptible cases at location $k$ and $j$, $N\_{k}$ and $N\_{j}$ are the total population of the two locations. Combining the above probability of introducing the disease into a new location with the SIR model we get:

$$S\_{j,t+1}=S\_{j,t}-\frac{\beta\_{j,t}S\_{j,t}I\_{j,t}}{N\_{j}}-\frac{S\_{j,t}\sum\_km\_{j,k}^tx\_{k,t}\beta\_{k,t}}{N\_{j}+\sum\_km\_{j,k}^t}$$

$$I\_{j,t+1}=I\_{j,t}+\frac{\beta\_{j,t}S\_{j,t}I\_{j,t}}{N\_{j}}+\frac{S\_{j,t}\sum\_km\_{j,k}^tx\_{k,t}\beta\_{k,t}}{N\_{j}+\sum\_km\_{j,k}^t}-\gamma I\_{j,t}$$

$$R\_{j,t+1}=R\_{j,t}+\gamma I\_{j,t}$$





## Assumptions

$\beta$: 0.4

$\gamma$: 0.1 (meaning the average recovery day is 10)
