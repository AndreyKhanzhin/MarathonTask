#1 Запрос выводит полное имя и код страны зарегистрированных на текущий марафон бегунов.
select distinct u."firstname", u."lastname", r2.countrycode 
from "Registration" r join "Runner" r2 on r2.runnerid = r.runnerid 
join "User" u on r2.email = u."email"
join "RegistrationEvent" re on re.registrationid = r.registrationid 
join "Event" e on re.eventid = e.eventid 
where e.marathonid = '5';

#2 Запрос выводит пол, возраст и событие, в котором участвовал бегун.
select u.firstname, u.lastname, g.gender , EXTRACT(YEAR FROM age(r2.dateofbirth)), e.eventname
from "Event" e join "RegistrationEvent" re on re.eventid = e.eventid 
join "Registration" r on r.registrationid = re.registrationid 
join "Runner" r2 on r2.runnerid = r.runnerid
join "User" u on u.email = r2.email
join "Gender" g on g.genderid = r2.genderid;

#3 Запрос выводит полное имя бегуна, название марафона и события, в которых участвовал бегун.
select (u."firstname"||''|| u."lastname") as FullName,  age(ev."startdatetime", rnr."dateofbirth"), g."gender", m."marathonname", ev."eventname",  c."countryname", 
TO_CHAR((rei."racetime" || ' seconds')::interval, 'HH24:MI:SS') AS race_time,
ROW_NUMBER() OVER (PARTITION BY ev."eventname" ORDER BY rei."racetime" ASC) AS pposition
FROM "Event" ev
INNER JOIN "Marathon" m ON ev."marathonid" = m."marathonid"
inner join "Country" c on m."countrycode" = c."countrycode"
INNER JOIN "RegistrationEvent" rei ON ev."eventid" = rei."eventid"
inner join "Registration" reg on rei."registrationid" = reg."registrationid" 
inner join "Runner" rnr on reg."runnerid" = rnr."runnerid"
inner join "User" u on u."email" = rnr."email"
inner join "Gender" g on rnr."genderid" =  g."genderid" 
WHERE ev."marathonid" = '3' AND rei."racetime" != 0
ORDER BY ev."eventname" asc, g."gender" asc, race_time asc;

select g."gender", m.marathonname , ev.eventname , 
TO_CHAR((rei.racetime || ' seconds')::interval, 'HH24:MI:SS') AS race_time,
ROW_NUMBER() OVER (PARTITION BY ev.eventname ORDER BY rei.racetime ASC) AS pposition
FROM "Event" ev
INNER JOIN "Marathon" m ON ev.marathonid = m.marathonid 
INNER JOIN "RegistrationEvent" rei ON ev.eventid = rei.eventid 
inner join "Registration" reg on rei.registrationid = reg.registrationid 
inner join "Runner" rnr on reg.runnerid = rnr.runnerid 
inner join "Gender" g on rnr.genderid =  g."genderid" 
WHERE ev.marathonid = '3' AND rei.racetime != 0
ORDER BY ev.eventname ASC, g.gender asc, race_time asc;

#4 
select u."firstname", u."lastname", extract(year from age(rnr."dateofbirth")), g."gender", m."marathonname", ev."eventname",  c."countryname", 
TO_CHAR((rei."racetime" || ' seconds')::interval, 'HH24:MI:SS') AS race_time,
ROW_NUMBER() OVER (PARTITION BY ev."eventname" ORDER BY rei."racetime" ASC) AS pposition
FROM "Event" ev
INNER JOIN "Marathon" m ON ev."marathonid" = m."marathonid"
inner join "Country" c on m."countrycode" = c."countrycode"
INNER JOIN "RegistrationEvent" rei ON ev."eventid" = rei."eventid"
inner join "Registration" reg on rei."registrationid" = reg."registrationid" 
inner join "Runner" rnr on reg."runnerid" = rnr."runnerid"
inner join "User" u on u."email" = rnr."email"
inner join "Gender" g on rnr."genderid" =  g."genderid" 
WHERE ev."marathonid" = '3' AND rei."racetime" != 0
ORDER BY ev."eventname" asc, g."gender" asc, race_time asc;

#5 Запрос выводит список бегунов, зарегистрированных на текущий марафон.
select distinct u.firstname, u.lastname, rnr.email, rgs.registrationstatus
from "User" u
inner JOIN "Role" r on u.roleid = r.roleid
inner join "Runner" rnr on u.email = rnr.email
inner join "Registration" reg on rnr.runnerid = reg.runnerid
inner join "RegistrationEvent" rei on reg."registrationid" = rei.registrationid
inner join "Event" ev on rei."eventid" = ev."eventid"
inner join "RegistrationStatus" rgs on reg.registrationstatusid = rgs.registrationstatusid
where ev.eventtypeid = 'FM' and rgs.registrationstatusid = 1

select count(*) from (select distinct u.firstname, u.lastname, rnr.email, rgs.registrationstatus
from "User" u
inner JOIN "Role" r on u.roleid = r.roleid
inner join "Runner" rn on u.email = rn.email
inner join "Registration" re on rn.runnerid = re.runnerid
inner join "RegistrationEvent" re on reg."registrationid" = re.registrationid
inner join "Event" e on re."eventid" = e."eventid"
inner join "RegistrationStatus" rs on re.registrationstatusid = rs.registrationstatusid
where e.eventtypeid = 'FM' and rs.registrationstatusid = 1) as asd;

#6 Запрос выводит имена благотворительных организаций, всех спонсоров этих организаций и общую сумму пожертвований.
select c."charityname" as charity, count(distinct s."sponsorshipid") as all_sponsors, sum(s."amount") AS total_cash                    
from "Charity" c 
join "Registration" r on c."charityid" = r."charityid"
join "Sponsorship" s on r."registrationid" = s."registrationid"
group by c."charityname"  
order by charity;

#7 Запрос проверяет пароль бегуна на соблюдение условий, а также возраст бегуна.
select u."password", EXTRACT(YEAR from age(r.dateofbirth)) as "Age"
from "User" u join "Runner" r on u.email = r.email
where u.roleid = 'R'
and
u."password" like '______%'
and u."password" ~ '[a-z]'
and u."password" ~ '[0-9]'
and u."password" ~ '[!@#$%^]'
and EXTRACT(YEAR FROM AGE(r.dateofbirth)) > 10;
