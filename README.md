# title


## Introduction
Have you ever been stuck in a power outage, patiently waiting for the electricity to come back only for DTE’s predictions to be wrong again? What if you could predict how long an outage will last by yourself? As prolonged outages lead to greater disruptions in daily life, it is important energy companies provide consumers with an accurate estimate of how long a power outage will last, so they can prepare, whether it's for a few hours or several days without electricity.

This analysis will explore this dataset which contains climate, geographic, and economic information pertaining to major power outages in the United States from January 2000 – July 2016. This dataset contains 1534 rows (observations) and 55 columns (features). 

## Investigative Question
Specifically, the following analysis will focus on power outages caused by severe weather to answer the following question:
- How does the cause, climate region, and climate category impact the duration of power outages due to severe weather?

## Key Features
The following columns will be used to address this question:
- OUTAGE.DURATION: Duration of outage events (in minutes)
- CAUSE.CATEGORY.DETAIL: Detailed description of the event categories causing the major power outages such as heavy wind, thunderstorm, tornado, etc.
- CLIMATE.REGION: U.S. Climate regions as specified by National Centers for Environmental Information (nine climatically consistent regions in continental U.S.A.). This includes Central, East North Central, Northeast, Northwest, South, Southeast, Southwest, West, and West North Central.
- CLIMATE.CATEGORY: Represents the climate episodes corresponding to the years. The categories—“Warm”, “Cold” or “Normal” episodes of the climate are based on a threshold of ± 0.5 °C for the Oceanic Niño Index (ONI)

## Data Cleaning
To ensure a meaningful and valid data analysis, the following steps were taken to clean and organize the dataset:
**Removing Unnecessary Columns**: The original Excel sheet containing the dataset included empty rows at the top and left, which did not contribute any meaningful information, so the dataset was read in starting from the sixth row and second column.
**Extracting Relevant Information**: Since the research question specifically focuses on power outages caused by severe weather, only the rows where the value of CAUSE.CATEGORY was “severe weather” were kept.
**Dropping Missing Values in the Response**: Since the analysis aims to predict outage duration, any rows where the value of OUTAGE.DURATION was NaN were dropped since this analysis utilizes supervised learning, which requires a known target value in order to train the model. Furthermore, rows where OUTAGE.DURATION were equal to 0 were also dropped, since it is unreasonable for a power outage to last 0 minutes, and will lead to lower predictions than it should.
**Creating New Features**: In order to make the response more interpretable, a new column, OUTAGE.DURATION.HOURS was created by dividing OUTAGE.DURATION by 60. This new column holds the outage duration in hours rather than minutes.
**Handling Missing Values in the Predictors**: Each predictor column in the reduced dataframe was checked for missing values, revealing 175 NaN values in the CAUSE.CATEGORY.DETAIL column and 3 NaN values in the CLIMATE.REGION category. This issue was addressed with imputation, as described below. 

## Imputation
Rather than dropping 175 total observations, which is a rather large portion of the data, conditional probabilistic imputation was utilized to fill in the missing values in the CAUSE.CATEGORY.DETAIL and CLIMATE.REGION column. Intuitively, the temperature of a region are relevant indicators to the location of that region as well as the types of weather patterns present and the occurrence of severe weather events. After plotting CAUSE.CATEGORY.DETAIL against CLIMATE.CATEGORY in the figure below, it can be seen that the number of winter storms is much higher for cold climates compared to warm climates. Furthermore, most outages in normal climates were caused by thunderstorms, so it is not unlikely that the missing value associated with a normal climate is also thunderstorm. Similar conclusions can be drawn from the plot of CLIMATE. REGION against CLIMATE.CATEGORY where most cold climates occur in the northeast and most warm climates occur in the southeast.

As such, all observations in the dataset were grouped by CLIMATE.CATEGORY, and within that group, a random sample of CAUSE.CATEGORY.DETAIL values were chosen to fill in the missing values. This ensures that the overall distribution of CAUSE.CATEGORY.DETAIL stays relatively the same and is not skewed toward any particular severe weather event.

After cleaning, the final data frame that will be used for analysis has 741 observations.

## Exploratory Data Analysis

