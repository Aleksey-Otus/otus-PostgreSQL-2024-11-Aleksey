<br><b> Выполнение</b>

<br>• Подключились к ВМ с Postgresql, созданной в Yandex Cloud, к созданной ранее базе demo. База была скачена в рамках проектной работы.


<br>• Для создания различных соединений использовались таблицы из базы demo. Диаграмма таблиц:
<br>

<img width="1487" height="849" alt="1" src="https://github.com/user-attachments/assets/76a33faa-d702-4764-9c53-3d7a06632a91" />


<br>• Задание: Реализовать прямое соединение двух или более таблиц.
В данном запросе выводится информация только по тем аэропортам, которые участвуют в полётах:
<br>

```
select	F.flight_no,
		F.departure_airport,
		D.airport_name,
		D.city,		
		F.arrival_airport,
		A.airport_name,
		A.city 
from bookings.flights F
  join bookings.airports_data A on F.arrival_airport=A.airport_code
  join bookings.airports_data D on F.departure_airport=D.airport_code;

```

<img width="1703" height="860" alt="2" src="https://github.com/user-attachments/assets/e4632dfe-c4eb-41ca-b410-2131b10dcee0" />


<br>• Задание: Реализовать левостороннее (или правостороннее) соединение двух или более таблиц.
В данном запросе выводится информация только по тем полётах, на которые нет билетов:
<br>

```
select F.* 
from bookings.flights F
  left join bookings.ticket_flights T on F.flight_id=T.flight_id
where T.flight_id is null;

```


<img width="1698" height="858" alt="3" src="https://github.com/user-attachments/assets/c87f56b7-3743-42df-9318-656097ae4c66" />

<br>• Задание: Реализовать кросс соединение двух или более таблиц.
В данном запросе выводится полная информация из двух таблиц, т.е. объединение первой строки первой таблицы с каждой строкой второй таблицы:
<br>

```
select * 
from bookings.aircrafts_data
  cross join bookings.airports_data;

```


<img width="1693" height="858" alt="4" src="https://github.com/user-attachments/assets/2229f37d-0ebb-4e10-b1ea-a888b7b63057" />


<br>• Задание: Реализовать полное соединение двух или более таблиц.
Данный запрос выведет все совпадающие элементы, а также все несовпадающие элементы как из одной таблицы, так и из другой:
<br>

```
select * 
from bookings.flights F
  full join bookings.aircrafts_data A on F.aircraft_code=A.aircraft_code;

```

<img width="1693" height="857" alt="5" src="https://github.com/user-attachments/assets/feb2ae59-337a-4028-b794-a8da1f40f052" />


<br>• Задание: Реализовать полное соединение двух или более таблиц.
Реализован итоговый запрос по таблицам из предметной области полёты. Запрос отображает дынне о всех полётах с указанием доступных мест в самолёте (qnt_seats) и о количестве пассажиров (qnt_boarding_passes):
<br>

```
select	F.flight_id,
    	F.flight_no,
    	F.status,
    	D.airport_name as departure_airport_name, 
    	A.airport_name as arrival_airport_name,
    	F.scheduled_departure,
    	timezone(A.timezone, F.scheduled_arrival) as scheduled_arrival_local,
    	F.scheduled_arrival - F.scheduled_departure as scheduled_duration,
    	S.qnt_seats,
    	T.qnt_boarding_passes,
    	F.aircraft_code,
    	C.model
from bookings.flights F
  cross join lateral (select count(*) as qnt_seats from bookings.seats where aircraft_code=F.aircraft_code) S
  cross join lateral (select count(*) as qnt_boarding_passes from bookings.boarding_passes where flight_id=F.flight_id) T
  left join bookings.airports A on F.arrival_airport = A.airport_code
  left join bookings.airports D on F.departure_airport = D.airport_code
  left join bookings.aircrafts_data C on F.aircraft_code = C.aircraft_code
order by T.qnt_boarding_passes desc;

```

<img width="1781" height="930" alt="6" src="https://github.com/user-attachments/assets/c59035b7-415d-4203-bc86-ed5d6152722c" />


