# Optimizing an ML Pipeline in Azure

## Overview
This project is part of the Udacity Azure ML Nanodegree.
In this project, we build and optimize an Azure ML pipeline using the Python SDK and a provided Scikit-learn model.
This model is then compared to an Azure AutoML run.

## Summary

The aim of this project is to apply two methods to determine the model with the best accuracy to predict whether a customer will or won't subscribe to a given bank service (classification).  
- The first method used is to fix the model, in our case a logistic regression, and let Azure HyperDrive determine the best parameters.
- The second method is to apply Azure AutoML to the same dataset and let it determine the model and parameters that give the best accuracy. 

The project uses a unique dataset made out of 32 950 observations - with some caveats: 
three features are highly correlated (emp.var.rate, euribor3m and nr.employed) and the outcomes are largely imbalanced 
(88.6% rejected the offer, 11.2% accepted it, leading to a null accuracy of almost 89% from the start). 
So, the gap between the accuracy of the first method (91.76%) with the second (91.68%) and the null accuracy (88.6%) is relatively small. 
[Analysis](bankmarketing.html). More work needs to be done on the dataset to provide more positive cases. 

Relying exclusively on the accuracy, the best performing model was the logistics regression, at the end. 

# Details of the setups of the 2 methods

## Method 1 - Logistics Regression (Scikit-learn Pipeline)
**Pipeline architecture, including data, hyperparameter tuning, and classification algorithm.**
The classification algorithm was provided for this part of the project. Most of the 21 features were numerical, the few categorical ones could be easily be one-hot-encoded and the targets were binary ones (0 for rejection, 1 for acceptance). So, a logistics regression model looks well appropriate for such classification task with this size and structure of the dataset. Dealing with a classification, the primary metric that makes most sense in that case is the Accuracy - a criteria that shall be maximized. 

The pipeline architecture was split into a python script, that contains the data cleanup, the split into train & test datasets as well as the logistics regression model itself. 
The script uses a parser to define as arguments the two hyperparameters required by the model: 
- one for the regularization strength and 
- one for the maximum number of iterations. 
This will allow to be called repetitively from the notebook, by HyperDrive, during the search for optimal parameters.

**Benefits of the parameter sampler:**
The pipeline is driven from the Jupyter notebook udacity-project.ipynb that defines the context for the search of the optimal hyperparameters (within that context).
Hyperparameter space: I chose the RandomParameterSampling, mainly for speed reason, as the usual alternative, GridParameterSampling, would have triggered an exhaustive search over the complete space, for a gain that proved to be relatively small at the end. Also, GridParameterSampling only allows discrete values, while random sampling is more open as it allows also the use of continuous values. 

**Benefits of the early stopping policy:**
Regarding early termination of poorly performing runs, I used the BanditPolicy. The BanditPolicy defines a slack factor (defined here to 0.1). All runs that fall outside the slack factor with respect to the best performing run will be terminated, saving time and budget. 

**Tuned hyperparameters and accuracy obtained:**
```
{'Regularization Strength:': 1.1777921298523928, 'Max iterations:': 50, 'Accuracy': 0.9176024279210926}
```

## Method 2 - AutoML
**Model and hyperparameters generated by AutoML.**
In this case, we let AutoML determine the best model and the optimal hyperparameters. 
The model selected was a Voting Ensemble, with the following details
![BestModelDetails](https://user-images.githubusercontent.com/36628203/119268455-5d32f600-bbf3-11eb-8c70-34eefd122935.png)

## Pipeline comparison
AutoML is more comfortable, as it does the model selection for you, not only the hyperparameters optimization as Hyperdrive does. 
Now, I was expecting AutoML to come out with a better accuracy, but it did not. As we used a K-Fold cross-validation parameter of 5, the test set size was set to 20% - it could be interesting to come back to the most common value for k-folds (10) and check the impact, if any, on the accuracy. 

## Future work
**Areas of improvements**


## Proof of cluster clean up
below the command included in the notebook. Please note that the command was executed without error message, but also without returning any message either. 
![Proof](https://user-images.githubusercontent.com/36628203/119268978-a71cdb80-bbf5-11eb-8ad4-076aaef0fd83.png)
