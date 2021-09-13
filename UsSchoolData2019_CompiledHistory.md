The table called UsSchoolData2019_Compiled went through several iterations before becoming the final thing. (See UsSchoolData2019_CompiledOLD for first version.)

This was used to add NCES descriptions (UsSchoolData2019_CompiledOLD2)...

```tsql
SELECT 
[NCESSCH]
      ,[SCHID]
      ,[ST_SCHID]
      ,[SCH_NAME]
      ,[LEAID]
      ,[ST_LEAID]
      ,[LEA_NAME]
      ,[WEBSITE]
      ,[FIPST]
      ,[STATE_AGENCY_NO]
      ,[ST]
      ,[STATENAME]
      ,[charter]
      ,[adultsServedN]
      ,[SCH_TYPE_TEXT]
      ,[SchoolLevel]
      ,[gradesServed]
      ,[Enrollment]
      ,[ethnTotalFlag]
      ,[LunchNumerator]
      ,[freeRedRate]
      ,[freeRedLunchRateCalc]
      ,[possibleLunchProgDataIssue]
      ,[extremeLunchProgDataFlag]
      ,[TITLEI_STATUS_TEXT]
      ,[VIRTUAL_TEXT]
      ,[NSLP_STATUS_TEXT]
      ,[SHARED_TIME]
      ,[schAddressOneLine]
      ,[schAddressTwoLinesA]
      ,[schAddressTwoLinesB]
      ,[LAT]
      ,[LON]
      ,[OPSTFIPS]
      ,[STFIP]
      ,[CNTY]
      ,[NMCNTY]
      ,[LOCALE],
	  NcesDescr = (case
				when c.locale = '11' then 'City Large'
				when c.locale = '12' then 'City Midsize'
				when c.locale = '13' then 'City Small'
				when c.locale = '21' then 'Suburban Large'
				when c.locale = '22' then 'Suburban Midsize'
				when c.locale = '23' then 'Suburban Small'
				when c.locale = '31' then 'Town Fringe'
				when c.locale = '32' then 'Town Distant'
				when c.locale = '33' then 'Town Remote'
				when c.locale = '41' then 'Rural Fringe'
				when c.locale = '42' then 'Rural Distant'
				when c.locale = '43' then 'Rural Remote'
				else '' end)

      ,[CBSA]
      ,[NMCBSA]
      ,[CBSATYPE]
      ,[CSA]
      ,[NMCSA]
      ,[NECTA]
      ,[NMNECTA]
      ,[CD]
      ,[SLDL]
      ,[SLDU]
      ,[homemadeLeaId]
      ,[EstTotalPopInDistrict]
      ,[EstSchoolAgePovertyInDistrict]
      ,[EstSchoolAgeInDistrict]
      ,[EstSchoolAgePovertyInDistrictRate]
      ,[EnrollmentFloat]
      ,[LatinoCount]
      ,[LatinoRate]
      ,[BlackCount]
      ,[BlackRate]
      ,[WhiteCount]
      ,[WhiteRate]
      ,[PacIslCount]
      ,[PacIslRate]
      ,[NativeCount]
      ,[NativeRate]
      ,[AsianCount]
      ,[AsianRate]
      ,[TwoPlusCount]
      ,[TwoPlusRate]
      ,[NotSpecifiedCount]
	  --into UsSchoolData2019_Compiled
  FROM [Engage].[dbo].[UsSchoolData2019_CompiledOLD] c
  ```
  
 ------------------
 
  This was used to add population estimates (UsSchoolData2019_CompiledOLD3)
  
