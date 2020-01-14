# Analysis of the StackOverFlow 2018 survey  

Part 1 : Objective <br>
Part 2 : Steps<br> 
Part 3 : Results<br>


### **Part 1 : Objective**   <br>
Analysis of the current software market with data from the **_StackOverFlow 2018 survey_**, with a focus on **data science** positions in the US. <br> <br>
We will base our data mainly on seeking the necessary skill sets, average and net salary, gender focused analysis, and other soft skills necessary to succeed in the market. 
Other conditions we query by are that respondents must not be students and must be employed full time and salary entry must not be empty.

---
### **Part 2 : Steps**   <br> 
1. Since we're attempting to practice our SQL skills along the way, we've also implemented our database into an SQLite database. 

_Creating the SQL query to create the table_
``` python
main_table = "stack_overflow_survey"
statement = 'create table if not exists ' + main_table  + ' (' 
for column in df.columns.values:
    all_columns.append(column)
    column_data_type = determine_data_type_of_list(df[column].tolist()) 
    if column_data_type == str: 
        statement = (statement + '\n{} text,').format(column.lower() )
    elif column == "Respondent":
        
        statement = (statement + '\n{} int not null primary key,').format(column.lower() )
    else:
        statement = (statement + '\n' + '{} {}' + ',').format(column.lower(), "int") 
statement = statement[:-1] + ');' 
```
_Creating the tables_
```python
#CREATE TABLES
create_table_sql = statement
sql_table_name = "Stack_Over_Flow_Survey.db"
conn = create_connection(sql_table_name) 
create_table(conn , create_table_sql)
```

_Inserting the values to "stack_overflow_survey"_
```python 
insert_sql = """INSERT INTO stack_overflow_survey (respondent, hobby, opensource, country, student, employment, formaleducation, undergradmajor, companysize, devtype, yearscoding, yearscodingprof, jobsatisfaction, careersatisfaction, hopefiveyears, jobsearchstatus, lastnewjob, assessjob1, assessjob2, assessjob3, assessjob4, assessjob5, assessjob6, assessjob7, assessjob8, assessjob9, assessjob10, assessbenefits1, assessbenefits2, assessbenefits3, assessbenefits4, assessbenefits5, assessbenefits6, assessbenefits7, assessbenefits8, assessbenefits9, assessbenefits10, assessbenefits11, jobcontactpriorities1, jobcontactpriorities2, jobcontactpriorities3, jobcontactpriorities4, jobcontactpriorities5, jobemailpriorities1, jobemailpriorities2, jobemailpriorities3, jobemailpriorities4, jobemailpriorities5, jobemailpriorities6, jobemailpriorities7, updatecv, currency, salary, salarytype, convertedsalary, currencysymbol, communicationtools, timefullyproductive, educationtypes, selftaughttypes, timeafterbootcamp, hackathonreasons, agreedisagree1, agreedisagree2, agreedisagree3, languageworkedwith, languagedesirenextyear, databaseworkedwith, databasedesirenextyear, platformworkedwith, platformdesirenextyear, frameworkworkedwith, frameworkdesirenextyear, ide, operatingsystem, numbermonitors, methodology, versioncontrol, checkincode, adblocker, adblockerdisable, adblockerreasons, adsagreedisagree1, adsagreedisagree2, adsagreedisagree3, adsactions, adspriorities1, adspriorities2, adspriorities3, adspriorities4, adspriorities5, adspriorities6, adspriorities7, aidangerous, aiinteresting, airesponsible, aifuture, ethicschoice, ethicsreport, ethicsresponsible, ethicalimplications, stackoverflowrecommend, stackoverflowvisit, stackoverflowhasaccount, stackoverflowparticipate, stackoverflowjobs, stackoverflowdevstory, stackoverflowjobsrecommend, stackoverflowconsidermember, hypotheticaltools1, hypotheticaltools2, hypotheticaltools3, hypotheticaltools4, hypotheticaltools5, waketime, hourscomputer, hoursoutside, skipmeals, ergonomicdevices, exercise, gender, sexualorientation, educationparents, raceethnicity, age, dependents, militaryus, surveytoolong, surveyeasy) 
VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
""" 
################ INSERT FUNCTION###############
def insert_rows_to_db(conn, data_tuple): 
    try:
        c = conn.cursor()
        c.executemany(insert_sql, data_tuple)
        conn.commit()
    except Error as e:  
        print(e)

################ EXTRACT DATA###############
index_print_times = 0
test_times = 30
all_respondet_answer_tuple = []
for index, value in  df.iterrows(): 
    respondent_answer_tuple = [] 
    for columns in all_columns: 
        respondent_answer_tuple.append(value[columns])
        index_print_times += 1  
#     insert_rows_to_db(conn, tuple(respondent_answer_tuple))
    all_respondet_answer_tuple.append(tuple(respondent_answer_tuple)) 
    
############### IF ERROR, DELETE .DB FILE THEN RERUN #########################

```
 > ###### Notice the `executemany` function within the `insert_rows_to_db` function. This is essential in shortening the time required to run the whole inserting  action. It will take a third of the time in our case than inserting each row one by one through reducing the I/O counts required.

  

2. We will first analyze our audience(the survey takers), the impact that different groups of survey takers makes will help us differentiate how age/experience affects our skill sets and necessity to improve to the next step in our career. 

