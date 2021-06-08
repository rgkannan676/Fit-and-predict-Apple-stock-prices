# 1.	Data Analysis

The provided Apple stock prices has multiple attributes, they are open, high, low, close and adjusted prices. For the analysis of the stock prices, the adjusted close stock values are selected as it gives a  better idea of the overall value of the stock.
Analysing the time plot of the data, we can see that the data is not stationary. The stock goes up and down but considering the whole data, we can see an upward trend factor. The variance of the model is also not constant. The stock data shows higher variability with higher stock value.
From the time plot , it can be observed that the stocks are volatile. The stock data has volatility clustering, i.e., the conditional variance of the stock data varies over time. 

![image](https://user-images.githubusercontent.com/29349268/121207155-a2ba1900-c8ab-11eb-82c1-5cd6cc7b9dc8.png)

Figure 1: Above is the data-time plot. We can observe an increasing trend.

The R code for reading the file is given below. From the csv file, adjusted close data is chosen for analysing the stock price.
```
#reading the data drom CSV file.
AppleStock <- read.table("AAPL.csv",header=TRUE, sep=',')
#Selecting the Adjucent Close column from table
AppleStock_AdjClose <- subset(AppleStock, select = c('Adj.Close'))
#Converting the values to a time series data
AppleStock_AdjClose <- as.ts(AppleStock_AdjClose)
#plotting the graph.
ts.plot(AppleStock_AdjClose)
```

# 2.	Make Data Stationary and Analyse Properties

From the data analysis It was observed that the data has a trend component and a changing variance. Taking the logarithm of the data can make the variance constant. The trend component can be removed by taking the difference of the data. It is multiplied by a value of 100 so that they can be interpreted as percentage changes in the stock.
The R code for this is:
```
AppleStock_AdjClose_stationary = diff(log(AppleStock_AdjClose)) * 100
ts.plot(AppleStock_AdjClose_stationary)
```
![image](https://user-images.githubusercontent.com/29349268/121207281-bf565100-c8ab-11eb-8581-4cafc53dcf25.png)
 
Figure 2: Time plot obtained after applying logarithm and difference.

From Figure 2, we can see that the data has become stationary with constant mean.  This can be confirmed by analysing the ACF and PACF of the obtained data. From Figure 3, we can observe that the ACF and PACF cut off and reduces rapidly concluding that the data is stationary. From the ACF and PACF plot, it can be understood that the data has very little serial correlation. 
But in the data analysis part, it was observed that the data has volatility clustering. So along with correlation, need to check whether the data is serially independent. For this, the absolute or squared value of the obtained stationary data is checked. 

![image](https://user-images.githubusercontent.com/29349268/121207354-cb421300-c8ab-11eb-9746-fc00245c29e7.png)

Figure 3: ACF and PACF plot of the data obtained after applying logarithm and difference on the stock data.

![image](https://user-images.githubusercontent.com/29349268/121207386-d1d08a80-c8ab-11eb-8fea-763358240d1d.png)

Figure 4: ACF and PACF plot of the Absolute value of the data.

![image](https://user-images.githubusercontent.com/29349268/121207418-d7c66b80-c8ab-11eb-9271-a05520b220f6.png)

Figure 5: ACF and PACF plot of the Square value of the data.

Analysing ACF and PACF of absolute and squared values in Figure 4 and 5, it can be observed that there is a significant autocorrelation. This is an evidence that the stock data is  not independently and identically distributed.
A Q-Q plot can be used to analyse how the data is distributed.  From Q-Q plot in Figure 6, it can be observed that the data deviates from the normal distribution and is a heavy tailed distribution. It can be confirmed by measuring the (excess) kurtosis by checking if the value is positive or negative. If positive, it is a heavy tailed distribution otherwise light tailed.
The R code for the Q-Q plot and finding kurtosis is given below.
```
dev.off()
qqnorm(AppleStock_AdjClose_stationary)
qqline(AppleStock_AdjClose_stationary)
kurtosis(AppleStock_AdjClose_stationary)

[1] 2.037644
```

The kurtosis is a positive value(2.037644) confirming that the distribution is a heavy tailed distribution.

![image](https://user-images.githubusercontent.com/29349268/121207533-f0cf1c80-c8ab-11eb-81e8-098e6b9b9e72.png)
 
Figure 6: Q-Q plot of the data.

# 3.	Find Best Model Fit.

From analysis of the transformed data, we observed that the data is stationary and  is not independently and identically distributed. So, in this section, I will try to fit both ARMA and GARCH models and choose the best model. 

## 3.1	ARMA 

The P and Q of the ARMA model can be found using the Extended Autocorrelation Function (EACF) table.
```
eacf(AppleStock_AdjClose_stationary)
```

![image](https://user-images.githubusercontent.com/29349268/121207751-1d833400-c8ac-11eb-9712-d807ce7c2c3c.png)

Table 1: EACF of the data.

From the EACF table , we can find the candidates for P and Q for the ARMA model. I have chosen 4 sets of  P and Q values from the potential candidates of P and Q. These selected candidates are (0,1), (1,1), (2,2), (3,3). 
Need to check whether the models are adequate to represent the data by checking the p values in the diagnostic log.

### 3.1.1	ARMA(0,1)
```
ARMA_0_1 = arima (AppleStock_AdjClose_stationary, order = c(0,0,1))
tsdiag(ARMA_0_1)
ARMA_0_1
```
![image](https://user-images.githubusercontent.com/29349268/121207857-2f64d700-c8ac-11eb-99e4-b0a13344acec.png)
 
Figure 7: Diagnostic plot of ARMA(0,1).

From Figure 7,  we can observe the p value is higher that critical value 0.05. Since p values are high, this model can be considered as a good model that fits the data.
From the model details, AIC = 1115.42 for ARMA(0,1).

### 3.1.2	ARMA(1,1)
```
ARMA_1_1 = arima (AppleStock_AdjClose_stationary, order = c(1,0,1))
tsdiag(ARMA_1_1)
ARMA_1_1
```

![image](https://user-images.githubusercontent.com/29349268/121207922-3f7cb680-c8ac-11eb-91bb-e86808464f95.png)

Figure 8: Diagnostic plot of ARMA(1,1).

From Figure 8,  we can observe the p value is higher that critical value 0.05. Since p values are high, this model can be considered as a good model that fits the data.
From the model details, AIC = 1116.79 for ARMA(1,1).

### 3.1.3	ARMA(2,2)
```
ARMA_2_2 = arima (AppleStock_AdjClose_stationary, order = c(2,0,2))
tsdiag(ARMA_2_2)
ARMA_2_2
```
![image](https://user-images.githubusercontent.com/29349268/121208002-50c5c300-c8ac-11eb-9d32-17c5241cba0b.png)

Figure 9: Diagnostic plot of ARMA(2,2).

From Figure 9,  we can observe the p value is higher that critical value 0.05. Since p values are high, this model can be considered as a good model that fits the data.
From the model details, AIC = 1120.52 for ARMA(2,2).

### 3.1.4	ARMA(3,3)
```
ARMA_3_3 = arima (AppleStock_AdjClose_stationary, order = c(3,0,3))
tsdiag(ARMA_3_3)
ARMA_3_3
```
![image](https://user-images.githubusercontent.com/29349268/121208086-62a76600-c8ac-11eb-8db2-3b1392e32c84.png)

Figure 10: Diagnostic plot of ARMA(3,3).

From Figure 10, we can observe the p value is higher that critical value 0.05. Since p values are high, this model can be considered as a good model that fits the data.
From the model details, AIC = 1124.15 for ARMA(3,3).

### 3.1.5	Selecting Best ARMA Model 

All the four candidates are adequate to fit the data. The best model can be selected by checking the model with the least AIC value. 

![image](https://user-images.githubusercontent.com/29349268/121208281-879bd900-c8ac-11eb-8757-0ab745b2981b.png)

From the table, it can be found that the ARMA(0,1) has the lowest AIC value, hence taken as the best model. 
	
## 3.2	GARCH 

From the data analysis, it was found that the data has volatility clusters. GARH model can be used to fit this type of data.

To Find the P and Q of GARH, compute the EACF of the absolute and squared values of the data.

```
eacf(abs(AppleStock_AdjClose_stationary))
```

![image](https://user-images.githubusercontent.com/29349268/121208498-aef2a600-c8ac-11eb-931d-5338fe499622.png)


Table 2: EACF of the absolute value of the data.

```
eacf(AppleStock_AdjClose_stationary ^ 2)
```

![image](https://user-images.githubusercontent.com/29349268/121208606-c5006680-c8ac-11eb-8b52-d39def49aeb3.png)

Table 3: EACF of the square value of the data.

From Table 2,  the EACF of the absolute data, the selected candidates for P and Q are (0,1), (1,1), (2,2), (3,3).

From Table 3,  the EACF of the square of data, the selected candidates for P and Q are (2,0), (2,1), (2,2)

Need to check whether the models are adequate to represent the data by checking the p values in the diagnostic log.

### 3.2.1	GARCH(0,1)
```
GARCH_0_1 = garch(x=AppleStock_AdjClose_stationary,order=c(0,1))
summary(GARCH_0_1)
AIC(GARCH_0_1)
```
The p value is 0.964 > 0.05, therefore can be considered as a good model. The obtained AIC is  1114.681.

### 3.2.2	GARCH(1,1) 
```
GARCH_1_1 = garch(x=AppleStock_AdjClose_stationary,order=c(1,1))
summary(GARCH_1_1)
AIC(GARCH_1_1)
```
The p value is 0.5022> 0.05, therefore can be considered as a good model. The obtained AIC is  1110.219.

### 3.2.3	GARCH(2,0) 
```
GARCH_2_0 = garch(x=AppleStock_AdjClose_stationary,order=c(2,0))
summary(GARCH_2_0)
AIC(GARCH_2_0)
```
The p value is 0.7699> 0.05, therefore can be considered as a good model. The obtained AIC is  1112.554.

### 3.2.4	GARCH(2,1)
```
GARCH_2_1 = garch(x=AppleStock_AdjClose_stationary,order=c(2,1))
summary(GARCH_2_1)
AIC(GARCH_2_1)
```
The p value is 0.7351> 0.05, therefore can be considered as a good model. The obtained AIC is  1113.629.

### 3.2.5	GARCH(2,2) 
```
GARCH_2_2 = garch(x=AppleStock_AdjClose_stationary,order=c(2,2))
summary(GARCH_2_2)
AIC(GARCH_2_2)
```
The p value is 0.9323> 0.05, therefore can be considered as a good model. The obtained AIC is  1108.542.

###  3.2.6	GARCH(3,3) 
```
GARCH_3_3 = garch(x=AppleStock_AdjClose_stationary,order=c(3,3))
summary(GARCH_3_3)
AIC(GARCH_3_3)
```
The p value is 0.9198> 0.05, therefore can be considered as a good model. The obtained AIC is   1108.099.

### 3.2.7	Selecting Best GARCH Model

![image](https://user-images.githubusercontent.com/29349268/121208973-0bee5c00-c8ad-11eb-94b0-0e67a87cd362.png)

From the table, it can be found that the GARCH(3,3) has the lowest AIC value, hence taken as the best model. 

### 3.2.8	Model Diagnostic of GARCH(3,3)
In this section, we will check whether the suggested model, GARCH(3,3) is supported by data. The R code is provided below.
```
par(mfrow=c(4,1))
plot(residuals(GARCH_3_3),type='h', main='GARCH(3,3)',ylab='Standardized Residuals')
qqnorm(residuals(GARCH_3_3)); qqline(residuals(GARCH_3_3))
acf(residuals(GARCH_3_3)^2,na.action=na.omit)
gBox(GARCH_3_3,method='squared')
```
![image](https://user-images.githubusercontent.com/29349268/121209046-1ad50e80-c8ad-11eb-95ee-90e71207cefa.png)
 
Figure 11: Model diagnostic plot of GARCH(3,3).

In the residuals plot, we can observe that there are no large values present. The Q-Q plot has largely straight-line pattern suggesting majority are in normal distribution. The ACF plot of squared residuals has all the values between the boundary, showing that there is no correlation. The p-values are also higher suggesting that the squared residuals are uncorrelated over time, and hence the standardized residuals may be independent.

# 4.	Model Forecasting

## 4.1	Creating the Forecast Model with ARMA and GARCH

The best ARMA model selected is ARMA(0,1) and the best GARCH model selected is GARCH(3,3). We can combine these 2 models with the help of ‘ugarchspec’ function where we can mention both the GARCH and ARMA models to create a new model. R code for specifying the model details and fitting it with the data is given below.
```
ForecastModel_Spec=ugarchspec(variance.model=list(garchOrder=c(3,3)),mean.model=list(armaOrder=c(0,1)))
ForecastModel_Fit=ugarchfit(spec=ForecastModel_Spec,data=AppleStock_AdjClose_stationary)
ForecastModel_Fit
plot(ForecastModel_Fit , which = 'all')
```
![image](https://user-images.githubusercontent.com/29349268/121209164-32ac9280-c8ad-11eb-9436-117af9c3cc98.png)

Figure 12: Forecast model Details. 

The p-value of the model > 0.05 for both Ljung-Box Test on Standardized Residuals and Squared Residuals shows that the model is a good fit for the data. 

## 4.2	Forecast Data
The forecast of the next 10 values is done using ‘ugarchforecast’. The models created in the previous section is fitted and the points are generated.
```
Forecast_Data=ugarchforecast(ForecastModel_Fit, n.ahead=10)
Forecast_Data_values = fitted(Forecast_Data)
Forecast_Data_values
```
![image](https://user-images.githubusercontent.com/29349268/121209779-b8c8d900-c8ad-11eb-8381-0f5dd16bb711.png)

 This is the prediction for the stationary data. This need to be converted back to get the original values. The steps used to convert the data in to a stationary one is given below.

AppleStock_AdjClose_stationary = diff(log(AppleStock_AdjClose)) * 100

Converting back the transformations, we get the below equation. Cumulative sum is used to invert the difference action.

AppleStock_AdjClose_predicted = exp(cumsum(Forecast_Data_values/100)) * AppleStock_AdjClose[length(AppleStock_AdjClose)]

This is used to convert the data back to original scale. Below is the R code.
```
AppleStock_AdjClose_predicted =  exp(cumsum(Forecast_Data_values/100)) * AppleStock_AdjClose[length(AppleStock_AdjClose)] 

AppleStock_AdjClose_predicted
```
134.3521 134.7905 135.2304 135.6717 136.1145 136.5587 137.0044 137.4515 137.9000 138.3501 are the transformed values of the prediction. 

Using this, we can plot the forecast graph. Below the R code to plot the forecast.
```
#Plot data
plot(1:253, AppleStock_AdjClose, xlim = c(0, 300), ylim=c(50, 150), ylab="Apple_Stock" , xlab="Time" ,main="Stock Forecast")
lines(1:253, AppleStock_AdjClose, type="l" )
#Plot the the Forecasted values
lines(254:263, AppleStock_AdjClose_predicted, type="o", col="green")
#Add legend
legend( "topleft" , lty =1, pch=1, col=1:3 ,c ("Data", "Forecast" ))
```
![image](https://user-images.githubusercontent.com/29349268/121209551-8028ff80-c8ad-11eb-84f8-e7f60258ba17.png)

Figure 13: Shows the predicted values and data in the original scale.
