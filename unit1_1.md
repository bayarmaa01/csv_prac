# Software Installation and Configuration Guide

This guide walks through the process of installing and configuring necessary software tools and libraries for our testing laboratory environment.

## Prerequisites

- Computer with Windows 10/11, macOS (10.15+), or Linux (Ubuntu 20.04+ recommended)
- Administrator/sudo privileges
- Minimum 8GB RAM, 20GB free disk space
- Internet connection

## Step 1: Install Python Environment

### Windows
1. Download Python 3.10+ from [python.org](https://python.org)
2. During installation, check "Add Python to PATH"
3. Open Command Prompt and verify installation:
   ```
   python --version
   ```

### macOS
1. Install Homebrew if not already installed:
   ```
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```
2. Install Python using Homebrew:
   ```
   brew install python
   ```
3. Verify installation:
   ```
   python3 --version
   ```

### Linux
1. Update package lists:
   ```
   sudo apt update
   ```
2. Install Python and development tools:
   ```
   sudo apt install python3 python3-pip python3-dev build-essential
   ```
3. Verify installation:
   ```
   python3 --version
   ```

## Step 2: Set Up Virtual Environment

1. Install virtualenv:
   ```
   pip install virtualenv
   ```

2. Create a project directory:
   ```
   mkdir testing_lab
   cd testing_lab
   ```

3. Create and activate virtual environment:
   ```
   # For Windows
   virtualenv venv
   venv\Scripts\activate

   # For macOS/Linux
   virtualenv venv
   source venv/bin/activate
   ```

## Step 3: Install Testing Libraries

With your virtual environment activated, install the following libraries:

```
pip install pytest selenium appium requests pytest-html pytest-xdist allure-pytest webdriver-manager
```

## Step 4: Install Browser Drivers

### Chrome Driver
1. Install Chrome browser if not already installed
2. Install Chrome WebDriver:
   ```
   pip install webdriver-manager
   ```
3. In your Python code, use:
   ```python
   from selenium import webdriver
   from webdriver_manager.chrome import ChromeDriverManager
   from selenium.webdriver.chrome.service import Service

   driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()))
   ```

### Firefox Driver
1. Install Firefox browser if not already installed
2. Install Gecko WebDriver:
   ```
   pip install webdriver-manager
   ```
3. In your Python code, use:
   ```python
   from selenium import webdriver
   from webdriver_manager.firefox import GeckoDriverManager
   from selenium.webdriver.firefox.service import Service

   driver = webdriver.Firefox(service=Service(GeckoDriverManager().install()))
   ```

## Step 5: Install Mobile Testing Tools

### Android Testing
1. Install Android Studio from [developer.android.com](https://developer.android.com/studio)
2. Install JDK 8+ and set JAVA_HOME environment variable
3. Install Appium:
   ```
   npm install -g appium
   ```
4. Install Appium Doctor to verify setup:
   ```
   npm install -g appium-doctor
   appium-doctor
   ```
5. Create AVD (Android Virtual Device) through Android Studio

### iOS Testing (macOS only)
1. Install Xcode from App Store
2. Install Appium as above
3. Install libimobiledevice:
   ```
   brew install libimobiledevice
   ```

## Step 6: Install API Testing Tools

1. Install Postman from [postman.com](https://www.postman.com/downloads/)
2. Install Newman for command-line collection runs:
   ```
   npm install -g newman
   ```
3. Install additional tools:
   ```
   pip install pytest-bdd behave requests-mock jsonschema
   ```

## Step 7: Configure Git and Version Control

1. Install Git:
   - Windows: Download from [git-scm.com](https://git-scm.com/download/win)
   - macOS: `brew install git`
   - Linux: `sudo apt install git`

2. Configure Git:
   ```
   git config --global user.name "Your Name"
   git config --global user.email "your.email@example.com"
   ```

3. Set up SSH keys for GitHub:
   ```
   ssh-keygen -t ed25519 -C "your.email@example.com"
   ```
   Follow prompts, then add the key to your GitHub account

## Step 8: Install Reporting Tools

1. Install Allure command-line tool:
   - Windows: `scoop install allure` (requires Scoop)
   - macOS: `brew install allure`
   - Linux: 
     ```
     curl -o allure-2.20.1.tgz -OLs https://repo.maven.apache.org/maven2/io/qameta/allure/allure-commandline/2.20.1/allure-commandline-2.20.1.tgz
     sudo tar -zxvf allure-2.20.1.tgz -C /opt/
     sudo ln -s /opt/allure-2.20.1/bin/allure /usr/bin/allure
     ```

2. Install Jenkins (optional, for CI/CD):
   - Download from [jenkins.io](https://www.jenkins.io/download/)
   - Follow installation instructions for your OS

## Step 9: Configure IDE 

1. Install Visual Studio Code from [code.visualstudio.com](https://code.visualstudio.com/)
2. Install recommended extensions:
   - Python
   - Pylance
   - Test Explorer UI
   - Python Test Explorer
   - GitLens
   - REST Client

## Step 10: Verify Setup

1. Create a sample test file `test_setup.py`:
   ```python
   def test_python_setup():
       assert True

   def test_selenium_import():
       from selenium import webdriver
       from selenium.webdriver.chrome.service import Service
       from webdriver_manager.chrome import ChromeDriverManager
       assert True

   def test_requests_import():
       import requests
       assert True
   ```

2. Run the test to verify everything is installed correctly:
   ```
   pytest test_setup.py -v
   ```

## Troubleshooting

### Python Path Issues
- Check system environment variables
- Ensure Python is in PATH
- Try using `python3` instead of `python` on macOS/Linux

### WebDriver Issues
- Update browsers to latest version
- Manually download WebDriver if automatic installation fails
- Check browser and WebDriver version compatibility

### Permission Issues
- Run commands with administrator/sudo privileges
- Check file permissions for executable files
- On macOS/Linux, you may need to run: `chmod +x /path/to/driver`
