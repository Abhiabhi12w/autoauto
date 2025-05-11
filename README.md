I wanna automate an analysis in excel macro enabled workkbook

I have to analysis how many policies are there (count)

in that file 

1. In AAC columns which has name as TASK_TYPE we have to apply filter on 6th row (on 6-AAC cell) and filter out 999 values.
2. Then in E column (has name Scenario_Group) we update all blanks as 999.
3. Then remove flter
4. Then in K columns which has name as HISTORY_MONTHS we have to apply filter on 6th row and filter out values having Run_Date.
5. Then in E column (has name Scenario_Group) we update all blanks as 999.
6. Then remove flter
7. In AAC columns which has name as TASK_TYPE we have to apply filter on 6th row (on 6-AAC cell) and filter out 889 values.
8. Then in E column (has name Scenario_Group) we update all blanks as 889.
9. Remove filter
10. Now in AAB we have field named as ADO_ID, here we have for example 5 ids repeated mutiple times so we only take ids one time each and for each id we need count.

how we do 10th step manually is 

in ado_id we first apply filter by ctrl+shift+l, then select 1st id only and unsleect others

then we update policy count by  adding all the values in I column and also we need K columns data

output should be like 

ado_id 889/999? policy-count K-columns-values

for example:

1442851 889	4 1

if it has mulitple values in in K column

for example:

1442851 889	4 1,3


write the output to a new excel file
