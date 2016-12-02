# Deans-List-Queries-for-Tracker
Queries from DeansList using PS language

Overlapping OSS - either starting on the same day or overlapping dates 

```SQL
select 
i.studentschoolid as student_number
,i.schoolid as school_name
,i.incidentid as error_group
,p.startdate as error_date
,cal.yearid as yearid
,i.gradelevelshort as grade_level
,s.lastfirst as lastfirst
,'Overlapping OSS' as error, 
'DeansList' as sourcesystem,
1 ERRORID 
from
custom.custom_dlincidents_raw i 
left join custom.custom_dlpenalties_raw p on p.incidentid = i.incidentid
join custom.custom_dlschoolbridge sb on sb.dlschoolid = i.schoolid
join powerschool.powerschool_schools sch on sch.school_number = sb.psschoolid
join powerschool.powerschool_students s on s.student_number = i.studentschoolid
JOIN (SELECT DISTINCT
        CASE 
            WHEN DATEPART(MM,CD.DATE_VALUE)>=7 THEN RIGHT(DATEPART(YY,CD.DATE_VALUE),2)+10
            WHEN DATEPART(MM,CD.DATE_VALUE)<=6 THEN RIGHT(DATEPART(YY,CD.DATE_VALUE),2)+9
            ELSE NULL 
        END YEARID
        ,DATE_VALUE
        FROM POWERSCHOOL.POWERSCHOOL_CALENDAR_DAY CD
        ) CAL ON CAL.DATE_VALUE = I.CREATETS
inner join(select penaltyname, startdate, incidentid, enddate, studentschoolid
    from custom.custom_dlpenalties_raw p 
    where penaltyname = 'OSS') sub on
    sub.studentschoolid = i.studentschoolid 
    and sub.incidentid != i.incidentid 
    and p.startdate = sub.startdate
    and p.enddate = sub.enddate

where p.penaltyname = 'OSS'

union all
```

1 incident with OSS and expulsion - based on Incident ID

```SQL
select 
i.studentschoolid as student_number
,i.schoolid as school_name
,i.incidentid as error_group 
,i.issuets as error_date
,cal.yearid as yearid
,i.gradelevelshort as grade_level
,s.lastfirst as lastfirst
,'1 incident ID with multiple incidents' as error
,'DeansList' as sourcesystem
,count(i.incidentid) as numoccurences,
2 errorid 
from
custom.custom_dlincidents_raw i 
left join custom.custom_dlpenalties_raw p on p.incidentid = i.incidentid
join custom.custom_dlschoolbridge sb on sb.dlschoolid = i.schoolid
join powerschool.powerschool_schools sch on sch.school_number = sb.psschoolid
join powerschool.powerschool_students s on s.student_number = i.studentschoolid
JOIN (SELECT DISTINCT
        CASE 
            WHEN DATEPART(MM,CD.DATE_VALUE)>=7 THEN RIGHT(DATEPART(YY,CD.DATE_VALUE),2)+10
            WHEN DATEPART(MM,CD.DATE_VALUE)<=6 THEN RIGHT(DATEPART(YY,CD.DATE_VALUE),2)+9
            ELSE NULL 
        END YEARID
        ,DATE_VALUE
        FROM POWERSCHOOL.POWERSCHOOL_CALENDAR_DAY CD
        ) CAL ON CAL.DATE_VALUE = I.CREATETS
     where penaltyname in ('OSS', 'Expulsion')
     group by i.incidentid, i.studentschoolid
     having (count(i.incidentid) > 1)

union all 
```

