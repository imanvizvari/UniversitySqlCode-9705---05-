# UniversitySqlCode-9705---05-

CREATE DATABASE University 
GO
CREATE TABLE Person
(
	Id int IDENTITY PRIMARY KEY, 
	Name nvarchar(50) NOT NULL,
	Family nvarchar(50) NOT NULL,
	NationalCode char(12) UNIQUE,
	UserName varchar(50) UNIQUE,
	[Password] nvarchar(50),
	CellPhone char(11) UNIQUE,
	BirthDate date,
	CreateDate datetime DEFAULT GETDATE() 
)
GO
CREATE TABLE Student
(
	Id int PRIMARY KEY IDENTITY,
	PersonId int FOREIGN KEY REFERENCES Person(Id) ON DELETE CASCADE ON UPDATE CASCADE,
	StdNumber nvarchar(10) NOT NULL
)
GO
CREATE TABLE Teacher
(
	Id int PRIMARY KEY IDENTITY,
	PersonId int FOREIGN KEY REFERENCES Person(Id) ON DELETE CASCADE ON UPDATE CASCADE,
	PersonalityId nvarchar(10) NOT NULL,
	Field nvarchar(50) NOT NULL
)
GO
CREATE TABLE Cource
(
	Id int PRIMARY KEY IDENTITY,
	NameCource nvarchar(50) NOT NULL UNIQUE,
	CourceTime int NOT NULL,
	UnitCount int
)
GO
CREATE TABLE Cource_Cyllabus
(
	Id int PRIMARY KEY IDENTITY,
	CourceId int FOREIGN KEY REFERENCES Cource(Id),
	[Session] nvarchar(50) NOT NULL,
	Detail nvarchar(100) NOT NULL,
	CreateDate datetime DEFAULT GETDATE() 
)
GO
CREATE TABLE Pre_Cource
(
	CourceId int FOREIGN KEY REFERENCES Cource(Id),
	ParentId int FOREIGN KEY REFERENCES Cource(Id)
)
GO
CREATE TABLE Term
(
	Id int PRIMARY KEY NOT NULL,
	StartTime date NOT NULL,
	EndTime date NOT NULL,
	NameTerm nvarchar(20)
)
GO
CREATE TABLE Cource_Term
(
	Id int IDENTITY PRIMARY KEY,
	TermId int FOREIGN KEY REFERENCES Term(Id),
	TeacherId int FOREIGN KEY REFERENCES Teacher(Id),
	CourceId int FOREIGN KEY REFERENCES Cource(Id),
	ExamDate datetime NOT NULL
)
GO
CREATE TABLE Student_Cource_Term
(
	StudentId int FOREIGN KEY REFERENCES Student(Id),
	Cource_TermId int FOREIGN KEY REFERENCES Cource_Term(Id)
)
GO
CREATE TABLE Class
(
	Id int IDENTITY PRIMARY KEY,
	ClassName nvarchar(50) NOT NULL,
	ClassAddress nvarchar(200) NOT NULL
)

GO
CREATE TABLE Scores
(
	Id int IDENTITY PRIMARY KEY,
	Cource_TermId int FOREIGN KEY REFERENCES Cource_Term(Id),
	StudentId int FOREIGN KEY REFERENCES Student(Id),
	Score numeric(4,2)
)
GO

CREATE TABLE ScheduleCource
(
	Id int IDENTITY PRIMARY KEY,
	ClassId int FOREIGN KEY REFERENCES Class(Id),
	Cource_TermId int FOREIGN KEY REFERENCES Cource_Term(Id),
	HoldingDate datetime NOT NULL 
)
GO
CREATE TABLE Call_The_Rolle
(
	Id int IDENTITY PRIMARY KEY,
	ScheduleCourceId int FOREIGN KEY REFERENCES ScheduleCource(Id),
	StudentId int FOREIGN KEY REFERENCES Student(Id),
	[Status] bit 
)
GO
CREATE TABLE CourceDetail
(
	Id int IDENTITY PRIMARY KEY,
	ScheduleCourceId int FOREIGN KEY REFERENCES ScheduleCource(Id),
	Title nvarchar(50) NOT NULL,
	Detail nvarchar(100)
)
GO
CREATE TABLE Holiday
(
	Id int PRIMARY KEY,
	Vacation date
)
GO

