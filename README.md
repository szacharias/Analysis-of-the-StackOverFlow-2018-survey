# Analysis of the StackOverFlow 2018 survey  

Part 1 : Objective <br>
Part 2 : Steps<br>
Part 3 : Code<br>
Part 4 : Results<br>


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



We will follow up by analyzing how distributed our audience are distributed throughout the world. Perhaps seeing a rise in survey takers in India, with this information we can analyze specific problems developers from different parts of the world are going through. With a followup in analysis of type of employment(full time, part time, contractors) and different fields of workers(health, tech, retail.. etc).  One other point in audience analysis that we are particularly interested in is imposter syndrome; as we learn more of the industry, we feel like we know less about everything, but is that what's really happening? With analysis to audience's thoughts to imposter syndrome, hopefully we can help those not so confident in oneself and let them know that not knowing everything is fine.

Furthermore, the next thing we're most interested in is how and what we are expected as a software engineer. The analysis of skill sets will allow us to prepare and perhaps even help others decide what classes or skills are required to be given to students just entering the field. Perhaps even tools including IDEs and databases to pick up for a better opportunity of getting a job, maybe using a Linux based system will help out your job search. Language preference is also a strong point in our analysis, with an overabundance of programming languages available, what are the better options and what does the corporate side of things enjoy but we are unaware. 

Finally we analyse salary. Salary is one of if not the most important component for a job. No one likes to be low balled or underpaid, right? We will analyze our data with respect to different states in America and what our anticipated salary would be for each position of different levels. Or even figure out how salary increases as our years of experience increases and what else is expected from us deeper into our career.


---
### **Part 3 : Code**   <br> 


---
### **Part 4 : Results**   <br> 
