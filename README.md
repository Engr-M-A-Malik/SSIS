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

![0](https://github.com/Engr-M-A-Malik/SSIS/assets/174581545/6f062e14-61c0-44ee-b44c-458481a2d5ac)

1. **Source:**

   - Table: `tblemployee` from the database `70-461`
   
   ![Employee Data Retrieval]

![1](https://github.com/Engr-M-A-Malik/SSIS/assets/174581545/e59c0fc3-cd3f-40eb-bca4-d0220c4ece2a)

2. **Derived Column Transformation: Salary Calculation**
   - Expression: `EmployeeNumber * 1000`
   
   ![Salary Calculation]

![2](https://github.com/Engr-M-A-Malik/SSIS/assets/174581545/7420df7d-f7d4-476e-8b37-d1f910b6a1a7)

3. **Destination:**
   - Table: `employee_info` in the database `source_database`
   
   ![Loading into Source]

![3](https://github.com/Engr-M-A-Malik/SSIS/assets/174581545/88653cfc-f2c0-4314-84b9-b634d4c412d8)

### Data Flow Task 2: Transformations and Loading

![4](https://github.com/Engr-M-A-Malik/SSIS/assets/174581545/c0c88307-c16e-4c96-9728-5d124727f2f4)

1. **Source:**
   - Table: `employee_info` from the database `source_database`
   
   ![Extracting Data From Source]

![5](https://github.com/Engr-M-A-Malik/SSIS/assets/174581545/5d67cb45-f118-46e0-8278-72374d243429)

2. **Derived Column Transformation: Full Name Calculation**
   - Expression: `(DT_STR,50,1252)(EmployeeFirstName + " " + (ISNULL(EmployeeMiddleName) ? "" : EmployeeMiddleName + " ") + EmployeeLastName)`
   
   ![Full_Name Calculation]

![6](https://github.com/Engr-M-A-Malik/SSIS/assets/174581545/3e4b7483-b4e0-4958-99d4-767fa224a747)

3. **Derived Column Transformation: Age Calculation**
   - Expression: `(DT_I4)((DATEDIFF("DD",(DT_DBTIMESTAMP)DateOfBirth,GETDATE()) / 365.25))`
   
   ![Age Calculation]

![7](https://github.com/Engr-M-A-Malik/SSIS/assets/174581545/6afcee52-c576-497c-82ee-f7be046372b7)

4. **Derived Column Transformation: Salary Grade Calculation**
   - Expression: `(DT_STR,2,1252)(Salary > 1000000 ? "A" : Salary > 500000 ? "B" : Salary > 250000 ? "C" : "D")`
   
   ![Salary_Grade Calculation]

![8](https://github.com/Engr-M-A-Malik/SSIS/assets/174581545/67a89013-d04c-4f66-8e2a-969fc8eeea63)

5. **Destination:**
   - Table: `Transformed_Employee_Info` in the database `destination_database`
   
   ![Destination Loading]

![9](https://github.com/Engr-M-A-Malik/SSIS/assets/174581545/f47ae62d-90e0-48a8-89d5-dcee346d12b2)

## Control Flow

The control flow integrates both data flow tasks and ensures they are executed sequentially.

![Control Flow]

![10](https://github.com/Engr-M-A-Malik/SSIS/assets/174581545/26b7667a-bf95-4a19-8428-bca0bc0332a2)

## Event Handler

### OnError Event Handler

This event handler captures errors during package execution and logs them into the `LogError` table in the `destination_database`.

1. **Execute SQL Task: Log Error**
   - SQL Command: `insert into LogError(Package_Name,Error_Messages) values(?,?)`
   - Parameters:
     - `Package_Name`: System variable `System::PackageName`
     - `Error_Messages`: System variable `System::ErrorDescription`
   
   ![Error Handling]

![11](https://github.com/Engr-M-A-Malik/SSIS/assets/174581545/b5b98757-1cf5-4a43-826a-c9e23c456431)
![12](https://github.com/Engr-M-A-Malik/SSIS/assets/174581545/4eea9db8-a2b5-414f-a5a5-b1bea7f2fbf1)
![13](https://github.com/Engr-M-A-Malik/SSIS/assets/174581545/6669469b-4389-4806-909f-5aaf6c3645dd)

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

