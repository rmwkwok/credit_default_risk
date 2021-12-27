## Glossory

| term | meaning |
| :--- | :--- |
| PD | probability of default |
| default | delinquency, not necessarily permanent loss |
| exposure / balance / credit exposure / outstanding loan | money lent to users |
| Good user / good month / good exposure | respective bill was paid (fully or partial) |
| Bad user / bad month / bad exposure | respective bill was not paid (less than minimun pay) |
| Transactors | users who paid the full bill within the grace period |
| Revolvers | users who continuously carried a balance on their account |

## Objective

Build a model given user's data to control loss in response to a recession forecast.

## Business model

![](https://raw.githubusercontent.com/rmwkwok/credit_default_risk/main/images/business_model_math.png)

## Control and metrics

In the period of recession we tend to be more conservative which requires a higher expected recall level from the model's "default" prediction. Thus, at development stage, recall is a control, and the **expected precision** and the **expected ROI** are the consequence and thus metrics. At deployment stage, however, real-world recall and ROI became the metrics for both the model and the business.

![business_control_metrics](https://raw.githubusercontent.com/rmwkwok/credit_default_risk/main/images/business_control_metrics.png)

### Expected ROI

It is estimated using a portfolio in August, pretending that the same transactions / overdue amount would happen for Novemeber (which was a bad assumption). When an user is predicted "default", his/her card is freezed so no additional transaction fee nor overdue fee would be generated, and the bank would not suffer loss from the user's transactions.

Only the true good users will be counted to the numerator(revenue) of the ROI, and only the true bad users will be counted towards the denominator (cost) of the ROI.

## Datasets

3 datasets were used for modeling

### raw features (25 features)
Features as provided. Exceptions include:

| Feature | description | 
| :--- | :--- |
| PAY_0 | removed as it distributed differently than the other PAY_X and thus suspicious |
| EDUCATION, MARRAGE | regrouped for balancing |

### raw+engineered features (31 features)
Engineered features in the EDA process include:

| Feature | description | 
| :--- | :--- |
| BILL_VARIANCE | monthly fluctuation of amount billed |
| LONGEST_DELAY | longest delay period ever |
| NO_BAD_MONTHS | number of bad (delinquent) months |
| BILL_TO_CREDIT | resemble debt-to-income ratio |
| PAID_TO_BILLED | Ratio of amount paid to billed. A degree of goodness of user |
| BAD_MONTH_PROXIMITY | how long ago is the last bad month |

### causal model features (5 features; With thought stories, more realistic stories given industry experience)
5 features were picked from the following causal model built on the data and some common sense (better industry knowledge)

![causal model](https://raw.githubusercontent.com/rmwkwok/credit_default_risk/main/images/model.png)

| Feature | possible story (PD: probability of default) | correlation | 
| :--- | :--- | :--- |
| MARRIAGE | - | Married person <-> higher PD |
| LIMIT_BAL | Sharing common causes with PD such as stability of income | Lower credits <-> higher PD |
| LONGEST_DELAY | Sharing common causes with PD such as stability of income | Longer <-> higher PD
| NO_BAD_MONTHS | - | More bad months <-> higher PD |
| BAD_MONTH_PROXIMITY | Upset with recent experience of delinquent penalty | More recent -> higher PD |

## Model results

### Precision-Recall tradeoff

From the following precision-recall tradeoff (right graph), the best model was the upper-most one in the region of interest, which is a LGB decision trees trained with the 31 features.

![](https://raw.githubusercontent.com/rmwkwok/credit_default_risk/main/images/precision_recall_curve.png)

### Profit-Recall tradeoff / ROI-Recall tradeoff

From the model's tradeoffs between profit/ROI and recall, using a recall over 0.6 will start to see profit dropping, and when recall is above 0.8, the ROI became volatile, and required careful examination of the remaining users.

![](https://raw.githubusercontent.com/rmwkwok/credit_default_risk/main/images/profit_roi_recall_curve.png)


### Profit-Cost tradeoff / ROI-Cost tradeoff

The x-axis of the above curve can be translated to the cost, which is proportional to the credit exposure, which is a more instinctive control.

![](https://raw.githubusercontent.com/rmwkwok/credit_default_risk/main/images/profit_roi_cost_curve.png)

## Model improvement action items

| Given | To improve |
| :--- | :--- | 
| Industry experience and knowledge (e.g. current rules for approving credit)| causal model -> robost prediction model |
| Longer period of data | understanding of PD seasonality -> robost prediction model |
| Longer period of data | understanding of spending trend -> better model feature |
| More user data (demographic, credit, etc.) | causal model and more model feature |
| User address | incoporation of macroscopic economic feature (e.g. unemployment, consumption, income) |
| User transaction data | understanding of change of user's spending practice -> more features |
| User transaction data | decoupling of BILL_AMOUNT to TRANSACTION_AMOUNT, PAY_AMOUNT and PENALTY_AMOUNT -> more accurate feature  |

Generally, with more data and understanding,
- an ensemble of models can be built to predict from different perspectives
- better user segmentation (portfolio) could be done for differentiation of treatment and modeling

## General idea of making a plan

An action plan was illustrated in below, together with important inputs showing on the left hand side of the flow chart. One key concern is that usually a grace period can be up to 2 months, which is also the waiting time for us to finally find out if an user will miss the payment or not. The long period of waiting is not favourable in an organization faster-paced, and may invalidate the result for short-term reuse(e.g. due to seasonality). Therefore, sample size control had to be careful to allow more cycles running in parallel while for each cycle maintain the statistical significance. 

![](https://raw.githubusercontent.com/rmwkwok/credit_default_risk/main/images/action_plan.png)

## Modeling summary

- Recall/cost/profit/ROI/etc could be used as control to fine-tune the outcome of the model, 
- Improvement needed to make user segmentation and better prediction results.
- Engineered features were helpful, because (1) it took 3 out of the 5 places in the casual selected feature set, (2) it provided improvement over the raw features.
