# Animal Intakes And Outcomes In A Center Analysis

## Introduction
In this project, we will import data into Python and clean it properly. After that, we will analyze the data and answer all the stakeholder's questions to gain insights about the animals in the center.

## Explain Dataset
#### Acc_Intakes.csv 
- Age Upon Intake: The age of the animal when it was brought to the facility
- Animal ID: A unique identifier for each animal.
- Animal Type: The species of the animal. In both examples, this is 'Dog'.
- Breed: The breed of the animal.
- Color: The color of the animal's coat.
- Datetime: The date and time when the animal was intake.
- Datetime2: This seems to be a repeat of the intake date and time
- Found Location: The location where the animal was found before being brought to the facility.
- Intake Condition: The health or physical condition of the animal at intake.
- Intake Type: The circumstances under which the animal was brought to the facility.
- Name: The name given to the animal. 
- Sex Upon Intake: The gender and reproductive status of the animal when it arrived.

#### Acc_Outcomes.csv
- Age Upon Outcome: This is the age of the animal at the time they left the facility
- Animal ID: A unique identifier assigned to each animal.
- Animal Type: The species of the animal.
- Breed: The breed of the animal.
- Color: The color of the animal's coat.
- Date of Birth: The birth date of the animal. 
- Datetime: The date and time when the animal left the facility.
- Monthyear: This seems to be a repeat of the outcome date and time
- Name: The name of the animal
- Outcome Subtype: Additional details about the outcome.
- Outcome Type: The general category of the outcome.
- Sex Upon Outcome: The gender and reproductive status of the animal when it left the facility.

