# Project Phase # 3 & Datathon 6 – High Fidelity Report
Team 16: Priyonto Saha, Hunter Pozzebon, Yacine Marouf

## Introduction
As of 2019, 8.8% of Canadians are living with diabetes and there are approximately 549 new diagnoses daily (LeBlanc et al., 2019). Although there is no conclusive cure for type 2 diabetes (Joslin Education Team, n.d.), insulin resistance can take years to develop, meaning that the prevention and delay of the disease is the best defense against this epidemic (DPPRG, 2002). The predominant methods of Diabetes prevention include lifestyle and diet adjustments (ADA, 2021). Multiple studies report significant correlations between metabolic biomarkers, exercise rates, and diabetes incidence (Biavashi, 2023; Ahmed, 2021). 

Previous studies have been conducted using the same database of electronic medical records (EMRs) that attempted to predict diabetes (Perveen et al, 2019; Perveen et al 2020, Naveed et al., 2023). However, they either attempted to predict future prognostic results, not time to event for diabetes or did not properly adjust for the right censoring that is prevalent with time-to-event data.

As such, our main research questions consist of the following: 
Can we predict the time to diabetes onset based on metabolic biomarker levels and comorbidities?
Which variables are most important in predicting the time to diabetes onset?
What type of language is used in the current research on diabetes prevention?
How often is diversity and equity discussed in research?
In what context are diversity and equity terms used?

This study will incorporate methods used in survival analysis. We plan to design two machine learning models for prediction: A baseline random forest classifier that assumes independence between data points and ignores the temporal correlation that will be used, and a random survival forest (RSF) utilizing survival trees which account for time to event data (Ishwaran, 2008). With a focus on survival and prevention, our model aims to predict the risk of diabetes in conjunction with the amount of time before diabetes onset. This will allow clinicians to advise a patient on how to prolong time before diabetes and determine the best times for a patient to check in for future monitoring and testing.

## Methods
### Dataset 
For this project, we used the CHL5230 dataset, which is a dataset from the Canadian Primary Care Sentinel Surveillance Network (CPCSSN) consisting of 10,000 records from 8602 patients with 43 features. To generate the dataset, systolic blood pressure (sBP) measurements from all patients over the age of 17 were joined with other clinical measurements that were closest in time within a specific time period, alongside the specific dates for the measurements. Then, all records with missing data were removed from consideration, along with patients on insulin who may have type 1 diabetes and patients using corticosteroids which can affect blood sugar levels. This dataset was then randomly sampled to generate the 10,000 records that make up the CHL5230 dataset. The dataset we analyzed includes age and sex, continuous variable clinical measures such as systolic blood pressure (sBP), body mass index (BMI), low-density lipoprotein (LDL), high-density lipoprotein (HDL), triglyceride (TG), fasting blood sugar (FBS), HbA1c (A1c), and total cholesterol (TC), as well as comorbidity binary indicators of depression, hypertension (HTN), osteoarthritis (OA), chronic obstructive pulmonary disease (COPD).

### Exploratory Data Analysis
Exploratory data analysis consisted of missing data analysis, summary statistics, and a correlation matrix. In addition, histograms were plotted to visualize the distribution of the predictors, along with boxplots to determine outliers in the data. To identify the type of missing data we conducted logistic regression models with binary missingness indicators as the outcome, where a value of 1 implied a missing value. Missingness was then deemed missing completely at random (MCAR) if the logistic regression showed no significance and missing at random (MAR) otherwise. However, we cannot fully ignore the possibility of these missing observations being missing not at random (MNAR). We then dropped any records where the only missing values were MCAR and used multiple imputation by chained equations through Scikit Learn’s IterativeImputer command to impute MAR missing data. There is no unexplained correlation found in the correlation matrix. The higher correlation between total cholesterol, HDL, and LDL is explained by the fact that total cholesterol in blood predominantly consists of HDL and LDL. Patients with higher A1c and FBS show higher rates of diabetes outcomes. This is not surprising as they are used as a diagnostic tool for diabetes (Goyal et al, 2023).