**Frequency of Power Outage Durations**
This histogram shows the frequency of different duration ranges, and was created by partitioning OUTAGE.DURATION.HOURS into 30 bins. Based on the plot, the frequency of different power outage durations is heavily skewed to the right, with most power outages lasting below 75 hours. However, there are a few outliers that lasted around 800 hours (over a month long!). Based on this plot alone, the best estimate for power outage duration would be between 0 - 25 hours. 

**Severe Weather Event vs. Average Outage Duration**
This bar chart shows the average outage duration for each type of severe weather event that has caused a power outage, and was created by grouping the data by CAUSE.CATEGORY.DETAIL and calculating the mean for each group. It can be seen that power outages due to flooding (~280 hours) last the longest and are over twice as long as power outages caused by hurricanes (~116 hours), which are the second longest. By analyzing the relationship between these two variables, more informed estimates can be made to answer the research question. For example, outages caused by tornadoes typically last 70 hours and should be predicted to last longer than outages caused by earthquakes.

**Pivot Table**
This table shows the average power outage duration for each combination of climate category (cold, normal, warm) and climate region (Central, East North Central, Northeast, Northwest, South, Southeast, Southwest, West, and West North Central). Based on the table, it appears that normal climates in the West North Central region have the shortest outages lasting 0.07 hours on average, and warm climates in the Southwest region have the longest outages lasting 431.21 hours on average. It can also be seen that power outages in warm climates usually last longer than outages in normal and cold climates. Additionally, outages in the South, West, and West North Central are generally shorter than outages elsewhere in the United States. 
By combining the information this table provides with the bar chart above, we can answer the research question with even greater granularity. For example, the duration of an outage caused by a thunderstorm in a normal climate is generally shorter than an outage caused by a thunderstorm in a warm climate.

However, each of the cells should be analyzed in context of the problem. For example, cold climates are probably uncommon in the Southwest region, suggesting that the 1.10 hour outage duration is likely based on a limited number of outages and thus not representative of the region’s overall outage patterns. Similarly, the values in the West North Central column are significantly lower than those of any other region, which may indicate a limited number of data points rather than genuinely shorter power outages.

## Problem Identification
The ultimate goal of this report is to predict the length of a power outage caused by severe weather (OUTAGE.DURATION) so power companies can more accurately project when power will be returned, which will ultimately help consumers prepare in terms of supplies and shelter. OUTAGE.DURATION is a quantitative variable since you can calculate the min, max, and average values and they will still make sense in context. So, this prediction problem will be analyzed using a regression model. 

At the time of prediction, the power outage will have already started, so the cause and climate of the region will be known. These factors are also key determinants in the type and severity of weather that can cause an outage; for example, winter storms are much worse in cold climates compared to normal climates and will thus most likely lead to longer power outages. As such, the severe weather event (CAUSE.CATEGORY.DETAIL) along with the climate of where the outage took place (CLIMATE.REGION and CLIMATE.CATEGORY) will be used as predictors in this model. 

The model will be evaluated using Mean Squared Error (MSE), as it penalizes large prediction errors more heavily. This is important in this context since major prediction errors by energy companies can have significant consequences on consumers who are relying on these predictions to prepare. Root mean squared error or RMSE can also be calculated from MSE quite easily and serves as a good measure of the model’s average prediction error.

## Baseline Model
The initial model is a multiple linear regression model with the following features:
Write out equation with y, x, and betas
- Response:
OUTAGE.DURATION
Type: Quantitative
Description: Duration of outage events (in minutes)
- Predictors:
    - CAUSE.CATEGORY.DETAIL: Categorical nominal
    - CLIMATE.REGION: Categorical nominal
    - CLIMATE.CATEGORY: Categorical nominal

Because the three predictors are all categorical variables, they must be one-hot-encoded before they can be used as predictors. The base model was implemented using an sklearn Pipeline composed of a ColumnTransformer with a OneHotEncoder and a LinearRegression model before being fit on the training portion of the dataset. A 75-25 train to test ratio was used for this base model as well as all of the following model iterations. 

The test MSE of this model was 6453 hours^2 and the train MSE of this model was 6179 hours^2. Taking the root of the test MSE yields an “average error” of about 80 hours. The large test MSE and RMSE values indicate that this baseline model is failing to capture significant trends in the power outage duration, and the large difference between the train and test MSE indicates that the current model may be overfitting to specific details or noise in the training dataset. Thus, the current model can definitely be improved through feature engineering and adding more diverse predictors like economic data as well as testing different modeling algorithms designed to reduce overfitting, all of which is explored below.

