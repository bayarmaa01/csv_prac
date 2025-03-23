# Testing Mobile Web Applications with Appium

This guide provides practical examples for testing mobile web applications using Appium.

## What is Mobile Web Testing?

Mobile web testing involves testing websites on mobile browsers (Safari, Chrome, etc.) to ensure they work properly on mobile devices.

## Setting Up for Mobile Web Testing

### Java Example Setup

```java
import io.appium.java_client.AppiumDriver;
import io.appium.java_client.android.AndroidDriver;
import org.openqa.selenium.remote.DesiredCapabilities;
import org.testng.annotations.*;
import java.net.URL;
import java.time.Duration;

public class MobileWebTest {
    private AppiumDriver driver;
    
    @BeforeTest
    public void setUp() throws Exception {
        DesiredCapabilities capabilities = new DesiredCapabilities();
        capabilities.setCapability("platformName", "Android");
        capabilities.setCapability("deviceName", "emulator-5554");
        capabilities.setCapability("automationName", "UiAutomator2");
        capabilities.setCapability("browserName", "Chrome");
        // Optional: Specify Chrome/Safari driver path
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
    public void testWebsiteOnMobile() {
        // Navigate to a website
        driver.get("https://www.example.com");
        
        // Rest of the test...
    }
}
```

## iOS Safari Example

```java
import io.appium.java_client.AppiumDriver;
import io.appium.java_client.ios.IOSDriver;
import org.openqa.selenium.remote.DesiredCapabilities;
import org.testng.annotations.*;
import java.net.URL;
import java.time.Duration;

public class IOSWebTest {
    private AppiumDriver driver;
    
    @BeforeTest
    public void setUp() throws Exception {
        DesiredCapabilities capabilities = new DesiredCapabilities();
        capabilities.setCapability("platformName", "iOS");
        capabilities.setCapability("deviceName", "iPhone 14");
        capabilities.setCapability("platformVersion", "16.0");
        capabilities.setCapability("automationName", "XCUITest");
        capabilities.setCapability("browserName", "Safari");
        
        driver = new IOSDriver(new URL("http://localhost:4723/wd/hub"), capabilities);
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

## Complete Mobile Web Test Example

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

public class MobileWebCompleteTest {
    private AppiumDriver driver;
    
    @BeforeTest
    public void setUp() throws Exception {
        DesiredCapabilities capabilities = new DesiredCapabilities();
        capabilities.setCapability("platformName", "Android");
        capabilities.setCapability("deviceName", "emulator-5554");
        capabilities.setCapability("automationName", "UiAutomator2");
        capabilities.setCapability("browserName", "Chrome");
        
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
    public void testMobileWebLogin() {
        // 1. Navigate to a website
        driver.get("https://the-internet.herokuapp.com/login");
        
        // 2. Wait for page to load
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        WebElement usernameField = wait.until(
            ExpectedConditions.visibilityOfElementLocated(By.id("username"))
        );
        
        // 3. Enter login credentials
        usernameField.sendKeys("tomsmith");
        
        WebElement passwordField = driver.findElement(By.id("password"));
        passwordField.sendKeys("SuperSecretPassword!");
        
        // 4. Submit form
        WebElement loginButton = driver.findElement(By.cssSelector("button[type='submit']"));
        loginButton.click();
        
        // 5. Verify successful login
        WebElement successMessage = wait.until(
            ExpectedConditions.visibilityOfElementLocated(By.cssSelector(".flash.success"))
        );
        Assert.assertTrue(successMessage.getText().contains("You logged into a secure area!"));
        
        // 6. Test responsive design elements
        WebElement secureArea = driver.findElement(By.cssSelector(".example h2"));
        Assert.assertEquals(secureArea.getText(), "Secure Area");
    }
    
    @Test
    public void testResponsiveDesign() {
        // 1. Navigate to a responsive website
        driver.get("https://the-internet.herokuapp.com");
        
        // 2. Get window size
        int width = driver.manage().window().getSize().getWidth();
        int height = driver.manage().window().getSize().getHeight();
        System.out.println("Window size: " + width + "x" + height);
        
        // 3. Verify menu is visible
        WebElement menu = driver.findElement(By.cssSelector("ul"));
        Assert.assertTrue(menu.isDisplayed());
        
        // 4. Check specific mobile elements if they exist
        try {
            WebElement mobileMenu = driver.findElement(By.cssSelector(".mobile-menu"));
            Assert.assertTrue(mobileMenu.isDisplayed());
        } catch (Exception e) {
            System.out.println("Mobile menu not found - might not be a mobile-specific element");
        }
    }
}
```

## Testing Responsive Web Design

```java
@Test
public void testResponsiveLayouts() {
    // Navigate to website
    driver.get("https://your-responsive-website.com");
    
    // Get current window size
    int initialWidth = driver.manage().window().getSize().getWidth();
    int height = driver.manage().window().getSize().getHeight();
    
    // Check desktop version elements
    WebElement desktopNav = driver.findElement(By.id("desktop-nav"));
    Assert.assertTrue(desktopNav.isDisplayed());
    
    // Resize window to simulate mobile size
    // Note: This may not work on all devices/emulators
    try {
        driver.manage().window().setSize(new Dimension(375, height));
        
        // Wait for responsive changes
        Thread.sleep(1000);
        
        // Check if desktop nav is hidden and mobile nav is shown
        Assert.assertFalse(desktopNav.isDisplayed());
        
        WebElement mobileNav = driver.findElement(By.id("mobile-nav"));
        Assert.assertTrue(mobileNav.isDisplayed());
    } catch (Exception e) {
        System.out.println("Window resize not supported or element not found: " + e.getMessage());
    } finally {
        // Restore original size
        driver.manage().window().setSize(new Dimension(initialWidth, height));
    }
}
```

## Handling Mobile-Specific Features

### 1. Device Orientation

```java
@Test
public void testOrientationChange() {
    // Navigate to website
    driver.get("https://your-responsive-website.com");
    
    // Get initial orientation
    String initialOrientation = ((Rotatable) driver).getOrientation().value();
    System.out.println("Initial orientation: " + initialOrientation);
    
    // Test in portrait
    if (!initialOrientation.equals("PORTRAIT")) {
        ((Rotatable) driver).rotate(ScreenOrientation.PORTRAIT);
    }
    
    // Check layout in portrait
    WebElement element = driver.findElement(By.id("responsive-element"));
    int portraitWidth = element.getSize().getWidth();
    
    // Rotate to landscape
    ((Rotatable) driver).rotate(ScreenOrientation.LANDSCAPE);
    
    // Wait for rotation to complete
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(5));
    wait.until(driver -> ((Rotatable) driver).getOrientation().value().equals("LANDSCAPE"));
    
    // Check layout in landscape
    int landscapeWidth = element.getSize().getWidth();
    
    // Verify width changed with orientation
    Assert.assertNotEquals(portraitWidth, landscapeWidth);
    
    //
