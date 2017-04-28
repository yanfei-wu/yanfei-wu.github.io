+++
draft = false
image = "blog/employee_file/employees.jpg"
title = "Machine Learning for Employee Retention"
type = "post"
author = "Yanfei Wu"
date = "2017-04-28"
description = "This post describes a machine learning approach to implementing an effective employee retention program."
tags = [
"Python",
"machine learning",
"classification"
]
+++

*Note*: The code and full report for this project is available in [my Github Repository](https://github.com/yanfei-wu/ml_udacity/tree/master/capstone).

### 1. Introduction      
 
Recruiting excellent employees is one thing, but keeping them is another. While losing employees who are poor performers can have positive effects, high employee turnover rate is generally regarded as bad for the business. It increases expenses since the process of identifying, hiring and training employees is very expensive. Studies have shown that cost related to directly replacing an employee can be as high as 50–60% of the employee’s annual salary, and the total cost of turnover can reach as high as 90–200% of the employee’s annual salary.[1] Even worse, frequent employee turnover can destroy the company morale, resulting in decreased performance in the workplace. Therefore, retaining its valuable and talented employees is vital to a company's success.  

A novel approach to implementing an effective retention program and preventing key workers from leaving prematurely is to use machine learning techniques. For example, a supervised classification model can be trained on a dataset containing features related to the employees and whether they left the company (a dataset that is available to many companies). Building such a model provides insights to key factors that result in employee turnover. It also allows the management and the human resource team to predict which employee is going to leave so that they could intervene immediately.  

The goal of this project is to build such a supervised binary classification model. A simulated dataset (available in [Kaggle](https://www.kaggle.com/ludobenistant/hr-analytics)) containing a series of employee-related features and a binary class label of whether the employee left or not is used. The expected outcome of this project is to help the management and the human resource team:  
  
1. predict which current employee is going to leave (class label) so that they could intervene immediately;
2. identify which are the most important factors (features) that lead to employee turnover so that changes can be implemented to ensure employees remain in place while maintaining high work performance and productivity.    


### 2. Datasets and Exploratory Visualization 

The dataset contains 14,999 rows and 10 columns. There are both numerical and categorical features in this dataset, including:   

* `'satisfaction_level'`: employee satisfaction level (numerical, float numbers between 0 and 1)
* `'last_evaluation'`: the score from the employee's last evaluation (numerical, float numbers between 0 and 1)
* `'number_project'`: the number of projects the employee worked on (numerical, integers)
* `'average_monthly_hours'`: the average monthly hours the employee spent on work (numerical, integers)
* `'time_spend_company'`: number of year the employee spent at the company (numerical, integers)
* `'work_accident'`: whether the employee had a work accident (categorical, '0' - no, '1' - yes)
* `'promotion_last_5year'`: whether the employee had a promotion in the last 5 years (categorical, '0' - no, '1' - yes)
* `'sales'`: the Department the employee works (categorical, 'sales', 'accounting', 'hr', 'technical', 'support', 'management', 'IT', 'product_mng', 'marketing', 'RandD')
* `'salary'`: the salary of the employee (categorical, 'low', 'medium', 'high')   

The target variable in the dataset is:    
  
* `'left'`: whether the employee has left or not (categorical, '0' - no, '1' - yes)

*Note that the number of employees who left in the dataset is 3571 and is 23.81% of the total employees (14,999).* As discussed above, with the unbalanced class distributions, care must be taken when choosing performance measure for the models. F2 score will be used in order to weight recall higher than precision.  

Also note is that there are no `NaN` values and all the numerical values are within reasonable range. No outliers are detected. 

Both the numerical and the categorical features are visualized below.  
<img src="../employee_file/numerical.png" class="img-responsive" style="display: block; margin: auto;" />  

*Some observations from the numerical features are*:  
  
* `time_spend_company` and `number_project` take integer values. This dataset only considers employees at the company for 2 or more years with 2 or more projects.
* `'average_monthly_hours'`, `'last_evaluation'`, and `'satisfaction_level'` take continuous values. All of them show bimodal distributions.  
 
<img src="../employee_file/categorical.png" class="img-responsive" style="display: block; margin: auto;" />

*Some Observations from the categorical features are*:   
  
* `'work_accident'` and `'promotion_last_5years'` are two-level categorical variables. Both show very unbalanced distribution (with the majority being category 0: no work accident or no promotion in last 5 years)
* `'salary'` has 3 levels: low, medium and high. The salary structure seems to reflect the reality, with the majority of employees having low and medium salaries.
* Employees from 10 different departments are included in the dataset, with the majority of employees from sales, technical and support. This also somewhat reflects the reality in many companies.

A correlation matrix of the dataset (excluding `'department'`) is shown below.  
<img src="../employee_file/correlationmatrix.png" class="img-responsive" style="display: block; margin: auto;" />  

*The correlation matrix shows that:*     
  
* There is a strong negative correlation between `'satisfaction_level'` and `'left'` with a correlation coefficient of -0.39.
* Variables `'number_project'`, `'average_monthly_hours'`, and `'last_evaluation'` show strong positive correlations, which makes sense intuitively.   


### 4. Models

#### 4.1 Algorithms and Techniques 

It is a binary classification problem and there are a number of algorithms available in the `scikit-learn` library. To find the best model for the problem, 6 different algirithms are used to train the dataset, including Logistic Regression, Linear Discriminant Analysis, K-Nearest Neighbors, Decision Trees, Support Vector Machines, and Random Forest. All the algorithms are evaluated out-of-box, i.e., default hyperparameters will be used. A naive benchmark model is also built to set a baseline for model performance. The benchmark model is to predict that all the employees are leaving. In other words, the management and HR team treat every single employee as if he/she is leaving. Note that implementing such a naive classifier will probably lower the employee turnover rate but most of the time, it is not a realistic retention program for most companies. So, it is only for comparison purpose here. 

Typically, the performance of a classification model can be measured by accuracy, i.e., the percentage of correct predictions. But in this case, it is not the best metric due to the class imbalance (~25% class 'left' vs ~75% class 'stay'). Instead, we can use a combination of precision and recall as an unambiguous way to show the prediction results. In this specific problem, we would like to avoid predicting valuable employees who are actually leaving as staying and thus taking no actions. Therefore, the so-called F2 score is used. It is a weighted average of precision and recall and it places more emphasis on recall (minimizing False Negatives). 

Note that before the dataset can be used to train the classificatin models, some preprocessing steps are done, including one-hot encoding of the categorical features and normalization of the numerical features. The latter ensures that each feature is treated equally when applying supervised learners.    

#### 4.2 Model Selection and Feature Importance 

In order to evaluate each algorithm, a 10-fold cross validation procedure is carried out. The cross validation F2 scores for the 6 models are shown in the boxplot below. The F2 score of the benchmark model is also plotted as a baseline.     

<img src="../employee_file/modelcomparison.png" class="img-responsive" style="display: block; margin: auto;" />  

Based on the comparison, Decision Trees classifier and Random Forest both perform very well. They give much better F2 score than the benchmark classifier. Generally, Random Forest algorithm is less likely to overfit the data compared to Decision Trees. So in this case, it is chosen as the best model. 

The Random Forest model is further tuned with `GridSearchCV`. The best 10-fold cross validation F2 score is 0.9754, which shows slight improvevment than the score from the out-of-box Ramdom Forest model (0.9660). 

With a classification model like the Random Forest classifier built above, management and human resource team are now able to predict if a current employee is leaving so that they can take immediate internvention. Another key information for an effective employee retention program is the knowledge of the most important factors on employee turnover. Luckily, with Random Forest classifier, we can easily obtain the feature importance from a trained model. A barplot of the top 5 features from the optimized Random Forest model trained on the entire training set is shown below.   

Thses 5 features consists about **95%** of the total feature importance. `'satisfaction_level'` is determined to be the most important feature, which is consistant with its high correlation coefficient with the target variable `'left'` as shown in the correlation matrix. The other features highly relevant to employee turnover are `'average_monthly_hours'`, `'number_project'`, `'time_spend_company'`, and `'last_evaluation'`.   

<img src="../employee_file/featureimportance.png" class="img-responsive" style="display: block; margin: auto;" />   

### 5. Conclusion

To summarize, this project outlines a machine learning approach to help implement an effective employee retention program. A synthesized dataset containing both numerical and categorical features of the employees, as well as a binary label of whether the employee left or not is used to build a binary classification model. The categorical features are transformed to numerical values by one-hot encoding. The numerical features are rescaled to the same range (0 to 1) so that the learners can treat each feature equally. Different classification algorithms are compared. The performance of the models are measured by the F2 score in order to put more weights on recall. K fold cross validation procedure is implemented to evaluate the models. Random Forest classifier is chosen as it gives the best cross validation F2 score. The model is further tuned using `GridSearchCV`. The final model shows excellent F2 score (much better than the naive benchmark classifier) and appears to be generalizable to unseen data. With this classification model, we can predict whether a current employee will leave given the features of the employee. Feature importance analysis reveals the top 5 features, which consist about 95% of the total importance. These top features can help management and human resource team understand why their emplpyees leave.  

However, the prediction results seem too good to be true in reality. But data leakage should not be a problem here since feature normalization is performed on each fold of cross validation instead of on the entire dataset. So, one possible explanation for this exceptionally good result could be that the synthesized dataset is just too ideal. The real-world data is normally a lot messier. Also, it is reasonable to believe that employee turnover characteristic would be different for each individual company, or at least for each type of industry. Therefore, this specific model obtained here is likely not suitable for specific companies out there. Nevertheless, the general machine learning approach described here should be a good practice in reality.


-----------  

*[1] Cascio, W.F. 2006. Managing Human Resources: Productivity, Quality of Work Life, Profits (7th ed.). Burr Ridge, IL: Irwin/McGraw-Hill. Mitchell, T.R., Holtom, B.C., & Lee, T.W. 2001. How to keep your best employees* 