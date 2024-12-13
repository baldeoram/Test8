# SpringBootProjectSetupTool

## Use Cases

### 1. Use as an API Mock

This project can generate a mock REST API implementation defined by an OpenAPI 3.0 specification in the `.yaml` file. The generated implementation can then be deployed to simulate API behaviour without connecting to a database or backend. The response is generated from the examples in the `.yaml` file.

#### How to Use:

- **Define your API**: Create your API specification in a `.yaml` file following the OpenAPI 3.0 standard, ensuring that example values are provided for data objects.

- **Locate the script**: You can find the setup tool script on the dev-resource under the `script/` directory.

- **Specify the YAML file**: Pass the path to your custom `.yaml` file as the first argument when running the script. Example usage: `./SpringBootProjectSetupTool.sh /path/to/your/api.yaml projectName`.

- **Generate multiple projects**: You can create multiple Spring Boot projects by providing custom project names. For example:  
  `./SpringBootProjectSetupTool.sh /path/to/your/api.yaml ProjectA ProjectB`.

- **Run the API**: The script will generate a Spring Boot REST API project, compile it, and get your mock API running.

- **Deploy**: You can also package the project and deploy it to your server for execution.

### Example
The package comes with an example API specification that mimics an `Employee API`. Using this specification, if you generate the application and execute it, you will find the following endpoints working

#### Get All Employees

- **Endpoint**: `http://localhost:8081/api/v1/employees`
- **Method**: `GET`
- **Response**:
  ```json
  [
    {
      "id": 1,
      "name": "John Doe",
      "age": 30,
      "position": "Software Engineer"
    }
  ]

### 2. Extend This Mock to Create a Real API

You can extend the mock implementation to build a fully functional API. During this implementation you can avoid writing the boiler-platter code for endpoints or creating models. They are all will be there for you. You can simply focus on the actual logic that you need to build (e.g., database connections, service layer, etc.).


#### Adding layers:

- Create custom controllers and implement the necessary service and repository layers to handle real data.
- Integrate the application with a database (e.g., PostgreSQL, MySQL) to persist employee records.

For example, by adding the following code in the /org/delta/handler/controller/AppController.java, you can modify the output of the response of the `GET /v1/employee`.

```java

@Override
public ResponseEntity<List<Employee>> employeesGet() {
    log.info("Fetching list of all employees");
    Employee employee1 = new Employee();
    employee1.setId(1L);
    employee1.setName("John Doe");
    employee1.setAge(30);
    employee1.setPosition("Software Engineer");

    Employee employee2 = new Employee();
    employee2.setId(2L);
    employee2.setName("Jane Smith");
    employee2.setAge(28);
    employee2.setPosition("Product Manager");
    List<Employee> employees = List.of(employee1, employee2);
    return new ResponseEntity<>(employees, HttpStatus.OK);
}

```

When you compile and execute the above changes and then reach out to the Endpoint, you will see the following:

- **Endpoint**: `http://localhost:8081/api/v1/employees`
- **Method**: `GET`
- **Response**:
  ```json
  [
    {
        "id": 1,
        "name": "John Doe",
        "age": 30,
        "position": "Software Engineer"
    },
    {
        "id": 2,
        "name": "Jane Smith",
        "age": 28,
        "position": "Product Manager"
    }
  ]

### 3. Adding Authentication and Security Integration Using Mustache and OpenAPI Specification

This tool now supports generating Spring Boot APIs with security and authentication capabilities based on the OpenAPI specification. By adding the `components.securitySchemes` section in your `.yaml` file, you can define security schemes like OAuth2. The generated API will include authentication mechanisms, such as OAuth2, by adding the relevant dependencies and applying authorization checks on endpoints.

#### How to Use:

- **Define Security Schemes in OpenAPI**: Add the security configuration to the OpenAPI `.yaml` file.
- **Set Endpoint-Level Security Requirements**: For each endpoint, define the required security level by specifying the security section with the desired scope.

- **Run the Setup Tool**: Run the `SpringBootProjectSetupTool.sh` script, specifying the OpenAPI file and project name. The generated code will include dependencies for security based on the provided security schemes, such as OAuth2.

#### Example Security Configuration:

```yaml
components:
  securitySchemes:
    oauth2:
      type: oauth2
      flows:
        authorizationCode:
          authorizationUrl: https://your-auth-server.com/oauth/authorize
          tokenUrl: https://your-auth-server.com/oauth/token
          scopes:
            user: Grants user access
            admin: Grants access to admin operations
            superadmin: Grants access to superadmin operations

paths:
  /users:
    post:
      tags:
        - UserController
      summary: Create a new user
      operationId: createUser
      security:
        - oauth2:
            - superadmin
