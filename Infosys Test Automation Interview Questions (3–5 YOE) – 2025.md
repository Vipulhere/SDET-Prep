# Round 1 – Technical Interview Answers

---

## 1. Explain your project and your roles & responsibilities

### Context
This question helps interviewers understand your practical experience, the scale of projects you've worked on, and your understanding of SDET responsibilities beyond just writing test cases.

### Detailed Answer

**Project Overview:**
I worked on an **E-commerce Platform** (or **Banking Application** / **Healthcare System** - choose based on your experience) that handles multiple user flows including product browsing, cart management, checkout, payment processing, and order tracking. The platform serves both web and mobile applications with RESTful APIs as the backend.

**Key Responsibilities:**

1. **Test Framework Development & Maintenance**
   - Designed and implemented a hybrid test automation framework using Selenium WebDriver, TestNG, and Maven
   - Created reusable components, page object models, and utility classes
   - Maintained framework architecture to support parallel execution and CI/CD integration

2. **Test Strategy & Planning**
   - Collaborated with product managers, developers, and business analysts to understand requirements
   - Created test plans, test cases, and test data strategies
   - Prioritized test cases based on risk analysis and business impact

3. **API Testing**
   - Performed comprehensive API testing using Postman, RestAssured, and custom scripts
   - Validated request/response payloads, status codes, error handling, and edge cases
   - Created automated API test suites integrated with CI/CD pipeline

4. **Test Execution & Reporting**
   - Executed manual and automated test suites across multiple environments
   - Generated detailed test reports and dashboards using ExtentReports, Allure, or custom reporting
   - Analyzed test results, identified root causes of failures, and tracked defects

5. **CI/CD Integration**
   - Integrated test automation with Jenkins/GitLab CI for continuous testing
   - Configured test execution triggers (on commit, nightly builds, pre-deployment)
   - Managed test environments and test data

6. **Code Quality & Best Practices**
   - Performed code reviews for test automation code
   - Ensured adherence to coding standards, design patterns, and best practices
   - Mentored junior QA engineers on automation and testing concepts

7. **Performance & Security Testing**
   - Conducted basic performance testing using JMeter or similar tools
   - Performed security testing for authentication, authorization, and data validation
   - Identified bottlenecks and security vulnerabilities

### Scenario-Based Example

**Real Scenario:**
During a critical release, we discovered a payment gateway integration issue in production. As the SDET lead, I:
- Immediately created a test scenario to reproduce the issue
- Wrote automated tests to validate all payment gateway endpoints
- Collaborated with the backend team to identify the root cause (missing error handling for timeout scenarios)
- Enhanced the test framework to include payment gateway mocking for faster feedback
- Implemented monitoring alerts for payment-related API failures

**Impact:** Reduced payment-related defects by 60% and improved test coverage for critical payment flows.

---

## 2. Have you built a framework from the beginning? Explain how you designed it

### Context
This question assesses your ability to architect a test automation framework, make design decisions, and understand the principles of scalable test automation.

### Detailed Answer

**Yes, I designed and built a Hybrid Test Automation Framework from scratch.** Here's my approach:

### Framework Design Process

#### **Phase 1: Requirements Analysis**
- **Stakeholder Input:** Gathered requirements from QA team, developers, and DevOps
- **Technology Stack:** Java, Selenium WebDriver, TestNG, Maven, Jenkins
- **Scope:** Web application testing, API testing, cross-browser testing, parallel execution

#### **Phase 2: Architecture Design**

**Framework Structure:**
```
TestFramework/
├── src/main/java/
│   ├── base/
│   │   ├── BaseTest.java
│   │   └── DriverManager.java
│   ├── pages/
│   │   ├── LoginPage.java
│   │   └── HomePage.java
│   ├── utils/
│   │   ├── ConfigReader.java
│   │   ├── ExcelReader.java
│   │   ├── ScreenshotUtil.java
│   │   └── WaitUtil.java
│   ├── listeners/
│   │   └── TestListener.java
│   └── constants/
│       └── Constants.java
├── src/test/java/
│   └── tests/
│       ├── LoginTest.java
│       └── CheckoutTest.java
├── src/test/resources/
│   ├── config.properties
│   ├── testdata/
│   │   └── TestData.xlsx
│   └── testng.xml
└── pom.xml
```

#### **Phase 3: Core Components Implementation**

**1. Base Test Class**
```java
package base;

import org.openqa.selenium.WebDriver;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Parameters;
import utils.ConfigReader;
import utils.DriverManager;

public class BaseTest {
    protected WebDriver driver;
    
    @BeforeMethod
    @Parameters("browser")
    public void setUp(String browser) {
        driver = DriverManager.getDriver(browser);
        driver.get(ConfigReader.getProperty("baseURL"));
        driver.manage().window().maximize();
    }
    
    @AfterMethod
    public void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

**2. Driver Manager (Singleton Pattern)**
```java
package utils;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.firefox.FirefoxDriver;
import org.openqa.selenium.edge.EdgeDriver;

public class DriverManager {
    private static ThreadLocal<WebDriver> driver = new ThreadLocal<>();
    
    public static WebDriver getDriver(String browserName) {
        if (driver.get() == null) {
            switch (browserName.toLowerCase()) {
                case "chrome":
                    driver.set(new ChromeDriver());
                    break;
                case "firefox":
                    driver.set(new FirefoxDriver());
                    break;
                case "edge":
                    driver.set(new EdgeDriver());
                    break;
                default:
                    throw new IllegalArgumentException("Invalid browser: " + browserName);
            }
        }
        return driver.get();
    }
    
    public static void closeDriver() {
        if (driver.get() != null) {
            driver.get().quit();
            driver.remove();
        }
    }
}
```

**3. Page Object Model Implementation**
```java
package pages;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import utils.WaitUtil;

public class LoginPage {
    private WebDriver driver;
    
    @FindBy(id = "username")
    private WebElement usernameField;
    
    @FindBy(id = "password")
    private WebElement passwordField;
    
    @FindBy(id = "loginButton")
    private WebElement loginButton;
    
    @FindBy(className = "error-message")
    private WebElement errorMessage;
    
    public LoginPage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }
    
    public void enterUsername(String username) {
        WaitUtil.waitForElementVisible(driver, usernameField);
        usernameField.clear();
        usernameField.sendKeys(username);
    }
    
    public void enterPassword(String password) {
        passwordField.clear();
        passwordField.sendKeys(password);
    }
    
    public void clickLogin() {
        loginButton.click();
    }
    
    public String getErrorMessage() {
        WaitUtil.waitForElementVisible(driver, errorMessage);
        return errorMessage.getText();
    }
    
    public void login(String username, String password) {
        enterUsername(username);
        enterPassword(password);
        clickLogin();
    }
}
```

**4. Test Listener for Screenshots on Failure**
```java
package listeners;

import org.testng.ITestListener;
import org.testng.ITestResult;
import utils.ScreenshotUtil;
import utils.DriverManager;

public class TestListener implements ITestListener {
    
    @Override
    public void onTestFailure(ITestResult result) {
        ScreenshotUtil.captureScreenshot(
            DriverManager.getDriver("chrome"), 
            result.getMethod().getMethodName()
        );
    }
}
```

**5. Configuration Reader**
```java
package utils;

import java.io.FileInputStream;
import java.io.IOException;
import java.util.Properties;

public class ConfigReader {
    private static Properties properties;
    
