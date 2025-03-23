# Web Application Test Automation with Selenium WebDriver

This comprehensive guide provides practical examples for automating web application testing using Selenium WebDriver. The guide covers setup, element interaction, implementing test cases, and generating reports.

## Table of Contents
1. [Selenium WebDriver Setup](#selenium-webdriver-setup)
2. [Web Element Interaction Strategies](#web-element-interaction-strategies)
3. [Implementing Test Cases](#implementing-test-cases)
4. [Test Validation and Reporting](#test-validation-and-reporting)

## Selenium WebDriver Setup

### Prerequisites
- Java JDK 11+ installed
- Maven or Gradle for dependency management
- IDE (IntelliJ IDEA, Eclipse, or VS Code)
- Basic understanding of Java programming

### Step 1: Create a Java Maven Project

1. Create a new Maven project in your IDE or use the following command:
   ```bash
   mvn archetype:generate -DgroupId=com.example -DartifactId=selenium-automation -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
   ```

2. Navigate to the project directory:
   ```bash
   cd selenium-automation
   ```

3. Open the `pom.xml` file and add the following dependencies:

   ```xml
   <dependencies>
       <!-- Selenium WebDriver -->
       <dependency>
           <groupId>org.seleniumhq.selenium</groupId>
           <artifactId>selenium-java</artifactId>
           <version>4.11.0</version>
       </dependency>
       
       <!-- TestNG for test framework -->
       <dependency>
           <groupId>org.testng</groupId>
           <artifactId>testng</artifactId>
           <version>7.7.1</version>
       </dependency>
       
       <!-- WebDriverManager for browser driver management -->
       <dependency>
           <groupId>io.github.bonigarcia</groupId>
           <artifactId>webdrivermanager</artifactId>
           <version>5.4.1</version>
       </dependency>
       
       <!-- ExtentReports for reporting -->
       <dependency>
           <groupId>com.aventstack</groupId>
           <artifactId>extentreports</artifactId>
           <version>5.0.9</version>
       </dependency>
   </dependencies>
   ```

### Step 2: Create Base Test Class

Create a base test class that will handle WebDriver initialization and cleanup:

```java
// src/test/java/com/example/base/BaseTest.java
package com.example.base;

import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.firefox.FirefoxDriver;
import org.openqa.selenium.edge.EdgeDriver;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Parameters;

import java.time.Duration;

public class BaseTest {
    protected WebDriver driver;
    
    @BeforeMethod
    @Parameters({"browser"})
    public void setup(String browser) {
        // Setup WebDriver based on browser parameter
        switch (browser.toLowerCase()) {
            case "firefox":
                WebDriverManager.firefoxdriver().setup();
                driver = new FirefoxDriver();
                break;
            case "edge":
                WebDriverManager.edgedriver().setup();
                driver = new EdgeDriver();
                break;
            default:
                WebDriverManager.chromedriver().setup();
                driver = new ChromeDriver();
                break;
        }
        
        // Configure WebDriver
        driver.manage().window().maximize();
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
        driver.manage().timeouts().pageLoadTimeout(Duration.ofSeconds(30));
    }
    
    @AfterMethod
    public void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

### Step 3: Create TestNG XML Configuration

Create a TestNG XML file to configure test execution:

```xml
<!-- testng.xml -->
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="Selenium Test Suite">
    <parameter name="browser" value="chrome"/>
    <test name="Web Application Tests">
        <classes>
            <class name="com.example.tests.LoginTest"/>
            <class name="com.example.tests.RegistrationTest"/>
            <class name="com.example.tests.DataEntryTest"/>
        </classes>
    </test>
</suite>
```

### Step 4: Run Your First Test

Create a simple test to verify your setup:

```java
// src/test/java/com/example/tests/SetupVerificationTest.java
package com.example.tests;

import com.example.base.BaseTest;
import org.testng.Assert;
import org.testng.annotations.Test;

public class SetupVerificationTest extends BaseTest {
    
    @Test
    public void verifySeleniumSetup() {
        driver.get("https://www.selenium.dev/");
        String title = driver.getTitle();
        System.out.println("Page title: " + title);
        Assert.assertTrue(title.contains("Selenium"), "Page title should contain 'Selenium'");
    }
}
```

Run the test using TestNG in your IDE or with Maven:
```bash
mvn test -Dtest=SetupVerificationTest
```

## Web Element Interaction Strategies

### Common Locator Types

Create a reference class for different locator strategies:

```java
// src/test/java/com/example/locators/LocatorExample.java
package com.example.locators;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;

import java.time.Duration;
import java.util.List;

public class LocatorExample {
    private WebDriver driver;
    private WebDriverWait wait;
    
    public LocatorExample(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    }
    
    public void demonstrateLocators() {
        // 1. By ID - Most preferred if available
        WebElement elementById = driver.findElement(By.id("username"));
        
        // 2. By Name
        WebElement elementByName = driver.findElement(By.name("password"));
        
        // 3. By Class Name - Returns the first matching element
        WebElement elementByClass = driver.findElement(By.className("login-button"));
        
        // 4. By Tag Name - Often returns multiple elements
        List<WebElement> elementsByTag = driver.findElements(By.tagName("input"));
        
        // 5. By Link Text - For <a> elements
        WebElement elementByLinkText = driver.findElement(By.linkText("Forgot Password?"));
        
        // 6. By Partial Link Text - For <a> elements with partial match
        WebElement elementByPartialLinkText = driver.findElement(By.partialLinkText("Forgot"));
        
        // 7. By CSS Selector - Very powerful
        WebElement elementByCss = driver.findElement(By.cssSelector(".login-form #username"));
        
        // 8. By XPath - Powerful but slower
        WebElement elementByXPath = driver.findElement(By.xpath("//div[@class='login-form']/input[@id='username']"));
        
        // 9. By CSS selector with attribute
        WebElement elementByCssAttr = driver.findElement(By.cssSelector("input[placeholder='Enter username']"));
        
        // 10. By XPath with text content
        WebElement elementByXPathText = driver.findElement(By.xpath("//button[text()='Login']"));
    }
}
```

### Step 1: Create a Page Object Class

Implement the Page Object Model to organize web elements and actions:

```java
// src/main/java/com/example/pages/LoginPage.java
package com.example.pages;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;

import java.time.Duration;

public class LoginPage {
    private WebDriver driver;
    private WebDriverWait wait;
    
    // Elements using @FindBy annotation
    @FindBy(id = "username")
    private WebElement usernameInput;
    
    @FindBy(name = "password")
    private WebElement passwordInput;
    
    @FindBy(css = "button.login-button")
    private WebElement loginButton;
    
    @FindBy(linkText = "Forgot Password?")
    private WebElement forgotPasswordLink;
    
    @FindBy(xpath = "//div[contains(@class,'error-message')]")
    private WebElement errorMessage;
    
    // Constructor
    public LoginPage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        PageFactory.initElements(driver, this);
    }
    
    // Page actions
    public void navigateTo(String url) {
        driver.get(url);
    }
    
    public void enterUsername(String username) {
        wait.until(ExpectedConditions.visibilityOf(usernameInput));
        usernameInput.clear();
        usernameInput.sendKeys(username);
    }
    
    public void enterPassword(String password) {
        passwordInput.clear();
        passwordInput.sendKeys(password);
    }
    
    public void clickLoginButton() {
        wait.until(ExpectedConditions.elementToBeClickable(loginButton));
        loginButton.click();
    }
    
    public void clickForgotPassword() {
        forgotPasswordLink.click();
    }
    
    public String getErrorMessage() {
        wait.until(ExpectedConditions.visibilityOf(errorMessage));
        return errorMessage.getText();
    }
    
    public boolean isLoginButtonDisplayed() {
        return loginButton.isDisplayed();
    }
    
    // Combined operations
    public void login(String username, String password) {
        enterUsername(username);
        enterPassword(password);
        clickLoginButton();
    }
}
```

### Step 2: Element Interaction Examples

Create a demonstration class with different interaction methods:

```java
// src/test/java/com/example/examples/ElementInteractionDemo.java
package com.example.examples;

import com.example.base.BaseTest;
import org.openqa.selenium.By;
import org.openqa.selenium.JavascriptExecutor;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.interactions.Actions;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.Select;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.testng.annotations.Test;

import java.time.Duration;

public class ElementInteractionDemo extends BaseTest {
    
    @Test
    public void demonstrateElementInteractions() {
        driver.get("https://the-internet.herokuapp.com/");
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        
        // 1. Click operation
        WebElement formAuthLink = driver.findElement(By.linkText("Form Authentication"));
        formAuthLink.click();
        
        // 2. Text input operation
        WebElement usernameInput = driver.findElement(By.id("username"));
        usernameInput.sendKeys("tomsmith");
        
        // 3. Clear text operation
        usernameInput.clear();
        usernameInput.sendKeys("tomsmith");
        
        // 4. Explicit wait for element
        WebElement passwordInput = wait.until(
            ExpectedConditions.visibilityOfElementLocated(By.id("password"))
        );
        passwordInput.sendKeys("SuperSecretPassword!");
        
        // 5. Get attribute value
        String placeholderText = usernameInput.getAttribute("placeholder");
        System.out.println("Placeholder text: " + placeholderText);
        
        // 6. Check if element is displayed/enabled
        boolean isLoginEnabled = driver.findElement(By.cssSelector("button[type='submit']")).isEnabled();
        System.out.println("Login button enabled: " + isLoginEnabled);
        
        // 7. Submit form
        driver.findElement(By.cssSelector("#login")).submit();
        
        // Return to homepage for more examples
        driver.get("https://the-internet.herokuapp.com/");
        
        // 8. Select from dropdown
        driver.findElement(By.linkText("Dropdown")).click();
        WebElement dropdown = driver.findElement(By.id("dropdown"));
        Select select = new Select(dropdown);
        select.selectByVisibleText("Option 2");
        
        // 9. Get selected option
        WebElement selectedOption = select.getFirstSelectedOption();
        System.out.println("Selected option: " + selectedOption.getText());
        
        // 10. Mouse hover action
        driver.get("https://the-internet.herokuapp.com/hovers");
        WebElement image = driver.findElement(By.cssSelector(".figure:first-child"));
        Actions actions = new Actions(driver);
        actions.moveToElement(image).perform();
        
        // 11. JavaScript execution
        JavascriptExecutor js = (JavascriptExecutor) driver;
        js.executeScript("arguments[0].scrollIntoView(true);", image);
        js.executeScript("alert('Element scrolled into view')");
        driver.switchTo().alert().dismiss();
        
        // 12. Handle checkboxes
        driver.get("https://the-internet.herokuapp.com/checkboxes");
        WebElement checkbox1 = driver.findElement(By.cssSelector("input[type='checkbox']:nth-child(1)"));
        if (!checkbox1.isSelected()) {
            checkbox1.click();
        }
    }
}
```

### Step 3: Advanced Element Location Strategies

Create examples for handling dynamic elements and complex scenarios:

```java
// src/test/java/com/example/examples/AdvancedLocatorDemo.java
package com.example.examples;

import com.example.base.BaseTest;
import org.openqa.selenium.By;
import org.openqa.selenium.JavascriptExecutor;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.testng.annotations.Test;

import java.time.Duration;
import java.util.List;

public class AdvancedLocatorDemo extends BaseTest {
    
    @Test
    public void demonstrateAdvancedLocators() {
        driver.get("https://www.saucedemo.com/");
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        
        // Login first
        driver.findElement(By.id("user-name")).sendKeys("standard_user");
        driver.findElement(By.id("password")).sendKeys("secret_sauce");
        driver.findElement(By.id("login-button")).click();
        
        // 1. Find elements within another element (container)
        WebElement inventoryContainer = driver.findElement(By.id("inventory_container"));
        List<WebElement> inventoryItems = inventoryContainer.findElements(By.className("inventory_item"));
        System.out.println("Number of inventory items: " + inventoryItems.size());
        
        // 2. Find element by its text using XPath
        WebElement sauceLabsBackpack = driver.findElement(
            By.xpath("//div[text()='Sauce Labs Backpack']")
        );
        System.out.println("Found product: " + sauceLabsBackpack.getText());
        
        // 3. Find element by partial text
        WebElement sauceLabsProduct = driver.findElement(
            By.xpath("//div[contains(text(),'Sauce Labs')]")
        );
        System.out.println("Found product with partial text: " + sauceLabsProduct.getText());
        
        // 4. Find element by its parent
        WebElement priceParent = driver.findElement(
            By.xpath("//div[text()='Sauce Labs Backpack']/parent::div/parent::div//div[@class='inventory_item_price']")
        );
        System.out.println("Price of Sauce Labs Backpack: " + priceParent.getText());
        
        // 5. Find element by its child
        WebElement inventoryItemWithButton = driver.findElement(
            By.xpath("//div[@class='inventory_item']//button[text()='Add to cart']/..")
        );
        
        // 6. Find by CSS Selector with multiple attributes
        WebElement specialProduct = driver.findElement(
            By.cssSelector(".inventory_item .inventory_item_description:has(div.inventory_item_price:contains('$29.99'))")
        );
        
        // 7. Find elements with OR condition
        List<WebElement> expensiveProducts = driver.findElements(
            By.xpath("//div[@class='inventory_item_price' and (text()='$29.99' or text()='$49.99')]")
        );
        System.out.println("Number of expensive products: " + expensiveProducts.size());
        
        // 8. Find by position
        WebElement thirdProduct = driver.findElement(
            By.xpath("(//div[@class='inventory_item'])[3]")
        );
        
        // 9. Find elements with attribute starting with value
        List<WebElement> addToCartButtons = driver.findElements(
            By.cssSelector("button[id^='add-to-cart']")
        );
        System.out.println("Number of 'Add to cart' buttons: " + addToCartButtons.size());
        
        // 10. Find elements with attribute ending with value
        List<WebElement> itemsWithId = driver.findElements(
            By.cssSelector("div[id$='_item']")
        );
        System.out.println("Items with ID ending with '_item': " + itemsWithId.size());
    }
}
```

## Implementing Test Cases

### Step 1: Create Model Classes for Test Data

```java
// src/main/java/com/example/models/User.java
package com.example.models;

public class User {
    private String firstName;
    private String lastName;
    private String email;
    private String username;
    private String password;
    
    // Constructor
    public User(String firstName, String lastName, String email, String username, String password) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.email = email;
        this.username = username;
        this.password = password;
    }
    
    // Getters and setters
    public String getFirstName() {
        return firstName;
    }
    
    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }
    
    public String getLastName() {
        return lastName;
    }
    
    public void setLastName(String lastName) {
        this.lastName = lastName;
    }
    
    public String getEmail() {
        return email;
    }
    
    public void setEmail(String email) {
        this.email = email;
    }
    
    public String getUsername() {
        return username;
    }
    
    public void setUsername(String username) {
        this.username = username;
    }
    
    public String getPassword() {
        return password;
    }
    
    public void setPassword(String password) {
        this.password = password;
    }
    
    // Builder pattern for creating User objects
    public static class Builder {
        private String firstName;
        private String lastName;
        private String email;
        private String username;
        private String password;
        
        public Builder firstName(String firstName) {
            this.firstName = firstName;
            return this;
        }
        
        public Builder lastName(String lastName) {
            this.lastName = lastName;
            return this;
        }
        
        public Builder email(String email) {
            this.email = email;
            return this;
        }
        
        public Builder username(String username) {
            this.username = username;
            return this;
        }
        
        public Builder password(String password) {
            this.password = password;
            return this;
        }
        
        public User build() {
            return new User(firstName, lastName, email, username, password);
        }
    }
}
```

### Step 2: Create Page Object Classes

Create a Registration Page Object:

```java
// src/main/java/com/example/pages/RegistrationPage.java
package com.example.pages;

import com.example.models.User;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;

import java.time.Duration;

public class RegistrationPage {
    private WebDriver driver;
    private WebDriverWait wait;
    
    @FindBy(id = "firstName")
    private WebElement firstNameInput;
    
    @FindBy(id = "lastName")
    private WebElement lastNameInput;
    
    @FindBy(id = "email")
    private WebElement emailInput;
    
    @FindBy(id = "username")
    private WebElement usernameInput;
    
    @FindBy(id = "password")
    private WebElement passwordInput;
    
    @FindBy(id = "confirmPassword")
    private WebElement confirmPasswordInput;
    
    @FindBy(id = "termsCheckbox")
    private WebElement termsCheckbox;
    
    @FindBy(id = "registerButton")
    private WebElement registerButton;
    
    @FindBy(className = "registration-success")
    private WebElement successMessage;
    
    @FindBy(className = "error-message")
    private WebElement errorMessage;
    
    public RegistrationPage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        PageFactory.initElements(driver, this);
    }
    
    public void navigateTo(String url) {
        driver.get(url);
    }
    
    public void enterFirstName(String firstName) {
        wait.until(ExpectedConditions.visibilityOf(firstNameInput));
        firstNameInput.clear();
        firstNameInput.sendKeys(firstName);
    }
    
    public void enterLastName(String lastName) {
        lastNameInput.clear();
        lastNameInput.sendKeys(lastName);
    }
    
    public void enterEmail(String email) {
        emailInput.clear();
        emailInput.sendKeys(email);
    }
    
    public void enterUsername(String username) {
        usernameInput.clear();
        usernameInput.sendKeys(username);
    }
    
    public void enterPassword(String password) {
        passwordInput.clear();
        passwordInput.sendKeys(password);
    }
    
    public void enterConfirmPassword(String password) {
        confirmPasswordInput.clear();
        confirmPasswordInput.sendKeys(password);
    }
    
    public void acceptTerms() {
        if (!termsCheckbox.isSelected()) {
            termsCheckbox.click();
        }
    }
    
    public void clickRegisterButton() {
        wait.until(ExpectedConditions.elementToBeClickable(registerButton));
        registerButton.click();
    }
    
    public String getSuccessMessage() {
        wait.until(ExpectedConditions.visibilityOf(successMessage));
        return successMessage.getText();
    }
    
    public String getErrorMessage() {
        wait.until(ExpectedConditions.visibilityOf(errorMessage));
        return errorMessage.getText();
    }
    
    public void registerUser(User user) {
        enterFirstName(user.getFirstName());
        enterLastName(user.getLastName());
        enterEmail(user.getEmail());
        enterUsername(user.getUsername());
        enterPassword(user.getPassword());
        enterConfirmPassword(user.getPassword());
        acceptTerms();
        clickRegisterButton();
    }
}
```

Create a Data Entry Form Page Object:

```java
// src/main/java/com/example/pages/DataEntryFormPage.java
package com.example.pages;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.Select;
import org.openqa.selenium.support.ui.WebDriverWait;

