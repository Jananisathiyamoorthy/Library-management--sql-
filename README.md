# Library-management--sql-
This project includes managing data and performing function using postgresql.

# Objectives:-

1.Database Setup: Design and initialize the Library Management System database, including tables for branches, employees, members, books, issued records, and return records.

2.Operations: Implement Create, Read, Update, and Delete functionalities for managing data within the system
      
3.CTAS (Create Table As Select): Use CTAS statements to generate new tables derived from query results.

# Project Structure
# 1. Database Setup

   <img width="1267" height="915" alt="Screenshot_20251028_224223" src="https://github.com/user-attachments/assets/ada84fd2-a42b-4063-a27c-9aa674b4c740" />

Database Creation: Created a database named ppj.

Table Creation: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.   
               
```sql
--ppjlibrary management system
--branch table
drop table if exists branch;
create table branch(branch_id  varchar(10) primary key,
                   manager_id varchar(20), branch_address varchar(50),
                    contact_no varchar(20)
);
--employer table
drop table if exists employees;
create table employees(emp_id varchar(19) primary key,
                       emp_name varchar(19),
					   position varchar(15),
					   salary int,
					   branch_id varchar(10) --fk
);
--create books table
drop table if exists books;
create table books(isbn varchar(20) primary key,
                   book_title varchar(75),
				   category	varchar(20),
				   rental_price float,
				   status varchar(115),
				   author varchar(50),
				   publisher varchar(50)
);

--create table members
drop table if exists members;
create table members(member_id varchar(30) primary key,
                    member_name varchar(50),
					member_address varchar(50),
					reg_date date
);

--create table issued status
create table issued_status (issued_id varchar(10) primary key,
                            issued_member_id varchar(10), --fk
						  issued_book_name varchar(75),
						  issued_date date,
						  issued_book_isbn varchar(25), --fk
						  issued_emp_id varchar(10)      --fk

-- create table returnstatus

CREATE TABLE return_status
(
            return_id VARCHAR(10) PRIMARY KEY,
            issued_id VARCHAR(30),
            return_book_name VARCHAR(80),
            return_date DATE,
			return_book_isbn varchar(20)
);

--foreign key

alter table issued_status
add constraint fk_members
foreign key(issued_member_id)
references members(member_id);

alter table issued_status
add constraint fk_books
foreign key(issued_book_isbn)
references books(isbn);

alter table issued_status
add constraint fk_empid
foreign key(issued_emp_id)
references employees(emp_id);

alter table employees
add constraint fk_branchid
foreign key(branch_id)
references branch(branch_id);

select * from books;
select * from branch;
select * from employees;
select * from issued_status;
select * from members;
```



 ### 2. CRUD Operations--create,read,update,delete 
```sql

-- Task 1. Create a New Book Record
-- '978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

insert into books(isbn ,
                   book_title ,
				   category	,
				   rental_price ,
				   status,
				   author,
				   publisher)
values 
('978-1-60129-456-2', 'To Kill a Mockingbird',
'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');

select * from books;


-- Task 2: Update an Existing Member's Address
update members   --table
set member_address='125 main street'       --column
where member_id='C101';
select * from members;


-- Task 3: Delete a Record from the Issued Status Table
-- Objective: Delete the record with issued_id = 'IS104' from the issued_status table.

select * from issued_status;

delete from issued_status
--where = 'IS107';
-- it cannot delete because it has the same data oin return table 
where issued_id= 'IS121';

-- Task 4: Retrieve All Books Issued by a Specific Employee
-- Objective: Select all books issued by the employee with emp_id = 'E101'.

select issued_book_name from issued_status
where issued_emp_id='E101';


-- Task 5: List Members Who Have Issued More Than One Book
-- Objective: Use GROUP BY to find members who have issued more than one book.

select issued_emp_id,count(issued_id) from issued_status
group by 1
having count(issued_id)>1;

--find members who have issued more than one book.
select issued_emp_id from issued_status
group by 1
having count(issued_id)>1;
```
### 3. CTAS (Create Table As Select)

```sql
-- Task 6: Create Summary Tables**: Used CTAS 
--to generate new tables based on query results - each book and total book_issued_cnt
select * from books as b
join 
issued_status as ist on ist.issued_book_isbn = b.isbn;

select b.isbn,count(ist.issued_id) as no_issued from books as b
join 
issued_status as ist on ist.issued_book_isbn = b.isbn
group by 1

select b.isbn,b.book_title,count(ist.issued_id) as no_issued from books as b
join 
issued_status as ist on ist.issued_book_isbn = b.isbn
group by 1

create table book_counts
as
select b.isbn,b.book_title,count(ist.issued_id) as no_issued from books as b
join 
issued_status as ist on ist.issued_book_isbn = b.isbn
group by 1,2;

select * from book_counts;

```

