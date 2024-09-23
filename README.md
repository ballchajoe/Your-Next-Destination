## **Capstone Project: Take Flight**
=========================
[takeoff](/src/takeoff.webp)

## Table of Contents
- [Project Overview](#project-overview)
- [Original Data Dictionary](#original-data-dictionary)
- [Data Dictionary for Model](#data-dictionary-for-model)
- [Dataset](#dataset)
- [Project Organization](#project-organization)
    - [Data Cleaning](#data-cleaning)
    - [Exploratory Data Analysis](#eda)
    - [Pre-processing](#preprocessing)
    - [Modeling](#modeling)
    - [Performance](#performance)
    - [Conclusion/Future Steps](#conclusion--future-steps)
- [Resources](#resources)

## Project Overview
One area of interest for me is transportation, especially for either airline or automotive industry. A challenge an airline could potentially face is flight and fleet logistics. If airlines can know in advance which destinations will be popular in the upcoming month, they can adjust their flight schedules and fleet maintenance to increase or decrease flights to certain destinations. With this logistical knowledge, profit can be made not only from airline ticket sales but also saving from overhead and unneeded flight maintenance. Most major airlines plan their schedule about a year in advance; therefore, this project will forecast up to a year on a monthly basis.

Feel free to check out the notebooks in the [Resources](#resources) section below for the full in-depth analysis.

## Original Data Dictionary
* DEPARTURES_PERFORMED: Completed scheduled flights
* SEATS: Total number of seats of the flights
* PASSENGERS: Total number of passengers on the flights
* CARRIER: Airline name abbreviated
* CARRIER_NAME: Designated airline name
* ORIGIN_AIRPORT_ID: Flight's origin airport ID
* ORIGIN_CITY_NAME: Flight's origin city
* ORIGIN_COUNTRY_NAME: Flight's origin country
* DEST_AIRPORT_ID: Flight's destination airpot ID
* DEST_CITY_NAME: Flight's destination city
* DEST_COUNTRY_NAME: Flight's destination country
* YEAR: Year flight performed
* MONTH: Month flight performed
* DATE: Year-Month-Day

## Data Dictionary for Model
* Datetime index ranging from 2013-01-01 to 2024-05-01
* Destinations:
    * Cities flown from USA with more than 5000 passengers per month

## Dataset
The initial dataset was downloaded from [Bureau of Transportation Statistics](https://www.transtats.bts.gov/DL_SelectFields.aspx?gnoyr_VQ=FJE&QO_fu146_anzr=Nv4%20Pn44vr45). It included all non-stop international flights to and from United States. About a decade of flight data from Jan 2013 to May 2024 was chosen for this project since at the start of this project, the latest data available was up to May 2024. There was about 978,500 rows and the features of this dataset can be found under Original Data Dictionary. The link to the data files can be found in the [Resources](#resources) section below.

## Project Organization

### Data Cleaning
The dataset could only be downloaded by the year so the 12 files were loaded in and merged into one dataframe. As this project is a time series model, a datetime index created by combining `MONTH` and `YEAR` columns as the `DATE` column. There were null values in the dataset for some of the city, country, and carrier name/ID. These were easily filled with the lookup tables that can be found in the [references](/references/) folder. There were also some duplicated rows which were simply dropped. A base point or base country was needed for the model and United States was chosen since this dataset is for non-stop internation flights to and from United States. To make United States the base country, `DEST_COUNTRY_NAME` was filtered countries other than United States. To ensure that the dataset included only the flights that happened, it was filtered for `DEPARTURES_PERFORMED`, `SEATS`, and `PASSENGERS` having value greater than 0. Filtering the dataset narrowed down the destinations to 171 countries.

### EDA
After the dataset was cleaned and filtered, next step was to perform some Exploratory Data Analysis (EDA). The total number of passengers over the time frame was analyzed first to get an overall view of seasonality and trend.
[Total Passengers](/src/EDA/Total%20Passengers.png)
An yearly seasonality with an upwards trend was seen from Jan 2013 to about Jan 2020. A huge drop in passenger volume from Dec 2019 to Apr 2020 was recorded. This was as we all know due to the COVID pandemic. It had a massive impact to the airline industry and this became a sub-focus point in the project. While the main focus of the project was to forecast passenger volume of destinations to attribute it to popularity, this anomaly in the dataset was also explored, analyzed, and evaluted later on. From Apr 2020, the recovery of the airline industry was seen and by 2022, seasonality and trend seemed to return as it was before.

Since the fight data was recorded on a monthly basis and due to yearly seasonality, total number of passengers were detailed monthly for each year.
[Monthly Passengers over the Years](/src/EDA/Total%20Passengers%20Monthly.png)
As it was observed in the overall plot, the trend was upwards and that was evident with 2019 being on top of the stack. The impact of the pandemic was also evident with 2020 being on the bottom, then the recovery each year afterwards. The monthly pattern was the same throughout the years with passenger volume peaking in the summer season.

The analysis continued focusing on the monthly basis data. Top 10 destinations each month over the years were analyzed. 
[Monthly Top 10](/src/EDA/Monthly%20Top%2010.png)
There were destinations that were continuously in the top 10 and their order seem to be consistent such as London, United Kingdom being on top. There were some destinations that periodically made it into the top 10. Then the lockdown due to the pandemic shuffled everything around. The ranking of top 10 changed and new destinations made it to the top 10 once or few times. However, after 2020 things seemed to return to as they were before. Plotting the destinations individually, it was observed how many destinations made it to the monthly top 10 over the years and especially which destinations benefited from the pandemic.
[Monthly Top 10 Individually](src/EDA/Monthly%20Top%2010%20Ind.png)

### Preprocessing
Pre-processing for this time series model included four parts: feature engineering, stationarity, differencing, and plotting the autocorrelation (ACF) and partial autocorrelation functions (PACF). 
Feature engineering was performed to transform the dataset into an input that fit the scope of the project. As forecasting destinations was the objective, the destinations and their number of passengers became the target. From data cleaning, it was determined there were 171 different destinations. Two filters were applied to narrow down these destinations. First filter was to set a minimum number of passengers threshold by finding the monthly top 10 destination with the lowest number of passengers. Second filter was to keep the destinations with less than 12 months of data missing. This was determined from the fact that the longest cumulative lockdown during the pandemic was about 9 to 10 months. After the filtering, the destinations were narrowed down to 61.

Similar to the EDA, the total number of passengers over the time frame was analyzed to get an overall idea.
[Total Passenger Stationarity](src/Preprocessing/Total%20Passenger%20Stationarity.png)
Visually inspecting the plot, it was determined that the data was non-stationary as the rolling mean and rolling standard deviation were not constant; rolling mean was increasing and rolling standard deviation had minor fluctuations. Another visual check for stationarity of a time series model is through decomposition. A time series model is composed of three parts: trend, seasonal, and residual.
[Total Passenger Decomposition](src/Preprocessing/Total%20Passenger%20Decomp.png)
If the time series model shows trend (upward or downward) or the residual is not random like white noise, then it is non-stationary as was the case with the total number of passengers. This was verified through Augmented Dickey-Fuller (ADF) test. The null hypotheses for ADF is that the time series model is non-stationary and since the p-value for this model was greater than 0.05, the null hypotheses was not rejected. Therefore, it was determined differencing will be required to make it stationary. ADF test was applied to each of the 61 destinations and after 3rd order of differencing, all the data was stationary.

An ARIMA model consists of three parts: auto-regressive (AR or p), integrated (I or d), and moving average (MA or q). Component *d* is determined from order of differencing, *p* from the largest lag on the ACF plot, and *q* from the largest lag on the PACF plot. SARIMA is an extension of ARIMA model with seasonality and its components are defined by the non-seasonal components (p,q,d) with seasonal components (P,Q,D,m) where m is the recurring fixed interval. The fixed interval for this model was set to 12 as the seasonality recurred yearly or 12 months. ACF and PACF were plotted for each of the 61 destinations and observed that the largest lags (other than at 0) for ACF was either at 1 or 2 and for PACF was either at 1 or 7.

After the dataset was feature engineered, differentiated, and components for SARIMA model determined, it was ready for modeling.

### Modeling
Two types of time series models were used, SARIMA and Meta's Prophet. For each model, it was trained on the dataset up to 2020 and the entire dataset to compare the impact of the COVID pandemic or the data anomaly. From feature engineering, 171 destinations were filtered down to 61 but it would still be inefficient and time consuming to write code to set up each of the 61 destinations with specific SARIMA model parameters. Luckily with the use of `auto_arima` function in a custom function `smodel`, the 61 destinations were forecasted and plotted. The `auto_arima` function does a grid search based on given range of parameters and for this project, those ranges were concluded in the pre-processing phase. The train, test, and forecast set with confidence intervals were plotted from this model.
Meta's Prophet model was much simpler to set up and quicker to run. It even provided documentation on how to utilize the `holiday` parameter to have the model not capture the COVID lockdown period in its trend. One minor thing was that it required the date column to be labeled `ds` and the target or destination column to be labeled `y`. The internal `plot_plotly` function only plots the train and forecast set with the confidence intervals.

### Performance
Mean Absolute Error (MAE) and Mean Absolute Percentage Error (MAPE) were chosen as the performance metric for these models. For ease of interpretation, MAPE results are summarized below.

|Result|SARIMA (2013-2024)|SARIMA (2013-2020)|Prophet(2013-2024)|Prophet(2013-2020)|
|:----|:----:|:----:|:----:|:----:|
|MAPE|4.4-34%|2.7-29%|2.1-46%|3.2-18%|
|MAX|599%|53%|155%|57%|
|MIN|4.4%|2.7%|2.1%|3.2%|

Models evaluated up to 2020 yielded a narrower band of MAPE. Models evaluated with the entire dataset had some extreme maximum cases. For example, SARIMA (2013-2024) had a destination with 599% MAPE, meaning its forecasted value was almost 600% off from the actual (test) value.

### Conclusion / Future Steps
The models with pre-COVID dataset (2013-2020) seemed to have performed better forecasting since it yieled narrower band of MAPE. However, can or should the anomaly in the data which in this case 2020-2022 be omitted? That's two years or 24 months of missing data! Interestingly enough, 3 of the 4 models came to similar conclusion on forecasting the destinations for the next 12 months.

[SARIMA](/src/Modeling/SARIMA.png)

[SARIMA pre-COVID](/src/Modeling/SARIMA%20pre%20COVID.png)

[Prophet pre-COVID](/src/Modeling/Prophet%20pre%20COVID.png)

Prophet model with the entire dataset forecasted the same destinations and more!

[Prophet](/src/Modeling/Prophet.png)

Which goes to show that anomalies in the data shouldn't be simply ignored as it could lead to interesting discoveries.

Another type of model that could be explored in the future is implementing neural networks. Gated Recurrent Unit (GRU) have been used for time series modeling and would be interesting to compare the result to the SARIMA and Prophet models.

Until next time...have a safe flight!


## Resources 

* [link to data](https://drive.google.com/drive/folders/1T0hff42Qi_SOpBSqS0ZcokF-bIDLpDss?usp=sharing)

* Notebooks
    - [01-data-loading-cleaning](/notebooks/01-data-loading-cleaning.ipynb)
    - [02-eda](/notebooks/02-eda.ipynb)
    - [03-pre-processing](/notebooks/03-pre-processing.ipynb)
    - [04-modeling-findings](/notebooks/04-modeling-findings.ipynb)

* References
    - [L_AIRPORT_ID](/references/L_AIRPORT_ID.csv)
    - [L_CARRIER_HISTORY](/references/L_CARRIER_HISTORY.csv)

* [capstone_joe_cha.yml](/capstone_joe_cha.yml)
    - Conda environment specification

* README.md
    - Project landing page (this page)

* [LICENSE](/LICENSE.md)
    - Project license

[Back to Top](#capstone-project-take-flight)