# Student-Streaks-Analysis-with-SQL-Project
Identify top learners in an online subscription platform using a MySQL database

This project aims to analyze student activity streaks, such as attendance or assignment submissions, using SQL. The goal is to provide insights into student behavior patterns and identify trends that could help in improving educational outcomes.

Project Outline
1. Project Introduction
Objective: To analyze student activity streaks and provide insights into patterns of behavior.
Data Source: Synthetic or real data on student activities (attendance, assignments, etc.).
Tools: SQL (PostgreSQL, MySQL, or any preferred SQL database).

2. Data Collection and Preparation
Data Requirements:
  Student Information: Student ID, Name, Class, etc.
  Activity Logs: Date, Student ID, Activity Type (attendance, submission, etc.), Status (present, submitted, etc.).
Data Sources: School databases, learning management systems, or synthetic data generators.
Data Cleaning: Ensure data consistency, handle missing values, and normalize data formats

3. Database Design
Schema Design:
  Students Table: students(id, name, class, ...)
  Activities Table: activities(id, student_id, date, activity_type, status)
  Activity Types Table: activity_types(id, type)
Relationships:
students.id -> activities.student_id
activity_types.id -> activities.activity_type

4. SQL Queries for Analysis
Streak Calculation:

Daily Streaks: Continuous days of activity (attendance or submissions)
Sql Code
SELECT student_id, activity_type, date,
       ROW_NUMBER() OVER(PARTITION BY student_id, activity_type ORDER BY date) -
       ROW_NUMBER() OVER(PARTITION BY student_id, activity_type ORDER BY date) AS streak_id
FROM activities
WHERE status = 'present' OR status = 'submitted';

Consecutive Days: Finding longest streaks
sql code
SELECT student_id, activity_type, COUNT(*) AS streak_length
FROM (
    SELECT student_id, activity_type,
           DATE_DIFF('day', LAG(date) OVER(PARTITION BY student_id, activity_type ORDER BY date), date) AS diff
    FROM activities
    WHERE status = 'present' OR status = 'submitted'
) AS temp
WHERE diff = 1
GROUP BY student_id, activity_type
ORDER BY streak_length DESC;

Aggregated Insights:
Average Streak Length:
sql code
SELECT student_id, activity_type, AVG(streak_length) AS avg_streak_length
FROM (
    SELECT student_id, activity_type, COUNT(*) AS streak_length
    FROM (
        SELECT student_id, activity_type,
               DATE_DIFF('day', LAG(date) OVER(PARTITION BY student_id, activity_type ORDER BY date), date) AS diff
        FROM activities
        WHERE status = 'present' OR status = 'submitted'
    ) AS temp
    WHERE diff = 1
    GROUP BY student_id, activity_type
) AS streaks
GROUP BY student_id, activity_type;

5. Visualization and Reporting
Tools: Tableau, Power BI, or any BI tool.
Dashboards:
Streak Trends: Visualization of streak lengths over time.
Top Performers: Students with the longest streaks.
Streak Distribution: Distribution of streak lengths across the student population.

6. Conclusion
Findings: Summarize key insights from the data.
Recommendations: Suggest interventions or strategies to improve student engagement based on the analysis.

7. Future Work
Predictive Analysis: Using machine learning to predict future streaks.
Automated Alerts: Setting up alerts for students who are at risk of breaking their streaks.

Implementation Example
Here’s a simplified example of how you might implement some of these steps in SQL:

Create Tables:
sql code

CREATE TABLE students (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    class VARCHAR(50)
);

CREATE TABLE activities (
    id INT PRIMARY KEY,
    student_id INT,
    date DATE,
    activity_type INT,
    status VARCHAR(50),
    FOREIGN KEY (student_id) REFERENCES students(id),
    FOREIGN KEY (activity_type) REFERENCES activity_types(id)
);

CREATE TABLE activity_types (
    id INT PRIMARY KEY,
    type VARCHAR(50)
);

Insert Sample Data:
INSERT INTO students (id, name, class) VALUES (1, 'John Doe', 'Math'), (2, 'Jane Smith', 'Science');
INSERT INTO activity_types (id, type) VALUES (1, 'attendance'), (2, 'submission');
INSERT INTO activities (id, student_id, date, activity_type, status) VALUES 
(1, 1, '2023-01-01', 1, 'present'), 
(2, 1, '2023-01-02', 1, 'present'),
(3, 1, '2023-01-03', 1, 'present'),
(4, 2, '2023-01-01', 1, 'absent'),
(5, 2, '2023-01-02', 1, 'present'),
(6, 2, '2023-01-03', 1, 'present');

Analyze Streaks:
WITH streaks AS (
    SELECT student_id, date, 
           ROW_NUMBER() OVER (PARTITION BY student_id ORDER BY date) 
           - ROW_NUMBER() OVER (PARTITION BY student_id ORDER BY date) AS streak_id
    FROM activities
    WHERE status = 'present'
)
SELECT student_id, MIN(date) AS streak_start, MAX(date) AS streak_end, COUNT(*) AS streak_length
FROM streaks
GROUP BY student_id, streak_id
ORDER BY streak_length DESC;

This example sets up the basic structure for your project. You can expand on this by adding more complex analyses, refining your SQL queries, and integrating visualization tools for a more comprehensive analysis.

