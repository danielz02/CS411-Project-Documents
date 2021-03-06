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
    SubjectID VARCHAR(10) NOT NULL,
    SubjectName VARCHAR(100) NOT NULL,
    DepartmentCode VARCHAR(5) NOT NULL,
    TermID INT NOT NULL,
    PRIMARY KEY(SubjectID, TermID),
    FOREIGN KEY (TermID) REFERENCES Terms(TermID) ON UPDATE CASCADE ON DELETE CASCADE
);
```

```mysql
CREATE TABLE Departments(
    TermID INT,
    SubjectID VARCHAR(5),
    DepartmentName VARCHAR(255),
    CollegeCode VARCHAR(5),
    DepartmentCode INT,
    ContactName VARCHAR(255),
    ContactTitle VARCHAR(255),
    AddressLine1 VARCHAR(255),
    AddressLine2 VARCHAR(255),
    PhoneNumber VARCHAR(40),
    Url VARCHAR(255),
    DepartmentDescription VARCHAR(255),
    PRIMARY KEY (TermID, DepartmentCode, SubjectID),
    FOREIGN KEY (TermID) REFERENCES Terms(TermID)
);
```

```mysql
CREATE TABLE Courses (
    SubjectID VARCHAR(10) NOT NULL,
    TermID INT NOT NULL,
    CourseID INT NOT NULL,
    CourseName VARCHAR(255) NOT NULL,
    CreditHours VARCHAR(20) NOT NULL,
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
    SectionNumber VARCHAR(10),
    Credits INT,
    StatusCode VARCHAR(10) NOT NULL,
    PartOfTerm VARCHAR(5) NOT NULL,
    EnrollmentStatus VARCHAR(255) NOT NULL,
    SectionText VARCHAR(255),
    SectionNotes VARCHAR(255),
    SectionCappArea VARCHAR(255),
    StartDate DATE,
    EndDate DATE,
    PRIMARY KEY (CRN, TermID),
    FOREIGN KEY (TermID) REFERENCES Terms(TermID) ON DELETE CASCADE ON UPDATE CASCADE,
    FOREIGN KEY (SubjectID) REFERENCES Subjects(SubjectID) ON DELETE CASCADE ON UPDATE CASCADE,
    FOREIGN KEY (CourseID) REFERENCES Courses(CourseID) ON DELETE CASCADE ON UPDATE CASCADE
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
    MeetingID INT NOT NULL,
    TypeCode VARCHAR(5) NOT NULL,
    TypeName VARCHAR(255) NOT NULL,
    StartTime TIME,
    EndTime TIME,
    DaysOfWeek VARCHAR(5),
    BuildingName VARCHAR(255),
    RoomNumber INT,
    FOREIGN KEY (CRN, TermID) REFERENCES Sections(CRN, TermID) ON UPDATE CASCADE ON DELETE CASCADE
);
```

```mysql
CREATE TABLE Users(
    UUID VARCHAR(255) PRIMARY KEY,
    Email VARCHAR(255) UNIQUE NOT NULL,
    Name VARCHAR(255) NOT NULL,
    SaltedPassword VARCHAR(255) NOT NULL
);
```

```mysql
CREATE TABLE Enrollments(
    UUID VARCHAR(255) NOT NULL,
    TermID INT NOT NULL,
    CRN INT NOT NULL,
    PRIMARY KEY (UUID, TermID, CRN),
    FOREIGN KEY (UUID) REFERENCES Users(UUID) ON UPDATE CASCADE ON DELETE CASCADE,
    FOREIGN KEY (TermID) REFERENCES Terms(TermID) ON UPDATE CASCADE ON DELETE CASCADE,
    FOREIGN KEY (CRN) REFERENCES Sections(CRN) ON UPDATE CASCADE ON DELETE CASCADE
);
```

```mysql
CREATE TABLE Instructors(
    CRN INT NOT NULL,
    TermID INT NOT NULL,
    MeetingID INT NOT NULL,
    FullName VARCHAR(255),
    LastName VARCHAR(255),
    FirstName VARCHAR(255),
    PRIMARY KEY (CRN, TermID, MeetingID, FullName),
    FOREIGN KEY (CRN) REFERENCES Sections(CRN) ON UPDATE CASCADE ON DELETE CASCADE,
    FOREIGN KEY (TermID) REFERENCES Terms(TermID) ON UPDATE CASCADE ON DELETE CASCADE
);
```

+ Instructor information can be `null`

```mysql
CREATE TABLE Remarks(
    RID INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    UID VARCHAR(255) NOT NULL,
    CRN INT NOT NULL,
    TermID INT NOT NULL,
    Remark VARCHAR(255) NOT NULL,
    FOREIGN KEY (CRN) REFERENCES Enrollments(CRN),
    FOREIGN KEY (TermID) REFERENCES Enrollments(TermID),
    FOREIGN KEY (UID) REFERENCES Enrollments(UUID) ON UPDATE CASCADE ON DELETE CASCADE
);
```

```mysql
CREATE TABLE GPA(
    TermID INT NOT NULL,
    TermName VARCHAR(20) NOT NULL,
    CRN INT NOT NULL,
    SubjectID VARCHAR(10) NOT NULL,
    CourseID INT NOT NULL,
    CourseName VARCHAR(255) NOT NULL,
    TypeCode VARCHAR(5) NOT NULL,
    Ap INT,
    A INT,
    Am INT,
    Bp INT,
    B INT,
    Bm INT,
    Cp INT,
    C INT,
    Cm INT,
    Dp INT,
    D INT,
    Dm INT,
    F INT,
    W INT,
    PrimaryInstructor VARCHAR(255),
    PRIMARY KEY (TermID, CRN),
    FOREIGN KEY (TermID) REFERENCES Terms(TermID),
    FOREIGN KEY (CRN) REFERENCES Sections(CRN)
);
```

```mysql
DELIMITER ;;

