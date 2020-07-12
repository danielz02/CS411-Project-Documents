```mysql
CREATE TABLE Terms(
    TermID INT NOT NULL PRIMARY KEY,
    TermName VARCHAR(10) NOT NULL,
    TermDetailUrl VARCHAR(255) NOT NULL CHECK ( TermDetailUrl LIKE "%https%" ),
    CalendarYear INT NOT NULL,
    PublicIndicator BOOL NOT NULL,
    ArchiveIndicator BOOL NOT NULL,
    AttendingTerm BOOL,
    DefaultTerm BOOL,
    EnrollingTerm BOOL
);
```

```mysql
CREATE TABLE Subjects(
    SubjectID VARCHAR(10) NOT NULL PRIMARY KEY,
    SubjectName VARCHAR(100) NOT NULL,
    DepartmentCode VARCHAR(5) NOT NULL,
    TermID INT NOT NULL,
    FOREIGN KEY (TermID) REFERENCES Terms(TermID) ON UPDATE CASCADE ON DELETE CASCADE
);
```

```mysql
CREATE TABLE Departments(
    DepartmentName VARCHAR(255),
    CollegeCode VARCHAR(5),
    DepartmentCode INT PRIMARY KEY,
    ContactName VARCHAR(255),
    ContactTitle VARCHAR(255),
    AddressLine1 VARCHAR(255),
    AddressLine2 VARCHAR(255),
    PhoneNumber VARCHAR(20),
    Url VARCHAR(255),
    DepartmentDescription VARCHAR(255)
);
```

```mysql
CREATE TABLE Courses (
    SubjectID VARCHAR(10) NOT NULL,
    TermID INT NOT NULL,
    CourseID INT NOT NULL,
    CourseName VARCHAR(255) NOT NULL,
    CourseDescription VARCHAR(255),
    CourseSectionInformation VARCHAR(255),
    SectionDegreeAttributes VARCHAR(255),
    SectionRegistrationNotes VARCHAR(255),
    ClassScheduleInformation VARCHAR(255),
    GenEdCategories VARCHAR(10),
    PRIMARY KEY (SubjectID, TermID, CourseID),
    FOREIGN KEY (TermID) REFERENCES Terms(TermID) ON DELETE CASCADE ON UPDATE CASCADE,
    FOREIGN KEY (SubjectID) REFERENCES Subjects(SubjectID) ON DELETE CASCADE ON UPDATE CASCADE
);
```

+ `SubjectID` is the abbreviation of the subject. For example, `CS`  and `ECE`.
+ `CourseID` is the identifier of the course under a certain subject. For example, the `CourseID` for `CS411` is `411`.
+ `TermID` is an unique identification generated by course explorer for each term.
+ `Credits` is a string encoded by ourselves. We need to do this because some courses have different options for credit hours.
+ `CourseSectionInformation` usually describes prerequisites.
+ `SectionDegreeAttributes` usually describes the GenEd attribute that the course could fulfill.
+ `GenEdCategories` is an encoded string of GenEd codes. For example, `QR:1QR1` means Quantitative Reasoning I.

```mysql
CREATE INDEX Courses_CourseID_index
	ON Courses (CourseID);

CREATE TABLE Sections(
    CRN INT NOT NULL,
    TermID INT NOT NULL,
    CourseID INT NOT NULL,
    SubjectID VARCHAR(10) NOT NULL,
    SectionNumber VARCHAR(10) NOT NULL,
    Credits INT NOT NULL,
    StatusCode VARCHAR(10) NOT NULL ,
    PartOfTerm VARCHAR(5) NOT NULL,
    EnrollmentStatus VARCHAR(255) NOT NULL,
    SectionCappArea VARCHAR(255),
    StartDate DATE,
    EndDate DATE,
    SectionText VARCHAR(255),
    PRIMARY KEY (CRN, TermID),
    FOREIGN KEY (TermID) REFERENCES Terms(TermID),
    FOREIGN KEY (SubjectID) REFERENCES Subjects(SubjectID),
    FOREIGN KEY (CourseID) REFERENCES Courses(CourseID)
);
```

+ We use `CRN` and `TermID` as primary key because `CRN` is only unique within a certain term. For example, 2020 Fall's `CS 225` section `AL1 (35917)` has the same CRN as 2020 Spring's `LAW 610`.

+ Since `CourseID` is a foreign key, it has to be in an index of the referenced relation where `CourseID` is the starting column.

  > MySQL requires indexes on foreign keys and referenced keys so that foreign key checks can be fast and not require a table scan. In the referencing table, there must be an index where the foreign key columns are listed as the first columns in the same order.
  >
  > InnoDB permits a foreign key to reference any index column or group of columns. However, in the referenced table, there must be an index where the referenced columns are listed as the first columns in the same order.

```mysql
CREATE TABLE Meetings(
    CRN INT NOT NULL,
    TermID INT NOT NULL,
    TypeCode VARCHAR(5) NOT NULL,
    TypeName VARCHAR(255) NOT NULL,
    StartTime TIME,
    EndTime TIME,
    DaysOfWeek VARCHAR(5),
    BuildingName VARCHAR(255),
    RoomNumber INT,
    PRIMARY KEY (CRN, TermID),
    FOREIGN KEY (CRN, TermID) REFERENCES Sections(CRN, TermID)
);
```

```mysql
CREATE TABLE Users(
    UID VARCHAR(255) NOT NULL,
    Email VARCHAR(255) NOT NULL,
    Name VARCHAR(255) NOT NULL,
    SaltedPassword VARCHAR(255) NOT NULL
);
```
