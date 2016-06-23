# time-series

Exploring Monthly Birth Data

The number of live births (in hundreds of thousands) per month in the U.S. was collected for the past 31 years (data/birth.txt) starting in January 1980 and ending December, 2010. We will be exploring this time series using various methods.

Calculate some aggregated statistics by month and year. What months have the highest birthrates? Any intuition as to why?

Use time series regression methods to fit a model. Using OLS in statsmodels, fit the overall trend with increasing polynomial terms.

Examine the summary information and compare each model to the model before it (i.e. simple linear with time compared to 2nd order polynomial with time + time^2)

Now that you have fit trend, add in the monthly component via dummy variables to capture seasonality. You could also try to create a 'seasons of the year' variable and fit the quarterly time series instead of the original
monthly time you plotted earlier.


![birth_data_with_avgs](/images/birthdata_with_averages.png?raw=true "Birth Data with AVGs")
