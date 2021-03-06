This SQL combines the base file with other data sets to create a file with compiled data for all campuses in the US.

The resulting file, called UsSchoolData2019_CompiledOLD, is in the repository. 
NOTE THE OLD. Another file, called UsSchoolData2019_Compiled, is there, too. It also has the reading and math test scores for each campus.

sources:
UsSchoolData2019_Base aka base file from SQL query called FederalSchoolBaseBuildSQL in repository
NCES_Membership2019 aka Membership from https://nces.ed.gov/ccd/files.asp#Fiscal:2,LevelId:7,SchoolYearId:33,Page:1
NCES_Directory2019 aka Directory from https://nces.ed.gov/ccd/files.asp#Fiscal:2,LevelId:7,SchoolYearId:33,Page:1
CENSUS_SchoolDistrictPovertyEstimates2019 aka SAIPE School District Estimates for 2019 Estimates from every school in the nation TXT from https://www.census.gov/data/datasets/2019/demo/saipe/2019-school-districts.html


```tsql

with RaceEthnicityCounts as (
select distinct ncessch, RACE_ETHNICITY, n = sum(cast(STUDENT_COUNT as int))
from [dbo].NCES_Membership2019
where TOTAL_INDICATOR = 'Category Set A - By Race/Ethnicity; Sex; Grade'
group by NCESSCH, race_ethnicity),

EnrollmentFloatTable as (select bf.NCESSCH, EnrollmentFloat = cast(bf.enrollment as float) from [Engage].[dbo].[UsSchoolData2019_Base] bf),

CensusPoverty as (SELECT distinct [State], homemadeLeaId = concat(c.StateFips,c.DistrictId), DistrictName = [SchoolDistrict],
EstTotalPopInDistrict = cast(REPLACE(EstTotalPop,',','') as float),
EstPop5_17PovertyFloat =  cast(REPLACE(EstPop5_17Poverty,',','') as float),
EstPop5_17Float = cast(REPLACE(EstPop5_17,',','') as float),
EstPop5_17PovertyRate = (case
			when cast(REPLACE(EstPop5_17,',','') as float) = 0 then cast(REPLACE(EstPop5_17,',','') as float)
			else cast(REPLACE(EstPop5_17Poverty,',','') as float)/cast(REPLACE(EstPop5_17,',','') as float) end)
  FROM [Engage].[dbo].[CENSUS_SchoolDistrictPovertyEstimates2019] c)

SELECT b.NCESSCH, b.SCHID, b.ST_SCHID, b.SCH_NAME, 
b.LEAID, b.ST_LEAID, d.LEA_NAME, d.WEBSITE, 
b.FIPST, d.STATE_AGENCY_NO, b.ST, b.STATENAME,


charter = d.CHARTER_TEXT, 

adultsServedN = m.STUDENT_COUNT, d.SCH_TYPE_TEXT, 
SchoolLevel = (case
					when d.GSHI in ('kg', '01', '02') then 'Early Elementary'
					else d.[LEVEL] end),
gradesServed = concat(d.GSLO, '-', d.GSHI),
b.[Enrollment],
ethnTotalFlag = (case
					when (hl.n + bl.n + w.n + pac.n + natv.n + a.n + two.n + ns.n) = 0 then 'No Ethnic Groupings'
					else '' end),
b.[LunchNumerator],
b.[freeRedRate],
	  freeRedLunchRateCalc = (case
					when b.extremeLunchProgDataFlag = 'Extreme 10+' then null
					when b.possibleLunchProgDataIssue <> '' then 1
					else b.FreeRedRate end),


b.[possibleLunchProgDataIssue],
b.[extremeLunchProgDataFlag],
b.[TITLEI_STATUS_TEXT],
b.[VIRTUAL_TEXT],
b.[NSLP_STATUS_TEXT],
b.[SHARED_TIME],
b.[schAddressOneLine],
b.[schAddressTwoLinesA],
b.[schAddressTwoLinesB],
b.[LAT],
b.[LON],
b.[OPSTFIPS],
b.[STFIP],
b.[CNTY],
b.[NMCNTY],
b.[LOCALE],
b.[CBSA],
b.[NMCBSA],
b.[CBSATYPE],
b.[CSA],
b.[NMCSA],
b.[NECTA],
b.[NMNECTA],
b.[CD],
b.[SLDL],
b.[SLDU], 
cp.homemadeLeaId, cp.EstTotalPopInDistrict, EstSchoolAgePovertyInDistrict = cp.EstPop5_17PovertyFloat, EstSchoolAgeInDistrict = cp.EstPop5_17Float, EstSchoolAgePovertyInDistrictRate = cp.EstPop5_17PovertyRate,
eft.EnrollmentFloat,
LatinoCount = hl.n, LatinoRate = hl.n/eft.EnrollmentFloat,
BlackCount = bl.n, BlackRate = bl.n/eft.EnrollmentFloat,
WhiteCount = w.n, WhiteRate = w.n/eft.EnrollmentFloat,
PacIslCount = pac.n, PacIslRate = pac.n/eft.EnrollmentFloat,
NativeCount = natv.n, NativeRate = natv.n/eft.EnrollmentFloat,
AsianCount = a.n, AsianRate = a.n/eft.EnrollmentFloat,
TwoPlusCount = two.n, TwoPlusRate = two.n/eft.EnrollmentFloat,
NotSpecifiedCount = ns.n

--into [UsSchoolData2019_Compiled]

FROM [Engage].[dbo].[UsSchoolData2019_Base] b

left join EnrollmentFloatTable eft
on eft.NCESSCH = b.NCESSCH

left join NCES_Directory2019 d
on d.NCESSCH = b.NCESSCH

left join CensusPoverty cp
on cp.homemadeLeaId = b.LEAID and cp.[State] = b.ST

left join NCES_Membership2019 m
on m.NCESSCH = b.NCESSCH and m.GRADE = 'adult education' and m.STUDENT_COUNT <> '0' and m.TOTAL_INDICATOR = 'Subtotal 4 - By Grade'

left join RaceEthnicityCounts hl
on hl.NCESSCH = b.NCESSCH and hl.RACE_ETHNICITY = 'Hispanic/Latino'
left join RaceEthnicityCounts bl
on bl.NCESSCH = b.NCESSCH and bl.RACE_ETHNICITY = 'Black or African American'
left join RaceEthnicityCounts w
on w.NCESSCH = b.NCESSCH and w.RACE_ETHNICITY = 'White'
left join RaceEthnicityCounts pac
on pac.NCESSCH = b.NCESSCH and pac.RACE_ETHNICITY = 'Native Hawaiian or Other Pacific Islander'
left join RaceEthnicityCounts natv
on natv.NCESSCH = b.NCESSCH and natv.RACE_ETHNICITY = 'American Indian or Alaska Native'
left join RaceEthnicityCounts a
on a.NCESSCH = b.NCESSCH and a.RACE_ETHNICITY = 'Asian'
left join RaceEthnicityCounts two
on two.NCESSCH = b.NCESSCH and two.RACE_ETHNICITY = 'Two or more races'
left join RaceEthnicityCounts ns
on ns.NCESSCH = b.NCESSCH and ns.RACE_ETHNICITY = 'Not Specified'

 where d.[LEVEL] <>'Adult Education' and d.SCH_TYPE_TEXT = 'Regular School' and d.LEVEL <> 'ungraded' --study inclusion criteria
							--There are 7 campuses with the level "ungraded" and "regular school":
							--1 in AR, 1 in MI, and 5 in MS. They are either adult ed,
							--early childhood, or special needs. Not sure why they were categorized as "regular" when they are not.


--and b.[Enrollment] <> (hl.n + bl.n + w.n + pac.n + natv.n + a.n + two.n + ns.n)
--394 schools have enrollment <> sum of ethnicities.
--Discrepancies range from -3078 to 39. Other than the 12 that just have 0 ethnicity data (see below),
--all of the discrepanices are half the number in the "notSpecified" column. It doesn't matter, because I don't need
--those anyhow. I will just use the ones that are there to calculate a percent and flag the 12 with no breakouts.

 --12 schools representing 6 districts have 0 ethnic/race numbers.
----NCESSCH			SCHID	ST_SCHID
----010336001889	0101889	AL-200-0106
----040012201585	0401585	AZ-4325-5496
----170011904408	1704408	IL-34-049-1120-02-3404911202006
----172901003042	1703042	IL-05-016-2250-17-0501622500001
----172901003043	1703043	IL-05-016-2250-17-0501622500002
----421260002390	4202390	PA-111343603-2390
----421260002391	4202391	PA-111343603-2391
----421260002393	4202393	PA-111343603-2393
----421260002395	4202395	PA-111343603-2395
----421260002399	4202399	PA-111343603-2399
----421260002400	4202400	PA-111343603-2400
----421260004947	4204947	PA-111343603-4947

  --and d.CHARTER_TEXT = 'No' 
 --d.GSHI = 'ug' --in ('pk', 'kg', '01', '02')
 
 --and b.SCH_NAME like '%adult%'
 --and d.G_AE_OFFERED = 'yes'
 --and b.ST = 'vt' and d.[LEVEL] not in ('elementary', 'middle') --it appears that all vt high schools "offer" adult ed, although 
																-- only 9 campuses have adults enrolled and the only one that breaks
																-- into double-digits is Burlington HS (500282000063) with 34
 --and b.ST = 'tx'
 --and b.st <> 'ak'

  --order by 1
  --b.extremeLunchProgDataFlag desc,-- b.possibleLunchProgDataIssue desc, b.freeRedRate desc

```


