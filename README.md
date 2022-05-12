# NFL 4th Down Analysis

Daniel Lin

## A Little Background

This paper assumes some basic knowledge about the rules of American football. For those unfamiliar, a team has four downs to score or gain 10 yards for a 1st down (new set of downs). If the team does not reach a "1st down" or score in those 4 tries, the possession of the football is transferred to the other team. This transfer of possession is important because on 4th down teams can generally do 1 of 3 things:

1. Punt the ball
2. Kick a field goal (3 pts)
3. Go for it

It is common for a team to punt the ball if they are not in field goal range. This is intuitive because the offense do not want to put the other team in a position to score easily when the ball changes possession. 

Kicking a field goal means the ball must be place kicked (technically it can be drop kicked too), and the ball must sail between the uprights of the crossbar.  You can imagine, it is easier to punt the ball far than kick a field goal. The accuracy component adds another layer of difficulty for field goals. 

Going for it means it's 4th down and you attempt to reach the next set of downs or score a TD (7 pts) instead of punting or attempting a field goal. Historically, teams will do this when:

1. They need a TD
2. The ball is just out of field goal range
3. There are a short amount of yards to go for the 4th down

In the 2020 NFC championship game, the Packers were down 8 points with a couple minutes left to go. It was 4th and goal with 8 yards to go and instead of going for it with Rodgers at the helm, they decide to kick a field goal. This proved to be a pivotal decision as the Packers never got the ball back and lost the game.

So what is the optimal strategy on 4th down? Football is a complex game with many moving factors but this paper aims to dissect some of these components.

## 4th Down Statistics

What do teams currently do on 4th down?

| Season | Punt | Field Goal | Pass | Rush | Other |
| --- | --- | --- | --- | --- | --- |
| 2017 | 62.8% | 23.7% | 8.1% | 3.8% | 1.6% |
| 2018 | 60.9% | 22.9% | 10.1% | 4.3% | 1.7% |
| 2019 | 58.9% | 23.3% | 11.4% | 4.4% | 2.0% |
| 2020 | 54.8% | 24.0% | 12.2% | 6.4% | 2.6% |
| 2021 | 54.9% | 23.3% | 13.1% | 6.1% | 2.5% |

### Yards to Go by Play Type

Here is the distribution of yards to go on 4th down plays. 

![4th_rush.png](NFL%204th%20Down%20Analysis%208c29b2be16504a479641a7984fa932b4/4th_rush.png)

![4th_pass.png](NFL%204th%20Down%20Analysis%208c29b2be16504a479641a7984fa932b4/4th_pass.png)

![4th_fg (1).png](NFL%204th%20Down%20Analysis%208c29b2be16504a479641a7984fa932b4/4th_fg_(1).png)

![4th_punt.png](NFL%204th%20Down%20Analysis%208c29b2be16504a479641a7984fa932b4/4th_punt.png)

### Conversion rate by yards-to-go

**Rush Plays**

| Yards To Go | Conversion Rate | Distribution of Plays |
| --- | --- | --- |
| 1 | 66% | 88% |
| 2 | 52% | 8% |
| 3 | 43% | 1% |
| 3+ | 19% | 3% |

It is not surprising that the majority of rush plays happen in short yardage scenarios.

Conversion rates are extremely high for 1 yard to go.

**Pass Plays**

| Yards To Go | Conversion Rate | Distribution of Plays |
| --- | --- | --- |
| 1 | 54% | 17% |
| 2 | 55% | 15% |
| 3 | 44% | 11% |
| 4 | 41% | 11% |
| 5 | 41% | 8% |
| 6 | 47% | 7% |
| 7 | 44% | 5% |
| 8 | 37% | 3% |
| 9 | 21% | 2% |
| 10 | 27% | 8% |
| 10+ | 25% | 13% |

## Bayesian Logistic Model

Our goal is to predict the posterior distribution of converting a 4th down given certain variables.

Since our response variable is a conversion rate, logistic regression instantly comes to mind. Our model is below.

$$
log(\frac{p}{1-p}) = B_0+ B_1x_1+B_2x_2+B_{12}x_1x_2+B_{3}x_3
$$