Referrals more than 30 days old that have not been resolved
```SQL
select 
i.studentschoolid as student_number
,i.schoolid as school_name
,i.incidentid as error_group
,i.CreateTS as error_date 
,cal.yearid as yearid
,i.gradelevelshort as grade_level
,s.lastfirst as lastfirst
,'Unresolved referrals older than 30 days' as error
,'DeansList' as sourcesystem,
3 errorid
from
custom.custom_dlincidents_raw i 
join custom.custom_dlschoolbridge sb on sb.dlschoolid = i.schoolid
join powerschool.powerschool_schools sch on sch.school_number = sb.psschoolid
join powerschool.powerschool_students s on s.student_number = i.studentschoolid
JOIN (SELECT DISTINCT
        CASE 
            WHEN DATEPART(MM,CD.DATE_VALUE)>=7 THEN RIGHT(DATEPART(YY,CD.DATE_VALUE),2)+10
            WHEN DATEPART(MM,CD.DATE_VALUE)<=6 THEN RIGHT(DATEPART(YY,CD.DATE_VALUE),2)+9
            ELSE NULL 
        END YEARID
        ,DATE_VALUE
        FROM POWERSCHOOL.POWERSCHOOL_CALENDAR_DAY CD
        ) CAL ON CAL.DATE_VALUE = I.CREATETS
    where isreferral = 'True'
    and i.CloseTS IS NULL
    and i.CreateTS < '10-OCT-2016'

union all 
```

Missing infraction
```SQL
select 
i.studentschoolid as student_number
,i.schoolid as school_name
,i.incidentid as error_group
,i.issuets as error_date
,cal.yearid as yearid
,i.gradelevelshort as grade_level
,s.lastfirst as lastfirst
,'Missing Infraction' as error
,'DeansList' as sourcesystem,
4 errorid
from custom.custom_dlincidents_raw i
join custom.custom_dlschoolbridge sb on sb.dlschoolid = i.schoolid
join powerschool.powerschool_schools sch on sch.school_number = sb.psschoolid
join powerschool.powerschool_students s on s.student_number = i.studentschoolid
JOIN (SELECT DISTINCT
        CASE 
            WHEN DATEPART(MM,CD.DATE_VALUE)>=7 THEN RIGHT(DATEPART(YY,CD.DATE_VALUE),2)+10
            WHEN DATEPART(MM,CD.DATE_VALUE)<=6 THEN RIGHT(DATEPART(YY,CD.DATE_VALUE),2)+9
            ELSE NULL 
        END YEARID
        ,DATE_VALUE
        FROM POWERSCHOOL.POWERSCHOOL_CALENDAR_DAY CD
        ) CAL ON CAL.DATE_VALUE = I.CREATETS
where infraction IS NULL

union all

```

Missing injury type 
```SQL
select 
i.studentschoolid as student_number
,i.schoolid as school_name
,i.incidentid as error_group
,i.issuets as error_date
,cal.yearid as yearid
,i.gradelevelshort as grade_level
,s.lastfirst as lastfirst
,'Missing injury type' as error
,'DeansList' as sourcesystem,
5 errorid
from custom.custom_dlincidents_raw i
join custom.custom_dlschoolbridge sb on sb.dlschoolid = i.schoolid
join powerschool.powerschool_schools sch on sch.school_number = sb.psschoolid
join powerschool.powerschool_students s on s.student_number = i.studentschoolid
JOIN (SELECT DISTINCT
        CASE 
            WHEN DATEPART(MM,CD.DATE_VALUE)>=7 THEN RIGHT(DATEPART(YY,CD.DATE_VALUE),2)+10
            WHEN DATEPART(MM,CD.DATE_VALUE)<=6 THEN RIGHT(DATEPART(YY,CD.DATE_VALUE),2)+9
            ELSE NULL 
        END YEARID
        ,DATE_VALUE
        FROM POWERSCHOOL.POWERSCHOOL_CALENDAR_DAY CD
        ) CAL ON CAL.DATE_VALUE = I.CREATETS
where injurytype IS NULL 
and infraction in ('Bullying', 'Fighting', 'Sexual Misconduct or Harrassment', 
'Theft', 'Threatening Physical Harm', 'Violent Incident (WITH physical injury) (VIOWINJ)')

union all
```