    static {
        try {
            properties = new Properties();
            FileInputStream fis = new FileInputStream(
                System.getProperty("user.dir") + "/src/test/resources/config.properties"
            );
            properties.load(fis);
            fis.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    public static String getProperty(String key) {
        return properties.getProperty(key);
    }
}
```

**6. Sample Test Class**
```java
package tests;

import base.BaseTest;
import org.testng.Assert;
import org.testng.annotations.Test;
import pages.LoginPage;
import pages.HomePage;

public class LoginTest extends BaseTest {
    
    @Test
    public void testSuccessfulLogin() {
        LoginPage loginPage = new LoginPage(driver);
        loginPage.login("validUser", "validPassword");
        
        HomePage homePage = new HomePage(driver);
        Assert.assertTrue(homePage.isUserLoggedIn(), "User should be logged in");
    }
    
    @Test
    public void testInvalidLogin() {
        LoginPage loginPage = new LoginPage(driver);
        loginPage.login("invalidUser", "invalidPassword");
        
        Assert.assertEquals(
            loginPage.getErrorMessage(), 
            "Invalid credentials",
            "Error message should match"
        );
    }
}
```

#### **Phase 4: Design Principles Applied**

1. **Page Object Model (POM):** Separated page logic from test logic
2. **Singleton Pattern:** Ensured single WebDriver instance per thread
3. **Factory Pattern:** Centralized driver creation
4. **Dependency Injection:** Used TestNG for test lifecycle management
5. **Separation of Concerns:** Utils, pages, tests in separate packages
6. **Data-Driven Testing:** Externalized test data to Excel/JSON files
7. **Configuration Management:** Externalized configuration to properties files

#### **Phase 5: Advanced Features**

- **Parallel Execution:** Configured TestNG for parallel test execution
- **Cross-Browser Testing:** Supported Chrome, Firefox, Edge
- **Reporting:** Integrated ExtentReports/Allure for detailed reports
- **CI/CD Integration:** Configured Jenkins pipeline
- **Retry Mechanism:** Implemented retry logic for flaky tests

### Scenario-Based Example

**Real Scenario:**
The framework needed to support testing across 3 environments (Dev, QA, Staging) with different configurations. I designed it to:
- Read environment-specific properties files
- Use Maven profiles to switch environments: `mvn test -Pqa`
- Implement environment-specific test data management
- Create environment validation checks before test execution

**Challenge:** Tests were failing intermittently due to timing issues.
**Solution:** Implemented custom wait utilities with explicit waits and retry mechanisms, reducing flaky tests by 80%.

---

## 3. How is Comparable different from Comparator?

### Context
This is a core Java concept question that tests your understanding of object comparison mechanisms, which is crucial when working with collections, sorting, and data structures in test automation.

### Detailed Answer

Both `Comparable` and `Comparator` are interfaces in Java used for sorting objects, but they differ in their implementation approach and use cases.

### Key Differences

| Aspect | Comparable | Comparator |
|--------|-----------|------------|
| **Package** | `java.lang.Comparable` | `java.util.Comparator` |
| **Method** | `compareTo(T o)` | `compare(T o1, T o2)` |
| **Implementation** | Inside the class being compared | Separate class or anonymous class |
| **Sorting Logic** | Natural/default ordering | Custom/multiple sorting orders |
| **Modification** | Requires modifying the class | No modification needed |
| **Usage** | `Collections.sort(list)` | `Collections.sort(list, comparator)` |
| **Single/Multiple** | Single sorting order | Multiple sorting orders possible |

### Code Examples

#### **Comparable Implementation**

```java
// Student class implementing Comparable
public class Student implements Comparable<Student> {
    private String name;
    private int age;
    private double grade;
    
    public Student(String name, int age, double grade) {
        this.name = name;
        this.age = age;
        this.grade = grade;
    }
    
    // Natural ordering: Sort by name (alphabetical)
    @Override
    public int compareTo(Student other) {
        return this.name.compareTo(other.name);
    }
    
    // Getters
    public String getName() { return name; }
    public int getAge() { return age; }
    public double getGrade() { return grade; }
    
    @Override
    public String toString() {
        return "Student{name='" + name + "', age=" + age + ", grade=" + grade + "}";
    }
}

// Usage
public class ComparableExample {
    public static void main(String[] args) {
        List<Student> students = new ArrayList<>();
        students.add(new Student("Alice", 20, 85.5));
        students.add(new Student("Bob", 19, 90.0));
        students.add(new Student("Charlie", 21, 78.5));
        
        // Natural sorting (by name)
        Collections.sort(students);
        
        System.out.println("Sorted by name:");
        students.forEach(System.out::println);
        // Output:
        // Student{name='Alice', age=20, grade=85.5}
        // Student{name='Bob', age=19, grade=90.0}
        // Student{name='Charlie', age=21, grade=78.5}
    }
}
```

#### **Comparator Implementation**

```java
// Separate Comparator classes
public class AgeComparator implements Comparator<Student> {
    @Override
    public int compare(Student s1, Student s2) {
        return Integer.compare(s1.getAge(), s2.getAge());
    }
}

public class GradeComparator implements Comparator<Student> {
    @Override
    public int compare(Student s1, Student s2) {
        return Double.compare(s2.getGrade(), s1.getGrade()); // Descending order
    }
}

// Usage
public class ComparatorExample {
    public static void main(String[] args) {
        List<Student> students = new ArrayList<>();
        students.add(new Student("Alice", 20, 85.5));
        students.add(new Student("Bob", 19, 90.0));
        students.add(new Student("Charlie", 21, 78.5));
        
        // Sort by age using Comparator
        Collections.sort(students, new AgeComparator());
        System.out.println("Sorted by age:");
        students.forEach(System.out::println);
        
        // Sort by grade (descending) using Comparator
        Collections.sort(students, new GradeComparator());
        System.out.println("\nSorted by grade (descending):");
        students.forEach(System.out::println);
        
        // Using Lambda expression (Java 8+)
        Collections.sort(students, (s1, s2) -> 
            Integer.compare(s1.getAge(), s2.getAge())
        );
        
        // Using Method Reference
        Collections.sort(students, Comparator.comparing(Student::getAge));
        
        // Multiple criteria sorting
        Collections.sort(students, 
            Comparator.comparing(Student::getGrade)
                      .reversed()
                      .thenComparing(Student::getName)
        );
    }
}
```

### When to Use Which?

**Use Comparable when:**
- You have control over the class source code
- There's a single, natural/default ordering
- The ordering is an inherent property of the object
- Example: `String`, `Integer`, `Date` classes

**Use Comparator when:**
- You cannot modify the class (third-party library)
- You need multiple sorting criteria
- You want different sorting orders for the same class
- Example: Sorting products by price, name, rating, etc.

### Scenario-Based Example

**Real Scenario in Test Automation:**

```java
// Test Result class for reporting
public class TestResult implements Comparable<TestResult> {
    private String testName;
    private String status; // PASS, FAIL, SKIP
    private long executionTime;
    private Date timestamp;
    
    // Natural ordering: Sort by timestamp (oldest first)
    @Override
    public int compareTo(TestResult other) {
        return this.timestamp.compareTo(other.timestamp);
    }
    
    // Getters...
}

// Different sorting needs in test reports
public class TestReportGenerator {
    
    public void generateReport(List<TestResult> results) {
        // Sort by execution time (slowest first) - using Comparator
        Collections.sort(results, new Comparator<TestResult>() {
            @Override
            public int compare(TestResult r1, TestResult r2) {
                return Long.compare(r2.getExecutionTime(), r1.getExecutionTime());
            }
        });
        
        // Sort failed tests first - using Comparator
        Collections.sort(results, (r1, r2) -> {
            if (r1.getStatus().equals("FAIL") && !r2.getStatus().equals("FAIL")) {
                return -1;
            } else if (!r1.getStatus().equals("FAIL") && r2.getStatus().equals("FAIL")) {
                return 1;
            }
            return r1.getTestName().compareTo(r2.getTestName());
        });
    }
}
```

**Use Case:** In a test execution report, you might want to:
- Display tests chronologically (Comparable - natural order)
- Show failed tests first (Comparator - custom order)
- Sort by execution time for performance analysis (Comparator - custom order)

---

## 4. When do you use the finally block?

### Context
This question tests your understanding of exception handling, resource management, and ensuring critical code execution regardless of exceptions. This is crucial in test automation for cleanup operations.

### Detailed Answer

The `finally` block in Java is used to execute code that **must run regardless of whether an exception occurs or not**. It's part of the try-catch-finally exception handling mechanism.

### Key Characteristics

1. **Always Executes:** The finally block executes even if:
   - No exception occurs
   - An exception is thrown and caught
   - An exception is thrown and not caught
   - A return statement is in the try or catch block

2. **Exception:** The only way finally doesn't execute is if:
   - `System.exit()` is called
   - JVM crashes
   - The thread is killed

### Common Use Cases

1. **Resource Cleanup** (Most Common)
   - Closing database connections
   - Closing file streams
   - Closing browser drivers
   - Releasing locks

2. **Cleanup Operations**
   - Deleting temporary files
   - Resetting test data
   - Closing network connections

3. **Logging/Auditing**
   - Logging execution completion
   - Recording test execution time
   - Updating status flags

### Code Examples

#### **Example 1: Resource Cleanup (WebDriver)**

```java
public class WebDriverExample {
    
    public void testWithFinally() {
        WebDriver driver = null;
        try {
            driver = new ChromeDriver();
            driver.get("https://example.com");
            // Some test operations
            WebElement element = driver.findElement(By.id("submit"));
            element.click();
            // If exception occurs here, driver won't close without finally
        } catch (NoSuchElementException e) {
            System.out.println("Element not found: " + e.getMessage());
        } catch (Exception e) {
            System.out.println("Unexpected error: " + e.getMessage());
        } finally {
            // This ALWAYS executes, ensuring driver is closed
            if (driver != null) {
                driver.quit();
                System.out.println("Driver closed successfully");
            }
        }
    }
}
```

#### **Example 2: File Operations**

```java
public class FileOperationExample {
    
    public void readFileWithFinally() {
        FileInputStream fis = null;
        try {
            fis = new FileInputStream("testdata.xlsx");
            // Read file operations
            int data = fis.read();
            // Process data...
        } catch (FileNotFoundException e) {
            System.out.println("File not found: " + e.getMessage());
        } catch (IOException e) {
            System.out.println("IO Error: " + e.getMessage());
        } finally {
            // Always close the stream to prevent resource leaks
            if (fis != null) {
                try {
                    fis.close();
                    System.out.println("File stream closed");
                } catch (IOException e) {
                    System.out.println("Error closing stream: " + e.getMessage());
                }
            }
        }
    }
}
```

#### **Example 3: Try-With-Resources (Modern Approach)**

```java
// Java 7+ Try-With-Resources automatically handles finally
public class TryWithResourcesExample {
    
    public void modernApproach() {
        // Automatically closes resources, equivalent to finally block
        try (FileInputStream fis = new FileInputStream("testdata.xlsx");
             BufferedInputStream bis = new BufferedInputStream(fis)) {
            
            // Read operations
            int data = bis.read();
            
        } catch (IOException e) {
            System.out.println("Error: " + e.getMessage());
        }
        // Resources are automatically closed here (implicit finally)
    }
    
    // For WebDriver (if implementing AutoCloseable)
    public void testWithAutoClose() {
        try (WebDriver driver = new ChromeDriver()) {
            driver.get("https://example.com");
            // Test operations
        } catch (Exception e) {
            System.out.println("Error: " + e.getMessage());
        }
        // Driver automatically closed
    }
}
```

#### **Example 4: Test Data Cleanup**

```java
public class TestDataCleanupExample {
    
    public void testWithCleanup() {
        String testUserId = null;
        try {
            // Create test data
            testUserId = createTestUser("testuser@example.com");
            
            // Perform test operations
            performLoginTest(testUserId);
            performCheckoutTest(testUserId);
            
        } catch (AssertionError e) {
            System.out.println("Test failed: " + e.getMessage());
            throw e; // Re-throw to mark test as failed
        } catch (Exception e) {
            System.out.println("Unexpected error: " + e.getMessage());
        } finally {
            // Always cleanup test data, even if test fails
            if (testUserId != null) {
                deleteTestUser(testUserId);
                System.out.println("Test data cleaned up");
            }
        }
    }
    
    private String createTestUser(String email) {
        // API call to create user
        return "user_12345";
    }
    
    private void deleteTestUser(String userId) {
        // API call to delete user
        System.out.println("Deleting user: " + userId);
    }
}
```

#### **Example 5: Finally with Return Statement**

```java
public class FinallyWithReturnExample {
    
    public int demonstrateFinally() {
        try {
            System.out.println("In try block");
            return 10; // Return value is stored
        } catch (Exception e) {
            System.out.println("In catch block");
            return 20;
        } finally {
            System.out.println("In finally block - ALWAYS executes");
            // This executes even though return is in try block
            // Note: You can modify return value, but it's not recommended
        }
        // Output:
        // In try block
        // In finally block - ALWAYS executes
        // Returns 10
    }
    
    public int finallyModifyingReturn() {
        int value = 0;
        try {
            value = 10;
            return value;
        } finally {
            value = 20; // This won't change the return value
            // But if you return here, it will override
            // return value; // This would return 20
        }
    }
}
```

### Scenario-Based Example

**Real Scenario in Test Automation:**

```java
public class TestExecutionWithFinally {
    private WebDriver driver;
    private String screenshotPath;
    
    @Test
    public void testCheckoutProcess() {
        try {
            // Setup
            driver = new ChromeDriver();
            driver.get("https://ecommerce.example.com");
            
            // Test steps
            login("user@example.com", "password");
            addProductToCart("product_123");
            proceedToCheckout();
            enterPaymentDetails();
            completePurchase();
            
            // Assertions
            Assert.assertTrue(isOrderConfirmed(), "Order should be confirmed");
            
        } catch (AssertionError e) {
            // Test failure - capture screenshot
            screenshotPath = captureScreenshot(driver, "checkout_failure");
            System.out.println("Test failed: " + e.getMessage());
            throw e; // Re-throw to mark test as failed
            
        } catch (Exception e) {
            // Unexpected error
            screenshotPath = captureScreenshot(driver, "checkout_error");
            System.out.println("Unexpected error: " + e.getMessage());
            throw new RuntimeException("Test execution failed", e);
            
        } finally {
            // CRITICAL: Always execute cleanup
            try {
                // Close browser
                if (driver != null) {
                    driver.quit();
                }
                
                // Cleanup test data
                cleanupTestData();
                
                // Log test completion
                logTestCompletion(screenshotPath);
                
            } catch (Exception cleanupError) {
                System.err.println("Error during cleanup: " + cleanupError.getMessage());
            }
        }
    }
    
    private void cleanupTestData() {
        // Delete test orders, reset cart, etc.
        System.out.println("Cleaning up test data...");
    }
    
    private void logTestCompletion(String screenshot) {
        // Log to test reporting system
        System.out.println("Test execution completed. Screenshot: " + screenshot);
    }
}
```

**Why Finally is Critical Here:**
- Even if test fails or crashes, browser must be closed (prevents resource leaks)
- Test data cleanup ensures no pollution for next test run
- Screenshot path logging ensures we can track failures
- Without finally, a crash would leave browser open and test data uncleaned

### Best Practices

1. **Always use finally for resource cleanup**
2. **Use try-with-resources when possible** (Java 7+)
3. **Keep finally block simple** - avoid complex logic
4. **Handle exceptions in finally** - wrap cleanup code in try-catch
5. **Don't return from finally** - it can mask exceptions

---

## 5. What is the difference between Scrum and Kanban?

### Context
This question assesses your understanding of Agile methodologies, which is essential for SDETs working in Agile teams. Understanding these frameworks helps in test planning, sprint planning, and daily collaboration.

### Detailed Answer

Both **Scrum** and **Kanban** are Agile frameworks for managing work, but they differ in structure, roles, ceremonies, and approach to workflow management.

### Key Differences

| Aspect | Scrum | Kanban |
|--------|-------|--------|
| **Structure** | Time-boxed iterations (Sprints) | Continuous flow |
| **Sprint Duration** | Fixed length (1-4 weeks, typically 2 weeks) | No sprints |
| **Roles** | Product Owner, Scrum Master, Development Team | No prescribed roles |
| **Ceremonies** | Sprint Planning, Daily Standup, Sprint Review, Retrospective | No mandatory ceremonies |
| **Work Items** | User Stories in Sprint Backlog | Work items in Kanban Board |
| **Commitment** | Team commits to Sprint Backlog | No commitment, work pulled as capacity allows |
| **Changes** | Changes not allowed during sprint | Changes can be made anytime |
| **Metrics** | Velocity, Sprint Burndown | Cycle Time, Lead Time, Throughput |
| **Board** | Resets each sprint | Continuous, persistent board |
| **Planning** | Sprint Planning at start of each sprint | Planning on-demand or continuous |

### Detailed Comparison

#### **Scrum Framework**

**Core Components:**

1. **Roles:**
   - **Product Owner:** Manages product backlog, prioritizes features
   - **Scrum Master:** Facilitates Scrum process, removes impediments
   - **Development Team:** Cross-functional team (includes SDETs, developers, etc.)

2. **Artifacts:**
   - **Product Backlog:** Prioritized list of features/user stories
   - **Sprint Backlog:** Items selected for current sprint
   - **Increment:** Working product at end of sprint

3. **Ceremonies:**
   - **Sprint Planning:** Select items for sprint (2-4 hours for 2-week sprint)
   - **Daily Standup:** 15-minute sync meeting
   - **Sprint Review:** Demo of completed work (1-2 hours)
   - **Sprint Retrospective:** Process improvement discussion (1-2 hours)

4. **Sprint Cycle:**
```
Sprint Planning → Daily Standups → Sprint Review → Retrospective → Next Sprint
     (2 weeks)         (Daily)         (Demo)        (Improve)
```

**Example Sprint Timeline:**
- **Week 1-2:** Sprint execution
- **Day 1:** Sprint Planning
- **Daily:** 15-min standup
- **Day 14:** Sprint Review + Retrospective

#### **Kanban Framework**

**Core Components:**

1. **Kanban Board:**
   - Visual representation of work flow
   - Columns: To Do → In Progress → Testing → Done
   - Work In Progress (WIP) limits on each column

2. **Principles:**
   - Visualize workflow
   - Limit work in progress
   - Manage flow
   - Make policies explicit
   - Implement feedback loops
   - Improve collaboratively

3. **No Fixed Roles:**
   - Team members can take on any role as needed
   - Focus on continuous improvement

4. **Continuous Flow:**
```
Backlog → To Do → In Progress → Testing → Code Review → Done
           (WIP:3)   (WIP:2)    (WIP:2)      (WIP:2)
```

### Visual Comparison

#### **Scrum Board (Resets Each Sprint)**
```
Sprint 1 (Week 1-2)
┌─────────┬──────────────┬──────────┬──────┐
│ To Do   │ In Progress  │ Testing  │ Done │
├─────────┼──────────────┼──────────┼──────┤
│ Story 3 │ Story 1       │ Story 2  │      │
│ Story 4 │              │          │      │
└─────────┴──────────────┴──────────┴──────┘

Sprint 2 (Week 3-4) - Board Resets
┌─────────┬──────────────┬──────────┬──────┐
│ To Do   │ In Progress  │ Testing  │ Done │
├─────────┼──────────────┼──────────┼──────┤
│ Story 5 │ Story 6       │          │      │
│ Story 7 │              │          │      │
└─────────┴──────────────┴──────────┴──────┘
```

#### **Kanban Board (Continuous)**
```
┌─────────┬──────────────┬──────────┬──────┐
│ Backlog │ In Progress  │ Testing  │ Done │
├─────────┼──────────────┼──────────┼──────┤
│ Story 8 │ Story 5       │ Story 4  │ S1-3 │
│ Story 9 │ Story 6       │          │ S7   │
│ Story 10│ (WIP:2)       │ (WIP:2)  │      │
└─────────┴──────────────┴──────────┴──────┘
         (WIP:3)
```

### Scenario-Based Examples

#### **Scenario 1: Testing in Scrum**

**Context:** SDET working in a 2-week Scrum sprint

**Week 1:**
- **Day 1 (Sprint Planning):** SDET estimates test cases for user stories, identifies test automation needs
- **Day 2-5:** SDET writes test cases, sets up test data, begins automation
- **Daily Standup:** SDET reports: "Completed test cases for Story 1, blocked on Story 2 due to missing API documentation"

**Week 2:**
- **Day 8-10:** Execute test cases, fix automation scripts, perform regression testing
- **Day 11:** Prepare test summary, demo scenarios
- **Day 12 (Sprint Review):** Demo test results, show test coverage metrics
- **Day 12 (Retrospective):** Discuss: "Test automation took longer than estimated, need better time estimation"

**Characteristics:**
- Fixed scope for 2 weeks
- Cannot add new stories mid-sprint
- Focus on completing committed work
- Clear sprint goal

#### **Scenario 2: Testing in Kanban**

**Context:** SDET working in Kanban team

**Workflow:**
- **Monday:** Pull Story A from backlog, start writing test cases (WIP limit: 2 testing items)
- **Tuesday:** Story A moved to "In Progress - Testing", Story B pulled when capacity available
- **Wednesday:** Urgent bug fix comes in, can immediately pull into "Testing" column (if WIP allows)
- **Thursday:** Story A completed, moved to "Done", pull next story
- **Friday:** Review cycle time metrics, identify bottlenecks

**Characteristics:**
- No fixed sprint boundaries
- Can prioritize urgent work anytime
- Focus on flow and cycle time
- Continuous improvement based on metrics

### When to Use Which?

**Use Scrum when:**
- Team needs structure and discipline
- Regular releases are acceptable (sprint-based)
- Requirements are relatively stable during sprint
- Team benefits from time-boxed planning
- Clear sprint goals help focus

**Use Kanban when:**
- Work arrives unpredictably (support, maintenance)
- Need flexibility to reprioritize frequently
- Focus on reducing cycle time
- Team is mature and self-organizing
- Continuous delivery is preferred

### Hybrid Approach: Scrumban

Many teams combine both:
- Use Kanban board for visualization
- Have daily standups (Scrum ceremony)
- Work in sprints but allow flexibility (Kanban principle)
- Use WIP limits (Kanban) within sprint (Scrum)

### SDET Perspective

**In Scrum:**
- Plan test activities during sprint planning
- Estimate test effort for user stories
- Execute tests within sprint timeline
- Demo test results in sprint review
- Improve testing process in retrospective

**In Kanban:**
- Pull test work as capacity allows
- Focus on reducing test cycle time
- Continuously improve test automation
- Respond quickly to urgent test requests
- Monitor test throughput metrics

---

## 6. Explain Smoke vs Sanity and Regression vs Retesting

### Context
This question tests your understanding of different testing types and their purposes. As an SDET, you need to clearly distinguish between these to plan test strategies effectively and communicate with stakeholders.

### Detailed Answer

These are fundamental testing types that serve different purposes in the software testing lifecycle. Understanding when and how to use each is critical for efficient test execution.

---

## **Smoke Testing vs Sanity Testing**

### **Smoke Testing**

**Definition:** Smoke testing is a **preliminary test** performed to verify that the **critical functionalities** of the application are working correctly after a build deployment.

**Purpose:**
- Quick validation that build is stable enough for further testing
- Catch major issues early before detailed testing
- "Build verification test" - ensures build didn't break core features

**Characteristics:**
- **Broad and shallow** - covers major features, not detailed
- **Quick execution** - typically 15-30 minutes
- **Performed after every build** - first test after deployment
- **Automated** - usually part of CI/CD pipeline
- **Non-exhaustive** - doesn't test all features deeply

**When to Perform:**
- After new build deployment
- After major code merges
- Before starting detailed testing
- In CI/CD pipeline as gatekeeper

**Example Test Cases:**
```
✓ Application launches successfully
✓ User can login
✓ Homepage loads correctly
✓ Database connection works
✓ Main navigation menu functions
✓ Critical API endpoints respond
```

### **Sanity Testing**

**Definition:** Sanity testing is a **focused test** performed to verify that **specific functionality** or **bug fix** works as expected after code changes.

**Purpose:**
- Verify that specific changes work correctly
- Ensure bug fixes don't introduce new issues
- Quick check on modified functionality

**Characteristics:**
- **Narrow and deep** - focuses on specific area/feature
- **Quick execution** - typically 10-20 minutes
- **Performed after bug fixes or small changes**
- **Manual or automated** - depends on scope
- **Selective** - only tests affected functionality

**When to Perform:**
- After bug fixes
- After small feature changes
- Before detailed regression testing
- To verify specific functionality works

**Example Test Cases (After Login Bug Fix):**
```
✓ Login with valid credentials works
✓ Login with invalid credentials shows error
✓ "Remember me" checkbox functions
✓ Password reset link works
```

### **Key Differences: Smoke vs Sanity**

| Aspect | Smoke Testing | Sanity Testing |
|--------|---------------|----------------|
| **Scope** | Broad - all critical features | Narrow - specific functionality |
| **Purpose** | Verify build stability | Verify specific changes work |
| **When** | After every build | After bug fixes/changes |
| **Depth** | Shallow - surface level | Deep - detailed for specific area |
| **Test Cases** | Pre-defined set | Ad-hoc, based on changes |
| **Automation** | Usually automated | Often manual |
| **Time** | 15-30 minutes | 10-20 minutes |
| **Goal** | "Is build testable?" | "Do changes work correctly?" |

### **Visual Representation**

```
Build Deployment
       ↓
   [Smoke Testing] ← "Is the build stable?"
       ↓ PASS
   Detailed Testing
       ↓
   Bug Found & Fixed
       ↓
   [Sanity Testing] ← "Does the fix work?"
       ↓ PASS
   Continue Testing
```

---

## **Regression Testing vs Retesting**

### **Regression Testing**

**Definition:** Regression testing ensures that **existing functionality** still works correctly after **new changes** (features, bug fixes, enhancements) are introduced.

**Purpose:**
- Verify that new changes don't break existing features
- Ensure overall system stability
- Maintain quality of previously working features

**Characteristics:**
- **Comprehensive** - tests existing functionality
- **Repeated** - same tests run multiple times
- **Automated** - usually automated for efficiency
- **Time-consuming** - can take hours/days
- **Preventive** - prevents regression bugs

**When to Perform:**
- After new feature development
- After bug fixes
- After code refactoring
- After environment changes
- Before production release

**Types:**
1. **Full Regression:** All test cases
2. **Partial Regression:** Selected test cases (affected areas)
3. **Unit Regression:** Unit level tests

### **Retesting**

**Definition:** Retesting is **re-executing** a **specific test case** that **previously failed** to verify that the **bug fix** works correctly.

**Purpose:**
- Verify that specific bug is fixed
- Confirm fix doesn't break anything else
- Validate defect resolution

**Characteristics:**
- **Focused** - only failed test cases
- **Specific** - targets particular bug/issue
- **Quick** - only relevant tests
- **Manual or automated** - depends on test
- **Reactive** - responds to failures

**When to Perform:**
- After bug fix is implemented
- To verify defect resolution
- Before closing bug ticket
- After code changes for specific issue

### **Key Differences: Regression vs Retesting**

| Aspect | Regression Testing | Retesting |
|--------|-------------------|-----------|
| **Purpose** | Verify existing features work | Verify specific bug is fixed |
| **Scope** | All/some existing features | Only failed test cases |
| **When** | After any code change | After bug fix |
| **Test Cases** | Pre-defined regression suite | Previously failed tests |
| **Focus** | Prevent new bugs | Verify fix works |
| **Time** | Longer (hours/days) | Shorter (minutes) |
| **Automation** | Usually automated | Often manual |
| **Type** | Preventive | Reactive |

### **Visual Representation**

```
New Feature Added / Bug Fixed
         ↓
    [Regression Testing]
    ┌─────────────────┐
    │ Test Case 1     │ ← Existing feature
    │ Test Case 2     │ ← Existing feature
    │ Test Case 3     │ ← Existing feature
    │ Test Case 4     │ ← Existing feature
    └─────────────────┘
         ↓
    Test Case 2 FAILS
         ↓
    Bug Reported & Fixed
         ↓
    [Retesting]
    ┌─────────────────┐
    │ Test Case 2     │ ← Only failed test
    └─────────────────┘
         ↓ PASS
    Bug Closed
```

---

## **Code Examples: Test Automation Perspective**

### **Smoke Test Suite**

```java
package tests.smoke;

import org.testng.annotations.Test;
import base.BaseTest;
import pages.LoginPage;
import pages.HomePage;
import pages.ProductPage;

public class SmokeTestSuite extends BaseTest {
    
    @Test(priority = 1, description = "Verify application launches")
    public void testApplicationLaunch() {
        driver.get(baseURL);
        Assert.assertTrue(driver.getTitle().contains("E-Commerce"), 
            "Application should launch");
    }
    
    @Test(priority = 2, description = "Verify login functionality")
    public void testLogin() {
        LoginPage loginPage = new LoginPage(driver);
        loginPage.login("validUser", "validPassword");
        Assert.assertTrue(loginPage.isLoggedIn(), "Login should work");
    }
    
    @Test(priority = 3, description = "Verify homepage loads")
    public void testHomepageLoads() {
        HomePage homePage = new HomePage(driver);
        Assert.assertTrue(homePage.isPageLoaded(), "Homepage should load");
    }
    
    @Test(priority = 4, description = "Verify product search")
    public void testProductSearch() {
        HomePage homePage = new HomePage(driver);
        homePage.searchProduct("laptop");
        Assert.assertTrue(homePage.areResultsDisplayed(), 
            "Search results should display");
    }
}
```

### **Sanity Test (After Login Bug Fix)**

```java
package tests.sanity;

import org.testng.annotations.Test;
import base.BaseTest;
import pages.LoginPage;

public class LoginSanityTest extends BaseTest {
    
    @Test(description = "Verify login bug fix - valid credentials")
    public void testLoginWithValidCredentials() {
        LoginPage loginPage = new LoginPage(driver);
        loginPage.login("testuser@example.com", "Password123");
        Assert.assertTrue(loginPage.isLoggedIn(), 
            "Login with valid credentials should work");
    }
    
    @Test(description = "Verify login bug fix - invalid credentials")
    public void testLoginWithInvalidCredentials() {
        LoginPage loginPage = new LoginPage(driver);
        loginPage.login("invalid@example.com", "wrongpass");
        Assert.assertEquals(loginPage.getErrorMessage(), 
            "Invalid credentials", "Error message should display");
    }
    
    @Test(description = "Verify remember me functionality")
    public void testRememberMe() {
        LoginPage loginPage = new LoginPage(driver);
        loginPage.loginWithRememberMe("testuser@example.com", "Password123");
        // Close and reopen browser
        driver.quit();
        driver = new ChromeDriver();
        driver.get(baseURL);
        Assert.assertTrue(loginPage.isUserRemembered(), 
            "User should be remembered");
    }
}
```

### **Regression Test Suite**

```java
package tests.regression;

import org.testng.annotations.Test;
import base.BaseTest;

public class FullRegressionSuite extends BaseTest {
    
    // Login Regression Tests
    @Test(groups = "regression", priority = 1)
    public void testLoginFunctionality() {
        // Test login scenarios
    }
    
    // Checkout Regression Tests
    @Test(groups = "regression", priority = 2)
    public void testCheckoutProcess() {
        // Test checkout scenarios
    }
    
    // Payment Regression Tests
    @Test(groups = "regression", priority = 3)
    public void testPaymentProcessing() {
        // Test payment scenarios
    }
    
    // Order Management Regression Tests
    @Test(groups = "regression", priority = 4)
    public void testOrderManagement() {
        // Test order scenarios
    }
    
    // Profile Regression Tests
    @Test(groups = "regression", priority = 5)
    public void testUserProfile() {
        // Test profile scenarios
    }
}

// TestNG XML for Regression
/*
<suite name="Regression Suite">
    <test name="Full Regression">
        <groups>
            <run>
                <include name="regression"/>
            </run>
        </groups>
        <classes>
            <class name="tests.regression.FullRegressionSuite"/>
        </classes>
    </test>
</suite>
*/
```

### **Retesting Example**

```java
package tests.retest;

import org.testng.annotations.Test;
import base.BaseTest;
import pages.CheckoutPage;

public class CheckoutRetest extends BaseTest {
    
    // This test failed previously due to payment gateway timeout bug
    // After bug fix, retesting to verify fix works
    @Test(description = "Retest: Payment gateway timeout issue - BUG-1234")
    public void testPaymentGatewayTimeout() {
        CheckoutPage checkoutPage = new CheckoutPage(driver);
        
        // Steps that previously failed
        checkoutPage.addProductToCart("product_123");
        checkoutPage.proceedToCheckout();
        checkoutPage.enterPaymentDetails("4111111111111111", "12/25", "123");
        
        // Previously: Timeout error occurred here
        // After fix: Should complete successfully
        checkoutPage.submitPayment();
        
        Assert.assertTrue(checkoutPage.isPaymentSuccessful(), 
            "Payment should complete without timeout - BUG-1234 fixed");
    }
}
```

---

## **Scenario-Based Real Questions**

### **Scenario 1: Build Deployment Decision**

**Situation:** New build deployed to QA environment. 200 test cases in regression suite.

**Question:** What testing approach would you follow?

**Answer:**
1. **First:** Run **Smoke Tests** (15-20 critical test cases, ~20 minutes)
   - If smoke tests fail → **STOP**, don't proceed, report to dev team
   - If smoke tests pass → Proceed to next step

2. **Second:** Run **Sanity Tests** (if specific bug fixes were included)
   - Verify those specific fixes work

3. **Third:** Run **Regression Tests** (full or partial based on changes)
   - Automated regression suite
   - Focus on affected areas if partial regression

4. **If bugs found:** After fixes, perform **Retesting** of failed test cases

**Rationale:** Smoke tests act as gatekeeper, preventing wasted time on unstable builds.

---

### **Scenario 2: Bug Fix Verification**

**Situation:** Bug BUG-5678 fixed: "Login fails with special characters in password"

**Question:** How would you verify the fix?

**Answer:**
1. **Retesting:**
   - Re-run the exact test case that failed originally
   - Test with special characters: `P@ssw0rd!@#`
   - Verify login works now

2. **Sanity Testing:**
   - Test login with various special characters
   - Test edge cases: `!@#$%^&*()`
   - Verify error handling still works for invalid cases

3. **Regression Testing:**
   - Run login-related regression tests
   - Ensure fix didn't break other login scenarios
   - Test password reset, remember me, etc.

**Rationale:** Retest verifies fix, sanity ensures it works broadly, regression ensures no side effects.

---

### **Scenario 3: Time-Constrained Release**

**Situation:** Production release in 2 hours. Limited time for testing.

**Question:** What testing strategy would you use?

**Answer:**
1. **Smoke Testing** (MUST DO - 20 minutes)
   - Critical path: Login → Browse → Add to Cart → Checkout → Payment
   - If smoke fails → **BLOCK RELEASE**

2. **Sanity Testing** (if time permits - 15 minutes)
   - Test only the new features/changes in this release
   - Verify critical bug fixes

3. **Partial Regression** (if time permits - 30 minutes)
   - Only affected areas/modules
   - High-priority test cases

4. **Skip Full Regression** (not feasible in 2 hours)

**Rationale:** Smoke tests ensure critical functionality works. Better to have limited confidence than no confidence.

---

### **Scenario 4: CI/CD Pipeline Integration**

**Situation:** Setting up automated testing in CI/CD pipeline.

**Question:** Where would you place each test type?

**Answer:**

```
CI/CD Pipeline:
┌─────────────────────────────────────────┐
│ 1. Code Commit                          │
│    ↓                                    │
│ 2. Build                                │
│    ↓                                    │
│ 3. [SMOKE TESTS] ← Automated, Gate     │
│    ↓ PASS                               │
│ 4. Unit Tests                           │
│    ↓                                    │
│ 5. Integration Tests                    │
│    ↓                                    │
│ 6. [SANITY TESTS] ← If specific changes │
│    ↓                                    │
│ 7. [REGRESSION TESTS] ← Automated suite │
│    ↓                                    │
│ 8. Deploy to QA                         │
└─────────────────────────────────────────┘

After Bug Fix:
┌─────────────────────────────────────────┐
│ Bug Fix Committed                       │
│    ↓                                    │
│ [RETESTING] ← Automated, specific test  │
│    ↓ PASS                               │
│ Continue Pipeline                       │
└─────────────────────────────────────────┘
```

**Rationale:** Smoke tests prevent bad builds from proceeding. Regression ensures stability. Retesting verifies fixes.

---

## **Summary Table**

| Test Type | Purpose | When | Scope | Time | Automation |
|-----------|---------|------|-------|------|------------|
| **Smoke** | Verify build stability | After build | Broad, shallow | 15-30 min | Usually |
| **Sanity** | Verify specific changes | After fixes | Narrow, deep | 10-20 min | Often manual |
| **Regression** | Verify existing features | After changes | Comprehensive | Hours/Days | Usually |
| **Retesting** | Verify bug fix | After fix | Specific failed tests | Minutes | Often manual |

---

## **Best Practices**

1. **Always run smoke tests first** - Don't waste time on unstable builds
2. **Automate smoke and regression** - Saves time, ensures consistency
3. **Keep smoke tests fast** - Should complete in < 30 minutes
4. **Maintain separate test suites** - Smoke, Sanity, Regression, Retest
5. **Document test coverage** - Know what each suite covers
6. **Review and update regularly** - Keep test suites relevant
7. **Use test tags/groups** - Easy to run specific suites (TestNG groups, JUnit tags)

---

## 7. How do you make TestNG take screenshots only for failed test cases?

### Context
This question tests your understanding of TestNG listeners, exception handling, and automation best practices. Screenshot capture on failure is crucial for debugging and reporting in test automation.

### Detailed Answer

TestNG provides **ITestListener** interface that allows you to hook into the test execution lifecycle. By implementing this interface, you can capture screenshots automatically when tests fail, without modifying individual test methods.

### Implementation Approaches

#### **Approach 1: Using ITestListener (Recommended)**

**Step 1: Create Screenshot Utility Class**

```java
package utils;

import org.apache.commons.io.FileUtils;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import java.io.File;
import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class ScreenshotUtil {
    
    private static final String SCREENSHOT_DIR = System.getProperty("user.dir") + "/screenshots/";
    
    /**
     * Captures screenshot and saves it with timestamp
     * @param driver WebDriver instance
     * @param screenshotName Name for the screenshot file
     * @return Path to the saved screenshot
     */
    public static String captureScreenshot(WebDriver driver, String screenshotName) {
        String timestamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
        String fileName = screenshotName + "_" + timestamp + ".png";
        String filePath = SCREENSHOT_DIR + fileName;
        
        try {
            // Create directory if it doesn't exist
            File directory = new File(SCREENSHOT_DIR);
            if (!directory.exists()) {
                directory.mkdirs();
            }
            
            // Capture screenshot
            TakesScreenshot ts = (TakesScreenshot) driver;
            File sourceFile = ts.getScreenshotAs(OutputType.FILE);
            File destinationFile = new File(filePath);
            
            // Copy file to destination
            FileUtils.copyFile(sourceFile, destinationFile);
            
            System.out.println("Screenshot captured: " + filePath);
            return filePath;
            
        } catch (IOException e) {
            System.out.println("Exception while taking screenshot: " + e.getMessage());
            return null;
        }
    }
    
    /**
     * Captures screenshot as Base64 string (for embedding in reports)
     */
    public static String captureScreenshotAsBase64(WebDriver driver) {
        TakesScreenshot ts = (TakesScreenshot) driver;
        return ts.getScreenshotAs(OutputType.BASE64);
    }
}
```

**Step 2: Create TestNG Listener Class**

```java
package listeners;

import org.testng.ITestListener;
import org.testng.ITestResult;
import org.testng.Reporter;
import utils.ScreenshotUtil;
import utils.DriverManager;

public class ScreenshotListener implements ITestListener {
    
    @Override
    public void onTestFailure(ITestResult result) {
        // Get WebDriver instance (assuming stored in ThreadLocal)
        WebDriver driver = DriverManager.getDriver("chrome");
        
        if (driver != null) {
            // Capture screenshot
            String screenshotPath = ScreenshotUtil.captureScreenshot(
                driver, 
                result.getMethod().getMethodName()
            );
            
            // Attach screenshot to TestNG report
            if (screenshotPath != null) {
                Reporter.log("<br><a href='" + screenshotPath + "'> <img src='" + screenshotPath + 
                    "' height='200' width='200'/> </a><br>");
                
                // Also log the path
                Reporter.log("Screenshot saved at: " + screenshotPath);
                
                // For ExtentReports or Allure integration
                // ExtentTestManager.getTest().addScreenCaptureFromPath(screenshotPath);
            }
        }
    }
    
    // Optional: Capture screenshot on test skip
    @Override
    public void onTestSkipped(ITestResult result) {
        WebDriver driver = DriverManager.getDriver("chrome");
        if (driver != null) {
            ScreenshotUtil.captureScreenshot(
                driver, 
                result.getMethod().getMethodName() + "_SKIPPED"
            );
        }
    }
}
```

**Step 3: Configure TestNG XML to Use Listener**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="Test Suite" verbose="1">
    
    <!-- Register the listener -->
    <listeners>
        <listener class-name="listeners.ScreenshotListener"/>
    </listeners>
    
    <test name="Test Cases">
        <classes>
            <class name="tests.LoginTest"/>
            <class name="tests.CheckoutTest"/>
        </classes>
    </test>
</suite>
```

**Step 4: Alternative - Register Listener Programmatically**

```java
package tests;

import org.testng.TestListenerAdapter;
import org.testng.TestNG;
import listeners.ScreenshotListener;

public class TestRunner {
    public static void main(String[] args) {
        TestNG testng = new TestNG();
        testng.setTestClasses(new Class[] { LoginTest.class });
        
        // Add listener programmatically
        testng.addListener(new ScreenshotListener());
        
        testng.run();
    }
}
```

#### **Approach 2: Using @AfterMethod with Conditional Screenshot**

```java
package base;

import org.openqa.selenium.WebDriver;
import org.testng.ITestResult;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import utils.DriverManager;
import utils.ScreenshotUtil;

public class BaseTest {
    protected WebDriver driver;
    
    @BeforeMethod
    public void setUp() {
        driver = DriverManager.getDriver("chrome");
        driver.get("https://example.com");
    }
    
    @AfterMethod
    public void tearDown(ITestResult result) {
        // Capture screenshot only if test failed
        if (result.getStatus() == ITestResult.FAILURE) {
            ScreenshotUtil.captureScreenshot(
                driver, 
                result.getMethod().getMethodName()
            );
        }
        
        if (driver != null) {
            driver.quit();
        }
    }
}
```

#### **Approach 3: Using IRetryAnalyzer with Screenshot**

```java
package listeners;

import org.testng.IRetryAnalyzer;
import org.testng.ITestResult;
import utils.ScreenshotUtil;
import utils.DriverManager;

public class RetryAnalyzerWithScreenshot implements IRetryAnalyzer {
    private int retryCount = 0;
    private static final int MAX_RETRY_COUNT = 2;
    
    @Override
    public boolean retry(ITestResult result) {
        if (retryCount < MAX_RETRY_COUNT) {
            // Capture screenshot before retry
            WebDriver driver = DriverManager.getDriver("chrome");
            if (driver != null) {
                ScreenshotUtil.captureScreenshot(
                    driver, 
                    result.getMethod().getMethodName() + "_RETRY_" + retryCount
                );
            }
            retryCount++;
            return true;
        }
        return false;
    }
}

// Usage in test class
@Test(retryAnalyzer = RetryAnalyzerWithScreenshot.class)
public void flakyTest() {
    // Test code
}
```

#### **Approach 4: Enhanced Listener with Multiple Conditions**

```java
package listeners;

import org.testng.ITestListener;
import org.testng.ITestResult;
import org.testng.Reporter;
import utils.ScreenshotUtil;
import utils.DriverManager;

public class EnhancedScreenshotListener implements ITestListener {
    
    @Override
    public void onTestFailure(ITestResult result) {
        captureScreenshot(result, "FAILED");
    }
    
    @Override
    public void onTestSkipped(ITestResult result) {
        // Optional: Capture screenshot on skip
        if (shouldCaptureOnSkip()) {
            captureScreenshot(result, "SKIPPED");
        }
    }
    
    private void captureScreenshot(ITestResult result, String status) {
        try {
            WebDriver driver = DriverManager.getDriver("chrome");
            
            if (driver != null) {
                String methodName = result.getMethod().getMethodName();
                String className = result.getTestClass().getName();
                String screenshotName = className + "_" + methodName + "_" + status;
                
                String screenshotPath = ScreenshotUtil.captureScreenshot(
                    driver, 
                    screenshotName
                );
                
                // Log to TestNG report
                Reporter.log("<br>Test " + status + ": " + methodName);
                Reporter.log("<br><a href='" + screenshotPath + "'>View Screenshot</a>");
                
                // Store in result for external reporting tools
                result.setAttribute("screenshot", screenshotPath);
                
                // Log exception if present
                if (result.getThrowable() != null) {
                    Reporter.log("<br>Exception: " + result.getThrowable().getMessage());
                }
            }
        } catch (Exception e) {
            System.err.println("Error capturing screenshot: " + e.getMessage());
        }
    }
    
    private boolean shouldCaptureOnSkip() {
        // Configuration: return true if you want screenshots on skip
        return false; // Usually false, only failures need screenshots
    }
}
```

### Integration with Reporting Tools

#### **ExtentReports Integration**

```java
package listeners;

import com.aventstack.extentreports.ExtentReports;
import com.aventstack.extentreports.ExtentTest;
import com.aventstack.extentreports.MediaEntityBuilder;
import com.aventstack.extentreports.Status;
import org.testng.ITestListener;
import org.testng.ITestResult;
import utils.ScreenshotUtil;
import utils.DriverManager;

public class ExtentReportListener implements ITestListener {
    private static ExtentReports extent = new ExtentReports();
    private static ThreadLocal<ExtentTest> test = new ThreadLocal<>();
    
    @Override
    public void onTestFailure(ITestResult result) {
        WebDriver driver = DriverManager.getDriver("chrome");
        
        if (driver != null) {
            String screenshotPath = ScreenshotUtil.captureScreenshot(
                driver, 
                result.getMethod().getMethodName()
            );
            
            // Attach screenshot to ExtentReport
            try {
                test.get().fail(result.getThrowable());
                test.get().fail("Screenshot", MediaEntityBuilder
                    .createScreenCaptureFromPath(screenshotPath).build());
            } catch (Exception e) {
                test.get().fail("Screenshot capture failed: " + e.getMessage());
            }
        }
    }
}
```

#### **Allure Integration**

```java
package listeners;

import io.qameta.allure.Attachment;
import org.testng.ITestListener;
import org.testng.ITestResult;
import utils.ScreenshotUtil;
import utils.DriverManager;
import org.openqa.selenium.WebDriver;

public class AllureScreenshotListener implements ITestListener {
    
    @Override
    public void onTestFailure(ITestResult result) {
        WebDriver driver = DriverManager.getDriver("chrome");
        if (driver != null) {
            attachScreenshot(driver, result.getMethod().getMethodName());
        }
    }
    
    @Attachment(value = "Screenshot", type = "image/png")
    public byte[] attachScreenshot(WebDriver driver, String testName) {
        String base64Screenshot = ScreenshotUtil.captureScreenshotAsBase64(driver);
        return java.util.Base64.getDecoder().decode(base64Screenshot);
    }
}
```

### Complete Example: Test Class

```java
package tests;

import base.BaseTest;
import org.testng.Assert;
import org.testng.annotations.Test;
import pages.LoginPage;

public class LoginTest extends BaseTest {
    
    @Test
    public void testSuccessfulLogin() {
        LoginPage loginPage = new LoginPage(driver);
        loginPage.login("validUser", "validPassword");
        Assert.assertTrue(loginPage.isLoggedIn(), "Login should succeed");
    }
    
    @Test
    public void testInvalidLogin() {
        LoginPage loginPage = new LoginPage(driver);
        loginPage.login("invalidUser", "invalidPassword");
        // This will fail and trigger screenshot capture
        Assert.assertTrue(loginPage.isLoggedIn(), "Login should fail");
    }
}
```

### Scenario-Based Example

**Real Scenario:**
In a CI/CD pipeline, tests run overnight and failures need to be debugged the next morning. Without screenshots, it's difficult to understand what went wrong.

**Solution Implementation:**

```java
// Enhanced listener that captures screenshots and sends email notifications
package listeners;

import org.testng.ITestListener;
import org.testng.ITestResult;
import utils.ScreenshotUtil;
import utils.DriverManager;
import utils.EmailUtil;

public class CIListener implements ITestListener {
    private int failureCount = 0;
    
    @Override
    public void onTestFailure(ITestResult result) {
        failureCount++;
        WebDriver driver = DriverManager.getDriver("chrome");
        
        if (driver != null) {
            String screenshotPath = ScreenshotUtil.captureScreenshot(
                driver, 
                result.getMethod().getMethodName()
            );
            
            // Store failure details
            String failureDetails = 
                "Test: " + result.getMethod().getMethodName() + "\n" +
                "Class: " + result.getTestClass().getName() + "\n" +
                "Exception: " + result.getThrowable().getMessage() + "\n" +
                "Screenshot: " + screenshotPath;
            
            // In CI environment, send email notification
            if (System.getProperty("ci.environment") != null) {
                EmailUtil.sendFailureNotification(failureDetails, screenshotPath);
            }
        }
    }
    
    @Override
    public void onFinish(ITestResult result) {
        // Summary after all tests complete
        System.out.println("Total failures: " + failureCount);
    }
}
```

**Benefits:**
- Automatic screenshot capture on failure
- No code changes needed in test methods
- Screenshots attached to reports
- Easy debugging of failures

### Best Practices

1. **Use ITestListener** - Clean separation of concerns
2. **Store screenshots with meaningful names** - Include test name, timestamp, status
3. **Organize screenshot directory** - Create subdirectories by date/test run
4. **Clean up old screenshots** - Implement cleanup mechanism
5. **Handle driver null checks** - Screenshot capture should not fail tests
6. **Integrate with reporting tools** - Attach screenshots to test reports
7. **Capture only on failure** - Don't waste storage on passed tests
8. **Use Base64 for reports** - Some reporting tools prefer Base64 embedded images

---

## 8. How do you perform data-driven testing in Postman?

### Context
This question tests your understanding of API testing strategies, data management, and how to efficiently test APIs with multiple data sets. Data-driven testing is essential for comprehensive API validation.

### Detailed Answer

Data-driven testing in Postman allows you to run the same API request with multiple sets of data, making testing efficient and comprehensive. Postman provides several methods to achieve this.

### Method 1: Using CSV/JSON Files (Collection Runner)

#### **Step 1: Create Test Data File**

**CSV Format (testdata.csv):**
```csv
username,password,expectedStatus,expectedMessage
john.doe@example.com,Password123,200,Login successful
jane.smith@example.com,WrongPass,401,Invalid credentials
admin@example.com,Admin123,200,Login successful
invalid@example.com,Pass123,401,Invalid credentials
```

**JSON Format (testdata.json):**
```json
[
    {
        "username": "john.doe@example.com",
        "password": "Password123",
        "expectedStatus": 200,
        "expectedMessage": "Login successful"
    },
    {
        "username": "jane.smith@example.com",
        "password": "WrongPass",
        "expectedStatus": 401,
        "expectedMessage": "Invalid credentials"
    },
    {
        "username": "admin@example.com",
        "password": "Admin123",
        "expectedStatus": 200,
        "expectedMessage": "Login successful"
    }
]
```

#### **Step 2: Configure Postman Request**

**Request Setup:**
- **Method:** POST
- **URL:** `https://api.example.com/login`
- **Body (raw JSON):**
```json
{
    "username": "{{username}}",
    "password": "{{password}}"
}
```

**Pre-request Script:**
```javascript
// Optional: Generate dynamic data
pm.environment.set("timestamp", new Date().getTime());
```

**Tests Script:**
```javascript
// Parse response
var jsonData = pm.response.json();

// Assert status code
pm.test("Status code is " + pm.variables.get("expectedStatus"), function () {
    pm.response.to.have.status(parseInt(pm.variables.get("expectedStatus")));
});

// Assert response message
pm.test("Response message matches", function () {
    pm.expect(jsonData.message).to.eql(pm.variables.get("expectedMessage"));
});

// Assert response time
pm.test("Response time is less than 500ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(500);
});

// Store token if login successful
if (pm.response.code === 200 && jsonData.token) {
    pm.environment.set("authToken", jsonData.token);
}

// Log test data for debugging
console.log("Test Data - Username: " + pm.variables.get("username"));
console.log("Test Data - Expected Status: " + pm.variables.get("expectedStatus"));
```

#### **Step 3: Run Collection Runner**

1. Click **Collections** → Select your collection
2. Click **Run** button
3. Click **Select File** → Choose your CSV/JSON file
4. Configure options:
   - **Iterations:** Number of times to run (usually matches data rows)
   - **Delay:** Delay between requests
   - **Data Format:** CSV or JSON
5. Click **Run Collection**

### Method 2: Using Pre-request Scripts with Arrays

**Pre-request Script:**
```javascript
// Define test data array
var testData = [
    {username: "user1@example.com", password: "Pass123", expectedStatus: 200},
    {username: "user2@example.com", password: "WrongPass", expectedStatus: 401},
    {username: "user3@example.com", password: "Pass456", expectedStatus: 200}
];

// Get current iteration (set in Collection Runner)
var currentIteration = pm.info.iteration || 0;
var data = testData[currentIteration];

// Set variables for this iteration
pm.variables.set("username", data.username);
pm.variables.set("password", data.password);
pm.variables.set("expectedStatus", data.expectedStatus);
```

**Request Body:**
```json
{
    "username": "{{username}}",
    "password": "{{password}}"
}
```

### Method 3: Using Environment Variables with Multiple Environments

**Create Multiple Environments:**

**Environment 1: Test Data Set 1**
```json
{
    "username": "testuser1@example.com",
    "password": "Test123",
    "expectedStatus": "200"
}
```

**Environment 2: Test Data Set 2**
```json
{
    "username": "testuser2@example.com",
    "password": "Test456",
    "expectedStatus": "200"
}
```

Run collection multiple times, switching environments each time.

### Method 4: Using Newman (Command Line) with Data Files

**Install Newman:**
```bash
npm install -g newman
npm install -g newman-reporter-html
```

**Run with CSV Data:**
```bash
newman run "Collection.json" \
    -d "testdata.csv" \
    -e "Environment.json" \
    -r html \
    --reporters cli,html \
    --reporter-html-export "report.html"
```

**Run with JSON Data:**
```bash
newman run "Collection.json" \
    -d "testdata.json" \
    -e "Environment.json" \
    -n 3 \
    --delay-request 1000
```

### Method 5: Dynamic Data Generation

**Pre-request Script for Dynamic Data:**
```javascript
// Generate random email
function generateRandomEmail() {
    var random = Math.floor(Math.random() * 10000);
    return "testuser" + random + "@example.com";
}

// Generate random password
function generateRandomPassword() {
    return "Test" + Math.floor(Math.random() * 1000) + "!";
}

// Set dynamic variables
pm.variables.set("dynamicUsername", generateRandomEmail());
pm.variables.set("dynamicPassword", generateRandomPassword());
pm.variables.set("timestamp", new Date().toISOString());
```

### Complete Example: E-commerce API Testing

#### **Test Data File (ecommerce_testdata.csv):**
```csv
productId,quantity,expectedStatus,expectedTotal,userId
101,2,200,199.98,user123
102,1,200,49.99,user123
999,1,404,0,user123
101,0,400,0,user123
101,-1,400,0,user123
```

#### **Add to Cart Request:**

**URL:** `POST https://api.example.com/cart/add`

**Body:**
```json
{
    "productId": {{productId}},
    "quantity": {{quantity}},
    "userId": "{{userId}}"
}
```

**Tests:**
```javascript
pm.test("Status code validation", function () {
    pm.response.to.have.status(parseInt(pm.variables.get("expectedStatus")));
});

pm.test("Total amount validation", function () {
    if (pm.response.code === 200) {
        var jsonData = pm.response.json();
        pm.expect(jsonData.total).to.eql(parseFloat(pm.variables.get("expectedTotal")));
    }
});

pm.test("Response schema validation", function () {
    if (pm.response.code === 200) {
        pm.response.to.have.jsonSchema({
            "type": "object",
            "properties": {
                "cartId": {"type": "string"},
                "total": {"type": "number"},
                "items": {"type": "array"}
            },
            "required": ["cartId", "total", "items"]
        });
    }
});

// Store cart ID for subsequent requests
if (pm.response.code === 200) {
    var jsonData = pm.response.json();
    pm.environment.set("cartId", jsonData.cartId);
}
```

### Method 6: Using Postman Scripts for Complex Data-Driven Scenarios

**Pre-request Script (Complex Scenario):**
```javascript
// Read data from external file or generate based on conditions
var testScenarios = [
    {
        name: "Valid Purchase",
        productId: 101,
        quantity: 2,
        paymentMethod: "credit_card",
        expectedStatus: 200
    },
    {
        name: "Out of Stock",
        productId: 999,
        quantity: 1,
        paymentMethod: "credit_card",
        expectedStatus: 400
    },
    {
        name: "Invalid Payment",
        productId: 101,
        quantity: 1,
        paymentMethod: "invalid",
        expectedStatus: 400
    }
];

var iteration = pm.info.iteration;
var scenario = testScenarios[iteration % testScenarios.length];

// Set variables
pm.variables.set("scenarioName", scenario.name);
pm.variables.set("productId", scenario.productId);
pm.variables.set("quantity", scenario.quantity);
pm.variables.set("paymentMethod", scenario.paymentMethod);
pm.variables.set("expectedStatus", scenario.expectedStatus);

console.log("Running scenario: " + scenario.name);
```

### Scenario-Based Example

**Real Scenario: Testing User Registration API**

**Requirement:** Test registration API with 100 different user combinations (valid/invalid emails, passwords, etc.)

**Solution:**

**Step 1: Create testdata.csv**
```csv
email,password,firstName,lastName,expectedStatus,expectedError
valid@example.com,Pass123!,John,Doe,201,
invalid-email,Pass123!,Jane,Smith,400,Invalid email format
test@example.com,weak,Alice,Brown,400,Password too weak
test@example.com,ValidPass123!,Bob,Wilson,201,
test@example.com,ValidPass123!,Charlie,Green,409,Email already exists
```

**Step 2: Configure Registration Request**

**URL:** `POST https://api.example.com/register`

**Body:**
```json
{
    "email": "{{email}}",
    "password": "{{password}}",
    "firstName": "{{firstName}}",
    "lastName": "{{lastName}}"
}
```

**Tests:**
```javascript
// Status code validation
pm.test("Status code is " + pm.variables.get("expectedStatus"), function () {
    pm.response.to.have.status(parseInt(pm.variables.get("expectedStatus")));
});

// Error message validation for failed cases
if (pm.response.code >= 400) {
    pm.test("Error message validation", function () {
        var jsonData = pm.response.json();
        var expectedError = pm.variables.get("expectedError");
        if (expectedError) {
            pm.expect(jsonData.error).to.include(expectedError);
        }
    });
}

// Success validation
if (pm.response.code === 201) {
    pm.test("User created successfully", function () {
        var jsonData = pm.response.json();
        pm.expect(jsonData).to.have.property("userId");
        pm.expect(jsonData.email).to.eql(pm.variables.get("email"));
    });
    
    // Store user ID for cleanup
    pm.environment.set("createdUserId", jsonData.userId);
}
```

**Step 3: Run Collection Runner**
- Select CSV file with 100 rows
- Set iterations to 100
- Run and review results

**Benefits:**
- Test 100 scenarios in one run
- Easy to add/modify test data
- Clear pass/fail results per data set
- Reusable for regression testing

### Best Practices

1. **Use CSV for simple data** - Easy to edit in Excel
2. **Use JSON for complex nested data** - Better for hierarchical structures
3. **Validate all responses** - Don't just check status codes
4. **Store dynamic values** - Save tokens, IDs for subsequent requests
5. **Use descriptive variable names** - Makes tests readable
6. **Handle both success and failure cases** - Test negative scenarios
7. **Clean up test data** - Delete created resources after tests
8. **Use Pre-request scripts** - For dynamic data generation
9. **Document test data** - Add comments explaining test scenarios
10. **Version control test data** - Keep data files in Git

---

## 9. What important checks do you do while testing an API?

### Context
This question evaluates your comprehensive understanding of API testing, covering functional, non-functional, and security aspects. As an SDET, you need to think beyond just status codes.

### Detailed Answer

API testing requires a holistic approach covering multiple dimensions. Here's a comprehensive checklist organized by category:

### 1. Functional Testing Checks

#### **Request Validation**
- ✅ **HTTP Method Validation:** Verify correct method (GET, POST, PUT, DELETE, PATCH)
- ✅ **Endpoint URL:** Correct path, parameters, query strings
- ✅ **Request Headers:** Content-Type, Authorization, Accept headers
- ✅ **Request Body:** Valid JSON/XML structure, required fields present
- ✅ **Path Parameters:** Valid format, type checking
- ✅ **Query Parameters:** Correct values, encoding, special characters

#### **Response Validation**
- ✅ **Status Codes:** Correct HTTP status (200, 201, 400, 401, 404, 500, etc.)
- ✅ **Response Headers:** Content-Type, Content-Length, CORS headers
- ✅ **Response Body:** Valid JSON/XML structure
- ✅ **Response Schema:** Matches expected schema/structure
- ✅ **Data Accuracy:** Response data matches expected values
- ✅ **Data Types:** Correct data types (string, number, boolean, array, object)

### 2. Data Validation Checks

#### **Input Validation**
- ✅ **Required Fields:** Missing required fields return 400
- ✅ **Field Types:** Wrong data types return 400
- ✅ **Field Length:** Min/max length validation
- ✅ **Special Characters:** Handling of special chars, SQL injection attempts
- ✅ **Boundary Values:** Min, max, null, empty string, zero
- ✅ **Format Validation:** Email format, date format, phone number format

#### **Output Validation**
- ✅ **Data Completeness:** All expected fields present
- ✅ **Data Consistency:** Data matches database state
- ✅ **Data Formatting:** Dates, numbers, currency formatted correctly
- ✅ **Null Handling:** Null values handled appropriately
- ✅ **Empty Arrays/Objects:** Proper handling of empty collections

### 3. Error Handling Checks

- ✅ **Error Messages:** Clear, meaningful error messages
- ✅ **Error Codes:** Consistent error code structure
- ✅ **Error Format:** Standardized error response format
- ✅ **4xx Errors:** Proper client error handling (400, 401, 403, 404, 409, 422)
- ✅ **5xx Errors:** Server error handling (500, 502, 503, 504)
- ✅ **Exception Handling:** Graceful handling of unexpected errors
- ✅ **Error Logging:** Errors logged appropriately (without exposing sensitive data)

### 4. Authentication & Authorization Checks

- ✅ **Authentication:** Valid tokens/credentials work
- ✅ **Invalid Credentials:** Rejected with 401
- ✅ **Expired Tokens:** Handled appropriately
- ✅ **Missing Auth:** Requests without auth return 401
- ✅ **Authorization:** Role-based access control (RBAC)
- ✅ **Permission Checks:** Users can only access authorized resources
- ✅ **Token Refresh:** Refresh token mechanism works

### 5. Performance Checks

- ✅ **Response Time:** Meets SLA requirements (< 200ms for simple, < 1s for complex)
- ✅ **Throughput:** Handles expected load
- ✅ **Concurrent Requests:** Multiple simultaneous requests handled
- ✅ **Timeout Handling:** Proper timeout configuration
- ✅ **Resource Usage:** Memory, CPU usage acceptable
- ✅ **Database Queries:** Optimized, no N+1 queries

### 6. Security Checks

- ✅ **SQL Injection:** Inputs sanitized, no SQL injection possible
- ✅ **XSS (Cross-Site Scripting):** Scripts in input properly escaped
- ✅ **CSRF Protection:** CSRF tokens validated
- ✅ **Sensitive Data:** No sensitive data in URLs, logs, error messages
- ✅ **HTTPS:** API uses HTTPS (not HTTP)
- ✅ **Input Sanitization:** All inputs sanitized before processing
- ✅ **Rate Limiting:** Prevents abuse, DDoS protection
- ✅ **Data Encryption:** Sensitive data encrypted in transit and at rest

### 7. Integration Checks

- ✅ **Database Integration:** Data persisted correctly
- ✅ **External Services:** Third-party API integrations work
- ✅ **Message Queues:** Async operations processed correctly
- ✅ **File Uploads:** File handling works correctly
- ✅ **Webhooks:** Webhook calls triggered correctly

### 8. Business Logic Checks

- ✅ **Workflow Validation:** Business rules enforced
- ✅ **State Transitions:** Valid state changes allowed
- ✅ **Calculations:** Business calculations accurate
- ✅ **Validation Rules:** Business validation rules applied
- ✅ **Idempotency:** PUT/DELETE operations idempotent

### Code Examples: Comprehensive API Test Suite

#### **Example 1: Complete API Test with All Checks**

```java
package api.tests;

import io.restassured.RestAssured;
import io.restassured.http.ContentType;
import io.restassured.response.Response;
import org.testng.annotations.BeforeClass;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.*;

public class ComprehensiveAPITest {
    
    private String authToken;
    private String baseURL = "https://api.example.com";
    
    @BeforeClass
    public void setup() {
        RestAssured.baseURI = baseURL;
        authToken = getAuthToken();
    }
    
    @Test(priority = 1, description = "Test API with all validation checks")
    public void testCreateUserWithAllChecks() {
        String requestBody = """
            {
                "email": "testuser@example.com",
                "password": "SecurePass123!",
                "firstName": "John",
                "lastName": "Doe",
                "age": 25
            }
            """;
        
        Response response = given()
            .header("Content-Type", "application/json")
            .header("Authorization", "Bearer " + authToken)
            .body(requestBody)
        .when()
            .post("/users")
        .then()
            // Status Code Check
            .statusCode(201)
            
            // Response Time Check
            .time(lessThan(1000L))
            
            // Response Header Checks
            .header("Content-Type", containsString("application/json"))
            .header("Location", notNullValue())
            
            // Response Body Schema Checks
            .body("userId", notNullValue())
            .body("email", equalTo("testuser@example.com"))
            .body("firstName", equalTo("John"))
            .body("lastName", equalTo("Doe"))
            .body("age", equalTo(25))
            .body("createdAt", notNullValue())
            
            // Data Type Checks
            .body("userId", instanceOf(String.class))
            .body("age", instanceOf(Integer.class))
            
            // Response Structure Checks
            .body("$", hasKey("userId"))
            .body("$", hasKey("email"))
            .body("$", hasKey("createdAt"))
            
            // Extract response for further validation
            .extract().response();
        
        // Additional Business Logic Checks
        String userId = response.jsonPath().getString("userId");
        assert userId != null && !userId.isEmpty() : "User ID should not be empty";
        
        // Verify user was created in database (integration check)
        verifyUserInDatabase(userId);
    }
    
    @Test(priority = 2, description = "Test error handling - missing required field")
    public void testErrorHandlingMissingField() {
        String invalidRequestBody = """
            {
                "email": "test@example.com",
                "password": "Pass123"
                // Missing firstName and lastName
            }
            """;
        
        given()
            .header("Content-Type", "application/json")
            .header("Authorization", "Bearer " + authToken)
            .body(invalidRequestBody)
        .when()
            .post("/users")
        .then()
            // Error Status Code Check
            .statusCode(400)
            
            // Error Message Check
            .body("error", notNullValue())
            .body("error.message", containsString("required"))
            .body("error.code", equalTo("VALIDATION_ERROR"))
            
            // Error Format Check
            .body("error.field", hasItems("firstName", "lastName"));
    }
    
    @Test(priority = 3, description = "Test authentication - invalid token")
    public void testAuthenticationInvalidToken() {
        given()
            .header("Content-Type", "application/json")
            .header("Authorization", "Bearer invalid_token")
            .body("{}")
        .when()
            .post("/users")
        .then()
            .statusCode(401)
            .body("error.message", containsString("Unauthorized"));
    }
    
    @Test(priority = 4, description = "Test authorization - insufficient permissions")
    public void testAuthorization() {
        String userToken = getRegularUserToken(); // Non-admin token
        
        given()
            .header("Content-Type", "application/json")
            .header("Authorization", "Bearer " + userToken)
        .when()
            .delete("/users/admin123") // Regular user trying to delete admin
        .then()
            .statusCode(403)
            .body("error.message", containsString("Forbidden"));
    }
    
    @Test(priority = 5, description = "Test input validation - invalid email format")
    public void testInputValidation() {
        String invalidRequestBody = """
            {
                "email": "invalid-email-format",
                "password": "Pass123",
                "firstName": "John",
                "lastName": "Doe"
            }
            """;
        
        given()
            .header("Content-Type", "application/json")
            .body(invalidRequestBody)
        .when()
            .post("/users")
        .then()
            .statusCode(400)
            .body("error.field", equalTo("email"))
            .body("error.message", containsString("invalid format"));
    }
    
    @Test(priority = 6, description = "Test security - SQL injection attempt")
    public void testSQLInjection() {
        String maliciousRequestBody = """
            {
                "email": "test@example.com'; DROP TABLE users; --",
                "password": "Pass123",
                "firstName": "John",
                "lastName": "Doe"
            }
            """;
        
        given()
            .header("Content-Type", "application/json")
            .body(maliciousRequestBody)
        .when()
            .post("/users")
        .then()
            .statusCode(400) // Should reject, not execute SQL
            .body("error.message", not(containsString("SQL")));
        
        // Verify database integrity (table still exists)
        verifyDatabaseIntegrity();
    }
    
    @Test(priority = 7, description = "Test performance - response time")
    public void testPerformance() {
        long startTime = System.currentTimeMillis();
        
        given()
            .header("Authorization", "Bearer " + authToken)
        .when()
            .get("/users")
        .then()
            .statusCode(200)
            .time(lessThan(500L)); // Should respond in < 500ms
        
        long endTime = System.currentTimeMillis();
        long responseTime = endTime - startTime;
        
        assert responseTime < 500 : "Response time should be less than 500ms";
    }
    
    @Test(priority = 8, description = "Test idempotency - PUT request")
    public void testIdempotency() {
        String userId = "user123";
        String requestBody = """
            {
                "firstName": "Updated",
                "lastName": "Name"
            }
            """;
        
        // First PUT request
        Response response1 = given()
            .header("Authorization", "Bearer " + authToken)
            .body(requestBody)
        .when()
            .put("/users/" + userId)
        .then()
            .statusCode(200)
            .extract().response();
        
        // Second PUT request with same data (idempotent)
        Response response2 = given()
            .header("Authorization", "Bearer " + authToken)
            .body(requestBody)
        .when()
            .put("/users/" + userId)
        .then()
            .statusCode(200)
            .extract().response();
        
        // Verify same result
        assert response1.jsonPath().getString("updatedAt")
            .equals(response2.jsonPath().getString("updatedAt"));
    }
    
    // Helper methods
    private String getAuthToken() {
        // Implementation to get auth token
        return "valid_token_here";
    }
    
    private String getRegularUserToken() {
        // Implementation to get regular user token
        return "regular_user_token";
    }
    
    private void verifyUserInDatabase(String userId) {
        // Database verification logic
    }
    
    private void verifyDatabaseIntegrity() {
        // Verify database tables exist
    }
}
```

#### **Example 2: Postman Test Script with All Checks**

```javascript
// Postman Test Script - Comprehensive API Checks

// 1. Status Code Check
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// 2. Response Time Check
pm.test("Response time is less than 500ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(500);
});

// 3. Response Header Checks
pm.test("Content-Type is application/json", function () {
    pm.expect(pm.response.headers.get("Content-Type")).to.include("application/json");
});

pm.test("CORS headers present", function () {
    pm.expect(pm.response.headers.has("Access-Control-Allow-Origin")).to.be.true;
});

// 4. Response Body Schema Validation
pm.test("Response has required fields", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property("id");
    pm.expect(jsonData).to.have.property("name");
    pm.expect(jsonData).to.have.property("email");
});

// 5. Data Type Validation
pm.test("Data types are correct", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.id).to.be.a('number');
    pm.expect(jsonData.name).to.be.a('string');
    pm.expect(jsonData.email).to.be.a('string');
    pm.expect(jsonData.active).to.be.a('boolean');
});

// 6. Data Value Validation
pm.test("Email format is valid", function () {
    var jsonData = pm.response.json();
    var emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    pm.expect(emailRegex.test(jsonData.email)).to.be.true;
});

// 7. Array Validation
pm.test("Items array is not empty", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.items).to.be.an('array');
    pm.expect(jsonData.items.length).to.be.above(0);
});

// 8. Error Handling Check (for error scenarios)
if (pm.response.code >= 400) {
    pm.test("Error response has proper structure", function () {
        var jsonData = pm.response.json();
        pm.expect(jsonData).to.have.property("error");
        pm.expect(jsonData.error).to.have.property("message");
        pm.expect(jsonData.error).to.have.property("code");
    });
}

// 9. Security Checks
pm.test("No sensitive data in response", function () {
    var responseText = pm.response.text();
    pm.expect(responseText).to.not.include("password");
    pm.expect(responseText).to.not.include("creditCard");
    pm.expect(responseText).to.not.include("ssn");
});

// 10. Business Logic Validation
pm.test("Total matches sum of items", function () {
    var jsonData = pm.response.json();
    var calculatedTotal = jsonData.items.reduce((sum, item) => sum + item.price, 0);
    pm.expect(jsonData.total).to.eql(calculatedTotal);
});

// 11. Store values for subsequent requests
if (pm.response.code === 200) {
    var jsonData = pm.response.json();
    pm.environment.set("userId", jsonData.id);
    pm.environment.set("authToken", jsonData.token);
}
```

### Scenario-Based Example

**Real Scenario: Testing Payment API**

**Comprehensive Test Checklist:**

```java
@Test
public void testPaymentAPIComprehensive() {
    // 1. FUNCTIONAL CHECKS
    // - Valid payment processed successfully
    // - Payment amount matches request
    // - Payment status updated correctly
    
    // 2. DATA VALIDATION
    // - Invalid card number rejected (400)
    // - Expired card rejected (400)
    // - Invalid CVV rejected (400)
    // - Amount validation (negative, zero, too large)
    
    // 3. ERROR HANDLING
    // - Network timeout handled (504)
    // - Payment gateway error handled (502)
    // - Insufficient funds (402)
    // - Card declined (402)
    
    // 4. AUTHENTICATION
    // - Valid token processes payment
    // - Invalid token rejected (401)
    // - Expired token rejected (401)
    
    // 5. AUTHORIZATION
    // - User can only pay for their own orders
    // - Admin can process refunds
    // - Regular user cannot refund (403)
    
    // 6. SECURITY
    // - Card number encrypted in request/response
    // - No card details in logs
    // - SQL injection attempts blocked
    // - XSS attempts sanitized
    
    // 7. PERFORMANCE
    // - Payment processes in < 2 seconds
    // - Handles 100 concurrent payments
    // - No memory leaks
    
    // 8. INTEGRATION
    // - Payment recorded in database
    // - Order status updated
    // - Email notification sent
    // - Webhook triggered
    
    // 9. BUSINESS LOGIC
    // - Payment amount matches order total
    // - Refund amount cannot exceed payment
    // - Partial refunds handled correctly
    // - Currency conversion accurate
}
```

### API Testing Checklist Summary

| Category | Key Checks |
|----------|-----------|
| **Functional** | Status codes, Response structure, Data accuracy |
| **Data Validation** | Required fields, Data types, Format validation, Boundary values |
| **Error Handling** | Error messages, Error codes, 4xx/5xx handling |
| **Authentication** | Token validation, Expired tokens, Missing auth |
| **Authorization** | Role-based access, Permission checks |
| **Performance** | Response time, Throughput, Concurrent requests |
| **Security** | SQL injection, XSS, CSRF, Data encryption, Rate limiting |
| **Integration** | Database, External services, Webhooks |
| **Business Logic** | Workflow validation, Calculations, State transitions |

### Best Practices

1. **Start with happy path** - Verify basic functionality first
2. **Test negative scenarios** - More important than positive cases
3. **Validate response schema** - Use JSON schema validation
4. **Check error handling** - Ensure proper error responses
5. **Test security** - Authentication, authorization, injection attacks
6. **Monitor performance** - Set response time thresholds
7. **Test edge cases** - Boundary values, null, empty, special characters
8. **Verify data integrity** - Check database after API calls
9. **Test idempotency** - PUT/DELETE should be idempotent
10. **Document test cases** - Clear test documentation

---

## 10. If 49 out of 200 tests fail, how will you get only the failed test details in TestNG?

### Context
This question tests your knowledge of TestNG reporting features, result analysis, and debugging strategies. When dealing with large test suites, efficiently identifying and analyzing failures is crucial.

### Detailed Answer

TestNG provides multiple ways to extract and analyze failed test details. Here are the most effective approaches:

### Method 1: Using TestNG HTML Reports (Built-in)

**Location:** After test execution, TestNG generates HTML reports in `test-output` folder.

**Files to Check:**
- `test-output/index.html` - Main report with summary
- `test-output/emailable-report.html` - Email-friendly report
- `test-output/testng-results.xml` - XML results

**Steps:**
1. Run tests: `mvn test` or run TestNG suite
2. Navigate to `test-output/index.html`
3. Click on **"Failed tests"** section
4. View detailed failure information

### Method 2: Using TestNG Listeners to Extract Failed Tests

#### **Custom Listener to Collect Failed Tests**

```java
package listeners;

import org.testng.ITestListener;
import org.testng.ITestResult;
import java.io.FileWriter;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

public class FailedTestCollector implements ITestListener {
    private List<FailedTestInfo> failedTests = new ArrayList<>();
    
    @Override
    public void onTestFailure(ITestResult result) {
        FailedTestInfo failedTest = new FailedTestInfo(
            result.getTestClass().getName(),
            result.getMethod().getMethodName(),
            result.getThrowable().getMessage(),
            result.getThrowable().getStackTrace()
        );
        failedTests.add(failedTest);
    }
    
    @Override
    public void onFinish(org.testng.ITestContext context) {
        // Write failed tests to file
        writeFailedTestsToFile();
        
        // Print summary
        System.out.println("\n========== FAILED TESTS SUMMARY ==========");
        System.out.println("Total Failed Tests: " + failedTests.size());
        System.out.println("\nFailed Test Details:");
        
        for (int i = 0; i < failedTests.size(); i++) {
            FailedTestInfo test = failedTests.get(i);
            System.out.println("\n" + (i + 1) + ". " + test.getClassName() + "." + test.getMethodName());
            System.out.println("   Error: " + test.getErrorMessage());
        }
    }
    
    private void writeFailedTestsToFile() {
        try (FileWriter writer = new FileWriter("failed_tests.txt")) {
            writer.write("FAILED TESTS REPORT\n");
            writer.write("==================\n\n");
            
            for (FailedTestInfo test : failedTests) {
                writer.write("Class: " + test.getClassName() + "\n");
                writer.write("Method: " + test.getMethodName() + "\n");
                writer.write("Error: " + test.getErrorMessage() + "\n");
                writer.write("Stack Trace:\n");
                for (StackTraceElement element : test.getStackTrace()) {
                    writer.write("  " + element.toString() + "\n");
                }
                writer.write("\n" + "=".repeat(50) + "\n\n");
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

// Helper class to store failed test information
class FailedTestInfo {
    private String className;
    private String methodName;
    private String errorMessage;
    private StackTraceElement[] stackTrace;
    
    public FailedTestInfo(String className, String methodName, 
                         String errorMessage, StackTraceElement[] stackTrace) {
        this.className = className;
        this.methodName = methodName;
        this.errorMessage = errorMessage;
        this.stackTrace = stackTrace;
    }
    
    // Getters
    public String getClassName() { return className; }
    public String getMethodName() { return methodName; }
    public String getErrorMessage() { return errorMessage; }
    public StackTraceElement[] getStackTrace() { return stackTrace; }
}
```

**Register Listener in testng.xml:**
```xml
<suite name="Test Suite">
    <listeners>
        <listener class-name="listeners.FailedTestCollector"/>
    </listeners>
    <test name="Tests">
        <classes>
            <class name="tests.*"/>
        </classes>
    </test>
</suite>
```

### Method 3: Programmatically Extract Failed Tests from TestNG Results

```java
package utils;

import org.testng.ITestResult;
import org.testng.TestListenerAdapter;
import java.util.List;

public class FailedTestExtractor extends TestListenerAdapter {
    
    @Override
    public void onFinish(org.testng.ITestContext context) {
        List<ITestResult> failedTests = context.getFailedTests().getAllResults();
        
        System.out.println("\n========== FAILED TESTS DETAILS ==========");
        System.out.println("Total Failed: " + failedTests.size());
        System.out.println("Total Passed: " + context.getPassedTests().size());
        System.out.println("Total Skipped: " + context.getSkippedTests().size());
        System.out.println("\nFailed Test List:\n");
        
        for (int i = 0; i < failedTests.size(); i++) {
            ITestResult result = failedTests.get(i);
            System.out.println((i + 1) + ". " + 
                result.getTestClass().getName() + "." + 
                result.getMethod().getMethodName());
            System.out.println("   Status: " + getStatus(result.getStatus()));
            System.out.println("   Error: " + result.getThrowable().getMessage());
            System.out.println("   Duration: " + (result.getEndMillis() - result.getStartMillis()) + "ms");
            System.out.println();
        }
        
        // Generate retry XML for failed tests only
        generateRetryXML(failedTests);
    }
    
    private String getStatus(int status) {
        switch (status) {
            case ITestResult.SUCCESS: return "PASS";
            case ITestResult.FAILURE: return "FAIL";
            case ITestResult.SKIP: return "SKIP";
            default: return "UNKNOWN";
        }
    }
    
    private void generateRetryXML(List<ITestResult> failedTests) {
        // Generate TestNG XML with only failed tests for retry
        StringBuilder xml = new StringBuilder();
        xml.append("<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n");
        xml.append("<!DOCTYPE suite SYSTEM \"https://testng.org/testng-1.0.dtd\">\n");
        xml.append("<suite name=\"Failed Tests Retry Suite\">\n");
        xml.append("  <test name=\"Retry Failed Tests\">\n");
        xml.append("    <classes>\n");
        
        String currentClass = "";
        for (ITestResult result : failedTests) {
            String className = result.getTestClass().getName();
            if (!className.equals(currentClass)) {
                if (!currentClass.isEmpty()) {
                    xml.append("    </class>\n");
                }
                xml.append("    <class name=\"").append(className).append("\">\n");
                xml.append("      <methods>\n");
                currentClass = className;
            }
            xml.append("        <include name=\"").append(result.getMethod().getMethodName()).append("\"/>\n");
        }
        if (!currentClass.isEmpty()) {
            xml.append("      </methods>\n");
            xml.append("    </class>\n");
        }
        xml.append("    </classes>\n");
        xml.append("  </test>\n");
        xml.append("</suite>\n");
        
        // Write to file
        try (java.io.FileWriter writer = new java.io.FileWriter("failed_tests_retry.xml")) {
            writer.write(xml.toString());
            System.out.println("Retry XML generated: failed_tests_retry.xml");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### Method 4: Using TestNG's IReporter Interface

```java
package listeners;

import org.testng.IReporter;
import org.testng.ISuite;
import org.testng.ISuiteResult;
import org.testng.ITestContext;
import org.testng.ITestResult;
import org.testng.xml.XmlSuite;

import java.io.FileWriter;
import java.io.IOException;
import java.util.List;
import java.util.Map;

public class FailedTestReporter implements IReporter {
    
    @Override
    public void generateReport(List<XmlSuite> xmlSuites, List<ISuite> suites,
                              String outputDirectory) {
        
        StringBuilder report = new StringBuilder();
        report.append("FAILED TESTS REPORT\n");
        report.append("==================\n\n");
        
        int totalFailed = 0;
        
        for (ISuite suite : suites) {
            Map<String, ISuiteResult> suiteResults = suite.getResults();
            
            for (ISuiteResult suiteResult : suiteResults.values()) {
                ITestContext testContext = suiteResult.getTestContext();
                
                // Get failed tests
                List<ITestResult> failedTests = testContext.getFailedTests().getAllResults();
                totalFailed += failedTests.size();
                
                report.append("Suite: ").append(suite.getName()).append("\n");
                report.append("Failed Tests: ").append(failedTests.size()).append("\n\n");
                
                for (ITestResult result : failedTests) {
                    report.append("Test: ").append(result.getTestClass().getName())
                          .append(".").append(result.getMethod().getMethodName()).append("\n");
                    report.append("Status: FAILED\n");
                    report.append("Error: ").append(result.getThrowable().getMessage()).append("\n");
                    
                    if (result.getThrowable().getStackTrace() != null) {
                        report.append("Stack Trace:\n");
                        for (StackTraceElement element : result.getThrowable().getStackTrace()) {
                            report.append("  ").append(element.toString()).append("\n");
                        }
                    }
                    report.append("\n").append("-".repeat(50)).append("\n\n");
                }
            }
        }
        
        report.append("\nTOTAL FAILED TESTS: ").append(totalFailed).append("\n");
        
        // Write to file
        try (FileWriter writer = new FileWriter(outputDirectory + "/failed_tests_report.txt")) {
            writer.write(report.toString());
            System.out.println("Failed tests report generated: " + outputDirectory + "/failed_tests_report.txt");
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        // Print to console
        System.out.println(report.toString());
    }
}
```

**Register in testng.xml:**
```xml
<suite name="Test Suite">
    <listeners>
        <listener class-name="listeners.FailedTestReporter"/>
    </listeners>
    <!-- test classes -->
</suite>
```

### Method 5: Using Maven Surefire Plugin to Filter Failed Tests

**pom.xml Configuration:**
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.0.0</version>
    <configuration>
        <includes>
            <!-- Run all tests first -->
        </includes>
        <reportFormat>xml</reportFormat>
    </configuration>
</plugin>
```

**Extract Failed Tests Script:**
```bash
#!/bin/bash
# Extract failed test names from TestNG XML results

grep -oP '(?<=classname=")[^"]*' test-output/testng-results.xml | \
grep -A1 "status=\"FAIL\"" | \
awk '/classname/ {class=$0} /methodname/ {print class"."$0}' > failed_tests.txt

echo "Failed tests extracted to failed_tests.txt"
```

### Method 6: Using TestNG's TestRunner API

```java
package utils;

import org.testng.TestNG;
import org.testng.xml.XmlSuite;
import org.testng.xml.XmlTest;
import org.testng.xml.XmlClass;
import org.testng.ITestResult;
import org.testng.ITestListener;
import org.testng.ITestContext;

import java.util.ArrayList;
import java.util.List;

public class FailedTestRunner {
    
    public static void main(String[] args) {
        // First run - get all results
        TestNG testng = new TestNG();
        testng.setTestClasses(new Class[] {
            tests.LoginTest.class,
            tests.CheckoutTest.class,
            // ... all test classes
        });
        
        FailedTestListener listener = new FailedTestListener();
        testng.addListener(listener);
        testng.run();
        
        // Get failed test methods
        List<String> failedTests = listener.getFailedTestMethods();
        
        System.out.println("Failed Tests (" + failedTests.size() + "):");
        failedTests.forEach(System.out::println);
        
        // Optionally, rerun only failed tests
        rerunFailedTests(failedTests);
    }
    
    private static void rerunFailedTests(List<String> failedTestMethods) {
        // Implementation to rerun specific failed tests
    }
}

class FailedTestListener implements ITestListener {
    private List<String> failedTests = new ArrayList<>();
    
    @Override
    public void onTestFailure(ITestResult result) {
        String testName = result.getTestClass().getName() + "." + 
                         result.getMethod().getMethodName();
        failedTests.add(testName);
    }
    
    public List<String> getFailedTestMethods() {
        return failedTests;
    }
}
```

### Method 7: Using ExtentReports or Allure for Better Visualization

#### **ExtentReports Integration:**

```java
package listeners;

import com.aventstack.extentreports.ExtentReports;
import com.aventstack.extentreports.ExtentTest;
import com.aventstack.extentreports.Status;
import org.testng.ITestListener;
import org.testng.ITestResult;

public class ExtentFailedTestListener implements ITestListener {
    private ExtentReports extent = new ExtentReports();
    private ExtentTest test;
    
    @Override
    public void onTestFailure(ITestResult result) {
        test.log(Status.FAIL, "Test Failed: " + result.getMethod().getMethodName());
        test.log(Status.FAIL, result.getThrowable());
        
        // Screenshot attachment if available
        // test.addScreenCaptureFromPath(screenshotPath);
    }
    
    @Override
    public void onFinish(org.testng.ITestContext context) {
        extent.flush();
        
        // Extract failed tests summary
        System.out.println("Failed Tests: " + context.getFailedTests().size());
        context.getFailedTests().getAllResults().forEach(result -> {
            System.out.println("- " + result.getTestClass().getName() + 
                             "." + result.getMethod().getMethodName());
        });
    }
}
```

### Scenario-Based Example

**Real Scenario: 49 out of 200 tests failed in CI pipeline**

**Problem:** Need to quickly identify and analyze failures without going through all 200 test results.

**Solution:**

```java
package utils;

import org.testng.ITestContext;
import org.testng.ITestResult;
import org.testng.TestListenerAdapter;

import java.io.FileWriter;
import java.io.IOException;
import java.util.*;

public class CIFailedTestAnalyzer extends TestListenerAdapter {
    
    @Override
    public void onFinish(ITestContext context) {
        // Get failed tests
        Set<ITestResult> failedTests = context.getFailedTests().getAllResults();
        
        // Analyze failures
        Map<String, Integer> failureByClass = new HashMap<>();
        Map<String, Integer> failureByException = new HashMap<>();
        
        for (ITestResult result : failedTests) {
            // Group by class
            String className = result.getTestClass().getName();
            failureByClass.put(className, 
                failureByClass.getOrDefault(className, 0) + 1);
            
            // Group by exception type
            String exceptionType = result.getThrowable().getClass().getSimpleName();
            failureByException.put(exceptionType, 
                failureByException.getOrDefault(exceptionType, 0) + 1);
        }
        
        // Generate comprehensive report
        generateFailureReport(context, failedTests, failureByClass, failureByException);
        
        // Send notification (email, Slack, etc.)
        sendFailureNotification(context, failedTests.size());
    }
    
    private void generateFailureReport(ITestContext context, 
                                      Set<ITestResult> failedTests,
                                      Map<String, Integer> failureByClass,
                                      Map<String, Integer> failureByException) {
        
        StringBuilder report = new StringBuilder();
        report.append("TEST EXECUTION FAILURE REPORT\n");
        report.append("=".repeat(50)).append("\n\n");
        
        report.append("SUMMARY\n");
        report.append("-".repeat(50)).append("\n");
        report.append("Total Tests: ").append(context.getAllTestMethods().length).append("\n");
        report.append("Passed: ").append(context.getPassedTests().size()).append("\n");
        report.append("Failed: ").append(failedTests.size()).append("\n");
        report.append("Skipped: ").append(context.getSkippedTests().size()).append("\n");
        report.append("Success Rate: ").append(
            String.format("%.2f%%", 
                (context.getPassedTests().size() * 100.0 / context.getAllTestMethods().length))
        ).append("\n\n");
        
        report.append("FAILURES BY CLASS\n");
        report.append("-".repeat(50)).append("\n");
        failureByClass.entrySet().stream()
            .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
            .forEach(entry -> 
                report.append(entry.getKey()).append(": ").append(entry.getValue()).append("\n")
            );
        report.append("\n");
        
        report.append("FAILURES BY EXCEPTION TYPE\n");
        report.append("-".repeat(50)).append("\n");
        failureByException.entrySet().stream()
            .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
            .forEach(entry -> 
                report.append(entry.getKey()).append(": ").append(entry.getValue()).append("\n")
            );
        report.append("\n");
        
        report.append("DETAILED FAILURE LIST\n");
        report.append("-".repeat(50)).append("\n");
        int index = 1;
        for (ITestResult result : failedTests) {
            report.append(index++).append(". ")
                  .append(result.getTestClass().getName())
                  .append(".")
                  .append(result.getMethod().getMethodName())
                  .append("\n");
            report.append("   Error: ").append(result.getThrowable().getMessage()).append("\n");
            report.append("   Duration: ").append(
                result.getEndMillis() - result.getStartMillis()
            ).append("ms\n\n");
        }
        
        // Write to file
        try (FileWriter writer = new FileWriter("failure_report.txt")) {
            writer.write(report.toString());
            System.out.println("Failure report generated: failure_report.txt");
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        // Print summary to console
        System.out.println("\n" + report.toString());
    }
    
    private void sendFailureNotification(ITestContext context, int failureCount) {
        // Implementation to send email/Slack notification
        System.out.println("Sending failure notification: " + failureCount + " tests failed");
    }
}
```

**Output Example:**
```
TEST EXECUTION FAILURE REPORT
==================================================

SUMMARY
--------------------------------------------------
Total Tests: 200
Passed: 151
Failed: 49
Skipped: 0
Success Rate: 75.50%

FAILURES BY CLASS
--------------------------------------------------
tests.LoginTest: 15
tests.CheckoutTest: 12
tests.PaymentTest: 10
tests.OrderTest: 8
tests.ProfileTest: 4

FAILURES BY EXCEPTION TYPE
--------------------------------------------------
AssertionError: 25
TimeoutException: 12
NoSuchElementException: 8
NullPointerException: 4

DETAILED FAILURE LIST
--------------------------------------------------
1. tests.LoginTest.testInvalidLogin
   Error: Expected true but found false
   Duration: 1234ms

2. tests.CheckoutTest.testAddToCart
   Error: Element not found
   Duration: 567ms
...
```

### Best Practices

1. **Use TestNG listeners** - Automatic collection of failed tests
2. **Generate retry XML** - Automatically create XML to rerun only failed tests
3. **Group failures** - Analyze by class, exception type, duration
4. **Generate reports** - Create detailed failure reports
5. **Integrate with CI/CD** - Send notifications on failures
6. **Store failure details** - Save to files for later analysis
7. **Use reporting tools** - ExtentReports, Allure for better visualization
8. **Automate retry** - Automatically rerun failed tests

---

## 11. What is the use of the Maven Surefire plugin?

### Context
This question tests your understanding of Maven build lifecycle, test execution, and how Maven integrates with testing frameworks. The Surefire plugin is essential for running tests in Maven projects.

### Detailed Answer

The **Maven Surefire Plugin** is responsible for executing unit tests during the Maven test phase. It's the default test execution plugin that runs tests and generates reports.

### Primary Uses

1. **Test Execution:** Runs tests during `mvn test` or `mvn verify` phases
2. **Test Reporting:** Generates test execution reports (XML and HTML)
3. **Test Filtering:** Allows running specific tests, classes, or packages
4. **Parallel Execution:** Supports parallel test execution
5. **Test Integration:** Integrates with JUnit, TestNG, and other testing frameworks

### Key Features

- **Automatic Test Discovery:** Finds and runs tests automatically
- **Multiple Framework Support:** JUnit 3/4/5, TestNG, etc.
- **Report Generation:** Creates test reports in `target/surefire-reports`
- **Configuration Flexibility:** Extensive configuration options
- **CI/CD Integration:** Works seamlessly with CI/CD pipelines

### Basic Configuration

#### **Default pom.xml (No Explicit Configuration Needed)**

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>test-project</artifactId>
    <version>1.0.0</version>
    
    <dependencies>
        <dependency>
            <groupId>org.testng</groupId>
            <artifactId>testng</artifactId>
            <version>7.8.0</version>
        </dependency>
    </dependencies>
    
    <!-- Surefire plugin is included by default -->
    <!-- No explicit configuration needed for basic usage -->
</project>
```

**Default Behavior:**
- Runs tests matching: `**/Test*.java`, `**/*Test.java`, `**/*Tests.java`, `**/*TestCase.java`
- Generates reports in `target/surefire-reports`
- Executes during `mvn test` phase

### Advanced Configuration Examples

#### **Example 1: Basic Configuration**

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.0.0</version>
            <configuration>
                <!-- Include specific test patterns -->
                <includes>
                    <include>**/*Test.java</include>
                    <include>**/*Tests.java</include>
                </includes>
                
                <!-- Exclude specific test patterns -->
                <excludes>
                    <exclude>**/*IntegrationTest.java</exclude>
                </excludes>
                
                <!-- Test output directory -->
                <reportsDirectory>${project.build.directory}/surefire-reports</reportsDirectory>
            </configuration>
        </plugin>
    </plugins>
</build>
```

#### **Example 2: TestNG Configuration**

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.0.0</version>
    <configuration>
        <!-- Use TestNG suite file -->
        <suiteXmlFiles>
            <suiteXmlFile>src/test/resources/testng.xml</suiteXmlFile>
        </suiteXmlFiles>
        
        <!-- Or use TestNG groups -->
        <groups>regression,smoke</groups>
        <excludedGroups>slow</excludedGroups>
        
        <!-- Parallel execution -->
        <parallel>methods</parallel>
        <threadCount>4</threadCount>
        
        <!-- Test timeout -->
        <testFailureIgnore>false</testFailureIgnore>
    </configuration>
</plugin>
```

#### **Example 3: Running Specific Tests**

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.0.0</version>
    <configuration>
        <!-- Run specific test class -->
        <includes>
            <include>**/LoginTest.java</include>
        </includes>
        
        <!-- Or run specific test method -->
        <!-- Use command line: mvn test -Dtest=LoginTest#testLogin -->
    </configuration>
</plugin>
```

**Command Line Usage:**
```bash
# Run specific test class
mvn test -Dtest=LoginTest

# Run specific test method
mvn test -Dtest=LoginTest#testLogin

# Run multiple test classes
mvn test -Dtest=LoginTest,CheckoutTest

# Run tests matching pattern
mvn test -Dtest=*Test

# Run tests in specific package
mvn test -Dtest=com.example.tests.*
```

#### **Example 4: Parallel Execution**

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.0.0</version>
    <configuration>
        <!-- Parallel execution options: methods, classes, both, suites -->
        <parallel>methods</parallel>
        
        <!-- Number of threads -->
        <threadCount>4</threadCount>
        
        <!-- Use all available processors -->
        <useUnlimitedThreads>true</useUnlimitedThreads>
        
        <!-- PerCoreThreadCount: threads per CPU core -->
        <perCoreThreadCount>true</perCoreThreadCount>
        
        <!-- Thread count multiplier -->
        <threadCountSuites>1</threadCountSuites>
        <threadCountClasses>1</threadCountClasses>
        <threadCountMethods>4</threadCountMethods>
    </configuration>
</plugin>
```

#### **Example 5: Skipping Tests**

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.0.0</version>
    <configuration>
        <!-- Skip tests (not recommended for production) -->
        <skipTests>false</skipTests>
        
        <!-- Skip test execution but compile them -->
        <skip>false</skip>
    </configuration>
</plugin>
```

**Command Line:**
```bash
# Skip tests
mvn install -DskipTests

# Skip test compilation and execution
mvn install -Dmaven.test.skip=true
```

#### **Example 6: Test Timeout and Failure Handling**

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.0.0</version>
    <configuration>
        <!-- Test timeout in seconds -->
        <testFailureIgnore>false</testFailureIgnore>
        
        <!-- Stop after first failure -->
        <skipAfterFailureCount>1</skipAfterFailureCount>
        
        <!-- Rerun failed tests -->
        <rerunFailingTestsCount>2</rerunFailingTestsCount>
        
        <!-- Fork options for JVM -->
        <forkCount>1</forkCount>
        <reuseForks>true</reuseForks>
    </configuration>
</plugin>
```

#### **Example 7: System Properties and Environment Variables**

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.0.0</version>
    <configuration>
        <!-- System properties for tests -->
        <systemPropertyVariables>
            <browser>chrome</browser>
            <environment>qa</environment>
            <baseURL>https://qa.example.com</baseURL>
        </systemPropertyVariables>
        
        <!-- Environment variables -->
        <environmentVariables>
            <JAVA_HOME>${java.home}</JAVA_HOME>
            <TEST_ENV>qa</TEST_ENV>
        </environmentVariables>
    </configuration>
</plugin>
```

#### **Example 8: Report Configuration**

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.0.0</version>
    <configuration>
        <!-- Report format -->
        <reportFormat>xml</reportFormat>
        
        <!-- Generate HTML reports -->
        <reportNameSuffix>test-report</reportNameSuffix>
        
        <!-- Print summary -->
        <printSummary>true</printSummary>
        
        <!-- Show standard output -->
        <redirectTestOutputToFile>false</redirectTestOutputToFile>
    </configuration>
</plugin>
```

### Complete Example: Production-Ready Configuration

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.0.0</version>
            <configuration>
                <!-- TestNG Configuration -->
                <suiteXmlFiles>
                    <suiteXmlFile>src/test/resources/testng.xml</suiteXmlFile>
                </suiteXmlFiles>
                
                <!-- Parallel Execution -->
                <parallel>methods</parallel>
                <threadCount>4</threadCount>
                
                <!-- Test Patterns -->
                <includes>
                    <include>**/*Test.java</include>
                </includes>
                <excludes>
                    <exclude>**/*IntegrationTest.java</exclude>
                    <exclude>**/*SlowTest.java</exclude>
                </excludes>
                
                <!-- System Properties -->
                <systemPropertyVariables>
                    <browser>${browser}</browser>
                    <environment>${environment}</environment>
                </systemPropertyVariables>
                
                <!-- Failure Handling -->
                <testFailureIgnore>false</testFailureIgnore>
                <rerunFailingTestsCount>2</rerunFailingTestsCount>
                
                <!-- Reporting -->
                <reportFormat>xml</reportFormat>
                <printSummary>true</printSummary>
                
                <!-- JVM Options -->
                <argLine>
                    -Xmx1024m
                    -XX:MaxPermSize=256m
                    -Dfile.encoding=UTF-8
                </argLine>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### Maven Lifecycle Integration

```
mvn clean
    ↓
mvn compile
    ↓
mvn test-compile
    ↓
mvn test  ← Surefire plugin executes here
    ↓
mvn package
    ↓
mvn verify  ← Can also run tests here
    ↓
mvn install
```

**Key Phases:**
- **test-compile:** Compiles test source code
- **test:** Surefire plugin runs tests
- **verify:** Can run integration tests (Failsafe plugin)

### Surefire vs Failsafe Plugin

| Aspect | Surefire Plugin | Failsafe Plugin |
|--------|----------------|-----------------|
| **Purpose** | Unit tests | Integration tests |
| **Phase** | `test` phase | `verify` phase |
| **Failure Behavior** | Fails build | Continues build |
| **Naming Convention** | `*Test.java` | `*IT.java`, `*IntegrationTest.java` |
| **Use Case** | Fast, isolated tests | Slow, integration tests |

### Scenario-Based Example

**Real Scenario: CI/CD Pipeline Configuration**

**Requirement:** Run tests in CI pipeline with specific configuration for different environments.

**Solution:**

```xml
<profiles>
    <!-- Development Profile -->
    <profile>
        <id>dev</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <version>3.0.0</version>
                    <configuration>
                        <includes>
                            <include>**/*Test.java</include>
                        </includes>
                        <parallel>methods</parallel>
                        <threadCount>2</threadCount>
                        <systemPropertyVariables>
                            <environment>dev</environment>
                            <browser>chrome</browser>
                        </systemPropertyVariables>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </profile>
    
    <!-- CI Profile -->
    <profile>
        <id>ci</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <version>3.0.0</version>
                    <configuration>
                        <suiteXmlFiles>
                            <suiteXmlFile>src/test/resources/testng-ci.xml</suiteXmlFile>
                        </suiteXmlFiles>
                        <parallel>methods</parallel>
                        <threadCount>4</threadCount>
                        <testFailureIgnore>false</testFailureIgnore>
                        <systemPropertyVariables>
                            <environment>ci</environment>
                            <browser>headless</browser>
                        </systemPropertyVariables>
                        <argLine>
                            -Xmx2048m
                            -Dfile.encoding=UTF-8
                        </argLine>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```

**Usage:**
```bash
# Run with dev profile
mvn test -Pdev

# Run with CI profile
mvn test -Pci

# Run in CI pipeline
mvn clean test -Pci
```

### Common Use Cases

1. **Running Specific Tests:**
   ```bash
   mvn test -Dtest=LoginTest
   ```

2. **Running Tests in Parallel:**
   ```xml
   <parallel>methods</parallel>
   <threadCount>4</threadCount>
   ```

3. **Skipping Tests:**
   ```bash
   mvn install -DskipTests
   ```

4. **Passing System Properties:**
   ```bash
   mvn test -Dbrowser=chrome -Denvironment=qa
   ```

5. **Using TestNG Suite:**
   ```xml
   <suiteXmlFiles>
       <suiteXmlFile>testng.xml</suiteXmlFile>
   </suiteXmlFiles>
   ```

### Best Practices

1. **Use appropriate version** - Keep Surefire plugin updated
2. **Configure parallel execution** - Speed up test execution
3. **Exclude integration tests** - Use Failsafe for integration tests
4. **Set proper timeouts** - Prevent hanging tests
5. **Use profiles** - Different configurations for different environments
6. **Configure JVM options** - Set memory and other JVM parameters
7. **Generate reports** - Ensure reports are generated for analysis
8. **Don't skip tests in CI** - Always run tests in CI/CD pipeline

### Summary

The Maven Surefire Plugin is essential for:
- ✅ **Executing tests** during Maven build lifecycle
- ✅ **Generating test reports** for analysis
- ✅ **Filtering tests** (include/exclude patterns)
- ✅ **Parallel execution** for faster test runs
- ✅ **CI/CD integration** for automated testing
- ✅ **Framework support** (JUnit, TestNG, etc.)
- ✅ **Configuration flexibility** for different scenarios

---

# Round 2 – Technical + Coding Interview Answers

---

## 1. How do you decide which test cases should be automated or not?

### Context
This question evaluates your decision-making process, understanding of automation ROI, and ability to prioritize test automation efforts. As an SDET, you need to make strategic decisions about what to automate.

### Detailed Answer

Deciding which test cases to automate requires a strategic approach considering multiple factors. Not all tests should be automated - some are better suited for manual testing.

### Decision Framework

#### **Factors to Consider**

1. **Test Frequency**
   - **Automate:** Tests run frequently (daily, every build, regression)
   - **Manual:** One-time tests, exploratory testing

2. **Test Stability**
   - **Automate:** Stable functionality, well-defined requirements
   - **Manual:** Frequently changing features, unstable UI

3. **Test Complexity**
   - **Automate:** Repetitive, rule-based tests
   - **Manual:** Complex scenarios requiring human judgment

4. **ROI (Return on Investment)**
   - **Automate:** High ROI - saves significant time over multiple runs
   - **Manual:** Low ROI - takes longer to automate than manual execution

5. **Risk and Business Impact**
   - **Automate:** Critical business flows, high-risk areas
   - **Manual:** Low-risk, cosmetic features

6. **Test Data Requirements**
   - **Automate:** Tests with predictable, reusable test data
   - **Manual:** Tests requiring complex, dynamic data setup

7. **Execution Time**
   - **Automate:** Long-running tests, overnight execution
   - **Manual:** Quick ad-hoc checks

### Decision Matrix

| Test Characteristic | Automate? | Reason |
|---------------------|-----------|--------|
| Runs daily/every build | ✅ Yes | High frequency = High ROI |
| Critical business flow | ✅ Yes | High risk = Must automate |
| Stable functionality | ✅ Yes | Low maintenance cost |
| Repetitive tasks | ✅ Yes | Automation excels at repetition |
| One-time test | ❌ No | Not worth automation effort |
| Exploratory testing | ❌ No | Requires human creativity |
| UI changes frequently | ❌ No | High maintenance cost |
| Requires human judgment | ❌ No | Cannot be automated |
| Complex setup (one-time) | ❌ No | Low ROI |
| Visual validation | ⚠️ Maybe | Use visual testing tools if needed |

### Test Case Categories

#### **✅ MUST Automate**

1. **Smoke Tests**
   - Critical path tests run after every build
   - Example: Login, basic navigation, core functionality

2. **Regression Tests**
   - Tests that verify existing functionality
   - Example: All previous bug fixes, core features

3. **Data-Driven Tests**
   - Tests with multiple data sets
   - Example: Testing login with 100 different credentials

4. **API Tests**
   - Backend API validation
   - Example: REST API endpoints, response validation

5. **Performance Tests**
   - Load, stress, performance testing
   - Example: Response time, throughput tests

6. **Repetitive Tests**
   - Tests executed multiple times
   - Example: Form validations, field-level checks

#### **❌ Should NOT Automate**

1. **Exploratory Testing**
   - Ad-hoc, creative testing
   - Requires human intuition and creativity

2. **Usability Testing**
   - User experience evaluation
   - Requires human perception

3. **One-Time Tests**
   - Tests run only once
   - ROI doesn't justify automation

4. **Frequently Changing Tests**
   - Unstable requirements/UI
   - High maintenance cost

5. **Complex Setup Tests**
   - Tests requiring extensive manual setup
   - Example: Testing with physical hardware

6. **Visual Design Tests**
   - UI design, layout, colors
   - Better suited for manual review

#### **⚠️ MAYBE Automate (Case-by-Case)**

1. **Integration Tests**
   - Depends on complexity and frequency
   - Automate if stable and frequent

2. **End-to-End Tests**
   - Automate critical paths only
   - Keep suite small and focused

3. **Cross-Browser Tests**
   - Automate on critical browsers
   - Manual for edge cases

### Decision Process Flowchart

```
Start
  ↓
Is test executed frequently? (Daily/Every build)
  ↓ YES                    ↓ NO
Is it critical business flow?
  ↓ YES                    ↓ NO
Is functionality stable?
  ↓ YES                    ↓ NO
Is it repetitive?
  ↓ YES                    ↓ NO
Calculate ROI
  ↓
ROI > Threshold?
  ↓ YES                    ↓ NO
AUTOMATE              MANUAL TEST
```

### Code Example: Automation Decision Framework

```java
package framework;

public class AutomationDecisionFramework {
    
    public enum AutomationDecision {
        AUTOMATE,
        MANUAL,
        MAYBE_AUTOMATE
    }
    
    public static class TestCase {
        private String testName;
        private int executionFrequency; // Times per week
        private boolean isCritical;
        private boolean isStable;
        private boolean isRepetitive;
        private int manualExecutionTime; // Minutes
        private int automationSetupTime; // Hours
        private int automationMaintenanceTime; // Hours per month
        
        // Getters and setters
        public String getTestName() { return testName; }
        public void setTestName(String testName) { this.testName = testName; }
        public int getExecutionFrequency() { return executionFrequency; }
        public void setExecutionFrequency(int executionFrequency) { 
            this.executionFrequency = executionFrequency; 
        }
        public boolean isCritical() { return isCritical; }
        public void setCritical(boolean critical) { isCritical = critical; }
        public boolean isStable() { return isStable; }
        public void setStable(boolean stable) { isStable = stable; }
        public boolean isRepetitive() { return isRepetitive; }
        public void setRepetitive(boolean repetitive) { isRepetitive = repetitive; }
        public int getManualExecutionTime() { return manualExecutionTime; }
        public void setManualExecutionTime(int manualExecutionTime) { 
            this.manualExecutionTime = manualExecutionTime; 
        }
        public int getAutomationSetupTime() { return automationSetupTime; }
        public void setAutomationSetupTime(int automationSetupTime) { 
            this.automationSetupTime = automationSetupTime; 
        }
        public int getAutomationMaintenanceTime() { 
            return automationMaintenanceTime; 
        }
        public void setAutomationMaintenanceTime(int automationMaintenanceTime) { 
            this.automationMaintenanceTime = automationMaintenanceTime; 
        }
    }
    
    /**
     * Decides whether to automate a test case based on multiple factors
     */
    public static AutomationDecision shouldAutomate(TestCase testCase) {
        // Rule 1: High frequency + Critical = Automate
        if (testCase.getExecutionFrequency() >= 5 && testCase.isCritical()) {
            return AutomationDecision.AUTOMATE;
        }
        
        // Rule 2: Low frequency + Not critical = Manual
        if (testCase.getExecutionFrequency() < 2 && !testCase.isCritical()) {
            return AutomationDecision.MANUAL;
        }
        
        // Rule 3: Unstable functionality = Manual
        if (!testCase.isStable()) {
            return AutomationDecision.MANUAL;
        }
        
        // Rule 4: Calculate ROI
        double roi = calculateROI(testCase);
        if (roi > 2.0) { // ROI threshold: 2x
            return AutomationDecision.AUTOMATE;
        } else if (roi < 0.5) {
            return AutomationDecision.MANUAL;
        }
        
        // Rule 5: Repetitive + Stable = Automate
        if (testCase.isRepetitive() && testCase.isStable()) {
            return AutomationDecision.AUTOMATE;
        }
        
        return AutomationDecision.MAYBE_AUTOMATE;
    }
    
    /**
     * Calculate ROI: (Time saved - Setup time - Maintenance time) / Setup time
     */
    private static double calculateROI(TestCase testCase) {
        // Time saved per execution (manual time)
        double timeSavedPerExecution = testCase.getManualExecutionTime() / 60.0; // Convert to hours
        
        // Total time saved per month
        int executionsPerMonth = testCase.getExecutionFrequency() * 4; // Approximate
        double totalTimeSaved = timeSavedPerExecution * executionsPerMonth;
        
        // Total cost (setup + maintenance)
        double setupCost = testCase.getAutomationSetupTime();
        double maintenanceCost = testCase.getAutomationMaintenanceTime();
        double totalCost = setupCost + maintenanceCost;
        
        if (totalCost == 0) return 0;
        
        // ROI = Time saved / Cost
        return totalTimeSaved / totalCost;
    }
    
    /**
     * Example usage
     */
    public static void main(String[] args) {
        // Example 1: Login test - High frequency, critical, stable
        TestCase loginTest = new TestCase();
        loginTest.setTestName("User Login Test");
        loginTest.setExecutionFrequency(20); // Daily
        loginTest.setCritical(true);
        loginTest.setStable(true);
        loginTest.setRepetitive(true);
        loginTest.setManualExecutionTime(5); // 5 minutes
        loginTest.setAutomationSetupTime(2); // 2 hours
        loginTest.setAutomationMaintenanceTime(1); // 1 hour/month
        
        AutomationDecision decision = shouldAutomate(loginTest);
        System.out.println("Login Test Decision: " + decision);
        // Output: AUTOMATE
        
        // Example 2: One-time UI design review
        TestCase designReview = new TestCase();
        designReview.setTestName("UI Design Review");
        designReview.setExecutionFrequency(1); // Once
        designReview.setCritical(false);
        designReview.setStable(false);
        designReview.setRepetitive(false);
        designReview.setManualExecutionTime(30);
        designReview.setAutomationSetupTime(8);
        designReview.setAutomationMaintenanceTime(2);
        
        decision = shouldAutomate(designReview);
        System.out.println("Design Review Decision: " + decision);
        // Output: MANUAL
        
        // Example 3: Regression test suite
        TestCase regressionTest = new TestCase();
        regressionTest.setTestName("Checkout Regression Test");
        regressionTest.setExecutionFrequency(10); // Twice weekly
        regressionTest.setCritical(true);
        regressionTest.setStable(true);
        regressionTest.setRepetitive(true);
        regressionTest.setManualExecutionTime(60); // 1 hour
        regressionTest.setAutomationSetupTime(4); // 4 hours
        regressionTest.setAutomationMaintenanceTime(2); // 2 hours/month
        
        decision = shouldAutomate(regressionTest);
        System.out.println("Regression Test Decision: " + decision);
        // Output: AUTOMATE
    }
}
```

### Scenario-Based Examples

#### **Scenario 1: E-commerce Checkout Flow**

**Test Case:** "Verify complete checkout process with payment"

**Analysis:**
- **Frequency:** Runs daily (smoke test) + regression (weekly)
- **Criticality:** High - directly impacts revenue
- **Stability:** Stable - core business functionality
- **Repetitive:** Yes - same steps every time
- **Manual Time:** 15 minutes per execution
- **Automation Setup:** 4 hours
- **Maintenance:** 1 hour/month

**Decision:** ✅ **AUTOMATE**
- High frequency + critical = Must automate
- ROI: (15 min × 30 executions/month) / (4 hours setup + 1 hour maintenance) = 7.5x
- Saves 7.5 hours/month after initial setup

#### **Scenario 2: UI Color Scheme Review**

**Test Case:** "Verify button colors match brand guidelines"

**Analysis:**
- **Frequency:** Once per release
- **Criticality:** Low - cosmetic
- **Stability:** Low - design changes frequently
- **Repetitive:** No - requires visual judgment
- **Manual Time:** 10 minutes
- **Automation Setup:** 6 hours (visual testing tools)
- **Maintenance:** 3 hours/month (design changes)

**Decision:** ❌ **MANUAL**
- Low frequency + low criticality = Not worth automating
- ROI: (10 min × 1 execution) / (6 hours + 3 hours) = 0.02x (very low)
- Visual judgment better done manually

#### **Scenario 3: API Response Validation**

**Test Case:** "Validate all API endpoints return correct status codes"

**Analysis:**
- **Frequency:** Every build (20+ times/day)
- **Criticality:** High - API is core functionality
- **Stability:** High - API contracts are stable
- **Repetitive:** Yes - same validation logic
- **Manual Time:** 30 minutes
- **Automation Setup:** 2 hours
- **Maintenance:** 0.5 hours/month

**Decision:** ✅ **AUTOMATE**
- Very high frequency + critical = Must automate
- ROI: (30 min × 400 executions/month) / (2 hours + 0.5 hours) = 80x
- Excellent ROI, low maintenance

### Best Practices

1. **Start with High-ROI Tests**
   - Focus on frequently executed, critical tests first
   - Build automation foundation with high-value tests

2. **Consider Maintenance Cost**
   - Factor in ongoing maintenance time
   - Unstable tests may not be worth automating

3. **Balance Automation Coverage**
   - Don't try to automate everything
   - Maintain 70-80% automation, 20-30% manual

4. **Review and Reassess**
   - Regularly review automation decisions
   - Decommission tests that no longer provide value

5. **Consider Test Pyramid**
   - More unit tests (automated)
   - Fewer integration tests (automated)
   - Even fewer E2E tests (selective automation)

6. **Factor in Team Skills**
   - Consider team's automation expertise
   - Start simple, build complexity over time

7. **Document Decisions**
   - Document why tests were automated or not
   - Helps future decision-making

### Summary Checklist

**Automate if:**
- ✅ Runs frequently (daily/weekly)
- ✅ Critical business functionality
- ✅ Stable requirements/UI
- ✅ Repetitive tasks
- ✅ High ROI (saves significant time)
- ✅ Well-defined, predictable scenarios

**Don't Automate if:**
- ❌ Runs rarely (once or twice)
- ❌ Requires human judgment/creativity
- ❌ Frequently changing
- ❌ Low ROI
- ❌ Complex one-time setup
- ❌ Visual/design validation

---

## 2. Write Selenium code to find broken links on a webpage

### Context
This question tests your Selenium WebDriver skills, HTTP client knowledge, exception handling, and ability to write utility methods for common web testing tasks.

### Detailed Answer

Finding broken links involves:
1. Collecting all links from the webpage
2. Extracting URLs from link elements
3. Making HTTP requests to verify link accessibility
4. Reporting broken links

### Implementation Approaches

#### **Approach 1: Using Java HttpURLConnection**

```java
package utils;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.By;
import java.io.IOException;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.ArrayList;
import java.util.List;

public class BrokenLinkChecker {
    private WebDriver driver;
    private List<String> brokenLinks = new ArrayList<>();
    private List<String> validLinks = new ArrayList<>();
    
    public BrokenLinkChecker(WebDriver driver) {
        this.driver = driver;
    }
    
    /**
     * Find all broken links on the current page
     * @return List of broken link URLs
     */
    public List<String> findBrokenLinks() {
        // Get all anchor tags
        List<WebElement> links = driver.findElements(By.tagName("a"));
        
        System.out.println("Total links found: " + links.size());
        
        for (WebElement link : links) {
            String url = link.getAttribute("href");
            
            if (url == null || url.isEmpty()) {
                System.out.println("Link with no href attribute: " + link.getText());
                continue;
            }
            
            // Skip mailto, tel, javascript links
            if (url.startsWith("mailto:") || url.startsWith("tel:") || 
                url.startsWith("javascript:") || url.startsWith("#")) {
                continue;
            }
            
            // Check if link is broken
            if (isLinkBroken(url)) {
                brokenLinks.add(url);
                System.out.println("BROKEN LINK: " + url + " - Text: " + link.getText());
            } else {
                validLinks.add(url);
            }
        }
        
        return brokenLinks;
    }
    
    /**
     * Check if a link is broken by making HTTP request
     * @param linkUrl URL to check
     * @return true if link is broken, false otherwise
     */
    private boolean isLinkBroken(String linkUrl) {
        try {
            URL url = new URL(linkUrl);
            HttpURLConnection httpURLConnection = (HttpURLConnection) url.openConnection();
            httpURLConnection.setConnectTimeout(5000); // 5 seconds timeout
            httpURLConnection.setReadTimeout(5000);
            httpURLConnection.setRequestMethod("HEAD"); // Use HEAD for faster checking
            
            int responseCode = httpURLConnection.getResponseCode();
            
            // Consider 2xx and 3xx as valid
            if (responseCode >= 200 && responseCode < 400) {
                return false; // Link is valid
            } else {
                return true; // Link is broken
            }
            
        } catch (IOException e) {
            // Exception means link is broken or unreachable
            System.out.println("Exception checking link " + linkUrl + ": " + e.getMessage());
            return true;
        }
    }
    
    /**
     * Get detailed information about broken links
     * @return Map with link URL and error details
     */
    public void printBrokenLinksReport() {
        System.out.println("\n========== BROKEN LINKS REPORT ==========");
        System.out.println("Total Broken Links: " + brokenLinks.size());
        System.out.println("Total Valid Links: " + validLinks.size());
        
        if (!brokenLinks.isEmpty()) {
            System.out.println("\nBroken Links:");
            brokenLinks.forEach(link -> System.out.println("  - " + link));
        }
    }
    
    /**
     * Get response code for a specific URL
     * @param url URL to check
     * @return HTTP response code
     */
    public int getResponseCode(String url) {
        try {
            URL urlObj = new URL(url);
            HttpURLConnection connection = (HttpURLConnection) urlObj.openConnection();
            connection.setRequestMethod("HEAD");
            connection.setConnectTimeout(5000);
            connection.setReadTimeout(5000);
            return connection.getResponseCode();
        } catch (IOException e) {
            return -1; // Error code
        }
    }
}
```

#### **Approach 2: Enhanced Version with Detailed Reporting**

```java
package utils;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.By;
import java.io.IOException;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.*;

public class EnhancedBrokenLinkChecker {
    private WebDriver driver;
    
    public static class LinkStatus {
        private String url;
        private String linkText;
        private int responseCode;
        private String status;
        private String errorMessage;
        
        public LinkStatus(String url, String linkText, int responseCode, 
                         String status, String errorMessage) {
            this.url = url;
            this.linkText = linkText;
            this.responseCode = responseCode;
            this.status = status;
            this.errorMessage = errorMessage;
        }
        
        // Getters
        public String getUrl() { return url; }
        public String getLinkText() { return linkText; }
        public int getResponseCode() { return responseCode; }
        public String getStatus() { return status; }
        public String getErrorMessage() { return errorMessage; }
    }
    
    private List<LinkStatus> linkStatuses = new ArrayList<>();
    
    public EnhancedBrokenLinkChecker(WebDriver driver) {
        this.driver = driver;
    }
    
    /**
     * Check all links on the page and return detailed status
     */
    public List<LinkStatus> checkAllLinks() {
        List<WebElement> links = driver.findElements(By.tagName("a"));
        System.out.println("Checking " + links.size() + " links...");
        
        for (WebElement link : links) {
            String url = link.getAttribute("href");
            String linkText = link.getText().trim();
            
            if (url == null || url.isEmpty()) {
                linkStatuses.add(new LinkStatus(
                    "N/A", linkText, -1, "INVALID", "No href attribute"
                ));
                continue;
            }
            
            // Skip special protocol links
            if (shouldSkipLink(url)) {
                continue;
            }
            
            // Check link status
            LinkStatus status = checkLinkStatus(url, linkText);
            linkStatuses.add(status);
        }
        
        return linkStatuses;
    }
    
    /**
     * Check status of a single link
     */
    private LinkStatus checkLinkStatus(String url, String linkText) {
        try {
            // Handle relative URLs
            if (url.startsWith("/")) {
                String baseURL = new URL(driver.getCurrentUrl()).getProtocol() + "://" +
                               new URL(driver.getCurrentUrl()).getHost();
                url = baseURL + url;
            }
            
            URL urlObj = new URL(url);
            HttpURLConnection connection = (HttpURLConnection) urlObj.openConnection();
            connection.setRequestMethod("HEAD");
            connection.setConnectTimeout(10000);
            connection.setReadTimeout(10000);
            connection.setInstanceFollowRedirects(false); // Don't follow redirects automatically
            
            int responseCode = connection.getResponseCode();
            String status = getStatusFromCode(responseCode);
            
            return new LinkStatus(url, linkText, responseCode, status, null);
            
        } catch (IOException e) {
            return new LinkStatus(url, linkText, -1, "BROKEN", e.getMessage());
        }
    }
    
    /**
     * Determine if link should be skipped
     */
    private boolean shouldSkipLink(String url) {
        return url.startsWith("mailto:") ||
               url.startsWith("tel:") ||
               url.startsWith("javascript:") ||
               url.startsWith("#") ||
               url.startsWith("ftp:");
    }
    
    /**
     * Get status description from HTTP response code
     */
    private String getStatusFromCode(int responseCode) {
        if (responseCode >= 200 && responseCode < 300) {
            return "VALID";
        } else if (responseCode >= 300 && responseCode < 400) {
            return "REDIRECT";
        } else if (responseCode >= 400 && responseCode < 500) {
            return "CLIENT_ERROR";
        } else if (responseCode >= 500) {
            return "SERVER_ERROR";
        }
        return "UNKNOWN";
    }
    
    /**
     * Get only broken links
     */
    public List<LinkStatus> getBrokenLinks() {
        List<LinkStatus> broken = new ArrayList<>();
        for (LinkStatus status : linkStatuses) {
            if (status.getStatus().equals("BROKEN") || 
                status.getStatus().equals("CLIENT_ERROR") ||
                status.getStatus().equals("SERVER_ERROR")) {
                broken.add(status);
            }
        }
        return broken;
    }
    
    /**
     * Print comprehensive report
     */
    public void printReport() {
        List<LinkStatus> broken = getBrokenLinks();
        List<LinkStatus> valid = new ArrayList<>();
        List<LinkStatus> redirects = new ArrayList<>();
        
        for (LinkStatus status : linkStatuses) {
            if (status.getStatus().equals("VALID")) {
                valid.add(status);
            } else if (status.getStatus().equals("REDIRECT")) {
                redirects.add(status);
            }
        }
        
        System.out.println("\n========== LINK STATUS REPORT ==========");
        System.out.println("Total Links Checked: " + linkStatuses.size());
        System.out.println("Valid Links: " + valid.size());
        System.out.println("Broken Links: " + broken.size());
        System.out.println("Redirects: " + redirects.size());
        
        if (!broken.isEmpty()) {
            System.out.println("\n--- BROKEN LINKS ---");
            for (LinkStatus status : broken) {
                System.out.println("URL: " + status.getUrl());
                System.out.println("Text: " + status.getLinkText());
                System.out.println("Response Code: " + status.getResponseCode());
                System.out.println("Error: " + status.getErrorMessage());
                System.out.println("---");
            }
        }
    }
}
```

#### **Approach 3: Using Apache HttpClient (More Robust)**

```java
package utils;

import org.apache.http.HttpResponse;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpHead;
import org.apache.http.impl.client.HttpClientBuilder;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.By;
import java.util.ArrayList;
import java.util.List;

public class HttpClientBrokenLinkChecker {
    private WebDriver driver;
    private HttpClient httpClient;
    
    public HttpClientBrokenLinkChecker(WebDriver driver) {
        this.driver = driver;
        this.httpClient = HttpClientBuilder.create().build();
    }
    
    /**
     * Find broken links using Apache HttpClient
     */
    public List<String> findBrokenLinks() {
        List<String> brokenLinks = new ArrayList<>();
        List<WebElement> links = driver.findElements(By.tagName("a"));
        
        for (WebElement link : links) {
            String url = link.getAttribute("href");
            
            if (url == null || url.isEmpty() || shouldSkip(url)) {
                continue;
            }
            
            try {
                HttpHead request = new HttpHead(url);
                HttpResponse response = httpClient.execute(request);
                int statusCode = response.getStatusLine().getStatusCode();
                
                if (statusCode >= 400) {
                    brokenLinks.add(url);
                    System.out.println("Broken: " + url + " - Status: " + statusCode);
                }
            } catch (Exception e) {
                brokenLinks.add(url);
                System.out.println("Broken: " + url + " - Error: " + e.getMessage());
            }
        }
        
        return brokenLinks;
    }
    
    private boolean shouldSkip(String url) {
        return url.startsWith("mailto:") || url.startsWith("tel:") || 
               url.startsWith("javascript:") || url.startsWith("#");
    }
}
```

### Complete Test Example

```java
package tests;

import base.BaseTest;
import org.testng.Assert;
import org.testng.annotations.Test;
import utils.BrokenLinkChecker;

public class BrokenLinkTest extends BaseTest {
    
    @Test
    public void testBrokenLinksOnHomepage() {
        // Navigate to page
        driver.get("https://example.com");
        
        // Check for broken links
        BrokenLinkChecker checker = new BrokenLinkChecker(driver);
        List<String> brokenLinks = checker.findBrokenLinks();
        
        // Print report
        checker.printBrokenLinksReport();
        
        // Assert no broken links
        Assert.assertTrue(brokenLinks.isEmpty(), 
            "Found " + brokenLinks.size() + " broken links: " + brokenLinks);
    }
    
    @Test
    public void testBrokenLinksOnMultiplePages() {
        String[] pages = {
            "https://example.com",
            "https://example.com/products",
            "https://example.com/about"
        };
        
        List<String> allBrokenLinks = new ArrayList<>();
        
        for (String page : pages) {
            driver.get(page);
            BrokenLinkChecker checker = new BrokenLinkChecker(driver);
            List<String> brokenLinks = checker.findBrokenLinks();
            allBrokenLinks.addAll(brokenLinks);
        }
        
        Assert.assertTrue(allBrokenLinks.isEmpty(), 
            "Found broken links across pages: " + allBrokenLinks);
    }
}
```

### Scenario-Based Example

**Real Scenario: Checking all links on an e-commerce website**

```java
@Test
public void testEcommerceSiteLinks() {
    driver.get("https://ecommerce.example.com");
    
    EnhancedBrokenLinkChecker checker = new EnhancedBrokenLinkChecker(driver);
    List<LinkStatus> allLinks = checker.checkAllLinks();
    List<LinkStatus> brokenLinks = checker.getBrokenLinks();
    
    // Generate detailed report
    checker.printReport();
    
    // Assert critical links are not broken
    List<String> criticalLinks = Arrays.asList(
        "https://ecommerce.example.com/checkout",
        "https://ecommerce.example.com/cart",
        "https://ecommerce.example.com/login"
    );
    
    for (String criticalLink : criticalLinks) {
        boolean isBroken = brokenLinks.stream()
            .anyMatch(link -> link.getUrl().equals(criticalLink));
        Assert.assertFalse(isBroken, 
            "Critical link is broken: " + criticalLink);
    }
    
    // Log summary
    System.out.println("Total links checked: " + allLinks.size());
    System.out.println("Broken links: " + brokenLinks.size());
    System.out.println("Success rate: " + 
        ((allLinks.size() - brokenLinks.size()) * 100.0 / allLinks.size()) + "%");
}
```

### Best Practices

1. **Use HEAD requests** - Faster than GET for link checking
2. **Set timeouts** - Prevent hanging on slow/unreachable links
3. **Handle redirects** - Check if redirects are valid (3xx codes)
4. **Skip special protocols** - mailto, tel, javascript links
5. **Handle relative URLs** - Convert to absolute URLs
6. **Parallel execution** - Check multiple links simultaneously for speed
7. **Detailed reporting** - Log URL, link text, response code, error
8. **Retry mechanism** - Retry failed links (might be temporary network issues)

---

## 3. How do you switch between frames and return to the parent frame?

### Context
This question tests your understanding of Selenium WebDriver's frame handling, which is essential when working with iframes in modern web applications.

### Detailed Answer

Frames (iframes) are HTML elements that embed another HTML document within the current page. Selenium requires explicit switching to interact with elements inside frames.

### Key Concepts

1. **Frame Switching:** Move WebDriver focus to a frame
2. **Parent Frame:** Return to immediate parent frame
3. **Default Content:** Return to main page (exit all frames)

### Methods to Switch Frames

#### **Method 1: Switch by Index**

```java
// Switch to frame by index (0-based)
driver.switchTo().frame(0);  // First frame
driver.switchTo().frame(1);  // Second frame

// Return to parent frame
driver.switchTo().parentFrame();

// Return to default content (main page)
driver.switchTo().defaultContent();
```

#### **Method 2: Switch by Name or ID**

```java
// Switch to frame by name attribute
driver.switchTo().frame("frameName");

// Switch to frame by id attribute
driver.switchTo().frame("frameId");

// Return to parent
driver.switchTo().parentFrame();
```

#### **Method 3: Switch by WebElement**

```java
// Find frame element and switch to it
WebElement frameElement = driver.findElement(By.id("myFrame"));
driver.switchTo().frame(frameElement);

// Or using other locators
WebElement frameByXpath = driver.findElement(By.xpath("//iframe[@src='example.html']"));
driver.switchTo().frame(frameByXpath);
```

### Complete Code Examples

#### **Example 1: Basic Frame Switching**

```java
package utils;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.By;
import org.openqa.selenium.NoSuchFrameException;

public class FrameHandler {
    private WebDriver driver;
    
    public FrameHandler(WebDriver driver) {
        this.driver = driver;
    }
    
    /**
     * Switch to frame by index
     */
    public void switchToFrameByIndex(int index) {
        try {
            driver.switchTo().frame(index);
            System.out.println("Switched to frame at index: " + index);
        } catch (NoSuchFrameException e) {
            System.out.println("Frame not found at index: " + index);
            throw e;
        }
    }
    
    /**
     * Switch to frame by name or id
     */
    public void switchToFrameByNameOrId(String nameOrId) {
        try {
            driver.switchTo().frame(nameOrId);
            System.out.println("Switched to frame: " + nameOrId);
        } catch (NoSuchFrameException e) {
            System.out.println("Frame not found: " + nameOrId);
            throw e;
        }
    }
    
    /**
     * Switch to frame by WebElement
     */
    public void switchToFrameByElement(WebElement frameElement) {
        try {
            driver.switchTo().frame(frameElement);
            System.out.println("Switched to frame element");
        } catch (NoSuchFrameException e) {
            System.out.println("Frame element not found");
            throw e;
        }
    }
    
    /**
     * Switch to parent frame
     */
    public void switchToParentFrame() {
        driver.switchTo().parentFrame();
        System.out.println("Switched to parent frame");
    }
    
    /**
     * Switch to default content (main page)
     */
    public void switchToDefaultContent() {
        driver.switchTo().defaultContent();
        System.out.println("Switched to default content");
    }
}
```

#### **Example 2: Nested Frames Handling**

```java
package tests;

import base.BaseTest;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.By;
import org.testng.annotations.Test;

public class NestedFrameTest extends BaseTest {
    
    @Test
    public void testNestedFrames() {
        driver.get("https://example.com/nested-frames");
        
        // Main page
        System.out.println("Current context: Main page");
        
        // Switch to outer frame
        driver.switchTo().frame("outerFrame");
        System.out.println("Switched to outer frame");
        
        // Switch to inner frame (nested)
        driver.switchTo().frame("innerFrame");
        System.out.println("Switched to inner frame");
        
        // Interact with element in inner frame
        WebElement innerElement = driver.findElement(By.id("innerElement"));
        innerElement.click();
        
        // Return to parent frame (outer frame)
        driver.switchTo().parentFrame();
        System.out.println("Returned to outer frame");
        
        // Interact with element in outer frame
        WebElement outerElement = driver.findElement(By.id("outerElement"));
        outerElement.click();
        
        // Return to main page (default content)
        driver.switchTo().defaultContent();
        System.out.println("Returned to main page");
        
        // Now can interact with main page elements
        WebElement mainElement = driver.findElement(By.id("mainElement"));
        mainElement.click();
    }
}
```

#### **Example 3: Multiple Frames on Same Page**

```java
@Test
public void testMultipleFrames() {
    driver.get("https://example.com/multiple-frames");
    
    // Work with first frame
    driver.switchTo().frame("frame1");
    WebElement element1 = driver.findElement(By.id("input1"));
    element1.sendKeys("Test Data 1");
    driver.switchTo().defaultContent(); // Return to main page
    
    // Work with second frame
    driver.switchTo().frame("frame2");
    WebElement element2 = driver.findElement(By.id("input2"));
    element2.sendKeys("Test Data 2");
    driver.switchTo().defaultContent(); // Return to main page
    
    // Work with third frame
    WebElement frame3 = driver.findElement(By.xpath("//iframe[@name='frame3']"));
    driver.switchTo().frame(frame3);
    WebElement element3 = driver.findElement(By.id("input3"));
    element3.sendKeys("Test Data 3");
    driver.switchTo().defaultContent();
}
```

#### **Example 4: Utility Class for Frame Operations**

```java
package utils;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.By;
import org.openqa.selenium.NoSuchFrameException;
import java.util.ArrayList;
import java.util.List;

public class FrameUtility {
    private WebDriver driver;
    private List<String> frameHistory = new ArrayList<>(); // Track frame navigation
    
    public FrameUtility(WebDriver driver) {
        this.driver = driver;
    }
    
    /**
     * Switch to frame and track in history
     */
    public void switchToFrame(String frameIdentifier) {
        try {
            // Try by name/id first
            driver.switchTo().frame(frameIdentifier);
            frameHistory.add("Frame: " + frameIdentifier);
        } catch (NoSuchFrameException e) {
            // Try by WebElement
            try {
                WebElement frame = driver.findElement(By.name(frameIdentifier));
                driver.switchTo().frame(frame);
                frameHistory.add("Frame Element: " + frameIdentifier);
            } catch (Exception ex) {
                throw new NoSuchFrameException("Frame not found: " + frameIdentifier);
            }
        }
    }
    
    /**
     * Switch to frame by index
     */
    public void switchToFrame(int index) {
        driver.switchTo().frame(index);
        frameHistory.add("Frame Index: " + index);
    }
    
    /**
     * Switch to parent frame
     */
    public void switchToParentFrame() {
        driver.switchTo().parentFrame();
        if (!frameHistory.isEmpty()) {
            frameHistory.remove(frameHistory.size() - 1);
        }
    }
    
    /**
     * Switch to default content and clear history
     */
    public void switchToDefaultContent() {
        driver.switchTo().defaultContent();
        frameHistory.clear();
    }
    
    /**
     * Get current frame context
     */
    public String getCurrentFrameContext() {
        if (frameHistory.isEmpty()) {
            return "Main Page (Default Content)";
        }
        return frameHistory.get(frameHistory.size() - 1);
    }
    
    /**
     * Print frame navigation history
     */
    public void printFrameHistory() {
        System.out.println("Frame Navigation History:");
        if (frameHistory.isEmpty()) {
            System.out.println("  - Main Page");
        } else {
            for (int i = 0; i < frameHistory.size(); i++) {
                System.out.println("  " + (i + 1) + ". " + frameHistory.get(i));
            }
        }
    }
    
    /**
     * Find all frames on the page
     */
    public List<WebElement> findAllFrames() {
        // Switch to default content first
        switchToDefaultContent();
        
        // Find all iframe elements
        return driver.findElements(By.tagName("iframe"));
    }
    
    /**
     * Count frames on the page
     */
    public int getFrameCount() {
        return findAllFrames().size();
    }
}
```

#### **Example 5: Real-World Scenario - Payment Gateway Frame**

```java
@Test
public void testPaymentGatewayInFrame() {
    driver.get("https://ecommerce.example.com/checkout");
    
    // Fill checkout form on main page
    driver.findElement(By.id("shippingAddress")).sendKeys("123 Main St");
    driver.findElement(By.id("city")).sendKeys("New York");
    
    // Click "Pay Now" button which opens payment gateway in iframe
    driver.findElement(By.id("payNowButton")).click();
    
    // Wait for iframe to load
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    WebElement paymentFrame = wait.until(ExpectedConditions
        .presenceOfElementLocated(By.id("paymentGatewayFrame")));
    
    // Switch to payment gateway frame
    driver.switchTo().frame(paymentFrame);
    
    // Fill payment details inside frame
    driver.findElement(By.id("cardNumber")).sendKeys("4111111111111111");
    driver.findElement(By.id("expiryDate")).sendKeys("12/25");
    driver.findElement(By.id("cvv")).sendKeys("123");
    driver.findElement(By.id("cardholderName")).sendKeys("John Doe");
    
    // Submit payment
    driver.findElement(By.id("submitPayment")).click();
    
    // Return to main page (default content)
    driver.switchTo().defaultContent();
    
    // Verify order confirmation on main page
    WebElement confirmation = wait.until(ExpectedConditions
        .presenceOfElementLocated(By.id("orderConfirmation")));
    Assert.assertTrue(confirmation.isDisplayed(), "Order confirmation should be displayed");
}
```

### Common Patterns and Best Practices

#### **Pattern 1: Always Return to Default Content**

```java
@Test
public void testWithFrameCleanup() {
    try {
        // Switch to frame
        driver.switchTo().frame("myFrame");
        
        // Perform actions in frame
        driver.findElement(By.id("element")).click();
        
    } finally {
        // Always return to default content
        driver.switchTo().defaultContent();
    }
}
```

#### **Pattern 2: Wait for Frame Before Switching**

```java
public void switchToFrameSafely(String frameId) {
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    WebElement frame = wait.until(ExpectedConditions
        .presenceOfElementLocated(By.id(frameId)));
    driver.switchTo().frame(frame);
}
```

#### **Pattern 3: Verify Frame Context**

```java
public boolean isInFrame() {
    try {
        // Try to find element in main page
        driver.findElement(By.tagName("body"));
        return false; // In main page
    } catch (Exception e) {
        return true; // Likely in a frame
    }
}
```

### Scenario-Based Example

**Real Scenario: Testing embedded video player**

```java
@Test
public void testVideoPlayerInFrame() {
    driver.get("https://example.com/video-page");
    
    // Main page - click play button
    driver.findElement(By.id("playVideoButton")).click();
    
    // Wait and switch to video player frame
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    WebElement videoFrame = wait.until(ExpectedConditions
        .presenceOfElementLocated(By.xpath("//iframe[contains(@src, 'youtube')]")));
    
    driver.switchTo().frame(videoFrame);
    
    // Interact with video player controls inside frame
    WebElement playButton = wait.until(ExpectedConditions
        .elementToBeClickable(By.className("ytp-play-button")));
    playButton.click();
    
    // Verify video is playing
    WebElement videoPlayer = driver.findElement(By.className("video-stream"));
    Assert.assertTrue(videoPlayer.isDisplayed(), "Video should be playing");
    
    // Return to main page
    driver.switchTo().defaultContent();
    
    // Verify video info on main page
    WebElement videoTitle = driver.findElement(By.id("videoTitle"));
    Assert.assertNotNull(videoTitle.getText(), "Video title should be displayed");
}
```

### Summary

**Key Methods:**
- `driver.switchTo().frame(index)` - Switch by index
- `driver.switchTo().frame(nameOrId)` - Switch by name/id
- `driver.switchTo().frame(webElement)` - Switch by element
- `driver.switchTo().parentFrame()` - Return to parent frame
- `driver.switchTo().defaultContent()` - Return to main page

**Best Practices:**
1. Always return to default content after frame operations
2. Wait for frame to load before switching
3. Use try-finally to ensure cleanup
4. Track frame navigation for debugging
5. Verify you're in the correct frame context

---

## 4. You have 10 links — how will you print the title of each link easily?

### Context
This question tests your ability to work with collections, loops, and Selenium WebElement operations efficiently. It's a common task in web automation.

### Detailed Answer

There are multiple ways to print link titles. The most efficient approach uses Selenium's `findElements()` to get all links and iterate through them.

### Implementation Approaches

#### **Approach 1: Using findElements() with Enhanced For Loop**

```java
package utils;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.By;
import java.util.List;

public class LinkTitlePrinter {
    private WebDriver driver;
    
    public LinkTitlePrinter(WebDriver driver) {
        this.driver = driver;
    }
    
    /**
     * Print titles of all links on the page
     */
    public void printAllLinkTitles() {
        // Get all anchor tags
        List<WebElement> links = driver.findElements(By.tagName("a"));
        
        System.out.println("Total links found: " + links.size());
        System.out.println("\nLink Titles:");
        System.out.println("=".repeat(50));
        
        // Iterate through links and print titles
        for (WebElement link : links) {
            String title = link.getAttribute("title");
            String linkText = link.getText().trim();
            String href = link.getAttribute("href");
            
            // Use title attribute if available, otherwise use link text
            String displayTitle = (title != null && !title.isEmpty()) 
                ? title 
                : (linkText.isEmpty() ? href : linkText);
            
            System.out.println("Title: " + displayTitle);
        }
    }
    
    /**
     * Print titles with index numbers
     */
    public void printLinkTitlesWithIndex() {
        List<WebElement> links = driver.findElements(By.tagName("a"));
        
        System.out.println("\nLink Titles with Index:");
        System.out.println("=".repeat(50));
        
        for (int i = 0; i < links.size(); i++) {
            WebElement link = links.get(i);
            String title = getLinkTitle(link);
            System.out.println((i + 1) + ". " + title);
        }
    }
    
    /**
     * Get title of a link (title attribute or text)
     */
    private String getLinkTitle(WebElement link) {
        String title = link.getAttribute("title");
        if (title != null && !title.isEmpty()) {
            return title;
        }
        
        String linkText = link.getText().trim();
        if (!linkText.isEmpty()) {
            return linkText;
        }
        
        String href = link.getAttribute("href");
        return (href != null) ? href : "No title";
    }
}
```

#### **Approach 2: Using Java 8 Streams (More Concise)**

```java
package utils;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.By;
import java.util.List;
import java.util.stream.Collectors;

public class StreamLinkTitlePrinter {
    private WebDriver driver;
    
    public StreamLinkTitlePrinter(WebDriver driver) {
        this.driver = driver;
    }
    
    /**
     * Print all link titles using streams
     */
    public void printLinkTitles() {
        List<WebElement> links = driver.findElements(By.tagName("a"));
        
        System.out.println("Total links: " + links.size());
        System.out.println("\nLink Titles:");
        
        links.stream()
            .map(this::getLinkTitle)
            .forEach(title -> System.out.println("  - " + title));
    }
    
    /**
     * Get list of all link titles
     */
    public List<String> getAllLinkTitles() {
        return driver.findElements(By.tagName("a"))
            .stream()
            .map(this::getLinkTitle)
            .collect(Collectors.toList());
    }
    
    /**
     * Print link titles with URL
     */
    public void printLinkTitlesWithURL() {
        driver.findElements(By.tagName("a"))
            .stream()
            .forEach(link -> {
                String title = getLinkTitle(link);
                String url = link.getAttribute("href");
                System.out.println("Title: " + title + " | URL: " + url);
            });
    }
    
    private String getLinkTitle(WebElement link) {
        String title = link.getAttribute("title");
        if (title != null && !title.isEmpty()) {
            return title;
        }
        String text = link.getText().trim();
        return text.isEmpty() ? link.getAttribute("href") : text;
    }
}
```

#### **Approach 3: Enhanced Version with Filtering**

```java
package utils;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.By;
import java.util.ArrayList;
import java.util.List;

public class EnhancedLinkTitlePrinter {
    private WebDriver driver;
    
    public static class LinkInfo {
        private int index;
        private String title;
        private String url;
        private String linkText;
        
        public LinkInfo(int index, String title, String url, String linkText) {
            this.index = index;
            this.title = title;
            this.url = url;
            this.linkText = linkText;
        }
        
        // Getters
        public int getIndex() { return index; }
        public String getTitle() { return title; }
        public String getUrl() { return url; }
        public String getLinkText() { return linkText; }
        
        @Override
        public String toString() {
            return String.format("%d. Title: %s | Text: %s | URL: %s", 
                index, title, linkText, url);
        }
    }
    
    public EnhancedLinkTitlePrinter(WebDriver driver) {
        this.driver = driver;
    }
    
    /**
     * Get all link information
     */
    public List<LinkInfo> getAllLinkInfo() {
        List<WebElement> links = driver.findElements(By.tagName("a"));
        List<LinkInfo> linkInfoList = new ArrayList<>();
        
        for (int i = 0; i < links.size(); i++) {
            WebElement link = links.get(i);
            String title = link.getAttribute("title");
            String url = link.getAttribute("href");
            String linkText = link.getText().trim();
            
            // Use title attribute, fallback to link text, then URL
            String displayTitle = (title != null && !title.isEmpty()) 
                ? title 
                : (linkText.isEmpty() ? url : linkText);
            
            linkInfoList.add(new LinkInfo(i + 1, displayTitle, url, linkText));
        }
        
        return linkInfoList;
    }
    
    /**
     * Print all link titles in formatted way
     */
    public void printAllLinkTitles() {
        List<LinkInfo> links = getAllLinkInfo();
        
        System.out.println("\n" + "=".repeat(70));
        System.out.println("LINK TITLES REPORT");
        System.out.println("=".repeat(70));
        System.out.println("Total Links: " + links.size());
        System.out.println("-".repeat(70));
        
        for (LinkInfo link : links) {
            System.out.println(link.toString());
        }
        
        System.out.println("=".repeat(70));
    }
    
    /**
     * Print only visible links
     */
    public void printVisibleLinkTitles() {
        List<WebElement> links = driver.findElements(By.tagName("a"));
        
        System.out.println("\nVisible Link Titles:");
        int visibleCount = 0;
        
        for (WebElement link : links) {
            if (link.isDisplayed()) {
                visibleCount++;
                String title = getLinkTitle(link);
                System.out.println(visibleCount + ". " + title);
            }
        }
        
        System.out.println("\nTotal visible links: " + visibleCount);
    }
    
    /**
     * Print links with specific text pattern
     */
    public void printLinksWithPattern(String pattern) {
        List<WebElement> links = driver.findElements(By.tagName("a"));
        
        System.out.println("\nLinks containing '" + pattern + "':");
        
        links.stream()
            .filter(link -> {
                String text = link.getText().toLowerCase();
                String title = link.getAttribute("title");
                return (text.contains(pattern.toLowerCase()) || 
                       (title != null && title.toLowerCase().contains(pattern.toLowerCase())));
            })
            .forEach(link -> {
                String title = getLinkTitle(link);
                System.out.println("  - " + title);
            });
    }
    
    private String getLinkTitle(WebElement link) {
        String title = link.getAttribute("title");
        if (title != null && !title.isEmpty()) {
            return title;
        }
        String text = link.getText().trim();
        return text.isEmpty() ? link.getAttribute("href") : text;
    }
}
```

### Complete Test Examples

#### **Example 1: Simple Print All Titles**

```java
package tests;

import base.BaseTest;
import org.testng.annotations.Test;
import utils.LinkTitlePrinter;

public class LinkTitleTest extends BaseTest {
    
    @Test
    public void testPrintAllLinkTitles() {
        driver.get("https://example.com");
        
        LinkTitlePrinter printer = new LinkTitlePrinter(driver);
        printer.printAllLinkTitles();
    }
}
```

#### **Example 2: Print Titles and Verify Count**

```java
@Test
public void testLinkTitlesAndCount() {
    driver.get("https://example.com");
    
    List<WebElement> links = driver.findElements(By.tagName("a"));
    
    System.out.println("Total links: " + links.size());
    Assert.assertEquals(links.size(), 10, "Should have exactly 10 links");
    
    System.out.println("\nLink Titles:");
    for (int i = 0; i < links.size(); i++) {
        WebElement link = links.get(i);
        String title = link.getAttribute("title");
        String linkText = link.getText().trim();
        String displayTitle = (title != null && !title.isEmpty()) 
            ? title 
            : (linkText.isEmpty() ? "No title" : linkText);
        
        System.out.println((i + 1) + ". " + displayTitle);
        
        // Verify title is not empty
        Assert.assertFalse(displayTitle.isEmpty(), 
            "Link " + (i + 1) + " should have a title");
    }
}
```

#### **Example 3: Store Titles in List**

```java
@Test
public void testStoreLinkTitles() {
    driver.get("https://example.com");
    
    List<WebElement> links = driver.findElements(By.tagName("a"));
    List<String> linkTitles = new ArrayList<>();
    
    for (WebElement link : links) {
        String title = link.getAttribute("title");
        String linkText = link.getText().trim();
        String displayTitle = (title != null && !title.isEmpty()) 
            ? title 
            : linkText;
        linkTitles.add(displayTitle);
    }
    
    // Print all titles
    System.out.println("Stored Link Titles:");
    linkTitles.forEach(title -> System.out.println("  - " + title));
    
    // Use titles for further processing
    Assert.assertEquals(linkTitles.size(), 10, "Should have 10 link titles");
}
```

#### **Example 4: Print Titles with Additional Info**

```java
@Test
public void testPrintLinkTitlesWithDetails() {
    driver.get("https://example.com");
    
    List<WebElement> links = driver.findElements(By.tagName("a"));
    
    System.out.println("\n" + "=".repeat(80));
    System.out.println("LINK DETAILS");
    System.out.println("=".repeat(80));
    
    for (int i = 0; i < links.size(); i++) {
        WebElement link = links.get(i);
        
        String title = link.getAttribute("title");
        String linkText = link.getText().trim();
        String href = link.getAttribute("href");
        boolean isDisplayed = link.isDisplayed();
        boolean isEnabled = link.isEnabled();
        
        String displayTitle = (title != null && !title.isEmpty()) 
            ? title 
            : (linkText.isEmpty() ? href : linkText);
        
        System.out.println("\nLink #" + (i + 1) + ":");
        System.out.println("  Title: " + displayTitle);
        System.out.println("  Text: " + linkText);
        System.out.println("  URL: " + href);
        System.out.println("  Visible: " + isDisplayed);
        System.out.println("  Enabled: " + isEnabled);
    }
    
    System.out.println("\n" + "=".repeat(80));
}
```

### Scenario-Based Example

**Real Scenario: E-commerce navigation menu links**

```java
@Test
public void testEcommerceNavigationLinks() {
    driver.get("https://ecommerce.example.com");
    
    // Find navigation menu
    WebElement navMenu = driver.findElement(By.className("main-navigation"));
    
    // Get all links within navigation
    List<WebElement> navLinks = navMenu.findElements(By.tagName("a"));
    
    System.out.println("Navigation Menu Links (" + navLinks.size() + "):");
    System.out.println("-".repeat(50));
    
    List<String> expectedTitles = Arrays.asList(
        "Home", "Products", "About", "Contact", "Cart"
    );
    
    for (int i = 0; i < navLinks.size(); i++) {
        WebElement link = navLinks.get(i);
        String title = link.getAttribute("title");
        String linkText = link.getText().trim();
        String displayTitle = (title != null && !title.isEmpty()) 
            ? title 
            : linkText;
        
        System.out.println((i + 1) + ". " + displayTitle);
        
        // Verify link is visible and enabled
        Assert.assertTrue(link.isDisplayed(), 
            "Link should be visible: " + displayTitle);
        Assert.assertTrue(link.isEnabled(), 
            "Link should be enabled: " + displayTitle);
    }
    
    // Verify we have expected number of links
    Assert.assertEquals(navLinks.size(), 10, 
        "Should have 10 navigation links");
}
```

### One-Liner Solutions

```java
// Using Java 8 Streams - One liner
driver.findElements(By.tagName("a"))
    .forEach(link -> System.out.println(
        link.getAttribute("title") != null ? 
        link.getAttribute("title") : 
        link.getText()
    ));

// Store in list - One liner
List<String> titles = driver.findElements(By.tagName("a"))
    .stream()
    .map(link -> link.getAttribute("title") != null ? 
         link.getAttribute("title") : link.getText())
    .collect(Collectors.toList());
```

### Best Practices

1. **Use findElements()** - Gets all links at once (efficient)
2. **Handle null titles** - Check for null before using
3. **Fallback to link text** - Use text if title attribute missing
4. **Filter visible links** - Use `isDisplayed()` if needed
5. **Use streams for processing** - More concise and functional
6. **Store in collections** - For further processing/assertions
7. **Add error handling** - Handle exceptions gracefully

### Summary

**Easiest Approach:**
```java
List<WebElement> links = driver.findElements(By.tagName("a"));
for (WebElement link : links) {
    System.out.println(link.getAttribute("title") != null ? 
        link.getAttribute("title") : link.getText());
}
```

**Most Concise:**
```java
driver.findElements(By.tagName("a"))
    .forEach(link -> System.out.println(
        link.getAttribute("title") != null ? 
        link.getAttribute("title") : link.getText()
    ));
```

---

## 5. When should you use implicit wait and when should you use explicit wait?

### Context
This question tests your understanding of Selenium WebDriver wait strategies, which are crucial for handling dynamic web elements and avoiding flaky tests.

### Detailed Answer

Both **Implicit Wait** and **Explicit Wait** are used to handle timing issues in Selenium, but they work differently and should be used in different scenarios.

### Implicit Wait

**Definition:** Implicit wait tells WebDriver to poll the DOM for a certain amount of time when trying to find elements that are not immediately available.

**How it works:**
- Set once for the entire WebDriver instance
- Applies to all `findElement()` and `findElements()` calls
- Waits for element to appear (not for specific conditions)
- Throws `NoSuchElementException` if element not found within timeout

**Syntax:**
```java
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
```

### Explicit Wait

**Definition:** Explicit wait waits for a specific condition to be met before proceeding.

**How it works:**
- Applied to specific elements/conditions
- More flexible - can wait for various conditions
- Uses `WebDriverWait` and `ExpectedConditions`
- More precise control over wait conditions

**Syntax:**
```java
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
WebElement element = wait.until(ExpectedConditions.presenceOfElementLocated(By.id("elementId")));
```

### Key Differences

| Aspect | Implicit Wait | Explicit Wait |
|--------|---------------|--------------|
| **Scope** | Applies to all elements | Applies to specific element/condition |
| **Condition** | Only waits for element presence | Waits for various conditions (visibility, clickable, etc.) |
| **Performance** | Slower (waits for every element) | Faster (waits only when needed) |
| **Flexibility** | Less flexible | More flexible |
| **Exception** | NoSuchElementException | TimeoutException |
| **Best Practice** | Not recommended | Recommended |

### When to Use Implicit Wait

**Use Implicit Wait when:**
- ✅ You want a simple, global timeout for all elements
- ✅ Working with stable applications where elements load consistently
- ✅ Quick prototyping or simple test scripts
- ⚠️ **Note:** Generally NOT recommended in production code

**Example:**
```java
// Set implicit wait once
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));

// All findElement calls will wait up to 10 seconds
WebElement element1 = driver.findElement(By.id("element1"));
WebElement element2 = driver.findElement(By.id("element2"));
// Both will wait up to 10 seconds if not found immediately
```

### When to Use Explicit Wait

**Use Explicit Wait when:**
- ✅ Waiting for specific conditions (visibility, clickable, text present, etc.)
- ✅ Different elements need different wait times
- ✅ You want better performance (waits only when needed)
- ✅ Working with dynamic content
- ✅ **Best Practice:** Use explicit wait in production code

**Example:**
```java
// Wait for element to be visible
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
WebElement button = wait.until(
    ExpectedConditions.elementToBeClickable(By.id("submitButton"))
);
button.click();

// Wait for text to be present
wait.until(ExpectedConditions.textToBePresentInElementLocated(
    By.id("status"), "Success"
));
```

### Code Examples

#### **Example 1: Implicit Wait (Not Recommended)**

```java
package tests;

import base.BaseTest;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import org.testng.annotations.Test;
import java.time.Duration;

public class ImplicitWaitExample extends BaseTest {
    
