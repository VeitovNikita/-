/* Запрос №1
 Список пациентов в алфавитном порядке старше 18 лет,
 которые были записаны на первую половину сентября с указанием ФИО в одном столбце,
 датой приёма в формате дд.мм.гггг, ФИО врача, указанием в виде текста, 
 были ли оказаны услуги. Столбцам присвоить смысловые названия.
 */

select 
    concat(p.LastName, ' ', p.FirstName, ' ', p.SecondName, ' ') as "ФИО пациента",
    to_char(v.datapriema, 'dd.mm.yyyy') as "Дата приёма",
    u.name as "ФИО врача",
    servicename as "Наименование услуги",
    case 
        when us.done = 1 then 'Услуги оказаны'
        else 'Услуги не оказаны'
    end as "Статус услуг"
from patients p
join visits v on p.id = v.patid
join users u on v.doctorid = u.id
left join usluga us on v.id = us.masterid 
where 
    extract(month from v.datapriema) = 9
    and extract(day from v.datapriema) between 1 and 15
    and date_part('year', age(current_date, p.birthdate)) > 18
order by "ФИО пациента"


/* Запрос №2
Список врачей, к которым были записаны пациенты на терапевтическое или хирургическое лечение,
с указанием объема денег, которые были получены за оказанные услуги этими врачами, 
и сколько еще денег было бы получено от пациентов, если бы оставшиеся услуги были оказаны. 
Столбцам присвоить смысловые названия.
 */

select 
    u.name as "ФИО врача",
    coalesce(sum(case when us.done = 1 then pd.itogo else 0 end), 0) as "Получено денег",
    coalesce(sum(case when us.done = 0 then us.BasePrice else 0 end), 0) as "Потенциальный доход",
      count(us.id) as "Всего услуг",
      count(case when us.done = 1 then 1 end) as "Оказано услуг",
      count(CASE WHEN us.done = 0 then 1 end) as "Не оказано услуг"
from users u 
join visits v on u.ID = v.doctorid
join usluga us on v.ID = us.masterid
left join paydocs pd on v.ID = pd.visitid
where (
    us.servicename ilike '%терап%' 
    OR us.servicename ilike '%хирург%'
)
group by u.id, u.name
having count(us.id) > 0
order by "Получено денег" desc
