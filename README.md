# CRM Call Center Perfomance SQL Project

### Overview

This project contains SQL scripts for querying a CRM database from a call center environment. The focus is on extracting and cleaning data to produce key performance metrics, which are then exported to Excel for further analysis and reporting.

### Data Model

The database is structured into several tables:

   - team_leaders: Stores team leader details.

   - lobs: Lists Lines of Business (e.g., T1 Voice, T1 Chat, T2 Voice, T2 Chat, French Voice, French Chat).

   - customers: Contains customer information (name, email, country, signup date).

   - agents: Contains agent details, including team leader association and hire date.

   - calls: Records detailed call information, including call link, call time, call date, duration, hold time, CSAT score, case status, and case number, with foreign keys to 
     agents, customers, and lobs.


### Objectives

  - Extract actionable insights (e.g., agent performance, call volume trends, CSAT analysis) from a CRM system.
    
  - Utilize advanced SQL techniques like joins, subqueries, window functions, and data cleaning.
  
  - Export results for further analysis in Excel to support workforce management decisions.

### Key SQL Techniques Used
   - Basic Querying: SELECT, WHERE, ORDER BY, and LIMIT.
   - Aggregations & Grouping: GROUP BY, HAVING, and ROLLUP.
   - Joins: Inner, outer, self, natural joins, and USING keyword.
   - Subqueries & CTEs: For modular and complex query building.
   - Window Functions: ROW_NUMBER, RANK, DENSE_RANK, LAG, and LEAD.
   - Data Cleaning: Using functions like TRIM, UPPER, LOWER, INITCAP, COALESCE, REPLACE.
   - Import/Export: COPY and \copy for CSV operations.

### Tools used

  - PostgreSQL : Querting Data
  - MS Excel : Data Analysis and Visualisation [Spreadsheet](microsoft.com)