CREATE PROCEDURE SearchCourse(IN CRN INT, CourseName VARCHAR(255), SubjectID VARCHAR(5), ID INT, IsCurrentTerm BOOL, NumRecords INT)
BEGIN
    DECLARE CurrentTerm INT;
    SELECT TermID INTO CurrentTerm FROM Terms WHERE AttendingTerm = TRUE;

    SET NumRecords = IFNULL(NumRecords, 50);
    SET IsCurrentTerm = IFNULL(IsCurrentTerm, TRUE);

    IF CRN IS NULL AND CourseName IS NULL AND SubjectID IS NULL AND ID IS NULL THEN
        SIGNAL SQLSTATE VALUE '23333'
        SET MESSAGE_TEXT = 'Don\'t even try to blow up the database! You must specify at least one search field!';
    END IF ;

    SELECT c.TermID, t.TermName, s.CRN, c.SubjectID, c.CourseID, c.CourseName,
           m.TypeName, i.FullName, m.DaysOfWeek, m.StartTime, m.EndTime, s.StartDate
    FROM Terms t NATURAL JOIN Sections s NATURAL JOIN Meetings m NATURAL JOIN Courses c NATURAL JOIN Instructors i
    WHERE s.CRN = IFNULL(CRN, s.CRN) AND c.CourseName LIKE CONCAT('%', IFNULL(CourseName, c.CourseName), '%') AND
          c.SubjectID = IFNULL(SubjectID, c.SubjectID) AND c.CourseID = IFNULL(ID, c.CourseID) AND
          (NOT IsCurrentTerm OR c.TermID = CurrentTerm)
    ORDER BY TermID DESC
    LIMIT NumRecords;

END ;;

DELIMITER ;


DELIMITER ;;

CREATE PROCEDURE AddRemark(IN Email VARCHAR(255), CRN INT, TermID INT, Remark VARCHAR(255))
BEGIN
    DECLARE UID VARCHAR(255);

    IF Email IS NULL OR CRN IS NULL OR TermID IS NULL OR Remark IS NULL THEN
        SIGNAL SQLSTATE VALUE '23333'
        SET MESSAGE_TEXT = 'Missing field!';
    END IF ;

    SELECT UUID INTO UID FROM Users u WHERE u.Email = Email;

    INSERT INTO Remarks(UID, CRN, TermID, Remark) VALUE (UID, CRN, TermID, Remark);
    SELECT RID, UID, CRN, TermID, Remark FROM Remarks WHERE RID = LAST_INSERT_ID();
END ;;

DELIMITER ;

DELIMITER ;;

CREATE PROCEDURE ModifyRemark(IN TargetRID INT, userEmail VARCHAR(255), TargetCRN INT, TargetTermID INT, NewRemark VARCHAR(255))
BEGIN
    DECLARE userID VARCHAR(255);

    IF userEmail IS NULL OR TargetCRN IS NULL OR TargetTermID IS NULL OR NewRemark IS NULL THEN
        SIGNAL SQLSTATE VALUE '23333'
        SET MESSAGE_TEXT = 'Missing field!';
    END IF ;

    SELECT UUID INTO userID FROM Users u WHERE u.Email = userEmail;

    UPDATE Remarks SET Remark = NewRemark WHERE RID = TargetRID AND UID = userID AND CRN = TargetCRN AND TermID = TargetTermID;
    SELECT RID, UID, CRN, TermID, Remark FROM Remarks WHERE RID = TargetRID;
END ;;

DELIMITER ;


DELIMITER ;;