```tsql
with crosswalk as (
select distinct CBSACode, CBSATitle from CBSACrosswalk5 cr
)

SELECT c.[NCESSCH]
      ,c.[SCHID]
      ,c.[ST_SCHID]
      ,c.[SCH_NAME]
      ,c.[LEAID]
      ,c.[ST_LEAID]
      ,c.[LEA_NAME]
      ,c.[WEBSITE]
      ,c.[FIPST]
      ,c.[STATE_AGENCY_NO]
      ,c.[ST]
      ,c.[STATENAME]
      ,c.[charter]
      ,c.[adultsServedN]
      ,c.[SCH_TYPE_TEXT]
      ,c.[SchoolLevel]
      ,c.[gradesServed]
      ,c.[Enrollment]
      ,c.[ethnTotalFlag]
      ,c.[LunchNumerator]
      ,c.[freeRedRate]
      ,c.[freeRedLunchRateCalc]
      ,c.[possibleLunchProgDataIssue]
      ,c.[extremeLunchProgDataFlag]
      ,c.[TITLEI_STATUS_TEXT]
      ,c.[VIRTUAL_TEXT]
      ,c.[NSLP_STATUS_TEXT]
      ,c.[SHARED_TIME]
      ,c.[schAddressOneLine]
      ,c.[schAddressTwoLinesA]
      ,c.[schAddressTwoLinesB]
      ,c.[LAT]
      ,c.[LON]
      ,c.[OPSTFIPS]
      ,c.[STFIP]
      ,c.[CNTY]
      ,c.[NMCNTY]
      ,c.[LOCALE]
      ,c.[NcesDescr]
      ,c.[CBSA], cr.CBSACode
      ,c.[NMCBSA], cr.CBSATitle, p.TrimNmsbsa, p.est2019, CbsaPopEst2019 = cast(REPLACE( p.est2019,',','') as float)-- CbsaPopEst2019 = cast(p.est2019 as float)
      ,c.[CBSATYPE]
      ,c.[CSA]
      ,c.[NMCSA]
      ,c.[NECTA]
      ,c.[NMNECTA]
      ,c.[CD]
      ,c.[SLDL]
      ,c.[SLDU]
      ,c.[homemadeLeaId]
      ,c.[EstTotalPopInDistrict]
      ,c.[EstSchoolAgePovertyInDistrict]
      ,c.[EstSchoolAgeInDistrict]
      ,c.[EstSchoolAgePovertyInDistrictRate]
      ,c.[EnrollmentFloat]
      ,c.[LatinoCount]
      ,c.[LatinoRate]
      ,c.[BlackCount]
      ,c.[BlackRate]
      ,c.[WhiteCount]
      ,c.[WhiteRate]
      ,c.[PacIslCount]
      ,c.[PacIslRate]
      ,c.[NativeCount]
      ,c.[NativeRate]
      ,c.[AsianCount]
      ,c.[AsianRate]
      ,c.[TwoPlusCount]
      ,c.[TwoPlusRate]
      ,c.[NotSpecifiedCount]
	  --select  c.*
	  --into [UsSchoolData2019_Compiled]
  FROM [Engage].[dbo].[UsSchoolData2019_CompiledOLD2] c

left join crosswalk cr
on c.cbsa = cr.CBSACode 

left join [Engage].[dbo].[NCES_CbsaPopEst2010_2019] p --source: Combined Statistical Area Population Estimates and Estimated Components of Change: April 1, 2010 to July 1, 2019 (CSA-EST2019-alldata); https://www.census.gov/data/tables/time-series/demo/popest/2010s-total-metro-and-micro-statistical-areas.html
on p.trimnmsbsa = concat(cr.cbsatitle, ' Metro Area')

--order by 3 desc

--where c.[CBSA] <> 'n'
--and CBSATYPE = '1'
--and TrimNmsbsa is null
--where --c.cbsa = '19430' -- 
--c.NMCBSA like '%Dayton%' and st = 'oh'

--order by CBSATYPE

```

---------------------

This was used to add test scores (UsSchoolData2019_Compiled final)

```tsql

SELECT  
	   c.[NCESSCH]
      ,c.[SCHID]
      ,c.[ST_SCHID]
      ,c.[SCH_NAME]
      ,c.[LEAID]
      ,c.[ST_LEAID]
      ,c.[LEA_NAME]
      ,c.[WEBSITE]
      ,c.[FIPST]
      ,c.[STATE_AGENCY_NO]
      ,c.[ST]
      ,c.[STATENAME]
      ,c.[charter]
      ,c.[adultsServedN]
      ,c.[SCH_TYPE_TEXT]
      ,c.[SchoolLevel]
      ,c.[gradesServed]
      ,c.[Enrollment]
      ,c.[ethnTotalFlag]
      ,c.[LunchNumerator]
      ,c.[freeRedRate]
      ,c.[freeRedLunchRateCalc]
      ,c.[possibleLunchProgDataIssue]
      ,c.[extremeLunchProgDataFlag]
      ,c.[TITLEI_STATUS_TEXT]
      ,c.[VIRTUAL_TEXT]
      ,c.[NSLP_STATUS_TEXT]
      ,c.[SHARED_TIME]
      ,c.[schAddressOneLine]
      ,c.[schAddressTwoLinesA]
      ,c.[schAddressTwoLinesB]
      ,c.[LAT]
      ,c.[LON]
      ,c.[OPSTFIPS]
      ,c.[STFIP]
      ,c.[CNTY]
      ,c.[NMCNTY]
      ,c.[LOCALE]
      ,c.[NcesDescr]
      ,c.[CBSA]
      ,c.[CBSACode]
      ,c.[NMCBSA]
      ,c.[CBSATitle]
      ,c.[TrimNmsbsa]
      ,c.[est2019]
      ,c.[CbsaPopEst2019]
      ,c.[CBSATYPE]
      ,c.[CSA]
      ,c.[NMCSA]
      ,c.[NECTA]
      ,c.[NMNECTA]
      ,c.[CD]
      ,c.[SLDL]
      ,c.[SLDU]
      ,c.[homemadeLeaId]
      ,c.[EstTotalPopInDistrict]
      ,c.[EstSchoolAgePovertyInDistrict]
      ,c.[EstSchoolAgeInDistrict]
      ,c.[EstSchoolAgePovertyInDistrictRate]
      ,c.[EnrollmentFloat]
      ,c.[LatinoCount]
      ,c.[LatinoRate]
      ,c.[BlackCount]
      ,c.[BlackRate]
      ,c.[WhiteCount]
      ,c.[WhiteRate]
      ,c.[PacIslCount]
      ,c.[PacIslRate]
      ,c.[NativeCount]
      ,c.[NativeRate]
      ,c.[AsianCount]
      ,c.[AsianRate]
      ,c.[TwoPlusCount]
      ,c.[TwoPlusRate]
      ,c.[NotSpecifiedCount],

	   rdgTestPart2019 = t.[readingNumValidFloat]
      ,rdgTestProfRate2019 = t.[readingPctProfFloat]
      ,mathTestPart2019 = t.[mathNumValidFloat]
      ,mathTestProfRate2019 = t.[mathPctProfFloat]
	  --select * 
	  --into UsSchoolData2019_Compiled
  FROM [Engage].[dbo].[UsSchoolData2019_CompiledOLD3] c

  left join NationalTestScores2019 t --(See NationwidePassRatesSQL)
  on t.ncessch = c.NCESSCH
  
  ```
