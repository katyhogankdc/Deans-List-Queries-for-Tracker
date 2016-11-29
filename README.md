# Deans-List-Queries-for-Tracker
Queries from DeansList using PS language

Overlapping OSS - either starting on the same day or overlapping dates 

```SQL
select 
p.penaltyname as error_group
,i.incidentid 
,schoolid as school_name
,p.startdate as error_date
,p.enddate 
,i.studentschoolid as student_number
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
i.incidentid as error_group 
,i.studentschoolid as student_number
,i.schoolid as school_name
,i.gradelevelshort as grade_level
,i.issuets as error_date
,sch.abbreviation as school_name 
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
i.category
,i.incidentid
,i.isreferral
,i.CreateTS
,i.CloseTS
from
custom.custom_dlincidents_raw i 
	where isreferral = 'True'
	and i.CloseTS IS NULL
	and i.CreateTS < '10-OCT-2016'
```

Missing infraction
```SQL
select 
infraction
,incidentid
,studentschoolid
from custom.custom_dlincidents_raw
where infraction IS NULL
```

Missing injury type 
```SQL
select 
injurytype
,infraction
,studentschoolid
,incidentid
from custom.custom_dlincidents_raw
where injurytype IS NULL 
and infraction in ('Bullying', 'Fighting', 'Sexual Misconduct or Harrassment', 
'Theft', 'Threatening Physical Harm', 'Violent Incident (WITH physical injury) (VIOWINJ)')
```

