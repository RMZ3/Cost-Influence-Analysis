# Cost-Influence-Analysis

# Purpose

This analysis is to be performed on the output of the HHS Buysmarter Mid-Model. This function uses statistical methods (Kruskal-Wallis H-test for categorical variables and correlation t-test for numerical variables) to determine if an attribute is likely to be cost influencing or not cost influencing. 

# Description

This code requries a dataframe from the Mid-Model and a product/service/equipment label as input. It can also be used on a dataframe from the Inference Protocol as input, however, the results may be less accurate since these results are not standardized. The output of this model is a python dicutionary with each relivant variable as a key and either "Cost Influencing" or "Not CI" as values. The confidence level of this model is set to 95%. 

import pandas as pd
import re
import numpy as np
import sys
from scipy import stats
from scipy.stats import mstats
from itertools import repeat

pd.set_option('display.max_row', 500)
pd.set_option('display.max_columns', 50)

data = pd.read_excel("TestData.xlsx")

def cost_inf(df, prod_serv_equip):
    if prod_serv_equip.lower() == 'product':
        cat_vars = ['color', 'size', 'brand', 'model_number', 'part_number', 'vendor', 'shipping_method']
        num_vars = ['quantity', 'weight', 'shipping_time']
    elif prod_serv_equip.lower() == 'service':
        cat_vars = ['software_package', 'vendor', 'education_level', 'license', 'certification', 'specialty']
        num_vars = ['quantity', 'years_in_profession']
    elif prod_serv_equip.lower() == 'equipment':
        cat_vars = ['color', 'software_package', 'size', 'brand', 'model_number', 'part_number', 'vendor', 'shipping_method', 'license', 'certification']
        num_vars = ['quantity', 'weight', 'shipping_time']
    elif prod_serv_equip.lower() == 'all':
        cat_vars = ['color', 'software_package', 'size', 'brand', 'model_number', 'part_number', 'vendor', 'shipping_method', 'education_level', 'license', 'certification', 'specialty']
        num_vars = ['quantity', 'weight', 'shipping_time', 'years_in_profession']
    else: 
        raise ValueError("prod_serv_equip must be either 'product', 'service', or 'equipment'")
    
    results = {}
    
    for j in cat_vars:
        # get the required variables
        df = data[['unit_price',j]]
        
        # drop any record with na on either variable
        df = df.dropna()
    
        # remove all non-numeric values from unit_price except decimal
        # then try to convert to float, if we can't, drop that row
        non_decimal = re.compile(r'[^\d.]+')
        for i, row in df.iterrows():
            try:
                row['unit_price'] = float(non_decimal.sub('', str(row['unit_price'])))
            except:
                row['unit_price'] = np.nan
        df = df.dropna()
        
        # get all unique values (uhhh, I should have just used groupby())
        arr = df[j].unique()
        col_list2 = arr.tolist()
        col_list = [x for x in col_list2 if str(x) != 'nan']
    
        # group all unique values by unit price and store in list of lists
        subsets = [[] for i in repeat(None, len(col_list))]
        for i in range(len(col_list)):
            subsets[i] = list(df[df[j] == col_list[i]]['unit_price'])
        
        # print some stuff out for all the fans at home
        print(j)
        print("number of categories: " + str(len(subsets)))
        
        # ok, time for the Kruskal-Wallis H-test. This will test if variable is cost influencing or not
        ''' 
        The H statistic outcome is essentially a measure of the variance between groups divided by the variance within groups. 
        The higher this number is, the more likely the variable is NOT cost influencing, but this number can easily be skewed
        in peculiar ways if the sample sizes are too small. 
        
        A pval of < 0.05 indicates that the variable IS statistically cost influencing. We really only care about the pvalue here. 
        '''
        args=[l for l in subsets]
        try:
            H, pval = mstats.kruskalwallis(*args)
            print("H statistic:          " + str(H))
            print("p-val:               ", str(pval))
            if pval < 0.05:
                results[j] = "Cost Influencing"
                print("Cost Influencing", '\n')
            if pval >= 0.05:
                results[j] = "Not CI"
                print("Not Cost Influencing", '\n')
        except: 
            print("all identical \n")
            results[j] = "Not CI"
        
    for j in num_vars:
        # get the required variables
        df = data[['unit_price',j]]
        
        # drop any record with na on either variable
        df = df.dropna()
        
        # convert both columns to numeric. If we can't, remove the record
        non_decimal = re.compile(r'[^\d.]+')
        for i, row in df.iterrows():
            try:
                row['unit_price'] = float(non_decimal.sub('', str(row['unit_price'])))
            except:
                row['unit_price'] = np.nan
            try:
                row[j] = float(non_decimal.sub('', str(row[j])))
            except:
                row[j] = np.nan
        df = df.dropna()
        
        # calculate Pearson's correlation coefficient
        r, p = stats.pearsonr(df['unit_price'], df[j])
        
        # convert Pearson's correlation coefficient into a Fishers’ Z-score
        r_z = np.arctanh(r)
        se = 1/np.sqrt(len(df)-3)
        alpha = 0.05
        z = stats.norm.ppf(1-alpha/2)
        lo_z, hi_z = r_z-z*se, r_z+z*se
        lo_z, hi_z
            
        # reverse the transformation
        lo, hi = np.tanh((lo_z, hi_z))
        print(j)
        print("Rz score: ", str(r_z))
        print("Confidence Interval:", lo, hi)
        
        # check if statistic falls within confidence interval, if so, not cost influencing
        if lo < r_z and hi > r_z:
            results[j] = "Not CI"
            print('Not Cost Influencing')
        else:
            results[j] = "Cost Influencing"
            print('Cost Influencing')
        print('')
    return results


results = cost_inf(data, 'product')

results

Returns:
{'color': 'Not CI',
 'size': 'Not CI',
 'brand': 'Cost Influencing',
 'model_number': 'Not CI',
 'part_number': 'Not CI',
 'vendor': 'Cost Influencing',
 'shipping_method': 'Not CI',
 'quantity': 'Not CI',
 'weight': 'Not CI',
 'shipping_time': 'Not CI'}
