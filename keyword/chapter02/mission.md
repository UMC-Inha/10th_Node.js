select *
from book b
left join book_category bc
on bc.id = b.book_category_id
where bk.name = "소설"

select m.name, nickname, ..., count(*) as rent_num
from member m
left join rent r
on m.id =  r.member_id
group by m.id
order by rent_num desc
limit 5

select m.name, bc.name, count(*) as book_count
from member m
join rent r on m.id = r.member_id
join book b on b.id = r.book_id
join book_category bc on b.book_category_id = bc.id
group by m.id, bc.id

SELECT 
    m.name, 
    bc.name AS category, 
    COUNT(bl.id) AS like_count
FROM member m
JOIN book_likes bl ON m.id = bl.member_id
JOIN book b ON bl.book_id = b.id
JOIN book_category bc ON b.book_category_id = bc.id
GROUP BY m.id, m.name, bc.id, bc.name
ORDER BY m.name, like_count DESC;