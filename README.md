# Data Normalization and Entity-Relationship Diagramming

## 1. Convert to 4NF

## Table containg original data
| assignment_id | student_id | due_date | professor | assignment_topic                | classroom | grade | relevant_reading    | professor_email   |
| :------------ | :--------- | :------- | :-------- | :------------------------------ | :-------- | :---- | :------------------ | :---------------- |
| 1             | 1          | 23.02.21 | Melvin    | Data normalization              | WWH 101   | 80    | Deumlich Chapter 3  | l.melvin@foo.edu  |
| 2             | 7          | 18.11.21 | Logston   | Single table queries            | 60FA 314  | 25    | Dümmlers Chapter 11 | e.logston@foo.edu |
| 1             | 4          | 23.02.21 | Melvin    | Data normalization              | WWH 101   | 75    | Deumlich Chapter 3  | l.melvin@foo.edu  |
| 5             | 2          | 05.05.21 | Logston   | Python and pandas               | 60FA 314  | 92    | Dümmlers Chapter 14 | e.logston@foo.edu |
| 4             | 2          | 04.07.21 | Nevarez   | Spreadsheet aggregate functions | WWH 201   | 65    | Zehnder Page 87     | i.nevarez@foo.edu |
| ...           | ...        | ...      | ...       | ...                             | ...       | ...   | ...                 | ...               |

## Original Data Set Non-Compliance with 4NF

### Multi-Valued Dependencies
A professor can give multiple assignments, and each assignment can have multiple relevant readings. These two facts are independent of each other, but they are stored in the same table. 

### Anomalies
If we want to add a new assignment without an associated reading or vice versa, the current design requires us to insert incomplete or placeholder data. Deleting a record that includes a unique reading but shares an assignment with other records would result in the loss of that reading.

The table violates 4NF because it does not separate the two independent multi-valued dependencies (assignments given by professors and readings related to assignments)

## 4NF Compliant Data Set

### Professors Table
Each professor is uniquely identified by professor_id

| professor_id | professor_name | professor_email     |
|--------------|----------------|---------------------|
| 1            | Melvin         | l.melvin@foo.edu    |
| 2            | Logston        | e.logston@foo.edu   |
| 3            | Nevarez        | i.nevarez@foo.edu   |

### Classrooms Table
Each classroom is uniquely identified by classroom_id

| classroom_id | classroom_location |
|--------------|--------------------|
| 101          | WWH 101            |
| 314          | 60FA 314           |
| 201          | WWH 201            |

### Courses Table
Each course is uniquely identified by course_id

| course_id | course_name                          |
|-----------|--------------------------------------|
| 1         | Data normalization                   |
| 2         | Single table queries                 |
| 3         | Python and pandas                    |
| 4         | Spreadsheet aggregate functions      |

### Sections Table
Each section is uniquely identified by section_id

| section_id | course_id | professor_id | classroom_id | semester   |
|------------|-----------|--------------|--------------|------------|
| 1          | 1         | 1            | 101          | Spring 2021|
| 2          | 2         | 2            | 314          | Fall 2021  |
| 3          | 3         | 2            | 314          | Spring 2021|
| 4          | 4         | 3            | 201          | Summer 2021|

### Assignments Table
Each assignment is uniquely identified by assignment_id

| assignment_id | section_id | due_date  | assignment_topic                |
|---------------|------------|-----------|---------------------------------|
| 1             | 1          | 23.02.21  | Data normalization              |
| 2             | 2          | 18.11.21  | Single table queries            |
| 3             | 3          | 05.05.21  | Python and pandas               |
| 4             | 4


## SQL Code for Creating Tables and Inserting Data

CREATE TABLE Professors (
    professor_id INT PRIMARY KEY,
    professor_name VARCHAR(255),
    professor_email VARCHAR(255)
);

INSERT INTO Professors (professor_id, professor_name, professor_email) VALUES
(1, 'Melvin', 'l.melvin@foo.edu'),
(2, 'Logston', 'e.logston@foo.edu'),
(3, 'Nevarez', 'i.nevarez@foo.edu');

The Professors table stores information about each professor, including their name and email address. Each professor is assigned a unique ID (professor_id) to distinguish them in the database.


CREATE TABLE Classrooms (
    classroom_id INT PRIMARY KEY,
    classroom_location VARCHAR(255)
);

INSERT INTO Classrooms (classroom_id, classroom_location) VALUES
(101, 'WWH 101'),
(314, '60FA 314'),
(201, 'WWH 201');

The Classrooms table keeps track of where classes take place. Each classroom gets its own ID number, and the table lists where each one is.

