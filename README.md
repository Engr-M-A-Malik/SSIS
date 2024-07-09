# SSIS Package Project

## Project Overview

This project involves transforming employee data from one database to another, performing various calculations, and handling errors appropriately. The project consists of two main data flow tasks and an event handler for logging errors.

## Databases and Tables

### Source Database: `70-461`

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

### Intermediate Database: `source_database`

#### Table: `employee_info`

| Column Name          | Data Type   | Description                       |
|----------------------|-------------|-----------------------------------|
| EmployeeFirstName    | varchar(50) | Employee's first name             |
| EmployeeMiddleName   | varchar(50) | Employee's middle name (nullable) |
| EmployeeLastName     | varchar(50) | Employee's last name              |
| DateOfBirth          | nvarchar(10)| Employee's date of birth          |
| Salary               | int         | Calculated salary                 |

### Destination Database: `destination_database`

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

1. **Source:** Table `tblemployee` from the database `70-461`.
2. **Derived Column Transformation:** Calculate salary by multiplying `EmployeeNumber` by 1000.
3. **Destination:** Table `employee_info` in the database `source_database`.

### Data Flow Task 2: Transformations and Loading

1. **Source:** Table `employee_info` from the database `source_database`.
2. **Derived Column Transformation:**
   - **Full Name Calculation:** Concatenate `EmployeeFirstName`, `EmployeeMiddleName`, and `EmployeeLastName`.
   - **Age Calculation:** Calculate age from `DateOfBirth`.
   - **Salary Grade Calculation:** Assign salary grade based on salary.
3. **Destination:** Table `Transformed_Employee_Info` in the database `destination_database`.

## Control Flow

The control flow integrates both data flow tasks and ensures they are executed sequentially.

## Event Handler

### OnError Event Handler

Captures errors during package execution and logs them into the `LogError` table in the `destination_database`.

1. **Execute SQL Task:** Log error details with SQL command:
   ```sql
   insert into LogError(Package_Name,Error_Messages)
   values(?,?)
   
## Parameters

- **Package_Name:** System variable `System::PackageName`
- **Error_Messages:** System variable `System::ErrorDescription`

## Expressions Used in Derived Column Transformations

### Data Flow Task 1: Salary Calculation
- **Expression:** `EmployeeNumber * 1000`

### Data Flow Task 2: Full Name Calculation
- **Expression:** `(DT_STR,50,1252)(EmployeeFirstName + " " + (ISNULL(EmployeeMiddleName) ? "" : EmployeeMiddleName + " ") + EmployeeLastName)`

### Data Flow Task 2: Age Calculation
- **Expression:** `(DT_I4)((DATEDIFF("DD",(DT_DBTIMESTAMP)DateOfBirth,GETDATE()) / 365.25))`

### Data Flow Task 2: Salary Grade Calculation
- **Expression:** `(DT_STR,2,1252)(Salary > 1000000 ? "A" : Salary > 500000 ? "B" : Salary > 250000 ? "C" : "D")`

## Detailed Documentation

For detailed documentation, including images and more in-depth explanations, please visit our [Wiki Documentation](https://github.com/Engr-M-A-Malik/SSIS/wiki).

## Conclusion

This README provides a high-level overview of the SSIS package, including the data flows, transformations, and error handling mechanisms. For a comprehensive guide, please refer to the [Wiki Documentation](https://github.com/Engr-M-A-Malik/SSIS/wiki).