### 4. Data Analysis & Findings

```sql
-- Task 7. **Retrieve All Books in a Specific Category:

select * from books
where category='Classic';



-- Task 8: Find Total Rental Income by Category:
select b.category,sum(b.rental_price)
from books as b
join 
issued_status as ist on ist.issued_book_isbn = b.isbn
group by 1

select b.category,sum(b.rental_price),count(*)
from books as b
join 
issued_status as ist on ist.issued_book_isbn = b.isbn
group by 1

-- Task 9. List Members Who Registered in the Last 180 Days**:

select * from members
where reg_date >= current_date - interval '180 days';
        
insert into members(member_id,member_name,member_address,reg_date)
values('C118','sam','145 main st','2024-06-01'),
('C119','john','133 main st','2024-05-01');


-- Task 10: List Employees with Their Branch Manager's Name and their branch details**:
 select * from branch
 
  select * from employees as el
  join branch as b
  on b.branch_id=el.branch_id
  join employees as e2
  on b.manager_id=e2.emp_id

  select el.*,
         e2.emp_name as manager,
		 b.manager_id
  from employees as el
  join branch as b
  on b.branch_id=el.branch_id
  join employees as e2
  on b.manager_id=e2.emp_id

  -- Task 11. Create a Table of Books with Rental Price Above a Certain Threshold 7 usd
create table booksrentalprice
as
select * from books
where rental_price> 7;

select * from booksrentalprice

-- Task 12: Retrieve the List of Books Not Yet Returned
 select 
	* from issued_status as iss
	left join return_status as rs
	on iss.issued_id=rs.issued_id
	where rs.return_id is null

select distinct iss.issued_book_name
from issued_status as iss
	left join return_status as rs
	on iss.issued_id=rs.issued_id
	where rs.return_id is null
```


