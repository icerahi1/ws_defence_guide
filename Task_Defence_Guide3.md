# Task 3: JAX-RS Web Service - Final Defense Guide

Welcome to your study guide! This document is created specifically for you. Since you are new to Java, this guide breaks down everything step-by-step so you can confidently answer any question your teacher asks.

---

## 1. Project Goal & Overview
**The Goal:** Create a RESTful Web Service using **JAX-RS** (Java API for RESTful Web Services) via Spring Boot. 
A Web Service allows different applications to talk to each other over the internet using standard HTTP methods (like GET and POST).

**The Tech Stack:**
- **Java 17**: The programming language.
- **Spring Boot**: A framework that makes it easy to create stand-alone, production-ready Java applications.
- **Jersey**: The specific library we used to implement the JAX-RS standard.
- **Maven**: Our build tool (it downloads the libraries defined in `pom.xml`).

---

## 2. The Code Workflow (How it all connects)

When a user (or Postman) sends a request to `http://localhost:8080/students`:

1. **The Request hits `Application.java`**: Spring Boot receives the request.
2. **`JerseyConfig.java` routes it**: This configuration tells Spring Boot where our JAX-RS endpoints are. It points to `StudentResource.java`.
3. **`StudentResource.java` (The Endpoint)**: This class catches the specific URL (`/students`) and HTTP method (`GET`, `POST`). It doesn't do the heavy lifting; it calls the Service layer.
4. **`StudentService.java` (The Logic)**: The Resource talks to the Service Interface (`StudentService`). The actual work is done in the implementation (`StudentServiceImpl.java`). This layer retrieves the data from our in-memory list.
5. **The Data (`Student.java` & `Subject.java`)**: The Service returns `Student` objects, which are automatically converted to JSON format and sent back as the HTTP Response.

---

## 3. S.O.L.I.D. Principles Explained
Your teacher will ask about this. We used S.O.L.I.D principles, specifically:

- **Single Responsibility Principle (SRP)**: Every class has one job. `StudentResource` only handles HTTP web traffic. `StudentServiceImpl` only handles business logic and data storage. `Student` only holds data.
- **Dependency Inversion Principle (DIP)**: High-level modules should not depend on low-level modules; both should depend on abstractions. Notice how `StudentResource` uses the `StudentService` **Interface**, not the `StudentServiceImpl` class. This means we can change the database implementation later without touching the web code!

---

## 4. Master Your Annotations (CRITICAL)

Your teacher will definitely ask you about the `@` symbols (annotations) in your code. Here is your cheat sheet:

### Spring Boot Annotations
- **`@SpringBootApplication`** *(in Application.java)*: This tells Java "This is a Spring Boot app, configure everything automatically and start the server here."
- **`@Component`** *(in JerseyConfig.java & StudentResource.java)*: Tells Spring to create an object of this class automatically so we can use it anywhere (Dependency Injection).
- **`@Service`** *(in StudentServiceImpl.java)*: Same as `@Component`, but specifically tells developers "This class contains business logic."
- **`@Autowired`** *(in StudentResource.java)*: This is magic. It tells Spring to automatically plug in (inject) the `StudentService` into our `StudentResource` without us having to write `new StudentServiceImpl()`.

### JAX-RS (Jersey) Annotations
- **`@Path("/students")`**: Maps the web address. It means this class or method is triggered when someone visits `/students`.
- **`@GET`**: Tells the server this method should run when a client requests data (HTTP GET).
- **`@POST`**: Tells the server this method should run when a client sends new data to create (HTTP POST).
- **`@DELETE`**: Tells the server to run this when a client wants to delete data.
- **`@Produces(MediaType.APPLICATION_JSON)`**: Tells the browser/Postman "The data I am sending back to you will be in JSON format."
- **`@Consumes(MediaType.APPLICATION_JSON)`**: Tells the server "The data the user is sending me (like in a POST request) is formatted as JSON."
- **`@PathParam("id")`**: Extracts a variable from the URL. For example, in `/students/1`, it grabs the `1` and passes it to your method as `int id`.

---

## 5. How to Run the Project

1. Open your IDE (like IntelliJ IDEA or Eclipse).
2. Click **File -> Open** and select the `Task3` folder.
3. Your IDE will recognize it as a Maven project and download dependencies.
4. Open `Application.java` and click the green "Run" button.
5. The console will say Tomcat started on port 8080.

---

## 6. How to Test (SoapUI, Postman, Swagger)

Once the app is running, use **Postman** to test it:

### Test 1: Get All Students (GET)
- **Method**: `GET`
- **URL**: `http://localhost:8080/students`
- **Click Send**. You will see a JSON array of students in the response!

### Test 2: Get One Student (GET)
- **Method**: `GET`
- **URL**: `http://localhost:8080/students/1`
- **Click Send**. You will see just Alice's data.

### Test 3: Create a Student (POST)
- **Method**: `POST`
- **URL**: `http://localhost:8080/students`
- Go to the **Body** tab in Postman, select **raw**, and choose **JSON**. Paste this:
  ```json
  {
    "id": 3,
    "name": "John Doe",
    "gpa": 3.9,
    "active": true,
    "grade": "A",
    "subjects": [
      {
        "name": "Computer Science",
        "credits": 4
      }
    ]
  }
  ```
- **Click Send**. The server will return status `201 Created`.

### OpenAPI & Swagger (WADL)
Jersey automatically generates a WADL (Web Application Description Language) file.
- You can see it by going to `http://localhost:8080/application.wadl` in your browser.
- Following your assignment steps, you can save this WADL file, go to **APIMATIC** (https://www.apimatic.io/transformer), and convert it to OpenAPI JSON.
- Then, paste that JSON into **Swagger Editor** (https://editor.swagger.io/) to see a beautiful visual map of your API where you can click buttons to test the endpoints!

Good luck with your defense! You have a perfectly structured, documented, and professional application.
