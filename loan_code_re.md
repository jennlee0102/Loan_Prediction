---
title: "Perfomance Comparison of Machine Learning Models on Personal Loan Prediction"
author: "Jae Eun Lee"
date: "12/17/2023"
format:
  html:
    embed-resources: true
    theme: cosmo
    code-line-numbers: true
    number_examples: true
    number_sections: true
    number_chapters: true
    linkcolor: blue
editor: visual
fig-cap-location: top
---

## Abstract

In the evolving landscape of the financial sector, loans play a pivotal role in contributing to a bank's income, accompanied by substantial financial risks. The demand for loans is on the rise globally, as highlighted in the 2023 TransUnion report. With the surge in online banking post-2019 pandemic, predicting loan approval has become crucial. This paper delves into the application of various machine learning models, including logistic regression, random forest, XGBoost, and neural network, to enhance loan approval predictions. Through a comprehensive literature review, we draw on studies to underscore the efficacy of machine learning approaches. The main objective of this paper is to find the model with the best performance on predicting whether the loan to particular person will be assigned or not. This paper is mainly divided into three sections: (1) introduction about our project, including the background, literature review and the theoretical knowledge of the models we need to discuss. (2) data processing, describing the data, clean the data, rearrange and spilt into new reasonable data set. (3) Comparison of machine learning models on collected data by some statistical metrics of the result of the prediction of each model.

**Keywords**: Loans; Logistic regression; random forest; XGBoost; Neural network

## 1. Introduction

Loans are a crucial source of income for the financial sector, but they also come with significant financial risks. The interest on loans accounts for a significant portion of a bank's assets. The loan shows an upward trend. According to the 2023 TransUnion [article](https://newsroom.transunion.com/q3-2023-ciir/), the demand for loans is growing and a large number of people apply for loans for various reasons showing an upward trend. However, not all of them can be approved due to the risk of loan default. 3.75% of personal loan accounts are 60 days or more past due. Determining whether to approve loans to specific individuals is getting more important to bank. Furthermore, after 2019 pandemic, online banking is highly activated and the prediction of loan approval is helpful both to customers and bank. Various machine learning models are employed, each offering distinct advantages to enhance the accuracy.

#### **Literature review**

In the study conducted by Arun et al. (2016) focused on loan approval prediction, they proposed five machine learning models: Random Forest, Support Vector Machine, Linear Models, Neural Network, and Adaboost.

