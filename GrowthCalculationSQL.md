For accountability in Texas, half points are awarded for partial growth. These are tests where the student did not grow a full year based on the scale score but did maintain the pass level (e.g. they were at "meets" in 2018 on math and at "meets" in 2019 on math). To remove these half-points from the data and show only full year's growth, this query was used with the camprate file.

For my dissertation, only the Dallas ISD schools were used. The file called GrowthCalculation2019 includes all Texas campuses, not just Dallas.

```tsql
--This is the methodology to calculate growth rate baesd on scale score, omitting the half-points for staying at the same growth level.
--Add the last 4 columns and divide by the denominator... This is explained in detail in the Data Sources and Collection Procedures section of your dissertation

select cdc_campus, --cdc_district, 
'denominator' = BothTestsNumber, 
BothAlllevelstoDidNotMeet_1, BothAlllevelstoApproaches1, BothAlllevelstoMeets_1, BothAlllevelstoMasters_NA,
year_growth_num = sum(cast(BothAlllevelstoDidNotMeet_1 as float) + cast(BothAlllevelstoApproaches1 as float) + cast(BothAlllevelstoMeets_1 as float) + cast(BothAlllevelstoMasters_NA as float)),
year_growth_rate = ((sum(cast(BothAlllevelstoDidNotMeet_1 as float) + cast(BothAlllevelstoApproaches1 as float) + cast(BothAlllevelstoMeets_1 as float) + cast(BothAlllevelstoMasters_NA as float)))/(BothTestsNumber))
--,* 

from DISS_tapr_camp_d2a_2019

where cdc_district = '057905' and cast(BothTestsNumber as float) > 0 --the 11 with a denominator of 0 are either not rated or rated with paired campuses --see list below
group by cdc_campus, BothTestsNumber, BothAlllevelstoDidNotMeet_1, BothAlllevelstoApproaches1, BothAlllevelstoMeets_1, BothAlllevelstoMasters_NA

order by 1
```