##  Advanced SQL Operations
```sql
--Task 13: Identify Members with Overdue Books
--Write a query to identify members who have overdue books (assume a 30-day return period).
--Display the member's name, book title, issue date, and days overdue.

--issuedstatus=members==books=return status
--filter books
--over due>30
                                      --first join
select * from issued_status as ist
join members as m
on m.member_id=ist.issued_member_id
join books as b
on b.isbn=ist.issued_book_isbn
left join return_status as rs
on rs.issued_id=ist.issued_id
                                    --then select columns needed
select 
       ist.issued_member_id,
	   m.member_name,
	   b.book_title,
	   ist.issued_date,
	   rs.return_date
from issued_status as ist
join members as m
on m.member_id=ist.issued_member_id
join books as b
on b.isbn=ist.issued_book_isbn
left join return_status as rs
on rs.issued_id=ist.issued_id
                                          --showing return date null column in the table

select 
       ist.issued_member_id,
	   m.member_name,
	   b.book_title,
	   ist.issued_date,
	   rs.return_date
from issued_status as ist
join members as m
on m.member_id=ist.issued_member_id
join books as b
on b.isbn=ist.issued_book_isbn
left join return_status as rs
on rs.issued_id=ist.issued_id
where rs.return_date is null

                                       -- overdue date
select current_date

                                        --to identify the overdue subtract from current date
select 
       ist.issued_member_id,
	   m.member_name,
	   b.book_title,
	   ist.issued_date,
	   rs.return_date,
	   current_date- ist.issued_date as over_due_days
from issued_status as ist
join members as m
on m.member_id=ist.issued_member_id
join books as b
on b.isbn=ist.issued_book_isbn
left join return_status as rs
on rs.issued_id=ist.issued_id				
where rs.return_date is null
                                 --if over due days less than 7 we dont need that so
select 
       ist.issued_member_id,
	   m.member_name,
	   b.book_title,
	   ist.issued_date,
	   rs.return_date,
	   current_date- ist.issued_date as over_due_days
from issued_status as ist
join members as m
on m.member_id=ist.issued_member_id
join books as b
on b.isbn=ist.issued_book_isbn
left join return_status as rs
on rs.issued_id=ist.issued_id				
where rs.return_date is null
and (current_date- ist.issued_date)>30
order by 1

select * from books;
select * from branch;
select * from employees;
select * from issued_status;
select * from members;
select * from return_status;

--Task 14: Update Book Status on Return
--Write a query to update the status of books in the books table to "available" 
--when they are returned (based on entries in the return_status table).

--2 methods store procedure and manual
--manual method

select *  from issued_status
select * from books
where isbn='978-0-451-52994-2'

update books
set status='no'
where isbn='978-0-451-52994-2'          --check where isbn in issue table updated or not

select * from issued_status
where issued_book_isbn='978-0-451-52994-2'
select * from return_status
where issued_id='IS130'             --no data appears bcoz not returned

--sif book return so update

insert into return_status(return_id,issued_id,return_date)
values('RS125','IS130',current_date);
select * from return_status
where issued_id='IS130';

update books
set status='yes'
where isbn='978-0-451-52994-2'  

--store method procedure
--syntx
create or replace procedure add_return_records()
language plpgsql
as $$

declare 

begin
--all your logic and code
--inserting into returns on based on users input
end;
$$
             --in bracket add


			 
create or replace procedure add_return_records(p_return_id varchar(10),p_issued_id varchar(30))
language plpgsql

as $$

declare v_isbn varchar(20);
        v_book_name varchar(75);
begin
insert into return_status(return_id,issued_id,return_date)
values(p_return_id,p_issued_id,current_date);
select issued_book_isbn,
       issued_book_name
into v_isbn,
     v_book_name from issued_status
where issued_id=p_issued_id;

update books
set status='yes'
where isbn='v_isbn'; 

raise notice 'thank you for returning thre book:%', v_book_name;

end;
$$

call add_return_records()


-- to test the function in add_return_records
--we can take is135 book in issue table and book table it is available

select * from books
where isbn='978-0-307-58837-1';
--it status no so we can take exzmple of this book

select * from  issued_status
where issued_id='IS135'
select * from return_status

delete from return_status
where issued_id='IS135'

-- to update the record of is135
--calling function
call add_return_records('RS138','IS135');  --thanku for returning book appears 
--verify it in books table

select * from books
where isbn='978-0-307-58837-1';

--another example IS140


update books
set status='no'
where isbn='978-0-330-25864-8'; 

call add_return_records('RS148','IS140');  --thanku for returning book appears 

select * from  books
where isbn='978-0-330-25864-8'; 


--
--task 15: Branch Performance Report
--Create a query that generates a performance report for each branch, 
--showing the number of books issued, the number of books returned, 
--and the total revenue generated from book rentals.

select * from branch
select * from issued_status  --empidasociated with branch

select * from employees   --- empid with branch id
select * from books   --rental information price
select * from return_status   ---total no books returned

--join 1st issue table and employee
create table branch_reports
as
select 
       b.branch_id,b.manager_id,
	   sum(bk.rental_price) as total_revenue,
	   count(ist.issued_id) as number_book_issued,
	   count(rs.return_id ) as number_book_retuern

from issued_status as ist
join employees as e
on e.emp_id=ist.issued_emp_id
--join branch
join branch as b
on e.branch_id=b.branch_id
--join return we want only return books so we use left join
left join return_status as rs
on rs.issued_id=ist.issued_id
--join book
join books as bk
on ist.issued_book_isbn=bk.isbn
--to order
group by 1,2

select * from branch_reports;


--Task 16: CTAS: Create a Table of Active Members
--Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing
--members who have issued at least one book in the last 6 months.

create table active_members
as
select * from members
where member_id
in
(
select 
      distinct issued_member_id
from issued_status
where issued_date >= current_date - interval '24 months'  --data is in 2024 april today 2025 so i used 24 months
)

select * from active_members

--Task 17: Find Employees with the Most Book Issues Processed
--Write a query to find the top 3 employees who have processed the most book issues.
--Display the employee name, number of books processed, and their branch.

select  
     e.emp_name,
	 b.*,
	 count(ist.issued_id) as no_book_processed
from issued_status as ist
join employees as e
on 
e.emp_id=ist.issued_emp_id
join
branch as b
on e.branch_id=b.branch_id
group by 1,2

```

## Data Analysis:

Evaluation of book categories, employee salary distribution, member registration trends, and book issuance statistics.


## Conclusion:

This project showcases the practical use of SQL in designing and managing a Library Management System. It encompasses database creation, data handling, and advanced query techniques, offering a effective data management and analytical reporting.