``` python 
select_sql = "SELECT ? "
select_from_table = "FROM " + main_table + " WHERE country = 'United States' AND Student = 'No' AND employment = 'Employed full-time' AND convertedsalary is not null"
select_columns = ["Respondent", "Hobby" , "OpenSource" , "Country" , "Student" , "Employment" , 
                  "FormalEducation" , "UndergradMajor" , "CompanySize" , "DevType" , "YearsCoding" ,
                  "YearsCodingProf" , "JobSatisfaction" , "CareerSatisfaction" , "HopeFiveYears" , "JobSearchStatus" , 
                  "LastNewJob" ,"Age" , "MilitaryUS" , "RaceEthnicity" , "EducationParents" , "Gender" ,"WakeTime" ,
                  "HoursComputer" , "HoursOutside" , "SkipMeals" , "ErgonomicDevices", "convertedsalary"]
```
Notice we specified <br> `country = 'United States' AND Student = 'No' AND employment = 'Employed full-time' AND convertedsalary is not null` <br>
in our sql query
this is essential in the process since theres alot of respondents that dont match what we're looking for, with this although very specific limitations, it gives us a clearer idea of what we're looking at as a whole.

``` python 
select_column_whole = " "
for column in select_columns:
    select_column_whole = select_column_whole + column + " ,"
select_column_whole = select_column_whole[:-1] 
select_sql = select_sql.replace("?" , select_column_whole, 1) + select_from_table 
# print(select_sql)
c = conn.cursor()
c.execute(select_sql)
conn.commit()

select_respondent_reponse = c.fetchall()
selected_response = pd.DataFrame(select_respondent_reponse , columns = select_columns)
selected_response = selected_response.set_index('Respondent')
```
try printing `selected_response` to get an idea of the data we have thus far, it's essential we understand the data we're processing.

Next, out of curiosity, we check the countries participating in the survey and the top contributors
``` python 
country_responses = set(selected_response["Country"])
total_appearance_of_each_country = Counter(selected_response["Country"])
```

``` 
Counter({'United States': 20309, 'India': 13721, 'Germany': 6459, 'United Kingdom': 6221, 'Canada': 3393, 'Russian Federation': 2869, 'France': 2572, 'Brazil': 2505, 'Poland': 2122, 'Australia': 2018, 'Netherlands': 1841, 'Spain': 1769, 'Italy': 1535, 'Ukraine': 1279, 'Sweden': 1164, 'Pakistan': 1050, 'China': 1037, 'Switzerland': 1010, 'Turkey': 1004, 'Israel': 1003, 'Iran, Islamic Republic of...': 921, 'Romania': 793, 'Austria': 788, 'Czech Republic': 784, 'Belgium': 743, 'Mexico': 736, 'Bangladesh': 697, 'Denmark': 653, 'South Africa': 637, 'Indonesia': 630, 'Argentina': 611, 'Norway': 565, 'New Zealand': 557, 'Ireland': 554, 'Portugal': 528, 'Finland': 521, 'Philippines': 520, 'Greece': 516, 'Hungary': 470, 'Sri Lanka': 454, 'Bulgaria': 425, 'Egypt': 419, None: 412, 'Nigeria': 399, 'Singapore': 376, 'Malaysia': 363, 'Japan': 361, 'Serbia': 358, 'Colombia': 339, 'Belarus': 339, 'Viet Nam': 331, 'Nepal': 295, 'Lithuania': 257, 'Croatia': 241, 'Chile': 238, 'Slovakia': 238, 'Slovenia': 238, 'Hong Kong (S.A.R.)': 219, 'Thailand': 213, 'Morocco': 213, 'Taiwan': 207, 'Kenya': 194, 'United Arab Emirates': 193, 'Estonia': 189, 'South Korea': 169, 'Tunisia': 163, 'Latvia': 145, 'Algeria': 130, 'Saudi Arabia': 130, 'Peru': 128, 'Bosnia and Herzegovina': 125, 'Venezuela, Bolivarian Republic of...': 123, 'Armenia': 117, 'Dominican Republic': 115, 'Albania': 109, 'Lebanon': 107, 'Kazakhstan': 107, 'Uruguay': 102, 'Costa Rica': 98, 'Other Country (Not Listed Above)': 84, 'Jordan': 83, 'Azerbaijan': 78, 'Ghana': 76, 'Republic of Moldova': 76, 'Georgia': 75, 'Uganda': 67, 'Malta': 66, 'Cuba': 65, 'Ecuador': 65, 'Afghanistan': 64, 'Republic of Korea': 62, 'Cambodia': 60, 'Ethiopia': 60, 'Luxembourg': 59, 'Uzbekistan': 56, 'Syrian Arab Republic': 56, 'Myanmar': 55, 'The former Yugoslav Republic of Macedonia': 54, 'Guatemala': 50, 'Paraguay': 49, 'United Republic of Tanzania': 49, 'Cyprus': 49, 'El Salvador': 46, 'Iceland': 45, 'Bolivia': 44, 'Iraq': 42, 'Zimbabwe': 39, 'Mauritius': 35, 'Cameroon': 34, 'Panama': 32, 'Nicaragua': 31, 'Honduras': 25, 'Sudan': 25, 'Qatar': 23, 'Kuwait': 23, 'Kyrgyzstan': 22, 'Mozambique': 21, 'Mongolia': 21, 'Oman': 20, 'Jamaica': 20, 'Trinidad and Tobago': 20, 'Madagascar': 19, 'Rwanda': 18, 'Bahrain': 16, 'Montenegro': 15, 'Andorra': 15, 'Libyan Arab Jamahiriya': 14, 'Yemen': 13, 'Senegal': 13, 'Angola': 11, 'Somalia': 10, 'Maldives': 10, 'Benin': 10, 'Antigua and Barbuda': 9, 'Malawi': 9, 'Zambia': 9, 'Fiji': 8, 'Haiti': 8, 'Democratic Republic of the Congo': 7, "Côte d'Ivoire": 7, 'Bhutan': 6, 'Turkmenistan': 6, 'Namibia': 6, 'Bahamas': 6, 'Congo, Republic of the...': 6, 'Liechtenstein': 5, 'Gabon': 5, 'Gambia': 5, 'Togo': 5, 'Burkina Faso': 5, 'Botswana': 4, 'Barbados': 4, 'Tajikistan': 4, 'Monaco': 4, 'North Korea': 4, 'Swaziland': 3, 'Guinea': 3, 'Mauritania': 3, 'Liberia': 3, 'Cape Verde': 3, 'Micronesia, Federated States of...': 2, 'Lesotho': 2, 'Marshall Islands': 2, 'Central African Republic': 2, 'Palau': 2, 'Suriname': 2, 'Dominica': 2, 'Niger': 2, 'Guyana': 2, "Democratic People's Republic of Korea": 2, 'San Marino': 1, 'Burundi': 1, 'Sierra Leone': 1, 'Grenada': 1, 'Belize': 1, 'Timor-Leste': 1, 'Solomon Islands': 1, 'Saint Lucia': 1, 'Nauru': 1, 'Mali': 1, 'Brunei Darussalam': 1, 'Eritrea': 1, 'Djibouti': 1, 'Guinea-Bissau': 1})
```
We can see that the top contributing countries are as follow in decreasing order.
```
['United States', 'India', 'Germany', 'United Kingdom', 'Canada', 'Russian Federation', 'France', 'Brazil', 'Poland', 'Australia', 'Netherlands', 'Spain', 'Italy', 'Ukraine', 'Sweden', 'Pakistan', 'China', 'Switzerland', 'Turkey', 'Israel']
```
It's clear that the countries with english as it's main language are the highest contributors, seeing that the resources provided on StackOverFlow are mostly english.
We can also easily see that China, with the highest population, are slightly lower on the chart, this is due an abundance of resources in mandarin therefore they have their own source of question inquiry. 

