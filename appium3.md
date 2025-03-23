# Testing Hybrid Mobile Applications with Appium

This guide provides practical examples for testing hybrid mobile applications using Appium.

## What are Hybrid Apps?

Hybrid apps are built using web technologies (HTML, CSS, JavaScript) but are wrapped in a native container. They run inside a WebView and can access device features through plugins.

## Setting Up for Hybrid App Testing

### Java Example Setup

```java
import io.appium.java_client.AppiumDriver;
import io.appium.java_client.android.AndroidDriver;
import org.openqa.selenium.remote.DesiredCapabilities;
import org.testng.annotations.*;
import java.net.URL;
import java.time.Duration;

public class HybridAppTest {
    private AppiumDriver driver;
    
    @BeforeTest
    public void setUp() throws Exception {
        DesiredCapabilities capabilities = new DesiredCapabilities();
        capabilities.setCapability("platformName", "Android");
        capabilities.setCapability("deviceName", "emulator-5554");
        capabilities.setCapability("automationName", "UiAutomator2");
        capabilities.setCapability("app", "/path/to/hybrid-app.apk");
        // For hybrid apps, chromedriverExecutable might be needed
        capabilities.setCapability("chromedriverExecutable", "/path/to/chromedriver");
        
        driver = new AndroidDriver(new URL("http://localhost:4723/wd/hub"), capabilities);
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
    }
    
    @AfterTest
    public void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

## Context Switching

The key to testing hybrid apps is switching between NATIVE_APP and WEBVIEW contexts.

```java
@Test
public void testHybridApp() throws InterruptedException {
    // First interact with native part of the app
    driver.findElement(By.id("com.example.hybridapp:id/webview_button")).click();
    
    // Wait for WebView to load
    Thread.sleep(2000);
    
    // Get all contexts
    Set<String> contextNames = driver.getContextHandles();
    for (String contextName : contextNames) {
        System.out.println(contextName);
    }
    
    // Switch to WebView context
    for (String contextName : contextNames) {
        if (contextName.contains("WEBVIEW")) {
            driver.context(contextName);
            break;
        }
    }
    
    // Now you can interact with WebView using standard Selenium methods
    driver.findElement(By.id("username")).sendKeys("testuser");
    driver.findElement(By.id("password")).sendKeys("password123");
    driver.findElement(By.id("login")).click();
    
    // Switch back to native context if needed
    driver.context("NATIVE_APP");
}
```

## Complete Hybrid App Test Example

```java
import io.appium.java_client.AppiumDriver;
import io.appium.java_client.android.AndroidDriver;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.remote.DesiredCapabilities;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.testng.Assert;
import org.testng.annotations.*;

import java.net.URL;
import java.time.Duration;
import java.util.Set;

public class HybridAppCompleteTest {
    private AppiumDriver driver;
    
