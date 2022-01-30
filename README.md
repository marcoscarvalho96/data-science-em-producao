# Rossmann Sales Predict

![](/img/sales-forecast.png)

# Business Problem

Rossmann GmbH is one of the largest drug store chains in Europe with around 56,200 employee and around 4,000 stores. After a weekly meeting with Rossmann's CFO, some store managers came to me proposing a sales forecast project to predict how much their stores are going to sell in the next 6 weeks. The core reason for this project is to support the CFO in deciding which stores to renovate based on estimated sales.

# Understanding the Dataset

We have two separate tables to analyze, the first is the **Sales** table, which contains information such as: date, store, total sale value, number of customers in the store in the given day and informations about holidays. The second table is the **Store**, which contains: id, type, assortment, distance from competitors, promotions and etc.

The data available for the project correspond to a period of 30 months of company sales from January/2013 to June/2015 and what I want with this data is to understand the patterns and then project how much each store are going to sell in the next 6 weeks, that means from early August to mid-September 2015.

# Business Assumptions

To better understand the data behavior and fit the data for the models, I have to assume some things, as you can see below:

- **Competition Distance:** It is expressed in meters but, some records has zero values. So I assume that zero competition distance is the same as there are no competitors close to the store. But, this zero values will cause bias in the ML models, so to fix this problem I assumed a fixed value (100,000 meters) higher than the highest value in dataset.
- **Assortment:** I assumed there is a hierarchy between the assortment types. So, stores that has assortment type C must offer types A and B too.
- **Store Open:** The identifier for days where stores were closed is 0 and closed days are not interest because it also have 0 sales, I removed all records with store open status "closed".
- **Sales Prediction:** In agreement with the CFO, I presumed they would like to see the total sales for each store at the end of the sixth weeks.

My strategy to solve this challenge was based in CRISP-DM Cycle:

![](/img/crisp-ds.png)

# Machine Learning Models Performance Evalutation

Machine Learning is the focus of the Rossmann project, because here is where we are going to find some interesting insights over the data and really predict what the CFO need. I tested four Machine Learning Algorithms: Linear Regression, Lasso Regression, Random Forest Regression and XGBoost Regressor. To understand the algorithms performance I used 3 different metrics: MAE, MAPE and RMSE.

Bellow I leave the results for each model after cross validation:

|       Model Name          |        MAE CV       |     MAPE CV    |      RMSE CV       |
|:-------------------------:|:-------------------:|:--------------:|:------------------:|
| Linear Regression         |  2081.73 +/- 295.63 | 30.26 +/- 1.66 | 2952.52 +/- 468.37 |
| Lasso Regression          |  2116.38 +/- 341.58 | 29.2  +/- 1.18 | 3057.75 +/- 504.26 |
| Random Forest Regressor   |  836.89  +/- 217.42 | 11.61 +/- 2.32 | 1254.75 +/- 316.61 |
| XGBoost Regressor         |  2889.54 +/- 588.52 | 34.54 +/- 1.39 | 3714.69 +/- 456.1  |

I choose to follow the first cycle of CRISP with XGBoost even with worse performance comparing to the other models, because the CFO requested that the project to not go beyond the budget already defined for the analytics area and all Machine Learning projects have to be aligned with budget.

The main concern here is to not overload our servers with a heavy model when put this into production. So I assumed the following questions to choose XGBoost:

- Can the final model be sustained by the business infrastructure?
- Do the results (direct or indirect) affect or are affected by any internal company policy?

In this case, the model will be hosted on a free cloud (Heroku) where we have a space limitation and Random Forest has a size estimated in 1GB, meanwhile the XGBoost is much smaller with a size around 300MB.

After Hyperparameter Fine Tuning with Random Search, the metric problem was solved, increasing a lot the model metrics MAE, MAPE and RMSE.

|    Model Name        |     MAE    |    MAPE%    |     RMSE     |
|:--------------------:|:----------:|:-----------:|:------------:|
|  XGBoost Regressor   |   703.33   |   10.32     |   1,016.12   |
#
After Hyperparameter we had a RMSE better than Random Forest.