3. We will also likely figure out how the market reacts to a new candidate for certain jobs for either a male and a female. 
```python 
# All Gender type options
gender_responses = selected_response["Gender"]
# print(gender_responses)
total_gender_responses = []
for i in gender_responses: 
    if i != None:
        i = i.split(";") 
        if type(i) == list:
            for q in i :
                total_gender_responses.append(q)
        else:
            total_gender_responses.append(i)
        
        
set_gender_response = set(total_gender_responses)
print("All Gender type options \n" , set_gender_response) 
```
The gender types listed as an option in the survey are as follow :
```
'Non-binary, genderqueer, or gender non-conforming', 'Female', 'Transgender', 'Male'
```
Since non-binary genders are an extreme minority, we classify them as a whole under the `Others` category with all due respect. 

```python
# Count respondents from all categories
gender_dict = {}
gender_dict["Male"] = 0
gender_dict["Female"] = 0 
gender_dict["Others"] = 0
gender_dict["Undisclosed"] = 0
for i in selected_response["Gender"]:
#     print(i) 
    if i is None: 
        gender_dict["Undisclosed"] +=1 
    elif "Male" in i:
        gender_dict["Male"] += 1
    elif "Female" in i :
        gender_dict["Female"] +=1 
    else:
        gender_dict["Others"] +=1  

gender_count = []
gender_count.append(gender_dict["Male"])
gender_count.append(gender_dict["Female"])
# gender_count.append(gender_dict["Undisclosed"])
# gender_count.append(gender_dict["Others"])
gender_label = "Male" , "Female" # , "Undisclosed" , "Others"

# GRAPHING GENDER
figureObject, axesObject = plt.subplots() 

plt.pie(gender_count,
        labels=gender_label, 
        autopct='%1.2f', 
        startangle=90) 

plt.axis('equal')

fig = plt.gcf()
fig.set_size_inches(18.5, 10.5)

plt.savefig("./Figure/GenderPlot.png")

plt.show()
```
We can see that in the market are male dominated with an 91%.

``` python  
#Gender to salary plot
plt.figure(figsize=(10,6)) 
temp = selected_response[selected_response.DevType == 'Data scientist or machine learning specialist']
temp['convertedsalary'] = temp['convertedsalary'].apply(lambda x: float(x))
temp = temp[temp.convertedsalary < 250000]
ax = sns.boxplot(x="Gender", 
                 y="convertedsalary", 
                 hue="Gender", 
                 data=temp)
```
 With this chart we see that the median of salary in relevance to gender of male and female has very minor difference, however the range of male salary is broader and the female salary is more compacted.


4. We follow up by analyzing the age group 
```python 
age_responses = selected_response["Age"]
age_responses.unique()
age_responses = selected_response.groupby("Age").nunique()
age_responses = selected_response.Age.value_counts()
age_label = []
age_value = []
for i in age_responses:
#     print*()
    age_label.append(age_responses[age_responses == i].index[0])
    age_value.append(i)
fig1, ax1 = plt.subplots()
ax1.pie(age_value, labels=age_label, autopct='%1.1f%%', shadow=True)
ax1.axis('equal')
fig = plt.gcf()
fig.set_size_inches(18.5, 10.5)
plt.savefig("./Figure/age_pie")
plt.show()
```
We can easily see that the the age range of 25~34 is the most populated at 52%, with the age group of 35~44 at 24% and finally 18~24 at 12%

