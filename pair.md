## Hitch Demand Prediction

**Installation**

If you want to run the seasonal ARIMA module in `statsmodels` on your own laptop, use the following code to install 
the latest version of `statsmodels 0.7`

```python
git clone https://github.com/statsmodels/statsmodels.git
cd statsmodels
python setup.py build_ext --inplace
sudo python setup.py install
```

<br>

For this exercise, we'll be returning to a dataset of Hitch client logins 
and attempting to predict future demand based on historical data.  

<br>

1. First, load the data (the dataset for this exercise is located in 
    `data/logins.json`). Pandas has a nice convenient function to read in 
    JSON called `read_json()`. We'd like the data to be a pandas series 
    (rather than a dataframe, which is the default). You can control this 
    with the `typ` parameter (look at the [read_json](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.io.json.read_json.html) docs for how to use it).

2. Lets turn the series into a pandas `TimeSeries`:

    The indices should be the timestamps. The values created should be a 
    series of ones so that we can count them. Your `TimeSeries` object 
    should look like this:

    ```
    2012-03-01 00:05:55    1
    2012-03-01 00:06:23    1
    2012-03-01 00:06:52    1
    ```

3. We would like to resample the data to a daily frequency (and to an 
    hourly frequency for use later).

    Now you can use [resample](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.resample.html) 
    to get a count of the number of datapoints for every hour. Look at the 
    documentation to figure out which parameters you need to set.

    Look at examples on the [timeseries](http://pandas.pydata.org/pandas-docs/stable/timeseries.html) 
    documentation of uses of the `resample` method.

    Your daily data should look like this:

    ```
    2012-03-01    268
    2012-03-02    314
    2012-03-03    521
    ```

    Your hourly data should look like this:

    ```
    2012-03-01 00:00:00    31
    2012-03-01 01:00:00    18
    2012-03-01 02:00:00    37
    ```

4. Before we dive into any statistical or machine learning methods for 
    predicting future data, let's take a look at the data we already have. 
    First just plot the daily data using `.plot()` and see if we can pick out 
    any immediate trends. 

5. Create a dataframe that has count and dayofweek features (refer to 
    morning sprint). It should look like this:

    ```
                  count   dayofweek
    2012-03-01      268         3
    2012-03-02      314         4
    2012-03-03      521         5
    ```   

6. You should see a strong seasonal component to the data. I'd be willing to 
    bet that the peaks are weekend demand. Let's see if that's true by 
    highlighting the weekends on our plot.

    You can do this highlighting with matplotlib's function [fill_between](http://matplotlib.org/api/axes_api.html?highlight=fill_between#matplotlib.axes.Axes.fill_between).

    You can then create a `weekend` Boolean variable by identifying your
    `dayofweek` variable with values 5 or 6.

    Now, use `fill_between` to fill in the data.

7. Add a best fit line to this graph. Do you observe any trend?

    For the `plot` function, you can use the `lw` parameter to make the lines 
    thinner, which might make the graph easier to visualize. You can also 
    change the color of the line with the `style` parameter.

8. Lets turn our attention now to Box-Jenkins Methods.  First thing is to 
   determine if the data is stationary. Plot the ACF/PACF with 28 lags 
   (Approx. 4 weeks -- See snippet of code below). Does the data appear to be stationary?
 
   What does ACF/PACF indicate? (See notes if you are unsure)

   ```python
   fig = plt.figure(figsize=(12,8))
   ax1 = fig.add_subplot(211)
   fig = sm.graphics.tsa.plot_acf(your_data, lags=28, ax=ax1)
   ax2 = fig.add_subplot(212)
   fig = sm.graphics.tsa.plot_pacf(your_data, lags=28, ax=ax2)
   ```

9. Hopefully you have found that the data is not stationary, i.e. not constant mean/variance
   over time. One of the ways to address the changing mean is to detrend the data, which means
   to use ordinary least square regression (`OLS`) on the data and treat the residuals as your new time
   series. 

   Detrend the data. Plot the detrended data and its ACF/PACF.

10. Another way to achieve stationarity in the data is to use the differencing function (1st difference 
    can remove linear trend; 2nd order difference can remove curvature, etc). 

    In some cases with seasonal data, the seasonal difference has to be taken to achieve stationarity, i.e.
    the difference between this January and the last January if the data is monthly.
    
    - Use the `pd.Series.diff(periods=1)` function to take the 1st difference 
      and plot the differenced data and its ACF/PACF
      
    - Use the `pd.Series.diff(periods=7)` function to take the 1st seasonal 
      difference and plot the differenced data and its ACF/PACF.

11. Out of the plots made in `9.` and `10.`, which appears to be more stationary. Explain your answer by 
    describing your observations with the ACF/PACF plots

12. Fit a series of seasonal ARIMA models of order `(p,d,q)x(P,D,Q)xL` to the 
    original day data. See below for an example to run seasonal ARIMA in `statsmodels`:

    For example, lets assume you've identified a seasonal difference and first difference is needed to achieve stationarity
    and you observe a significant seasonal lags at 7 and 14 in the ACF, and a significant seasonal lag
    at 7 in the PACF in addition to several significant nonseasonal lags in both ACF/PACF.  In general,
    you should try to assess the seasonal components and you should pick the component that cuts off early.
    So, first, you could try a `(0,1,0)x(1,1,0)x7` SARIMA model.
    
    ```python
    model=statsmodels.tsa.SARIMAX(your_ts, order=(0,1,0), seasonal_order=(1,1,0,7)).fit()
    ```
    
    You would then examine the residuals of the model to see if additional seasonal terms need to be fit 
    or nonseasonal terms.  For instance, if no additional seasonal terms are significant, but there are
    significant lags at 1, 2, and 3 of the PACF, we would update the previous model to  `(3,1,0)x(1,1,0)x7`
    SARIMA model.

    ```python
    model=statsmodels.tsa.SARIMAX(your_ts, order=(3,1,0), seasonal_order=(1,1,0,7)).fit()
    ```
    
    And you would iterate in this manner until the ACF/PACF looks like white noise. If you have identified
    multipl SARIMA models, you can compare them using AIC choosing the model that minimizes it.
    
10. Perform the same steps 4-12 on the hourly data (use periods=24 for seasonal
    difference and 72 lags for ACF/PACF plots).


#Extra Credit

1. Fit an ARIMA model to the birth data. Use the first 30 years of data to 
    predict the 31st year. Be sure to:
    - Transform the data into a stationary time series.
        - Explore detrending as well as differencing
        - Examine variance stabilizing transformation if needed
    
    - Using ACF/PACF visual methods, iterate through potential models to 
      identify at least 2
    - Select a final model and plot your fitted model with predictions 
      overlayed on the raw data

