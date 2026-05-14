# Hotel Tembo Analysis
A short project on hotel data answering business steering questions 

# Project Overview and analysis

# Project Scenario

You have just been hired as a Junior Data Analyst at **Tembo Hotel & Suites**, a mid-range business hotel in Nairobi. The hotel has been running since 2023 and keeping records in a spreadsheet. That spreadsheet is a mess.

## 💬 Message from the Hotel Director

> "We need to understand how the hotel is performing. The data team before you kept everything in Excel and it is full of errors.  
> I need you to clean this data, load it into our new database, analyse it, and present your findings to the management team on Friday.  
> I want to know: which rooms make us the most money, which months are busiest, how our staff are performing, and whether our guests are happy.  
> Make it look professional — use Power BI for the visuals."

This is your team's job for the week. By Friday, you will present your findings as a data analyst team — not as students.

---

# Project Timeline

| Day       | Focus                          | What You Deliver at End of Day |
|----------|--------------------------------|--------------------------------|
| Monday   | Setup + Data Cleaning           | Clean table loaded in PostgreSQL. All dirty problems fixed. Screenshot of `SELECT *` showing clean data. 

---

# The Data – `tembo_hotel_dirty.csv`

You will receive a CSV file with data on hotel booking records. This is real-world style data — it has many problems that you must find and fix before any analysis can begin.

---

# Columns in the CSV

| Column                | Description                         | Expected Clean Format |
|----------------------|-------------------------------------|------------------------|
| booking_id           | Unique booking reference            | BK0001, BK0002 ... |
| guest_name           | Full name of guest                  | Title Case (e.g. Alice Mwangi) |
| guest_phone          | Guest phone number                  | 07XXXXXXXX (10 digits, no dashes or +254) |
| guest_city           | Guest home city                     | Title Case (e.g. Nairobi) |
| guest_nationality    | Guest nationality                   | Title Case (e.g. Kenyan) |
| room_no              | Room number                         | 101, 201, 301 etc. |
| room_type            | Type of room                        | Standard / Deluxe / Suite / Penthouse |
| room_rate_per_night  | Nightly rate in KES                 | Numeric (e.g. 5500) |
| check_in_date        | Date guest checked in               | YYYY-MM-DD |
| check_out_date       | Date guest checked out              | YYYY-MM-DD |
| nights_stayed        | Number of nights                    | Positive integer |
| staff_name           | Staff who handled booking           | Title Case |
| staff_department     | Staff department                    | Front Desk / Housekeeping / Restaurant / Security / Management |
| staff_salary         | Staff monthly salary in KES         | Numeric (e.g. 42000) |
| payment_method       | How guest paid                      | M-Pesa / Cash / Card / Bank Transfer |
| booking_status       | Outcome of booking                  | Checked Out / Cancelled / No Show |
| total_amount         | Total billed in KES                 | Numeric (e.g. 11000) |
| service_used         | Extra service used (if any)         | Room Service / Laundry / Airport Pickup or blank |
| service_price        | Price of service in KES             | Numeric or blank |
| guest_rating         | Guest satisfaction rating           | Integer (1 to 5) |

---

# Known Dirty Data Problems to Fix

Your job is to identify every instance of each problem and fix it using SQL.  

---

# Deliverables

## Deliverable 1 — Clean Database

- DDL: `CREATE SCHEMA` + all tables with correct data types, primary keys, and constraints  
- Import: CSV loaded into a staging table using pgAdmin's import tool  
- Cleaning script: All 22 dirty problems identified and fixed with SQL (commented and organised)  
- Clean view: A final view called `v_clean_bookings` that shows only clean, valid data  

---

## Deliverable 2 — Analysis Queries

At minimum, your group must write queries answering these business questions:

1. **Revenue analysis**
   - Total revenue by month  
   - Revenue by room type  
   - Revenue by payment method  

2. **Occupancy**
   - Which room types are booked most?  
   - Average nights stayed per room type  

3. **Guest insights**
   - Top 10 cities guests come from  
   - Average rating per room type  

4. **Staff performance**
   - Which staff handled the most bookings?  
   - Which department generates the most revenue?  

5. **Trends**
   - Revenue growth month over month (window function)  
   - Busiest vs quietest months  

6. **Cancellations**
   - Cancellation rate per room type  
   - Revenue lost from cancellations and no-shows  

---

## Deliverable 3 — Views, Indexes & Power BI 

- At least **4 views** created (one per major business area: revenue, occupancy, guests, staff)  
- Indexes on all key columns:
  - `room_no`
  - `staff_name`
  - `guest_city`
  - `check_in_date`
- Power BI connected to your PostgreSQL database  
- At least **5 visuals** in Power BI  

---