``` python
plt.figure(figsize=(10,6))
temp = selected_response[selected_response.DevType == 'Data scientist or machine learning specialist']
temp['convertedsalary'] = temp['convertedsalary'].apply(lambda x: float(x))
temp = temp[temp.convertedsalary < 250000]
ax = sns.boxplot(x="Age", 
                 y="convertedsalary", 
                 hue="Age", 
                 data=temp)
plt.savefig("./Figure/boxplot age group.png")
```
In this plot we see that although age does not significantly affect median salary, it's easily inferred that salary does increase as age increases.

5. To follow, we always wonder how much does contribution to opensource affects our chances

```python 
open_source_response = selected_response["OpenSource"]
selected_response.convertedsalary = pd.to_numeric(selected_response.convertedsalary)
selected_response.info()
temp = selected_response[selected_response.DevType == 'Data scientist or machine learning specialist']
temp = temp[temp.convertedsalary < 250000]
ax = sns.boxplot(x="OpenSource", 
                 y="convertedsalary", 
                 hue="OpenSource", 
                 data=temp)
```

```python 
open_source_count_unique = selected_response.OpenSource.value_counts()
open_source_label = []
open_source_count_total = []
for i in open_source_count_unique: 
    open_source_label.append(open_source_count_unique[open_source_count_unique == i ].index[0])
    open_source_count_total.append(i)

```
Only 44.2% of respondents contribute to opensource

6. Next we focus on jobtypes
``` python 
def query_year_and_dev():
    query_sql = "SELECT YearsCoding, DevType from " + main_table + " WHERE DevType is not null"
    c = conn.cursor()
    c.execute(query_sql)
    conn.commit()
    rows  = c.fetchall()
    return rows
years_coding_and_dev_type = pd.DataFrame(query_year_and_dev(), columns = ["YearsCoding" , "DevType"]) 
# Split options and find specific options
def find_set_dev_type():
    unique_list = []
    null_list_dev_type = []
    for i in years_coding_and_dev_type.DevType:
        if i is not None:
            split_list = i.split(";") 
            for i in split_list:
                unique_list.append(i)
        else :
            null_list_dev_type.append(i) 
    return set(unique_list)
find_set_dev_type()
```
We see that the options for job types are as follow
```
'Back-end developer',
 'C-suite executive (CEO, CTO, etc.)',
 'Data or business analyst',
 'Data scientist or machine learning specialist',
 'Database administrator',
 'Designer',
 'Desktop or enterprise applications developer',
 'DevOps specialist',
 'Educator or academic researcher',
 'Embedded applications or devices developer',
 'Engineering manager',
 'Front-end developer',
 'Full-stack developer',
 'Game or graphics developer',
 'Marketing or sales professional',
 'Mobile developer',
 'Product manager',
 'QA or test developer',
 'Student',
 'System administrator'
```
Since we're focused on finding data science position relative information, we look at `'Data or business analyst',
 'Data scientist or machine learning specialist'` primarily
 We also categorize software engineers as `'Back-end developer',  
 'Database administrator', 
 'Desktop or enterprise applications developer',
 'DevOps specialist', 
 'Embedded applications or devices developer',
 'Engineering manager',
 'Front-end developer',
 'Full-stack developer',
 'Game or graphics developer', 
 'Mobile developer', 
 'QA or test developer', 
 'System administrator'`

 ```python 
 def find_year_and_dev_by_DevType():
    found_data_scientist = []
    coding_years_set = []
    for individual_position_block in years_coding_and_dev_type.values:
        
        this_man_is_a_data_scientist = False
        if individual_position_block[1] is not None:
            individual_position_block_list = individual_position_block[1].split(";")
            for indi_positions in individual_position_block_list:
                if indi_positions in data_science_position_titles:
                    this_man_is_a_data_scientist = True
        if this_man_is_a_data_scientist == True:
            found_data_scientist.append(individual_position_block)
            coding_years_set.append(individual_position_block[0])
    coding_years_set = set(coding_years_set)
    return found_data_scientist, coding_years_set


found_years_devtype_ds , coding_years_set = find_year_and_dev_by_DevType() 
```

```python
def count_total_ds_in_age_group():
    total_ds_in_age_group = {}
    for i in range(0 , len(found_years_devtype_ds)): 
        try:
            total_ds_in_age_group[found_years_devtype_ds[i][0]] += 1
        except:
            total_ds_in_age_group[found_years_devtype_ds[i][0]] = 1
    return total_ds_in_age_group
total_ds_in_age_group = count_total_ds_in_age_group()
# Sort in Age Order
total_ds_in_age_group = { '0-2 years': 1071, '3-5 years': 2798,'6-8 years': 2482,'9-11 years': 1553, '12-14 years': 1071,  '15-17 years': 887,  '18-20 years': 785,'24-26 years': 334, '21-23 years': 425, '27-29 years': 174, '30 or more years': 721}
```

```python
plt.bar(total_ds_in_age_group.keys(), total_ds_in_age_group.values(), align='center', alpha=0.5)
plt.ylabel('Number of Data Scientist Respondents')
plt.title('Data Scientist Age Groups')
fig = plt.gcf()
fig.set_size_inches(20, 10) 

