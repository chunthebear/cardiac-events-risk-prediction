# Risk Prediction of Cardiac Events Using Measurements from Stress Echocardiography Tests

## Introduction
This project provides me with the opportunity to practise and apply the analytical skills with R using existing templates (Balshaw, 2021) to predict the risk of having cardiac events using a series of predictors, which are the measurements taken during stress echocardiography tests. The data provided is to be processed, interpreted, and analyzed. The data is then used to build several logistic regression models which are then validated. Finally, the results are presented, interpreted, and discussed. The ultimate objective of this project is to find the predictors that best predict the risk of cardiac events. The R codes are in the attached R notebook.

### Dataset Description

The raw dataset contains 558 observations and 31 data attributes. These attributes are outcome and predictor variables which will be further explained in the section on dataset preparation. The brief meanings of these variables could be found in R notebook. Essentially, the data is collected from a randomized control trial done on 220 men and 338 women to determine if the drug dobutamine could inform the risk of having a cardiac event by measuring patient vitals and echocardiography (ECG). Certain patient histories are also part of the data to help predict the risk. 

## Methods

The given dataset is loaded, assessed, further processed, and interpreted. Several logistic regression models are built with stepwise regression with forward, backward, and both directions. A preliminary evaluation is done on the models by checking the predictors used by different models to check for differences, using the Receiver Operating Characteristic (ROC) curves and the Area under the ROC Curve (AUC) scores to check for performance, and using a training and test sub-dataset to check for overfitting. Then, cross-validation is used to further validate the models to mitigate potential overfitting and to choose the most reliable one, and the predictors used in this model would be the best predictors. 

### Process Diagram