```

#### Endpoints Security:

Each endpoint will include `@PreAuthorize` annotations, with access levels set according to the security section in the OpenAPI `.yaml` file.

#### Example of Generated Code with Security Annotations:

In the generated API code, each endpoint that requires authorization will have `@PreAuthorize` annotations. For example:

```java
@RestController
@Slf4j
@RequestMapping("/api/v2")
public class CustomEmployeesApiController implements DefaultApi {

    @PreAuthorize("hasAuthority('read')")
    @Override
    public ResponseEntity<List<Employee>> employeesGet() {
        log.info("Fetching list of all employees");
        // ... code to fetch employees
    }

    @PreAuthorize("hasAuthority('admin')")
    @Override
    public ResponseEntity<Void> employeesEmployeeIdDelete(@PathVariable("employeeId") Long employeeId) {
        log.info("Deleting employee with ID: {}", employeeId);
        return new ResponseEntity<>(HttpStatus.NO_CONTENT);
    }
    // Additional endpoints with security applied
}
```

You can find the full implementation on the main branch in this repository - https://github.com/DC-Suburb/tools.springbootrestapi.codegenerator


### Adding new version or Endpoints:

To add a new Endpoint or version, do the following
- Implement custom logic for the new or modified endpoint.
- For example, to create a new version `v2` with modified employee data: Add the following class at - /org/delta/handler/controller/CustomEmployeesApiController.java

## CustomEmployeesApiController.java

```java
package org.delta.handler.controller;

import jakarta.validation.Valid;
import lombok.extern.slf4j.Slf4j;
import org.delta.handler.DefaultApi;
import org.delta.model.Employee;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@Slf4j
@RequestMapping("/api/v2")
public class CustomEmployeesApiController implements DefaultApi {

    @Override
    public ResponseEntity<Void> employeesEmployeeIdDelete(@PathVariable("employeeId") Long employeeId) {
        log.info("Deleting employee with ID: {}", employeeId);
        return new ResponseEntity<>(HttpStatus.NO_CONTENT);
    }

    @Override
    public ResponseEntity<Employee> employeesEmployeeIdGet(@PathVariable("employeeId") Long employeeId) {
        log.info("Fetching details for employee with ID: {}", employeeId);
        Employee employee = new Employee();
        employee.setId(employeeId);
        employee.setName("John Doe");
        employee.setAge(30);
        employee.setPosition("Software Engineer - From v2");
        return new ResponseEntity<>(employee, HttpStatus.OK);
    }

    @Override
    public ResponseEntity<Employee> employeesEmployeeIdPut(@PathVariable("employeeId") Long employeeId, @Valid @RequestBody Employee employee) {
        log.info("Updating employee with ID: {}", employeeId);
        Employee updatedEmployee = new Employee();
        updatedEmployee.setId(employeeId);
        updatedEmployee.setName(employee.getName());
        updatedEmployee.setAge(employee.getAge());
        updatedEmployee.setPosition(employee.getPosition());
        return new ResponseEntity<>(updatedEmployee, HttpStatus.OK);
    }

    @Override
    public ResponseEntity<List<Employee>> employeesGet() {
        log.info("Fetching list of all employees");
        Employee employee1 = new Employee();
        employee1.setId(1L);
        employee1.setName("John Doe");
        employee1.setAge(30);
        employee1.setPosition("Software Engineer - From v2");

        Employee employee2 = new Employee();
        employee2.setId(2L);
        employee2.setName("Jane Smith");
        employee2.setAge(28);
        employee2.setPosition("Product Manager - From v2");
        List<Employee> employees = List.of(employee1, employee2);
        return new ResponseEntity<>(employees, HttpStatus.OK);
    }

    @Override
    public ResponseEntity<Employee> employeesPost(@Valid @RequestBody Employee employee) {
        log.info("Creating a new employee with name: {}", employee.getName());
        Employee createdEmployee = new Employee();
        createdEmployee.setId(3L);
        createdEmployee.setName(employee.getName());
        createdEmployee.setAge(employee.getAge());
        createdEmployee.setPosition(employee.getPosition());
        return new ResponseEntity<>(createdEmployee, HttpStatus.CREATED);
    }
}

```


Once you compile and run the above changes and execute the code, you can get the following output.

#### Get All Employees

- **Endpoint**: `http://localhost:8081/api/v2/employees`
- **Method**: `GET`
- **Response**:
  ```json
  [
    {
        "id": 1,
        "name": "John Doe",
        "age": 30,
        "position": "Software Engineer - From v2"
    },
    {
        "id": 2,
        "name": "Jane Smith",
        "age": 28,
        "position": "Product Manager - From v2"
    }
  ]
