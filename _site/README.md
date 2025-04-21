# Predicting Power Outage Duration

By Grace Liu (gracela@umich.edu)

# Table of Contents
- [Introduction](#introduction)
- [Data Cleaning](#data-cleaning)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Problem Identification](#problem-identification)
- [Baseline Model](#baseline-model)
- [Final Model](#final-model)
- [Conclusion](#conclusion)


## Introduction
Have you ever been stuck in a power outage, patiently waiting for the electricity to come back only for DTE’s predictions to be wrong again? What if you could predict how long an outage will last by yourself? As prolonged outages lead to greater disruptions in daily life, it is important energy companies provide consumers with an accurate estimate of how long a power outage will last, so they can prepare, whether it's for a few hours or several days without electricity.

This analysis will explore this dataset which contains climate, geographic, and economic information pertaining to major power outages in the United States from January 2000 – July 2016. This dataset contains **1534 rows** (observations) and **55 columns** (features). 

## Investigative Question
Specifically, the following analysis will focus on power outages caused by severe weather to answer the following question:

- **How does the cause, climate region, and climate category impact the duration of power outages due to severe weather?**

## Key Features
The following columns will be used to address this question:
- **`OUTAGE.DURATION`**: Duration of outage events (in minutes)
- **`CAUSE.CATEGORY.DETAIL`**: Detailed description of the event categories causing the major power outages such as heavy wind, thunderstorm, tornado, etc.
- **`CLIMATE.REGION`**: U.S. Climate regions as specified by National Centers for Environmental Information (nine climatically consistent regions in continental U.S.A.). This includes Central, East North Central, Northeast, Northwest, South, Southeast, Southwest, West, and West North Central.
- **`CLIMATE.CATEGORY`**: Represents the climate episodes corresponding to the years. The categories—“Warm”, “Cold” or “Normal” episodes of the climate are based on a threshold of ± 0.5 °C for the Oceanic Niño Index (ONI)

## Data Cleaning
To ensure a meaningful and valid data analysis, the following steps were taken to clean and organize the dataset:

**Removing Unnecessary Columns**: The original Excel sheet containing the dataset included empty rows at the top and left, which did not contribute any meaningful information, so the dataset was read in starting from the sixth row and second column.

**Extracting Relevant Information**: Since the research question specifically focuses on power outages caused by severe weather, only the rows where the value of `CAUSE.CATEGORY` was “severe weather” were kept.

**Dropping Missing Values in the Response**: Since the analysis aims to predict outage duration, any rows where the value of `OUTAGE.DURATION` was NaN were dropped since this analysis utilizes supervised learning, which requires a known target value in order to train the model. Furthermore, rows where `OUTAGE.DURATION` were equal to 0 were also dropped, since it is unreasonable for a power outage to last 0 minutes, and will lead to lower predictions than it should.

**Creating New Features**: In order to make the response more interpretable, a new column, `OUTAGE.DURATION.HOURS` was created by dividing `OUTAGE.DURATION` by 60. This new column holds the outage duration in hours rather than minutes.

**Handling Missing Values in the Predictors**: Each predictor column in the reduced dataframe was checked for missing values, revealing 175 NaN values in the `CAUSE.CATEGORY.DETAIL` column and 3 NaN values in the `CLIMATE.REGION` category. This issue was addressed with imputation, as described below. 

## Imputation
Rather than dropping 175 total observations, which is a rather large portion of the data, **conditional probabilistic imputation** was utilized to fill in the missing values in the `CAUSE.CATEGORY.DETAIL` and `CLIMATE.REGION` column. Intuitively, the temperature of a region are relevant indicators to the location of that region as well as the types of weather patterns present and the occurrence of severe weather events. 

<div style="display: flex; gap: 40px; justify-content: center;">
  <iframe
    src="assets/cause-impute.html"
    width="600"
    height="500"
    frameborder="0"
    style="flex-shrink: 0;"
  ></iframe>

  <iframe
    src="assets/region-impute.html"
    width="600"
    height="500"
    frameborder="0"
    style="flex-shrink: 0;"
  ></iframe>
</div>

After plotting `CAUSE.CATEGORY.DETAIL` against `CLIMATE.CATEGORY` in the figure above, it can be seen that the number of winter storms is much higher for cold climates compared to warm climates. Furthermore, most outages in normal climates were caused by thunderstorms, so it is not unlikely that the missing value associated with a normal climate is also thunderstorm. Similar conclusions can be drawn from the plot of `CLIMATE.REGION` against `CLIMATE.CATEGORY` where most cold climates occur in the northeast and most warm climates occur in the southeast.

As such, all observations in the dataset were grouped by `CLIMATE.CATEGORY`, and within that group, a random sample of `CAUSE.CATEGORY.DETAIL` values were chosen to fill in the missing values. This ensures that the overall distribution of `CAUSE.CATEGORY.DETAIL` stays relatively the same and is not skewed toward any particular severe weather event, as seen in the before and after distributions below.

<div style="display: flex; gap: 20px; justify-content: center;">
  <iframe
    src="assets/before-impute.html"
    width="600"
    height="500"
    frameborder="0"
    style="flex-shrink: 0;"
  ></iframe>

  <iframe
    src="assets/after-impute.html"
    width="600"
    height="500"
    frameborder="0"
    style="flex-shrink: 0;"
  ></iframe>
</div>

After cleaning, the final data frame that will be used for analysis has 741 observations. The head of the cleaned dataset can be seen below:

|   OBS |   YEAR |   MONTH | U.S._STATE   | POSTAL.CODE   | NERC.REGION   | CLIMATE.REGION     |   ANOMALY.LEVEL | CLIMATE.CATEGORY   | OUTAGE.START.DATE   | OUTAGE.START.TIME   | OUTAGE.RESTORATION.DATE   | OUTAGE.RESTORATION.TIME   | CAUSE.CATEGORY   | CAUSE.CATEGORY.DETAIL   |   HURRICANE.NAMES |   OUTAGE.DURATION |   DEMAND.LOSS.MW |   CUSTOMERS.AFFECTED |   RES.PRICE |   COM.PRICE |   IND.PRICE |   TOTAL.PRICE |   RES.SALES |   COM.SALES |   IND.SALES |   TOTAL.SALES |   RES.PERCEN |   COM.PERCEN |   IND.PERCEN |   RES.CUSTOMERS |   COM.CUSTOMERS |   IND.CUSTOMERS |   TOTAL.CUSTOMERS |   RES.CUST.PCT |   COM.CUST.PCT |   IND.CUST.PCT |   PC.REALGSP.STATE |   PC.REALGSP.USA |   PC.REALGSP.REL |   PC.REALGSP.CHANGE |   UTIL.REALGSP |   TOTAL.REALGSP |   UTIL.CONTRI |   PI.UTIL.OFUSA |   POPULATION |   POPPCT_URBAN |   POPPCT_UC |   POPDEN_URBAN |   POPDEN_UC |   POPDEN_RURAL |   AREAPCT_URBAN |   AREAPCT_UC |   PCT_LAND |   PCT_WATER_TOT |   PCT_WATER_INLAND | OUTAGE.START        | OUTAGE.RESTORATION   |   OUTAGE.DURATION.HOURS |
|------:|-------:|--------:|:-------------|:--------------|:--------------|:-------------------|----------------:|:-------------------|:--------------------|:--------------------|:--------------------------|:--------------------------|:-----------------|:------------------------|------------------:|------------------:|-----------------:|---------------------:|------------:|------------:|------------:|--------------:|------------:|------------:|------------:|--------------:|-------------:|-------------:|-------------:|----------------:|----------------:|----------------:|------------------:|---------------:|---------------:|---------------:|-------------------:|-----------------:|-----------------:|--------------------:|---------------:|----------------:|--------------:|----------------:|-------------:|---------------:|------------:|---------------:|------------:|---------------:|----------------:|-------------:|-----------:|----------------:|-------------------:|:--------------------|:---------------------|------------------------:|
|     1 |   2011 |       7 | Minnesota    | MN            | MRO           | East North Central |            -0.3 | normal             | 2011-07-01 00:00:00 | 17:00:00            | 2011-07-03 00:00:00       | 20:00:00                  | severe weather   | thunderstorm            |               nan |              3060 |              nan |                70000 |       11.6  |        9.18 |        6.81 |          9.28 | 2.33292e+06 | 2.11477e+06 | 2.11329e+06 |   6.56252e+06 |      35.5491 |      32.225  |      32.2024 |         2308736 |          276286 |           10673 |           2595696 |        88.9448 |        10.644  |       0.411181 |              51268 |            47586 |          1.07738 |                 1.6 |           4802 |          274182 |       1.75139 |             2.2 |      5348119 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 | 2011-07-01 17:00:00 | 2011-07-03 20:00:00  |                    51   |
|     3 |   2010 |      10 | Minnesota    | MN            | MRO           | East North Central |            -1.5 | cold               | 2010-10-26 00:00:00 | 20:00:00            | 2010-10-28 00:00:00       | 22:00:00                  | severe weather   | heavy wind              |               nan |              3000 |              nan |                70000 |       10.87 |        8.19 |        6.07 |          8.15 | 1.46729e+06 | 1.80168e+06 | 1.9513e+06  |   5.22212e+06 |      28.0977 |      34.501  |      37.366  |         2300291 |          276463 |           10150 |           2586905 |        88.9206 |        10.687  |       0.392361 |              50447 |            47287 |          1.06683 |                 2.7 |           4571 |          267895 |       1.70627 |             2.1 |      5310903 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 | 2010-10-26 20:00:00 | 2010-10-28 22:00:00  |                    50   |
|     4 |   2012 |       6 | Minnesota    | MN            | MRO           | East North Central |            -0.1 | normal             | 2012-06-19 00:00:00 | 04:30:00            | 2012-06-20 00:00:00       | 23:00:00                  | severe weather   | thunderstorm            |               nan |              2550 |              nan |                68200 |       11.79 |        9.25 |        6.71 |          9.19 | 1.85152e+06 | 1.94117e+06 | 1.99303e+06 |   5.78706e+06 |      31.9941 |      33.5433 |      34.4393 |         2317336 |          278466 |           11010 |           2606813 |        88.8954 |        10.6822 |       0.422355 |              51598 |            48156 |          1.07148 |                 0.6 |           5364 |          277627 |       1.93209 |             2.2 |      5380443 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 | 2012-06-19 04:30:00 | 2012-06-20 23:00:00  |                    42.5 |
|     5 |   2015 |       7 | Minnesota    | MN            | MRO           | East North Central |             1.2 | warm               | 2015-07-18 00:00:00 | 02:00:00            | 2015-07-19 00:00:00       | 07:00:00                  | severe weather   | wind/rain               |               nan |              1740 |              250 |               250000 |       13.07 |       10.16 |        7.74 |         10.43 | 2.02888e+06 | 2.16161e+06 | 1.77794e+06 |   5.97034e+06 |      33.9826 |      36.2059 |      29.7795 |         2374674 |          289044 |            9812 |           2673531 |        88.8216 |        10.8113 |       0.367005 |              54431 |            49844 |          1.09203 |                 1.7 |           4873 |          292023 |       1.6687  |             2.2 |      5489594 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 | 2015-07-18 02:00:00 | 2015-07-19 07:00:00  |                    29   |
|     6 |   2010 |      11 | Minnesota    | MN            | MRO           | East North Central |            -1.4 | cold               | 2010-11-13 00:00:00 | 15:00:00            | 2010-11-14 00:00:00       | 22:00:00                  | severe weather   | winter storm            |               nan |              1860 |              nan |                60000 |       10.63 |        8.34 |        6.15 |          8.28 | 1.67635e+06 | 1.78614e+06 | 1.90987e+06 |   5.37415e+06 |      31.1928 |      33.2358 |      35.5382 |         2300291 |          276463 |           10150 |           2586905 |        88.9206 |        10.687  |       0.392361 |              50447 |            47287 |          1.06683 |                 2.7 |           4571 |          267895 |       1.70627 |             2.1 |      5310903 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 | 2010-11-13 15:00:00 | 2010-11-14 22:00:00  |                    31   |

## Exploratory Data Analysis

**<u>Univariate Analysis of Power Outage Duration</u>**

  <iframe
    src="assets/eda-1.html"
    width="800"
    height="500"
    frameborder="0"
  ></iframe>

This histogram shows the frequency of different duration ranges, and was created by partitioning `OUTAGE.DURATION.HOURS` into 30 bins. Based on the plot, the frequency of different power outage durations is heavily skewed to the right, with most power outages lasting below 75 hours. However, there are a few outliers that lasted around 800 hours (over a month long!). Based on this plot alone, the best estimate for power outage duration would be between 0 - 25 hours. 


**<u>Bivariate Analysis of Severe Weather Events and Outage Duration</u>**

  <iframe
    src="assets/eda-2.html"
    width="800"
    height="500"
    frameborder="0"
  ></iframe>

This bar chart shows the average outage duration for each type of severe weather event that has caused a power outage, and was created by grouping the data by `CAUSE.CATEGORY.DETAIL` and calculating the mean for each group. It can be seen that power outages due to flooding (~280 hours) last the longest and are over twice as long as power outages caused by hurricanes (~116 hours), which are the second longest. By analyzing the relationship between these two variables, more informed estimates can be made to answer the research question. For example, outages caused by tornadoes typically last 70 hours and should be predicted to last longer than outages caused by earthquakes.

**<u>Aggregate Analysis Climate Category and Climate Region</u>**

|   Central |   East North Central |   Northeast |   Northwest |   South |   Southeast |   Southwest |    West |   West North Central |
|----------:|---------------------:|------------:|------------:|--------:|------------:|------------:|--------:|---------------------:|
|   53.1178 |              77.5772 |     60.4458 |     59.7119 | 56.5302 |     37.1899 |    20.7611  | 44.216  |            1         |
|   53.1128 |              76.8629 |     77.9442 |     80.8542 | 89.8986 |     47.2148 |     4.71667 | 54.2135 |            0.0666667 |
|   72.05   |              55.1289 |     92.8125 |     91.031  | 48.7269 |     47.5672 |   310.303   | 49.0958 |           80.8833    |

This pivot table shows the average power outage duration for each combination of climate category (cold, normal, warm) and climate region (Central, East North Central, Northeast, Northwest, South, Southeast, Southwest, West, and West North Central). Based on the table, it appears that normal climates in the West North Central region have the shortest outages lasting 0.07 hours on average, and warm climates in the Southwest region have the longest outages lasting 431.21 hours on average. It can also be seen that power outages in warm climates usually last longer than outages in normal and cold climates. Additionally, outages in the South, West, and West North Central are generally shorter than outages elsewhere in the United States. 
By combining the information this table provides with the bar chart above, we can answer the research question with even greater granularity. For example, the duration of an outage caused by a thunderstorm in a normal climate is generally shorter than an outage caused by a thunderstorm in a warm climate.

However, each of the cells should be analyzed in context of the problem. For example, cold climates are probably uncommon in the Southwest region, suggesting that the 1.10 hour outage duration is likely based on a limited number of outages and thus not representative of the region’s overall outage patterns. Similarly, the values in the West North Central column are significantly lower than those of any other region, which may indicate a limited number of data points rather than genuinely shorter power outages.

## Problem Identification
The ultimate goal of this report is to predict the length of a power outage caused by severe weather (`OUTAGE.DURATION.HOURS`) so power companies can more accurately project when power will be returned, which will ultimately help consumers prepare in terms of supplies and shelter. `OUTAGE.DURATION.HOURS` is a quantitative variable so, this prediction problem will be analyzed using a regression model. 

At the time of prediction, the power outage will have already started, so the cause and climate of the region will be known. These factors are also key determinants in the type and severity of weather that can cause an outage; for example, winter storms are much worse in cold climates compared to normal climates and will thus most likely lead to longer power outages. As such, the severe weather event (`CAUSE.CATEGORY.DETAIL`) along with the climate of where the outage took place (`CLIMATE.REGION` and `CLIMATE.CATEGORY`) will be used as predictors in this model. 

The model will be evaluated using **Mean Squared Error (MSE)**, as it penalizes large prediction errors more heavily. This is important in this context since major prediction errors by energy companies can have significant consequences on consumers who are relying on these predictions to prepare. **Root mean squared error (RMSE)** can also be calculated from MSE quite easily and serves as a good measure of the model’s average prediction error.

## Baseline Model
The initial model is a multiple linear regression model with the following features:
- **Response**:
    - `OUTAGE.DURATION`: Quantitative
- **Predictors**:
    - `CAUSE.CATEGORY.DETAIL`: Categorical nominal
    - `CLIMATE.REGION`: Categorical nominal
    - `CLIMATE.CATEGORY`: Categorical nominal

Because the three predictors are all categorical variables, they must be one-hot-encoded before they can be used as predictors. The base model was implemented using an `sklearn Pipeline` composed of a `ColumnTransformer` with a `OneHotEncoder` and a `LinearRegression` model before being fit on the training portion of the dataset. A 75-25 train to test ratio was used for this base model as well as all of the following model iterations. 

- **Test MSE: 6948 hours<sup>2</sup>**
- **Train MSE: 6114 hours<sup>2</sup>**
- **Test RMSE: 83 hours**

The large test MSE and RMSE values indicate that this baseline model is failing to capture significant trends in the power outage duration, and the large difference between the train and test MSE indicates that the current model may be overfitting to specific details in the training dataset. Thus, there is significant room for improvement in the current model, whose predictions are, on average, off by 83 hours. The following model iterations explore feature engineering, more diverse predictors, and different modeling algorithms to strengthen the current model's predictive power.

## Final Model
**<u>Iteration 2</u>**

The first step taken to improve the baseline model was to engineer a new `Season` feature out of the `Month` column. 

  <iframe style="border: none; padding: 0; margin: 0; display: block;"
    src="assets/model2.html"
    width="800"
    height="500"
    frameborder="0"
  ></iframe>

Based on the plot above, there appears to be a relationship between the most common severe weather event and the month, where winter storms were the most common cause in the winter months (December, January, February) and thunderstorms were the most common cause during the summer months (June, July, August). Thus, the 12 months were binned into 4 different seasons:

- Winter: December, January, February
- Spring: March, April, May
- Summer: June, July, August
- Fall: September, October, and November

 This function was applied to the Month column to create a new `SEASONS` column. Since it is also categorical, the same process used to build the base model was applied (`OneHotEncode` the predictors (now including `SEASONS`) and fitting a `LinearRegression` model).

- **Test MSE: 6818 hours<sup>2</sup>**
- **Train MSE: 6044 hours<sup>2</sup>**

Compared to the base model, both the test and train MSE improved, however, there stille exists a large difference between the two, which is a sign of overfitting.

**<u>Iteration 3</u>**

To try and combat the overfitting, `LinearRegression` was substituted for `Ridge` regression, which is designed to reduce overfitting using a penalty term with strength &lambda;, while still keeping all of the predictors of interest. In order to find the optimal value of the hyperparameter, 5-fold cross validation was implemented using `GridSearchCV`, which tested &lambda; values ranging from 10<sup>-5</sup> to 10<sup>5</sup>, increasing by a factor of 10 at each step. 

- **Test MSE: 5670 hours<sup>2</sup>**
- **Train MSE: 6539 hours<sup>2</sup>**

This shows an improvement of about 1200 hours<sup>2</sup> compared to the base model’s test MSE. The optimal value for the &lambda; hyperparameter was 10, meaning the model performed best with a moderately strong penalty term.

**<u>Final Iteration</u>**

Finally, since all of the current predictors are categorical variables related to climate and weather, out of curiosity, the following two quantitative, economic features were added to the model, neither of which are missing any values in the dataset:
- **`UTIL.CONTRI`**: ​​Utility industry׳s contribution to the total GSP in the State (expressed as percent of the total real GDP that is contributed by the Utility industry) (in %)
    - This variable could be relevant since states with stronger utility sectors are likely to experience shorter power outages.
- **`POPPCT_URBAN`**: Percentage of the total population of the U.S. state represented by the urban population (in %)
    - This variable may also be relevant since states with larger urban populations tend to rely more heavily on electricity, making it more important for power outages to be resolved quickly.
Exploring the distributions of these two variables in the histograms below, it appears that neither are normally distributed. 

<div style="display: flex; gap: 30px; justify-content: center;">
  <iframe
    src="assets/util-contri.html"
    width="600"
    height="400"
    frameborder="0"
    style="flex-shrink: 0;"
  ></iframe>

  <iframe
    src="assets/poppct-urban.html"
    width="600"
    height="400"
    frameborder="0"
    style="flex-shrink: 0;"
  ></iframe>
</div>

The distribution of `UTIL.CONTRI` has a few outliers on the right end, and the distribution of `POPPCT_URBAN` is moderately skewed to the left, with outliers on the left end. However, ridge regression often performs better when the predictors are normally distributed so outliers and extreme values do not disproportionately influence coefficient estimates. As such, a `QuantileTransformer` with a normal distribution was applied to both of these features. 

**<u>Final Model</u>**

Thus, the final model is composed of a `ColumnTransformer` that applies a OneHotEncoder for the categorical features, and a `QuantileTransformer` for the quantitative features. This `ColumnTransformer` is then combined with a Ridge regression model into a single `pipeline`, which is trained using `GridSearchCV` to find the optimal regularization parameter &lambda; (which  is equal to 10) over 5 folds.

## Conclusion

- **Test MSE: 5470 hours<sup>2</sup>**
- **Train MSE: 6413 hours<sup>2</sup>**
- **Test RMSE: 74 hours**

Compared to the base model’s test and train MSE (6948 hour<sup>2</sup> and 6114 hour<sup>2</sup>, respectively), the test MSE decreased significantly, while the train MSE actually increased slightly. Interestingly, the test MSE is lower than the train MSE, which indicates that the final model generalizes better to unseen data compared to the base model. The decrease in test MSE is much larger than the increase in train MSE, which is a good sign that the final model is not overfitting and memorizing noise in the training data set, unlike the base model. The addition of the economic features was also beneficial in explaining more of the variability in the outage duration, even though they aren’t directly related to severe weather like the climate category and region. 

The test MSE and test RMSE decreased by about 21%, demonstrating an improvement in the final model’s ability to predict power outage duration. However, an error of 74 hours (3 days) on average is still quite large, indicating potential for further improvement through additional engineering.