Analyzing the metrics results we can see:

1. MAE - tells us that the model can be wrong in its predictions +/- $703.33.

2. MAPE - tells us how much our model can be wrong in a prediction in percentage, that is the same to say that the value of $703.33 is equal to 10.32% of the total value of the prediction.

3. RMSE - tells us that we can be wrong in 921 units of the prediction. Something to be alert is that this metric is sensibly to extreme values, although most cases are higher than the MAE, we only need to be worried is if the **RMSE 2x** is higher than **MAE** is something to look out for and investigate.

![](/img/salesxpredictions.png)

# Business Results

We made a really good progress until now, but only show metrics to the CFO will not help him to understand the predictions and decide which stores worth to be renovated. So bellow I converted the metrics into real business results.

The table bellow shows the TOTAL (in BRL) of predictions. Considering the best and worse scenarios.

|   Scenario     |      Values       |
|:--------------:|:-----------------:|
| predictions    | R$ 283,456,064.00 |
| worst_scenario | R$ 282,668,127.02 |
| best_scenario  | R$ 284,244,032.67 |
#
Below we have a scatter plot with all predictions. Notice that most are centered around a lone parallel to the X axis and MAPE around 11% in Y axis. But, there are points far apart, this is because these are stores for which the forecasts are not so accurate.

![](/img/scatter_plot.png)
#
With all that said, what does this deviation really represent?

Check the table for the 5 best cases:

|store|predictions |worst_scenario|best_scenario|MAE|MAPE (%)|
|-----|------------|--------------|-------------|---|----|
|259  |353,402.3312|533,460.9444  |534,585.6805|562.37|4.3|
|733  |649,454.2500|648,753.2012  |650,155.2987|701.05|4.9|
|672  |307,351.2812|306,909.8079  |307,792.7545|441.47|5.2|
|1089 |371.856,2500|371,280.0184  |372,432.4815|576.23|5.3|
|763  |226,936.0625|226,592.5782  |227,279.5467|343.48|5.4|


And the 5 worst cases:

|store|predictions |worst_scenario|best_scenario|MAE    |MAPE (%)|
|-----|------------|--------------|-------------|-------|----|
|292  |107,567.9375|104,245.3016  |110,890.5733 |3322.64|57.5|
|909  |232,030.0781|224,319.9419  |239,740.2142 |7710.14|51.7|
|876  |198,517.5312|194,561.8566  |202,473.2058 |3955.67|30.7|
|956  |136,210.0937|135,548.4395  |136,871.7479 |661.65 |25.9|
|675  |156,688.7968|155,881.2358  |157,496.3579 |807.56 |25.3|
#
The formula to calculate the above results was as follows:

1. I grouped the results by store and then calculated the sum of the predictions. With this we have the total predicted values for each store.

2. After that we calculate the MAE for each store

3. For the best scenario we sum the MAE to total sale prediction and for the worst scenario we subtract the MAE, then we obtain the results above.

Even in the worst scenario, we have a great result in this project. The MAE is in the hundreds while the sales predictions is in the millions.

# Model in Production

The model was deployed on the Heroku Cloud platform, a free platform from SalesForce. Below we have an example of the return in JSON of the sales forecast for store 27 on 16/09/2015, in the last line of the example "prediction" we see that the store will sell $5,729.48.

![](/img/deploy-heroku.png)

# Next Steps in this cycle
- Meeting with the business team to present the model, how to use it and its results;
- Collect usability and results feedback
- Increase model performance by 10%

# What to do in the next cycle?
- Test different Machine Learning models
- Build a pipeline to retrain model
- Get more data

Thanks for reading this far! If you have any questions or if you want to chat, just contact me at the links below:

- Phone: +55 11 94363-6853
- [Linkedin](https://www.linkedin.com/in/marcos-carvalhoo/)
- [Email](marcos.carvalho96@icloud.com)
