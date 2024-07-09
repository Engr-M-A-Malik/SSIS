# SSIS Package Documentation

## Project Overview

This SSIS package project transforms employee data from one database to another, performs various calculations, and handles errors appropriately. The package consists of two main data flow tasks and an event handler for logging errors.

## Source Database

### Database: `70-461`

#### Table: `tblemployee`

| Column Name          | Data Type   | Description                       |
|----------------------|-------------|-----------------------------------|
| EmployeeNumber       | int         | Employee identifier               |
| EmployeeFirstName    | varchar(50) | Employee's first name             |
| EmployeeMiddleName   | varchar(50) | Employee's middle name (nullable) |
| EmployeeLastName     | varchar(50) | Employee's last name              |
| EmployeeGovernmentID | char(10)    | Government ID of the employee     |
| DateOfBirth          | date        | Employee's date of birth          |
| Department           | varchar(19) | Employee's department (nullable)  |

## Intermediate Database

### Database: `source_database`

#### Table: `employee_info`

| Column Name          | Data Type   | Description                       |
|----------------------|-------------|-----------------------------------|
| EmployeeFirstName    | varchar(50) | Employee's first name             |
| EmployeeMiddleName   | varchar(50) | Employee's middle name (nullable) |
| EmployeeLastName     | varchar(50) | Employee's last name              |
| DateOfBirth          | nvarchar(10)| Employee's date of birth          |
| Salary               | int         | Calculated salary                 |

## Destination Database

### Database: `destination_database`

#### Table: `Transformed_Employee_Info`

| Column Name          | Data Type   | Description                       |
|----------------------|-------------|-----------------------------------|
| Salary               | int         | Calculated salary                 |
| Full_Name            | varchar(152)| Concatenated full name            |
| Age                  | int         | Calculated age from DOB           |
| Salary_Grade         | varchar(2)  | Assigned salary grade             |

#### Table: `LogError`

| Column Name          | Data Type   | Description                       |
|----------------------|-------------|-----------------------------------|
| id                   | int         | Primary key, auto-incremented     |
| Package_Name         | varchar(50) | Name of the SSIS package          |
| Error_Messages       | varchar(max)| Detailed error message            |

## Data Flow Tasks

### Data Flow Task 1: Salary Calculation

1. **Source:**
   - Table: `tblemployee` from the database `70-461`
   
   ![Employee Data Retrieval](upload your 1st image here)

2. **Derived Column Transformation: Salary Calculation**
   - Expression: `EmployeeNumber * 1000`
   
   ![Salary Calculation](upload your 1st image here)

3. **Destination:**
   - Table: `employee_info` in the database `source_database`
   
   ![Loading into Source](upload your 1st image here)

### Data Flow Task 2: Transformations and Loading

1. **Source:**
   - Table: `employee_info` from the database `source_database`
   
   ![Extracting Data From Source](upload your 2nd image here)

2. **Derived Column Transformation: Full Name Calculation**
   - Expression: `(DT_STR,50,1252)(EmployeeFirstName + " " + (ISNULL(EmployeeMiddleName) ? "" : EmployeeMiddleName + " ") + EmployeeLastName)`
   
   ![Full_Name Calculation](upload your 2nd image here)

3. **Derived Column Transformation: Age Calculation**
   - Expression: `(DT_I4)((DATEDIFF("DD",(DT_DBTIMESTAMP)DateOfBirth,GETDATE()) / 365.25))`
   
   ![Age Calculation](upload your 2nd image here)

4. **Derived Column Transformation: Salary Grade Calculation**
   - Expression: `(DT_STR,2,1252)(Salary > 1000000 ? "A" : Salary > 500000 ? "B" : Salary > 250000 ? "C" : "D")`
   
   ![Salary_Grade Calculation](upload your 2nd image here)

5. **Destination:**
   - Table: `Transformed_Employee_Info` in the database `destination_database`
   
   ![Destination Loading](upload your 2nd image here)

## Control Flow

The control flow integrates both data flow tasks and ensures they are executed sequentially.

![Control Flow](upload your 3rd image here)

## Event Handler

### OnError Event Handler

This event handler captures errors during package execution and logs them into the `LogError` table in the `destination_database`.

1. **Execute SQL Task: Log Error**
   - SQL Command: `insert into LogError(Package_Name,Error_Messages) values(?,?)`
   - Parameters:
     - `Package_Name`: System variable `System::PackageName`
     - `Error_Messages`: System variable `System::ErrorDescription`
   
   ![Error Handling](upload your 4th image here)

## Expressions Used in Derived Column Transformations

### Data Flow Task 1: Salary Calculation
- **Expression:** `EmployeeNumber * 1000`

### Data Flow Task 2: Full Name Calculation
- **Expression:** `(DT_STR,50,1252)(EmployeeFirstName + " " + (ISNULL(EmployeeMiddleName) ? "" : EmployeeMiddleName + " ") + EmployeeLastName)`

### Data Flow Task 2: Age Calculation
- **Expression:** `(DT_I4)((DATEDIFF("DD",(DT_DBTIMESTAMP)DateOfBirth,GETDATE()) / 365.25))`

### Data Flow Task 2: Salary Grade Calculation
- **Expression:** `(DT_STR,2,1252)(Salary > 1000000 ? "A" : Salary > 500000 ? "B" : Salary > 250000 ? "C" : "D")`

## Conclusion

This documentation provides a comprehensive overview of the SSIS package, including the data flows, transformations, and error handling mechanisms. Each component is explained in detail to ensure clarity and ease of understanding for future maintenance and enhancements.