CREATE PROCEDURE GetRemark(IN UserEmail VARCHAR(255), TargetCRN INT, TargetTermID INT, SearchString VARCHAR(255))
BEGIN
    DECLARE UserID VARCHAR(255);

    IF UserEmail IS NULL AND TargetCRN IS NULL AND TargetTermID IS NULL THEN
        SIGNAL SQLSTATE VALUE '23333'
        SET MESSAGE_TEXT = 'You must specify at least one search field!';
    END IF ;

    SELECT UUID INTO UserID FROM Users u WHERE u.Email = UserEmail;

    SELECT RID, Remark, TermID, CRN, CourseName, SubjectID, CourseID
    FROM Remarks r NATURAL JOIN Courses NATURAL JOIN Sections s
    WHERE s.CRN = IFNULL(TargetCRN, s.CRN) AND s.TermID = IFNULL(TargetTermID, s.TermID) AND
          r.UID = IFNULL(IFNULL(UserEmail, UserID), r.UID) AND
          r.Remark LIKE IFNULL(CONCAT('%', SearchString, '%'), r.Remark);
END ;;

DELIMITER ;
```

```mysql
CREATE TABLE Ratings(
    TeacherID INT NOT NULL PRIMARY KEY,
    NumRatings INT NOT NULL,
    FirstName VARCHAR(255),
    LastName VARCHAR(255),
    AvgRating REAL NOT NULL
);
```

```mysql
CREATE TABLE Comments(
    CID INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    ID VARCHAR(63) NOT NULL,
    LegacyID INT NOT NULL ,
    FirstName VARCHAR(255) NOT NULL,
    LastName VARCHAR(255) NOT NULL,
    Class VARCHAR(255) NOT NULL,
    Tags VARCHAR(255) NOT NULL,
    IsAttendanceMandatory BOOL,
    Clarity INT,
    DifficultyRating INT,
    HelpfulRating INT,
    WouldTakeAgain BOOL,
    IsForCredit BOOL,
    IsOnline BOOL,
    Grade VARCHAR(7),
    Comment TEXT,
    Date DATETIME
);
```



```mysql
-- auto-generated definition
create table Terms
(
    TermID           int          not null
        primary key,
    TermName         varchar(10)  not null,
    TermDetailUrl    varchar(255) not null,
    CalendarYear     int          not null,
    PublicIndicator  tinyint(1)   not null,
    ArchiveIndicator tinyint(1)   not null,
    AttendingTerm    tinyint(1)   null,
    DefaultTerm      tinyint(1)   null,
    EnrollingTerm    tinyint(1)   null
);


-- auto-generated definition
create table Subjects
(
    SubjectID      varchar(10)  not null
        primary key,
    SubjectName    varchar(100) not null,
    DepartmentCode varchar(5)   not null,
    TermID         int          not null
);

create index TermID
    on Subjects (TermID);

-- auto-generated definition
create table Departments
(
    DepartmentName        varchar(255) null,
    CollegeCode           varchar(5)   null,
    DepartmentCode        int          not null
        primary key,
    ContactName           varchar(255) null,
    ContactTitle          varchar(255) null,
    AddressLine1          varchar(255) null,
    AddressLine2          varchar(255) null,
    PhoneNumber           varchar(20)  null,
    Url                   varchar(255) null,
    DepartmentDescription varchar(255) null
);


-- auto-generated definition
create table Courses
(
    SubjectID                varchar(10)  not null,
    TermID                   int          not null,
    CourseID                 int          not null,
    CourseName               varchar(255) not null,
    CourseDescription        varchar(255) null,
    CourseSectionInformation varchar(255) null,
    SectionDegreeAttributes  varchar(255) null,
    SectionRegistrationNotes varchar(255) null,
    ClassScheduleInformation varchar(255) null,
    GenEdCategories          varchar(10)  null,
    primary key (SubjectID, TermID, CourseID),
    constraint Courses_ibfk_1
        foreign key (TermID) references Terms (TermID)
            on update cascade on delete cascade,
    constraint Courses_ibfk_2
        foreign key (SubjectID) references Subjects (SubjectID)
            on update cascade on delete cascade
);

create index Courses_CourseID_index
    on Courses (CourseID);

create index TermID
    on Courses (TermID);

-- auto-generated definition
create table Sections
(
    CRN                        int          not null,
    TermID                     int          not null,
    CourseID                   int          not null,
    SubjectID                  varchar(10)  not null,
    SectionNumber              varchar(10)  not null,
    Credits                    int          not null,
    StatusCode                 varchar(10)  not null,
    PartOfTerm                 varchar(5)   not null,
    EnrollmentStatus           varchar(255) not null,
    InstructorFirstNameInitial varchar(10)  null,
    InstructorLastName         varchar(255) null,
    SectionCappArea            varchar(255) null,
    StartDate                  date         null,
    EndDate                    date         null,
    SectionText                varchar(255) null,
    primary key (CRN, TermID),
    constraint Sections_ibfk_1
        foreign key (TermID) references Terms (TermID),
    constraint Sections_ibfk_2
        foreign key (SubjectID) references Subjects (SubjectID),
    constraint Sections_ibfk_3
        foreign key (CourseID) references Courses (CourseID)
);

