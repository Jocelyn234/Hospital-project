/*What is the age of patients at the time of admission?*/ 

SELECT * FROM patients;
SELECT* FROM main.admissions;

SELECT
patients.subject_id,
admissions.hadm_id,
	date_diff('year',dob,admittime) as patient_age,
from main.patients
inner join main.admissions
on patients.subject_id = admissions.subject_id
ORDER BY patients.subject_id;

2.	/*Group patients into various age categories based on their age at admission.*/

SELECT
patients.subject_id,
admissions.hadm_id,
	date_diff('year',dob,admittime) as age,
	CASE 
		when age <=30 then 'young'
		when age between 31 and 40 then 'youth'
		when age between 41 and 50 then 'middle'
		when age between 51 and 70 then 'old'
		else 'elderly'
	END as age_group
from main.patients
inner join main.admissions
on patients.subject_id = admissions.subject_id
ORDER BY patients.subject_id;

3.	/*How would you describe the demographics of admitted patients in terms of ethnicity?*/

SELECT 
	ethnicity,
	count(DISTINCT subject_id) AS number_of_patients,
	CASE 
		when ethnicity LIKE '%ASIAN%' then 'Asian'
		when ethnicity LIKE '%WHITE%' then 'White'
		when ethnicity LIKE '%BLACK%' then 'Black'
		when ethnicity LIKE '%HISPANIC%' then 'Hispanic'
		else 'Other'
	END as patients_ethnicity,
FROM main.admissions
GROUP BY ethnicity,
ORDER BY number_of_patients; 

4.	/*What is the average hospital length of stay, broken down by ethnicity?*/

select 
ethnicity,
	count(DISTINCT subject_id) AS number_of_patients,
	AVG(date_diff('day',admittime,dischtime)) AS hospital_stay,
from main.admissions 
GROUP BY ethnicity
ORDER BY number_of_patients desc;

5.	/*What are the most common diagnoses for each age group as defined in Question 1?*/
	
WITH diagnosis_agegroup AS(
select 
	CASE 
		when date_diff('year',dob,admittime) <=30 then 'young'
		when date_diff('year',dob,admittime) between 31 and 40 then 'youth'
		when date_diff('year',dob,admittime) between 41 and 50 then 'middle'
		when date_diff('year',dob,admittime) between 51 and 70 then 'old'
		else 'elderly'
	END as age_group,
	diagnosis,
	count (admissions.hadm_id) AS n_diagnosis,
	ROW_NUMBER () OVER (
		PARTITION BY age_group
		ORDER BY n_diagnosis desc)
		AS rn
	from main.admissions
	inner join main.patients
		on patients.subject_id = admissions.subject_id
GROUP BY diagnosis,age_group)
SELECT * FROM diagnosis_agegroup
WHERE rn = 1


Admissions
1.	/*How many unique patients were admitted?*/ 
	
SELECT 
	count (DISTINCT subject_id)
FROM main.admissions; 

2.	/*What is the total number of admissions?*/

SELECT 
	count(DISTINCT hadm_id), 
FROM main.admissions; 

3.	/*How many patients are admitted each year?*/

SELECT
	count(subject_id) AS number_of_patients,
	EXTRACT ('year'FROM admittime) AS admit_year 
FROM main.admissions
	GROUP BY admit_year,
	ORDER BY admit_year;

4.	/*What is the breakdown of admissions by weekdays for each year?*/

SELECT
	count(hadm_id) AS number_of_admissions,
	dayofweek(admittime) AS day,
	dayname(admittime) AS weekday,
	EXTRACT ('year'FROM admittime) AS admit_year, 
FROM main.admissions
	GROUP BY weekday, admit_year, day
	ORDER BY admit_year,day;

5.	/*What is the average hospital length of stay?*/

select 
	AVG(date_diff('day',admittime,dischtime)) AS hospital_stay,
from main.admissions; 


6.	/*Provide a breakdown of admissions by type and source of admission.*/
•	/*From where are patients typically admitted?*/
•	/*Where are patients discharged to?*/

SELECT
	admission_type, admission_location,
	count(hadm_id) as number_of_admissions,
FROM main.admissions
	GROUP BY admission_type, admission_location,
	ORDER BY admission_type, admission_location;

SELECT
	discharge_location, 
	count(hadm_id) as number_of_admissions,
FROM main.admissions
	GROUP BY discharge_location;
	
7.	/*What are the types and percentages of insurers covering the patients?*/

SELECT 
	insurance 
FROM main.admissions
GROUP BY insurance, 
 order BY insurance;

SELECT 
	insurance,
	count(hadm_id) as number_of_admissions,
(count(subject_id)/(SELECT count(subject_id) FROM main.admissions)*100) AS percentage_insurers
FROM main.admissions
	GROUP BY insurance,
	ORDER BY insurance;

8.	/*What percentage of patients are admitted from the Emergency Department (ED), and what is the ED admission rate?*/

SELECT 
	admission_type,
(count(subject_id)/(SELECT count(subject_id) FROM main.admissions)*100) AS percentage_patient
FROM main.admissions
	where admission_type = 'EMERGENCY'
	GROUP BY admission_type;


9.	/*What is the average length of time patients spend in the ED?*/

select 
	admission_type, 
	AVG(date_diff('day',admittime,dischtime)) AS ED_time,
from main.admissions
	WHERE admission_type = 'EMERGENCY'
	GROUP BY admission_type;
	
10.	/*How many patients passed away during their hospital stay?*/

SELECT  
	discharge_location,
	count(subject_id) as number_of_patients
FROM main.admissions
	Where discharge_location like 'DEAD/EXPIRED'
	GROUP BY discharge_location;
	
11.	/*How many admissions had a hospital length of stay greater than the average?*/

select 
 count (hadm_id) as greater_than_average_stay
from main.admissions
where date_diff('day',admittime,dischtime) > (

	SELECT 
		AVG(date_diff('day',admittime,dischtime)) as length_of_stay
	from main.admissions
);


12.	/*How many patients were admitted for a duration longer than the average length of stay for each admission type*/

select 
admission_type, 
 count (subject_id) as number_patients
from main.admissions
where date_diff('day',admittime,dischtime) > (

	SELECT 
		AVG(date_diff('day',admittime,dischtime)) as length_of_stay
	from main.admissions
)
GROUP BY admission_type;