    @Test
    public void testWithImplicitWait() {
        // Set implicit wait - applies to all elements
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
        
        driver.get("https://example.com");
        
        // These will wait up to 10 seconds if elements not found
        WebElement username = driver.findElement(By.id("username"));
        WebElement password = driver.findElement(By.id("password"));
        WebElement loginButton = driver.findElement(By.id("loginButton"));
        
        username.sendKeys("testuser");
        password.sendKeys("password");
        loginButton.click();
        
        // Problem: Even if element is found immediately, 
        // implicit wait still waits (wastes time)
    }
}
```

#### **Example 2: Explicit Wait (Recommended)**

```java
package tests;

import base.BaseTest;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriverWait;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.testng.annotations.Test;
import java.time.Duration;

public class ExplicitWaitExample extends BaseTest {
    
    @Test
    public void testWithExplicitWait() {
        driver.get("https://example.com");
        
        // Create explicit wait instance
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        
        // Wait for element to be visible
        WebElement username = wait.until(
            ExpectedConditions.visibilityOfElementLocated(By.id("username"))
        );
        username.sendKeys("testuser");
        
        // Wait for element to be present
        WebElement password = wait.until(
            ExpectedConditions.presenceOfElementLocated(By.id("password"))
        );
        password.sendKeys("password");
        
        // Wait for element to be clickable
        WebElement loginButton = wait.until(
            ExpectedConditions.elementToBeClickable(By.id("loginButton"))
        );
        loginButton.click();
        
        // Wait for success message
        wait.until(ExpectedConditions.textToBePresentInElementLocated(
            By.id("message"), "Login successful"
        ));
    }
}
```

#### **Example 3: Common Explicit Wait Conditions**

```java
package utils;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.WebDriverWait;
import org.openqa.selenium.support.ui.ExpectedConditions;
import java.time.Duration;
import java.util.List;

public class WaitUtil {
    private WebDriver driver;
    private WebDriverWait wait;
    
