---
title: "Bikes, Accidents and the Positivity Assumption"
date: '2026-02-23T23:28:32+05:00'
math: true
tags: ["causal-inference"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
disableShare: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---

Claims about cause and effect cannot be made without certain assumptions. For an informative walkthrough of what kinds of assumptions need to be made for causal inference, you can see [this lecture by Brady Neal](https://youtu.be/5x_pPemAVxs?si=CTQYYYERhhG3CJXU).

In this article, I'll be going through an example containing positivity assumption violations and what to do about them. First, we'll go over the causal question and data cleanup. Then we'll address our causal assumption.
## Questions about the Question
Do bikes cause more accidents than cars? I think the answer is yes. But I'm also biased. Can we substantiate the claim that bikes cause more accidents than cars?

Let's be more concrete. I'm thinking of my home country Pakistan. So, maybe it would be appropriate to restrict the question to Pakistan only.

We also don't know if vehicle choice affects accidents at all. Maybe road conditions are more to blame than any specific vehicle? How would we figure this out? Also, what do we mean by "affect"? Do we mean a causal effect? If there are multiple causes of an accident, can we estimate the proportions of these causal effects? How do we attribute "blame"? If the proportions are uniform, maybe that means that vehicle choice doesn't matter?

Maybe socioeconomic status also affects how likely an accident is supposed to occur, due to knowledge of traffic laws. We can use level of education as a proxy for this and limit our data to educated drivers.

Additionally, we also want to be clear about our causal model i.e. what causes what. Obviously the vehicle you're driving doesn't directly cause an accident just by driving it, but let's assume that it is a direct cause of an accident for simplicity.

Putting this all together, we have two main questions to ask: For a dataset of educated drivers in Pakistan, does vehicle choice have a causal effect on the occurrence of an accident and, if it does, how much does bike ownership contribute to it?

Let's assume that vehicle choice has *some* causal effect on accidents. Now, our question is, **"For a dataset of educated drivers in Pakistan, how much of a causal effect does bike ownership have on accidents?"** As we'll see below, this question may have to change to accommodate limitations in the data.

## The Data
For the data, I'll use a public dataset available on the Harvard Dataverse, which contains road traffic accident data from 2020-2023 in the city of Rawalpindi.
*M Shujaat Abid. 2024. “Road Traffic Accident Dataset, Rawalpindi-Punjab, Pakistan.” Harvard Dataverse. https://doi.org/10.7910/DVN/4VGTDR.*

It's convenient that I've lived here because I already have prior beliefs about how data will be distributed and some ideas about the causal model. In other words, I arguably have some "domain knowledge".

## Exploratory Data Analysis

First, we load our packages and read from the a file. Note: For brevity, I'll be omitting some messages that appear when running the R code.
```r
> library(tidyverse)
> library(readxl)
> rta <- read_excel(path = "./RTA Data 2020 to July 2023.xlsx")
```

```
Warning messages:
1: Expecting numeric in J21462 / R21462C10: got 'NULL' 
2: Expecting numeric in J27602 / R27602C10: got 'NULL' 
3: Expecting numeric in J29373 / R29373C10: got 'NULL' 
4: Expecting numeric in A31358 / R31358C1: got 'Hospital' 
5: Expecting numeric in B31358 / R31358C2: got 'A van hit the bike and runaway (bike no ML-3762)' 
6: Expecting numeric in E31358 / R31358C5: got 'Alive & unstable' 
7: Expecting numeric in J33186 / R33186C10: got 'NULL' 
```
So, we got some warnings about a few records. Let's take a peek at the data.
```r
> glimpse(rta)
```

```
Rows: 46,189
Columns: 25
$ EcYear                   <dbl> 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2…
$ EcNumber                 <dbl> 31486, 31485, 31483, 31482, 31479, 31477, 3…
$ CallTime                 <dttm> 2020-12-31 22:41:47, 2020-12-31 22:25:00, …
$ EmergencyArea            <chr> "NEAR APS SCHOOL FORT ROAD RWP", "Infront o…
$ TotalPatientsInEmergency <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1…
$ Gender                   <chr> "Male", "Male", "Male", "Male", "Male", "Ma…
$ Age                      <dbl> 27, 20, 48, 45, 22, 50, 18, 25, 18, 19, 22,…
$ HospitalName             <chr> "BBH", "NULL", "BBH", "NULL", "NULL", "NULL…
$ Reason                   <chr> "Bike Slip", "Car hit Footpath", "Rickshaw …
$ responsetime             <dbl> 10, 12, 10, 5, 5, 6, 5, 4, 4, 3, 1, 5, 5, 5…
$ EducationTitle           <chr> "Intermediate", "Illetrate", "Illetrate", "…
$ InjuryType               <chr> "Minor", "Minor", "Single Fracture", "Minor…
$ Cause                    <chr> "Over Speed", "Over Speed", "Over Speed", "…
$ PatientStatus            <chr> "Alive & unstable", "Alive & stable", "Aliv…
$ BicycleInvovled          <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
$ BikesInvolved            <dbl> 1, 0, 0, 0, 2, 2, 1, 1, 1, 0, 1, 1, 0, 2, 1…
$ BusesInvolved            <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
$ CarsInvolved             <dbl> 0, 1, 1, 2, 0, 0, 0, 0, 0, 0, 1, 1, 1, 0, 0…
$ CartInvovled             <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
$ RickshawsInvolved        <dbl> 0, 0, 1, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0…
$ TractorInvovled          <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
$ TrainsInvovled           <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
$ TrucksInvolved           <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
$ VansInvolved             <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
$ OthersInvolved           <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
```

We have over 45,000 records. I think we can safely ignore the few records that gave us a warning when we loaded the data. From the looks of it, they were 4 records with entry errors. Probably safe to ignore.
### Tidying Up The Data

First, we have some spring cleaning to do.
- There are spelling errors in the columns such as `Invovled` vs `Involved`
- There are spelling errors in the rows such as `Illetrate` vs `Illiterate`
- Some columns use a `"NULL"` string instead of `NA` for missing values

Additionally, some columns are unnecessary for our analysis and should be removed:
- `EcNumber`: It's unclear what this means but it's probably some sort of identifier. Interestingly, this number is not unique for each row. There are 3,782 `EcNumber` values that have been re-entered.
	```r
	> prta |> count(EcNumber) |> filter(n > 1)
	# A tibble: 3,792 × 2
	# ...omitted for brevity
	```
- `responsetime`: For this analysis, we don't care about what happened *after* the accident. Only what happened *before*. Response time doesn't help answer our original question.
- `HospitalName`: Similar to `responsetime`
- `TotalPatientsInEmergency`: This could have served as a proxy for the severity of the accident, but we already have `InjuryType` and `PatientStatus`

Removing the extra columns:
```r
> prta
# A tibble: 46,189 × 25
# ...omitted for brevity
> prta <- prta |> select(!c(EcNumber, responsetime, HospitalName, TotalPatientsInEmergency))
> prta
# A tibble: 46,189 × 21
# ...omitted for brevity
```

### Treatment Variable

What I find really interesting are the "Involved" columns. Looking at the `BikesInvolved` column, we can treat it like a binary treatment variable $T$ where $T=1$ when at least one bike is involved and $T=0$ otherwise. That gets rid of 9 columns.

Still, there is one glaring issue. The data records data for a single patient per row. *We don't know what the patient's own vehicle was.* The only chance we have is to infer it from `Reason` and the `Involved` columns. We can get around this by changing the question: "**For a dataset of educated drivers in Pakistan, how much of a causal effect does *bike involvement* have on accidents?**"

## The Positivity Assumption

The positivity assumption states that $P(T=t,X=x)>0$ for all treatments $t$ and covariates $x$. This assumption is violated because there are cases with 0 probability. For example, the probability of all the "Involved" columns being 1 is 0.

### `Involved` columns
Would it be solved if we transformed the data such that only `BikesInvolved` and `OthersInvolved` were left? In this case, we would combine the remaining "Involved" columns into the `OthersInvolved` column in order to not lose data. An easy way to do this would be to make a new column `NonBikesInvolved` and define it as the sum of all the other "Involved" columns other than `BikesInvolved`.

### Positivity and Confounding

This led me to thinking about Positivity in general. The way my dataset is, it's more likely for there to be positivity violations than not, because there are so many possible combinations of covariates. Take the location of accident for example. The location is very unlikely to be exactly the same for different people, especially if the location information is granular. To deal with this, I would have to join together locations and create "regions". But how would I do that? Wouldn't my choice of "region" affect the probabilities of different accident cases? What about other covariates like time? This all assumes that the random variables are discrete. What if they were continuous? What if my treatment were continuous? Wouldn't positivity always be violated because the probability of a single realization is always zero?

It turns out there is a term for this: The Positivity-Unconfoundedness Tradeoff(as described by [Brady Neal](https://youtu.be/4xc8VkrF98w?si=mX4IsES91qjItOyF&t=323)). This is also related to the Curse of Dimensionality in Machine Learning. Generally, controlling for more covariates is an intuitive and simple way to reduce confounding. However, the more covariates we add, the easier it becomes to find positivity violations. In other words, you end up creating subgroups for which we can't calculate causal effects. For example, if I control for location in our accidents dataset(i.e. the `EmergencyArea` column), I may end up with locations in which only bike-related accidents occurred and no accidents occurred for other vehicle types. In this example, I have nothing to compare the bike-related accidents to for that particular location. This is like a medical trial where only old people get the medicine and only young people get the placebo. There is no comparison to make!

### `EmergencyArea`, `CallTime`, `EcYear`
The problem with the `EmergencyArea` covariate is that there are 36882 unique values out of a total of 46189.
```r
> prta |> distinct(EmergencyArea) |> nrow()
[1] 36882
> prta |> select(EmergencyArea) |> nrow()
[1] 46189
```

I expect that if I were to plot these on a map, the locations would appear to cluster. Regardless, positivity is being violated and I have to perform some operation to handle it.

Coincidentally, I was thinking of how traffic conditions affect the likelihood of an accident. Intuitively, higher traffic areas should see higher rates of accidents. Given this intuition, we can transform `EmergencyArea` into a covariate that records how high the traffic is in a given location.

Of course, `CallTime` has the same positivity violation problem:
```r
> prta |> distinct(CallTime) |> nrow()
[1] 40171
> prta |> select(CallTime) |> nrow()
[1] 46189
```

Since traffic can vary by both location and time, we can combine `EmergencyArea` and `CallTime` into a single covariate `TrafficLevel` which records the level of traffic at that location and at that exact time.

This begs the question: how would I estimate `TrafficLevel` based on location and time? This might end up being a whole project so let's just note down the idea for now.

Also, there is overlap between `EcYear` and `CallTime` which should be handled. These two communicate effectively the same information. Although, `CallTime` has about 6000 missing values compared to `EcYear`.
```r
> prta |> filter(is.na(CallTime)) |> nrow()
[1] 5956
> prta |> filter(is.na(EcYear)) |> nrow()
[1] 1
```

### `Age`
This seems straightforward but again, this has the same problem as `EmergencyArea`. If I were to put ages into bins, which bins should I choose? And how many? We want positivity while also managing confounding. So, the question is: does age affect the likelihood of accidents?

Generally, younger people are more likely to be on the road, and more prone to risky driving(as can be seen in studies carried out in [New Zealand](<https://doi.org/10.1016/S0022-4375(01)00059-7>), [Canada, the US, Europe](https://doi.org/10.1016/j.iatssr.2020.08.005), and [Egypt](https://doi.org/10.1016/j.iatssr.2020.08.005)). I will assume that Egypt has similar drivers to Pakistan and use similar bins: young drivers(<=25), middle-aged drivers(26-59) and old drivers(>=60).

```r
> glimpse(prta |> mutate(Age=cut(Age, breaks=c(0,25,59,Inf))))
Rows: 46,189
Columns: 12
...
$ Age              <fct> "(25,59]", "(0,25]", "(25,59]", "(25,59]", "(0,25]…
...
```

## Next Steps

Although we are far from answering the causal question, we've made some progress and the above discussion should make it clear that satisfying the positivity assumption is non-trivial. At this point, one could take a few different steps such as:
- asking data collectors for clarification
- analyzing balance and overlap of covariates using propensity scores
- testing for and remediating violations of other assumptions such as exchangeability, consistency or SUTVA
