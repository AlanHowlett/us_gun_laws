<img src='/readme_images/main.jpg'>

## Project Motivation
In the United States, it seems we never have to go more than a few weeks without hearing about another mass shooting. With each new incident comes renewed calls to strengthen gun control laws, expand federal background checks, and get rid of assault rifles. Though the opposing faction promptly dismisses each appeal by citing 2nd Amendment rights, other discussions of practicality often emerge. Specifically, the efficacy of such laws is often called into question.

How do we know which laws work and which ones don’t? Are there any laws that can be implemented without infringing on individual rights and liberties? Here, we will use the power of machine learning and artificial intelligence to see if we can find answers to those questions.

## Data Sources
### Center for Disease Control - National Center for Health Statistics
Firearm mortality rates, as well as overall homicide and suicide rates from 1991-2016, was available via CDC's [WONDER](https://wonder.cdc.gov/mortSQL.html) tool.

### Giffords Law Center
Initial analysis utilzed the grading scale from [Giffords](https://lawcenter.giffords.org/scorecard/) to determine if better grades result in lower mortality rates.

### Everytown Research
Rather than rely on an opaque grading scale, [Everytown Research](https://everytownresearch.org/navigator/trends.html?dataset=background_checks) maintains a web database that tracks changes in specific features of gun laws on a state-by-state basis since 1991. Gun laws are broken down into 85 dimensions across 8 categories:

- Background Checks
- Criminals
- Domestic Violence
- Drugs and Alcohol
- Mental Illness
- Minimum Age
- Permitting Process
- Other

### Global Terrorism Database
In addition to homicide rates, we would also like to know what type of impact, if any, gun laws have on mass shootings. University of Maryland's [Global Terrorism Database](https://www.start.umd.edu/gtd/) provides international data for all types of terrorist attacks, including breakdown by weapon type.

## Redefining the Variables
The most common metric used to support arguments that gun laws are effective is the firearm mortality rate. Multiple studies show that as gun laws get stronger, firearm mortality rates drop. Here, we see that as state law grades increase, firearm mortality rates decrease:

<img src='/readme_images/firearm_mortality.png' style='width: 500px;'>

Despite the clear correlation, gun rights advocates generally point to two problems with this approach:

1. Grading scale is opaque and assumes lower mortality rates are a result of specific laws (assuming the conclusion).
2. People who commit homicide with a firearm are likely to use another weapon if guns are not accessible. If firearm homicide rates go down, but overall homicide rates remain the same, then gun laws don't really help anything

We examine these criticisms by plotting grades against overall homicide and overall suicide rates in a simple linear regression:

<img src='/readme_images/homicide_suicide.png'>

Based on the r-square values, it is observed that suicide rates are much more responsive to changes in gun laws than homicide rates. Interestingly enough, when measuring the impact of neighbor state law grades on homicide rates in the target state, a stronger relationship is observed:

<img src='/readme_images/homicide_suicide_neighbors.png'>

The implication here is that, if we want to predict homicide rates in California, we can do so with better accuracy by examing laws in Arizona, Nevada, and Oregon rather than California law itself.

These results show that there is validity to objections raised by gun rights advocates. Therefore, we take a new approach to analysis of state gun laws by redifining the variables.

- **Independent Variables**: Features of gun laws as defined by Everytown Research in home and target states.
- **Target Variables**: Overall homicide rate and frequency of mass shootings.

Taking this approach, we can gain more insight to specific features of gun laws that are most effective, and we eliminate concerns highlighted in point 2 by gun rights advocates.

It is further worth noting here that homicide rates do not include incidents of self-defense.

## Preprocessing & Feature Engineering
Everytown Research frames features as a series of yes/no questions, though application can be varied. For example, a question that asks if individuals with felonies are restricted from owning guns, the answer might apply to a few predefined categories:

- Handgun Possession
- Handgun Purchase
- Long Gun/Rifle Possession
- Long Gun/Rifle Purchase

All categories are weighted equally. So, responses to questions are ranked according to how restrictive the laws are. For example, a yes response to the above mentioned question for all categories would get a score of 4. If the response was yes for only handgun possession and no for everything else, the score would be 1. The higher the score, the more restrictive that feature of law.

The mean of neighbor state responses are also calculated and used as predictors as well. This provides greater insight to understanding how neighbor state laws can undermine laws in the target state.

## Backward Elimination
It doesn't help to use all 85 features of gun laws to make predictions. Instead, we want to isolate a select few questions that can be used to accurately predict homicide rates and have substantial variance in the rates such that the difference is, in a practical sense, meaningful. 

To accomplish this, a multiple linear regression model was created, and features were removed one at a time until all remaining features were found to be statistically significant. From there, features were ranked based on their correllation coefficients, with the top 20 remaining for more in depth analysis. Below we can see how these questions correlate with one another, as well as with the target variable in the bottom row.

<img src='/readme_images/questions_heatmap.png'>

However, attempting to predict homicide rates with multiple linear regression analysis, we were unable to acheive accuracy rates any better than using grades of neighbor states.

## Machine Learning Algorithms
Six different model architectures were used (including one neural network) with just over 1400 variants of those architectures through use of a hyperparameter gridsearch. Each model was trained on 80% of the data using 5-fold cross-validation, then tested on the remaining 20%. The explained variance scores for the top performing variant of each model can be found below, with XGBoost Regression achieving the highest score on test data. 

<img src='/readme_images/compare_models.png'>

We can also plot the predicted homicide rates against the actual homicide rates to help visualize the accuracy of the model. The blue line represents predicted rates, while red dots represent actual rates.

<img src='/readme_images/xgboost_predictions.png'>

By calculating the difference between predicted rates when all responses point to stricter laws vs weaker laws, we can get the maximum potential reduction in homicide rates. The model was trained and tested across 1,000 splits of data and the distribution of explained variance scores and potential reductions were plotted in histograms. From here, we see that the mean explained variance score was 91.23% and the mean maximum potential reduction in homicide rates is 54.21%

<img src='/readme_images/mean_EVRs.png'>
<img src='/readme_images/mean_reductions.png'>

## Most Relevant Questions
When isolating those questions that, on their own, have an implied potential to reduce rates by a minimum of 10%, we are left with 7 features of gun law. Some of these questions pertain to the target state, while others apply to neighbor states. Here, we can see precisely what those questions are, and we see the count of states that provide a response of "yes" on at least one dimension (handgun purchase, rifle possession, etc.).

<img src='/readme_images/top_questions.png'>

Overall, a few themes emerge. First, a majority of questions relate to neighbor state laws vs target state laws. In addition, we see a lot of questions related to concealed carry permits, as well as police discretion. Finally, we see two questions related to domestic violence and one related to fugitive firearm possession. Surprisingly, fewer than half of the states in the US prohibit fugitives from possessing firearms in 2019.

## Mass Shooting Analysis
The same approach used above was applied by using freqency of mass shootings as a target variable. This was attempted using regression analysis since some states had multiple shootings in a single year, but there was also an attempt to use a classification approach to indicate if a shooting took place at all. However, no model was able to make predictions with any acceptable level of accuracy. The results suggest that the drivers of mass shooting supercede any state law. Therefore, national law was considered.

The **Public Safety and Recreational Firearms Use Protection Act** (otherwise known as the assault weapons ban) was in place from September 1994 to September 2004. Using frequency of occurence for mass shootings and number of fatalities in a given year or single event, we performed a Student's T-Test to determine if a significant difference between means could be observed. For this analysis, we had data from the Global Terrorism database going back to 1982. Initial results of these tests can be found below.

<img src='/readme_images/hypotheses_init.png'>

All results were found to be significant to at least the 90% level, with the number of fatalities per incident significant at the 95% level. However, when we look at the data overtime, a problem was uncovered.

<img src='/readme_images/shooting_time_series.png'>

As we can see for the number of incidents and total fatalities in any given year, most of the data is post 2005. If the ban was truly effective, we should have expected to see a drop from 1994 to 2004, but instead it appears to be more steady until the ban was removed. Some might even argue that they see a consistent trend through the period with the ban. If we consider data only through 2004, we should theoretically be able to see a drop in rates, but all traces of significance disappear.

However, when looking at number of fatalities per incident, we observe something else entirely. We see almost precisely what is expected: a drop around 1994, and a subsequent increase after 2004. This suggests that, while assault weapons bans are unlikely to reduce the frequency of occurence of mass shootings, they *can* improve the likelihood for victims to survive.

## Next Steps & Recommendations
Overall, there are three general recommendations that can be made to states looking to reduce their homicide rates.

1. The fact that so many states allow fugitive firearm possession points to something bigger. It turns out that there are a number of laws that most would assume are already in place nation-wide, but actually aren't. In some cases, federal laws are already in place, but without analagous state laws, state and local officials cannot provide enforcement. Certain, uncontroversial gaps need to be closed by all states nationwide. These include, but are not limited to, the permission to own firearms by persons who:

  - renounced their US citizenship.
  - were found not guilty of a felony by reason of insanity.
  - were dishonorably dischaged from the military.
  - have been convicted of domestic violence misdemeanors.
  

2. The significance of laws pertaining to police discretion allude to a need for local communities to recognize threats in their own neighborhoods. It is unlikely that majorities on either side of the aisle will agree with yielding such power to public officials, though empowerment of immediate family members "to petition a court for temporary removal of guns from a person who poses a danger to self or others" may be an adequate substitute. Otherwise known as red-flag laws, such regulations are relatively new, having only been adopted by 4 states in 2016 and 17 states in 2019. It remains to be seen if these are truly effective, but it does seem like a plausible way forward.

3. Given the amount of influence that state laws have on homicide rates in neighboring states, it is imperative that state governors and legislators work with their regional allies. Rather than passing new laws, states like California are likely to be better off by finding ways to work with Arizona and Nevada, using what leverage they can to close some of the obvious gaps (see recommendation 1). Because the Supreme Court has explicitly ruled that the federal government cannot compel state and local governments to enforce federal law, this is likely a problem that needs solutions to come from local communities.

All of the recommendations above can be further supported with further analysis of the datasets herein. The next steps listed below are numbered according to their corresponding recommendation above:

1. Use correlation analyses to determine what other features of law are correlated with weak laws pertaining to domestic violence and fugitive firearm possession. Taking such an approach has the potential to lead to an identification of a unique bundle of laws that should be in place for every state. Such a bundle can be marketed nationwide, targeting state governments.

2. Develop new models more specific to analyzing the impact of red-flag laws on homicide and suicide rates.

3. Engineer models that identify states which have the greatest impact on their neighbors. For example, Texas may have lax laws, but if they are surrounded by states that also have lax laws, their external influence will be limited. A heatmap across the country would be helpful in this regard, with a "grading scheme" based on laws that impact neighbors relative to the neighbor laws themselves.

Ultimately, there are many ways this analysis can proceed further. For gun rights advocates, one might be inclined to pursue an analysis of these laws on other forms of crime. Do certain laws that limit gun ownership result in increased rates of robbery, car jackings, etc? This is a common argument made by gun rights advocates, so performing such an analysis might help their arguments as well, depending on the results. The objective should not be to prove one sight right and the other wrong, but to get a stronger grasp on the problems as a society. My hope is that this analysis moves us one step closer in that direction.