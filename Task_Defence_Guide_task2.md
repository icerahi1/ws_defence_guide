# Assessment Task 2: Task Defence Guide

This guide is designed to help you, as a beginner, understand everything about your JAX-WS and XSL Transformation Spring Boot project. Read through this carefully so you can comfortably answer any questions your teacher might ask.

---

## ⭐ Quick Cheat Sheet (For the Defence)
Use these simple answers if you are asked these common questions during your defence:

**Q: What does this project do?**
> *"This project is a SOAP Web Service built with Java and Spring Boot. It provides information about a Student and the subjects they are taking. It also has utility functions that take the XML data from the web service and transform it into a readable HTML webpage and a PDF document."*

**Q: How does the workflow operate?**
> *"First, Spring Boot starts and publishes our Web Service at port 8081. Second, a client sends an XML request asking for a student's data. Our service retrieves the student and JAXB converts it into XML to send back. We also have two other endpoints that transform this XML directly into an HTML string or a PDF document, and return that in the SOAP response."*

**Q: Explain the S.O.L.I.D. principles you used.**
> *"I used the Single Responsibility Principle. For example, my `TransformUtils` class only does one thing: it transforms files. My `Student` class only holds data. Everything is separated cleanly."*

---

## 1. Project Goal & Workflow
**Goal:** The goal of this task is to create a SOAP Web Service (JAX-WS) that provides data (a Student with their Subjects) and to seamlessly transform that XML data into user-friendly HTML and PDF formats directly from the web service endpoints using XSL/XSL-FO technologies.

**Workflow (How it all connects):**
1. **Spring Boot Starts:** The application starts up like a normal Java program.
2. **Endpoint Publishing:** During startup, `WebServiceConfig` uses `Endpoint.publish()` to expose your `StudentWebService` at a specific URL (`http://localhost:8081/ws/student`).
3. **SOAP Request:** A client (like SoapUI) sends an XML request to that URL. It can call `getStudent`, `getStudentHtml`, or `getStudentPdf`.
4. **Data Generation:** `StudentWebServiceImpl` receives the request and generates mock `Student` and `Subject` objects.
5. **Transformation & Response:** If `getStudent` is called, Spring Boot automatically converts the Java objects into XML using **JAXB** and sends the XML response back. If `getStudentHtml` or `getStudentPdf` is called, our `TransformUtils` intercepts the XML, transforms it using `student-html.xsl` or `student-pdf.xsl` via Apache FOP, and sends back the HTML string or PDF byte array directly inside the SOAP response.

---

## 2. Important Annotations You MUST Know
Your teacher will definitely ask you about the annotations. Here is exactly what they mean:

### JAX-WS (Web Services) Annotations:
* `@WebService`: This tells Java that the interface (`StudentWebService`) or class (`StudentWebServiceImpl`) is a SOAP Web Service.
* `@WebMethod`: This is placed on a method (like `getStudent`, `getStudentHtml`, `getStudentPdf`) to indicate that this specific method can be called over the internet as part of the web service.
* `@WebParam`: This maps the method parameter (like `String name`) to an XML element in the SOAP request.

### JAXB (XML Binding) Annotations:
These are found in your `Student` and `Subject` classes. JAXB stands for Java Architecture for XML Binding. It translates Java objects into XML.
* `@XmlRootElement(name = "student")`: Tells JAXB that this class is the top-level (root) tag of the XML document.
* `@XmlAccessorType(XmlAccessType.FIELD)`: Tells JAXB to automatically convert all the private fields (variables) in the class into XML tags without needing explicit getters/setters annotations.
* `@XmlType(name = "...", propOrder = {...})`: Specifies the XML schema type name for the class and defines the exact order in which the fields should appear in the generated XML.
* `@XmlElement(name = "active", required = true)`: Customizes the XML tag name for a specific field. For example, the `isActive` boolean in Java becomes `<active>` in XML. The `required = true` means the element must be present in the XML.
* `@XmlElementWrapper(name = "subjects")`: Creates an enclosing wrapper tag (e.g., `<subjects>`) around a list of items.