    public WaitUtil(WebDriver driver, int timeoutInSeconds) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(timeoutInSeconds));
    }
    
    /**
     * Wait for element to be visible
     */
    public WebElement waitForElementVisible(By locator) {
        return wait.until(ExpectedConditions.visibilityOfElementLocated(locator));
    }
    
    /**
     * Wait for element to be present in DOM
     */
    public WebElement waitForElementPresent(By locator) {
        return wait.until(ExpectedConditions.presenceOfElementLocated(locator));
    }
    
    /**
     * Wait for element to be clickable
     */
    public WebElement waitForElementClickable(By locator) {
        return wait.until(ExpectedConditions.elementToBeClickable(locator));
    }
    
    /**
     * Wait for element to disappear
     */
    public boolean waitForElementInvisible(By locator) {
        return wait.until(ExpectedConditions.invisibilityOfElementLocated(locator));
    }
    
    /**
     * Wait for text to be present in element
     */
    public boolean waitForTextPresent(By locator, String text) {
        return wait.until(ExpectedConditions.textToBePresentInElementLocated(locator, text));
    }
    
    /**
     * Wait for URL to contain text
     */
    public boolean waitForUrlContains(String text) {
        return wait.until(ExpectedConditions.urlContains(text));
    }
    
    /**
     * Wait for title to be
     */
    public boolean waitForTitle(String title) {
        return wait.until(ExpectedConditions.titleIs(title));
    }
    
    /**
     * Wait for number of elements
     */
    public List<WebElement> waitForNumberOfElements(By locator, int count) {
        return wait.until(ExpectedConditions.numberOfElementsToBe(locator, count));
    }
    
    /**
     * Wait for element to be selected
     */
    public boolean waitForElementSelected(By locator) {
        return wait.until(ExpectedConditions.elementToBeSelected(locator));
    }
    
    /**
     * Wait for alert to be present
     */
    public org.openqa.selenium.Alert waitForAlert() {
        return wait.until(ExpectedConditions.alertIsPresent());
    }
}
```

#### **Example 4: Best Practice - Combining Both (Not Recommended)**

```java
// ❌ BAD PRACTICE: Using both together
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10)); // Implicit
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(15)); // Explicit

