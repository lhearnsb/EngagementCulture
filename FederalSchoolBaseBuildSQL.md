This SQL query was used to join data about all campuses in the US in 2019 and create a base file of all campuses.

Note that there are some discrepancies in the reporting from different states, either because of differences in rules or user error. More detail at bottom.
This query attempts to create an "apples to apples" set of data on campuses throughout the US.

It uses the following datasets:

NCES_Membership2019, aka Membership from https://nces.ed.gov/ccd/files.asp#Fiscal:2,LevelId:7,SchoolYearId:33,Page:1
NCES_Lunch2019, aka Lunch Program Eligibility from https://nces.ed.gov/ccd/files.asp#Fiscal:2,LevelId:7,SchoolYearId:33,Page:1
NCES_Characteristics2019, aka School Characteristics from https://nces.ed.gov/ccd/files.asp#Fiscal:2,LevelId:7,SchoolYearId:33,Page:1
EDGE_Geocode2019, aka 2018-2019 Public School File from https://nces.ed.gov/programs/edge/Geographic/SchoolLocations


```
--This is the base build. The question is... why do some schools have over a 100% free/reduced lunch rate? Should we just use the direct cert if a campus has both?



with adultEd as (select distinct NCESSCH, adultsServed = 'y' from NCES_Membership2019 where grade	= 'Adult Education' and STUDENT_COUNT <> '' and STUDENT_COUNT <> '0'),

freeRedLunch as (select NCESSCH, FreeReducedN = sum(cast(student_count as int)) from Engage.[dbo].[NCES_Lunch2019]
where  (LUNCH_PROGRAM = 'No Category Codes' or DATA_GROUP = 'Direct Certification')
group by NCESSCH),

DirectCert as (
select dc.NCESSCH, DirectCertN = cast(dc.STUDENT_COUNT as float)
from NCES_Lunch2019 dc
where dc.DATA_GROUP = 'direct certification'),

NoCategoryCode as (
select nc.NCESSCH, nc.ST, NoCategoryN = cast(nc.STUDENT_COUNT as float)
from NCES_Lunch2019 nc
where nc.LUNCH_PROGRAM = 'No Category Codes'),

FreeLunchNum as (select nc.ncessch, nc.st,
		LunchNumerator = (case
		when nc.NoCategoryN = dc.DirectCertN then nc.NoCategoryN
		when nc.NoCategoryN > dc.DirectCertN then nc.NoCategoryN
		when dc.DirectCertN > nc.NoCategoryN then dc.DirectCertN
		else nc.NoCategoryN end),

nc.NoCategoryN, dc.DirectCertN from NoCategoryCode nc
left join DirectCert dc on dc.NCESSCH = nc.NCESSCH),

base as (select distinct m.NCESSCH,  
m.ST_SCHID, m.ST_LEAID, m.LEAID, m.SCHID, m.FIPST, m.STATENAME, m.ST, m.SCH_NAME, 
Enrollment = m.STUDENT_COUNT, fr.LunchNumerator, --reportedFreeRedLunch = l.FreeReducedN,
freeRedRate = (cast (fr.LunchNumerator as float)/cast(m.student_count as float)), a.adultsServed

from NCES_Membership2019 m left join freeRedLunch l on l.NCESSCH = m.NCESSCH 
left join adultEd a on a.NCESSCH = m.NCESSCH
left join FreeLunchNum fr on fr.NCESSCH = m.NCESSCH
where m.TOTAL_INDICATOR = 'Education Unit Total' --'Derived - Education Unit Total minus Adult Education Count'
and m.STUDENT_COUNT <> '' and m.STUDENT_COUNT <> '0')
--and m.st <> 'ak'
--order by 14 desc

select distinct   --fl.LunchNumerator,
--lunchEnrollmentDiff = fl.LunchNumerator - b.Enrollment, --d.[LEA_NAME],
b.*, 
			possibleLunchProgDataIssue = (case
			when  (fl.LunchNumerator - b.Enrollment) > 1 then concat ((fl.LunchNumerator - b.Enrollment), ' more free lunches than students')
			when  (fl.LunchNumerator - b.Enrollment) = 1 then concat ((fl.LunchNumerator - b.Enrollment), ' more free lunch than students')
			else '' end), 

			extremeLunchProgDataFlag =  (case
			when  (fl.LunchNumerator - b.Enrollment) >= 10 then 'Extreme 10+'
			else '' end), c.TITLEI_STATUS_TEXT, c.VIRTUAL_TEXT, c.NSLP_STATUS_TEXT, c.SHARED_TIME,
				   schAddressOneLine = concat(g.STREET, ' | ', g.CITY, ', ', g.[STATE], ' ', g.ZIP),
	   schAddressTwoLinesA = g.street,
	   schAddressTwoLinesB = concat(g.CITY, ', ', g.[STATE], ' ', g.ZIP)
	  ,g.[LAT]
      ,g.[LON]
      ,g.[OPSTFIPS]
      ,g.[STFIP]
      ,g.[CNTY]
      ,g.[NMCNTY]
      ,g.[LOCALE]
      ,g.[CBSA]
      ,g.[NMCBSA]
      ,g.[CBSATYPE]
      ,g.[CSA]
      ,g.[NMCSA]
      ,g.[NECTA]
      ,g.[NMNECTA]
      ,g.[CD]
      ,g.[SLDL]
      ,g.[SLDU]

--into UsSchoolData2019_Base
from base b 

--left join NCES_Lunch2019 ln
--on ln.NCESSCH = b.NCESSCH
left join FreeLunchNum fl
on fl.NCESSCH = b.NCESSCH

left join NCES_Characteristics2019 c
on c.ncessch = b.NCESSCH

left join EDGE_Geocode2019 g
on g.NCESSCH = b.NCESSCH

--left join [dbo].[NCES_Directory2019] d
--on d.ncessch = b.NCESSCH

--where b.st_schid = 'TX-105902-105902001'
--where (fl.LunchNumerator - b.Enrollment) > 0 --and b.ST <> 'ak'

order by 1

--66 schools have more free/reduced lunch recipients than students enrolled.
----53 of these are from Alaska. The 13 other represent Floria (1); Guam (2); Utah (8); and Washington (2)
----Most (34) are off by 1 or 2. Eight are off by 3. Two are off by 4. Twelve are off by between 5 and 9.
----Ten are off by 11 or more. All of these are from Alaska.
----Some are extreme. Fairview Elementary (020018000076) in Anchorage School District is over by 113.
------These unexpected numbers are verified using the NCES map here: https://data-nces.opendata.arcgis.com/datasets/public-school-characteristics-2018-19?geometry=159.474%2C62.072%2C-140.760%2C68.471&orderBy=OBJECTID&orderByAsc=false&selectedAttribute=LEAID
------It is unclear why Alaska has these discrepanices. 
------According to https://www.thedailymeal.com/eat/free-school-lunch-breakfast-every-state/slide-7, 
------Alaska has the same qualification requirements, although they are reimbursed more per meal due to the cost of food
--------"The NSLP reimburses Alaska schools for meals at a higher rate than any other state because of higher food costs on 
---------the Last Frontier — in 2018-19, schools in the state received $5.38 for each free meal served, as compared to $3.31 
---------in the 48 contiguous states. Eligibility rules for students, however, match the federal standard."
```
