# Contents
1. [Name of Candidate](#name-of-candidate)
2. [Overview of folder structure](#overview-of-folder-structure)
3. [Running instructions](#running-instructions)
4. [Description of logical steps/ flow of pipeline](#description-of-logical-steps-flow-of-pipeline)
5. [Overview of Key findings in EDA and Pipeline, Feature Engineering Choices](#overview-of-key-findings-in-eda-and-pipeline-feature-engineering-choices)
6. [Model choices](#model-choices)
7. [Evaluation choices](#evaluation-choices)
8. [Other Considerations](#other-considerations)
9. [Parting Words](#parting-words)
------------------------------
## Name of Candidate
------------------------------
[Back to content page](#contents)

Hi! My name is :

>Chng Yuan Long, Randy

Email:

>chngyuanlong@gmail.com

------------------------------
##  Overview of folder structure
------------------------------

[Back to content page](#contents)

### Folder structure:
```
AIAP
│  README.md
│  requirements.txt    
│  test-requirements.txt
|  run.sh
|  Dockerfile
|  eda.ipynb
|
└──data
|     survive.db
=======
|  tox.ini
|  
└──data
|  survive.db
|
└──src
   |  main.py
   |  file012.txt
   │
   └──config
   |     config.py
   |   
   └──preprocessing
   |     datamanager.py
   |
   └──tests
   |     test-datamanager.py
   |     test-predict.py
   |     test-train_pipeline.py
   |     test-pipeline.py
   |     test_bound_outliers.py
   |     test_load_from_database.py
   |     test_pipeline.py
   |     test_predict.py
   |     test_preprocess_data.py
   |     test_preprocess_input.py
   |
   └──model
         pipeline.pkl
         pipeline.py
         predict.py
         train_pipeline.py
```
### File Summary:
Format: File (folder)
- Usage

main.py (src)
- runs application

Config.py (src/config)
- Tweak variables in config/config files
   - File paths
   - model specific objects (CV, test ratio, random seed, params for cross validation)
   - Column names
   - Data related like Column names, default values on streamlit UI

Datamanager.py (src/Preprocessing)
- loads pipeline, data
- preprocesses input from application, data from database after reading

python files (src/tests)
- tests functions in the respective python files

Train-Pipeline.py (src/model)
- trains and scores the pipeline with the data in data folder
- outputs pipeline.pkl and a log on training outcome in the same folder

Pipeline.py (src/model)
- contains pipeline to transform data

predict.py (src/model)
- predicts inputs using pipeline trained on data in data folder

------------------------------
## Running instructions
------------------------------

[Back to content page](#contents)

You can run the application straight with either the bash script or from docker.
Optionally, you may run tests or train the pipeline on data in the data folder or run lint tools on the code with Tox.
A trained pipeline named pipeline.pkl should already be included in the src/model folder.

Default values are present on the application itself so that you can click on predict button at the end. If prediction is 0, message 'Please see a doctor!' will appear, otherwise it will appear as 'Please keep up the healthy habits'. Along with the message the predict class and the probability will appear as well. 

The instructions below assumes a Windows OS

### Tox

I have 3 environments in tox (train_pipeline, pytest, lint) with each for a specific function.
You may run tox command like so in the root directory to run all 3 back to back

> tox

Or you can run a specific environment like so

> tox -e pytest


### Running Main Application

1. Bash Script: 

Run bash script (run.sh) by double clicking it. Streamlit application should appear in your browser. 

2. Docker:

Please pull image by running command in terminal with docker running

   >docker pull hashketh/aiap

Once retrieved, please run command 

   >docker run hashketh/aiap

The streamlit should be available in your browser via 

   >localhost:8501

------------------------------
## Description of logical steps flow of pipeline
------------------------------

[Back to content page](#contents)

### Test

I imagine the user would like to test the application first to make sure that everything is working. After that they might want to train the model on the data, or they may wish to use linting tools to assist in cleaning or spotting issues with the code. They may do all using the package tox. 

For the testing, I tried to test as much of the functions in each python file as I can. I used a sample of the database to replicate the loading and preprocessing of the pipeline. This sample is saved in the data file under sample_df.csv. All of the test files are included in the test folder.

### Configuration

This deployment is done in streamlit and all of the variables are stored in config.py in the config folder save for the default values. This goes with all of the other variables like pathing on so on. If they have any configuration to be done they can tweak them in the config.py file.

### Training Data

> Train -> Ingest Data -> Preprocessing -> Train Pipeline -> Score -> Output Results

Say they train the pipeline, the pipeline.py will call on config.py for value of variables, pipeline.py for the loading of pipeline, datamanager.py for loading and preprocessing of the data. 

The preprocessing phase will include all of the transformation that was done from the eda jupyter notebook. This includes imputation of missing values, bounding of the outliers, replacement of the invalid values from smoke, ejection fraction and other features. It will also add the BMI feature.

The pipeline.py will train the pipeline on the data, score it and generate a txt file for the user to view the results. The resulting pipeline will be saved as a pickle file. User can either run the train_pipeline.py file directly or call it from tox.

### Run application

> Run application -> load pipeline -> consume inputs -> preprocess inputs -> predict -> display results

After that they can run the application. The main.py contains the Streamlit UI for it and its filled with the default values provided by config.py. If the user clicks on the predict button, main.py will call predict.py which in turn will call on pipeline.py to load the pipeline and datamanager.py to preprocess the input. Predict.py will generate both the prediction and the probability of the outcome. This will be displayed on the page. 

------------------------------
## Overview of Key findings in EDA and Pipeline Feature Engineering Choices
------------------------------

[Back to content page](#contents)

The dataset contains a moderate amount of features with 150K observations. Numerical features are typically tail heavy with some features require cleaning or imputing. Likewise the categorical features require some cleaning as well. All numerical features do not correlate with each other. 

The pipeline included median imputation of possible null values , bounding outliers within the distribution and the usual scaling or numerical features and one-hot encoding of categorical features. 

As I think that domain knowledge is useful in feature engineering and I do not have any medical knowledge, the only feature introduced is BMI which revealed to be a rather terrible feature. Through feature importance of both the random forest classifier and light Gradient boosting machine, I discovered that 5 features have a higher weight in determining the outcome. They are CK, Smoke, Gender, Diabetes and Age. 

------------------------------
## Model choices
------------------------------

[Back to content page](#contents)

I used the following models
- Logistic Regression (LOGREG)
- Support Vector Machines (SVM)
- K-Nearest Neighbours (KNN)
- Random Forest (RF)
- Light Gradient Boosting Machine (LGBM)

The models used to train were chosen based on how complex the models are, whether they are ensemble models or not and where it is instance or model based. I originally intended to compare the models on the validation data and then choose 1 to perform hyperparamter tunning to achieve better results. However the models happen to give me good results with the default values that I dont need to tune hyperparameters. 

Another selection criterion is also whether if there is any indication of overfitting on the data. Based on the training and test cross validation scores provided, I can see if a model is prone to overfit or not. If there is overfitting I can regularise the model or choose a less complex model. If there is underfitting I will choose a more complex model

On the second iteration, I chose to focus on 5 features with the highest weightage but I am unable to achieve the same score. Although it was a very comparable performance I think in terms of the severity of a false negative in context of the problem I am still comfortable with a perfect score with more features. Furthermore, training time is negligible at this point. Either one of the ensemble model would be fine but I settled on the random forest.

LOGREG is the simplest of them all being a linear model. A simple model has its usefulness however it is unable to fit well onto the data using the default values.

SVM is a very flexible model that allows me to reach both linear or polynominal solutions with its kernel methods and its hyper-parameters. But it requires a bit of knowledge in order to tune the hyperparameters properly.

KNN is an instance based model that does not have any algorithim but predicts based on distance of the training data to the new instances for predictions.

RF is an ensemble model of decision trees but it is prone to overfitting. Typically I will fit using default and then prune (regularise) the tree later. I like that the interpretation of the Tree is easy to understand.

LGBM is an ensemble model that improves on every iteration by adjusting to the residual error of the previous iteration. My understanding is that the LGBM is a variant of XGBoost that is faster. XGBoost is itself a more regularised variant of Gradient Boosting machine.   

I intitally chose to use LGBM as it provided the highest score on all metrics with accuracy taking precedence. It is also the fastest to train. However when I was building the application I have some issues running the LGBM model so I used random forest instead as it is the runner up with the same scoring on all metrics just a tad bit slower when training.

------------------------------
## Evaluation choices
------------------------------

[Back to content page](#contents)

As this is a classification problem, scores like recall, precision, accuracy, F1 score and the ROC AUC score are relevant. I got most of the metrics through sklearn's classification report.

I think its important to know before hand which metric should take priority before the problem is modelled. The problem is about predicting the surival of a patient suffering from heart artery disease and I think between choosing a low false negative rate or a low false positive rate, a low false negative rate will take priority since the outcome of a false positive (predicted death when it is survive) is less disastrous than a false negative (predicted survive when it is death). The model should have high recall

Beyond that, accuracy measures the true positive and true negative rates and is great to know the absolute performance of the model. F1 is good to see as a weighted measure of both recall and precision. F1 is a harmonic mean that is just another measurement of the mean similar to arithmetic mean or the geometric mean (where harmonic mean ≤ geometric mean ≤ arithmetic mean). F1 is penalised by low value and F1 will be high only when all components have high values. 

ROC AUC tells us how much is the model able to distinguish between the positive and the negative class, with 0.5 being an equal chance of the model to label the positive as negative and vice versa. 

All scores are bounded between 0 and 1 inclusive with higher values being better.

------------------------------
## Other Considerations
------------------------------

[Back to content page](#contents)

This deployment is built with ease of use and maintenance in mind. 

A couple of design choices are made to this end:
Tox allows me to run a couple of virtual environments and commands in a easy manner. With Tox, I can use pytest, run lint packages on the code and train the model on the training data with one command regardless of what virtual environment the user is in.

Pytest allows me and any other users to ensure that the code is working properly. I have written pre-train and post-train test cases so that I can cover both the data and the functions in the model and the expected behavior of the model.

Lint tools like black, isort and flake8 formats and flags out inconsistencies with the code, the docstrings and the imports in accordance with PEP8. I hope this improves readability and ease of use for other people using the application.

The model is also containerised in docker so we can avoid the "it only runs on my machine" problem. This is done in the event that the bash script fails to run the application for some reason.

------------------------------
## Parting Words
------------------------------

[Back to content page](#contents)

Thank you for reading all the way to the end of the README! I hope that everything is according to your expectations.

I had fun  practising what I have learnt especially the software engineering aspects of it. Many tutorials or courses on data science stops after you score the model! Thank you for allowing me to participate!
