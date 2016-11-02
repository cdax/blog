+++
date = "2013-02-05T12:10:43+05:30"
draft = false
title = "Dynamic Pivoting with T-SQL"
tags = ["sql"]

+++

In this post, I will share a cheap trick technique for pivoting a variable number of columns using Microsoft’s Transact-SQL. The idea of writing dynamic SQL to accomplish a pivot is nothing new, and can be found in discussion forums all over the web. Anyway, I’ve been looking for something to blog about, so I thought I’d go into more detail with this.


#### The Problem

You’re given two tables `Students` and `Courses`. They have the keys `STUDENT_ID` and `COURSE_ID`, respectively. Both tables are subject to frequent updates, inserts and deletes.

<iframe src="https://skydrive.live.com/embed?cid=FA469B013B97AF8D&#038;resid=FA469B013B97AF8D%21130&#038;authkey=AFpg9qTzEKTzrVg&#038;em=2&#038;wdAllowInteractivity=False&#038;Item=Students&#038;wdHideGridlines=False&#038;wdDownloadButton=True" width="223" height="260" frameborder="0" scrolling="no"></iframe> <iframe src="https://skydrive.live.com/embed?cid=FA469B013B97AF8D&#038;resid=FA469B013B97AF8D%21130&#038;authkey=AFpg9qTzEKTzrVg&#038;em=2&#038;wdAllowInteractivity=False&#038;Item=Courses&#038;wdHideGridlines=False&#038;wdDownloadButton=True" width="399" height="260" frameborder="0" scrolling="no"></iframe>

A third table called `Grades` stores the letter-value of the grade scored by each student, in each course that he took. A record in Grades is structured somewhat like this: (`STUDENT_ID | COURSE_ID | GRADE`).

<iframe src="https://skydrive.live.com/embed?cid=FA469B013B97AF8D&#038;resid=FA469B013B97AF8D%21130&#038;authkey=AFpg9qTzEKTzrVg&#038;em=2&#038;wdAllowInteractivity=False&#038;Item=Grades&#038;wdHideGridlines=False&#038;wdDownloadButton=True" width="292" height="239" frameborder="0" scrolling="no"></iframe>

What is required now, is a table or a view Report that has exactly one row for each student in Students, and a column for each of the courses in Courses, with the intersection of a row and a column denoting the grade received by a certain student in a certain course.

<iframe src="https://skydrive.live.com/embed?cid=FA469B013B97AF8D&#038;resid=FA469B013B97AF8D%21130&#038;authkey=AFpg9qTzEKTzrVg&#038;em=2&#038;wdAllowInteractivity=False&#038;Item=Report&#038;wdHideGridlines=False&#038;wdDownloadButton=True" width="600" height="260" frameborder="0" scrolling="no"></iframe>

While this would have been easy had Courses been a static table, implementing a pivot involving a variable number of pivot columns would require more than just your average T-SQL query. And that is because your average T-SQL query would require the number of pivot columns to be known at the time of writing the query.

#### The Solution

To put it plainly, what you do is engineer your `PIVOT` query by running a loop once for each pivot column, and piecing together the parts. Then you pass this query to `EXECUTE()`. Additionally, if the results of the `PIVOT` are to go inside a table, then this table will have to be created using a `CREATE TABLE` query, also built brick-by-brick inside a loop. Here’s what your code would look like, more or less.

