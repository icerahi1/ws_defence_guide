# Assessment Task 2: Task Defence Guide

This guide is designed to help you, as a beginner, understand everything about your JAX-WS and XSL Transformation Spring Boot project. Read through this carefully so you can comfortably answer any questions your teacher might ask.

---

## ⭐ Quick Cheat Sheet (For the Defence)
Use these simple answers if you are asked these common questions during your defence:

**Q: What does this project do?**
> *"This project is a SOAP Web Service built with Java and Spring Boot. It provides information about a Student and the subjects they are taking. It also has utility functions that take the XML data from the web service and transform it into a readable HTML webpage and a PDF document."*

**Q: How does the workflow operate?**
> *"First, Spring Boot starts and publishes our Web Service at port 8081. Second, a client sends an XML request asking for a student. Our service creates the student data and JAXB converts it into XML to send back. Finally, we can use our XSL templates to transform that XML into HTML or PDF."*

**Q: Explain the S.O.L.I.D. principles you used.**
> *"I used the Single Responsibility Principle. For example, my `TransformUtils` class only does one thing: it transforms files. My `Student` class only holds data. Everything is separated cleanly."*

---

## 1. Project Goal & Workflow
**Goal:** The goal of this task is to create a SOAP Web Service (JAX-WS) that provides data (a Student with their Subjects) and to transform that XML data into user-friendly HTML and PDF formats using XSL/XSL-FO technologies.

**Workflow (How it all connects):**
1. **Spring Boot Starts:** The application starts up like a normal Java program.
2. **Endpoint Publishing:** During startup, `WebServiceConfig` uses `Endpoint.publish()` to expose your `StudentWebService` at a specific URL (`http://localhost:8081/ws/student`).
3. **SOAP Request:** A client (like SoapUI or your generated code) sends an XML request to that URL asking for a student's data.
4. **Data Generation & Response:** `StudentWebServiceImpl` receives the request, generates mock `Student` and `Subject` objects, and returns them. Spring Boot automatically converts these Java objects into XML using **JAXB** and sends the XML response back.
5. **Transformation (XSL):** Behind the scenes (or via tests), `TransformUtils` can take that XML data and use `student-html.xsl` to generate an HTML page or `student-pdf.xsl` to generate a PDF using a library called Apache FOP.

---

## 2. Important Annotations You MUST Know
Your teacher will definitely ask you about the annotations. Here is exactly what they mean:

### JAX-WS (Web Services) Annotations:
* `@WebService`: This tells Java that the interface (`StudentWebService`) or class (`StudentWebServiceImpl`) is a SOAP Web Service.
* `@WebMethod`: This is placed on a method (like `getStudent`) to indicate that this specific method can be called over the internet as part of the web service.
* `@WebParam`: This maps the method parameter (like `String name`) to an XML element in the SOAP request.

### JAXB (XML Binding) Annotations:
These are found in your `Student` and `Subject` classes. JAXB stands for Java Architecture for XML Binding. It translates Java objects into XML.
* `@XmlRootElement(name = "student")`: Tells JAXB that this class is the top-level (root) tag of the XML document.
* `@XmlAccessorType(XmlAccessType.FIELD)`: Tells JAXB to automatically convert all the private fields (variables) in the class into XML tags.
* `@XmlElement(name = "active")`: Customizes the XML tag name for a specific field. For example, the `isActive` boolean in Java becomes `<active>` in XML.
* `@XmlElementWrapper(name = "subjects")`: Creates an enclosing wrapper tag (e.g., `<subjects>`) around a list of items.

### Spring Boot Annotations:
* `@SpringBootApplication`: This is the starting point of the project. It tells Spring Boot to automatically configure the application.
* `@Configuration`: Tells Spring that this class contains configuration setup (like our `WebServiceConfig`).
* `@Service`: Tells Spring Boot to manage the `StudentWebServiceImpl` class and make it available to the rest of the application.
* `@PostConstruct`: Tells Spring to run the `publishEndpoint()` method *immediately* after the application has started. This is how the web service is launched.

---

## 3. S.O.L.I.D. Principles Applied
Your requirements mentioned conforming to S.O.L.I.D. principles. Here is what you should say:
* **Single Responsibility Principle (SRP):** Every class has one job. 
  * `Student` and `Subject` only hold data.
  * `TransformUtils` only handles XML/PDF transformations.
  * `StudentWebServiceImpl` only handles Web Service logic.
* **Interface Segregation Principle (ISP):** We created a specific interface `StudentWebService` containing only the `getStudent` method, rather than a massive interface with unrelated methods.

---

## 4. Running the Project
1. **Build the Project:** Run `mvn clean install` in your terminal. This will compile the code and run the unit tests.
2. **Start the Service:** Run the `Task2Application` class from your IDE (or use `mvn spring-boot:run`).
3. **Verify the WSDL:** Open a web browser and go to: `http://localhost:8081/ws/student?wsdl`. If you see an XML file describing your service, it is running perfectly!

---

## 5. Testing with SoapUI / Postman
1. Open **SoapUI**.
2. Click **File -> New SOAP Project**.
3. In the "Initial WSDL" field, paste: `http://localhost:8081/ws/student?wsdl` and click OK.
4. SoapUI will generate a `getStudent` request. Expand it, double-click "Request 1".
5. Replace the `?` inside `<name>?</name>` with "John" and click the green play button.
6. You will see the XML response containing the Student and their Subjects on the right side!

---

## 6. Generating the Client (wsimport)
To generate the client stubs:
1. Ensure the Spring Boot application is **running**.
2. Double-click the `generate-client.bat` file in your project folder.
3. This runs the `wsimport` tool which downloads the WSDL from your running service and generates Java classes in `src/main/java`.

### How to test the Web Service using the generated client:
Once you have run `generate-client.bat`, you can create a test class like this anywhere in your test folder:

```java
import lt.viko.eif.ihasan.Task2.client.generated.StudentWebService;
import lt.viko.eif.ihasan.Task2.client.generated.StudentWebServiceImplService;
import lt.viko.eif.ihasan.Task2.client.generated.Student;

public class ClientTest {
    public static void main(String[] args) {
        // 1. Connect to the Service
        StudentWebServiceImplService service = new StudentWebServiceImplService();
        // 2. Get the interface
        StudentWebService port = service.getStudentWebServiceImplPort();
        // 3. Call the method!
        Student student = port.getStudent("MyName");
        System.out.println("Got student from Web Service: " + student.getName());
    }
}
```

*Good luck with your defence! You've got this!*