$$
p \sim Bernoulli(p)
$$

Our dependent variable is 1st Down Conversion.

$$
y = \begin{cases}   1& \text{succesfully convert on 4th down} \\
    0              & \text{turnover on downs}
\end{cases}
$$

The covariates for our model are below:

$$
x_1 = \text{yards to go}
$$

$$
x_2 = \text{is pass play}
$$

$$
x_3 = \text{in red zone}
$$

These should all be quite intuitive. Less yards to go generally implies a higher chance of conversion. The interaction of yards to go with play type is also important as a team is not going to find much success on rushing the ball when there are more than 3 yards to go.

In Bayesian Regression, the difference is our parameters: $B_0, B_1, B_2, B_3$  are all random variables. With the Bambi library, the default priors are weakly informative, that is, we want the data to drive the decisions. 

Below are the posterior distributions of the variables as well as the trace plots. 

![Screen Shot 2021-12-01 at 9.58.12 PM.png](NFL%204th%20Down%20Analysis%208c29b2be16504a479641a7984fa932b4/Screen_Shot_2021-12-01_at_9.58.12_PM.png)

 

We can analyze the 94% highest density interval as well looking at the chart below.

![Screen Shot 2021-12-01 at 9.59.13 PM.png](NFL%204th%20Down%20Analysis%208c29b2be16504a479641a7984fa932b4/Screen_Shot_2021-12-01_at_9.59.13_PM.png)

In contrast to the frequentist approach of confidence intervals, the 94% HDI set is equivalent to saying: given our data, there is a 94% probability that the parameter falls within our interval. 

Let's take the exponential of posterior mean estimates to better interpret the coefficients.

| Variable | Coefficient | exp(Coefficient) |
| --- | --- | --- |
| Intercept | 0.962 | 2.62 |
| ToGo | -0.296 | 0.74 |
| IsPass | -0.333 | 0.72 |
| ToGo:IsPass | 0.151 | 1.16 |
| RedZoneInd | -0.007 | 0.99 |

Right away we can see, directionally how each variable affects the odds or probability. One thing we want to note is being in the Red Zone does not seem to have much signal on converting. Our posterior mean coefficient is nearly 0 and our HDI set spans -0.057 to 0.041.

What is the probability of 4th down play with 1 yard to go and a rush play using the posterior means for the coefficients?

That would be 1.93 odds or 66%. Interesting enough, this matches the conversion rate of the historical data.

We can look at the exponential column as a multivariate factor of each variable. To summarize:

- For each yard to go, we expect a .74 multiplier or 26% decrease in odds of converting.
- Pass plays have a 28% decrease in odds of converting in the base case.
- However, our interaction term shows how pass plays convert better than rush plays as yards increase. Each yard to go, pass plays indicate a 16% increase in odds.
- As stated earlier, being in the Red Zone has little to no effect on conversions.

We can illustrate this by plotting probability of conversion to Yards To Go. Rush plays perform better at short yardage situations but it is clear pass plays perform better anything over 2 yards. At 2 yards to go, it seems there is not much difference between the two.

![Screen Shot 2021-12-01 at 11.25.50 PM.png](NFL%204th%20Down%20Analysis%208c29b2be16504a479641a7984fa932b4/Screen_Shot_2021-12-01_at_11.25.50_PM.png)

# Bottom Line

Over the past couple seasons, teams are going for it on 4th more than ever. This is commonly attributed to growing emphasis on analytics. Even then, 30% of punts are done with 1 yard to go. 

I understand there are many more components to the game (points, fatigue, risk, etc), however, it seems to me that going on 4th down in short yardage scenarios should be the norm and not the exception. Even at higher yardage scenarios, pass plays deem to be quite effective on 4th down.  In future iterations, I would love to get more rigorous with the data.

Maybe the Packers should have [gone for it](https://www.si.com/nfl/packers/news/heres-the-analytics-against-lafleurs-fourth-down-decision).

## Data Source

[http://nflsavant.com/about.php](http://nflsavant.com/about.php) (2017-2021 datasets)