plt.savefig("./Figure/Age_DataScientist.png")
plt.show()
```
The one information that immediately pops is how developers/scientists with 0~2 years of experience have close to no chance of recieving an offer for a data science positions, however once with 3+ years the opportunities reach it's peak. We concur that relative field experience and familiarity with data science is necessary for companies to entrust candidates with data science positions. <br>
Another trend noticed is gradually decreasing respondents as age goes up, we believe this could be an result of data science as a trend therefore more younger developer/scientist are in the market and also more experienced candidates will likely move to management or C-suite positions(CEO, COO, CTO)

```python
plt.figure(figsize=(15,6))
temp = selected_response[selected_response.DevType == 'Data scientist or machine learning specialist']
temp = temp[temp.convertedsalary < 250000]
ax = sns.boxplot(x="YearsCoding", 
                 y="convertedsalary", 
                 hue="YearsCoding", 
                 data=temp)
plt.savefig("./Figure/box plot years coding.png")
```
Visualizing age groups to salary, we again see how candidates with 0~2 year despite actually earning a job, will still likely recieve minimum salary, but with more than 2 years experience will highly increase your chances in the field.

7. Next we check if formal educations for the position is necessary
```python 
formal_education = selected_response[["FormalEducation" , "DevType"]]
cleaned_formal_education = []
for i in formal_education.values:
#     print(i)
#     break
    
    if i is not None and i[1] is not None and i[0] is not None: 
        #split multiple jobs positions
        dev_type = i[1].split(";")
        this_man_data_scientist = False
        for jobs in dev_type:
            if jobs in data_science_position_titles:
                this_man_data_scientist = True
        if this_man_data_scientist:
            cleaned_formal_education.append(i[0])
possible_education_credentials = set(cleaned_formal_education)
#possible_education_credentials
```
Possible Credentials :
```
'Associate degree',
 'Bachelor’s degree (BA, BS, B.Eng., etc.)',
 'I never completed any formal education',
 'Master’s degree (MA, MS, M.Eng., MBA, etc.)',
 'Other doctoral degree (Ph.D, Ed.D., etc.)',
 'Primary/elementary school',
 'Professional degree (JD, MD, etc.)',
 'Secondary school (e.g. American high school, German Realschule or Gymnasium, etc.)',
 'Some college/university study without earning a degree'
```

```python
total_degree_for_ds = {}
for i in formal_education.values:
    if i is not None and i[1] is not None and i[0] is not None:
        try:
            total_degree_for_ds[i[0]] +=1
        except :
            total_degree_for_ds[i[0]]= 1

```
```
'Some college/university study without earning a degree': 1220,
 'Bachelor’s degree (BA, BS, B.Eng., etc.)': 6408,
 'Master’s degree (MA, MS, M.Eng., MBA, etc.)': 1864,
 'Associate degree': 416,
 'Other doctoral degree (Ph.D, Ed.D., etc.)': 292,
 'Secondary school (e.g. American high school, German Realschule or Gymnasium, etc.)': 218,
 'I never completed any formal education': 20,
 'Primary/elementary school': 25,
 'Professional degree (JD, MD, etc.)': 30
 ```

 ```python
print_total_degree_keys = []
for i in total_degree_for_ds.keys():
    i = i.split(" " , 2) 
    print_total_degree_keys.append(i[0] + " " + i [1])
plt.bar(print_total_degree_keys, list(total_degree_for_ds.values()), align='center', alpha=0.5)
plt.ylabel('Number of Data Scientist Respondents')
plt.title('Data Scientist Education Degree')
fig = plt.gcf()
fig.set_size_inches(20, 10) 

plt.savefig("./Figure/Degree Data Scientist.png")
plt.show()
```
with this we see that most respondents with data science positions are bachelors to follow with Master degrees.
Next to compare salary to degrees
```python 
plt.figure(figsize=(15,6))
temp = selected_response[selected_response.DevType == 'Data scientist or machine learning specialist']
temp = temp[temp.convertedsalary < 250000]
ax = sns.boxplot(x="FormalEducation", 
                 y="convertedsalary", 
                 hue="FormalEducation", 
                 data=temp)
plt.xticks(rotation=10)
plt.savefig("./Figure/formal educaiton box plt.png")
```
We see that degree levels does matter for data scientist salary wise, but considering the time studying and cost for these degrees, are these degrees really worth it?
 
8. Next we look at language requirements and languages users are interested in trying for the following year

``` python 
def query_languages_and_next_year():
    query_sql = "SELECT languageworkedwith, languagedesirenextyear from " + main_table + " WHERE languageworkedwith is not null OR languagedesirenextyear is not null AND country = 'United States' AND Student = 'No' AND employment = 'Employed full-time' AND convertedsalary is not null "
    c = conn.cursor()
    c.execute(query_sql)
    conn.commit()
    rows  = c.fetchall()
    return rows
def query_languages_and_next_year_ds():
    query_sql = "SELECT languageworkedwith, languagedesirenextyear,devtype from " + main_table + " WHERE languageworkedwith is not null OR languagedesirenextyear is not null and devtype is not null AND country = 'United States' AND Student = 'No' AND employment = 'Employed full-time' AND convertedsalary is not null "
    c = conn.cursor()
    c.execute(query_sql)
    conn.commit()
    rows  = c.fetchall()
    return rows
