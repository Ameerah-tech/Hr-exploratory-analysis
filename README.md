-- Preview table
Select * from hr_employee_raw;

-- Duplicate table
create table Hr_employee_cleaning as select * from Hr_employee_raw;
-- Data Cleaning

-- Finding Duplicate
select * from (select *, row_number() over(partition by 
Age,Attrition,BusinessTravel,DailyRate,Department,DistanceFromHome,Education,EducationField,EmployeeCount,
EmployeeNumber,EnvironmentSatisfaction,Gender,HourlyRate,JobInvolvement,JobLevel,JobRole,JobSatisfaction,
MaritalStatus,MonthlyIncome,MonthlyRate,NumCompaniesWorked,Over18,OverTime,PercentSalaryHike,PerformanceRating,
RelationshipSatisfaction,StandardHours,StockOptionLevel,TotalWorkingYears,TrainingTimesLastYear,WorkLifeBalance,
YearsAtCompany,YearsInCurrentRole,YearsSinceLastPromotion,YearsWithCurrManager) 
duplicate from hr_employee_cleaning) as Hi where duplicate >1; -- No duplicate found

-- Standardize data type
Alter table hr_employee_cleaning modify Attrition varchar(10) ;
Alter table hr_employee_cleaning modify BusinessTravel varchar(50);
Alter table hr_employee_cleaning modify Department varchar(50);
Alter table hr_employee_cleaning modify EducationField varchar(50);
Alter table hr_employee_cleaning modify Gender varchar(50);
Alter table hr_employee_cleaning modify JobRole varchar(50);
Alter table hr_employee_cleaning modify MaritalStatus varchar(50);
Alter table hr_employee_cleaning modify Over18 varchar(50);
Alter table hr_employee_cleaning modify OverTime varchar(50);

-- Creating view
Create or replace view hr_eda as select Age,Attrition,department,Jobrole,Joblevel,Gender,Overtime,
YearsAtCompany,YearsInCurrentRole,StandardHours,EmployeeCount,PerformanceRating,JobSatisfaction,BusinessTravel,
RelationshipSatisfaction,MaritalStatus,MonthlyIncome,PercentSalaryHike,YearsSinceLastPromotion, TotalWorkingYears,
DistanceFromHome,WorkLifeBalance,(case when Age between 18 and 24 then '18-24'
when Age between 25 and 34 then '25-34'
when Age between 35 and 44 then '35-44'
when Age between 45 and 54 then '45-54'
when Age between 55 and 60 then '55-60' end) as Age_group, 
 (Case when Monthlyincome <=5000 and Overtime='Yes' then 'Underpaid & Overworked'
when monthlyincome <=10000 and overtime='Yes' then 'Fairlypaid & Overworked' 
when monthlyincome >10000 and overtime='Yes' then 'Wellpaid & Overworked'  
else 'Not Overworked' end) as Risk,
 (Case when WorkLifeBalance =1 then 'Poor'
when  WorkLifeBalance =2 then 'Fair'
when WorkLifeBalance =3 then 'Good' 
else 'Excellent' end) as Work_life_balance_level, 
 (Case when JobSatisfaction =1 then 'Low'
when  JobSatisfaction =2 then 'Medium'
when JobSatisfaction =3 then 'High' 
else 'Very High' end) as Job_Satisfaction_level 
from hr_employee_cleaning;

-- Analysis

-- Total Number of employee
Select count(*) as Total_Employee from hr_eda;  -- 1470 total employees

-- Total Number of employees who left
Select count(*) as Total_Employee_left from hr_eda where attrition='Yes'; -- 237 employee left

-- Total Number of employee by Department
Select Department,count(*) from hr_eda group by department;

-- Total Number of employess who left by department
Select Department,count(case when attrition='Yes' then 1
end) as Total_Employee_left  from hr_eda group by department;

-- Average Salary by Department
Select Department,Round(avg(Monthlyincome),2) as Average_salary from hr_eda group by Department;

-- Total Number of employee by Job Role
Select JobRole, count(*) from hr_eda group by JobRole;

-- Total Number of employess who left by Job Role
Select JobRole, count(*) as Total_Employee, count(case when attrition='Yes' then 1
end) as Total_Employee_left  from hr_eda group by JobRole;

-- Average Salary by Job Role
Select JobRole,Round(avg(monthlyincome),2) as Average_Salary from hr_eda group by JobRole;

-- Total Number of Employee who work Overtime
Select Count(*) as Overtime_Employee from hr_eda where Overtime ='Yes';

-- Total Number of Employee by Travel
Select BusinessTravel, Count(*) from hr_eda group by BusinessTravel;

-- Total Number of Employee who left by Travel
Select BusinessTravel, count(case when attrition='Yes' then 1 end) as Total_Employee_Left
from hr_eda group by BusinessTravel;

-- Total Number of Employee by Gender
Select Gender,count(*) from hr_eda group by Gender;

-- Total Number of Employee who left by Gender
Select Gender,count(case when Attrition ='Yes' then 1 end) as Total_Employee_Left
from hr_eda group by Gender;

-- Total Number of Employee by Age_group
select Age_group, count(*) as Total_Employee  
 from hr_eda group by age_group order by Age_group;
 
 -- Total Number of Employee who left by Age_group
select Age_group,count(case when attrition='Yes' then 1
end) as Total_Employee_left 
 from hr_eda group by age_group order by Age_group;
 
