# Cost-Influence-Analysis

# Purpose

This analysis is to be performed on the output of the HHS Buysmarter Mid-Model. This function uses statistical methods (Kruskal-Wallis H-test for categorical variables and correlation t-test for numerical variables) to determine if an attribute is likely to be cost influencing or not cost influencing. 

# Description

This code requries a dataframe from the Mid-Model and a product/service/equipment label as input. It can also be used on a dataframe from the Inference Protocol as input, however, the results may be less accurate since these results are not standardized. The output of this model is a python dicutionary with each relivant variable as a key and either "Cost Influencing" or "Not CI" as values. The confidence level of this model is set to 95%. 
