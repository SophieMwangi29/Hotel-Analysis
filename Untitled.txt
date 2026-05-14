-- creating the database
create database tembo;

--create schema
create schema tembo_main;

--ensure that all the querries refer to the schema
set search_path to tembo_main;

--confirm which schema is in current use
select current_schema();

create table staging_bookings(
booking_id text,
guest_name text,
guest_phone TEXT,
guest_city TEXT,
guest_nationality TEXT,
room_no TEXT,
room_type TEXT,
room_rate_per_night TEXT,
check_in_date TEXT,
check_out_date TEXT,
nights_stayed TEXT,
staff_name TEXT,
staff_department TEXT,
staff_salary TEXT,
payment_method TEXT,
booking_status TEXT,
total_amount TEXT,
service_used TEXT,
service_price TEXT,
guest_rating TEXT);

select * from staging_bookings;

create table clean_dup as select * from staging_bookings;

drop table clean_dup;

-- finding duplicate if any
select booking_id,  count(*) from clean_dup group by booking_id having count(*)>1;

--ctid is a hidden column identifying a rows specific location on a disk
SELECT ctid, booking_id from clean_dup where booking_id = 'BK0006';

--deleting from one of the ctid
delete from clean_dup where ctid = '(0,10)';

select * from clean_dup;


--adding the unique constraint
ALTER TABLE clean_dup
ADD CONSTRAINT unique_booking_id UNIQUE (booking_id);

--making our booking id a primary key
ALTER TABLE clean_dup
add constraint primary_key_booking_id primary key (booking_id);

-- Standardize the names into title case and verify that the name is trimmed
select
	guest_name,
	initcap(trim(guest_name)) as new_name,
	length(guest_name) as init_len,
	length(initcap(trim(guest_name))) 
from clean_dup;

-- update with the new trimmed name
update clean_dup
set guest_name = initcap(trim(guest_name));

select * from clean_dup;



--cleaning the phone numbers and counterchecking
select guest_phone,concat('0', right(regexp_replace(guest_phone, '[^0-9]','','g'),9)) from clean_dup; 

--Check to confirm if there was any not conforming to number of digits
SELECT 
guest_phone,
REGEXP_REPLACE(guest_phone, '[^0-9]', '', 'g') AS clean_phone
FROM clean_dup
WHERE LENGTH(REGEXP_REPLACE(guest_phone, '[^0-9]', '', 'g')) IN (0);

--update with the new phone number
update clean_dup
set guest_phone = concat('0', right(regexp_replace(guest_phone, '[^0-9]','','g'),9));

--replacing the zero with null

update clean_dup
set guest_phone =case 
	when guest_phone = '0' then null
	else guest_phone 
end ;

-- Clean city, nationality and staff name
update clean_dup
set guest_city = initcap(trim(guest_city)),
	guest_nationality = initcap(trim(guest_nationality)),
	staff_name = initcap(trim(staff_name));

--seeing the available room numbers
--There was nothing much to do with the room numbers
select distinct room_no, count(room_no) from clean_dup group by room_no;


--seeing the room types stated
select distinct room_type from clean_dup;

-- cleaninf room type
update clean_dup
set room_type = initcap(trim(room_type));

update clean_dup
set room_type = 
	case 
		when room_type = 'Dlx' then 'Deluxe'
		when room_type = 'Std' then 'Standard'
	else room_type
end;

alter table clean_dup
add constraint room_type_checker check(room_type in('Standard', 'Deluxe', 'Suite', 'Penthouse'));

--validate the changes
select distinct room_type from clean_dup;


--room rate per night we change the data type
alter table clean_dup
alter column room_rate_per_night type numeric
using room_rate_per_night:: numeric;

--convert to date, check in and check out date

alter table clean_dup
add column new_check_in_date2 date;

update clean_dup
set new_check_in_date2 = 
case 
	when check_in_date ~ '^[0-9]{2}-[0-9]{2}-[0-9]{2}$' then to_date(check_in_date, 'DD-MM-YY')--27-05-24
	when check_in_date ~ '^[0-9]{2}/[0-9]{2}/[0-9]{4}$' then to_date(check_in_date, 'DD/MM/YYYY')--08/04/2024
	when check_in_date ~ '^[0-9]{2}-(1[3-9]|[23][0-9])-[0-9]{4}$' then to_date(check_in_date, 'MM-DD-YYYY')
	when check_in_date ~ '^[0-9]{2}-[0-9]{2}-[0-9]{4}$' then to_date(check_in_date, 'DD-MM-YYYY')--04-10-2024
	else check_in_date:: date
end;

--this is to confirm that the check_in_date is cleaned
select check_in_date, new_check_in_date, new_check_in_date2 from clean_dup;

update clean_dup
set check_in_date = new_check_in_date;

alter table clean_dup
drop column new_check_in_date;