### Queries Used

  #### Database Setup:

  ``` sql
-- ============================================
-- 1. Create Table: team_leaders
-- ============================================
CREATE TABLE team_leaders (
    team_leader_id SERIAL PRIMARY KEY,
    team_leader_name VARCHAR(100) NOT NULL
);

INSERT INTO team_leaders (team_leader_name) VALUES
('Michael Scott'),
('Pam Beesly'),
('Jim Halpert');

-- ============================================
-- 2. Create Table: lobs (Lines of Business)
-- ============================================
CREATE TABLE lobs (
    lob_id SERIAL PRIMARY KEY,
    lob_name VARCHAR(50) NOT NULL UNIQUE
);

INSERT INTO lobs (lob_name) VALUES
('T1 Voice'),
('T1 Chat'),
('T2 Voice'),
('T2 Chat'),
('French Voice'),
('French Chat');

-- ============================================
-- 3. Create Table: customers
-- ============================================
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    email VARCHAR(100),
    country VARCHAR(50),
    signup_date DATE
);

INSERT INTO customers (customer_name, email, country, signup_date) VALUES
('John Doe', 'john.doe@example.com', 'USA', '2023-07-15'),
('Jane Smith', 'jane.smith@example.com', 'USA', '2023-07-20'),
('Carlos Gomez', 'carlos.gomez@example.com', 'MEXICO', '2023-07-22'),
('Marie Curie', 'marie.curie@example.com', 'FRANCE', '2023-07-18'),
('Liu Wei', 'liu.wei@example.com', 'CHINA', '2023-07-25'),
('Olga Ivanova', 'olga.ivanova@example.com', 'RUSSIA', '2023-07-30'),
('David Brown', 'david.brown@example.com', 'USA', '2023-07-16'),
('Emily Davis', 'emily.davis@example.com', 'CANADA', '2023-07-21'),
('Hiro Tanaka', 'hiro.tanaka@example.com', 'JAPAN', '2023-07-19'),
('Sophia Lee', 'sophia.lee@example.com', 'SOUTH KOREA', '2023-07-23');

-- ============================================
-- 4. Create Table: agents
-- ============================================
CREATE TABLE agents (
    agent_id SERIAL PRIMARY KEY,
    agent_name VARCHAR(100) NOT NULL,
    team_leader_id INT REFERENCES team_leaders(team_leader_id),
    hire_date DATE,
    status VARCHAR(20) DEFAULT 'Active'
);

INSERT INTO agents (agent_name, team_leader_id, hire_date, status) VALUES
('Alice Johnson', 1, '2023-01-15', 'Active'),
('Bob Smith', 1, '2022-05-20', 'Active'),
('Charlie Davis', 2, '2023-07-01', 'Active'),
('David Wilson', 2, '2021-12-10', 'Inactive'),
('Emma Brown', 3, '2023-03-05', 'Active'),
('Fiona White', 1, '2022-11-12', 'Active'),
('George Black', 2, '2023-06-25', 'Active'),
('Helen Clark', 3, '2023-04-30', 'Active'),
('Ian Miller', 1, '2023-02-28', 'Active'),
('Jane Doe', 2, '2023-05-18', 'Active');

-- ============================================
-- 5. Create Table: calls
-- ============================================
CREATE TABLE calls (
    call_id SERIAL PRIMARY KEY,
    call_link TEXT,
    call_time TIME,
    call_date DATE,
    duration INT,         -- Duration in seconds
    hold_time INT,        -- Hold time in seconds
    csat_score INTEGER CHECK (csat_score BETWEEN 1 AND 5),
    case_status VARCHAR(50),
    case_number CHAR(5),
    agent_id INT REFERENCES agents(agent_id),
    customer_id INT REFERENCES customers(customer_id),
    lob_id INT REFERENCES lobs(lob_id)
);

-- ============================================
-- 6. Insert Sample Data into calls (90 rows)
-- We'll insert 10 rows per day for 9 days: 2023-08-01 to 2023-08-09.
-- For each day, call numbers increment sequentially.
-- ============================================

-- Day 1: 2023-08-01 (Rows 10001-10010)
INSERT INTO calls (call_link, call_time, call_date, duration, hold_time, csat_score, case_status, case_number, agent_id, customer_id, lob_id)
VALUES
('https://salesforce.com/case/10001', '08:15:00', '2023-08-01', 180, 20, 5, 'Closed', '10001', 1, 1, 1),
('https://salesforce.com/case/10002', '09:00:00', '2023-08-01', 300, 30, 4, 'Escalated', '10002', 2, 2, 2),
('https://salesforce.com/case/10003', '10:30:00', '2023-08-01', 150, 10, 3, 'Open', '10003', 3, 3, 3),
('https://salesforce.com/case/10004', '11:00:00', '2023-08-01', 240, 15, 5, 'Closed', '10004', 4, 4, 4),
('https://salesforce.com/case/10005', '12:15:00', '2023-08-01', 210, 25, 2, 'Open', '10005', 5, 5, 5),
('https://salesforce.com/case/10006', '13:00:00', '2023-08-01', 360, 40, 4, 'Escalated', '10006', 6, 6, 6),
('https://salesforce.com/case/10007', '14:20:00', '2023-08-01', 120,  5, 3, 'Closed', '10007', 7, 7, 1),
('https://salesforce.com/case/10008', '15:45:00', '2023-08-01', 330, 35, 5, 'Open', '10008', 8, 8, 2),
('https://salesforce.com/case/10009', '16:10:00', '2023-08-01', 255, 20, 4, 'Closed', '10009', 9, 9, 3),
('https://salesforce.com/case/10010', '17:00:00', '2023-08-01', 225, 15, 2, 'Escalated', '10010', 10, 10, 4);

-- Day 2: 2023-08-02 (Rows 10011-10020)
INSERT INTO calls (call_link, call_time, call_date, duration, hold_time, csat_score, case_status, case_number, agent_id, customer_id, lob_id)
VALUES
('https://salesforce.com/case/10011', '08:20:00', '2023-08-02', 200, 18, 4, 'Closed', '10011', 1, 2, 1),
('https://salesforce.com/case/10012', '09:10:00', '2023-08-02', 190, 12, 3, 'Open', '10012', 2, 3, 2),
('https://salesforce.com/case/10013', '10:00:00', '2023-08-02', 320, 25, 5, 'Escalated', '10013', 3, 4, 3),
('https://salesforce.com/case/10014', '11:30:00', '2023-08-02', 180, 10, 2, 'Closed', '10014', 4, 5, 4),
('https://salesforce.com/case/10015', '12:00:00', '2023-08-02', 240, 20, 3, 'Open', '10015', 5, 6, 5),
('https://salesforce.com/case/10016', '13:30:00', '2023-08-02', 360, 30, 4, 'Closed', '10016', 6, 7, 6),
('https://salesforce.com/case/10017', '14:00:00', '2023-08-02', 400, 35, 5, 'Escalated', '10017', 7, 8, 1),
('https://salesforce.com/case/10018', '15:15:00', '2023-08-02', 150,  8, 2, 'Closed', '10018', 8, 9, 2),
('https://salesforce.com/case/10019', '16:30:00', '2023-08-02', 270, 22, 3, 'Open', '10019', 9, 10, 3),
('https://salesforce.com/case/10020', '17:45:00', '2023-08-02', 300, 28, 4, 'Escalated', '10020', 10, 1, 4);

-- Day 3: 2023-08-03 (Rows 10021-10030)
INSERT INTO calls (call_link, call_time, call_date, duration, hold_time, csat_score, case_status, case_number, agent_id, customer_id, lob_id)
VALUES
('https://salesforce.com/case/10021', '08:05:00', '2023-08-03', 220, 18, 3, 'Closed', '10021', 1, 3, 1),
('https://salesforce.com/case/10022', '09:25:00', '2023-08-03', 310, 25, 5, 'Open', '10022', 2, 4, 2),
('https://salesforce.com/case/10023', '10:10:00', '2023-08-03', 260, 20, 4, 'Escalated', '10023', 3, 5, 3),
('https://salesforce.com/case/10024', '11:40:00', '2023-08-03', 180, 12, 2, 'Closed', '10024', 4, 6, 4),
('https://salesforce.com/case/10025', '12:50:00', '2023-08-03', 240, 15, 3, 'Open', '10025', 5, 7, 5),
('https://salesforce.com/case/10026', '13:20:00', '2023-08-03', 350, 30, 4, 'Escalated', '10026', 6, 8, 6),
('https://salesforce.com/case/10027', '14:55:00', '2023-08-03', 200, 10, 3, 'Closed', '10027', 7, 9, 1),
('https://salesforce.com/case/10028', '15:35:00', '2023-08-03', 320, 28, 5, 'Open', '10028', 8, 10, 2),
('https://salesforce.com/case/10029', '16:15:00', '2023-08-03', 210, 18, 4, 'Escalated', '10029', 9, 1, 3),
('https://salesforce.com/case/10030', '17:10:00', '2023-08-03', 190, 15, 2, 'Closed', '10030', 10, 2, 4);

-- Day 4: 2023-08-04 (Rows 10031-10040)
INSERT INTO calls (call_link, call_time, call_date, duration, hold_time, csat_score, case_status, case_number, agent_id, customer_id, lob_id)
VALUES
('https://salesforce.com/case/10031', '08:30:00', '2023-08-04', 230, 20, 4, 'Closed', '10031', 1, 4, 1),
('https://salesforce.com/case/10032', '09:15:00', '2023-08-04', 250, 18, 3, 'Escalated', '10032', 2, 5, 2),
('https://salesforce.com/case/10033', '10:45:00', '2023-08-04', 310, 25, 5, 'Open', '10033', 3, 6, 3),
('https://salesforce.com/case/10034', '11:20:00', '2023-08-04', 190, 15, 2, 'Closed', '10034', 4, 7, 4),
('https://salesforce.com/case/10035', '12:05:00', '2023-08-04', 270, 20, 4, 'Escalated', '10035', 5, 8, 5),
('https://salesforce.com/case/10036', '13:40:00', '2023-08-04', 220, 12, 3, 'Open', '10036', 6, 9, 6),
('https://salesforce.com/case/10037', '14:10:00', '2023-08-04', 330, 30, 5, 'Closed', '10037', 7, 10, 1),
('https://salesforce.com/case/10038', '15:00:00', '2023-08-04', 240, 18, 4, 'Open', '10038', 8, 1, 2),
('https://salesforce.com/case/10039', '16:20:00', '2023-08-04', 210, 15, 3, 'Escalated', '10039', 9, 2, 3),
('https://salesforce.com/case/10040', '17:25:00', '2023-08-04', 200, 10, 2, 'Closed', '10040', 10, 3, 4);

-- Day 5: 2023-08-05 (Rows 10041-10050)
INSERT INTO calls (call_link, call_time, call_date, duration, hold_time, csat_score, case_status, case_number, agent_id, customer_id, lob_id)
VALUES
('https://salesforce.com/case/10041', '08:05:00', '2023-08-05', 220, 20, 3, 'Closed', '10041', 1, 5, 1),
('https://salesforce.com/case/10042', '09:35:00', '2023-08-05', 260, 22, 5, 'Open', '10042', 2, 6, 2),
('https://salesforce.com/case/10043', '10:25:00', '2023-08-05', 280, 25, 4, 'Escalated', '10043', 3, 7, 3),
('https://salesforce.com/case/10044', '11:15:00', '2023-08-05', 200, 15, 2, 'Closed', '10044', 4, 8, 4),
('https://salesforce.com/case/10045', '12:55:00', '2023-08-05', 300, 28, 4, 'Open', '10045', 5, 9, 5),
('https://salesforce.com/case/10046', '13:05:00', '2023-08-05', 240, 18, 3, 'Escalated', '10046', 6, 10, 6),
('https://salesforce.com/case/10047', '14:30:00', '2023-08-05', 310, 30, 5, 'Closed', '10047', 7, 1, 1),
('https://salesforce.com/case/10048', '15:45:00', '2023-08-05', 230, 20, 4, 'Open', '10048', 8, 2, 2),
('https://salesforce.com/case/10049', '16:35:00', '2023-08-05', 210, 15, 3, 'Escalated', '10049', 9, 3, 3),
('https://salesforce.com/case/10050', '17:10:00', '2023-08-05', 190, 12, 2, 'Closed', '10050', 10, 4, 4);

-- Day 6: 2023-08-06 (Rows 10051-10060)
INSERT INTO calls (call_link, call_time, call_date, duration, hold_time, csat_score, case_status, case_number, agent_id, customer_id, lob_id)
VALUES
('https://salesforce.com/case/10051', '08:15:00', '2023-08-06', 230, 20, 4, 'Closed', '10051', 1, 6, 1),
('https://salesforce.com/case/10052', '09:00:00', '2023-08-06', 250, 18, 3, 'Escalated', '10052', 2, 7, 2),
('https://salesforce.com/case/10053', '10:30:00', '2023-08-06', 320, 25, 5, 'Open', '10053', 3, 8, 3),
('https://salesforce.com/case/10054', '11:00:00', '2023-08-06', 190, 15, 2, 'Closed', '10054', 4, 9, 4),
('https://salesforce.com/case/10055', '12:15:00', '2023-08-06', 270, 20, 4, 'Escalated', '10055', 5, 10, 5),
('https://salesforce.com/case/10056', '13:00:00', '2023-08-06', 220, 12, 3, 'Open', '10056', 6, 1, 6),
('https://salesforce.com/case/10057', '14:20:00', '2023-08-06', 330, 30, 5, 'Closed', '10057', 7, 2, 1),
('https://salesforce.com/case/10058', '15:45:00', '2023-08-06', 240, 18, 4, 'Open', '10058', 8, 3, 2),
('https://salesforce.com/case/10059', '16:10:00', '2023-08-06', 210, 15, 3, 'Escalated', '10059', 9, 4, 3),
('https://salesforce.com/case/10060', '17:00:00', '2023-08-06', 200, 10, 2, 'Closed', '10060', 10, 5, 4);

-- Day 7: 2023-08-07 (Rows 10061-10070)
INSERT INTO calls (call_link, call_time, call_date, duration, hold_time, csat_score, case_status, case_number, agent_id, customer_id, lob_id)
VALUES
('https://salesforce.com/case/10061', '08:10:00', '2023-08-07', 220, 18, 3, 'Closed', '10061', 1, 7, 1),
('https://salesforce.com/case/10062', '09:25:00', '2023-08-07', 280, 22, 5, 'Open', '10062', 2, 8, 2),
('https://salesforce.com/case/10063', '10:05:00', '2023-08-07', 260, 20, 4, 'Escalated', '10063', 3, 9, 3),
('https://salesforce.com/case/10064', '11:15:00', '2023-08-07', 200, 15, 2, 'Closed', '10064', 4, 10, 4),
('https://salesforce.com/case/10065', '12:20:00', '2023-08-07', 240, 17, 3, 'Open', '10065', 5, 1, 5),
('https://salesforce.com/case/10066', '13:35:00', '2023-08-07', 300, 25, 4, 'Escalated', '10066', 6, 2, 6),
('https://salesforce.com/case/10067', '14:45:00', '2023-08-07', 210, 15, 3, 'Closed', '10067', 7, 3, 1),
('https://salesforce.com/case/10068', '15:30:00', '2023-08-07', 310, 28, 5, 'Open', '10068', 8, 4, 2),
('https://salesforce.com/case/10069', '16:40:00', '2023-08-07', 230, 18, 4, 'Escalated', '10069', 9, 5, 3),
('https://salesforce.com/case/10070', '17:20:00', '2023-08-07', 190, 12, 2, 'Closed', '10070', 10, 6, 4);

-- Day 8: 2023-08-08 (Rows 10071-10080)
INSERT INTO calls (call_link, call_time, call_date, duration, hold_time, csat_score, case_status, case_number, agent_id, customer_id, lob_id)
VALUES
('https://salesforce.com/case/10071', '08:15:00', '2023-08-08', 230, 20, 4, 'Closed', '10071', 1, 8, 1),
('https://salesforce.com/case/10072', '09:00:00', '2023-08-08', 250, 18, 3, 'Escalated', '10072', 2, 9, 2),
('https://salesforce.com/case/10073', '10:30:00', '2023-08-08', 320, 25, 5, 'Open', '10073', 3, 10, 3),
('https://salesforce.com/case/10074', '11:00:00', '2023-08-08', 190, 15, 2, 'Closed', '10074', 4, 1, 4),
('https://salesforce.com/case/10075', '12:15:00', '2023-08-08', 270, 20, 4, 'Escalated', '10075', 5, 2, 5),
('https://salesforce.com/case/10076', '13:00:00', '2023-08-08', 220, 12, 3, 'Open', '10076', 6, 3, 6),
('https://salesforce.com/case/10077', '14:20:00', '2023-08-08', 330, 30, 5, 'Closed', '10077', 7, 4, 1),
('https://salesforce.com/case/10078', '15:45:00', '2023-08-08', 240, 18, 4, 'Open', '10078', 8, 5, 2),
('https://salesforce.com/case/10079', '16:10:00', '2023-08-08', 210, 15, 3, 'Escalated', '10079', 9, 6, 3),
('https://salesforce.com/case/10080', '17:00:00', '2023-08-08', 200, 10, 2, 'Closed', '10080', 10, 7, 4);

-- Day 9: 2023-08-09 (Rows 10081-10090)
INSERT INTO calls (call_link, call_time, call_date, duration, hold_time, csat_score, case_status, case_number, agent_id, customer_id, lob_id)
VALUES
('https://salesforce.com/case/10081', '08:30:00', '2023-08-09', 230, 20, 4, 'Closed', '10081', 1, 9, 1),
('https://salesforce.com/case/10082', '09:15:00', '2023-08-09', 250, 18, 3, 'Escalated', '10082', 2, 10, 2),
('https://salesforce.com/case/10083', '10:45:00', '2023-08-09', 320, 25, 5, 'Open', '10083', 3, 1, 3),
('https://salesforce.com/case/10084', '11:20:00', '2023-08-09', 190, 15, 2, 'Closed', '10084', 4, 2, 4),
('https://salesforce.com/case/10085', '12:05:00', '2023-08-09', 270, 20, 4, 'Escalated', '10085', 5, 3, 5),
('https://salesforce.com/case/10086', '13:40:00', '2023-08-09', 220, 12, 3, 'Open', '10086', 6, 4, 6),
('https://salesforce.com/case/10087', '14:10:00', '2023-08-09', 330, 30, 5, 'Closed', '10087', 7, 5, 1),
('https://salesforce.com/case/10088', '15:00:00', '2023-08-09', 240, 18, 4, 'Open', '10088', 8, 6, 2),
('https://salesforce.com/case/10089', '16:20:00', '2023-08-09', 210, 15, 3, 'Escalated', '10089', 9, 7, 3),
('https://salesforce.com/case/10090', '17:25:00', '2023-08-09', 200, 10, 2, 'Closed', '10090', 10, 8, 4);

-- End of Database Creation and Sample Data Insertions.
```
## Analysis Queries

