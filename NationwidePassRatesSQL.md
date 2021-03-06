```tsql

--SQL to isolate math and reading pass rates from the United States Department of Education EdFacts Data Files (SCHOOL LEVEL CSV, LONG FILE)
--found at https://www2.ed.gov/about/inits/ed/edfacts/data-files/index.html

--Used to build table called NationalTestScores2019 that was later used to add test results to UsSchoolData2019_CompiledOLD.


with TestScores2019 as (
SELECT [SCHOOL_YEAR]
      ,[STNAM]
      ,[FIPST]
      ,[LEAID]
      ,[ST_LEAID]
      ,[LEANM]
      ,[NCESSCH]
      ,[ST_SCHID]
      ,[SCHNAM]
      ,[SUBJECT]
      ,[GRADE]
      ,[CATEGORY]
      ,[DATE_CUR],
	  NUMVALID
      ,[PCTPROF], 
	  NumValidFloat = cast([NUMVALID] as float),
	  PctProfFloat = cast((case
				 when pctprof = 'LE1' then '1'
				 when pctprof = 'ge99' then '99'
				 else PCTPROF end) as float)
  FROM [Engage].[dbo].[EdFactsElarLong2019]

   where grade = '00' and CATEGORY = 'all'
  and PCTPROF not in ('ps') and PCTPROF not like '%-%' and PCTPROF not like '%le5%' and PCTPROF not like '%ge95%' 
  and cast([NUMVALID] as float) >= 200

union (
SELECT [SCHOOL_YEAR]
      ,[STNAM]
      ,[FIPST]
      ,[LEAID]
      ,[ST_LEAID]
      ,[LEANM]
      ,[NCESSCH]
      ,[ST_SCHID]
      ,[SCHNAM]
      ,[SUBJECT]
      ,[GRADE]
      ,[CATEGORY]
      ,[DATE_CUR],
	  NUMVALID
      ,[PCTPROF], 
	  NumValidFloat = cast([NUMVALID] as float),
	  PctProfFloat = cast((case
				 when pctprof = 'LE1' then '1'
				 when pctprof = 'ge99' then '99'
				 else PCTPROF end) as float)
  FROM [Engage].[dbo].[EdFactsMathLong2019]

   where grade = '00' and CATEGORY = 'all'
  and PCTPROF not in ('ps') and PCTPROF not like '%-%' and PCTPROF not like '%le5%' and PCTPROF not like '%ge95%' 
  and cast([NUMVALID] as float) >= 200
))

select distinct b.NCESSCH, b.SCHOOL_YEAR, --r.NCESSCH, m.NCESSCH, 
b.ST_SCHID, b.SCHNAM,  b.GRADE, b.CATEGORY, 

readingNumValidFloat = r.NumValidFloat, readingPctProfFloat = r.PctProfFloat,
mathNumValidFloat = m.NumValidFloat, mathPctProfFloat = m.PctProfFloat

--into NationalTestScores2019
from TestScores2019 b

left join TestScores2019 r
on r.NCESSCH = b.NCESSCH and r.SUBJECT = 'rla'

left join TestScores2019 m
on m.NCESSCH = b.NCESSCH and m.SUBJECT = 'MTH'
  
--where b.ST_SCHID like '%tx-057905%' and b.SCHNAM like '%h s%'

--order by 2, 3








/*
ORIGINAL:

SELECT [SCHOOL_YEAR]
      ,[STNAM]
      ,[FIPST]
      ,[LEAID]
      ,[ST_LEAID]
      ,[LEANM]
      ,[NCESSCH] --federal school id; be careful if you open this in excel. It will change it to scientific notation and abridge part of it when converting back.
      ,[ST_SCHID] --based on state school IDs; substring can be used for joins with state-released data.
      ,[SCHNAM]
      ,[SUBJECT]
      ,[GRADE]
      ,[CATEGORY] --see below ofr category descriptions (all means "all students")
      ,[DATE_CUR]
      ,denominator = cast([NUMVALID] as float) --numvalid: total number of students who completed the state assessment and for whom a proficiency level was assigned
      ,[PCTPROF] --The percentage of students scoring at or above the state???s proficiency level on the assessment, ps = masked
	  --select top 1000 *
  FROM [Engage].[dbo].[EdFactsMathLong2019] --my name for EDF_SCH_AP_MTH_1819_PUBL; be sure to download the reading, too.
  where grade = '00' and CATEGORY = 'all' 
  --and PCTPROF not in ('ps') and PCTPROF not like '%le%' and PCTPROF not like '%ge%' and PCTPROF not like '%-%' --uncomment this to include binned and masked pass rates
  --and cast([NUMVALID] as float) >= 200


  order by 14, 15 desc

  --https://www2.ed.gov/about/inits/ed/edfacts/data-files/index.html
  --CATEGORIES:
  ----ALL = all students
  ----MAM = Native American
  ----MAS = Asian/Pacific Islander
  ----MHI = Latino
  ----MBL = Black
  ----MWH = White
  ----MTR = Two+ Races
  ----CWD = disabilities (IDEA)
  ----ECD = Economically Disadvantaged
  ----LEP = Limited English Proficient
  ----F	  = Female
  ----M	  = Male
  ----HOM = Homeless
  ----MIG = Migrant
  ----FCS = Foster Care
  ----MIL = Military Connected

*/
```