[Xia et al. (2017a)](https://www.sciencedirect.com/science/article/abs/pii/S0957417417301008?via%3Dihub) illustrated how they conducted a personal credit risk assessment using the XGBoost model. Their approach involved handling missing values, normalizing the initial data, employing the XGBoost model to assess feature importance, and ultimately utilizing the trained and optimized XGBoost model for predictive tasks.

In the 2019 research by [Zhou et al.](https://www.sciencedirect.com/science/article/abs/pii/S0378437119313652), they linearly weighted to the prediction using gradient-boosting decision trees, XGBoost, and LightGBM models. And they concluded that they are effective win unbalanced high-dimensional samples.

Additionally, the 2021 study by [Bhattad et al.](http://www.ijcstjournal.org/volume-9/issue-3/IJCST-V9I3P21.pdf), specifically focused on the decision tree model. Here, decision tree model showed 77% accuracy, highlighting the model's efficacy in classification tasks. Logistic regression demonstrated commendable accuracy at 78.91%, showcasing its strength in estimating parameters for binary regression and its relevance to tasks like loan approval. The research expanded to Random Forest, an ensemble learning method characterized by constructing multiple decision trees during training. Through extensive trials, including supervised and unsupervised transformation, the Random Forest achieved a remarkable accuracy of 80.20%, underscoring its effectiveness in handling diverse datasets.

In the recent studies, more deep learning approaches are applied to prediction with the larger data. In 2023 study by [Zhu et al.](https://www.sciencedirect.com/science/article/pii/S2666764923000218#bbib29), Personal loan prediction models are based on XGBoost, LightGBM, decision tree, and logistic regression, and this research came to the end with the conclusion that the LightGBM model has the best prediction performance.

In summary, this literature review provides valuable insights for advancing research in the field, guiding the development of a reliable bank loan prediction model.

While the studies mentioned above have yielded positive results through the application of machine learning techniques, prediction analyses alone do not provide decision-makers with sufficient information. Despite having a highly accurate predictive model, the decision-making process may not be reasonable.

This literature review helps us carry out our work and propose a reliable bank loan prediction model.

## 2. Data Description

The dataset for this project is sourced from Kaggle, available at [Loan Prediction Dataset](https://www.kaggle.com/datasets/yashpaloswal/loan-prediction-with-3-problem-statement).

### Variables Description

-   **Loan_ID**: Unique id for every loan request
-   **Gender**: Applicant's Gender
-   **Married**: Applicant's Marital status
-   **Dependents**: Number of dependents on the Applicant in the family
-   **Education**: Applicant's Education
-   **Self_Employed**: Whether the applicant is employed or the owner of the business
-   **ApplicantIncome**: Applicant's recorded income in US\$
-   **CoapplicantIncome**: Co-Applicant's recorded income in US\$
-   **LoanAmount**: Amount of Loan required in US\$
-   **Loan_Amount_Term**: Duration in which the applicant wants to pay back the loan amount
-   **Credit_History**: Whether the candidate has a good or bad credit history in the past
-   **Property_Area**: Type of locality of property given as mortgage, indicating property valuation
-   **Loan_Status**: Whether the loan was approved or not

The data was originally separated into training and test dataset.

```{r}
train <- read.csv("data/training_set.csv", header = TRUE)
test_df <- read.csv("data/testing_set.csv", header = TRUE)
sample_submission <- read.csv("data/sample_submission.csv")
```

We can see the imbalance of response variable between training and test set. Checking balance/distribution is important in classification problem because class imbalance can lead to biased models.

```{r}
prop.table(table(train$Loan_Status))
```

```{r}
prop.table(table(sample_submission$Loan_Status))
```

### Pre-processing

We merged them and then re-split to ensure randomizing the order of the examples in data set.

```{r}
test_merge <- merge(test_df, sample_submission, by = 'Loan_ID')
test_merge <- subset(test_merge, select = -X)
```

```{r, echo = FALSE}
# combine two dataset
df <- rbind(train, test_merge)

set.seed(42)

# shuffle the data
df <- df[sample(nrow(df)), ]
```

Original data set has 981 rows and 12 predictors.

```{r}
# convert response variable "Loan_Status" to a factor variable:
df$Loan_Status <- as.factor(ifelse(df$Loan_Status == "Y", 1, 0))
```

```{r}
# clean NA and blanks
df <- na.omit(df)
df <- df[!apply(df, 1, function(row) any(row == "")), ]
dim(df)
```

```{r}
# take out unnecessary column from data frame
df<- subset(df, select = -Loan_ID)
```

```{r}
# convert categorical variables to factors:
df$Gender <- as.factor(df$Gender)
df$Married <- as.factor(df$Married)
df$Dependents <- as.factor(df$Dependents)
df$Education <- as.factor(df$Education)
df$Self_Employed <- as.factor(df$Self_Employed)
df$Credit_History <- as.factor(df$Credit_History)
df$property_Area <- as.factor(df$property_Area)
```

After cleaning the data, we have got 687 rows in training set and 78 rows in the test set with 11 predictors and a response variable.

```{r}
# split the data into train(90) and test(10)
train.prop <- 0.90

strats <- df$Loan_Status
rr <- split(1:length(strats), strats)
idx <- sort(as.numeric(unlist(sapply(rr, function(x) sample(x, length(x) * train.prop)))))

df.train <- df[idx, ]
df.test <- df[-idx, ]

dim(df.train)
dim(df.test)
```

We then check the proportion of the two level of the binary response are the same in the train, test, and entire data.

```{r}
prop.table(table(df.train$Loan_Status))
```

```{r}
prop.table(table(df.test$Loan_Status))
```

```{r}
prop.table(table(df$Loan_Status))
```

We proceeded to plot distribution of some predictors. We used histogram to visualize predictors in the numerical values.

```{r, echo = FALSE}
par(mfrow=c(1,2))

hist(df.train$ApplicantIncome, main="figure 1.\nDistribution of ApplicantIncome", breaks=20, 
     col='lightblue', prob=T, xlab="ApplicantIncome")

hist(df.train$CoapplicantIncome, main="figure 2.\nDistribution of CoapplicantIncome", breaks=25, 
     col='lightpink', prob=T, xlab="CoapplicantIncome")
```

In figure 1, it seems that most of the applicants have income under 10000 and figure 2 shows that the most of the co-applicants have income under 5000.

```{r, echo = FALSE}
library(ggplot2)
library(gridExtra)

Gendertable <- data.frame(table(df.train$Gender))
colnames(Gendertable) <- c("Gender", "Count")

plot1 <-ggplot(Gendertable, aes(x = "", y = Count, fill = Gender)) +
  geom_col() +
  coord_polar(theta = "y") +
  scale_fill_manual(values = c("lightcoral","lightblue")) +
  ggtitle("figure 3.\nGender Distribution")

property_Areatable <- data.frame(table(df.train$property_Area))
colnames(property_Areatable) <- c("property_Area","Count")
plot2 <-ggplot(property_Areatable, aes(x = "", y = Count, fill = property_Area)) +
  geom_col() +
  coord_polar(theta = "y") +
  ggtitle("figure 4.\nArea Distribution")

grid.arrange(plot1, plot2, ncol = 2)
```

Figure 3 indicates that the male applicants constitutes a larger proportion than the female applicants. And as we can see in figure 4, it appears that applicants from rural, semi-urban, and urban areas have a similar distribution.

## 3. Goal

This paper aims to achieve the best predictive performance by comparing machine learning models on predicting whether the loan to particular person will be assigned or not. We will apply logistic regression, random forest, XGBoost, and neural network and assess which model can predict loan approval the best. We operate with distinct training and testing data sets.

## 4. Methodology

In this project, we will use four methods to get four different models to make the loan prediction, then we will compare their performance. The four models are logistic regression, random forest, gradient boosting and neural network separately. All of them can do the binary classification. Some technical details are given on the following.\

1.  logistic regression\
    The binary logit (or, logistic regression) model is a generalized linear model (GLIM) for explaining binary responses $Y_i$. Our goal is to model the binary responses as functions of 11 independent variables denoted by $X_{i,j},~j=1,\ldots,11$ for each $i$.\
    The *random component* of the GLIM is $$Y_i | \pi_i \sim\mbox{Bernoulli}(\pi_i).$$ The *systematic component* is $$\eta_i = \beta_0 + \sum_{j=1}^{11} \beta_j X_{i,j} = \mathbf{x}_i' \boldsymbol{\beta}.$$ with $\boldsymbol{\beta} = (\beta_0, \beta_1, \ldots,\beta_{11})'$, and $\mathbf{x}_i = (1, X_{i,1},\ldots, X_{i,11})'$.\
    The *logit link function* relates the $i$th mean response $\pi_i$ to the systematic component $\eta_i$: $$\mbox{logit}(\pi_i | \mathbf{x}_i) = \log\left(\frac{\pi_i}{1-\pi_i}\right) = \eta_i.$$\
    Since the mean response $\pi_i$ must lie in the interval $(0,1)$, whereas $\eta_i$ is real-valued, we need a function such as the logit function to link the two in a correct way.\
    By inverting the logit link function, we can write the binary logit model as $$\pi_i = P(Y_i =1 | \mathbf{x}_i) = \frac{\exp(\beta_0 + \sum_{j=1}^p \beta_j X_{i,j})}{1+ \exp(\beta_0 + \sum_{j=1}^p \beta_j X_{i,j})}.$$\
    We can estimate our coefficients of the model by maximize the log-likelihood function: $$\ell(\boldsymbol{\beta};\mathbf{y}) = \sum_{i=1}^n Y_i \mathbf{x}_i'\boldsymbol{\beta} - \sum_{i=1}^n\log[1+\exp(\mathbf{x}_i'\boldsymbol{\beta})].$$\
    We use the FS algorithm to obtain the maximum likelihood estimates (MLE's) $\hat{\beta}_{j}$ of the coefficients $\beta_j$ by maximizing $\ell(\boldsymbol{\beta}; \mathbf{y})$. For large $n$, the $z$-stat is useful for checking whether $\beta_j$ is a significant coefficient.

2.  random forest\
    The random forest (RF) is an *ensemble learning* method which consists of aggregating a large number of decision trees to avoid overfitting and build a better classification model.\
    The word *random* appears because in training the data, predictors are chosen randomly from the full set of predictors.\
    The word *forest* is used because output from multiple trees are used to make a decision. That is, two types of randomnesses go into constructing a random forest: (1)each tree is built on a random sample from the dataset, and (2) at each tree node, a subset of features are randomly selected to generate the best split.

3.  gradient boosting\
    Like random forests, *boosting* is also an out-of-the box learning algorithm. It gives good predictive performance for the response, usually in high-dimensional settings, with a large number of features.\
    Random forests build an ensemble of independent deep trees. In contrast, gradient boosting algorithms (GBMs) successively build an ensemble of shallow trees, each tree learning from the previous tree. When combined, these trees provide a highly accurate predictive algorithm.\
    For binary response modeling, the idea of boosting was introduced to improve the performance of *weak learners*. This was done by resampling the training data responses, giving more weight to the misclassified ones, thereby leading to a refined classifier (binary model) which would boost feature performance, especially in ambiguous areas of the feature space.\
    A popular variant is the *gradient boosting algorithm*, and XGBoost (acronym for eXtreme Gradient Boosting) is an increasingly popular tool for predictive modeling which we can fit using the *xgboost* package. This is sometimes referred to as regularized gradient boosting, and is thought to be faster and more accurate than using a package like *gbm*.

4.  neural network\
    A neural network is a computational model that simulates the structure and function of a biological nervous system, the basic unit of which is the neuron. A neural network consists of multiple layers of neurons, usually including an input layer, a hidden layer (there may be more than one), and an output layer. Each neuron is connected to all neurons in the previous layer, and each connection has a weight. A neural network learns to adjust these weights in order to enable it to perform effective pattern recognition and prediction on input data.\
    Neural networks can be used for binary variable classification mainly because of the following properties:\
    The hidden layer of neural networks introduces nonlinear mapping through nonlinear activation functions (e.g., ReLU, Sigmoid, Tanh, etc.), which allows the network to learn complex input-output relationships including nonlinear decision boundaries, which is important for dealing with complex classification problems.\
    Neural networks adaptively adjust weights from training data to learn features and patterns of the data. Through optimization algorithms such as gradient descent, neural networks are able to minimize the loss function, making the model's predictions close to the actual labels.\
    Neural networks are able to perform feature learning automatically, eliminating the need to provide features manually. By combining multiple hidden layers, the network can learn to extract high-level abstract features from the input data, which is useful for classification tasks.\
    The output layer typically uses a single neuron with a Sigmoid activation function to map a linear combination to the range \[0,1\], which can be interpreted as the probability that the sample belongs to a positive category.\
    Overall, the ability of neural networks to flexibly adapt to complex nonlinear relationships, automatically learn features, and output results in the form of probabilities makes it a powerful tool for solving binary variable classification problems.

## 5. Results from the Analysis

#### 5-1. Logistic Regression

***research question***: Are these variables useful for predicting whether the customer is eligible for loan?

##### a. full model

```{r}
full.logit <- glm( Loan_Status~ . ,data = df.train, 
                  family = binomial(link = "logit"))
summary(full.logit)
```

The p-value in MarriedYes, Credit_History1, and property_AreaSemiurban has lower value than 0.05, implying that they are statistically significant.

##### b. null model

```{r}
null.logit <- glm(Loan_Status ~ 1, data = df.train, 
                  family = binomial(link = "logit"))
summary(null.logit)
```

##### c. both model

The following steps show how we can use both backward and forward selection to choose predictors to explain the response variable, using the option direction="both" in the step() function.

```{r}
both.logit <- step(null.logit, list(lower = formula(null.logit),
                                    upper = formula(full.logit)),
                   direction = "both", trace = 0, data = df.train)
formula(both.logit)
summary(both.logit)
```

Three variables---Credit_History, property_Area, and Married---are significant both in full and both model. In full model, coefficient of ApplicantIncome is 9.592e-06, indicating that higher income leads to the higher probability of loan approval. Also, the above three variables with small p-values are included in the final logistic regression model after the stepwise selection process. The stepwise selection procedure aims to identify a subset of variables that optimizes a specific criterion (such as AIC) within the given data set. The both model has the smallest AIC, so it combines the best predictive performance and model generalization ability; The full model has the smallest residual, so it has the best ability to interpret the data. However, both model does not differ much in both AIC and residuals, so we need to make further judgment by their performance on the test set.

Now, we assess how well the models full.logit and both.logit fit the response from the test data.

```{r}
pred.full.test <- predict(full.logit, newdata = df.test, type = "response")
(table.full.test <- table(pred.full.test > 0.5, df.test$Loan_Status))
```

```{r}
pred.both.test <- predict(both.logit, newdata = df.test, type = "response")
(table.both.test <- table(pred.both.test > 0.5, df.test$Loan_Status))
```

```{r}
library(caret)
f <- ifelse(pred.both.test > 0.5,1,0)
(cm.logit <- confusionMatrix(reference=as.factor(df.test$Loan_Status), 
            data=as.factor(f), mode="everything"))
```

We see that both models have the same values on the confusion matrix with accuracy of 91.03%, sensitivity of 0.65, and specificity of 1.

### 5-2. Random Forest

```{r}
library(ranger)
fit.rf.ranger <- ranger(Loan_Status ~ . , data = df.train, 
                   importance = 'impurity', mtry = 3)
print(fit.rf.ranger)
```

OOB prediction error of 12.52% indicates the expected error rate on new, unseen data.

After training a random forest, we would like to understand which variables have the most predictive power. Variables with high importance will have a significant impact on the binary outcomes.

```{r}
# variable importance
library(vip)
(v1 <- vi(fit.rf.ranger))
```

This table shows how important each variable is in classifying the data. In random forest model, Credit_History has the highest importance.

We can now use the fitted Random Forest model to predict responses from the test data, and create the confusion matrix. The model correctly predicted 13 data as 0 and 57 data as 1.

```{r}
pred <- predict(fit.rf.ranger, data = df.test)
test_df <- data.frame(actual = df.test$Loan_Status, pred = NA)
test_df$pred <- pred$predictions
(conf_matrix_rf <- table(test_df$pred, test_df$actual))
```

In test set, the accuracy of the model is 89.74%, the sensitivity is 0.65, and the specificity is 0.98.

```{r}
sum(diag(conf_matrix_rf))/sum(conf_matrix_rf)
```

```{r}
sensitivity(conf_matrix_rf)
```

```{r}
specificity(conf_matrix_rf)
```

We checked the performance in train set.

```{r}
pred.train <- predict(fit.rf.ranger, data = df.train)
train_df <- data.frame(actual = df.train$Loan_Status, pred = NA)
train_df$pred <- pred.train$predictions
(conf_matrix_rf <- table(train_df$pred, train_df$actual)) #confusion matrix
```

The accuracy in train set is 98.25%.

```{r}
round(sum(diag(conf_matrix_rf))/sum(conf_matrix_rf)*100, 2)
```

### 5-3. XGBoost

```{r,  echo = FALSE}
library(xgboost)
library(Matrix)
# Transform the predictor matrix using dummy (or indicator or one-hot) encoding 
matrix_predictors.train <- 
  as.matrix(sparse.model.matrix(Loan_Status ~., data = df.train))[, -1]
matrix_predictors.test <- 
  as.matrix(sparse.model.matrix(Loan_Status ~., data = df.test))[, -1]
```

```{r, echo = FALSE}
# Train dataset
pred.train.gbm <- data.matrix(matrix_predictors.train) # predictors only
#convert factor to numeric
loan.train.gbm <- as.numeric(as.character(df.train$Loan_Status))
dtrain <- xgb.DMatrix(data = pred.train.gbm, label = loan.train.gbm)
# Test dataset
pred.test.gbm <- data.matrix(matrix_predictors.test) # predictors only
 #convert factor to numeric
loan.test.gbm <- as.numeric(as.character(df.test$Loan_Status))
dtest <- xgb.DMatrix(data = pred.test.gbm, label = loan.test.gbm)
```

```{r,  echo = FALSE}
# set up watchlist (two list) and param
watchlist <- list(train = dtrain, test = dtest)
param <- list(max_depth = 2, eta = 1, nthread = 2,
              objective = "binary:logistic", eval_metric = "auc")
```

We fit the XGBoost model and display the accuracy(AUC) for the training and test data at each of nround=9 rounds.

```{r}
model.xgb <- xgb.train(param, dtrain, nrounds = 9, watchlist)
```

We would like to see which variable has a high importance.

```{r}
xgb.importance(model = model.xgb)
```

We can use the fitted model to make predictions on the train data as follows and check the accuracy, sensitivity and specificity.

```{r}
# performance in test set
pred.y = predict(model.xgb, pred.test.gbm)
prediction <- as.numeric(pred.y > 0.5)

# Measure prediction accuracy on test data
(tab1<-table(prediction,loan.test.gbm))
```

```{r}
# accuracy
round(sum(diag(tab1))/sum(tab1)*100, 2)
```

```{r}
# sensitivity (recall)
sensitivity(tab1)
```

A sensitivity of 0.65 implies that the model correctly identified 65% of the true positive cases

```{r}
# specificity
specificity(tab1)
```

A specificity of 0.9655172 suggests that the model accurately identified 96.55% of the true negative cases.

Similarly, we can predict and test accuracy on the train data.

```{r}
pred.y.train <- predict(model.xgb, pred.train.gbm)
prediction.train <- as.numeric(pred.y.train > 0.5)
# Measure prediction accuracy on train data
(tab<-table(prediction.train,loan.train.gbm))
```

```{r}
round(sum(diag(tab))/sum(tab)*100, 2)
```

### 5-4. Neural Network

```{r, echo = FALSE}
library(neuralnet)
```

Normalization in neural networks improves the learning of meaningful features, promotes stability in the gradient descent process, and leads to enhanced performance on the training data. Then we process the data by hot-coding and changing names of some columns so that the data can be put into the model to analyze.

```{r}
# normalize data
normalize <- function(x) {
        return ((x - min(x)) / (max(x) - min(x)))
}
```

```{r, echo = FALSE}
df.train$ApplicantIncome <- normalize(df.train$ApplicantIncome)
df.train$CoapplicantIncome <- normalize(df.train$CoapplicantIncome)
df.train$LoanAmount <- normalize(df.train$LoanAmount)
df.train$Loan_Amount_Term <- normalize(df.train$Loan_Amount_Term)
df.test$ApplicantIncome <- normalize(df.test$ApplicantIncome)
df.test$CoapplicantIncome <- normalize(df.test$CoapplicantIncome)
df.test$LoanAmount <- normalize(df.test$LoanAmount)
df.test$Loan_Amount_Term <- normalize(df.test$Loan_Amount_Term)
```

```{r, echo = FALSE}
# hot-coding
df.train.nn <-as.matrix(sparse.model.matrix( ~., data = df.train))[, -1]
df.test.nn <-as.matrix(sparse.model.matrix( ~., data = df.test))[, -1]
# change column name
colnames(df.train.nn)[which(colnames(df.train.nn) == "Dependents3+")] <- "Dependents3p"
colnames(df.train.nn)[which(colnames(df.train.nn) == "EducationNot Graduate")] <- "Education"
colnames(df.test.nn)[which(colnames(df.test.nn) == "Dependents3+")] <- "Dependents3p"
colnames(df.test.nn)[which(colnames(df.test.nn) == "EducationNot Graduate")] <- "Education"
```

After processing the data, we can train our neurall network.

```{r}
# fit the nn
set.seed(123)
net_fit <-neuralnet(Loan_Status1~., data=df.train.nn, hidden = c(10,10), 
                        lifesign = "none", algorithm = "sag", 
                        err.fct = "sse", act.fct = "logistic",
                     linear.output=FALSE)
```

Then we can check the performance.

```{r}
# performance on train set
pred_nn.train <- predict(net_fit,newdata = df.train.nn)
prediction_train <- as.numeric(pred_nn.train > 0.5)
(tab1<-table(prediction_train,loan.train.gbm))
```

```{r}
sum(diag(tab1))/sum(tab1)
```

```{r}
pROC::roc(loan.train.gbm,prediction_train)
```

The neural network performs well in the train set, with high accuracy and high AUC.

```{r}
# performance on test set
pred_nn.test <- predict(net_fit,newdata = df.test.nn)
prediction_test <- as.numeric(pred_nn.test > 0.5)
(tab1<-table(prediction_test,loan.test.gbm))
```

In test data, the accuracy, sensitivity and specificity are lower than other three models above.

```{r}
round(sum(diag(tab1))/sum(tab1)*100, 2)
```

```{r}
sensitivity(tab1)
```

```{r}
specificity(tab1)
```

```{r}
# AUC
pROC::roc(loan.test.gbm,prediction_test)
```

AUC score is approximately 0.7, meaning that the model has some discriminatory power, but there is room for improvement.

## 6. Conclusion

The main purpose of the paper is to use predictive models to classify and analyze the loan applicants, and check which of them is most predictive. In this study, we applied logistic regression, random forest, XGBoost, and neural network for the prediction of loan approval. The prediction abilities of the linear regression and XGBoost models were better than those of the random forest and neural network models.

We found that Married, property_Area, and Credit_History variables are significant in full and both model, and both model was improved by variable selection. Both of them have almost the same performance in predicting whether to provide a loan to an applicant with the accuracy of 91.03%. In random forest model, among all variables, Credit_History, ApplicantIncome, and LoanAmount are important variables, they make a huge contribution on predicting the Loan_Status, and the accuracy of the random forest we built was 89.74%. In XGBoost model, the accuracy was 91.03% which is the same as in logistic regression model. For the last model, we constructed a neural network with 10 hidden layer and each layer has 10 nodes. This model has a good performance on training set but bad performance in test set with the AUC of 0.6888. We concluded that its reason is that the size of the data is too small.

The predictive ability of machine learning models will be further improved. For further study, it is imperative to explore additional datasets and engage in feature engineering to enhance our ability to predict whether customers will default on payments. Then, we can anticipate better performance especially in neural network model.

Among the four models considered, a comparison of statistical metrics on the test set indicates that both the logistic regression model and XGBoost exhibit superior predictive performance on our project's loan dataset. If the bank obtains a similar dataset from loan applicants, I would recommend considering logistic regression as a reference for making loan decisions. It's important to note that while machine learning results provide efficiency, they should complement rather than completely replace human decision-making. Therefore, the model outcomes can serve as a valuable tool to streamline the decision-making process.

## References

Ekta Gandotra, Divya Bansal, Sanjeev Sofat 2014, 'Malware Analysis and Classification: A Survey'available from http://www.scirp.org/journal/jis

Transunion 2023, 'Credit Balances on the Rise as Consumers Manage Higher Costs' available from https://newsroom.transunion.com/q3-2023-ciir/

Yufei Xia, Chuanzhe Liu, YuYing Li, Nana Liu 2017, 'A boosted decision tree approach using Bayesian hyper-parameter optimization for credit scoring' available from https://www.sciencedirect.com/science/article/abs/pii/S0957417417301008?via%3Dihub

Jing Zhou, Wei Li, Jiaxin Wang, Shuai Ding, Chengyi Xia 2019, 'Default prediction in P2P lending from high-dimensional data based on machine learning' available from https://www.sciencedirect.com/science/article/abs/pii/S0378437119313652

Sanket Bhattad, Sumit Bawane, Shweta Agrawal, Unnati Ramteke, Dr. P. B. Ambhore 2021, 'Loan Prediction using Machine Learning Algorithms' available from http://www.ijcstjournal.org/volume-9/issue-3/IJCST-V9I3P21.pdf

Xu Zhu, Qingyong Chu, Xinchang Song, Ping Hu, Lu Peng 2023 'Explainable prediction of loan default based on machine learning models available from https://www.sciencedirect.com/science/article/pii/S2666764923000218#bbib29