create index CourseID
    on Sections (CourseID);

create index SubjectID
    on Sections (SubjectID);

create index TermID
    on Sections (TermID);

-- auto-generated definition
create table Meetings
(
    CRN          int          not null,
    TermID       int          not null,
    TypeCode     varchar(5)   not null,
    TypeName     varchar(255) not null,
    StartTime    time         null,
    EndTime      time         null,
    DaysOfWeek   varchar(5)   null,
    BuildingName varchar(255) null,
    RoomNumber   int          null,
    primary key (CRN, TermID),
    constraint Meetings_ibfk_1
        foreign key (CRN, TermID) references Sections (CRN, TermID)
);


-- auto-generated definition
create table Enrollments
(
    TermID int not null,
    CRN    int not null,
    primary key (TermID, CRN),
    constraint Enrollments_ibfk_1
        foreign key (TermID) references Terms (TermID)
            on update cascade on delete cascade,
    constraint Enrollments_ibfk_2
        foreign key (CRN) references Sections (CRN)
            on update cascade on delete cascade
);

create index CRN
    on Enrollments (CRN);

-- auto-generated definition
create table Users
(
    UID            varchar(255) not null,
    Email          varchar(255) not null,
    Name           varchar(255) not null,
    SaltedPassword varchar(255) not null
);
```

```json
PUT /ratings/
{
  "settings": {
    "analysis": {
      "filter": {
        "autocomplete_filter": {
          "type": "edge_ngram",
          "min_gram": 1,
          "max_gram": 20
        }
      },
      "analyzer": {
        "autocomplete": { 
          "type": "custom",
          "tokenizer": "whitespace",
          "filter": [
            "lowercase",
            "autocomplete_filter"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "teacherID": {
        "type": "keyword"
      },
      "firstName": {
        "index": true,
        "type": "text",
        "analyzer": "autocomplete",
        "similarity": "BM25", 
        "search_analyzer": "standard"
      },
      "lastName": {
        "index": true,
        "type": "text",
        "analyzer": "autocomplete",
        "similarity": "BM25", 
        "search_analyzer": "standard"
      },
      "fullName": {
        "index": true,
        "type": "text",
        "analyzer": "autocomplete",
        "similarity": "BM25", 
        "search_analyzer": "standard" 
      },
      "numRatings": {
        "type": "integer"
      },
      "avgRating": {
        "type": "float"
      }
    }
  }
}
```

```json
PUT /comments/
{
  "settings": {
    "analysis": {
      "filter": {
        "autocomplete_filter": {
          "type": "edge_ngram",
          "min_gram": 1,
          "max_gram": 20
        }
      },
      "analyzer": {
        "autocomplete": { 
          "type": "custom",
          "tokenizer": "whitespace",
          "filter": [
            "lowercase",
            "autocomplete_filter"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "teacherID": {
        "type": "keyword"
      },
      "id": {
        "index": true,
        "type": "keyword"
      },
      "legacyId": {
        "index": true,
        "type": "keyword"
      },
      "firstName": {
        "index": true,
        "type": "text",
        "similarity": "BM25",
        "analyzer": "autocomplete", 
        "search_analyzer": "autocomplete" 
      },
      "lastName": {
        "index": true,
        "type": "text",
        "similarity": "BM25",
        "analyzer": "autocomplete", 
        "search_analyzer": "autocomplete" 
      },
      "fullName": {
        "index": true,
        "type": "text",
        "similarity": "BM25", 
        "analyzer": "autocomplete", 
        "search_analyzer": "autocomplete" 
      },
      "class": {
        "index": true,
        "type": "keyword",
        "similarity": "BM25"
      },
      "tags": {
        "index": true,
        "type": "text",
        "similarity": "BM25", 
        "analyzer": "autocomplete", 
        "search_analyzer": "autocomplete" 
      },
      "isAttendanceMandatory": {
        "type": "boolean"
      },
      "clarity": {
        "type": "integer"
      },
      "difficultyRating": {
        "type": "integer"
      },
      "helpfulRating": {
        "type": "integer"
      },
      "wouldTakeAgain": {
        "type": "boolean"
      },
      "isForCredit": {
        "type": "boolean"
      },
      "isOnline": {
        "type": "boolean"
      },
      "grade": {
        "type": "text"
      },
      "comment": {
        "index": true,
        "type": "text",
        "similarity": "BM25", 
        "analyzer": "autocomplete", 
        "search_analyzer": "autocomplete" 
      },
      "date": {
        "type": "date"
      }
    }
  }
}

GET /comments/_search
```

