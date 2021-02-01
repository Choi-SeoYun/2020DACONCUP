# 2020 DACON CUP
[한국어 README](https://github.com/Choi-SeoYun/2020DACONCUP/blob/main/README.md)

https://dacon.io/competitions/official/235683/overview/

- Contest Period: 18.12.2020 ~ 22.01.2021 17:59 (dd.mm.yyyy (hh:mm))
- Actual duration: 28.12.2020 ~ 22.01.2021 (dd.mm.yyyy)
- Team: Yonom unit. - KANG KIM CHOI
- Contents: Time series analysis


## 🏷 Overview

### Topics
Predict future user behavior patterns using historical Dacon data

### Background
Dacon is the largest artificial intelligence composition platform in Korea, with about 20,000 members, about 30,000 participating in competitions, 47 official competitions, and a total prize of 370 million won. At the end of the 2020 year, we predict user behavior patterns based on user behavior data from Dacon.

### Schedule
- Submission of the candidate code: 22 January, 2021 to 25 January, 2021
- Winner Code Evaluation: 25 January 2021 - 29 January 2021

※ The final winner of this competition will be invited to present an online presentation at 6 p.m. on 29 January.

## 🏷 Rules

### Evaluation
- Evaluation criteria: 'Weighted RMSE'

We predict four items: number of users, number of sessions, number of new visitors, and number of page views. Because each variable has a different size, we evaluate the model performance with weighted RMSE.

```shell
def dacon_rmse(true, pred):
    # true.shape // (N,4)
    # pred.shape // (N,4)
    # w0, w1, w2, w3 = train.csv has an average of 4 items: number of users, sessions, new visitors, and page views
    score = np.sqrt(np.mean(np.square(true[:,0] - pred[:,0]))) / w0 +\
            + np.sqrt(np.mean(np.square(true[:,1] - pred[:,1]))) / w1 +\
            + np.sqrt(np.mean(np.square(true[:,2] - pred[:,2]))) / w2 +\
            + np.sqrt(np.mean(np.square(true[:,3] - pred[:,3]))) / w3 +\
     return score
```

- Public Score: Scored with daily sum data from 9 November, 2020 to 8 December, 2020 → Not reflected in evaluation
- Private Score: Scored with daily sum data from 9 December, 2020 to 8 January, 2021 → Reflected in evaluation
- Participants must select the file they want to finally be graded from in the submission window. (Not selecting the final file will automatically select the file they submitted for the first time.)
- Private Score rankings released immediately after the competition are not finalised and the final winner will be determined after user evaluation code verification.


### Individual or Team Engagement Rules

- Maximum number of team members:
- Team composition is available from the start date of the competition to 11th January.
- Teams cannot be merged if the total number of submissions by team members exceeds 36 ((12 days after the competition) * (3 times daily maximum number of submissions))
- Each team member must have at least one submission record when teaming up
- Share the best results submitted by a team member immediately after the team is formed
- Only leaders can add team members and submit results

## 🏷 Data

- 2 total deliveries

### 1. Released December 18, 2020 at 5:00 PM

- train.csv
```
One-hour interval of user behavior data recorded during the period (9 September 2018 to 8 November 2020)

> columns : number of users, sessions, new visitors, page views
```

- submission.csv
```
Daily data from all zeroed (9 November 2020 to 8 January 2021)

> column : number of users, sessions, new visitors, page views
```
- info_user.csv (user info)
- info_login.csv (login information)
- info_compensation.csv (competition information)
- info_submission.csv (code submission information)



### 2. Released on 18 January, 2021 at 1:00 PM
- 2차_train.csv
```
One-hour interval of user behavior data recorded during the period (9 November 2020 to 8 December 2020)
column : number of users, sessions, new visitors, page views
```

## 💡 Models Used

- RandomForest - bad

> - Random forest loses time information as all values of input time step are entered as variables.
> - y values are increasing overall over time. However, the random forest is 'the predicted value of y = the average of the y values of the last leaf', so it cannot predict the out-of-graph y-values. → Decided to use deep learning

- LSTM - good
- GRU - bad
- SEQ2SEQ - not bad
- facebook prophet - good

### Selected Model
- LSTM + facebook prophet
    - LSTM Description
        - 6 lstm layers deep
        - input time step : 30days, output time step : 7days
        - input dimension : 5
            - Given y's (4 things)
            - Number of competition participants (1 type)
        - train data: 100 data from 100 days ago from the forecast date.
        - hidden size = 64, epochs = 500
    - prophet Description
        - Facebook-created time series prediction model
        - Train Full Term Data Usage → Predict by Submission Period
        - input dimension: 1 - self
        - changepoint_prior_scale=0.3
            - Flexibility/variability adjustment
            - default = 0.05
            - Large numbers solve underfitting
        - holidays_prior_scale=20
            - default=10
            - Reflect weekend status more than default value
        - Seasonality_mode = "multiplicative", but the resultant values are deviated and the default value "additive" is used.
        - Use y_hat of the resulting values as the predicted value
        - [prophet코드](https://github.com/hcworkplace/dccup2020/blob/main/DaconCup_04(facebook_prophet).ipynb)



## 💡 Pipeline

- Analysis
    - Modeling (Preprocess + Modeling + Predictive Visualization)
        - Pretreatment
            - hyperparameter : sequence length of input, output
            - That is, how many days to put in and how many days to predict.
    - Modeling
        - hyperparameter : number of Layers, hidden_size, epochs, lr
        - Predictive visualization
        ![image](https://user-images.githubusercontent.com/58651942/105654456-650eb200-5f01-11eb-919e-c41f1162814f.png)
- Validation
    - **Method 1) ** Time series prediction should not use validation as a normally enforced cross-validation. Use 'walk-forward validation' to split the train-test set to time step.
    ![image](https://user-images.githubusercontent.com/58651942/105652357-5eca0700-5efc-11eb-91c3-79d7b19c5b9f.png)

    - **Method 2)** Use secondary_train.csv as validation set


## 💡 Improve the way projects go

- Pipeline flow is carried out as above. Create a basic form to facilitate the progress of the .py file or .ipynb file

```
e.g.
- [n_step]analysis_analysis contents.ipynb
- [n_step]Preprocess.ipynb
    -- Which variable did you use?
    -- input, output appearance
    -- Create data file after pre-processing (like data_prepped_n_step.csv)
- [n_step]Modeling -Model name.ipynb
- [n_step] evaluation.ipynb -> Select Model

> Repeat these steps + organize them in notion by step
```
## 💡 Copyright
- Base line itself is copyrighted by DACON.
- The rest follow MIT copyright.