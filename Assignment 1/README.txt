Eric Cummings // BIS 643 // Sept 19, 2022

******************
Exercise 1
******************

def temp_tester(norm):
    def is_healthy(temp):
        if(abs(temp-norm)<=1): ## abs gives absolute value
            return True
        else:
            return False
    return is_healthy
    
human_tester = temp_tester(37)
chicken_tester = temp_tester(41.1)

print(chicken_tester(42)) # True -- i.e. not a fever for a chicken
print(human_tester(42))   # False -- this would be a severe fever for a human
print(chicken_tester(43)) # False
print(human_tester(35))  # False -- too low
print(human_tester(98.6)) # False -- normal in degrees F but our reference temp was in degrees



******************
Exercise 2
******************

Bin sizes: the choice of bin sizes (i.e. number of bins) influences the data visualization in the following ways: too few bins, and you don't have enough precision. Too many bins and you lose the resuming power of the graph.

Finding the outlier:
We use the following sql query: "SELECT name, weight, age FROM population WHERE age>30 AND weight < 30"
We are looking for the name (and weight/age) of a patient in the table population who is an adult (above 30) and who is very light (less than 30 kg). The values were chosen using the scatterplot made earlier.

# Setting up our data

We load the given database into Python as a pandas DataFrame:

import pandas as pd
import matplotlib.pyplot as plt
import math

import sqlite3
with sqlite3.connect("hw1-population.db") as db:
    data = pd.read_sql_query("SELECT * FROM population", db)

df = pd.DataFrame(data)

df

