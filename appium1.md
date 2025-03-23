# Appium Setup and Configuration Guide

This guide will walk you through setting up Appium for mobile application testing.

## Prerequisites

- Node.js and npm installed
- Java Development Kit (JDK) installed
- Android SDK installed (for Android testing)
- Xcode installed (for iOS testing, Mac only)
- Real device or emulator/simulator

## Installation Steps

1. **Install Appium Server**

   ```bash
   npm install -g appium
   ```

2. **Install Appium Doctor** (to verify your setup)

   ```bash
   npm install -g appium-doctor
   ```

3. **Verify Installation**

   ```bash
   appium-doctor
   ```

4. **Install Appium Client Library** (for your preferred language)

   For Java:
   ```xml
   <!-- Add to pom.xml -->
   <dependency>
       <groupId>io.appium</groupId>
       <artifactId>java-client</artifactId>
       <version>8.5.1</version>
   </dependency>
   ```

   For Python:
   ```bash
   pip install Appium-Python-Client
   ```

5. **Install Appium Inspector**
   
   Download from: https://github.com/appium/appium-inspector/releases

## Basic Configuration

### For Android Testing

1. **Set Environment Variables**

   ```bash
   export ANDROID_HOME=/path/to/android/sdk
   export JAVA_HOME=/path/to/jdk
   export PATH=$PATH:$ANDROID_HOME/platform-tools
   export PATH=$PATH:$ANDROID_HOME/tools
   ```

2. **Start Android Emulator**

   ```bash
   emulator -avd [emulator_name]
   ```

3. **Basic Appium Capabilities for Android**

   ```java
   DesiredCapabilities capabilities = new DesiredCapabilities();
   capabilities.setCapability("platformName", "Android");
   capabilities.setCapability("deviceName", "emulator-5554");
   capabilities.setCapability("automationName", "UiAutomator2");
   capabilities.setCapability("app", "/path/to/your/app.apk");
   ```

### For iOS Testing

1. **WebDriverAgent Setup** (required for iOS testing)

   ```bash
   cd /usr/local/lib/node_modules/appium/node_modules/appium-webdriveragent
   mkdir -p Resources/WebDriverAgent.bundle
   xcodebuild -project WebDriverAgent.xcodeproj -scheme WebDriverAgentRunner -destination 'id=<device-id>' test
   ```

2. **Basic Appium Capabilities for iOS**

   ```java
   DesiredCapabilities capabilities = new DesiredCapabilities();
   capabilities.setCapability("platformName", "iOS");
   capabilities.setCapability("deviceName", "iPhone 14");
   capabilities.setCapability("platformVersion", "16.0");
   capabilities.setCapability("automationName", "XCUITest");
   capabilities.setCapability("app", "/path/to/your/app.ipa");
   ```

## Starting Appium Server

1. **Command Line**

   ```bash
   appium --address 127.0.0.1 --port 4723
   ```

2. **Programmatically (Java)**

   ```java
   AppiumDriverLocalService service = new AppiumServiceBuilder()
       .withIPAddress("127.0.0.1")
       .usingPort(4723)
       .build();
   service.start();
   ```

## Troubleshooting

- Check Appium logs for error messages
- Ensure proper path to application file
- Verify device is connected and recognized
- Check that all dependencies are properly installed
- Update Android SDK tools and platforms as needed