### Modeling Pipelines
#### Random Forest
The initial model to be tested as the baseline is a random forest that utilizes the biomarkers: sBP, BMI, LDL, HDL, A1c, TG, FBS and total cholesterol, comorbidities: depression, HTN, OA, COPD, as well as age and sex as the features and diabetes onset, with categories “yes” and “no” as the outcome variable to classify. We first split the data 80:20 as a train-test split, then conducted gridsearch with 5-fold cross-validation to determine the optimal hyperparameters for the random forest. Random forest required no extra data pre-processing for the model and included all the features listed above. The dataset for random forest included the 14 features above and 10,000 records, with `Diabetes` as the target variable.

#### Random Survival Forest
The random survival forest model is an extension of the random forest model that accounts for the presence of censoring in survival data by using the log-rank splitting rule to determine the best split within the tree (Ishwaran, 2008). The start date was calculated by taking the earliest date of a lab test or comorbidity onset for each individual, and the end date was the feature `DM_OnsetDate`. Survival time was calculated by taking the difference between the end date and the start date. Right-censored records with a missing onset date use the end of the study of June 30th, 2015 as their end date to calculate survival time. Since survival analysis requires survival to be 100% at the start of the study, anyone with a negative survival time was removed from the dataset. The final dataset for the RSF included 7,756 records and 15 features, including age, sex, biomarkers, and comorbidities. Survival time is the new target variable, with the diabetes feature being a binary variable of whether the patient developed diabetes also indicating if the patient was right-censored. The random survival forest includes all previously mentioned biomarkers, the date those biomarkers were measured, comorbidities, and the date of diabetes onset. For this model, the train-test split was 75:25. We plan for 100 survival trees in the random forest, each with a max depth of 15, a minimum leaf sample of 100, and a minimum leaf split of 150.

#### Natural Language Processing
To identify diversity terms in research and the context they were in, we utilized word clouds, global vectors for word representation (GloVe) context embedding, and k- means unsupervised learning for clustering and visualization. The dataset of research articles was sourced from the National Center of Biotechnological Information (NCBI) and consisted of 50 primary analysis papers that were compiled with the use of search terms such as: Diabetes AND prevention, diagnosis, risk factors, early detection, prevalence, causes, machine learning, predict. The titles, abstracts, and discussion sections were then recorded. The pipeline for text preprocessing for our word cloud was: contraction removal, lowercasing, special character and number removal, tokenization, stop word removal, and finally lemmatization, with additional stopwords identified by repeated word cloud generation. 

The processed dataset (initially visualized through a word cloud) was vectorized using a pre-trained GloVe embedding model. The tokens were first filtered based on the pre-trained model’s vocabulary and then each transformed into individual 50 dimension vectors. The word embedding was visualized using k-means clusters reduced to 2 dimensions with t-distributed Stochastic Neighbour Embedding (t-SNE).

## Results
Of the 10,000 records for random forest and 7,756 records for random survival forest (RSF), 4 records (3 for RSF)  had a missing sBP measurement, 61 (39) had missing LDL measurements, 72 (65) had missing HDL measurements, 53 (48) had missing TG measurements, and 207 (175) had missing total cholesterol measurements, there were no records missing for comorbidities, age, and sex. After further analysis of the missing data, we determined that HDL and sBP were the only two variables that were MCAR, which allowed us to drop their missing records. LDL, TG, and total cholesterol were MAR and imputed. Thus our data sample size was 9941 records for the random forest and 7,756. 

### Random Forest
The tuned random forest grew 100 trees with a max depth of 10 and a minimum leaf sample of 15. The classification report showed that the model correctly predicted the patient’s diabetes the majority of the time with an accuracy score of 0.86. The model was able to accurately predict patients who have diabetes with a precision score of 0.83 and a recall score of 0.89. The model was also able to predict patients without diabetes accurately with a precision score of 0.89 and a recall score of 0.83. Out of 984 patients with diabetes, the model accurately predicts 867 of them. Out of the 945 patients without diabetes, the model accurately predicted 839 of them (figure 1). A1c and FBS were the most important features in predicting diabetes, followed by LDL and TC.

### Random Survival Forest
The classification report had a concordance index of 0.83, which is the evaluation measure for survival forest ("Evaluating survival models — scikit-survival 0.22.1," n.d.). This means that the model could accurately predict the relative order of diabetes onset time of two randomly selected individuals 83% of the time (Alabdallah et al, 2022).  The most important features within the model were Alc, FBS, and total cholesterol, with feature importance means of 0.1009, 0.0734, and 0.00281 respectively. The standard deviations for these means were all less than 0.005. These results conform with current literature and diabetes monitoring standards (Goyal et al, 2023). The differences in survival probability over time resulting from A1c and FBS are visualized in Figure 2. 