Our data has four columns : name, age, weight, eyecolor (the "fifth" column on the very left contains the unique keys of every row/person. Our data has 152361 rows/persons (the count starts at 0 in the table).

# Distribution of Age

ages = df.age #treating age as an object property

ages_mean = ages.mean()
print(f'Mean age in years: {ages_mean}')

ages_stdev = ages.std()
print(f'Standard deviation in years: {ages_stdev}')

ages_min = ages.min()
print(f'Minimum age in years: {ages_min}; in hours that is: {ages_min*12*30*24}')

ages_max = ages.max()
print(f'Maximum age in years: {ages_max}')


To determine the number of bins for a useful histogram, we use a simplified version of Sturge's rule stating that
K = 1 + 3.322 log10(N)

With K the optimal number of bins and N the size of our dataset and
N the size of our dataset (rows)


1+3.322*math.log10(len(df.index))

df.hist(column='age', bins=18)

This dataset seems to contain mainly people aged 0-65 and five times less seniors (70-100).

## Distribution of Weight

weights = df.weight #treating weight as an object property

weights_mean = weights.mean()
print(f'Mean weight in kg: {weights_mean}')

weights_stdev = weights.std()
print(f'Standard deviation in kg: {weights_stdev}')

weights_min = weights.min()
print(f'Minimum weight in years: {weights_min}')

weights_max = weights.max()
print(f'Maximum weight in kg: {weights_max}')

df.hist(column='weight', bins=18)

Given that the dataset is composed of a large majority of biologically adult humans, this distribution makes sense, as 70 kg is a "normal" adult weight and the long tail to the left is explained by the fact that babies, children and adolescents are also included in the dataset.

## Weight vs Age

weight_vs_age_scatterplot = df.plot(
    kind='scatter', #specify type of plot
    title= 'Scatterplot of Age vs Weight', #Specify plot title
    x='age', #Specify column for x axis
    y='weight', #Specify column for x axis
    s=1
    )

The scatterplot reflects the fact that between the ages of 0 and 20 (childhood + adolescence), there is a relationship between weight and age (the older, the heavier). Once an adult, there is no such relationship anymore: people stop growing. 
There is one clear outlier in this dataset: one person aged about 42y weighs about 20kg. We shall find the name of this patient with the following sql query:

weight_outlier_patient = pd.read_sql_query("SELECT name, weight, age FROM population WHERE age>30 AND weight < 30", db)

print(weight_outlier_patient)

The outlier is Anthony Freeman, aged 41.3 years and weighing 21.7kg.




******************
Exercise 3
******************

Data: Copyright 2021 by The New York Times Company 
Accessed on September 11th of 2022 at 1:05PM

import csv
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.dates as mdates
import numpy as np
from datetime import datetime

pd.options.mode.chained_assignment = None  # default='warn'

data = pd.read_csv("us-states.csv")

data

#data_transposed = data.T

data['date'] = pd.to_datetime(data['date']) ##This function converts a scalar DataFrame pandas datetime object.

## 
def plot_state_new_covid_cases(states):
    if(states is None):
        return 'Please enter a list of US States - at least one.'
    plt.figure(figsize=(25, 20))
    for state in states:
        one_state = data[data['state']==state] ## singling out unique state
        one_state['cases'] = one_state['cases'].diff() ##Calculates the difference of a DataFrame element compared with another element in the DataFrame (default is element in previous row).
        plt.plot(one_state['date'], one_state['cases'], label=state)
    ##formatting
    plt.legend()
    plt.title('New cases vs Date')
    plt.xlabel('date')
    plt.ylabel('daily new cases')
    ##display graph
    plt.show()
    
    
# Testing
example = ['California', 'Oregon', 'Washington'] ## comparing new covid cases on the West Coast States
plot_state_new_covid_cases(example)
example2 = ['Puerto Rico'] ##Also works with a single state/territory
plot_state_new_covid_cases(example2)


def max_cases_day(state):
    one_state = data[data['state']==state] ## single out state of interest
    one_state['cases'] = one_state['cases'].diff() ## make the cumulative cases column a new cases one
    record_date = data.loc[one_state['cases'].idxmax()]['date'] ##idxmax will give the index of the row with the highest (new)'cases' value
    print(f'In {state}, the record date for new COVID-19 cases was on {record_date}.')
    return record_date

max_cases_day('Puerto Rico')


def state_peak_time_comparer(state1, state2):
    record1 = datetime.strptime(max_cases_day(state1), '%Y-%m-%d')
    record2 = datetime.strptime(max_cases_day(state2), '%Y-%m-%d')
    delta = abs(record1 - record2) ##looking at absolute difference
    if record1 == record2:
        print(f'\n{state1} and {state2} had their peaks on the same day.')
    elif record1<record2:
        print(f'\n{state1} had its peak first, {delta.days} day(s) before {state2}.')
    else:
        print(f'\n{state2} had its peak first, {delta.days} day(s) before {state1}.')

state_peak_time_comparer('Hawaii', 'Florida') ##looking at the two states furthest apart

state_peak_time_comparer('Puerto Rico', 'Alaska') ##distant states in terms of latitudes

state_peak_time_comparer('Massachusetts', 'Connecticut') ##looking at states close to each other



******************
Exercise 4
******************
#IMPORTANT NOTICE
The desc2022 file is too heav to be pushed to GitHub, please download it to the directory you will be executing Exercise 4.ipynb in.

# importing element tree
import xml.etree.ElementTree as ET 

# Pass the path of the xml document 
tree = ET.parse('desc2022.xml') 

# get the parent tag 
root = tree.getroot() 

 

## Getting to Know the XML structure

root.tag

root.attrib

child=root[0]
print('\n')
print(child.tag, child.attrib)
print('\n')


for subchild in child:
    print(subchild.tag, subchild.attrib)

print('\n')
print(f'DescriptorName:  {child[1][0].text}')
print(f'First item of TreeNumberList:  {child[11][0].text}')


## Finding the DescriptorName using the DescriptorUI

def find_name_with_ui(ui):
    for child in root:## we iterate through all the root's children
        if child[0].text == ui: ## we know thanks to our code above that the first element of the child is the DescriptorUI
            return child[1][0].text ## we know the second element is the DescriptorName. 
            ## We have to go one step further in because of the nested String tag
    return 'No such DescriptorUI in XML file.'
        
        

print(find_name_with_ui('D007154'))

print(find_name_with_ui('D007184')) ## testing another ui
print(find_name_with_ui('D007888')) ## testing another ui
print(find_name_with_ui('D9999999999')) ## testing with erroneous input

## Finding the DescriptorUI using the DescriptorName

def find_ui_with_name(name):
    for child in root:## we iterate through all the root's children
        if child[1][0].text == name: ## we know the second element is the DescriptorName.
            return child[0].text   ## we know first element of the child is the DescriptorUI
            
    return 'No such DescriptorName in XML file.'

print(find_ui_with_name('Nervous System Diseases'))

print(find_ui_with_name('Incontinentia Pigmenti')) ## testing another ui
print(find_ui_with_name('Abdomen')) ## testing another ui
print(find_ui_with_name('Pop tarts addiction')) ## testing with bogus input

## Common parents

## we first write a function that finds the Treenumber given either a DescriptorName or DescriptorUI.
## We only admit one treenumber to simplify

def get_treeNumber_with_name_or_ui(heading_or_ui):
    ## if need be, we convert the parameter to DescriptorUI format 
    if not 'D0' in heading_or_ui:  ##all UIs begin with the letter D followed by the number 0
        heading_or_ui = find_ui_with_name(heading_or_ui)
    for child in root:
        if child[0].text == heading_or_ui:
            for concept in child.iter('TreeNumberList'):
                return concept[0].text
    return 'not found'


print(get_treeNumber_with_name_or_ui('Nervous System Diseases'))
print(get_treeNumber_with_name_or_ui('D007154'))
print(get_treeNumber_with_name_or_ui('Abdomen'))



def common_parents(parent1, parent2): ## the parents can only be in DescriptorName or DescriptorUI Format
    
    ## from the Name/UI, get the tree name
    parentTree1 = get_treeNumber_with_name_or_ui(parent1) + '.'
    parentTree2 = get_treeNumber_with_name_or_ui(parent2) + '.'
    
    result = set()  ##initializing an empty set

    for child in root:
        for concept in child.iter('TreeNumberList'): ## only looking at the TreeNumberList of each entry
            for treeNumber in concept:   ## iterating through each TreeNumber in that list
                if parentTree1 in treeNumber.text:
                    for treeNumber in concept:
                        if parentTree2 in treeNumber.text:
                            result.add(child[1][0].text) ## data structure set prevents duplicates automatically
    return result

print(common_parents('Nervous System Diseases', 'D007154'))

The above result lists all the medical conditions which are both Nervous system diseases and Immune System diseases: in short, immune system diseases of the nervous system. For instance the Guillain-Barre Syndrome (GBS) "is a rare neurological disorder in which the body's immune system mistakenly attacks part of its peripheral nervous system" (NINDS definition)

## Testing other input
print(common_parents('D001187', 'Prostheses and Implants')) 
## what are artificial organs (D001187) which are also Prostheses/Implants?


