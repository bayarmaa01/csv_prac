# Testing Native Mobile Applications with Appium

This guide provides practical examples for testing native mobile applications using Appium.

## What are Native Apps?

Native apps are built specifically for a particular mobile platform (iOS or Android) using the platform's native programming language and APIs.

## Setting Up a Test Project

### Java with TestNG Example

```java
import io.appium.java_client.AppiumDriver;
import io.appium.java_client.android.AndroidDriver;
import org.openqa.selenium.remote.DesiredCapabilities;
import org.testng.annotations.*;
import java.net.URL;
import java.time.Duration;

public class NativeAppTest {
    private AppiumDriver driver;
    
    @BeforeTest
    public void setUp() throws Exception {
        DesiredCapabilities capabilities = new DesiredCapabilities();
        capabilities.setCapability("platformName", "Android");
        capabilities.setCapability("deviceName", "emulator-5554");
        capabilities.setCapability("automationName", "UiAutomator2");
        capabilities.setCapability("app", "/path/to/example-app.apk");
        
        driver = new AndroidDriver(new URL("http://localhost:4723/wd/hub"), capabilities);
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
    }
    
    @AfterTest
    public void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }
    
    @Test
    public void testLogin() {
        // Test code will go here
    }
}
```

## Element Identification Methods

### 1. Using Resource IDs (Android)

```java
// Find element by resource ID
MobileElement usernameField = driver.findElement(By.id("com.example.app:id/username_field"));
usernameField.sendKeys("testuser");

MobileElement passwordField = driver.findElement(By.id("com.example.app:id/password_field"));
passwordField.sendKeys("password123");

MobileElement loginButton = driver.findElement(By.id("com.example.app:id/login_button"));
loginButton.click();
```

### 2. Using Accessibility IDs (Both platforms)

```java
// Find element by accessibility ID
MobileElement menuButton = driver.findElementByAccessibilityId("menu_button");
menuButton.click();
```

### 3. Using XPath (Last resort)

```java
// Find element by XPath
MobileElement header = driver.findElement(By.xpath("//android.widget.TextView[@text='Welcome']"));
Assert.assertTrue(header.isDisplayed());
```

## Complete Test Example: Calculator App

```java
import io.appium.java_client.AppiumDriver;
import io.appium.java_client.android.AndroidDriver;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.remote.DesiredCapabilities;
import org.testng.Assert;
import org.testng.annotations.*;

import java.net.URL;
import java.time.Duration;

public class CalculatorTest {
    private AppiumDriver driver;
    
    @BeforeTest
    public void setUp() throws Exception {
        DesiredCapabilities capabilities = new DesiredCapabilities();
        capabilities.setCapability("platformName", "Android");
        capabilities.setCapability("deviceName", "emulator-5554");
        capabilities.setCapability("automationName", "UiAutomator2");
        // Use the default calculator app on Android
        capabilities.setCapability("appPackage", "com.android.calculator2");
        capabilities.setCapability("appActivity", "com.android.calculator2.Calculator");
        
        driver = new AndroidDriver(new URL("http://localhost:4723/wd/hub"), capabilities);
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
    }
    
    @AfterTest
    public void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }
    
    @Test
    public void testAddition() {
        // Press 5
        WebElement five = driver.findElement(By.id("com.android.calculator2:id/digit_5"));
        five.click();
        
        // Press +
        WebElement plus = driver.findElement(By.id("com.android.calculator2:id/op_add"));
        plus.click();
        
        // Press 3
        WebElement three = driver.findElement(By.id("com.android.calculator2:id/digit_3"));
        three.click();
        
        // Press =
        WebElement equals = driver.findElement(By.id("com.android.calculator2:id/eq"));
        equals.click();
        
        // Verify result
        WebElement result = driver.findElement(By.id("com.android.calculator2:id/result"));
        Assert.assertEquals(result.getText(), "8");
    }
    
    @Test
    public void testMultiplication() {
        // Press 4
        WebElement four = driver.findElement(By.id("com.android.calculator2:id/digit_4"));
        four.click();
        
        // Press *
        WebElement multiply = driver.findElement(By.id("com.android.calculator2:id/op_mul"));
        multiply.click();
        
        // Press 6
        WebElement six = driver.findElement(By.id("com.android.calculator2:id/digit_6"));
        six.click();
        
        // Press =
        WebElement equals = driver.findElement(By.id("com.android.calculator2:id/eq"));
        equals.click();
        
        // Verify result
        WebElement result = driver.findElement(By.id("com.android.calculator2:id/result"));
        Assert.assertEquals(result.getText(), "24");
    }
}
```

## Testing Application Functionality

### Login Flow Test

```java
@Test
public void testLoginFlow() {
    // Enter username
    WebElement username = driver.findElement(By.id("com.example.app:id/username"));
    username.sendKeys("testuser");
    
    // Enter password
    WebElement password = driver.findElement(By.id("com.example.app:id/password"));
    password.sendKeys("password123");
    
    // Click login button
    WebElement loginButton = driver.findElement(By.id("com.example.app:id/login_button"));
    loginButton.click();
    
    // Verify successful login
    WebElement welcomeMessage = driver.findElement(By.id("com.example.app:id/welcome_message"));
    Assert.assertTrue(welcomeMessage.isDisplayed());
    Assert.assertEquals(welcomeMessage.getText(), "Welcome, testuser!");
}
```

## Best Practices

1. **Use Page Object Model**

```java
// LoginPage.java
public class LoginPage {
    private AppiumDriver driver;
    
    // Element locators
    private By usernameField = By.id("com.example.app:id/username");
    private By passwordField = By.id("com.example.app:id/password");
    private By loginButton = By.id("com.example.app:id/login_button");
    
    public LoginPage(AppiumDriver driver) {
        this.driver = driver;
    }
    
    public void enterUsername(String username) {
        driver.findElement(usernameField).sendKeys(username);
    }
    
    public void enterPassword(String password) {
        driver.findElement(passwordField).sendKeys(password);
    }
    
    public HomePage clickLogin() {
        driver.findElement(loginButton).click();
        return new HomePage(driver);
    }
    
    public HomePage loginAs(String username, String password) {
        enterUsername(username);
        enterPassword(password);
        return clickLogin();
    }
}
```

2. **Implement Waits**

```java
// Explicit wait example
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(30));
WebElement element = wait.until(
    ExpectedConditions.elementToBeClickable(By.id("com.example.app:id/button"))
);
element.click();
```

3. **Handle App Permissions**

```java
@Test
public void handlePermissions() {
    // When permissions dialog appears
    try {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(5));
        WebElement allowButton = wait.until(
            ExpectedConditions.elementToBeClickable(By.id("com.android.permissioncontroller:id/permission_allow_button"))
        );
        allowButton.click();
    } catch (Exception e) {
        System.out.println("No permission dialog appeared");
    }
}
```