![process_dia drawio](https://user-images.githubusercontent.com/16961563/151689703-582faa99-35fd-4b37-b27f-860edc75ee05.png)

_Figure 1. Process diagram._

### Dataset Preparation

We first should understand what is being predicted (outcome variable), and what are used for the prediction (predictor variables). In this case, a cardiac event is the outcome. We use the variable any.event in the dataset as the outcome variable since it is the composite outcome of any cardiac event, including death, myocardial infarction (MI), angioplasty (PTCA), and bypass surgery (CABG). Therefore, we remove the individual outcome variables newMI, newPTCA, newCABG, and death from our dataset since they would not be used. The rest of the variables are all predictor variables, and they are either categorical or numeric. The categorical predictors are gender, chestpain, restwma, posSE, hxofHT, hxofDM, hxofCig, hxofMI, hxofPTCA, hxofCABG, and ecg which are coverted to factor variables in R for the convenience of analysis. The rest of predictors are numeric, including bhr, basebp, basedp, pkhr, sbp, dp, dose, maxhr, pctMphr, mbp, dpmaxdo, dobdose, age, baseEF, and dobEF. 

### Data Quality Assessment

The R library VIM is used to identify any missing values which do not exist in the dataset. The library skimr is used to generate an overall data quality report (Business Science, 2021) which shows that all variables are complete. It also shows the number of unique values for each categorical variable, which agrees with the data dictionary, meaning they are valid. The validity of numeric variables could be verified if we have documentation on the range of numeric values. The function duplicated() is used to check for any duplicates which also do not exist. A manual check of the 558 observations in the dataset is also performed and arrives at the same conclusion. However, some values are considered to be abnormal during the manual check; for example, data case 413 has a basal heart rate of 210 bpm which is really high; data case 1 has a baseline cardiac ejection fraction of 27% which is really low (even for an 85-year-old patient). Therefore, it is suspected that outliers exist, and an outlier analysis using boxplots is performed to identify them. Instead of removing the data cases containing the outliers, the outliers are replaced with the mean value of the respective variable to keep the same distributions, due to the relatively small size of the dataset. 

### Descriptive Analysis

The distributions of the numeric predictors are plotted; most of them are normal distributions, with the exception of dobdose and dose. This is reasonable because the doses for the patients might be the same for most patients, creating skewed distributions. A log10 transformation of the x-axis is performed, making some distributions less skewed, suggesting the log10 form of numeric predictors should be used (although the difference is small). 
  
The relationships between predictors and the outcome are also plotted to see which predictors have a significant influence on the outcome. The numeric predictors baseef, dobdose, dobef, dp, and dpmaxdo have the steepest slope in their curves, and the lower their values are, the higher the risk of having a cardiac event. As for the categorical predictors, having a history of CABG (hxofCABG), smoking (CIG), diabetes (DM), hypertension (HT), MI, and PTCA lead to higher risk; having chestpain, ecg showing MI, and having the gender of a male also have a higher risk. However, one should note that there could be co-factors, so just judging from such plots to identify the most influencing predictors is not sufficient. 

### Model Building

Logistic regression models were constructed with stepwise regression in the forward, backward, and both directions to select the best 4 models automatedly by checking the Akaike information criterion (AIC) score (the lower, the better fit the model is (Zach, 2021)), and removing the least statically significant variables (usually removed when the p-value is more than 0.15 (Balshaw, 2021)). The forward direction starts building the model with 0 predictors and adds a predictor at each step; the backward direction starts with all the predictors and removes them; both directions could start with a small or big model. The stepwise process stops until the AIC score could not be reduced by at least 2 (Balshaw, 2021), or no more statistically significant variables could be added or removed. In total, there are 4 strategies resulting in 4 final models.
  
### Preliminary Comparisons

The two models built in the forward direction and in both directions starting with a small model are the same model, having 9 predictors restwma, posSE, hxofHT, hxofMI, ecg, pkhr, hxofDM, dobdose, and maxhr. We call this model the “forward model”, and its AIC is 424.14. The other two models built in the backward direction and in both directions starting with a big model are also the same model, having 10 predictors basebp, basedp, pkhr, pctmphr, restwma, posSE, hxofHT, hxofMI, hxofDM and ecg. We call this model the “backward model”, and its AIC is 425.48. The forward model performs slightly better judging from the slightly lower AIC value. 

![f2](https://user-images.githubusercontent.com/16961563/151690435-6cabd234-0556-4ccc-8989-bb5cbe540ac3.png)

_Figure 2. ROC and AUC for the forward and backward models._	

The ROC is plotted for both of them, and this time the backward model is slightly better having a slightly higher AUC of 0.799, as compared to that of 0.798 of the forward model, as seen in Figure 2. 
	Since the AUC scores are not suspiciously high, it is unclear if there exists overfitting. To check for overfitting, we divide the dataset into training and test sub-sets with a 4:1 ratio and only use the training set to fit the models, and use the test set to evaluate the models. The forward model has an AUC of 0.81 from training and 0.77 from testing; the backward model has an AUC of 0.81 from training and 0.76 from testing, as seen in Table 1. Since the training and testing AUC are not far from each other for both the models, the extent of overfitting is not large. The forward model is slightly better having a better testing AUC and the backward model might be slightly more overfit, but the two models are very close to each other in terms of their performance, and it is unclear which model is better and further validation is needed. 

 | Models | Training AUC | Testing AUC |
 | ----------- | ----------- | ----------- | 
 | Forward model | 0.81 | 0.77 |
 | Backward model | 0.81 | 0.76 |

_Table 1. Training and testing AUC for the forward and backward models._

### Cross-validation

k-fold cross-validation is conducted to mitigate overfitting (Balshaw, 2021). This means that we have k overall iterations of training and testing together, and k divisions of equal size of the complete datasets. (k-1) of the divisions are used for training, and the remaining 1 is for testing. For every iteration or fold, we shuffle the whole dataset, divide it, train, and test.

Firstly, 5-fold cross-validation is conducted on the two models. The forward model gives a mean training AUC of 0.798, and testing AUC of 0.744; the backward model gives a mean training AUC of 0.801 and testing AUC of 0.734 (seen in Table 2). It is clear the forward model is better, having a smaller difference between the training and testing AUC, and a higher testing AUC as well, likely less overfit. Then, 10-fold cross-validation is also conducted on the two models. The forward model gives a mean training AUC of 0.796, and testing AUC of 0.743; the backward model gives a mean training AUC of 0.798 and testing AUC of 0.723 (seen in Table 3). Again, the forward model gives better scores and is likely less overfit due to the smaller difference between training and test scores, as highlighted in green in Table 2 and 3. 

 | Models | Mean Training AUC | Mean Testing AUC | Difference | 
 | - | - | - | - |
 | Forward model | 0.798 | 0.744 | 0.054 | 
 | Backward model | 0.801 | 0.734 | 0.067 | 

_Table 2. Mean training and testing AUC, and their difference from 5-fold cross-validation._

 | Models | Mean Training AUC | Mean Testing AUC | Difference | 
 | - | - | - | - |
 | Forward model | 0.796 | 0.743 | 0.053 | 
 | Backward model | 0.798 | 0.723 | 0.075 | 

_Table 3. Mean training and testing AUC, and their difference from 10-fold cross-validation._

## Final Results

The forward model having predictors restwma, posSE, hxofHT, hxofMI, ecg, pkhr, hxofDM, dobdose, and maxhr is the better performing model after validation. In Table 4, these predictors are ranked based on their statistical significance using p-value. Depending on the confidence interval, some predictors would be statistically insignificant (Zach, 2021). For example, if the confidence interval is 95%, then the predictors hxofdm and those below it would not be statistically significant since (1 - p-value) is less than 95%. All predictors can be kept if the confidence interval is 90%.

 | Predictors | P-values | 
 | - | - |
 | posSE (value 1) | 0.000384 | 
 | restwma (value 1) | 0.001759 | 
 | hxofmi (value 1) | 0.013740 | 
 | pkhr | 0.018015 | 
 | ecg (value “MI”) | 0.042953 | 
 | hxofdm (value 1) | 0.060530 | 
 | hxofht (value 1) | 0.060790 | 
 | dobdose | 0.066424 | 
 | maxhr | 0.090682 | 

_Table 4. Ranked predictors and their p-values used in the final model._

The final model also has null deviance of 489.74 on 557 degrees of freedom, and residual deviance of 402.14 on 547 degrees of freedom. 

## Interpretation and Discussion 

As compared to the previous descriptive analysis, the results have been narrowed down to only 9 final predictors, disregarding those which are statistically insignificant. Moreover, the predictor restwma is not identified in the descriptive analysis. As compared to the original study (Krivokapich et al., 1999), the results largely overlap. History of MI (hxofMI), hypertension (hxofHT), or diabetes mellitus (hxofDM), presence of rest WMAs (restwma), ECG response (ecg), positive SE (posSE) are all common significant independent predictors. The original study also identifies the dobutamine EF (dobEF) to be a predictor, but is not identified by the logistic model constructed in this assignment. This assignment also identifies the peak heart rate (pkhr), maximum heart rate (maxhr), and dobutamine dose (dobdose) to be significant predictors, which are not found by the original study. However, it is important to note that the original study also constructed classification and regression trees (CART) for their analysis. 
  
Both 5-fold and 10-fold cross-validation are conducted. The test AUC scores are slightly higher from the 5-fold cross-validation. Since the dataset is relatively small, using 5 folds is potentially better because 10% of the dataset for 10 folds for testing might be too small and not able to capture the variation of the distribution of the dataset. Using 5-fold cross-validation gives us an adjusted AUC score of 0.798 - 0.054 = 0.744. For even more reliable validation, more numbers of folds could be used and the averages can be taken to reduce the negative impact of random fold splits (Brownlee, 2018). 
  
From the null deviance and residual deviance, we could calculate the Chi-Square statistic as the difference between null deviance and residual deviance, and use it and the difference in the degrees of freedom to calculate the p-value associated with it (Zach, 2021). From the calculation, we achieve a p-value that is close to 0, meaning that the final model is highly useful in predicting the risk of a cardiac event using its predictors.  

## Conclusion

This assignment reflects the learnings from the classes and successfully develops, validates, and interprets a final logistic regression model to predict cardiac events using measured predictors from stress echocardiography tests using R. The most significant predictors include restwma, posSE, hxofHT, hxofMI, ecg, pkhr, and hxofDM. These results also largely overlap the findings from the original studies with some differences. As part of the recommendation, more numbers of folds could be tried for cross-validation to take the average scores from repeated validations. Other types of models could be built and validated for comparison including decision trees and artificial neural networks to potentially achieve better results. 


## References

Balshaw, R. (2021). Predictive Modeling Slides.

Brownlee, J. (2018). Does it matter? R-Packages. https://cran.r-project.org/web/packages/cvms/vignettes/picking_the_number_of_folds_for_cross-validation.html

Business Science. (2021, March 9). Assess Your DATA QUALITY in R | R-bloggers. R-Bloggers. https://www.r-bloggers.com/2021/03/assess-your-data-quality-in-r/

Krivokapich, J., Child, J. S., Walter, D. O., & Garfinkel, A. (1999). Prognostic value of dobutamine stress echocardiography in predicting cardiac events in patients with known or suspected coronary artery disease. Journal of the American College of Cardiology, 33(3), 708–716. https://doi.org/10.1016/s0735-1097(98)00632-9

Zach. (2021, November 15). How to Interpret glm Output in R (With Example). Statology. https://www.statology.org/interpret-glm-output-in-r/
