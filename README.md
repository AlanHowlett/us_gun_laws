<img src='/readme_images/main.jpg'>

# Project Motivation
In the United States, it seems we never have to go more than a few weeks without hearing about another mass shooting. With each new incident comes renewed calls to strengthen gun control laws, expand federal background checks, and get rid of assault rifles. Though the opposing faction promptly dismisses each appeal by citing 2nd Amendment rights, other discussions of practicality often emerge. Specifically, the efficacy of such laws is often called into question.

How do we know which laws work and which ones don’t? Are there any laws that can be implemented without infringing on individual rights and liberties? Here, we will use the power of machine learning and artificial intelligence to see if we can find answers to those questions.

# Data Sources
## Center for Disease Control - National Center for Health Statistics
Firearm mortality rates, as well as overall homicide and suicide rates from 1991-2016, was available via CDC's [WONDER](https://wonder.cdc.gov/mortSQL.html) tool.

## Giffords Law Center
Initial analysis utilzed the grading scale from [Giffords](https://lawcenter.giffords.org/scorecard/) to determine if better grades result in lower mortality rates.

## Everytown Research
Rather than rely on an opaque grading scale, [Everytown Research](https://everytownresearch.org/navigator/trends.html?dataset=background_checks) maintains a web database that tracks changes in specific features of gun laws on a state-by-state basis since 1991. Gun laws are broken down into 85 dimensions across 8 categories:

- Background Checks
- Criminals
- Domestic Violence
- Drugs and Alcohol
- Mental Illness
- Minimum Age
- Permitting Process
- Other

## Global Terrorism Database
In addition to homicide rates, we would also like to know what type of impact, if any, gun laws have on mass shootings. University of Maryland's [Global Terrorism Database](https://www.start.umd.edu/gtd/) provides international data for all types of terrorist attacks, including breakdown by weapon type.

# Defining the Variables
The most common metric used to support arguments that gun laws are effective is the firearm mortality rate. Multiple studies show that as gun laws get stronger, firearm mortality rates drop. Here, we see that as state law grades increase, firearm mortality rates decrease:

<img src='/readme_images/firearm_mortality.png' style='width: 500px'>

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