### Natural Language Processing
The word cloud predominantly had clinical statistical terms such as individual, risk, factor, population, and performance. No diversity terms were seen in the word cloud. The k-means model was trained on 10 clusters for the discussion of which three clusters were related to diversity terms. One of the clusters was a mix of miscellaneous terms and typos that also included some oriental cities and ethnicities. A second cluster was predominantly public policy terms in relation to different demographic groups and organizations. The final cluster to note was related to ethnicities, countries, and nationalities. Other clusters revolved around scientific, statistical, metric and quantitative terms. In general, diversity terms were not usually found in the context of datasets and research, but in the context of public policy.

## Discussion

### 4.1 Findings
Using a combination of random forest with a random survival forest to predict diabetes and the time to diabetes onset will allow for a highly accurate and comprehensive diabetes screening and preventative treatment. This will give patients and healthcare workers an estimate of how severe the intervention to delay onset will need to be. Moreover, it can be used to ensure that the intervention is sustainable for the patient based on their goals of delaying diabetes and maintaining a desirable lifestyle.

Based on the NLP analysis, we noticed a serious gap regarding terms related to diversity, equity, and equality in research (figure 3). Based on the word cloud for the discussion text, there was a significant lack of terms representing any target population or demographic. The only words relating to the patients that are significant in the word cloud are “population” and “individuals”. This signifies a low priority in the identification of the target demographics within the healthcare dataset. The k-means analysis supports this evaluation. Out of the 10 clusters, only three had terms that were related to diversity, equity, and equality. One of the clusters was predominantly miscellaneous terms that also had oriental ethnicities. The two main clusters that had terms related to diversity and equity were organized in relation to either nationality or public policy. This implies that diversity terms are only spoken about in relation to either the general location of the dataset or in relation to possible policy and organizational impacts. The terms were not correlated with other research or quantitative analysis terms, which is concerning as it signifies that the datasets and machine learning models do not seem to address or account for possible demographic biases.

### Limitations
Random survival forests were computationally intensive even with tuning parameters specifically chosen for quick training time. This is in part due to a large sample size, and due to random survival forest models not having any GPU compatibility. 

We were unable to train our model on the full dataset due to left censorship for over 2000 observations. Furthermore, a small proportion of records within the dataset were from the same individuals. These records are different time points in which sBP was measured for the same individual but will have the same dates for diagnosis onsets and possible duplicates in measurement dates. This violates the independence assumption for random forests, but due to our feature engineering for the start dates adding inherent variability, we believe our model is not adversely affected. 

The search strategy for NLP was informal with no explicit inclusion or exclusion criteria and only data gathered from the abstract and the discussion, which may result in missing valid and relevant manuscripts and potential biases. The word cloud is a rather qualitative and visual method of analyzing word frequency in the text, which may result in missing information. Finally, utilizing GloVe can result in loss of contextual relationships between the words within the k-means model. 

### Future Research
Parameter tuning through grid search and K-fold CV would allow us to ensure the model maintains an optimal bias-variance tradeoff. This step has not been included within this report since the current C-Index of 0.83 is enough to show the capabilities of our general model pipeline even without thorough parameter tuning, and due to the computational intensity this addition presents. 
We also hope to somehow be able to incorporate the variables related to medication and survival time of comorbidities into our model, but we're unsure as to how to best integrate correlated feature spaces within the random forest model, which is further complicated by the random survival forest splitting rule used. This raised potential research questions of how to design splitting rules designed to accommodate highly correlated feature clusters and multi-modal data within random forest models through feature engineering.
Finally, the diversity analysis can be improved with a formal systematic review that has a proper search strategy with rigid inclusion and exclusion criteria. The language analysis can also include the full research manuscript text for a holistic view of diversity and equity terms in the paper. A final future pathway is to implement Word2Vec embedding for the model along with a more rigorous k-means implementation, or a classification model with pre-labeled text examples of diversity-related phrases.

## Individual Contributions
Yacine Marouf: EDA, data preprocessing (RF), NLP (word cloud, GloVe, k-means), NLP lit search

Hunter Pozzebon: Project management, data preprocessing (RSF), missingness analysis, NLP lit search

Priyonto Saha: Feature engineering, EDA, RSF, RF

Equal contributions towards initial literature review and manuscript writing.