## Clearn data before analyze.
#### Import pandas and numpy library
```python
import pandas as pd 
import numpy as np
```
#### Read file aac_intakes.csv and save as 'aac_intakes' dataframe
```python
# Read file aac_intakes.csv
aac_intakes = pd.read_csv('aac_intakes.csv')
aac_intakes.sort_values('animal_id')
```
#### Read file aac_outcomes.csv and save as 'aac_outcomes' dataframe
```python
# Read file aac_outcomes.csv
aac_outcomes = pd.read_csv('aac_outcomes.csv')
aac_outcomes.sort_values('animal_id')
```
As you can see in my dataset, the ID column has many duplicated values, 
#### Step 1: Convert data type 
#### Step 2: Copy old dataframes and clean the duplicate column 'animal_id' both new dataframes
```python
####Clearn the data####

# Convert object type to datetime64[ns] type in column datetime
aac_intakes[['datetime','datetime2']] = aac_intakes[['datetime','datetime2']].apply(pd.to_datetime)
aac_outcomes[['date_of_birth','datetime','monthyear']] = aac_outcomes[['date_of_birth','datetime','monthyear']].apply(pd.to_datetime)

# Clearn the duplicate row and nan in data
aac_intakes = aac_intakes.drop_duplicates()
aac_intakes = aac_intakes.dropna(subset = ['animal_id', 'datetime','datetime2'])

aac_outcomes = aac_outcomes.drop_duplicates()
aac_outcomes = aac_outcomes.dropna(subset = ['animal_id', 'date_of_birth','datetime','monthyear'])

# Select all duplicate values in column 'animal_id' in 2 file
ids_in = aac_intakes.animal_id
ids_out = aac_outcomes.animal_id

Duprows_aac_intakes = aac_intakes[ids_in.isin(ids_in[ids_in.duplicated()])].sort_values("animal_id")
Duprows_aac_outcomes = aac_outcomes[ids_out.isin(ids_out[ids_out.duplicated()])].sort_values("animal_id")

# The last time animal had add there information in data
idx_in = Duprows_aac_intakes.groupby('animal_id')['datetime'].idxmax()
new_idx_in = Duprows_aac_intakes.loc[idx_in]

idx_out = Duprows_aac_outcomes.groupby('animal_id')['datetime'].idxmax()
new_idx_out = Duprows_aac_outcomes.loc[idx_out]

# Delete all duplicate values in column 'animal_id' in 2 file
nonDup_aac_intakes = aac_intakes.drop_duplicates(subset=["animal_id"], keep=False)
nonDup_aac_outcomes = aac_outcomes.drop_duplicates(subset=["animal_id"], keep=False)
```
#### Step 3: Combine 2 data from datafarme not duplicate and dataframe was update by 'datetime'
```python
#Combine the data
Update_aac_intakes = pd.concat([nonDup_aac_intakes, new_idx_in])
Update_aac_outcomes = pd.concat([nonDup_aac_outcomes, new_idx_out])
```
```python
print('New dataset after clearing step in dataframe acc_intakes.csv is',aac_intakes.shape[0], 'rows')
print('New dataset after clearing step in dataframe acc_outcomes.csv is',aac_outcomes.shape[0], 'rows')
print('Dataframe acc_intakes.csv after update have',Update_aac_intakes.shape[0], 'rows')
print('Dataframe acc_outcomes.csv after update have',Update_aac_outcomes.shape[0], 'rows')
```
## Answers question
### Question number 1:
I find the most location where pet are found in aac_intakes data history record by sort_values() method and head() to select top 5
```python
'''1.Is there an area where more pets are found?'''
print(aac_intakes.found_location.value_counts().sort_values(ascending=[False]).head(5))

aac_intakes.groupby(by = 'found_location').count().sort_values(by = 'animal_id').tail().animal_id
```
### Question number 2:
First, I select rows data in aac_intakes dataframe to find the year record in 2015
Then caculated the highest number by groupby(), count(), sort_values() and max() method to find the month name
and find the highest number of animals had found in 2015
```python
'''2.What is the average number of pets found in a month in the year 2015? 
Are there months where there is a higher number of animals found?'''

# Find the animal had found in 2015
found_2015 = aac_intakes[(aac_intakes['datetime'] > "2015-01-01") & (aac_intakes['datetime'] < "2015-12-31")]

# Find the highest number of animals had found in month in 2015 
Max_month_found_in_2015 = found_2015.groupby(found_2015.datetime.dt.month)['animal_id'].count().sort_values().max()

# The month had highest number of animals had found in 2015
month = found_2015.groupby(found_2015.datetime.dt.month_name())['animal_id'].count().sort_values(ascending=False).index[0]

print('The average number of pets found in every month in 2015: ', round(found_2015.shape[0]/12 ,2))
print(month, 'is the highest number of animals had found with: ', Max_month_found_in_2015)
```
### Question number 3:
First, I caculated the number of animal had 'Adoption' values in 'outcome_type' 
columns in update dataframe 'Update_aac_outcomes'
Then I caculated the number of animal had take care in shelter in update data 'Update_aac_outcomes'
and caculated Ratio of incoming pets vs. adopted pets 
```python
'''3.What is the ratio of incoming pets vs. adopted pets?'''
# Check the number of animal had adoption in aac_outcomes
Adoption_aac_outcomes = Update_aac_outcomes[Update_aac_outcomes['outcome_type'] == 'Adoption']
number_Adoption = Adoption_aac_outcomes.animal_id.count()
print('Number of pets that had adoption in data is',number_Adoption)

#Check the number of animal incoming in aac_intakes
number_aac_intakes = Update_aac_outcomes.shape[0]
number_aac_intakes

# Ratio of incoming pets vs. adopted pets
Ratio = 100/number_aac_intakes * number_Adoption
print('The ratio of incoming pets vs. adopted pets is: ' + str(Ratio.round(3)) + '%')
```
### Question number 4:
I use groupby() and count() method and select 'animal_id' columns in update data 'Update_aac_intakes' to show the resault
```python
'''4. What is the distribution of the types of animals in the shelter?'''
Update_aac_intakes.groupby(Update_aac_intakes.intake_type).animal_id.count()
```
### Question number 5:
First, I need to find the top 5 breed's name by 'animal_type' = 'Dog'
Then I find the number of adoption base these name
Finally, I caculate the percentage of each breed and show the resault
```python
'''5. What are the adoption rates for specific breeds?
Find the top 5 dog breeds in the shelter (based on count) and then find the adoption percentage of each breed.'''

## Step 1 ##
# Find rows have 'Dog' breed in shelter
breed_Dog_Shelter = Update_aac_intakes[Update_aac_intakes['animal_type'] == 'Dog']
# Find the top 5 'Dog' breed in shelter
top5_breedDog_Shelter = breed_Dog_Shelter.groupby(breed_Dog_Shelter['breed']).animal_id.count().sort_values(ascending=[False]).head(5)

## Step 2 ##
# Find the 'Dog' breed and 'Adoption' outcome_type in Update_aac_outcomes
Dog_outcomes_Adoption = Update_aac_outcomes[(Update_aac_outcomes['animal_type'] == 'Dog') & (Update_aac_outcomes['outcome_type'] == 'Adoption')]
# Group data by 'breed' and count number 
breedDog_outcomes = Dog_outcomes_Adoption.groupby(Dog_outcomes_Adoption['breed']).animal_id.count()

## Step 3 ##
# List values for top 5 dogs in shelter
List1_number = []
for data_row in range(top5_breedDog_Shelter.shape[0]):
    values_data_row = top5_breedDog_Shelter.iloc[data_row]
    List1_number.append(values_data_row)
    
# List values find the number of top 5 dogs in shelter had record in breedDog_outcomes
List2_number = []
for breed in top5_breedDog_Shelter.index:
    data_row = breedDog_outcomes[breedDog_outcomes.index == str(breed)]
    values_data_row = data_row.iloc[0]
    List2_number.append(values_data_row)
    
# List for percentage of top 5 breed dog had adopted in shelter
List3_number = []  
for i in range(top5_breedDog_Shelter.shape[0]):
    number = round(100 / List1_number[i] * List2_number[i],2)
    List3_number.append(number)
    
# List for index use breed's dog data
List_Index = []
for i in top5_breedDog_Shelter.index:
    List_Index.append(i)

# Show the resault by create dataframe by 3 lists
display_Top5_percentage_breedDog = pd.DataFrame(list(zip(List_Index, List1_number, List2_number, List3_number)),
               columns =['Breed_Dog', 'Number_intakes', 'Number_adopted', 'Percentage'])
display_Top5_percentage_breedDog
```
### Question number 6:
Similar to number 5 but instead of find breed dog's name, I find the color's name
```python
'''6. What are the adoption rates for different colorings?
Find the top 5 colorings in the shelter (based on count) and then find the adoption percentage of each color.'''

## Step 1 ##
# Find rows have 'Dog' breed in shelter
breed_Dog_Shelter = Update_aac_intakes[Update_aac_intakes['animal_type'] == 'Dog']
# Find the top 5 'Dog' breed in shelter
top5_colors_Dog_Shelter = breed_Dog_Shelter.groupby(breed_Dog_Shelter['color']).animal_id.count().sort_values(ascending=[False]).head(5)
top5_colors_Dog_Shelter

## Step 2 ##
# Find the 'Dog' breed and 'Adoption' outcome_type in Update_aac_outcomes
Dog_outcomes_Adoption = Update_aac_outcomes[(Update_aac_outcomes['animal_type'] == 'Dog') & (Update_aac_outcomes['outcome_type'] == 'Adoption')]
# Group data by 'color' and count number 
colors_Dog_outcomes = Dog_outcomes_Adoption.groupby(Dog_outcomes_Adoption['color']).animal_id.count()
colors_Dog_outcomes

## Step 3 ##
# List values for top 5 color dog's in shelter
List1_number = []
for values in range(top5_colors_Dog_Shelter.shape[0]):
    values_data_row = top5_colors_Dog_Shelter.iloc[values]
    List1_number.append(values_data_row)

# List values find the number of top 5 dogs in shelter had record in breedDog_outcomes
List2_number = []
for values in top5_colors_Dog_Shelter.index:
    data_row = colors_Dog_outcomes[colors_Dog_outcomes.index == str(values)]
    values_data_row = data_row.iloc[0]
    List2_number.append(values_data_row)
    
# List for percentage of top 5 breed dog had adopted in shelter
List3_number = []  
for i in range(top5_colors_Dog_Shelter.shape[0]):
    number = round(100 / List1_number[i] * List2_number[i],2)
    List3_number.append(number)
    
# List for index use breed's dog data
List_Index = []
for i in top5_colors_Dog_Shelter.index:
    List_Index.append(i)
    
# Show the resault by create dataframe by 3 lists
display_Top5_percentage_colorsDog = pd.DataFrame(list(zip(List_Index, List1_number, List2_number, List3_number)),
               columns =['Color_dog', 'Number_intakes', 'Number_adopted', 'Percentage'])
display_Top5_percentage_colorsDog
```
### Question number 7.
First, I silce the update data for 'Update_aac_intakes' with 'animal_id', 'sex_upon_intake'
and 'Update_aac_outcomes' with 'animal_id', 'sex_upon_outcome','datetime' columns
Then, I use merge() method to find the intersection data to filter data animal had not spayed/neutered
and Find the animal have spayed/neutered by staff in shelter
Finally, I caculated the time with data had recorded, convert them to month number and caculated 
the average munber of animal had spayed/neutered by staff
```python
'''7. About how many animals are spayed/neutered each month?
This will help the shelter allocate resources and staff. Assume that all intact males and females will be spayed/neutered.'''
##Step 1: Check the first time and the last time that animal's information had record
# Step 1.1 Slice small data to see the 'sex_upon_intake' and 'sex_upon_outcome' column
short_intake = aac_intakes[['animal_id', 'sex_upon_intake']]
short_outcome = aac_outcomes[['animal_id', 'sex_upon_outcome','datetime']]
# Step 1.2 Drop duplicate values in the fisrt time that animal's information had record
new_short_intake = short_intake.drop_duplicates(subset = ['animal_id'])
# Step 1.3 Select the last date that animal's information had record
new_short_outcome = short_outcome.sort_values(by = 'datetime')
new_short_outcome = new_short_outcome.drop_duplicates(subset = ['animal_id'], keep = 'last')

## Step 2: Intersection 2 dataframe to see the each column values
# Use 'pd.merge' and give parameters 'how ='inner'' 
# to return only the rows in which the left table have matching keys values column in the right table
# and 'on' by column you choose
intersection_data = pd.merge(new_short_intake, new_short_outcome, how ='inner', on =['animal_id'])

## Step 3: Find the animal have not spayed/neutered before
intake_no_sp_ne = intersection_data[(intersection_data.sex_upon_intake != 'Spayed Female')
                     | (intersection_data.sex_upon_intake != 'Neutered Male')]

## Step 4: Find the animal have spayed/neutered after leave in shelter
outcome_sp_ne = intake_no_sp_ne[(intake_no_sp_ne.sex_upon_outcome == 'Neutered Male') 
          | (intake_no_sp_ne.sex_upon_outcome == 'Spayed Female')]

## Step 5: Find the animal have spayed/neutered by staff in shelter
filter_data = outcome_sp_ne[((outcome_sp_ne.sex_upon_intake != 'Neutered Male') 
                    & (outcome_sp_ne.sex_upon_outcome == 'Neutered Male')) 
                | ((outcome_sp_ne.sex_upon_intake != 'Spayed Female') 
                    & (outcome_sp_ne.sex_upon_outcome == 'Spayed Female'))]

## Step 6: Find the first time record in data 
# Use .min() method
first_time_record = filter_data['datetime'].min()

## Step 7: Find the last time record in data
# Use .max() method
last_time_record = filter_data['datetime'].max()

## Step 8: Total times had record in data by month
# Use '.np.timedelta64' calculate datetime data type
month_number_recrd = round((last_time_record - first_time_record)/np.timedelta64(1, 'M'),0)

## Step 9: Convert Timedelta to month int
month_number = (last_time_record - first_time_record)/np.timedelta64(1, 'M')

## Step 10: Caculated the average munber of animal had spayed/neutered by staff
print('There are',round(filter_data.shape[0] / month_number,2), 'pets had neutered/spayed each month')
```
## Contact Information
For further information, collaboration, or inquiries, feel free to contact [Phil Dinh](mailto:dinhthanhtrung2011@gmail.com).
