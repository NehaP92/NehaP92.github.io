---
layout: post
title:      "Real Estate Analysis using OLS Regression"
date:       2020-08-04 07:03:10 -0400
permalink:  zipcodes_and_data_analysis
---

When it comes to investing in real estate, the decision might sometime seem pretty combursome due to the range of factors affecting the vairation in prices. It is helpful to start with a general analysis based on the location, and then split the price range into groups for a proper regression fit. The split makes our model more effective since it removes the high and low-end houses as outliers from the basis average prices, while the location provides us with the information on where the houses from each group are most likely located.

The data used for this analysis is extracted from the king county house data. The features were first identified and categorized into numerical and categorical data. The numerical features are scaled using scikit learn's RobustScalar method, while the categorical data is one hot encoded to make the format usable with the statsmodel OLS regression function. The results below show the performance of the OLS regression fit on the entire data set before spiltting the price ranges into the normal and highend houses.

![](https://raw.githubusercontent.com/NehaP92/dsc-mod-2-project-online-pt-041320/master/model_results_base.png)

The model yeilded an R squared of 0.703. Further analysis on the validity of this model, using the model_analysis() function shows that the data is not normal and there seems to be large number of outliers on the higher end.

```
def model_analysis(model,df):

    """
		Takes the model and data frame and returns the model coeffecients, 
    evaluates p-values based on alpha 0.05, checks for normality through Q-Q plot,
    and analyses the homoscedasticity of our data
    input:
    model (OLS model)
    df (DataFrame): the DataFrame on wchich the analysis is done
    """
		
    residual = model.resid
    price = df['price']
    sns.scatterplot(price, residual);
    plt.axhline(0)
    display(model.params.to_frame().style.background_gradient())
    display(model.pvalues<0.5)
    display(sm.graphics.qqplot(model.resid, stats.norm, line='45', fit = True));
```
	
![](https://raw.githubusercontent.com/NehaP92/dsc-mod-2-project-online-pt-041320/master/base_QQ.png)

The homoscedasticity test also shows large hetroscedasticities, especially on the higher side.
	
![](https://raw.githubusercontent.com/NehaP92/dsc-mod-2-project-online-pt-041320/master/base_hom.png)
	
The first instinct to improve our model performance is to remove the outliers, but we will use these outliers to build a second model fitting these, for they would represent the few high end houses in King County.
	
	
### Splitting into 2 groups

The outliers are removed based on quantiles using the IQR_remove_outlier() function.

```
def IQR_remove_outlier(df, col):

	"""Removes outlier based on IQR"""

	Q1 = df[col].quantile(0.25)
	Q3 = df[col].quantile(0.75)
	IQR = Q3-Q1
	group = df[~((df[col]<(Q1-1.5*IQR)) | (df[col]>(Q3+1.5*IQR)))]
	return group
```

The model performance on group 1 increased to an R squared of 0.743

![](https://raw.githubusercontent.com/NehaP92/dsc-mod-2-project-online-pt-041320/master/group1_model.png)

We also see an improvement in the Q-Q plot and the improved homoscedasticity result:

![](https://raw.githubusercontent.com/NehaP92/dsc-mod-2-project-online-pt-041320/master/group1_analysis.png)

OLS regression on group 2 yeilded R squared of 0.746.

![](https://raw.githubusercontent.com/NehaP92/dsc-mod-2-project-online-pt-041320/master/group2_model2.png)

The top features affecting the house prices in normal range were found to be waterfront, view, area in sqft, condition, and location while for the high end houses were location, renovation, and condition.

You may try splitting the modle further into more groups for better analysis and results.