CREATE TABLE Vehicle
(
	Id int IDENTITY PRIMARY KEY,
	PersonId int FOREIGN KEY REFERENCES Person(Id),
	Plate nvarchar(10),
	VehicleName nvarchar(10),
	VehicleColor nvarchar(10)
)


-------------------------------------FUNCTION
GO
CREATE FUNCTION Fk_Check_Holiday
(
	@Date DATE
)
RETURNS nvarchar(10)
AS 
BEGIN
	IF EXISTS (SELECT Vacation FROM Holiday 
		   WHERE Vacation=@Date)
	     RETURN'True'
        RETURN'False'
END
-------------------------------------CONSTRAINT
GO
ALTER TABLE ScheduleCource WITH CHECK ADD CONSTRAINT CK_HOLIDAY CHECK (dbo.Fk_Check_Holiday(HoldingDate)='False')

-------------------------------------PROC1
GO
CREATE PROC SP_Control_Input_Parking
(
	@PlateNumber nvarchar(10)
)
AS
BEGIN
	IF EXISTS
		 (SELECT Plate FROM Vehicle 
				WHERE Plate=@PlateNumber)
				return'True'
	ELSE RETURN'False'
END	

---------------------------------------------------------------PROC2
GO
CREATE PROC SP_Teacher_ClassTime_Parking
(
	@PlateNumber nvarchar(10),
	@Id int
)
AS
BEGIN
	IF EXISTS 
			(SELECT V.Plate,T.Id,SC.HoldingDate 
			FROM Vehicle V
			INNER JOIN Teacher T
			ON V.PersonId=T.PersonId
			INNER JOIN Cource_Term CT
			ON T.Id=CT.TeacherId
			INNER JOIN dbo.Schedule_Cource SC
			ON CT.Id= SC.Cource_TermId 
			WHERE V.Plate=@PlateNumber AND T.Id=@Id AND CONVERT(date,HoldingDate)=CONVERT(date,GETDATE()))
		RETURN'True'
	ELSE
		RETURN'False'
END



-------------------------------------------PROC3
GO

CREATE PROC SP_Student_1Hour_Parking
(
	@PlateNumber nvarchar(10),
	@Id int
)
AS
BEGIN
	IF EXISTS 
			(SELECT V.Plate,S.Id,SC.HoldingDate 
			FROM Vehicle V
			INNER JOIN Student S
			ON V.PersonId=S.PersonId
			INNER JOIN Student_Cource_Term SCT
			ON S.Id=SCT.StudentId
			INNER JOIN dbo.Schedule_Cource SC
			ON SCT.Cource_TermId=SC.Cource_TermId 
			WHERE V.Plate=@PlateNumber AND S.Id=@Id AND GETDATE()>= DATEADD(HOUR,-1,HoldingDate))
		RETURN'True'
	ELSE
		RETURN'False'
END
------------------------------------------

GO
CREATE PROC SP_Student_ExamTime_Parking
(
	@PlateNumber nvarchar(10),
	@Id int
)
AS
BEGIN
	IF EXISTS 
			(SELECT V.Plate,S.Id,CT.ExamDate 
			FROM Vehicle V
			INNER JOIN Student S
			ON V.PersonId=S.PersonId
			INNER JOIN Student_Cource_Term SCT
			ON S.Id=SCT.StudentId
			INNER JOIN Cource_Term CT
			ON CT.Id=SCT.Cource_TermId 
			WHERE V.Plate=@PlateNumber AND S.Id=@Id AND CONVERT(date,ExamDate)=CONVERT(date,GETDATE()))
		RETURN'True'
	ELSE
		RETURN'False'
END
