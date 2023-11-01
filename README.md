# Project Phase # 2 & Datathon 3 – High Fidelity Report
## Introduction
As of 2019, 8.8% of Canadians are living with diabetes and there are approximately 549 new diagnoses
daily (LeBlanc et al., 2019). Although there is no conclusive cure for type 2 diabetes (Joslin Education
Team, n.d.), insulin resistance can take years to develop, meaning that the prevention and delay of the
disease is the best defence against this epidemic (DPPRG, 2002). The predominant methods of
preventing diabetes include lifestyle and diet adjustments (ADA, 2021). Multiple studies report
significant correlations between metabolic biomarkers, exercise rates, and diabetes incidence (Biavashi,
2023; Ahmed, 2021).
Previous studies have been conducted using the same database of electronic medical records (EMRs) that
attempted to predict diabetes while accounting for temporal inconsistencies by utilizing novel methods in
hidden Markov models (Perveen et al, 2019; Perveen et al 2020). However, they attempted to predict
future prognostic results, not time to event for diabetes. Some previous publications have attempted to
predict diabetes with data from EMRs (Naveed et al., 2023) but the models used in these studies did not
properly adjust for the right-censoring that is prevalent with time-to-event data.
As such, our main research questions consist of the following: 1) Can we predict the time to diabetes
onset based on metabolic biomarker levels and the dates that they were measured compared to the date of
diabetes onset? 2) Which variables affect the time to diabetes onset, and 3) How do they affect the time
to diabetes onset?
This study will incorporate methods used in survival analysis. We plan to design two machine learning
models for comparison: 1) a baseline random forest classifier that assumes independence between data
points and ignores the temporal correlation that will be used, and 2) a random survival forest utilizing
survival trees which account for time to event data (Ishwaran, 2008). With a focus on survival and
prevention, our model aims to predict the risk of diabetes in conjunction with the amount of time before
diabetes onset based on current biomarker test results. This will allow clinicians to advise a patient on
how to prolong time before diabetes and determine the best times for a patient to check in for future
monitoring and testing.