    @BeforeTest
    public void setUp() throws Exception {
        DesiredCapabilities capabilities = new DesiredCapabilities();
        capabilities.setCapability("platformName", "Android");
        capabilities.setCapability("deviceName", "emulator-5554");
        capabilities.setCapability("automationName", "UiAutomator2");
        capabilities.setCapability("app", "/path/to/hybrid-app.apk");
        capabilities.setCapability("chromedriverExecutable", "/path/to/chromedriver");
        
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
    public void testHybridAppLogin() {
        // 1. Start in Native context and navigate to WebView
        WebElement webviewButton = driver.findElement(By.id("com.example.hybridapp:id/webview_button"));
        webviewButton.click();
        
        // 2. Wait for WebView to be available
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        wait.until(driver -> {
            Set<String> contexts = driver.getContextHandles();
            return contexts.size() > 1;
        });
        
        // 3. Get all contexts and switch to WebView
        Set<String> contextNames = driver.getContextHandles();
        String webViewContext = null;
        for (String contextName : contextNames) {
            if (contextName.contains("WEBVIEW")) {
                webViewContext = contextName;
                break;
            }
        }
        
        // 4. Switch to WebView context
        driver.context(webViewContext);
        
        // 5. Now perform actions in the WebView
        WebElement usernameField = wait.until(
            ExpectedConditions.elementToBeClickable(By.id("username"))
        );
        usernameField.sendKeys("testuser");
        
        WebElement passwordField = driver.findElement(By.id("password"));
        passwordField.sendKeys("password123");
        
        WebElement loginButton = driver.findElement(By.id("login"));
        loginButton.click();
        
        // 6. Verify successful login in WebView
        WebElement welcomeMessage = wait.until(
            ExpectedConditions.visibilityOfElementLocated(By.id("welcome-message"))
        );
        Assert.assertEquals(welcomeMessage.getText(), "Welcome, testuser!");
        
        // 7. Switch back to native context
        driver.context("NATIVE_APP");
        
        // 8. Verify native elements if needed
        WebElement backButton = driver.findElement(By.id("com.example.hybridapp:id/back_button"));
        Assert.assertTrue(backButton.isDisplayed());
    }
}
```

## Finding WebView Elements

### Using Chrome DevTools for Element Inspection

1. Enable USB debugging on your device
2. Connect device to your computer
3. Open Chrome on your computer and navigate to `chrome://inspect`
4. Find your device and the WebView
5. Click "inspect" to open DevTools
6. Use the element inspector to find element attributes

### Using Appium Inspector

1. Start your test and switch to WebView context
2. Use Appium Inspector to locate elements in the WebView

### Common WebView Locator Strategies

```java
// By ID
driver.findElement(By.id("login-button")).click();

// By CSS Selector 
driver.findElement(By.cssSelector("button.primary")).click();

// By XPath
driver.findElement(By.xpath("//button[contains(text(),'Login')]")).click();
```

## Handling Multiple WebViews

```java
@Test
public void testMultipleWebViews() {
    // Get all contexts
    Set<String> contextNames = driver.getContextHandles();
    
    // Print all available contexts
    System.out.println("Available contexts:");
    for (String context : contextNames) {
        System.out.println(context);
    }
    
    // Switch to the second WebView (if multiple exist)
    String[] webViewContexts = contextNames.stream()
        .filter(context -> context.contains("WEBVIEW"))
        .toArray(String[]::new);
    
    if (webViewContexts.length > 1) {
        driver.context(webViewContexts[1]);
    } else {
        // Just switch to the available WebView
        for (String context : contextNames) {
            if (context.contains("WEBVIEW")) {
                driver.context(context);
                break;
            }
        }
    }
}
```

## Best Practices for Hybrid App Testing

1. **Use Appropriate ChromeDriver Version**

The ChromeDriver version should match the Chrome version in your WebView:

```java
capabilities.setCapability("chromedriverExecutable", "/path/to/specific/chromedriver");
```

2. **Handle Context Switching Properly**

Always check context availability before switching:

```java
// Utility method to switch to WebView
public void switchToWebView() {
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    wait.until(driver -> {
        Set<String> contexts = driver.getContextHandles();
        return contexts.size() > 1;
    });
    
    Set<String> contextNames = driver.getContextHandles();
    for (String contextName : contextNames) {
        if (contextName.contains("WEBVIEW")) {
            driver.context(contextName);
            return;
        }
    }
}

// Utility method to switch to Native
public void switchToNative() {
    driver.context("NATIVE_APP");
}
```

3. **Wait for Page Loads in WebView**

```java
// Wait for page to load in WebView
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
wait.until(ExpectedConditions.presenceOfElementLocated(By.id("page-loaded-indicator")));
```

4. **Handle JavaScript in WebView**

```java
// Execute JavaScript in WebView
JavascriptExecutor jsExecutor = (JavascriptExecutor) driver;
jsExecutor.executeScript("document.getElementById('username').value='testuser';");
```