queried_language_and_next_year = pd.DataFrame(query_languages_and_next_year() , columns = ["LanguageWorkedWith" , "LanguagesNextYear"])
queried_language_and_next_year_ds = pd.DataFrame(query_languages_and_next_year_ds() , columns = ["LanguageWorkedWith" , "LanguagesNextYear", "DevType"])

def find_languages_total_count():
    languages_all_this = {}
    languages_all_next = {}
    for i in queried_language_and_next_year.values: 
        if i is not None and i[0] is not None and i[1] is not None: 
            current_list = i[0].split(";") 
            next_list = i[1].split(";")
            for languages_current in current_list:
                try:
                    languages_all_this[languages_current] +=1
                except:
                    languages_all_this[languages_current] = 1
            for languages_next in next_list:
                try:
                    languages_all_next[languages_next] +=1
                except:
                    languages_all_next[languages_next] = 1
    return languages_all_this, languages_all_next
languages_all_this , languages_all_next = find_languages_total_count()
def find_languages_total_count_ds():
    languages_all_this = {}
    languages_all_next = {}
    for i in queried_language_and_next_year_ds.values: 
        if i is not None and i[0] is not None and i[1] is not None and i[2] is not None:
            jobs_titles = i[2].split(";")  
            
            this_man_is_ds = False
            # Check if job is data scientist
            for job_indi in jobs_titles:
                if job_indi in data_science_position_titles:
                    this_man_is_ds = True
            if this_man_is_ds :
                current_list = i[0].split(";") 
                next_list = i[1].split(";")
                for languages_current in current_list:
                    try:
                        languages_all_this[languages_current] +=1
                    except:
                        languages_all_this[languages_current] = 1
                for languages_next in next_list:
                    try:
                        languages_all_next[languages_next] +=1
                    except:
                        languages_all_next[languages_next] = 1
    return languages_all_this, languages_all_next
languages_all_this_ds , languages_all_next_ds = find_languages_total_count_ds()
```

Now we plot all users for all positions, the blue bar stands for current users, and the orange bar stands for anticipated users for the following year
``` python 
dict_languages_current_next = {"Current" : languages_all_this , "Next Year" : languages_all_next}
languages_dataframe = pd.DataFrame(dict_languages_current_next)
languages_plot = languages_dataframe.plot( kind= 'bar' , secondary_y= 'amount' , rot= 0 )
fig = plt.gcf()
fig.set_size_inches(60, 10) 
plt.savefig("./Figure/Languages_this_next.png")
plt.show()
```

Now we plot all users for data science positions, the blue bar stands for current users, and the orange bar stands for anticipated users for the following year
``` python 
dict_languages_current_next_ds = {"Current" : languages_all_this_ds , "Next Year" : languages_all_next_ds}
languages_dataframe_ds = pd.DataFrame(dict_languages_current_next_ds)
languages_plot_ds = languages_dataframe_ds.plot( kind= 'bar' , secondary_y= 'amount' , rot= 0 )
fig = plt.gcf()
fig.set_size_inches(60, 10) 
plt.savefig("./Figure/Languages_this_next_ds.png")
plt.show()
```

Now we take a look at the merged chart for all positions and data scientist, for the current year and the following year
``` python 
dict_languages_current_next_ds_merge = {"Current" : languages_all_this , "Next Year" : languages_all_next , "CurrentDS" : languages_all_this_ds , "Next Year DS" : languages_all_next_ds}
languages_dataframe_ds_merge = pd.DataFrame(dict_languages_current_next_ds_merge)
languages_plot_ds = languages_dataframe_ds_merge.plot( kind= 'bar' , secondary_y= 'amount' , rot= 0 )
fig = plt.gcf()
fig.set_size_inches(60, 10) 
plt.savefig("./Figure/Languages_this_next_ds_merge.png")
plt.show() 
```


9. Finally we look at the tools and platforms used

``` python 
def query_platforms_experience():
    query_sql = "SELECT platformworkedwith, platformdesirenextyear from " + main_table + " WHERE platformworkedwith is not null AND platformdesirenextyear is not null AND country = 'United States' AND Student = 'No' AND employment = 'Employed full-time' AND convertedsalary is not null "
    c = conn.cursor()
    c.execute(query_sql)
    conn.commit()
    rows  = c.fetchall()
    query_sql_ds = "SELECT platformworkedwith, platformdesirenextyear, devtype from " + main_table + " WHERE platformworkedwith is not null AND platformdesirenextyear is not null AND devtype is not null AND country = 'United States' AND Student = 'No' AND employment = 'Employed full-time' AND convertedsalary is not null "
    c = conn.cursor()
    c.execute(query_sql_ds)
    rows_ds  = c.fetchall()
    
    return rows, rows_ds