CREATE TABLE Courses (
    course_id INT PRIMARY KEY,
    course_name VARCHAR(255)
);

INSERT INTO Courses (course_id, course_name) VALUES
(1, 'Data normalization'),
(2, 'Single table queries'),
(3, 'Python and pandas'),
(4, 'Spreadsheet aggregate functions');

The Courses table details the courses offered, with each course assigned a unique course_id

CREATE TABLE Sections (
    section_id INT PRIMARY KEY,
    course_id INT,
    professor_id INT,
    classroom_id INT,
    semester VARCHAR(50),
    FOREIGN KEY (course_id) REFERENCES Courses(course_id),
    FOREIGN KEY (professor_id) REFERENCES Professors(professor_id),
    FOREIGN KEY (classroom_id) REFERENCES Classrooms(classroom_id)
);

INSERT INTO Sections (section_id, course_id, professor_id, classroom_id, semester) VALUES
(1, 1, 1, 101, 'Spring 2021'),
(2, 2, 2, 314, 'Fall 2021'),
(3, 3, 2, 314, 'Spring 2021'),
(4, 4, 3, 201, 'Summer 2021');

The Sections table links courses, professors, and classrooms, indicating where and by whom each course section is taught. Each section is uniquely identified by section_id


CREATE TABLE Assignments (
    assignment_id INT PRIMARY KEY,
    section_id INT,
    due_date DATE,
    assignment_topic VARCHAR(255),
    FOREIGN KEY (section_id) REFERENCES Sections(section_id)
);

INSERT INTO Assignments (assignment_id, section_id, due_date, assignment_topic) VALUES
(1, 1, '2021-02-23', 'Data normalization'),
(2, 2, '2021-11-18', 'Single table queries'),
(3, 1, '2021-02-23', 'Data normalization'),
(4, 3, '2021-05-05', 'Python and pandas'),
(5, 4, '2021-07-04', 'Spreadsheet aggregate functions');

The Assignments table assigns unique IDs to each assignment and links them to their respective sections, ensuring that the data is organized according to the normalized schema we've designed. Each Assignment has a due_date and a topic associated with a specific section.

CREATE TABLE Readings (
    reading_id INT PRIMARY KEY,
    assignment_id INT,
    relevant_reading VARCHAR(255),
    FOREIGN KEY (assignment_id) REFERENCES Assignments(assignment_id)
);

INSERT INTO Readings (reading_id, assignment_id, relevant_reading) VALUES
(1, 1, 'Deumlich Chapter 3'),
(2, 2, 'Dümmlers Chapter 11'),
(3, 4, 'Dümmlers Chapter 14'),
(4, 5, 'Zehnder Page 87');

The Readings table stores the  literature that are relevant to each assignment,  creating a link between assignments and their resources.

CREATE TABLE Grades (
    student_id INT,
    assignment_id INT,
    grade INT,
    PRIMARY KEY (student_id, assignment_id),
    FOREIGN KEY (student_id) REFERENCES Students(student_id),
    FOREIGN KEY (assignment_id) REFERENCES Assignments(assignment_id)
);

INSERT INTO Grades (student_id, assignment_id, grade) VALUES
(1, 1, 80),
(7, 2, 25),
(4, 3, 75),
(2, 4, 92),
(2, 5, 65);

The Grades table records the scores that students receive for each assignment. It references both the student_id and the assignment_id, illustrating the many-to-many relationship between students and assignments that is mediated through grades.

CREATE TABLE Students (
    student_id INT PRIMARY KEY,
    student_name VARCHAR(255)
);

INSERT INTO Students (student_id, student_name) VALUES
(1, 'John Doe'),
(2, 'Jane Smith'),
(4, 'Michael Brown'),
(7, 'Linda White');

the Students table lists the students, providing a unique ID and name for each. This structure ensures that student data is stored in a single, centralized location.

## ER Diagram

![ER Diagram](./images/Untitled%Diagram.drawio.svg)



## Changes for 4NF Compliance

The original dataset was restructured into a 4NF-compliant design by addressing the following issues:

### Multi-Valued Dependencies 
The original single table contained multiple pieces of information that were independent of each other, such as assignments and readings, which could lead to data redundancy and anomalies. To resolve this, independent entities were separated into distinct tables, each focusing on a single theme.

### Entity Separation 
Professors, classrooms, courses, and students were each given their own tables with unique identifiers. This separation allows for information about these entities to be stored once and referenced elsewhere in the database, reducing redundancy and simplifying updates.

### Relationships and Associations
The relationships between courses, professors, and classrooms are now explicitly modeled through the `Sections` table. Similarly, assignments are linked to sections, and readings to assignments, clarifying the dependencies and associations between different pieces of data.