### Spring Boot / Spring Annotations:
* `@SpringBootApplication`: This is the starting point of the project. It tells Spring Boot to automatically configure the application.
* `@Configuration`: Tells Spring that this class contains configuration setup (like our `WebServiceConfig`).
* `@Service`: Tells Spring Boot to manage the `StudentWebServiceImpl` class and make it available to the rest of the application as a business service.
* `@Component`: A generic Spring annotation that tells Spring to manage a class (like `TransformUtils`) so it can be injected elsewhere.
* `@Autowired`: Tells Spring to automatically inject (provide) an instance of a required component into another class. For example, we `@Autowired` `TransformUtils` into `StudentWebServiceImpl`.
* `@PostConstruct`: Tells Spring to run the `publishEndpoint()` method *immediately* after the application has started. This is how the web service is launched on our custom port.

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
4. SoapUI will generate three requests under the service: `getStudent`, `getStudentHtml`, and `getStudentPdf`.
5. Expand any of them and double-click "Request 1".
6. Replace the `?` inside `<name>?</name>` with a student's name (e.g. "John") and click the green play button.
7. **For `getStudent`**: You will see the standard XML response on the right side.
8. **For `getStudentHtml`**: You will see the transformed HTML markup response.
9. **For `getStudentPdf`**: You will see Base64 encoded data, which SoapUI can automatically interpret as a PDF.

---

## 6. Testing the Web Service Client

There are two ways to test your SOAP web service from a client perspective:

### Method A: Dynamic Client Integration Test (Automated & Recommended)
We have created a clean integration test at [StudentWebServiceClientTest](file:///c:/Users/s059812/Desktop/Task2/src/test/java/lt/viko/eif/ihasan/Task2/client/StudentWebServiceClientTest.java). This test runs automatically whenever you build the project or run tests (`mvn test`).

It uses **dynamic JAX-WS service creation**, which means it does not need any pre-generated classes to function. This is perfect for modern Java environments where `wsimport` is not bundled by default.

**How the code works:**
```java
// 1. Point to the running service WSDL
URL wsdlURL = new URL("http://localhost:8081/ws/student?wsdl");

// 2. Identify the target namespace and service name from the WSDL
QName serviceName = new QName("http://service.Task2.ihasan.eif.viko.lt/", "StudentWebServiceImplService");

// 3. Create the JAX-WS service manager
Service service = Service.create(wsdlURL, serviceName);

// 4. Retrieve the client port interface dynamically
StudentWebService client = service.getPort(StudentWebService.class);

// 5. Invoke the operations
Student student = client.getStudent("TestStudent");
```

### Method B: Generating Static Client Stubs (wsimport)
If you want to generate static stubs using the traditional command-line method:
1. Ensure the Spring Boot application is **running**.
2. Run the `generate-client.bat` file. *(Note: `wsimport` was removed from JDK 11+, so this requires the utility to be configured in your PATH).*
3. The script generates source files inside `src/main/java/lt/viko/eif/ihasan/Task2/client/generated`.
4. You can then run a standalone client application like this:
```java
import lt.viko.eif.ihasan.Task2.client.generated.StudentWebService;
import lt.viko.eif.ihasan.Task2.client.generated.StudentWebServiceImplService;
import lt.viko.eif.ihasan.Task2.client.generated.Student;

public class ClientRunner {
    public static void main(String[] args) {
        // Connect to the service endpoint
        StudentWebServiceImplService service = new StudentWebServiceImplService();
        StudentWebService port = service.getStudentWebServiceImplPort();
        
        // Execute the service operation
        Student student = port.getStudent("John Doe");
        System.out.println("Got student from Web Service: " + student.getName());
    }
}
```

*Good luck with your defence! You've got this!*