// Problem: Implicit wait adds to explicit wait timeout
// Total wait time = 15 (explicit) + 10 (implicit) = 25 seconds potentially
```

#### **Example 5: Best Practice - Explicit Wait Only**

```java
// ✅ GOOD PRACTICE: Use explicit wait only, disable implicit wait
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(0)); // Disable implicit

WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
WebElement element = wait.until(
    ExpectedConditions.visibilityOfElementLocated(By.id("element"))
);
```

### Scenario-Based Examples

#### **Scenario 1: Dynamic Content Loading**

**Problem:** Page loads, but content appears after AJAX call completes.

**Solution with Explicit Wait:**
```java
@Test
public void testDynamicContent() {
    driver.get("https://example.com/products");
    
    // Click filter button
    driver.findElement(By.id("filterButton")).click();
    
    // Wait for products to load dynamically
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(15));
    
    // Wait for product list to appear
    wait.until(ExpectedConditions.presenceOfElementLocated(By.className("product-list")));
    
    // Wait for specific number of products
    wait.until(ExpectedConditions.numberOfElementsToBe(
        By.className("product-item"), 10
    ));
    
    // Now interact with products
    List<WebElement> products = driver.findElements(By.className("product-item"));
    Assert.assertEquals(products.size(), 10);
}
```

#### **Scenario 2: Form Submission with Success Message**

**Problem:** After form submission, success message appears after processing.

**Solution:**
```java
@Test
public void testFormSubmission() {
    driver.get("https://example.com/contact");
    
    // Fill form
    driver.findElement(By.id("name")).sendKeys("John Doe");
    driver.findElement(By.id("email")).sendKeys("john@example.com");
    driver.findElement(By.id("message")).sendKeys("Test message");
    
    // Submit form
    driver.findElement(By.id("submitButton")).click();
    
    // Wait for success message to appear
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    WebElement successMessage = wait.until(
        ExpectedConditions.visibilityOfElementLocated(By.id("successMessage"))
    );
    
    // Verify message text
    Assert.assertEquals(successMessage.getText(), "Thank you! Your message has been sent.");
}
```

#### **Scenario 3: Page Navigation**

**Problem:** After clicking link, need to wait for new page to load.

**Solution:**
```java
@Test
public void testPageNavigation() {
    driver.get("https://example.com");
    
    // Click link
    driver.findElement(By.linkText("About Us")).click();
    
    // Wait for URL to change
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    wait.until(ExpectedConditions.urlContains("/about"));
    
    // Wait for page title
    wait.until(ExpectedConditions.titleIs("About Us - Example.com"));
    
    // Verify page loaded
    WebElement heading = wait.until(
        ExpectedConditions.visibilityOfElementLocated(By.tagName("h1"))
    );
    Assert.assertEquals(heading.getText(), "About Us");
}
```

### Best Practices

1. **Use Explicit Wait as Default**
   - More flexible and performant
   - Better control over wait conditions

2. **Disable Implicit Wait**
   - Set to 0 when using explicit waits
   - Prevents unexpected long waits

3. **Use Appropriate Timeouts**
   - Short timeouts (5-10 seconds) for fast operations
   - Longer timeouts (30+ seconds) for slow operations

4. **Wait for Specific Conditions**
   - `visibilityOfElementLocated` - Element is visible
   - `elementToBeClickable` - Element can be clicked
   - `presenceOfElementLocated` - Element exists in DOM

5. **Create Reusable Wait Utilities**
   - Centralize wait logic
   - Easy to maintain and update

6. **Handle TimeoutException**
   - Catch and handle timeouts gracefully
   - Provide meaningful error messages

### Summary

**Use Implicit Wait:**
- ⚠️ Rarely - only for simple scripts
- Not recommended for production code

**Use Explicit Wait:**
- ✅ Always - recommended approach
- More flexible and performant
- Better control over conditions

**Best Practice:**
```java
// Disable implicit wait
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(0));

