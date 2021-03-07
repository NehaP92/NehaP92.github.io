---
layout: post
title:      "Formation Classification using Pipelines"
date:       2020-10-10 03:00:16 -0400
permalink:  mod_3_project_blog_draft
---

Formation Evaluation has played an important role in many industries including but not limited to oil & gas, geothermal, and mining. It is also used to determine the facies of a certain rock formation based on the log readings measured during the exploration process. Facies determine the properties of that formation, the result of which is important to determine the presence of oil/gas/water, geothermal properties, etc. Different facies have different log measures and trends based on their properties.

*image

The data used for this classification model takes its input from 5 log readings (Gamma Ray (GR), Resisitivity (ILD_log10), Photoelectric Effect (PE), Neutron-Density Porosity Difference (DeltaPHI), Neutron-Density Porosity (PHID)) at multiple depth intervals and classifies the facies into 9 listed below.

1 : Non-Marine Sandstone
2: Non-Marine Coarse Siltstone
3: Non-Marine Fine Siltstone
4: Marine siltstone and shale
5: Mudstone (Limestone)
6: Wackestone (Limestone)
7: Dolomite
8: Packstone - Grainstone (Limestone)
9: Phylloid-algal bafflestone (Limestone)

### Creating Pipelines

Pipelines significantly reduce the redundancy and time in writing the codes for preprocessing the data and also model optimization. We would first create a function to create pipelines, using scikit learn, for preprocessing and fitting the classification model.

```
def preprocessing_trial(num_cols, cat_cols,
                  cat_imputer=KNNImputer(weights='distance'), 
                  encoder=OneHotEncoder(sparse=False, drop='first',handle_unknown='error'), 
                  num_imputer=KNNImputer(weights='distance'), 
                  transformation=PowerTransformer()):
    
    '''Builds a preprocessing pipeline and column transformation to data containing numerical and/or categorical data 
    based on the chosen classes/preprocessing methods
    --------------------------------
    Inputs:
    
    cat_imputer (class): Imputer class for categorical data. Default - KNNImputer() with weights as distance
    encoder (class): encoding class for categorical data. Default - OneHotEncoder() ignoring the unknowns and dropping first
    num_imputer (class): Imputer class for numerical data. Default - KNNImputer() with weights as distance
    transformation (class): linear scaling or non-linear transformation class. Default - PowerTransformer()
    num_cols (list): numerical columns
    cat_cols (list): categorical columns
    --------------------------------
    Output:
    
    ColumnTransformer pipeline to preprocess a given data
    --------------------------------'''
    
    cat_transformer = Pipeline(steps=[('ohe', encoder),
                                      ('impute', cat_imputer)])
    
    num_transformer = Pipeline(steps=[('impute', num_imputer),
                                  ('scaler', transformation)])
    
    preprocessing = ColumnTransformer(transformers=[('num', num_transformer,num_cols),
                                                ('cat', cat_transformer,cat_cols)])
    return preprocessing
```

The pipeline thus generated first transforms the categorical columns to one hoe encoded, the numerical columns to fill in the missing values and then use the scalar transformation, and lastly combine both numerical and categorical transformations into a single step.

```
def model_pipeline(model,preprocessor):
    
    '''Returns a model object for a given estimator and preprocessor
    ------------------------------------------
    Inputs:
    
    model (sklearn classifier class)
    preprocessor (pipeline or class): sklearn preprocessing class or pipeline
    ------------------------------------------
    
    Outputs:
    
    sklearn modelling pipeline
    -------------------------------------------'''
    
    model = Pipeline(steps = [('preprocessor', preprocessor),
                              ('model', model)])
    return model
```

This pipeline combines the preprossesing pipeline and modeling into one single step. So, when we pass the data into this pipeline, with a single line of code, it will preprocess using all the steps and also fit our scikit learn classification model.


### Scikit Learn Best Classifier

While analysing formations, it is paramount to be aware that the values near to depth are similar than the points away, except at the bounaries, since they are always present in layers. Therefore it is intuitive that KNeighbor Classifier should perform pretty well. The missing values were also filled using the same concept and choosing KNNImputer for the task.

To select the best classifier parameters, we would first fit a base KNN model followed by Scikit Learns's Gridsearch function.

#### Base Model

We will utilize our model pipeline here that will preprocess our data and create the model structure which can then be fit to train.

```
# Split the data into train and validation sets
X_train_model, X_validation, y_train_model, y_validation = train_test_split(X_train, y_train, random_state = 7)

#Define the numerical and categorical columns
categorical = ['Formation','Well Name','NM_M']
numerical = ['Depth','GR','ILD_log10','DeltaPHI','PHIND','PE','RELPOS']
```

```
knn_model_trial = model_pipeline(KNeighborsClassifier(weights='distance'),
                           preprocessing_trial(numerical,categorical, transformation=RobustScaler()))

knn_model = knn_model_trial.fit(X_train_model,y_train_model)
```

Once our base model is trained, we can use it in gridsearch to select the best parameters and so optimize and improve our model.

While using gridsearch with pipelines, it is important to note the format in which the gridsearch taken in the pipeline parameters we wish to optimize:

```
parameter = {'model__n_neighbors': [5,10,15,20],
             'model__weights': ['uniform','distance']}
cv = GridSearchCV(knn_model,param_grid = parameter, scoring = 'f1_macro')
cv.fit(X_train_model,y_train_model)
```

The best parameters can be listed using `model.best_params_`:

```
cv.best_params_
```

*output

Since the best value for n_neighbors is 5, which was the minimum of the range tha twas fed, we may run a new gridsearch with values lower than 5:

```

parameter = {'model__n_neighbors': [2,3,4,5],
             'model__weights': ['uniform','distance']}
cv = GridSearchCV(knn_model,param_grid = parameter,scoring = 'f1_macro')
cv.fit(X_train_model,y_train_model)
```

```
cv.best_params_
```

*output

Based on the classification report, the f1-macro-average score was 0.82.

*table

Looking at the individual errors, Facies 1, 5 and 9 seem to perform well with less errors (low false negatives) Facies 2 seem so show some errors while separating from 3 since both these facies share some similar properties. Similarly, 6 and 8 seem to show some mix ups. Facies 4 seem to have a few false negatives belonging to 6 and 8. A possible explaination of this relies on the size of the grains of these formations. Some of the logs determine porosity of the formation which depends largly on brain size and volumes.

*image

Feature correlation using KNN show that Marine-Non-Marine, PE log values, one of the formations, resistivity log, and depth have high positive influence on the classification inthat respective order, while Gamma ray values, N-D porosity, Delta N-D negatively influence the classification.

*image

Shapiro feature ranking shows the importance of these features to lie in the line of Resistivity N-D Porosity having the greatest influence regardless of the direction, followed by depth, GR, PE, and Relative position.

*image

The best classifier was selected based on a comparative analysis of various other scikit learn classifiers: Random Forest, Decision Tree, SVM, etc. Using pipelines reduces the amount of time utilized and lines of codes required to build all the models and is hence the most usefull tool for effective classification modellig.