queried_platform, queried_platform_ds = query_platforms_experience()
queried_platforms_this_next = pd.DataFrame(queried_platform , columns = ["PlatformsCurrent" , "PlatformsNext"])
queried_platforms_this_next_ds = pd.DataFrame(queried_platform_ds , columns = ["PlatformsCurrent" , "PlatformsNext", "DevType"])
def find_platforms_all():
    platforms_current = {}
    platforms_next = {}
    platforms_current_ds = {}
    platforms_next_ds = {}
    for i in queried_platforms_this_next_ds.values: 
        if i is not None and i[0] is not None and i[1] is not None:
            jobs_titles = i[2].split(";")  
            
            this_man_is_ds = False
            # Check if job is data scientist
            for job_indi in jobs_titles:
                if job_indi in data_science_position_titles:
                    this_man_is_ds = True
                     
            current_list = i[0].split(";") 
            next_list = i[1].split(";")
            for platforms_current_indi in current_list:
                try:
                    platforms_current[platforms_current_indi] +=1
                    if this_man_is_ds:
                        platforms_current_ds[platforms_current_indi] +=1
                except:
                    platforms_current[platforms_current_indi] = 1
                    if this_man_is_ds:
                        platforms_current_ds[platforms_current_indi] = 1 
            for platforms_next_indi in next_list:
                try:
                    platforms_next[platforms_next_indi] +=1
                    if this_man_is_ds:
                        platforms_next_ds[platforms_next_indi] +=1
                except:
                    platforms_next[platforms_next_indi] = 1
                    if this_man_is_ds:
                        platforms_next_ds[platforms_next_indi] = 1
             
                
                
    return platforms_current, platforms_next , platforms_current_ds, platforms_next_ds 
platforms_current , platforms_next, platforms_current_ds, platforms_next_ds  = find_platforms_all()
```

This is the merged chart of all positions in relevance to respondent count
``` python 
# Overall normal 
dict_platforms_count_normal = {"Current" : platforms_current , "Next Year" : platforms_next }
dict_platforms_count_normal_df = pd.DataFrame(dict_platforms_count_normal)
dict_platforms_count_df_normal_plot = dict_platforms_count_normal_df.plot( kind= 'bar' , secondary_y= 'amount' , rot= 0 )
fig = plt.gcf()
fig.set_size_inches(60, 10) 
plt.savefig("./Figure/platform_normal.png")
plt.show()
```
This is the merged chart of data science positions in relevance to respondent count
``` python 
# Overall normal + DS
dict_platforms_count_ds = {"Current" : platforms_current_ds , "Next Year" : platforms_next_ds }
dict_platforms_count_ds_df = pd.DataFrame(dict_platforms_count_ds)
dict_platforms_count_df_ds_plot = dict_platforms_count_ds_df.plot( kind= 'bar' , secondary_y= 'amount' , rot= 0 )
fig = plt.gcf()
fig.set_size_inches(60, 10) 
plt.savefig("./Figure/platform_ds.png")
plt.show()
```
To follow is the merged chart or all positions and data scientist, with the tools and platforms used this year and anticipated to use next year.
``` python 
# Overall normal + DS
dict_platforms_count = {"Current" : platforms_current , "Next Year" : platforms_next , "CurrentDS" : platforms_current_ds , "Next Year DS" : platforms_next_ds}
dict_platforms_count_df = pd.DataFrame(dict_platforms_count)
dict_platforms_count_df_plot = dict_platforms_count_df.plot( kind= 'bar' , secondary_y= 'amount' , rot= 0 )
fig = plt.gcf()
fig.set_size_inches(60, 10) 
plt.savefig("./Figure/platform_merge.png")
plt.show()
```

10. IDE
``` python 
def query_ide():
    query_sql = "SELECT ide, devtype from " + main_table + " WHERE ide is not null AND devtype is not null AND country = 'United States' AND Student = 'No' AND employment = 'Employed full-time' AND convertedsalary is not null "
    c = conn.cursor()
    c.execute(query_sql)
    conn.commit()
    rows  = c.fetchall() 
    
    return rows  
queried_ide = pd.DataFrame(query_ide() , columns = ["IDE" , "DevType"])

def find_ide_count():
    ide = {}
    ide_ds = {} 
    for i in queried_ide.values: 
        if i is not None and i[0] is not None and i[1] is not None:
            jobs_titles = i[1].split(";")  
            
            this_man_is_ds = False
            # Check if job is data scientist
            for job_indi in jobs_titles:
                if job_indi in data_science_position_titles:
                    this_man_is_ds = True
                     
            current_list = i[0].split(";")  
            for ide_indi in current_list:
                try:
                    ide[ide_indi] +=1
                    if this_man_is_ds:
                        ide_ds[ide_indi] +=1
                except:
                    ide[ide_indi] = 1
                    if this_man_is_ds:
                        ide_ds[ide_indi] = 1   
    return  ide , ide_ds 
ide , ide_ds = find_ide_count()

```

Plot for all respondents 
``` python 
##### PLOTTING ####
# Overall normal 
dict_ids_count = {"All" : ide , "DS Nerds :D" : ide_ds }
dict_ids_count_df = pd.DataFrame(dict_ids_count)
dict_ids_count_df_plot = dict_ids_count_df.plot( kind= 'bar' , secondary_y= 'amount' , rot= 0 )
fig = plt.gcf()
fig.set_size_inches(50, 10) 
plt.savefig("./Figure/ide_choices.png")
plt.show()
``` 
Plot for just data science respondents
``` python 
# Overall normal 
dict_ids_count = { "DS Nerds :D" : ide_ds }
dict_ids_count_df = pd.DataFrame(dict_ids_count)
dict_ids_count_df_plot = dict_ids_count_df.plot( kind= 'bar' , secondary_y= 'amount' , rot= 0 )
fig = plt.gcf()
fig.set_size_inches(50, 10) 
plt.savefig("./Figure/ide_choices_ds.png")
plt.show()