### Query to extract meaningful call data from the database
``` sql
SELECT   TO_CHAR(call_date,'DD-MM-YYYY') AS CallDate,
         CAST(case_number AS integer) AS CaseNumber,
		 TRIM(INITCAP(agent_name)) AS AgentName,
	     TRIM(lob_name) AS LobName,
         TRIM(call_link) AS call_URL,
         SUBSTRING(call_time :: text FROM 1 FOR 5) AS CallTime,
	     CAST(COALESCE (csat_score::TEXT, ' ') AS integer)  AS  csat,
	     CAST(COALESCE (duration::TEXT, ' ') AS integer) AS AHT,
	     CAST(COALESCE (hold_time::TEXT, ' ') AS integer) AS holdTime,
	     INITCAP(TRIM(case_status)) AS status
FROM     calls c
JOIN     lobs l
ON       c.lob_id = l.lob_id
JOIN     agents a
USING    (agent_id)
ORDER BY call_date,call_time
;
```
#### Below is a screenshot of the query in PgAdmin4

![Screenshot (5)](https://github.com/user-attachments/assets/e2a58f53-7fbf-4bfd-b873-8862e02c9bfe)

### Query to extract Team leaders and their agents along with their LOBs

``` sql
SELECT DISTINCT 
       a.agent_id,
       INITCAP(TRIM(a.agent_name)) AS agent_name,
       INITCAP(TRIM(tl.team_leader_name)) AS TlName,
       l.lob_name
FROM agents a
LEFT JOIN team_leaders tl ON a.team_leader_id = tl.team_leader_id
JOIN      calls c ON a.agent_id = c.agent_id
LEFT JOIN lobs l ON c.lob_id = l.lob_id
ORDER BY agent_id;
```
#### Below is a screenshot of the query in PgAdmin4

![Screenshot (7)](https://github.com/user-attachments/assets/cd98816e-be57-4ebe-a406-6105f21a62d9)