// Use explicit wait for specific conditions
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
wait.until(ExpectedConditions.elementToBeClickable(By.id("button")));
```

---

## 6. What is a NullPointerException in Java?

### Context
This question tests your understanding of Java fundamentals, exception handling, and common programming errors. NullPointerException is one of the most common runtime exceptions in Java.

### Detailed Answer

**NullPointerException (NPE)** is a runtime exception that occurs when you try to use a reference variable that has a `null` value, meaning it doesn't point to any object in memory.

### What Causes NullPointerException?

A NullPointerException is thrown when:
1. Calling a method on a `null` object
2. Accessing or modifying a field of a `null` object
3. Taking the length of a `null` array
4. Accessing or modifying slots of a `null` array
5. Throwing `null` as if it were a Throwable value

### Common Scenarios

#### **Scenario 1: Calling Method on Null Object**

```java
String str = null;
int length = str.length(); // NullPointerException
```

#### **Scenario 2: Accessing Array Element**

```java
String[] array = null;
String first = array[0]; // NullPointerException
```

#### **Scenario 3: Chained Method Calls**

```java
Person person = null;
String name = person.getName().toUpperCase(); // NullPointerException
```

#### **Scenario 4: Uninitialized Object**

```java
WebElement element = null;
element.click(); // NullPointerException
```

### Code Examples

#### **Example 1: Basic NullPointerException**

```java
public class NullPointerExample {
    public static void main(String[] args) {
        String text = null;
        
        // This will throw NullPointerException
        try {
            int length = text.length();
            System.out.println("Length: " + length);
        } catch (NullPointerException e) {
            System.out.println("NullPointerException caught: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

#### **Example 2: Null Check Before Use**

```java
public class NullCheckExample {
    public static void processString(String str) {
        // Check for null before using
        if (str != null) {
            System.out.println("String length: " + str.length());
            System.out.println("String uppercase: " + str.toUpperCase());
        } else {
            System.out.println("String is null, cannot process");
        }
    }
    
    public static void main(String[] args) {
        String validString = "Hello World";
        String nullString = null;
        
        processString(validString); // Works fine
        processString(nullString);  // Handles null gracefully
    }
}
```

#### **Example 3: Using Optional (Java 8+)**

```java
import java.util.Optional;

public class OptionalExample {
    public static void processString(Optional<String> str) {
        // Using Optional to handle null safely
        str.ifPresent(s -> {
            System.out.println("String length: " + s.length());
            System.out.println("String: " + s.toUpperCase());
        });
        
        // Or with default value
        String result = str.orElse("Default Value");
        System.out.println("Result: " + result);
    }
    
    public static void main(String[] args) {
        Optional<String> validString = Optional.of("Hello");
        Optional<String> nullString = Optional.empty();
        
        processString(validString);
        processString(nullString);
    }
}
```

#### **Example 4: Null Safety in Selenium**

```java
package utils;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.By;
import org.openqa.selenium.NoSuchElementException;

public class SafeElementFinder {
    private WebDriver driver;
    
    public SafeElementFinder(WebDriver driver) {
        this.driver = driver;
    }
    
    /**
     * Safely find element with null check
     */
    public WebElement findElementSafely(By locator) {
        try {
            WebElement element = driver.findElement(locator);
            if (element == null) {
                throw new NoSuchElementException("Element not found: " + locator);
            }
            return element;
        } catch (NoSuchElementException e) {
            System.out.println("Element not found: " + locator);
            return null;
        }
    }
    
    /**
     * Safely click element with null check
     */
    public void clickSafely(By locator) {
        WebElement element = findElementSafely(locator);
        if (element != null) {
            element.click();
        } else {
            throw new RuntimeException("Cannot click null element: " + locator);
        }
    }
    
    /**
     * Safely get text with null check
     */
    public String getTextSafely(By locator) {
        WebElement element = findElementSafely(locator);
        if (element != null) {
            return element.getText();
        }
        return ""; // Return empty string instead of null
    }
}
```

#### **Example 5: Null Safety in Test Methods**

```java
package tests;

import base.BaseTest;
import org.openqa.selenium.WebElement;
import org.testng.Assert;
import org.testng.annotations.Test;

public class NullSafetyTest extends BaseTest {
    
    @Test
    public void testWithNullCheck() {
        driver.get("https://example.com");
        
        // Find element - might be null
        WebElement element = null;
        try {
            element = driver.findElement(By.id("dynamicElement"));
        } catch (Exception e) {
            // Element not found
        }
        
        // Check for null before using
        if (element != null) {
            String text = element.getText();
            Assert.assertNotNull(text, "Element text should not be null");
        } else {
            System.out.println("Element not found, skipping test");
            // Or fail the test
            Assert.fail("Element not found");
        }
    }
    
    @Test
    public void testSafeElementAccess() {
        driver.get("https://example.com");
        
        // Safe way: Check if element exists before accessing
        List<WebElement> elements = driver.findElements(By.className("item"));
        
        if (elements != null && !elements.isEmpty()) {
            for (WebElement element : elements) {
                if (element != null) {
                    String text = element.getText();
                    if (text != null) {
                        System.out.println("Text: " + text);
                    }
                }
            }
        } else {
            System.out.println("No elements found");
        }
    }
}
```

### Prevention Strategies

#### **1. Null Checks**

```java
if (object != null) {
    object.method();
}
```

#### **2. Early Returns**

```java
public void processData(String data) {
    if (data == null) {
        return; // Early return
    }
    // Process data
    data.toUpperCase();
}
```

#### **3. Default Values**

```java
String result = (str != null) ? str : "Default";
```

#### **4. Optional (Java 8+)**

```java
Optional<String> optional = Optional.ofNullable(str);
String result = optional.orElse("Default");
```

#### **5. Assertions**

```java
assert object != null : "Object cannot be null";
object.method();
```

#### **6. Validators**

```java
public class Validator {
    public static void requireNonNull(Object obj, String message) {
        if (obj == null) {
            throw new IllegalArgumentException(message);
        }
    }
}

// Usage
Validator.requireNonNull(element, "Element cannot be null");
element.click();
```

### Scenario-Based Example

**Real Scenario: Handling Dynamic Elements in Selenium**

```java
@Test
public void testDynamicElementHandling() {
    driver.get("https://example.com");
    
    // Element might not exist initially
    WebElement popup = null;
    
    try {
        // Try to find popup (might be null)
        popup = driver.findElement(By.id("popup"));
    } catch (NoSuchElementException e) {
        // Popup doesn't exist, that's okay
        System.out.println("Popup not present");
    }
    
    // Safe handling
    if (popup != null && popup.isDisplayed()) {
        // Close popup if present
        WebElement closeButton = popup.findElement(By.className("close"));
        if (closeButton != null) {
            closeButton.click();
        }
    }
    
    // Continue with test
    WebElement mainContent = driver.findElement(By.id("mainContent"));
    Assert.assertNotNull(mainContent, "Main content should exist");
}
```

### Best Practices

1. **Always Check for Null**
   - Before calling methods on objects
   - Before accessing array elements

2. **Use Optional for Return Values**
   - Makes null handling explicit
   - Prevents accidental NPE

3. **Fail Fast**
   - Validate inputs early
   - Throw meaningful exceptions

4. **Use Assertions**
   - Document assumptions
   - Catch errors early in development

5. **Avoid Returning Null**
   - Return empty collections instead
   - Use Optional for nullable returns

6. **Use Null-Safe Libraries**
   - Apache Commons StringUtils
   - Google Guava Optional

### Summary

**NullPointerException occurs when:**
- Calling methods on `null` objects
- Accessing fields of `null` objects
- Using `null` arrays

**Prevention:**
- ✅ Always check for null before use
- ✅ Use Optional for nullable values
- ✅ Provide default values
- ✅ Fail fast with meaningful errors
- ✅ Use null-safe coding practices

---

## 7. Difference between driver.get() and driver.navigate().to()

### Context
This question tests your understanding of WebDriver navigation methods and when to use each. Both methods navigate to a URL, but they have subtle differences.

### Detailed Answer

Both `driver.get()` and `driver.navigate().to()` are used to navigate to a URL, but they have some differences in behavior and usage.

### driver.get()

**Definition:** Loads a new web page in the current browser window. This is a blocking call that waits for the page to load completely.

**Characteristics:**
- Simpler syntax
- Waits for page load
- Most commonly used
- Cannot go back/forward
- Resets browser history

**Syntax:**
```java
driver.get("https://example.com");
```

### driver.navigate().to()

**Definition:** Navigates to a URL. Part of the Navigation interface, which provides browser history navigation capabilities.

**Characteristics:**
- Part of Navigation interface
- Can navigate back/forward
- More flexible
- Same functionality as `get()` for basic navigation
- Maintains browser history

**Syntax:**
```java
driver.navigate().to("https://example.com");
```

### Key Differences

| Aspect | driver.get() | driver.navigate().to() |
|--------|--------------|------------------------|
| **Syntax** | Simpler | More verbose |
| **Return Type** | void | Navigation object (fluent) |
| **Browser History** | Resets | Maintains |
| **Back/Forward** | No | Yes (via navigate()) |
| **Usage** | Most common | Less common |
| **Functionality** | Same for basic navigation | Same + history control |

### Code Examples

#### **Example 1: Basic Usage**

```java
package tests;

import base.BaseTest;
import org.testng.annotations.Test;

public class NavigationTest extends BaseTest {
    
    @Test
    public void testDriverGet() {
        // Using driver.get() - simpler
        driver.get("https://example.com");
        
        // Navigate to another page
        driver.get("https://example.com/products");
    }
    
    @Test
    public void testNavigateTo() {
        // Using driver.navigate().to() - more verbose
        driver.navigate().to("https://example.com");
        
        // Navigate to another page
        driver.navigate().to("https://example.com/products");
    }
}
```

#### **Example 2: Browser History Navigation**

```java
@Test
public void testBrowserHistory() {
    // Navigate using navigate().to() to maintain history
    driver.navigate().to("https://example.com");
    System.out.println("Current URL: " + driver.getCurrentUrl()); // example.com
    
    driver.navigate().to("https://example.com/products");
    System.out.println("Current URL: " + driver.getCurrentUrl()); // products
    
    driver.navigate().to("https://example.com/about");
    System.out.println("Current URL: " + driver.getCurrentUrl()); // about
    
    // Go back
    driver.navigate().back();
    System.out.println("After back: " + driver.getCurrentUrl()); // products
    
    // Go forward
    driver.navigate().forward();
    System.out.println("After forward: " + driver.getCurrentUrl()); // about
    
    // Refresh page
    driver.navigate().refresh();
    System.out.println("After refresh: " + driver.getCurrentUrl()); // about
}
```

#### **Example 3: Navigation Interface Methods**

```java
package utils;

import org.openqa.selenium.WebDriver;

public class NavigationUtil {
    private WebDriver driver;
    
    public NavigationUtil(WebDriver driver) {
        this.driver = driver;
    }
    
    /**
     * Navigate to URL
     */
    public void navigateTo(String url) {
        driver.navigate().to(url);
    }
    
    /**
     * Go back in browser history
     */
    public void goBack() {
        driver.navigate().back();
    }
    
    /**
     * Go forward in browser history
     */
    public void goForward() {
        driver.navigate().forward();
    }
    
    /**
     * Refresh current page
     */
    public void refresh() {
        driver.navigate().refresh();
    }
    
    /**
     * Navigate back and verify URL
     */
    public void navigateBackAndVerify(String expectedUrl) {
        driver.navigate().back();
        String currentUrl = driver.getCurrentUrl();
        assert currentUrl.equals(expectedUrl) : 
            "Expected: " + expectedUrl + ", Got: " + currentUrl;
    }
}
```

#### **Example 4: Practical Usage Scenario**

```java
@Test
public void testEcommerceNavigation() {
    // Start at homepage
    driver.get("https://ecommerce.example.com");
    
    // Navigate to products (maintains history)
    driver.navigate().to("https://ecommerce.example.com/products");
    
    // Navigate to specific product
    driver.navigate().to("https://ecommerce.example.com/products/laptop");
    
    // Go back to products page
    driver.navigate().back();
    Assert.assertTrue(driver.getCurrentUrl().contains("/products"));
    
    // Go forward to product page again
    driver.navigate().forward();
    Assert.assertTrue(driver.getCurrentUrl().contains("/laptop"));
    
    // Refresh product page
    driver.navigate().refresh();
    
    // Go back to homepage
    driver.navigate().back();
    driver.navigate().back();
    Assert.assertTrue(driver.getCurrentUrl().equals("https://ecommerce.example.com/"));
}
```

### When to Use Which?

#### **Use driver.get() when:**
- ✅ Simple navigation to a URL
- ✅ Starting a new test flow
- ✅ Don't need browser history
- ✅ Most common use case

**Example:**
```java
@Test
public void testLogin() {
    driver.get("https://example.com/login");
    // Test login functionality
}
```

#### **Use driver.navigate().to() when:**
- ✅ Need browser history navigation
- ✅ Want to go back/forward
- ✅ Testing navigation flows
- ✅ Need refresh functionality

**Example:**
```java
@Test
public void testNavigationFlow() {
    driver.navigate().to("https://example.com");
    driver.navigate().to("https://example.com/products");
    driver.navigate().back(); // Go back
    driver.navigate().forward(); // Go forward
    driver.navigate().refresh(); // Refresh
}
```

### Important Notes

1. **Functionally Equivalent for Basic Navigation**
   ```java
   // These do the same thing
   driver.get("https://example.com");
   driver.navigate().to("https://example.com");
   ```

2. **Browser History**
   - `driver.get()` doesn't maintain history for back/forward
   - `driver.navigate().to()` maintains history

3. **Return Value**
   - `driver.get()` returns void
   - `driver.navigate().to()` returns Navigation object (allows chaining)

4. **Performance**
   - Both have same performance
   - Both wait for page load

### Scenario-Based Example

**Real Scenario: Testing multi-page form with back navigation**

```java
@Test
public void testMultiPageForm() {
    // Step 1: Start form
    driver.navigate().to("https://example.com/form/step1");
    driver.findElement(By.id("name")).sendKeys("John Doe");
    driver.findElement(By.id("next")).click();
    
    // Step 2: Address page
    driver.navigate().to("https://example.com/form/step2");
    driver.findElement(By.id("address")).sendKeys("123 Main St");
    
    // User clicks back button
    driver.navigate().back();
    
    // Verify we're back at step 1
    Assert.assertTrue(driver.getCurrentUrl().contains("step1"));
    Assert.assertEquals(
        driver.findElement(By.id("name")).getAttribute("value"), 
        "John Doe"
    );
    
    // Continue forward
    driver.navigate().forward();
    Assert.assertTrue(driver.getCurrentUrl().contains("step2"));
    
    // Complete form
    driver.findElement(By.id("submit")).click();
}
```

### Best Practices

1. **Use driver.get() for Simple Navigation**
   - Most common use case
   - Cleaner syntax

2. **Use driver.navigate().to() for History**
   - When testing back/forward functionality
   - When need refresh capability

3. **Be Consistent**
   - Use same method throughout test suite
   - Improves code readability

4. **Handle Navigation Errors**
   - Wrap in try-catch if needed
   - Verify URL after navigation

### Summary

**driver.get():**
- ✅ Simpler syntax
- ✅ Most commonly used
- ✅ Good for basic navigation

**driver.navigate().to():**
- ✅ Maintains browser history
- ✅ Allows back/forward/refresh
- ✅ Good for navigation testing

**Key Point:** For basic navigation, both are equivalent. Use `navigate().to()` when you need browser history functionality.

---

## 8. How do you handle the location permission popup in Selenium?

### Context
This question tests your knowledge of browser capabilities, ChromeOptions, and handling browser-specific features like permission popups that can't be handled with standard Selenium methods.

### Detailed Answer

Location permission popups are browser-native dialogs that appear when a website requests location access. These cannot be handled with standard Selenium `Alert` handling. Instead, we use browser-specific options to pre-configure permissions.

### Approaches

#### **Approach 1: Using ChromeOptions (Chrome/Edge)**

```java
package utils;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.edge.EdgeDriver;
import org.openqa.selenium.edge.EdgeOptions;

public class LocationPermissionHandler {
    
    /**
     * Create Chrome driver with location permission allowed
     */
    public static WebDriver createChromeDriverWithLocationAllowed() {
        ChromeOptions options = new ChromeOptions();
        
        // Allow location permission
        options.addArguments("--enable-features=Geolocation");
        
        // Set location preference
        options.addArguments("--use-fake-ui-for-media-stream");
        options.addArguments("--use-fake-device-for-media-stream");
        
        // Or use preferences
        Map<String, Object> prefs = new HashMap<>();
        prefs.put("profile.default_content_setting_values.geolocation", 1); // 1 = Allow, 2 = Block
        options.setExperimentalOption("prefs", prefs);
        
        return new ChromeDriver(options);
    }
    
    /**
     * Create Chrome driver with location permission blocked
     */
    public static WebDriver createChromeDriverWithLocationBlocked() {
        ChromeOptions options = new ChromeOptions();
        
        Map<String, Object> prefs = new HashMap<>();
        prefs.put("profile.default_content_setting_values.geolocation", 2); // 2 = Block
        options.setExperimentalOption("prefs", prefs);
        
        return new ChromeDriver(options);
    }
    
    /**
     * Create Chrome driver with custom location coordinates
     */
    public static WebDriver createChromeDriverWithCustomLocation(double latitude, double longitude) {
        ChromeOptions options = new ChromeOptions();
        
        // Set fake geolocation
        Map<String, Object> prefs = new HashMap<>();
        prefs.put("profile.default_content_setting_values.geolocation", 1);
        options.setExperimentalOption("prefs", prefs);
        
        // Set geolocation coordinates
        Map<String, Object> geoLocation = new HashMap<>();
        geoLocation.put("latitude", latitude);
        geoLocation.put("longitude", longitude);
        geoLocation.put("accuracy", 100);
        
        options.setExperimentalOption("preolocation", geoLocation);
        
        return new ChromeDriver(options);
    }
}
```

#### **Approach 2: Complete Implementation**

```java
package utils;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.firefox.FirefoxDriver;
import org.openqa.selenium.firefox.FirefoxOptions;
import org.openqa.selenium.firefox.FirefoxProfile;
import java.util.HashMap;
import java.util.Map;

public class BrowserPermissionHandler {
    
    /**
     * Chrome: Allow location permission
     */
    public static ChromeOptions getChromeOptionsWithLocationAllowed() {
        ChromeOptions options = new ChromeOptions();
        
        // Method 1: Using preferences
        Map<String, Object> prefs = new HashMap<>();
        prefs.put("profile.default_content_setting_values.geolocation", 1);
        options.setExperimentalOption("prefs", prefs);
        
        // Method 2: Using arguments
        options.addArguments("--enable-features=Geolocation");
        
        return options;
    }
    
    /**
     * Chrome: Block location permission
     */
    public static ChromeOptions getChromeOptionsWithLocationBlocked() {
        ChromeOptions options = new ChromeOptions();
        
        Map<String, Object> prefs = new HashMap<>();
        prefs.put("profile.default_content_setting_values.geolocation", 2);
        options.setExperimentalOption("prefs", prefs);
        
        return options;
    }
    
    /**
     * Chrome: Set custom location coordinates
     */
    public static ChromeOptions getChromeOptionsWithCustomLocation(
            double latitude, double longitude) {
        ChromeOptions options = new ChromeOptions();
        
        // Allow geolocation
        Map<String, Object> prefs = new HashMap<>();
        prefs.put("profile.default_content_setting_values.geolocation", 1);
        options.setExperimentalOption("prefs", prefs);
        
        // Set custom coordinates
        Map<String, Object> geoLocation = new HashMap<>();
        geoLocation.put("latitude", latitude);
        geoLocation.put("longitude", longitude);
        geoLocation.put("accuracy", 100);
        
        options.setExperimentalOption("preolocation", geoLocation);
        
        return options;
    }
    
    /**
     * Firefox: Allow location permission
     */
    public static FirefoxOptions getFirefoxOptionsWithLocationAllowed() {
        FirefoxOptions options = new FirefoxOptions();
        FirefoxProfile profile = new FirefoxProfile();
        
        // Set preference to allow geolocation
        profile.setPreference("geo.enabled", true);
        profile.setPreference("geo.prompt.testing", true);
        profile.setPreference("geo.prompt.testing.allow", true);
        
        options.setProfile(profile);
        return options;
    }
    
    /**
     * Firefox: Set custom location
     */
    public static FirefoxOptions getFirefoxOptionsWithCustomLocation(
            double latitude, double longitude) {
        FirefoxOptions options = new FirefoxOptions();
        FirefoxProfile profile = new FirefoxProfile();
        
        profile.setPreference("geo.enabled", true);
        profile.setPreference("geo.prompt.testing", true);
        profile.setPreference("geo.prompt.testing.allow", true);
        
        // Set custom coordinates
        profile.setPreference("geo.wifi.uri", 
            String.format("data:application/json,{\"location\":{\"lat\":%f,\"lng\":%f},\"accuracy\":100}", 
                latitude, longitude));
        
        options.setProfile(profile);
        return options;
    }
}
```

#### **Approach 3: Using in Test Classes**

```java
package tests;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import utils.BrowserPermissionHandler;
import java.util.HashMap;
import java.util.Map;

public class LocationPermissionTest {
    private WebDriver driver;
    
    @BeforeMethod
    public void setUp() {
        // Create driver with location permission allowed
        ChromeOptions options = BrowserPermissionHandler
            .getChromeOptionsWithLocationAllowed();
        driver = new ChromeDriver(options);
    }
    
    @Test
    public void testWithLocationAllowed() {
        driver.get("https://example.com/location-based-service");
        
        // Location permission popup won't appear
        // Website can access location directly
        
        // Verify location is accessible
        // (depends on website implementation)
    }
    
    @Test
    public void testWithCustomLocation() {
        // Set custom location (e.g., New York)
        ChromeOptions options = BrowserPermissionHandler
            .getChromeOptionsWithCustomLocation(40.7128, -74.0060);
        driver = new ChromeDriver(options);
        
        driver.get("https://example.com/location-based-service");
        
        // Website will use the custom coordinates
    }
    
    @AfterMethod
    public void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

#### **Approach 4: Using Chrome DevTools Protocol (Advanced)**

```java
package utils;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.devtools.DevTools;
import org.openqa.selenium.devtools.v85.emulation.Emulation;
import java.util.Optional;

public class AdvancedLocationHandler {
    
    /**
     * Set location using Chrome DevTools Protocol
     */
    public static void setLocationUsingDevTools(WebDriver driver, 
                                                double latitude, 
                                                double longitude) {
        if (driver instanceof ChromeDriver) {
            ChromeDriver chromeDriver = (ChromeDriver) driver;
            DevTools devTools = chromeDriver.getDevTools();
            devTools.createSession();
            
            // Set geolocation
            devTools.send(Emulation.setGeolocationOverride(
                Optional.of(latitude),
                Optional.of(longitude),
                Optional.of(100)
            ));
        }
    }
}
```

### Complete Example: Base Test Class

```java
package base;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import java.util.HashMap;
import java.util.Map;

public class BaseTestWithLocation {
    protected WebDriver driver;
    
    @BeforeMethod
    public void setUp() {
        ChromeOptions options = new ChromeOptions();
        
        // Configure location permission
        Map<String, Object> prefs = new HashMap<>();
        prefs.put("profile.default_content_setting_values.geolocation", 1); // Allow
        
        options.setExperimentalOption("prefs", prefs);
        
        // Optional: Set custom location
        Map<String, Object> geoLocation = new HashMap<>();
        geoLocation.put("latitude", 40.7128); // New York
        geoLocation.put("longitude", -74.0060);
        geoLocation.put("accuracy", 100);
        options.setExperimentalOption("preolocation", geoLocation);
        
        driver = new ChromeDriver(options);
        driver.manage().window().maximize();
    }
    
    @AfterMethod
    public void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

### Permission Values

**Chrome Permission Values:**
- `0` - Default/Ask
- `1` - Allow
- `2` - Block

**Common Permissions:**
```java
Map<String, Object> prefs = new HashMap<>();
prefs.put("profile.default_content_setting_values.geolocation", 1); // Location
prefs.put("profile.default_content_setting_values.notifications", 1); // Notifications
prefs.put("profile.default_content_setting_values.media_stream", 1); // Camera/Mic
prefs.put("profile.default_content_setting_values.camera", 1); // Camera
prefs.put("profile.default_content_setting_values.microphone", 1); // Microphone
```

### Scenario-Based Example

**Real Scenario: Testing location-based restaurant finder**

```java
@Test
public void testRestaurantFinderWithLocation() {
    // Setup driver with location allowed and custom coordinates
    ChromeOptions options = new ChromeOptions();
    
    Map<String, Object> prefs = new HashMap<>();
    prefs.put("profile.default_content_setting_values.geolocation", 1);
    options.setExperimentalOption("prefs", prefs);
    
    // Set location to San Francisco
    Map<String, Object> geoLocation = new HashMap<>();
    geoLocation.put("latitude", 37.7749);
    geoLocation.put("longitude", -122.4194);
    geoLocation.put("accuracy", 100);
    options.setExperimentalOption("preolocation", geoLocation);
    
    WebDriver driver = new ChromeDriver(options);
    
    try {
        driver.get("https://restaurant-finder.example.com");
        
        // Location permission popup won't appear
        // Website can access location directly
        
        // Verify restaurants near San Francisco are shown
        WebElement restaurantList = driver.findElement(By.id("restaurantList"));
        Assert.assertTrue(restaurantList.isDisplayed());
        
        // Verify location is set correctly
        WebElement locationDisplay = driver.findElement(By.id("currentLocation"));
        Assert.assertTrue(locationDisplay.getText().contains("San Francisco"));
        
    } finally {
        driver.quit();
    }
}
```

### Best Practices

1. **Configure Before Driver Creation**
   - Set permissions in ChromeOptions before creating driver
   - Cannot change permissions after driver is created

2. **Use Custom Coordinates for Testing**
   - Set specific coordinates for consistent test results
   - Useful for testing location-based features

3. **Handle Different Browsers**
   - Chrome: Use ChromeOptions and preferences
   - Firefox: Use FirefoxProfile and preferences
   - Edge: Similar to Chrome

4. **Test Both Allowed and Blocked**
   - Test with permission allowed
   - Test with permission blocked
   - Verify application handles both cases

5. **Document Location Settings**
   - Document which coordinates are used
   - Helps with test reproducibility

### Summary

**To handle location permission popup:**

1. **Chrome:**
   ```java
   ChromeOptions options = new ChromeOptions();
   Map<String, Object> prefs = new HashMap<>();
   prefs.put("profile.default_content_setting_values.geolocation", 1);
   options.setExperimentalOption("prefs", prefs);
   WebDriver driver = new ChromeDriver(options);
   ```

2. **Firefox:**
   ```java
   FirefoxOptions options = new FirefoxOptions();
   FirefoxProfile profile = new FirefoxProfile();
   profile.setPreference("geo.enabled", true);
   profile.setPreference("geo.prompt.testing.allow", true);
   options.setProfile(profile);
   WebDriver driver = new FirefoxDriver(options);
   ```

**Key Points:**
- ✅ Configure permissions before creating driver
- ✅ Cannot handle popup with Alert (it's browser-native)
- ✅ Use browser-specific options
- ✅ Can set custom coordinates for testing

---

## 9. How do you handle dynamic dropdowns in Selenium?

### Context
This question tests your ability to handle dynamic web elements that change based on user interaction or AJAX calls. Dynamic dropdowns are common in modern web applications.

### Detailed Answer

Dynamic dropdowns are dropdown menus where options are loaded dynamically (via AJAX, user input, or other interactions). They require special handling compared to static dropdowns.

### Types of Dynamic Dropdowns

1. **AJAX-loaded options** - Options load after page load
2. **Cascading dropdowns** - Options change based on previous selection
3. **Searchable dropdowns** - Options filter as you type
4. **Auto-complete dropdowns** - Suggestions appear as you type

### Handling Approaches

#### **Approach 1: Using Explicit Wait**

```java
package utils;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriverWait;
import org.openqa.selenium.support.ui.ExpectedConditions;
import java.time.Duration;
import java.util.List;

public class DynamicDropdownHandler {
    private WebDriver driver;
    private WebDriverWait wait;
    
    public DynamicDropdownHandler(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    }
    
    /**
     * Handle AJAX-loaded dropdown
     */
    public void selectFromAjaxDropdown(String dropdownId, String optionText) {
        // Click dropdown to trigger AJAX load
        WebElement dropdown = wait.until(
            ExpectedConditions.elementToBeClickable(By.id(dropdownId))
        );
        dropdown.click();
        
        // Wait for options to load
        wait.until(ExpectedConditions.presenceOfElementLocated(
            By.xpath("//ul[@class='dropdown-menu']/li")
        ));
        
        // Select option
        WebElement option = wait.until(
            ExpectedConditions.elementToBeClickable(
                By.xpath("//ul[@class='dropdown-menu']/li[text()='" + optionText + "']")
            )
        );
        option.click();
    }
    
    /**
     * Handle cascading dropdown (e.g., Country -> State -> City)
     */
    public void selectFromCascadingDropdown(String firstDropdownId, 
                                           String firstOption,
                                           String secondDropdownId,
                                           String secondOption) {
        // Select first dropdown
        selectFromAjaxDropdown(firstDropdownId, firstOption);
        
        // Wait for second dropdown to populate
        wait.until(ExpectedConditions.presenceOfElementLocated(
            By.id(secondDropdownId)
        ));
        
        // Select second dropdown
        selectFromAjaxDropdown(secondDropdownId, secondOption);
    }
    
    /**
     * Handle searchable dropdown
     */
    public void selectFromSearchableDropdown(String dropdownId, 
                                            String searchText,
                                            String optionText) {
        // Click dropdown
        WebElement dropdown = driver.findElement(By.id(dropdownId));
        dropdown.click();
        
        // Type search text
        WebElement searchInput = wait.until(
            ExpectedConditions.visibilityOfElementLocated(
                By.xpath("//input[@class='dropdown-search']")
            )
        );
        searchInput.sendKeys(searchText);
        
        // Wait for filtered options
        wait.until(ExpectedConditions.presenceOfElementLocated(
            By.xpath("//ul[@class='dropdown-menu']/li[contains(text(),'" + searchText + "')]")
        ));
        
        // Select option
        WebElement option = wait.until(
            ExpectedConditions.elementToBeClickable(
                By.xpath("//ul[@class='dropdown-menu']/li[text()='" + optionText + "']")
            )
        );
        option.click();
    }
}
```

#### **Approach 2: Using Select Class with Wait**

```java
package utils;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.By;
import org.openqa.selenium.support.ui.Select;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.openqa.selenium.support.ui.ExpectedConditions;
import java.time.Duration;
import java.util.List;

public class SelectDropdownHandler {
    private WebDriver driver;
    private WebDriverWait wait;
    
    public SelectDropdownHandler(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    }
    
    /**
     * Select from dynamic dropdown using Select class
     */
    public void selectByVisibleText(String dropdownId, String optionText) {
        // Wait for dropdown to be present
        WebElement dropdownElement = wait.until(
            ExpectedConditions.presenceOfElementLocated(By.id(dropdownId))
        );
        
        Select select = new Select(dropdownElement);
        
        // Wait for options to load
        wait.until(driver -> {
            List<WebElement> options = select.getOptions();
            return options.size() > 1; // More than just default option
        });
        
        // Select option
        select.selectByVisibleText(optionText);
    }
    
    /**
     * Select by value with wait
     */
    public void selectByValue(String dropdownId, String value) {
        WebElement dropdownElement = wait.until(
            ExpectedConditions.presenceOfElementLocated(By.id(dropdownId))
        );
        
        Select select = new Select(dropdownElement);
        
        // Wait for specific value to be available
        wait.until(driver -> {
            try {
                select.selectByValue(value);
                return true;
            } catch (Exception e) {
                return false;
            }
        });
    }
    
    /**
     * Select by index with wait
     */
    public void selectByIndex(String dropdownId, int index) {
        WebElement dropdownElement = wait.until(
            ExpectedConditions.presenceOfElementLocated(By.id(dropdownId))
        );
        
        Select select = new Select(dropdownElement);
        
        // Wait for enough options
        wait.until(driver -> select.getOptions().size() > index);
        
        select.selectByIndex(index);
    }
}
```

#### **Approach 3: Custom Dropdown Handler**

```java
package utils;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriverWait;
import org.openqa.selenium.support.ui.ExpectedConditions;
import java.time.Duration;
import java.util.List;

public class CustomDropdownHandler {
    private WebDriver driver;
    private WebDriverWait wait;
    
    public CustomDropdownHandler(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(15));
    }
    
    /**
     * Select option from custom dropdown (not using <select> tag)
     */
    public void selectFromCustomDropdown(String dropdownLocator, 
                                        String optionText) {
        // Click dropdown to open
        WebElement dropdown = wait.until(
            ExpectedConditions.elementToBeClickable(By.xpath(dropdownLocator))
        );
        dropdown.click();
        
        // Wait for dropdown menu to appear
        String menuXpath = "//div[@class='dropdown-menu' or @class='select-options']";
        wait.until(ExpectedConditions.visibilityOfElementLocated(By.xpath(menuXpath)));
        
        // Wait for options to load
        String optionXpath = menuXpath + "//li[text()='" + optionText + "']";
        WebElement option = wait.until(
            ExpectedConditions.elementToBeClickable(By.xpath(optionXpath))
        );
        option.click();
        
        // Wait for dropdown to close
        wait.until(ExpectedConditions.invisibilityOfElementLocated(By.xpath(menuXpath)));
    }
    
    /**
     * Select option by partial text match
     */
    public void selectByPartialText(String dropdownLocator, String partialText) {
        WebElement dropdown = wait.until(
            ExpectedConditions.elementToBeClickable(By.xpath(dropdownLocator))
        );
        dropdown.click();
        
        // Wait for options
        String optionXpath = "//li[contains(text(),'" + partialText + "')]";
        WebElement option = wait.until(
            ExpectedConditions.elementToBeClickable(By.xpath(optionXpath))
        );
        option.click();
    }
    
    /**
     * Get all available options from dropdown
     */
    public List<String> getAllOptions(String dropdownLocator) {
        WebElement dropdown = wait.until(
            ExpectedConditions.elementToBeClickable(By.xpath(dropdownLocator))
        );
        dropdown.click();
        
        // Wait for options to load
        wait.until(ExpectedConditions.presenceOfElementLocated(
            By.xpath("//li[@class='dropdown-option']")
        ));
        
        List<WebElement> options = driver.findElements(
            By.xpath("//li[@class='dropdown-option']")
        );
        
        return options.stream()
            .map(WebElement::getText)
            .collect(java.util.stream.Collectors.toList());
    }
}
```

### Complete Test Examples

#### **Example 1: AJAX Dropdown**

```java
package tests;

import base.BaseTest;
import org.testng.Assert;
import org.testng.annotations.Test;
import utils.DynamicDropdownHandler;

public class AjaxDropdownTest extends BaseTest {
    
    @Test
    public void testAjaxDropdown() {
        driver.get("https://example.com/ajax-dropdown");
        
        DynamicDropdownHandler handler = new DynamicDropdownHandler(driver);
        
        // Select option from AJAX-loaded dropdown
        handler.selectFromAjaxDropdown("countryDropdown", "United States");
        
        // Verify selection
        WebElement selectedOption = driver.findElement(By.id("selectedCountry"));
        Assert.assertEquals(selectedOption.getText(), "United States");
    }
}
```

#### **Example 2: Cascading Dropdowns**

```java
@Test
public void testCascadingDropdowns() {
    driver.get("https://example.com/address-form");
    
    DynamicDropdownHandler handler = new DynamicDropdownHandler(driver);
    
    // Select country first
    handler.selectFromCascadingDropdown(
        "countryDropdown", "United States",
        "stateDropdown", "California"
    );
    
    // State dropdown should now be populated
    WebElement stateDropdown = driver.findElement(By.id("stateDropdown"));
    Assert.assertTrue(stateDropdown.isEnabled());
    
    // Select state
    handler.selectFromAjaxDropdown("stateDropdown", "California");
    
    // City dropdown should now be populated
    handler.selectFromAjaxDropdown("cityDropdown", "Los Angeles");
}
```

#### **Example 3: Searchable Dropdown**

```java
@Test
public void testSearchableDropdown() {
    driver.get("https://example.com/product-search");
    
    DynamicDropdownHandler handler = new DynamicDropdownHandler(driver);
    
    // Search and select from dropdown
    handler.selectFromSearchableDropdown(
        "productDropdown", 
        "laptop", 
        "Gaming Laptop - $999"
    );
    
    // Verify selection
    WebElement selectedProduct = driver.findElement(By.id("selectedProduct"));
    Assert.assertTrue(selectedProduct.getText().contains("Gaming Laptop"));
}
```

### Scenario-Based Example

**Real Scenario: E-commerce product filter dropdown**

```java
@Test
public void testProductFilterDropdown() {
    driver.get("https://ecommerce.example.com/products");
    
    // Wait for page to load
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    wait.until(ExpectedConditions.presenceOfElementLocated(By.id("productList")));
    
    // Click filter dropdown
    WebElement filterDropdown = driver.findElement(By.id("filterDropdown"));
    filterDropdown.click();
    
    // Wait for filter options to load (AJAX)
    wait.until(ExpectedConditions.presenceOfElementLocated(
        By.xpath("//div[@class='filter-options']/div")
    ));
    
    // Select filter option
    WebElement priceFilter = wait.until(
        ExpectedConditions.elementToBeClickable(
            By.xpath("//div[text()='Price: Low to High']")
        )
    );
    priceFilter.click();
    
    // Wait for products to reload with new filter
    wait.until(ExpectedConditions.presenceOfElementLocated(
        By.xpath("//div[@class='product-item']")
    ));
    
    // Verify products are sorted
    List<WebElement> products = driver.findElements(By.className("product-item"));
    Assert.assertTrue(products.size() > 0, "Products should be displayed");
}
```

### Best Practices

1. **Always Use Explicit Wait**
   - Wait for dropdown to be clickable
   - Wait for options to load
   - Wait for selection to complete

2. **Handle AJAX Loading**
   - Wait for options to appear
   - Check option count if needed
   - Handle loading indicators

3. **Use Robust Locators**
   - Prefer IDs over XPath when possible
   - Use relative XPath for options
   - Avoid brittle selectors

4. **Handle Exceptions**
   - Catch NoSuchElementException
   - Retry mechanism for flaky dropdowns
   - Log errors for debugging

5. **Verify Selection**
   - Always verify option was selected
   - Check dropdown value after selection
   - Verify dependent dropdowns update

### Summary

**Key Strategies:**
- ✅ Use explicit wait for dynamic content
- ✅ Wait for options to load before selecting
- ✅ Handle cascading dropdowns sequentially
- ✅ Use Select class for standard dropdowns
- ✅ Custom handling for non-standard dropdowns

---

## 10. Difference between default and public access modifiers

### Context
This question tests your understanding of Java access modifiers, which control the visibility and accessibility of classes, methods, and variables.

### Detailed Answer

Access modifiers in Java control the scope and visibility of classes, methods, and variables. **Default** (package-private) and **Public** are two of the four access modifiers in Java.

### Access Modifiers Overview

Java has four access modifiers:
1. **private** - Only accessible within the same class
2. **default** (package-private) - Accessible within the same package
3. **protected** - Accessible within the same package and subclasses
4. **public** - Accessible from anywhere

### Default Access Modifier

**Definition:** When no access modifier is specified, it's called "default" or "package-private" access.

**Characteristics:**
- No keyword needed (just omit access modifier)
- Accessible only within the same package
- Not accessible from other packages
- Classes, methods, and variables can have default access

**Syntax:**
```java
// Default access - no modifier keyword
class MyClass {
    int myVariable;  // Default access
    void myMethod() { }  // Default access
}
```

### Public Access Modifier

**Definition:** Public access means the class, method, or variable is accessible from anywhere.

**Characteristics:**
- Uses `public` keyword
- Accessible from any package
- No restrictions on access
- Most permissive access level

**Syntax:**
```java
public class MyClass {
    public int myVariable;  // Public access
    public void myMethod() { }  // Public access
}
```

### Key Differences

| Aspect | Default (Package-Private) | Public |
|--------|---------------------------|--------|
| **Keyword** | None (no modifier) | `public` |
| **Visibility** | Same package only | Anywhere |
| **Access from same package** | ✅ Yes | ✅ Yes |
| **Access from different package** | ❌ No | ✅ Yes |
| **Access from subclass (different package)** | ❌ No | ✅ Yes |
| **Use Case** | Internal implementation | API/Interface |

### Code Examples

#### **Example 1: Default Access**

```java
// File: com.example.utils.Calculator.java
package com.example.utils;

class Calculator {  // Default access - no public keyword
    int add(int a, int b) {  // Default access method
        return a + b;
    }
    
    int subtract(int a, int b) {  // Default access method
        return a - b;
    }
}

// File: com.example.utils.MathHelper.java (same package)
package com.example.utils;

public class MathHelper {
    public void useCalculator() {
        Calculator calc = new Calculator();  // ✅ Can access - same package
        int result = calc.add(5, 3);  // ✅ Can access - same package
    }
}

// File: com.example.test.TestCalculator.java (different package)
package com.example.test;
import com.example.utils.Calculator;  // ❌ Compilation error - not accessible

public class TestCalculator {
    public void test() {
        Calculator calc = new Calculator();  // ❌ Error - different package
    }
}
```

#### **Example 2: Public Access**

```java
// File: com.example.utils.Calculator.java
package com.example.utils;

public class Calculator {  // Public access
    public int add(int a, int b) {  // Public method
        return a + b;
    }
    
    public int subtract(int a, int b) {  // Public method
        return a - b;
    }
}

// File: com.example.test.TestCalculator.java (different package)
package com.example.test;
import com.example.utils.Calculator;  // ✅ Can import - public class

public class TestCalculator {
    public void test() {
        Calculator calc = new Calculator();  // ✅ Can access - public
        int result = calc.add(5, 3);  // ✅ Can access - public method
    }
}
```

#### **Example 3: Mixed Access Modifiers**

```java
package com.example.utils;

public class DataProcessor {
    // Public method - accessible from anywhere
    public void processData(String data) {
        validateData(data);  // Can call default method from same class
        String cleaned = cleanData(data);  // Can call private method
        saveData(cleaned);
    }
    
    // Default method - accessible only within package
    void validateData(String data) {
        if (data == null || data.isEmpty()) {
            throw new IllegalArgumentException("Data cannot be empty");
        }
    }
    
    // Private method - accessible only within this class
    private String cleanData(String data) {
        return data.trim().toLowerCase();
    }
    
    // Default method - internal implementation
    void saveData(String data) {
        // Save to database
    }
}

// Same package - can access default methods
package com.example.utils;

public class DataHelper {
    public void helperMethod() {
        DataProcessor processor = new DataProcessor();
        processor.processData("test");  // ✅ Public method
        processor.validateData("test");  // ✅ Default method - same package
        // processor.cleanData("test");  // ❌ Error - private method
    }
}

// Different package - cannot access default methods
package com.example.test;

import com.example.utils.DataProcessor;

public class TestDataProcessor {
    public void test() {
        DataProcessor processor = new DataProcessor();
        processor.processData("test");  // ✅ Public method - accessible
        // processor.validateData("test");  // ❌ Error - default access
        // processor.saveData("test");  // ❌ Error - default access
    }
}
```

#### **Example 4: Test Automation Context**

```java
// Base test class - public for test framework access
package tests.base;

import org.openqa.selenium.WebDriver;

public class BaseTest {  // Public - accessible by test framework
    protected WebDriver driver;  // Protected - accessible by subclasses
    
    public void setUp() {  // Public - test framework can call
        initializeDriver();
    }
    
    void initializeDriver() {  // Default - internal implementation
        // Driver initialization logic
    }
    
    private void cleanup() {  // Private - internal only
        // Cleanup logic
    }
}

// Test class in different package
package tests.login;

import tests.base.BaseTest;  // ✅ Can import - public class

public class LoginTest extends BaseTest {
    public void testLogin() {
        setUp();  // ✅ Can call - public method
        // initializeDriver();  // ❌ Error - default access, different package
        // cleanup();  // ❌ Error - private method
    }
}
```

### When to Use Which?

#### **Use Default (Package-Private) when:**
- ✅ Internal implementation details
- ✅ Helper methods used only within package
- ✅ Want to hide implementation from other packages
- ✅ Creating internal APIs

**Example:**
```java
package com.example.utils;

class InternalHelper {  // Default - internal use only
    static void helperMethod() {
        // Internal helper logic
    }
}
```

#### **Use Public when:**
- ✅ Creating public API
- ✅ Methods/variables need to be accessible from anywhere
- ✅ Test classes need to access
- ✅ Framework or library interfaces

**Example:**
```java
package com.example.api;

public class UserService {  // Public - part of API
    public User getUserById(int id) {  // Public - API method
        return findUser(id);
    }
    
    private User findUser(int id) {  // Private - internal
        // Implementation
    }
}
```

### Summary Table

| Access Modifier | Same Class | Same Package | Subclass (Different Package) | Different Package |
|----------------|------------|--------------|-------------------------------|-------------------|
| **private** | ✅ | ❌ | ❌ | ❌ |
| **default** | ✅ | ✅ | ❌ | ❌ |
| **protected** | ✅ | ✅ | ✅ | ❌ |
| **public** | ✅ | ✅ | ✅ | ✅ |

### Best Practices

1. **Use Public Sparingly**
   - Only for API/public interface
   - Minimize public surface area

2. **Use Default for Internal Implementation**
   - Hide implementation details
   - Better encapsulation

3. **Principle of Least Privilege**
   - Use most restrictive access that works
   - Start with private/default, make public only if needed

4. **Document Public APIs**
   - Public methods are part of API contract
   - Should be well-documented

### Summary

**Default (Package-Private):**
- No keyword
- Accessible within same package only
- Use for internal implementation

**Public:**
- `public` keyword
- Accessible from anywhere
- Use for public API/interface

**Key Difference:** Default restricts access to the same package, while public allows access from anywhere.

---

## 11. What does method hiding mean in Java?

### Context
This question tests your understanding of advanced Java concepts, specifically the difference between method overriding and method hiding, which is often confused by developers.

### Detailed Answer

**Method hiding** occurs when a subclass defines a static method with the same signature as a static method in the parent class. This is different from method overriding, which applies to instance methods.

### Key Concepts

1. **Method Overriding** - For instance methods (non-static)
2. **Method Hiding** - For static methods
3. **Polymorphism** - Works with overriding, not with hiding

### Method Hiding vs Method Overriding

| Aspect | Method Overriding | Method Hiding |
|--------|-------------------|---------------|
| **Method Type** | Instance methods (non-static) | Static methods |
| **Binding** | Runtime (dynamic) | Compile-time (static) |
| **Polymorphism** | ✅ Yes | ❌ No |
| **@Override Annotation** | ✅ Can use | ❌ Cannot use |
| **Method Resolution** | Based on object type | Based on reference type |

### Code Examples

#### **Example 1: Method Hiding with Static Methods**

```java
class Parent {
    // Static method in parent
    static void display() {
        System.out.println("Parent static method");
    }
    
    // Instance method in parent
    void show() {
        System.out.println("Parent instance method");
    }
}

class Child extends Parent {
    // Method HIDING - static method with same signature
    static void display() {
        System.out.println("Child static method");
    }
    
    // Method OVERRIDING - instance method with same signature
    @Override
    void show() {
        System.out.println("Child instance method");
    }
}

public class MethodHidingExample {
    public static void main(String[] args) {
        Parent parent = new Parent();
        Child child = new Child();
        Parent parentRef = new Child();  // Parent reference, Child object
        
        // Static method calls - Method Hiding
        parent.display();        // Output: Parent static method
        child.display();        // Output: Child static method
        parentRef.display();     // Output: Parent static method (hiding - uses reference type)
        
        // Instance method calls - Method Overriding
        parent.show();           // Output: Parent instance method
        child.show();            // Output: Child instance method
        parentRef.show();        // Output: Child instance method (overriding - uses object type)
    }
}
```

**Output:**
```
Parent static method
Child static method
Parent static method        ← Method hiding - uses reference type
Parent instance method
Child instance method
Child instance method       ← Method overriding - uses object type
```

#### **Example 2: Understanding the Difference**

```java
class Animal {
    static void makeSound() {
        System.out.println("Animal makes a sound");
    }
    
    void eat() {
        System.out.println("Animal eats");
    }
}

class Dog extends Animal {
    // Method HIDING - static method
    static void makeSound() {
        System.out.println("Dog barks");
    }
    
    // Method OVERRIDING - instance method
    @Override
    void eat() {
        System.out.println("Dog eats dog food");
    }
}

public class MethodHidingDemo {
    public static void main(String[] args) {
        Animal animal = new Dog();  // Parent reference, Child object
        
        // Static method - Method Hiding
        // Resolution based on REFERENCE TYPE (Animal)
        animal.makeSound();  // Output: Animal makes a sound
        
        // Instance method - Method Overriding
        // Resolution based on OBJECT TYPE (Dog)
        animal.eat();  // Output: Dog eats dog food
        
        // To call child's static method, use child reference
        Dog dog = new Dog();
        dog.makeSound();  // Output: Dog barks
    }
}
```

#### **Example 3: Method Hiding with Variables**

```java
class Parent {
    static String name = "Parent";
    String instanceName = "Parent Instance";
}

class Child extends Parent {
    static String name = "Child";  // Hides parent's static variable
    String instanceName = "Child Instance";  // Hides parent's instance variable
}

public class VariableHiding {
    public static void main(String[] args) {
        Parent parent = new Child();
        
        // Static variable - uses reference type
        System.out.println(parent.name);  // Output: Parent
        
        // Instance variable - uses reference type (not overriding)
        System.out.println(parent.instanceName);  // Output: Parent Instance
        
        Child child = new Child();
        System.out.println(child.name);  // Output: Child
        System.out.println(child.instanceName);  // Output: Child Instance
    }
}
```

#### **Example 4: Practical Example - Test Framework**

```java
// Base test class
class BaseTest {
    // Static method - utility method
    static void printTestInfo(String testName) {
        System.out.println("Running test: " + testName);
        System.out.println("Base test framework");
    }
    
    // Instance method - can be overridden
    void setUp() {
        System.out.println("Base setup");
    }
}

// Specific test class
class LoginTest extends BaseTest {
    // Method HIDING - hides parent's static method
    static void printTestInfo(String testName) {
        System.out.println("Running test: " + testName);
        System.out.println("Login test specific info");
    }
    
    // Method OVERRIDING - overrides parent's instance method
    @Override
    void setUp() {
        System.out.println("Login test setup");
        super.setUp();  // Call parent method
    }
}

public class TestFrameworkExample {
    public static void main(String[] args) {
        BaseTest test = new LoginTest();
        
        // Static method - uses reference type (BaseTest)
        test.printTestInfo("testLogin");  
        // Output: Running test: testLogin
        //         Base test framework
        
        // Instance method - uses object type (LoginTest)
        test.setUp();
        // Output: Login test setup
        //         Base setup
    }
}
```

### Important Points

1. **Static Methods Cannot Be Overridden**
   - They can only be hidden
   - Resolution happens at compile-time

2. **Use @Override Only for Instance Methods**
   ```java
   class Child extends Parent {
       // ✅ Correct - instance method overriding
       @Override
       void instanceMethod() { }
       
       // ❌ Compilation error - cannot override static method
       // @Override
       // static void staticMethod() { }
   }
   ```

3. **Polymorphism Doesn't Work with Static Methods**
   ```java
   Parent obj = new Child();
   obj.staticMethod();  // Calls Parent's method, not Child's
   ```

4. **Best Practice: Call Static Methods Using Class Name**
   ```java
   // ✅ Good practice
   Parent.staticMethod();
   Child.staticMethod();
   
   // ⚠️ Works but not recommended
   Parent obj = new Child();
   obj.staticMethod();  // Calls Parent's method
   ```

### Scenario-Based Example

**Real Scenario: Utility class inheritance**

```java
// Base utility class
class StringUtils {
    // Static utility method
    static String capitalize(String str) {
        if (str == null || str.isEmpty()) {
            return str;
        }
        return str.substring(0, 1).toUpperCase() + str.substring(1);
    }
    
    // Instance method
    String process(String input) {
        return capitalize(input);
    }
}

// Extended utility class
class AdvancedStringUtils extends StringUtils {
    // Method HIDING - hides parent's static method
    static String capitalize(String str) {
        if (str == null || str.isEmpty()) {
            return str;
        }
        // Enhanced version - handles multiple words
        String[] words = str.split(" ");
        StringBuilder result = new StringBuilder();
        for (String word : words) {
            result.append(word.substring(0, 1).toUpperCase())
                  .append(word.substring(1))
                  .append(" ");
        }
        return result.toString().trim();
    }
    
    // Method OVERRIDING - overrides parent's instance method
    @Override
    String process(String input) {
        return capitalize(input);  // Uses hidden static method
    }
}

public class UtilityExample {
    public static void main(String[] args) {
        StringUtils utils1 = new AdvancedStringUtils();
        AdvancedStringUtils utils2 = new AdvancedStringUtils();
        
        // Static method - method hiding
        System.out.println(StringUtils.capitalize("hello world"));
        // Output: Hello world
        
        System.out.println(AdvancedStringUtils.capitalize("hello world"));
        // Output: Hello World
        
        // Instance method - method overriding
        System.out.println(utils1.process("hello world"));
        // Uses AdvancedStringUtils.process() which calls AdvancedStringUtils.capitalize()
        // Output: Hello World
    }
}
```

### Summary

**Method Hiding:**
- Applies to **static methods**
- Resolution based on **reference type** (compile-time)
- No polymorphism
- Cannot use `@Override` annotation

**Method Overriding:**
- Applies to **instance methods**
- Resolution based on **object type** (runtime)
- Supports polymorphism
- Can use `@Override` annotation

**Key Takeaway:** Static methods are hidden (not overridden), and method resolution happens at compile-time based on the reference type, not the object type.

---

## 12. Write Java code to print array elements appearing 2 or more times

### Context
This question tests your ability to work with arrays, collections, and implement logic to find duplicate elements. This is a common coding problem in technical interviews.

### Detailed Answer

There are multiple approaches to find elements that appear 2 or more times in an array. We'll explore several efficient solutions.

### Implementation Approaches

#### **Approach 1: Using HashMap (Most Efficient)**

```java
import java.util.*;

public class DuplicateElements {
    
    /**
     * Find elements appearing 2 or more times using HashMap
     * Time Complexity: O(n)
     * Space Complexity: O(n)
     */
    public static void printDuplicates(int[] arr) {
        Map<Integer, Integer> frequencyMap = new HashMap<>();
        
        // Count frequency of each element
        for (int num : arr) {
            frequencyMap.put(num, frequencyMap.getOrDefault(num, 0) + 1);
        }
        
        // Print elements appearing 2 or more times
        System.out.println("Elements appearing 2 or more times:");
        for (Map.Entry<Integer, Integer> entry : frequencyMap.entrySet()) {
            if (entry.getValue() >= 2) {
                System.out.println(entry.getKey() + " appears " + entry.getValue() + " times");
            }
        }
    }
    
    /**
     * Return list of duplicate elements
     */
    public static List<Integer> getDuplicates(int[] arr) {
        Map<Integer, Integer> frequencyMap = new HashMap<>();
        List<Integer> duplicates = new ArrayList<>();
        
        for (int num : arr) {
            frequencyMap.put(num, frequencyMap.getOrDefault(num, 0) + 1);
        }
        
        for (Map.Entry<Integer, Integer> entry : frequencyMap.entrySet()) {
            if (entry.getValue() >= 2) {
                duplicates.add(entry.getKey());
            }
        }
        
        return duplicates;
    }
    
    public static void main(String[] args) {
        int[] arr = {1, 2, 3, 2, 4, 5, 3, 3, 6, 7, 2};
        printDuplicates(arr);
        
        // Output:
        // Elements appearing 2 or more times:
        // 2 appears 3 times
        // 3 appears 3 times
    }
}
```

#### **Approach 2: Using HashSet (Find First Occurrence of Duplicates)**

```java
import java.util.*;

public class DuplicateElementsSet {
    
    /**
     * Find duplicate elements using HashSet
     * Time Complexity: O(n)
     * Space Complexity: O(n)
     */
    public static void printDuplicates(int[] arr) {
        Set<Integer> seen = new HashSet<>();
        Set<Integer> duplicates = new HashSet<>();
        
        for (int num : arr) {
            if (seen.contains(num)) {
                duplicates.add(num);
            } else {
                seen.add(num);
            }
        }
        
        System.out.println("Duplicate elements: " + duplicates);
    }
    
    /**
     * Print duplicates with their counts
     */
    public static void printDuplicatesWithCount(int[] arr) {
        Map<Integer, Integer> countMap = new HashMap<>();
        Set<Integer> seen = new HashSet<>();
        
        for (int num : arr) {
            if (seen.contains(num)) {
                countMap.put(num, countMap.getOrDefault(num, 1) + 1);
            } else {
                seen.add(num);
                countMap.put(num, 1);
            }
        }
        
        System.out.println("Elements appearing 2 or more times:");
        for (Map.Entry<Integer, Integer> entry : countMap.entrySet()) {
            if (entry.getValue() >= 2) {
                System.out.println(entry.getKey() + " appears " + entry.getValue() + " times");
            }
        }
    }
    
    public static void main(String[] args) {
        int[] arr = {1, 2, 3, 2, 4, 5, 3, 3, 6, 7, 2};
        printDuplicates(arr);
        printDuplicatesWithCount(arr);
    }
}
```

#### **Approach 3: Using Arrays.sort() (Space Efficient)**

```java
import java.util.*;

public class DuplicateElementsSort {
    
    /**
     * Find duplicates using sorting
     * Time Complexity: O(n log n)
     * Space Complexity: O(1) excluding input array
     */
    public static void printDuplicates(int[] arr) {
        Arrays.sort(arr);
        
        System.out.println("Elements appearing 2 or more times:");
        int count = 1;
        
        for (int i = 1; i < arr.length; i++) {
            if (arr[i] == arr[i - 1]) {
                count++;
            } else {
                if (count >= 2) {
                    System.out.println(arr[i - 1] + " appears " + count + " times");
                }
                count = 1;
            }
        }
        
        // Check last element
        if (count >= 2) {
            System.out.println(arr[arr.length - 1] + " appears " + count + " times");
        }
    }
    
    public static void main(String[] args) {
        int[] arr = {1, 2, 3, 2, 4, 5, 3, 3, 6, 7, 2};
        printDuplicates(arr);
    }
}
```

#### **Approach 4: Using Java 8 Streams (Functional Style)**

```java
import java.util.*;
import java.util.function.Function;
import java.util.stream.Collectors;

public class DuplicateElementsStream {
    
    /**
     * Find duplicates using Java 8 Streams
     */
    public static void printDuplicates(int[] arr) {
        Map<Integer, Long> frequencyMap = Arrays.stream(arr)
            .boxed()
            .collect(Collectors.groupingBy(
                Function.identity(),
                Collectors.counting()
            ));
        
        System.out.println("Elements appearing 2 or more times:");
        frequencyMap.entrySet().stream()
            .filter(entry -> entry.getValue() >= 2)
            .forEach(entry -> System.out.println(
                entry.getKey() + " appears " + entry.getValue() + " times"
            ));
    }
    
    /**
     * Get list of duplicate elements
     */
    public static List<Integer> getDuplicates(int[] arr) {
        Map<Integer, Long> frequencyMap = Arrays.stream(arr)
            .boxed()
            .collect(Collectors.groupingBy(
                Function.identity(),
                Collectors.counting()
            ));
        
        return frequencyMap.entrySet().stream()
            .filter(entry -> entry.getValue() >= 2)
            .map(Map.Entry::getKey)
            .collect(Collectors.toList());
    }
    
    public static void main(String[] args) {
        int[] arr = {1, 2, 3, 2, 4, 5, 3, 3, 6, 7, 2};
        printDuplicates(arr);
        
        List<Integer> duplicates = getDuplicates(arr);
        System.out.println("Duplicate elements list: " + duplicates);
    }
}
```

#### **Approach 5: Complete Solution with Multiple Options**

```java
import java.util.*;

public class FindDuplicates {
    
    /**
     * Method 1: Using HashMap - O(n) time, O(n) space
     */
    public static void findDuplicatesHashMap(int[] arr) {
        Map<Integer, Integer> map = new HashMap<>();
        
        for (int num : arr) {
            map.put(num, map.getOrDefault(num, 0) + 1);
        }
        
        System.out.println("Using HashMap:");
        for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
            if (entry.getValue() >= 2) {
                System.out.println(entry.getKey() + " -> " + entry.getValue() + " times");
            }
        }
    }
    
    /**
     * Method 2: Using HashSet - O(n) time, O(n) space
     */
    public static void findDuplicatesHashSet(int[] arr) {
        Set<Integer> seen = new HashSet<>();
        Set<Integer> duplicates = new HashSet<>();
        
        for (int num : arr) {
            if (!seen.add(num)) {  // add() returns false if element exists
                duplicates.add(num);
            }
        }
        
        System.out.println("Using HashSet - Duplicate elements: " + duplicates);
    }
    
    /**
     * Method 3: Using sorting - O(n log n) time, O(1) space
     */
    public static void findDuplicatesSort(int[] arr) {
        int[] sorted = arr.clone();
        Arrays.sort(sorted);
        
        System.out.println("Using Sorting:");
        int count = 1;
        for (int i = 1; i < sorted.length; i++) {
            if (sorted[i] == sorted[i - 1]) {
                count++;
            } else {
                if (count >= 2) {
                    System.out.println(sorted[i - 1] + " -> " + count + " times");
                }
                count = 1;
            }
        }
        if (count >= 2) {
            System.out.println(sorted[sorted.length - 1] + " -> " + count + " times");
        }
    }
    
    /**
     * Method 4: Brute force - O(n²) time, O(1) space
     */
    public static void findDuplicatesBruteForce(int[] arr) {
        System.out.println("Using Brute Force:");
        Set<Integer> printed = new HashSet<>();
        
        for (int i = 0; i < arr.length; i++) {
            if (printed.contains(arr[i])) {
                continue;
            }
            
            int count = 0;
            for (int j = 0; j < arr.length; j++) {
                if (arr[i] == arr[j]) {
                    count++;
                }
            }
            
            if (count >= 2) {
                System.out.println(arr[i] + " -> " + count + " times");
                printed.add(arr[i]);
            }
        }
    }
    
    public static void main(String[] args) {
        int[] arr = {1, 2, 3, 2, 4, 5, 3, 3, 6, 7, 2, 8, 8};
        
        System.out.println("Input array: " + Arrays.toString(arr));
        System.out.println();
        
        findDuplicatesHashMap(arr);
        System.out.println();
        
        findDuplicatesHashSet(arr);
        System.out.println();
        
        findDuplicatesSort(arr);
        System.out.println();
        
        findDuplicatesBruteForce(arr);
    }
}
```

### Test Cases

```java
public class DuplicateElementsTest {
    public static void main(String[] args) {
        // Test Case 1: Normal case
        System.out.println("Test Case 1:");
        int[] arr1 = {1, 2, 3, 2, 4, 5, 3, 3, 6, 7, 2};
        FindDuplicates.findDuplicatesHashMap(arr1);
        // Expected: 2 appears 3 times, 3 appears 3 times
        
        // Test Case 2: No duplicates
        System.out.println("\nTest Case 2:");
        int[] arr2 = {1, 2, 3, 4, 5};
        FindDuplicates.findDuplicatesHashMap(arr2);
        // Expected: No output
        
        // Test Case 3: All duplicates
        System.out.println("\nTest Case 3:");
        int[] arr3 = {5, 5, 5, 5};
        FindDuplicates.findDuplicatesHashMap(arr3);
        // Expected: 5 appears 4 times
        
        // Test Case 4: Empty array
        System.out.println("\nTest Case 4:");
        int[] arr4 = {};
        FindDuplicates.findDuplicatesHashMap(arr4);
        // Expected: No output
        
        // Test Case 5: Single element
        System.out.println("\nTest Case 5:");
        int[] arr5 = {1};
        FindDuplicates.findDuplicatesHashMap(arr5);
        // Expected: No output
    }
}
```

### Best Practices

1. **Choose Right Approach**
   - HashMap: Best for most cases (O(n) time)
   - Sorting: Best when space is concern (O(n log n) time)
   - HashSet: Good for just finding duplicates (O(n) time)

2. **Handle Edge Cases**
   - Empty array
   - Single element
   - All duplicates
   - No duplicates

3. **Consider Constraints**
   - Array size
   - Value range
   - Memory constraints

### Summary

**Recommended Approach:**
```java
// Using HashMap - Most efficient and clear
Map<Integer, Integer> map = new HashMap<>();
for (int num : arr) {
    map.put(num, map.getOrDefault(num, 0) + 1);
}
map.entrySet().stream()
    .filter(entry -> entry.getValue() >= 2)
    .forEach(entry -> System.out.println(
        entry.getKey() + " appears " + entry.getValue() + " times"
    ));
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

---

## 13. Write a program to read a sentence and print each word along with how many characters it contains.

### Context
This question tests your string manipulation skills, understanding of string methods, and ability to process text input. Common in coding interviews.

### Detailed Answer

This problem requires reading a sentence, splitting it into words, and counting characters in each word. We'll explore multiple approaches.

### Implementation Approaches

#### **Approach 1: Using String.split()**

```java
import java.util.Scanner;

public class WordCharacterCount {
    
    /**
     * Read sentence and print each word with character count
     */
    public static void printWordCharacterCount(String sentence) {
        // Split sentence into words (space as delimiter)
        String[] words = sentence.split("\\s+");
        
        System.out.println("Word\t\tCharacter Count");
        System.out.println("-".repeat(30));
        
        for (String word : words) {
            // Remove punctuation if needed
            String cleanWord = word.replaceAll("[^a-zA-Z0-9]", "");
            System.out.println(cleanWord + "\t\t" + cleanWord.length());
        }
    }
    
    /**
     * Enhanced version with punctuation handling
     */
    public static void printWordCharacterCountEnhanced(String sentence) {
        // Split by whitespace
        String[] words = sentence.split("\\s+");
        
        System.out.println("Word\t\tCharacter Count");
        System.out.println("-".repeat(40));
        
        for (String word : words) {
            // Remove punctuation from end of word
            String cleanWord = word.replaceAll("[^a-zA-Z0-9]", "");
            
            if (!cleanWord.isEmpty()) {
                System.out.printf("%-20s %d%n", cleanWord, cleanWord.length());
            }
        }
    }
    
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        System.out.print("Enter a sentence: ");
        String sentence = scanner.nextLine();
        
        printWordCharacterCount(sentence);
        
        scanner.close();
    }
}
```

#### **Approach 2: Using StringTokenizer**

```java
import java.util.StringTokenizer;

public class WordCharacterCountTokenizer {
    
    public static void printWordCharacterCount(String sentence) {
        StringTokenizer tokenizer = new StringTokenizer(sentence);
        
        System.out.println("Word\t\tCharacter Count");
        System.out.println("-".repeat(30));
        
        while (tokenizer.hasMoreTokens()) {
            String word = tokenizer.nextToken();
            // Remove punctuation
            String cleanWord = word.replaceAll("[^a-zA-Z0-9]", "");
            System.out.println(cleanWord + "\t\t" + cleanWord.length());
        }
    }
    
    public static void main(String[] args) {
        String sentence = "Hello world! This is a test sentence.";
        printWordCharacterCount(sentence);
    }
}
```

#### **Approach 3: Manual Parsing**

```java
public class WordCharacterCountManual {
    
    public static void printWordCharacterCount(String sentence) {
        System.out.println("Word\t\tCharacter Count");
        System.out.println("-".repeat(30));
        
        StringBuilder word = new StringBuilder();
        
        for (char c : sentence.toCharArray()) {
            if (Character.isLetterOrDigit(c)) {
                word.append(c);
            } else if (word.length() > 0) {
                // End of word
                System.out.println(word.toString() + "\t\t" + word.length());
                word.setLength(0); // Clear StringBuilder
            }
        }
        
        // Print last word if sentence doesn't end with punctuation
        if (word.length() > 0) {
            System.out.println(word.toString() + "\t\t" + word.length());
        }
    }
    
    public static void main(String[] args) {
        String sentence = "Hello world! This is a test.";
        printWordCharacterCount(sentence);
    }
}
```

#### **Approach 4: Using Java 8 Streams**

```java
import java.util.Arrays;
import java.util.stream.Collectors;

public class WordCharacterCountStream {
    
    public static void printWordCharacterCount(String sentence) {
        System.out.println("Word\t\tCharacter Count");
        System.out.println("-".repeat(30));
        
        Arrays.stream(sentence.split("\\s+"))
            .map(word -> word.replaceAll("[^a-zA-Z0-9]", ""))
            .filter(word -> !word.isEmpty())
            .forEach(word -> System.out.println(word + "\t\t" + word.length()));
    }
    
    /**
     * Enhanced version with formatting
     */
    public static void printWordCharacterCountFormatted(String sentence) {
        System.out.println("Word\t\t\tCharacter Count");
        System.out.println("-".repeat(40));
        
        Arrays.stream(sentence.split("\\s+"))
            .map(word -> word.replaceAll("[^a-zA-Z0-9]", ""))
            .filter(word -> !word.isEmpty())
            .forEach(word -> System.out.printf("%-20s %d%n", word, word.length()));
    }
    
    public static void main(String[] args) {
        String sentence = "Hello world! This is a test sentence.";
        printWordCharacterCount(sentence);
        System.out.println();
        printWordCharacterCountFormatted(sentence);
    }
}
```

#### **Approach 5: Complete Solution with Multiple Features**

```java
import java.util.*;
import java.util.stream.Collectors;

public class WordCharacterCounter {
    
    /**
     * Basic version - simple word count
     */
    public static void printWordCharacterCount(String sentence) {
        String[] words = sentence.split("\\s+");
        
        System.out.println("Word\t\tCharacter Count");
        System.out.println("-".repeat(30));
        
        for (String word : words) {
            String cleanWord = word.replaceAll("[^a-zA-Z0-9]", "");
            if (!cleanWord.isEmpty()) {
                System.out.println(cleanWord + "\t\t" + cleanWord.length());
            }
        }
    }
    
    /**
     * Enhanced version with statistics
     */
    public static void printWordCharacterCountWithStats(String sentence) {
        String[] words = sentence.split("\\s+");
        List<WordInfo> wordList = new ArrayList<>();
        
        for (String word : words) {
            String cleanWord = word.replaceAll("[^a-zA-Z0-9]", "");
            if (!cleanWord.isEmpty()) {
                wordList.add(new WordInfo(cleanWord, cleanWord.length()));
            }
        }
        
        // Print words
        System.out.println("Word\t\t\tCharacter Count");
        System.out.println("-".repeat(40));
        wordList.forEach(w -> System.out.printf("%-20s %d%n", w.word, w.length));
        
        // Print statistics
        System.out.println("\nStatistics:");
        System.out.println("Total words: " + wordList.size());
        System.out.println("Average word length: " + 
            wordList.stream().mapToInt(w -> w.length).average().orElse(0));
        System.out.println("Longest word: " + 
            wordList.stream().max(Comparator.comparingInt(w -> w.length))
                .map(w -> w.word).orElse(""));
        System.out.println("Shortest word: " + 
            wordList.stream().min(Comparator.comparingInt(w -> w.length))
                .map(w -> w.word).orElse(""));
    }
    
    /**
     * Version that preserves original word with punctuation
     */
    public static void printWordCharacterCountPreservePunctuation(String sentence) {
        String[] words = sentence.split("\\s+");
        
        System.out.println("Word\t\t\tCharacter Count (Letters Only)");
        System.out.println("-".repeat(50));
        
        for (String word : words) {
            String lettersOnly = word.replaceAll("[^a-zA-Z0-9]", "");
            if (!lettersOnly.isEmpty()) {
                System.out.printf("%-25s %d%n", word, lettersOnly.length());
            }
        }
    }
    
    /**
     * Interactive version with user input
     */
    public static void interactiveWordCounter() {
        Scanner scanner = new Scanner(System.in);
        
        System.out.println("Word Character Counter");
        System.out.println("Enter sentences (type 'exit' to quit):");
        
        while (true) {
            System.out.print("\nEnter a sentence: ");
            String sentence = scanner.nextLine();
            
            if (sentence.equalsIgnoreCase("exit")) {
                break;
            }
            
            if (sentence.trim().isEmpty()) {
                System.out.println("Please enter a valid sentence.");
                continue;
            }
            
            printWordCharacterCount(sentence);
        }
        
        scanner.close();
        System.out.println("Goodbye!");
    }
    
    // Helper class to store word information
    static class WordInfo {
        String word;
        int length;
        
        WordInfo(String word, int length) {
            this.word = word;
            this.length = length;
        }
    }
    
    public static void main(String[] args) {
        // Example 1: Basic usage
        String sentence1 = "Hello world! This is a test.";
        System.out.println("Example 1:");
        printWordCharacterCount(sentence1);
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // Example 2: With statistics
        String sentence2 = "The quick brown fox jumps over the lazy dog";
        System.out.println("Example 2 - With Statistics:");
        printWordCharacterCountWithStats(sentence2);
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // Example 3: Preserve punctuation
        String sentence3 = "Hello, world! How are you?";
        System.out.println("Example 3 - Preserve Punctuation:");
        printWordCharacterCountPreservePunctuation(sentence3);
        
        // Uncomment to run interactive version
        // interactiveWordCounter();
    }
}
```

### Test Cases

```java
public class WordCharacterCountTest {
    public static void main(String[] args) {
        // Test Case 1: Normal sentence
        System.out.println("Test Case 1:");
        WordCharacterCounter.printWordCharacterCount("Hello world");
        // Expected:
        // Hello    5
        // world   5
        
        // Test Case 2: With punctuation
        System.out.println("\nTest Case 2:");
        WordCharacterCounter.printWordCharacterCount("Hello, world! How are you?");
        // Expected: Punctuation removed, words counted
        
        // Test Case 3: Multiple spaces
        System.out.println("\nTest Case 3:");
        WordCharacterCounter.printWordCharacterCount("Hello    world    test");
        // Expected: Handles multiple spaces correctly
        
        // Test Case 4: Empty string
        System.out.println("\nTest Case 4:");
        WordCharacterCounter.printWordCharacterCount("");
        // Expected: No output
        
        // Test Case 5: Single word
        System.out.println("\nTest Case 5:");
        WordCharacterCounter.printWordCharacterCount("Hello");
        // Expected: Hello    5
    }
}
```

### Expected Output

```
Input: "Hello world! This is a test sentence."

Output:
Word            Character Count
------------------------------
Hello           5
world           5
This            4
is              2
a               1
test            4
sentence        8
```

### Best Practices

1. **Handle Punctuation**
   - Remove or preserve based on requirements
   - Use regex `[^a-zA-Z0-9]` to remove punctuation

2. **Handle Multiple Spaces**
   - Use `split("\\s+")` to handle multiple spaces
   - Trim input if needed

3. **Handle Edge Cases**
   - Empty strings
   - Single word
   - Only punctuation
   - Multiple consecutive spaces

4. **Format Output**
   - Use `printf` for aligned output
   - Consider table format for readability

### Summary

**Recommended Approach:**
```java
String[] words = sentence.split("\\s+");
for (String word : words) {
    String cleanWord = word.replaceAll("[^a-zA-Z0-9]", "");
    if (!cleanWord.isEmpty()) {
        System.out.println(cleanWord + "\t\t" + cleanWord.length());
    }
}
```

**Key Points:**
- Use `split("\\s+")` to split by whitespace
- Remove punctuation with regex if needed
- Handle edge cases (empty, single word, etc.)
- Format output for readability

---

*End of Document*