## Final Model
**Model 2**
The first step taken to improve the baseline model was to engineer a new Season feature out of the Month column. Based on the exploratory data analysis above, there appears to be a relationship between the most common severe weather event and the month, where winter storms were the most common cause in the winter months (December, January, February) and thunderstorms were the most common cause during the summer months (June, July, August). Thus, the 12 months were binned into 4 different seasons with December, January, and February categorized as “winter”; March, April, May categorized as “spring”; June, July, August categorized as “summer”; and September, October, and November categorized as “fall”. This function was applied to the Month column to create a new ‘Seasons’ column. Since it is also categorical, the same process used to build the base model was applied (OneHotEncoding the predictors (now including Seasons) and fitting a LinearRegression model).

The resulting test MSE was 6510 hours^2 and the train MSE was 6078 hours^2. Compared to the base model, the train MSE improved slightly, but the test MSE worsened significantly, which is a sign of overfitting, possibly due to the addition of another predictor, which increases model complexity.

**Model 3**
To try and combat the overfitting, LinearRegression was substituted for Ridge regression, which is designed to reduce overfitting using a penalty term with strength lambda, while still keeping all of the predictors of interest. In order to find the optimal value of the hyperparameter lambda, 5-fold cross validation was used, and implemented using GridSearchCV, which tested lambda values ranging from 10⁻⁵ to 10⁴, increasing by a factor of 10 at each step. The resulting test MSE was 5524 hours^2 and the train MSE was 6595 hours^2. This shows an improvement of about 900 hours^2 compared to the base model’s test MSE. The optimal value for the lambda hyperparameter was 10, meaning the model performed best with a moderately strong penalty term.

**Model 4**
Finally, since all of the current predictors are categorical variables related to climate and weather, out of curiosity, the following two quantitative, economic features were added to the model, neither of which are missing any values in the dataset:
UTIL.CONTRI: ​​Utility industry׳s contribution to the total GSP in the State (expressed as percent of the total real GDP that is contributed by the Utility industry) (in %)
This variable could be relevant since states with stronger utility sectors are likely to experience shorter power outages.
POPPCT_URBAN: Percentage of the total population of the U.S. state represented by the urban population (in %)
This variable may also be relevant since states with larger urban populations tend to rely more heavily on electricity, making it more important for power outages to be resolved quickly.
Exploring the distributions of these two variables in the histograms below, it appears that neither are normally distributed. The distribution of UTIL.CONTRI has a slight right tail as well as outliers on both ends. The distribution of POPPCT_URBAN is moderately skewed to the left, with outliers on the left end. However, ridge regression can perform better when the predictors are normally distributed so outliers and extreme values do not disproportionately influence coefficient estimates. As such, a QuantileTransformer with a normal distribution was applied to both of these features. 

### Final Model
Thus, the final model is composed of a ColumnTransformer that applies a OneHotEncoder for the categorical features, and a QuantileTransformer for the quantitative features. This ColumnTransformer is then combined with a Ridge regression model into a single pipeline, which is trained using GridSearchCV to find the optimal regularization parameter lambda (which  is equal to 10) over 5 folds.

## Conclusion
The final test MSE is 5289 hours^2 (RMSE = 72 hours) and the final train MSE is 6469 hours^2. Compared to the base model’s test and train MSE (6453 hours^2 and 6179 hours^2, respectively), the test MSE decreased significantly, while the train MSE actually increased slightly. Interestingly, the test MSE is lower than the train MSE, which indicates that the final model generalizes better to unseen data compared to the base model. The decrease in test MSE is much larger than the increase in train MSE, which is a good sign that the final model is not overfitting and memorizing noise in the training data set, unlike the base model. The addition of the economic features was also beneficial in explaining more of the variability in the outage duration, even though they aren’t directly related to severe weather like the climate category and region. 

Both test MSE and test RMSE decreased by about 10%, demonstrating an improvement in the final model’s ability to predict power outage duration. However, an error of 72 hours (3 days) on average is still quite large, indicating potential for further improvement through additional engineering.