-- Attrition rate
Select concat(round(count(case when attrition='Yes' then 1
end) * 100.0 / count(*),2),'%') as Attrition_rate from hr_eda; -- 16.12% attrition rate

-- Attrition rate by department
Select Department, concat(round(count(case when attrition='Yes' then 1
end) * 100.0 / count(*),2),'%') as Attrition_rate from hr_eda group by department;

-- Attrition rate by Job Role
Select JobRole, concat(round(count(case when attrition='Yes' then 1
end) * 100.0 / count(*),2),'%') as Attrition_rate from hr_eda group by JobRole;

-- Attrition rate by Overtime
Select overtime,concat(round(count(case when attrition='Yes' then 1 end)* 100.0 / count(*),2),'%')
as Attrition_rate from hr_eda group by overtime;

-- Attrition rate by Travel
Select BusinessTravel,concat(round(count(case when attrition='Yes' then 1 end) *100.0/ count(*),2),'%')
as Attrition_rate from hr_eda group by BusinessTravel;

-- Attrition rate by Gender
Select Gender,concat(round(count(case when attrition='Yes' then 1 end) *100.0/ count(*),2),'%')
as Attrition_rate from hr_eda group by Gender;

-- Attrition rate by Marital Status
Select MaritalStatus,concat(round(count(case when attrition='Yes' then 1 end) *100.0/ count(*),2),'%')
as Attrition_rate from hr_eda group by MaritalStatus;

-- Attrition rate by Age group
Select Age_group,concat(round(count(case when attrition='Yes' then 1 end) * 100.0/count(*),2),'%') as Attrition_rate 
from hr_eda group by Age_group order by Age_group;

-- Attriton rate by Work life balance level
Select Work_life_balance_level, concat(round(count(case when attrition='Yes' then 1 end) * 100.0/count(*),2),'%') as Attrition_rate 
from hr_eda group by Work_life_balance_level;

-- Attrition rate by Job satisfaction level
Select Job_satisfaction_level, concat(round(count(case when attrition='Yes' then 1 end) * 100.0/count(*),2),'%') as Attrition_rate 
from hr_eda group by  Job_satisfaction_level;

-- Attrition and Overtime rate by Department
Select Department,concat(round(count(case when Attrition='Yes' then 1 end) *100.0/count(*),2),'%') as Attrition_rate,
concat(round(count( case when Overtime='Yes' then 1 end) *100/ count(*),2),'%') as Overtime_rate 
from hr_eda group by Department;

-- Attrition rate, Overtime rate, Average Salary by Job Role
Select JobRole, count(*) as Total_Employee,round(avg(monthlyincome),2) as Avg_Salary,
concat(round(count(case when attrition='Yes' then 1 end) * 100.0 / count(*),2),'%') as Attrition_rate,
concat(round(count(case when overtime='Yes' then 1 end)*100.0 / count(*),2),'%')  as Overtime_rate 
from hr_eda group by JobRole order by jobrole;

-- Salary level by Attrition
Select (Case when Monthlyincome <=5000 then 'Underpaid'
when monthlyincome <=10000 then 'Fairlypaid'   
else 'Wellpaid' end) as Salary_level, concat(round(count(case when Attrition='Yes' then 1 end) *100/count(*),2),'%') as Attrition_rate
 from hr_eda group by Salary_level;
 
-- Do employees with experience and long promotion gap have higher attrition rate
Select Experience,Promotion,concat(round(count(case when Attrition='Yes' then 1 end)*100.0/count(*),2),'%') as Attrition_rate
 from (select Attrition, (case when TotalWorkingYears <=5 then '0-5 years'
when TotalWorkingYears between 6 and 10 then '6-10 years'
when TotalWorkingYears between 11 and 20 then '11-20 years'
else '20+ years' end) as Experience, 
(case when YearsSinceLastPromotion <= 5 then 'Recently Promoted'
when YearsSinceLastPromotion between 6 and 10 then 'Moderate Promotion Gap'
else 'Long Promotion Gap' end)as Promotion from hr_eda)as t group by Experience,Promotion order by Attrition_rate desc;

-- Examine how work-life balance and job satisfaction level affect employee attrition
select Work_life_balance_level, Job_satisfaction_level,
concat(round(count(case when Attrition='Yes' then 1 end) * 100.0 / count(*),2),'%') as Attrition_rate 
from hr_eda group by Work_life_balance_level, Job_satisfaction_level;

-- Examine employees by their job role, experience, promotion gap and overtime rate
with analysis as (
select JobRole, count(*) as Total_Employee,round(avg(monthlyincome),2) as Avg_Salary,
concat(round(count(case when attrition='Yes' then 1 end) * 100.0 / count(*),2),'%') as Attrition_rate,
concat(round(count(case when overtime='Yes' then 1 end)*100.0 / count(*),2),'%')  as Overtime_rate,
(case when TotalWorkingYears <=5 then '0-5 years'
when TotalWorkingYears between 6 and 10 then '6-10 years'
when TotalWorkingYears between 11 and 20 then '11-20 years'
else '20+ years' end) as Experience, 
(case when YearsSinceLastPromotion <= 5 then 'Recently Promoted'
when YearsSinceLastPromotion between 6 and 10 then 'Moderate Promotion Gap'
else 'Long Promotion Gap' end)as Promotion from hr_eda group by Jobrole,totalworkingyears,yearssincelastpromotion)
select distinct Jobrole, Experience,Promotion,Attrition_rate,Overtime_rate
from analysis order by jobrole;
# Hr-exploratory-analysis