``` 

11. Operating Systems
``` python 
def query_os():
    query_sql = "SELECT operatingsystem, devtype from " + main_table + " WHERE operatingsystem is not null AND devtype is not null AND country = 'United States' AND Student = 'No' AND employment = 'Employed full-time' AND convertedsalary is not null "
    c = conn.cursor()
    c.execute(query_sql)
    conn.commit()
    rows  = c.fetchall() 
    
    return rows  
queried_os = pd.DataFrame(query_os() , columns = ["IDE" , "DevType"])
def find_os_count():
    os = {}
    os_ds = {} 
    for i in queried_os.values: 
        if i is not None and i[0] is not None and i[1] is not None:
            jobs_titles = i[1].split(";")  
            
            this_man_is_ds = False
            # Check if job is data scientist
            for job_indi in jobs_titles:
                if job_indi in data_science_position_titles:
                    this_man_is_ds = True
                     
            current_list = i[0].split(";")  
            for os_options in current_list:
                try:
                    os[os_options] +=1
                    if this_man_is_ds:
                        os_ds[os_options] +=1
                except:
                    os[os_options] = 1
                    if this_man_is_ds:
                        os_ds[os_options] = 1   
    return os , os_ds 
os , os_ds =  find_os_count()
```
Linux based users are scarce, but when regarding percentage, alot of data scientist are using linux
- would strongly recommend linux
- > i use arch linux btw (from an ubuntu user)

``` python 
# Overall normal 
dict_os_count = {"All" : os , "DS Nerds :D" : os_ds }
dict_os_count_df = pd.DataFrame(dict_os_count)
dict_os_count_df_plt = dict_os_count_df.plot( kind= 'bar' , secondary_y= 'amount' , rot= 0 )
fig = plt.gcf()
fig.set_size_inches(15, 10) 
plt.savefig("./Figure/os.png")
plt.show()
```
12. Salary

``` python 
def query_salary_respondents():
    query_sql = "SELECT convertedsalary from " + main_table + " WHERE country = 'United States' AND Student = 'No' AND employment = 'Employed full-time' AND convertedsalary is not null "
    c = conn.cursor()
    c.execute(query_sql)
    conn.commit()
    rows  = c.fetchall() 
    none_count = 0
    has_value_count = 0
    for i in rows: 
        if i[0] is None:
            none_count +=1
        else:
            has_value_count+=1 
    return has_value_count / ( none_count + has_value_count)
percentage_salary_respondents = query_salary_respondents()

query_sql = "SELECT convertedsalary from " + main_table
c = conn.cursor()
c.execute(query_sql)
conn.commit()
rows  = c.fetchall() 
# print(rows)

is_none = 0
is_not_none = 0
for i in rows:
#     print(i[0])
    if i[0] is None or i[0] == 0:
        is_none +=1
    else:
        is_not_none +=1
print("percentage " , is_not_none/(is_none+is_not_none))
```
We see here that only 48% of respondents responded with their salary, therefore we think you should take all the responses with a grain of salt and use it just for reference.

Female salary
``` python 
def query_female_salary():
    
    query_sql = "SELECT convertedsalary, gender  from " + main_table + " WHERE gender like 'Female'"
    c = conn.cursor()
    c.execute(query_sql)
    conn.commit()
    rows  = c.fetchall() 
    female_salary_list = []
    for i in rows:
        if i[0] is not None:
#             print(float(i[0]))
            female_salary_list.append(float(i[0]))
    return female_salary_list
female_salary = pd.DataFrame(query_female_salary() , columns = ["Female Salary"])
print(female_salary.describe().round())
    
```
Male Salary
``` python
def query_male_salary():
    
    query_sql = "SELECT convertedsalary, gender  from " + main_table + " WHERE gender like 'Male'"
    c = conn.cursor()
    c.execute(query_sql)
    conn.commit()
    rows  = c.fetchall() 
    male_salary_list = []
    for i in rows:
        if i[0] is not None:
#             print(float(i[0]))
            male_salary_list.append(float(i[0]))
    return male_salary_list
male_salary = pd.DataFrame(query_male_salary() , columns = ["Male Salary"])
print(male_salary.describe().round())
    
```

Overall salary
``` python 
def query_all_salary():
    
    query_sql = "SELECT convertedsalary, gender  from " + main_table 
    c = conn.cursor()
    c.execute(query_sql)
    conn.commit()
    rows  = c.fetchall() 
    all_salary_list = []
    for i in rows:
        if i[0] is not None:
#             print(float(i[0]))
            all_salary_list.append(float(i[0]))
    return all_salary_list
all_salary = pd.DataFrame(query_all_salary() , columns = ["All Salary"])
print(all_salary.describe().round())
    
```

---
### **Part 3 : Results**   <br> 
All in all, we can see a few trends happening and expected to happen in the following year. However the data is extremely dirty and many of the answers are ambiguous and could vary highly with the respondents' perspective to the question and answer; also since only 49% of the respondents responded with their salary, we think the salary related responses maybe too biased so again, take it with a grain of salt and use it just for reference.
To the prospering data scientists, we believe that looking for data science positions right out of school may not be ideal and would recommend preparing for an alternative route including but not limited to software engineering and transfer into data science a few years after understanding the field. Projects and internships are highly recommended since it directly shows competence and real life experience in the field where it's really hard to quantify an candidate's capabilities. <br>
Good luck to all those looking for a positions ;)
<br>
If any edits are recommended or mistakes i've made, fell free to contact me, i really enjoy advice and corrections.