```
--Build a query to create the table 'Report'
IF OBJECT_ID("Report") IS NOT NULL
    DROP TABLE Report
DECLARE @create_query VARCHAR(MAX)
SET @create_query = "CREATE TABLE [Report] ( [STUDENT_NAME] VARCHAR(50)"

--Create a temp table to store the names and IDs of the pivot columns.
--The INSERT query here controls the order, names and formatting of the
--pivot columns
IF OBJECT_ID("tempdb..#temp_Courses") IS NOT NULL
    DROP TABLE #temp_Courses
CREATE TABLE #temp_Courses([COURSE_ID] VARCHAR(10), [COURSE_NAME] VARCHAR(50), [ROW_NUM] BIGINT);
INSERT INTO #temp_Courses
SELECT *, ROW_NUMBER() OVER (ORDER BY [COURSE_NAME] ASC) AS [ROW_NUM] FROM Courses;

--Get the count of pivot columns; this will be used in our loop below
DECLARE @column_count INT;
SET @column_count = (SELECT COUNT(*) FROM #temp_Courses);

--We'll build two parts of our PIVOT query separately, and add them together later
--(A look at the syntax of a PIVOT query will reveal why this was needed)
DECLARE @pivot_query VARCHAR(MAX);
DECLARE @column_id_list VARCHAR(MAX);
SET @pivot_query = "SELECT [STUDENT_NAME]";
SET @column_id_list = "";

DECLARE @row_number INT;
SET @row_number = 1;

DECLARE @column_id VARCHAR(10)
DECLARE @column_name VARCHAR(50)
--Loop through the records in Courses, putting together your PIVOT query piece-by-piece
WHILE @row_number <= @column_count
BEGIN
SET @column_id = (SELECT COURSE_ID FROM #temp_Courses WHERE ROW_NUM = @row_number);
SET @column_name = (SELECT COURSE_NAME FROM #temp_Courses WHERE ROW_NUM = @row_number);

SET @create_query = @create_query + ", [" + @column_name + "] VARCHAR(2)";
SET @pivot_query = @pivot_query + ", [" + @column_id + "] AS [" + @column_name + "]";

SET @column_id_list = @column_id_list + CASE WHEN @row_number = 1 THEN "" ELSE ", " END + "[" + @column_id + "]";
SET @row_number = @row_number + 1;
END

SET @create_query = @create_query + ");"
SET @pivot_query = @pivot_query + " FROM (
                        SELECT G.STUDENT_ID [STUDENT_ID], S.STUDENT_NAME [STUDENT_NAME], G.COURSE_ID [COURSE_ID], G.GRADE [GRADE]
                        FROM
                            Grades G
                            LEFT OUTER JOIN
                            Students S
                        ON  G.STUDENT_ID = S.STUDENT_ID
                     ) J
                    PIVOT (
                        MAX([GRADE]) FOR [COURSE_ID] IN (" + @column_id_list + ")
                    ) AS P;";

--Drop the temp table
IF OBJECT_ID("tempdb..#temp_Courses") IS NOT NULL
    DROP TABLE #temp_Courses

--Execute the pivot-query
EXECUTE(@create_query + "INSERT INTO [Report] " + @pivot_query)
```

The generated SQL queries look like this:

```
CREATE TABLE [Report] (
    [STUDENT_NAME] VARCHAR(50),
    [Algorithms: Design and Analysis, Part 1] VARCHAR(2),
    [Critical Thinking in Global Challenges] VARCHAR(2),
    [Developing Innovative Ideas for New Companies] VARCHAR(2),
    [E-learning and Digital Cultures] VARCHAR(2),
    [India`s Rivers] VARCHAR(2),
    [Intermediate Algebra] VARCHAR(2),
    [Introduction to Philosophy] VARCHAR(2),
    [Nutrition for Health Promotion] VARCHAR(2),
    [Pre-Calculus] VARCHAR(2),
    [Principles of Public Health] VARCHAR(2)
);

INSERT INTO [Report]
SELECT  [STUDENT_NAME],
        [2000000009] AS [Algorithms: Design and Analysis, Part 1],
        [2000000010] AS [Critical Thinking in Global Challenges],
        [2000000005] AS [Developing Innovative Ideas for New Companies],
        [2000000003] AS [E-learning and Digital Cultures],
        [2000000004] AS [India`s Rivers],
        [2000000001] AS [Intermediate Algebra],
        [2000000007] AS [Introduction to Philosophy],
        [2000000006] AS [Nutrition for Health Promotion],
        [2000000002] AS [Pre-Calculus],
        [2000000008] AS [Principles of Public Health]
FROM (
        SELECT G.STUDENT_ID [STUDENT_ID],
               S.STUDENT_NAME [STUDENT_NAME],
               G.COURSE_ID [COURSE_ID],
               G.GRADE [GRADE]
        FROM   Grades G
               LEFT OUTER JOIN
               Students S
               ON G.STUDENT_ID = S.STUDENT_ID
     ) J
PIVOT (
        MAX([GRADE])
        FOR [COURSE_ID]
        IN
        ([2000000009], [2000000010], [2000000005], [2000000003], [2000000004], [2000000001], [2000000007], [2000000006], [2000000002], [2000000008])
) AS P;
```

#### But Watch Out!

Column names used in the pivot query should not exceed the SQL Server limit of 128 characters. Like in the above query, if any course name were to exceed 128 characters, the `EXECUTE` would fail. This is a limitation imposed by SQL Server.
