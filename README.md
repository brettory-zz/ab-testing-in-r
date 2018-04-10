---
title: "A/B testing in R"
# author: "Brett Ory"
thumbnailImagePosition: left
thumbnailImage: https://lh3.googleusercontent.com/KF0o8OVU_qAEW_WklxVMefejBnbUIeYigadzbKq7wj-l1QG6d62O19sApLTyiYN92qLTHwcsJnRyEOjd0J4AK4f7IYyR-3ya8qU-xFg_EkKoZtb8w88CkPUsQ0Tpe6REz7Y43pnFO4xqX71Ji2xbCLeKjq2AKOOdfTa0ln8gwOPBeEUTXOb7zjWfrhLEIjyoutZGPBKu2ulMroP9eHijRF4NV6XLx7hY8IrppWTQLKfUifZloO4SaMBeKLpVJuCzni33BkOyaF_zZnH4V3C7XpiihOAoGdc5kCuSizW6B_kdzRqeP-6w0xSd5m9XElyRObcYeVz-evV9w2WYJnXrEJO5rdYz3ler3QrVyhy6AfpPZEpNncMWdTDLAbs5QZ5DcY1NKN_87LajNLSCM5gkpdbRhAE2TE0Pbr4f3HD9WnIuvQEHOQGGQN87bMhPZoTNzILOOqxHZxy2izaJAb4q-0ciItoOT-xDbcBgd7GjNQcKfX1lUuxFyVkUNNkNRXofBfrms1l4Fb6pOzxzjsavo6bBvCDZ3UgyO-YQJ7znUQyaMTsy_Oa0eK-segW1L5ngnvYqqrxNfyaSQtecKcAGPmwCDiIUNA7CtJjZwywc=w1307-h735-no
coverImage: https://lh3.googleusercontent.com/KF0o8OVU_qAEW_WklxVMefejBnbUIeYigadzbKq7wj-l1QG6d62O19sApLTyiYN92qLTHwcsJnRyEOjd0J4AK4f7IYyR-3ya8qU-xFg_EkKoZtb8w88CkPUsQ0Tpe6REz7Y43pnFO4xqX71Ji2xbCLeKjq2AKOOdfTa0ln8gwOPBeEUTXOb7zjWfrhLEIjyoutZGPBKu2ulMroP9eHijRF4NV6XLx7hY8IrppWTQLKfUifZloO4SaMBeKLpVJuCzni33BkOyaF_zZnH4V3C7XpiihOAoGdc5kCuSizW6B_kdzRqeP-6w0xSd5m9XElyRObcYeVz-evV9w2WYJnXrEJO5rdYz3ler3QrVyhy6AfpPZEpNncMWdTDLAbs5QZ5DcY1NKN_87LajNLSCM5gkpdbRhAE2TE0Pbr4f3HD9WnIuvQEHOQGGQN87bMhPZoTNzILOOqxHZxy2izaJAb4q-0ciItoOT-xDbcBgd7GjNQcKfX1lUuxFyVkUNNkNRXofBfrms1l4Fb6pOzxzjsavo6bBvCDZ3UgyO-YQJ7znUQyaMTsy_Oa0eK-segW1L5ngnvYqqrxNfyaSQtecKcAGPmwCDiIUNA7CtJjZwywc=w1307-h735-no
metaAlignment: center
coverMeta: in
date: 2018-04-10T21:13:14-05:00
categories: ["Personal projects"]
tags: ["R", "a/b testing", "experiments", "logistic regression"]
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Experimental data

Today I want to talk about two-sample hypothesis testing, or A/B testing in data science parlance. This topic has been on my mind a lot as of late because I'm planning on leaving academia for data science, and that means I may also be transitioning from analyzing survey data to running experiments on unstructured, "big" data. Because academia and business have different goals and resources available to them, the methods they use are also different, even when approaching a similar problem. 