import java.time.Duration;

public class DataEntryFormPage {
    private WebDriver driver;
    private WebDriverWait wait;
    
    @FindBy(id = "productName")
    private WebElement productNameInput;
    
    @FindBy(id = "productDescription")
    private WebElement productDescriptionInput;
    
    @FindBy(id = "productCategory")
    private WebElement productCategorySelect;
    
    @FindBy(id = "productPrice")
    private WebElement productPriceInput;
    
    @FindBy(id = "inStock")
    private WebElement inStockCheckbox;
    
    @FindBy(name = "productRating")
    private WebElement productRatingRadioButtons;
    
    @FindBy(id = "submitButton")
    private WebElement submitButton;
    
    @FindBy(className = "success-message")
    private WebElement successMessage;
    
    public DataEntryFormPage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        PageFactory.initElements(driver, this);
    }
    
    public void navigateTo(String url) {
        driver.get(url);
    }
    
    public void enterProductName(String name) {
        wait.until(ExpectedConditions.visibilityOf(productNameInput));
        productNameInput.clear();
        productNameInput.sendKeys(name);
    }
    
    public void enterProductDescription(String description) {
        productDescriptionInput.clear();
        productDescriptionInput.sendKeys(description);
    }
    
    public void selectProductCategory(String category) {
        Select categoryDropdown = new Select(productCategorySelect);
        categoryDropdown.selectByVisibleText(category);
    }
    
    public void enterProductPrice(double price) {
        productPriceInput.clear();
        productPriceInput.sendKeys(String.valueOf(price));
    }
    
    public void setInStock(boolean inStock) {
        if (inStockCheckbox.isSelected() != inStock) {
            inStockCheckbox.click();
        }
    }
    
    public void setProductRating(int rating) {
        // Implementation depends on how the radio buttons are structured
        // For simplicity, assuming the rating directly correlates to an element ID
        driver.findElement(org.openqa.selenium.By.id("rating" + rating)).click();
    }
    
    public void submitForm() {
        wait.until(ExpectedConditions.elementToBeClickable(submitButton));
        submitButton.click();
    }
    
    public String getSuccessMessage() {
        wait.until(ExpectedConditions.visibilityOf(successMessage));
        return successMessage.getText();
    }
    
    public void enterProductData(String name, String description, 
                                String category, double price, 
                                boolean inStock, int rating) {
        enterProductName(name);
        enterProductDescription(description);
        selectProductCategory(category);
        enterProductPrice(price);
        setInStock(inStock);
        setProductRating(rating);
        submitForm();
    }
}
```

### Step 3: Implement Test Cases

#### Login Test Case:

```java
// src/test/java/com/example/tests/LoginTest.java
package com.example.tests;

