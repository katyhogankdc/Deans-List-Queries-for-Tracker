# Deans-List-Queries-for-Tracker
Queries from DeansList using PS language

Overlapping OSS - either starting on the same day or overlapping dates 

```SQL
select 
i.studentschoolid as student_number
,schoolid as school_name
,p.penaltyname as error_group
,p.startdate as error_date
,cal.yearid
,i.gradelevelshort as grade_level
,s.lastfirst as lastfirst
,i.incidentid 
,p.enddate 
,'Overlapping OSS' as error, 
'DeansList' as sourcesystem
1 ERRORID 
from
custom.custom_dlincidents_raw i 
left join custom.custom_dlpenalties_raw p on p.incidentid = i.incidentid
join custom_dlschoolbridge sb on sb.dlschoolid = i.schoolid
join powerschool.powerschool_schools sch on sch.school_number = sb.psschoolid
join powerschool.powerschool_students s on s.student_number = i.studentschoolid
    and p.startdate = p.startdate
    and p.enddate = p.enddate
    where penaltyname = 'OSS'
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
,cal.yearid
,i.gradelevelshort as grade_level
,s.lastfirst as lastfirst
,'1 incident ID with multiple incidents' as error
,'DeansList' as sourcesystem
,count(i.incidentid) as numoccurences
2 error id 
from
custom.custom_dlincidents_raw i 
left join custom.custom_dlpenalties_raw p on p.incidentid = i.incidentid
join custom_dlschoolbridge sb on sb.dlschoolid = i.schoolid
join powerschool.powerschool_schools sch on sch.schoolnumber = sb.psschoolid
join powerschool.powerschool_students s on s.student_number = i.studentschoolid
     where penaltyname in ('OSS', 'Expulsion')
     group by i.incidentid, i.studentschoolid
     having (count(i.incidentid) > 1)

union all 
```

Referrals more than 30 days old that have not been resolved
```SQL
select 
i.studentschoolid as student_number
,sch.abbreviation as school_name
,i.category
,i.incidentid as error_group
,i.isreferral
,i.CreateTS as error_date 
,cal.yearid
,i.gradelevelshort as grade_level
,s.lastfirst as lastfirst
,i.CloseTS
,'Unresolved referrals older than 30 days' as error
,'DeansList' as sourcesystem
3 errorid
from
custom.custom_dlincidents_raw i 
    where isreferral = 'True'
    and i.CloseTS IS NULL
    and i.CreateTS < '10-OCT-2016'
```

Missing infraction
```SQL
select 
i.studentschoolid as student_number
,sch.abbreviation as school_name
,i.infraction as error_group
,i.issuets as error_date
,cal.yearid
,i.gradelevelshort as grade_level
,s.lastfirst as lastfirst
,'Missing Infraction' as error
,'DeansList' as sourcesystem
,incidentid
4 errorid
from custom.custom_dlincidents_raw
where infraction IS NULL
```

Missing injury type 
```SQL
select 
i.studentschoolid as student_number
,sch.abbreviation as school_name
,i.injurytype as error_group
,i.issuets as error_date
,cal.yearid
,i.gradelevelshort as grade_level
,s.lastfirst as lastfirst
,i.infraction
,i.incidentid
,'Missing injury type' as error
,'DeansList' as sourcesystem
5 errorid
from custom.custom_dlincidents_raw
where injurytype IS NULL 
and infraction in ('Bullying', 'Fighting', 'Sexual Misconduct or Harrassment', 
'Theft', 'Threatening Physical Harm', 'Violent Incident (WITH physical injury) (VIOWINJ)')
```