## 6.0 References
Prevention or delay of type 2 Diabetes:Standards of medical care in diabetes—2021. (2020). Diabetes Care, 44(Supplement_1), S34-S39. https://doi.org/10.2337/dc21-s003

Alabdallah, A., Ohlsson, M., Pashami, S., & Rögnvaldsson, T. (2022). The concordance index decomposition - A measure for a deeper understanding of survival prediction models. SSRN Electronic Journal. https://doi.org/10.2139/ssrn.4024162

Ahmed, F., AL-Habori, M., Al-Zabedi, E., & Saif-Ali, R. (2021). Impact of triglycerides and waist circumference on insulin resistance and β-cell function in non-diabetic first-degree relatives of type 2 diabetes. BMC Endocrine Disorders, 21(1). https://doi.org/10.1186/s12902-021-00788-5

Biavaschi, M., Melchiors Morsch, V. M., Jacobi, L. F., Hoppen, A., Bianchin, N., & Chitolina Schetinger, M. R. (2023). Predisposition to type 2 diabetes in aspects of the glycemic curve and glycated hemoglobin in healthy, young adults: A cross-sectional study. Canadian Journal of Diabetes, 47(7), 587-593. https://doi.org/10.1016/j.jcjd.2023.05.009 

Diabetes Prevention Program Research Group. (2002). Reduction in the incidence of type 2 diabetes with lifestyle intervention or metformin. New England Journal of Medicine, 346(6), 393-403. https://doi.org/10.1056/nejmoa012512

Evaluating survival models — scikit-survival 0.22.1. (n.d.). scikit-survival — scikit-survival 0.22.1. Retrieved December 8, 2023, from https://scikit-survival.readthedocs.io/en/stable/user_guide/evaluating-survival-models.html

Goyal R., Singhal M., Jialal I. (2023) Type 2 Diabetes. StatPearls [Internet]. Treasure Island (FL): StatPearls Publishing; 2023 Jan–. PMID: 30020625.

Ishwaran, H., Kogalur, U. B., Blackstone, E. H., & Lauer, M. S. (2008). Random survival forests. The Annals of Applied Statistics, 2(3), 841–860.

Joslin Education Team. (n.d.). Can Type 2 Diabetes Be Reversed? Beth Israel Lahey Health, Joslin Diabetes Center. https://www.joslin.org/patient-care/diabetes-education/diabetes-learning-center/can-type-2-diabetes-be-reversed#:~:text=According%20to%20recent%20research%2C%20type,by%20losing%20significant%20amounts%20of

LeBlanc, A. G., Gao, Y. J., McRae, L., & Pelletier, C. (2019). At-a-glance - Twenty years of diabetes surveillance using the Canadian chronic disease surveillance system. Health Promotion and Chronic Disease Prevention in Canada, 39(11), 306-309. https://doi.org/10.24095/hpcdp.39.11.03

Naveed, I., Kaleem, M. F., Keshavjee, K., & Guergachi, A. (2023). Artificial intelligence with temporal features outperforms machine learning in predicting diabetes. PLOS Digital Health, 2(10), e0000354. https://doi.org/10.1371/journal.pdig.0000354

Perveen, S., Shahbaz, M., Keshavjee, K., & Guergachi, A. (2019). Prognostic modeling and prevention of diabetes using machine learning technique. Scientific Reports, 9(1). https://doi.org/10.1038/s41598-019-49563-6

Perveen, S., Shahbaz, M., Ansari, M. S., Keshavjee, K., & Guergachi, A. (2020). A hybrid approach for modeling type 2 diabetes mellitus progression. Frontiers in Genetics, 10. https://doi.org/10.3389/fgene.2019.01076

Perveen, S., Shahbaz, M., Saba, T., Keshavjee, K., Rehman, A., & Guergachi, A. (2020). Handling irregularly sampled longitudinal data and prognostic modeling of diabetes using machine learning technique. IEEE Access, 8, 21875-21885. https://doi.org/10.1109/access.2020.2968608

Yang, M., Chang, W., Kuo, T., Shen, M., Yang, C., Tien, Y., Lai, B., Chen, Y., Chang, Y., & Yang, W. (2021). Identification of novel biomarkers for pre-diabetic diagnosis using a Combinational approach. Frontiers in Endocrinology, 12. https://doi.org/10.3389/fendo.2021.641336