import com.example.base.BaseTest;
import com.example.pages.LoginPage;
import org.testng.Assert;
import org.testng.annotations.DataProvider;
import org.testng.annotations.Test;

public class LoginTest extends BaseTest {
    
    private static final String LOGIN_URL = "https://www.saucedemo.com/";
    
    @DataProvider(name = "loginData")
    public Object[][] loginData() {
        return new Object[][] {
            {"standard_user", "secret_sauce", true, ""},
            {"locked_out_user", "secret_sauce", false, "Epic sadface: Sorry, this user has been locked out."},
            {"", "secret_sauce", false, "Epic sadface: Username is required"},
            {"standard_user", "", false, "Epic sadface: Password is required"},
            {"invalid_user", "secret_sauce", false, "Epic sadface: Username and password do not match any user in this service"}
        };
    }
    
    @Test(dataProvider = "loginData")
    public void testLogin(String username, String password, boolean shouldSucceed, String expectedError) {
        LoginPage loginPage = new LoginPage(driver);
        
        // Navigate to login page
        loginPage.navigateTo(LOGIN_URL);
        
        // Enter credentials and click login
        loginPage.enterUsername(username);
        loginPage.enterPassword(password);
        loginPage.clickLoginButton();
        
        // Verify the result
        if (shouldSucceed) {
            // Check if we landed on the inventory page
            String currentUrl = driver.getCurrentUrl();
            Assert.assertTrue(currentUrl.contains("inventory.html"), 
                             "Should be redirected to inventory page after successful login");
        } else {
            // Check error message
            String actualError = loginPage.getErrorMessage();
            Assert.assertEquals(actualError, expectedError, 
                               "Error message should match expected value");
        }
    }
    
    @Test
    public void testLoginLogout() {
        LoginPage loginPage = new LoginPage(driver);
        
        // Navigate to login page and login
        loginPage.navigateTo(LOGIN_URL);
        loginPage.login("standard_user", "secret_sauce");
        
        // Verify successful login
        String currentUrl = driver.getCurrentUrl();
        Assert.assertTrue(currentUrl.contains("inventory.html"), 
                         "Should be redirected to inventory page after login");
        
        // Logout
        driver.findElement(org.openqa.selenium.By.id("react-burger-menu-btn")).click();
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(2));
        WebElement logoutLink = wait.until(ExpectedConditions.elementToBeClickable(
            org.openqa.selenium.By.id("logout_sidebar_link")));
        logoutLink.click();
        
        // Verify logout
        Assert.assertTrue(loginPage.isLoginButtonDisplayed(), 
                         "Login button should be displayed after logout");
    }
}
```

#### Registration Test Case:

```java
// src/test/java/com/example/tests/RegistrationTest.java
package com.example.tests;

import com.example.base.BaseTest;
import com.example.models.User;
import com.example.pages.RegistrationPage;
import org.testng.Assert;
import