--cleaning the checkout date
update clean_dup
set check_out_date = 
case 
	when check_out_date ~ '^[0-9]{2}-[0-9]{2}-[0-9]{2}$' then to_date(check_out_date, 'DD-MM-YY')--27-05-24
	when check_out_date ~ '^[0-9]{2}/[0-9]{2}/[0-9]{4}$' then to_date(check_out_date, 'DD/MM/YYYY')--08/04/2024
	when check_out_date ~ '^[0-9]{2}-(1[3-9]|[23][0-9])-[0-9]{4}$' then to_date(check_out_date, 'MM-DD-YYYY') -- 30-07-24
	when check_out_date ~ '^[0-9]{2}-[0-9]{2}-[0-9]{4}$' then to_date(check_out_date, 'DD-MM-YYYY')--04-10-2024
	else check_out_date:: date
end;

-- cast dates onto date data type
alter table clean_dup
alter column check_in_date type date
using check_in_date::date;

alter table clean_dup
alter column check_out_date type date
using check_out_date::date;

--see if there are dates that are beating logic
select check_in_date, check_out_date, age(check_out_date,check_in_date) from clean_dup;
select 
	*,
	age(check_out_date,check_in_date)
from clean_dup where (check_out_date-check_in_date)<0 ;

--eliminated the dates that which when subtracted did not make sense
delete from clean_dup where(check_out_date-check_in_date) <0;


--Changing the nights stayed to positive integer
alter table clean_dup
alter column nights_stayed type int
using nights_stayed:: int;

select distinct nights_stayed from clean_dup;
-- from here we found a negative value in the nights stayed

--changing the negatives to positives in the nights stayed column
update clean_dup
set nights_stayed= 
case
		when nights_stayed <0 then  abs(nights_stayed)
		else nights_stayed
end;

-- implementint a constraint to ensure that future nights stayed are >= 0
alter table clean_dup
add constraint positive_nights check (nights_stayed>= 0);

--We have confirmed that the departments are as required
select distinct staff_department from clean_dup;

--next we should add a constraint to ensre that a department can only be values within the stipulated ones
alter table clean_dup
add constraint staff_dept_check 
check(staff_department in ('Front Desk', 'Housekeeping','Restaurant','Security','Management'));

--cleaning staff salary
-- updated our staff_salary columm by replacing all the unwanted characters like KES, with nothing

update clean_dup
set staff_salary=REGEXP_REPLACE(staff_salary, '[^0-9]', '', 'g'); -- we are replacing all characters not within 0-9 with empty string

-- updated all the unavailable values "" as NULL
update clean_dup
set staff_salary=NULL
where staff_salary = '';

-- finally changed the staff_salary column datatype to numeric

alter table clean_dup
alter column staff_salary type numeric
using staff_salary::numeric;


-- Cleaning payment_method-How guest paid (M-Pesa / Cash / Card / Bank Transfer) mpesa M-pesa

-- how many payment methods are there
select distinct payment_method from clean_dup;

--From here we noticed there is mpesa and M-pesa so we have to make them alike
select payment_method,count(*) from clean_dup
group by payment_method;
-- match all M-pesa to be uniform
update clean_dup
set payment_method=
case 
	when payment_method='mpesa' then 'M-Pesa'
	when payment_method='Mpesa' then 'M-Pesa'
	else initcap(payment_method)
END;

--Added a payment_method_check CONSTRAINT
--M-Pesa / Cash / Card / Bank Transfer

alter table  clean_dup
add constraint payment_method check(payment_method in ('M-Pesa', 
			'Cash', 
			'Card',
			'Bank Transfer'));

--cleaning booking status
-- how many booking status we got
select distinct booking_status from clean_dup;

-- There are two Checked out, lets standardize each
update clean_dup
set  booking_status = 
case
	when booking_status ='checked out' then 'Checked Out'
	else booking_status
end;

--Add constraint to ensure that all booking status going forward align
alter table clean_dup
add constraint booking_checker check(booking_status in ('Cancelled','No Show','Checked Out'));

-- cleaning Total billed
-- removing anything that is not a number
update clean_dup
set total_amount = regexp_replace(total_amount, '[^0-9]','','g');

--converting to numeric
alter table clean_dup
alter column total_amount type numeric
using (case
			when total_amount = '' then null
			else total_amount::numeric
		end);

--clean service_price to numeric data type or blank
alter table clean_dup
alter column service_price type numeric
using (case
			when service_price ='' then null
			else service_price::numeric
		end);


--cleaning service used- theres not much to be done here
select distinct service_used from clean_dup;

-- cleaning guest rating
select distinct guest_rating from clean_dup;

-- we noticed a number of things
--there were spaces
--remove spaces
update clean_dup
set guest_rating = trim(guest_rating);

--set to integer
alter table clean_dup
alter column guest_rating type int
using (case
			when guest_rating = '' then null
			else guest_rating::int
	end);

--adding a constraint to ensure that all the others align and remove 
update clean_dup
set guest_rating = null where guest_rating = 0;

alter table clean_dup
add constraint rating_checker check(guest_rating in(1,2,3,4,5));

select distinct guest_rating, count(*) from clean_dup group by guest_rating;

select * from clean_dup;

-- creating a view representing the clean data
create view v_clean_bookings as select * from clean_dup;