A common question in sociology might be: what are the factors associated with someone doing "X" instead of "Y"? In business, the factors are generally controlled and you have a clear preference--rather than an unbiased interest--in someone doing "X" rather than "Y". What this amounts to is that rather than running regressions on a whole host of variables, you can run experiments. The question then becomes: Is the specific factor "A" or "B" more likely to lead people to do "X" rather than "Y"? Whereas for the first example you would use logistic regression with an outcome variable measured in successes or failures, in the second example you would use the proportions test with outcomes measured in the proportions of people that choose X over Y.  

<br>

### The example

To outline the differences between a logistic regression and the proportions test, I'm going to generate some data for a simple example. 

Let's say our website currently has design "A". We want visitors to our website to click through and buy our product (let's code that as 1), but they also have the option of not buying the product (0). Our current click through rate is 2% and we want to increase our rate by 5% (to 2.1%). The UX team tells us that users will be more likely to respond to website design "B", so we're going to test their hypothesis using a randomized controlled experiment. In our experiment, users will either be exposed to website design "A" or "B" and they can either buy the product "1" or not buy the product "0". 

The first thing we need to know for our experiment is what our sample size should be for an A/B test to have enough statistical power to identify an increase in probability of 5%. We find that in R using the power.prop.test command. 
```{r}
p1 <- 0.02 # purchases / website visitors
p2 <- 1.05*p1 

power.prop.test(p1=p1, p2=p2, power= 0.85)
```

According to the test we need 360,567 visitors seeing web design "A" and the same amount seeing web design "B". Let's say our fictional website is really popular and we get approximately 500,000 visitors in one day. Even so, we will want to run the experiment long enough to make sure we have a good representation of weekday/weekend visitors and that our results are unbiased by external factors such as major holidays, bad weather, etc. Let's say we run the experiment for 7 days, ending up with 3.5 million visitors. Their data looks something like this:

```{r}
# raw data = 1.75 mil visitors see design A and same amount see design B. 
# Click through rate is 2% for A and 2.1% for B
set.seed(123)
A <- rbinom(1750000,1,.02)
B <- rbinom(1750000,1,.021)

# to create a data frame, we need a variable coded "0" if visitor saw web design A
des <- rep(0,1750000)

dfA <- as.data.frame(cbind(A,des)) # create data frame A
dfA$buy <- dfA$A

# and variable "des" should be coded as "1" if visitor saw web design B
des <- rep(1,1750000)

dfB <- as.data.frame(cbind(B,des))
dfB$buy <- dfB$B

# now combine dfA and dfB to df
df <- rbind(dfA[,c("buy","des")],dfB[,c("buy","des")])

# clean up global environment
rm(dfA,dfB,des)
```

As I mentioned before, there are two ways to tackle this problem--if we're concerned with why users click through, we might use a logistic regression, as follows:

## Logistic regression
```{r, warning=F,error=F,message=F}
start_time <- Sys.time()
model1 <- glm(buy ~ des,family=binomial(link='logit'),data=df)
summary(model1)
end_time <- Sys.time()

end_time - start_time
```

Using logistic regression we can regress the likelihood that people buy our product given a specific website design, and we see that design "A", coded 0, leads to fewer buys than design "B", coded 1. Logistic regression works, but at a run time of nearly 30 seconds it's not well suited to answering the simple question of which website design leads to more clicks. For this particular question we can better use a proportions test.

<br>

## Proportions test

R has a function built into the base program which can be used to test for significant differences between proportions. To use it we need to input a frequency table where rows are the different designs and columns are in the order of successes, then failures

```{r,}
# create a frequency table
freqTable <- table(df$des, df$buy)[, c(2,1)] 
freqTable 

# proportion test
start_time <- Sys.time()
prop.test(freqTable, conf.level = .95)
end_time <- Sys.time()

end_time - start_time
```

Here we also see that the proportions are significantly different from each other, but this test only took less than .01 seconds. When all you want to know is if the website design made a difference in the ultimate click through rate, the proportions test is clearly superior. On the other hand, the proportions test can't tell us why users bought the product, for that you need logistic regression.  

This post can be found on [GitHub](https://github.com/brettory/ab-testing-in-r/).

