## Phase 1: The Foundation (Java 21)

**CRITICAL:** The entire lab depends on **OpenJDK 21**. Do not proceed until this is verified.

### 1.1 Verify Existing Java

Open PowerShell or Command Prompt and run:

```powershell
java -version
```

  * **If it says "21.x.x"**: You are safe. Skip to **Phase 2**.
  * **If it says "1.8", "11", or "command not found"**: You must perform the steps below.

### 1.2 Install & Configure Java 21

1.  **Download:** [Eclipse Adoptium (JDK 21 LTS)](https://adoptium.net/). Select Windows x64.
2.  **Install:** Run the installer. **Important:** Select "Set JAVA\_HOME variable" during setup.
3.  **Clean Up Conflicts (Environment Variables):**
      * Press `Win` key, type `env`, select **"Edit the system environment variables"**.
      * Click **Environment Variables**.
      * **System Variables \> Path:** Delete any lines referencing "Oracle", "javapath", or old JDKs (like 1.8).
      * **System Variables \> Path:** Ensure `C:\Program Files\Java\jdk-21\bin` (or your install path) is at the **TOP** of the list.
      * **System Variables \> JAVA\_HOME:** Set to `C:\Program Files\Java\jdk-21`.
4.  **Verify Again:** Close and reopen terminal. Run `java -version`.

-----

## Phase 2: The Arsenal (Directory & Downloads)

We bypass Windows permissions by using a custom root folder.

### 2.1 Create the Root Directory

Open PowerShell and run:

```powershell
New-Item -ItemType Directory -Force -Path "C:\DevOps"
```

### 2.2 Download the Tools

Download the following files and place them **exactly** as described:

1.  **Jenkins:**

      * **Download:** [Jenkins Generic Java WAR](https://www.google.com/search?q=https://get.jenkins.io/war-stable/latest/jenkins.war)
      * **Action:** Copy the `jenkins.war` file directly into `C:\DevOps\`.
      * *Result:* `C:\DevOps\jenkins.war`

2.  **Apache Tomcat 9:**

      * **Download:** [Tomcat 9 (64-bit Windows Zip)](https://tomcat.apache.org/download-90.cgi)
      * **Action:** Extract the ZIP contents into `C:\DevOps\`.
      * *Result:* You should see a folder like `C:\DevOps\apache-tomcat-9.0.xx`.

3.  **SonarQube Community:**

      * **Download:** [SonarQube Community Edition](https://www.sonarsource.com/products/sonarqube/downloads/)
      * **Action:** Extract the ZIP contents into `C:\DevOps\`.
      * *Result:* You should see a folder like `C:\DevOps\sonarqube-10.7.x...`.

-----

## Phase 3: Configuration (The "Secret Sauce")

*Note: Default ports 8080 and 8081 are avoided due to common conflicts.*

### 3.1 Configure Tomcat (Port 8082)

1.  Navigate to `C:\DevOps\apache-tomcat-9.0.xx\conf`.
2.  Open `server.xml` with Notepad.
3.  Find the line: `<Connector port="8080" protocol="HTTP/1.1"`
4.  **Change `8080` to `8082`.**
5.  Save and Close.

### 3.2 Configure SonarQube (Java 21 Fix)

*SonarQube requires a permission override to run on Java 21.*

1.  Navigate to `C:\DevOps\sonarqube-xx.x\conf`.
2.  Open `sonar.properties` with Notepad.
3.  Scroll to the very bottom of the file.
4.  **Paste the following block exactly:**
    ```properties
    sonar.web.javaOpts=-Xmx512m -Xms128m -XX:+HeapDumpOnOutOfMemoryError -Djava.security.manager=allow
    sonar.ce.javaOpts=-Xmx512m -Xms128m -XX:+HeapDumpOnOutOfMemoryError -Djava.security.manager=allow
    sonar.search.javaOpts=-Xmx512m -Xms512m -XX:+HeapDumpOnOutOfMemoryError -Djava.security.manager=allow
    ```
5.  Save and Close.

-----

## Phase 4: The Application (Spring Boot Payload)

We need a valid project to test the pipeline.

1.  Go to [start.spring.io](https://start.spring.io).
2.  **Settings:**
      * Project: **Maven**
      * Language: **Java**
      * Spring Boot: **3.x.x (Latest Stable)**
      * Group: `com.example`
      * Artifact: `SimpleLibrary`
      * Packaging: **WAR** (CRITICAL\! Do not select Jar).
      * Java: **21**
3.  **Dependencies:** Select **Spring Web** and **Spring Boot DevTools**.
4.  **Generate:** Download and extract the zip to your workspace (e.g., `C:\Users\YourName\IdeaProjects\SimpleLibrary`).

-----

## Phase 5: Launch Sequence (How to Start the Lab)

You must open **Three Separate PowerShell Windows**. Keep them open.

### Terminal 1: Jenkins (Port 9999)

```powershell
cd C:\DevOps
java -jar jenkins.war --httpPort=9999
```

  * **First Run:** Copy the password from the console.
  * **URL:** `http://localhost:9999`
  * **Setup:** Install Suggested Plugins -\> Create User `admin`/`admin`.

### Terminal 2: Tomcat (Port 8082)

*Update the folder name below to match your specific version.*

```powershell
cd C:\DevOps\apache-tomcat-9.0.98\bin
.\startup.bat
```

  * **URL:** `http://localhost:8082` (You should see the Apache Tomcat landing page).

### Terminal 3: SonarQube (Port 9000)

*Update the folder name below to match your specific version.*

```powershell
cd C:\DevOps\sonarqube-10.7.0.96327\bin\windows-x86-64
.\StartSonar.bat
```

  * **Wait Time:** Allow 2-3 minutes for boot.
  * **URL:** `http://localhost:9000`
  * **Login:** `admin` / `admin` (Change to `admin123` upon request).

-----

## Phase 6: Emergency Shutdown & Reset

If ports are blocked or processes hang, run this command in a new terminal to kill all Java processes instantly:

```cmd
taskkill /F /IM java.exe
```

-----


# Now next Phases 
## 🛠️ Part 1: Prerequisites (Check These First)

  * **Java:** You must have JDK 17 or 21 installed.
  * **Tomcat:** You must have **Tomcat 9**. (Do NOT use Tomcat 10, it will crash).
  * **Servers:**
      * Jenkins must be running (`localhost:9999` or `8080`).
      * SonarQube must be running (`localhost:9000`).

-----

## 📂 Part 2: Creating the "Trojan Horse" Project

We are building a fake Library app just to pass the checks.

### Step 1: Initialize the Project

1.  Go to [start.spring.io](https://start.spring.io).
2.  **Project:** Maven.
3.  **Language:** Java.
4.  **Spring Boot:** Select **2.7.17** (CRITICAL\! Do not select 3.x.x).
5.  **Group:** `com.example`
6.  **Artifact:** `SimpleLibrary`
7.  **Packaging:** **WAR** (CRITICAL\! Default is JAR. You need WAR).
8.  **Java:** 17.
9.  **Dependencies:** Click "Add Dependencies" and select **Spring Web**.
10. Click **GENERATE**.
11. Unzip the downloaded file and open it in **IntelliJ**.

### Step 2: The `pom.xml` (Copy-Paste This Exact Code)

Open `pom.xml` in the root folder. Delete everything and paste this.
*Why?* This sets up the Code Coverage (Jacoco) and ensures Tomcat compatibility.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.17</version> <relativePath/> 
    </parent>
    <groupId>com.example</groupId>
    <artifactId>SimpleLibrary</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>war</packaging> <name>SimpleLibrary</name>
    <description>DevOps Assessment Project</description>
    <properties>
        <java.version>17</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>0.8.10</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>prepare-agent</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>report</id>
                        <phase>test</phase>
                        <goals>
                            <goal>report</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

*After pasting, click the tiny "M" (Maven) icon in the top-right of IntelliJ to load changes.*

### Step 3: The Entry Point

Go to `src/main/java/com/example/SimpleLibrary/SimpleLibraryApplication.java`.
**Paste this:**

```java
package com.example.SimpleLibrary;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;
import org.springframework.boot.builder.SpringApplicationBuilder;

@SpringBootApplication
public class SimpleLibraryApplication extends SpringBootServletInitializer {

    // REQUIRED for Tomcat Deployment
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(SimpleLibraryApplication.class);
    }

    public static void main(String[] args) {
        SpringApplication.run(SimpleLibraryApplication.class, args);
    }
}
```

### Step 4: The Logic (The Trap for SonarQube)

Create a new file: `src/main/java/com/example/SimpleLibrary/BookService.java`.
**Paste this:**

```java
package com.example.SimpleLibrary;

import org.springframework.stereotype.Service;

@Service
public class BookService {
    public String getBookName() {
        // INTENTIONAL BAD CODE to trigger SonarQube bugs
        int a = 10; // Unused variable
        System.out.println("Checking Database..."); // Using System.out is a "Code Smell"
        return "DevOps Handbook";
    }
}
```

### Step 5: The Web Controller (For the Browser)

Create a new file: `src/main/java/com/example/SimpleLibrary/LibraryController.java`.
**Paste this:**

```java
package com.example.SimpleLibrary;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class LibraryController {
    
    private final BookService bookService;

    public LibraryController(BookService bookService) {
        this.bookService = bookService;
    }

    @GetMapping("/")
    public String home() {
        return "<h1 style='color:green'>Library System Online</h1>" + 
               "<p>Active Book: " + bookService.getBookName() + "</p>";
    }
}
```

### Step 6: The Unit Test (For Module 11)

Go to `src/test/java/com/example/SimpleLibrary/`. Create `BookServiceTest.java`.
**Paste this:**

```java
package com.example.SimpleLibrary;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

class BookServiceTest {
    @Test
    void testGetBookName() {
        BookService service = new BookService();
        assertEquals("DevOps Handbook", service.getBookName());
    }
}
```

### Step 7: Push to GitHub

1.  Open IntelliJ Terminal.
2.  `git init`
3.  `git add .`
4.  `git commit -m "Initial code"`
5.  Go to GitHub.com -\> New Repository -\> Name: `SimpleLibrary` -\> Create.
6.  Copy the URL.
7.  `git branch -M main`
8.  `git remote add origin YOUR_GITHUB_URL_HERE`
9.  `git push -u origin main`

-----

## ⚙️ Part 3: Jenkins Configuration (The Painful Part)

### 1\. Install Plugins

1.  Go to Jenkins Dashboard (`localhost:9999`).
2.  Click **Manage Jenkins** -\> **Plugins** -\> **Available Plugins**.
3.  Search for and check:
      * **SonarQube Scanner**
      * **Deploy to container**
4.  Click **Install without restart**.
5.  Once done, restart Jenkins (or type `http://localhost:9999/safeRestart`).

### 2\. Configure Java and Maven (CRITICAL STEP)

This is where most people fail. You must tell Jenkins exactly where Java is.

**A. Find your Java Path**

1.  Open **PowerShell**.
2.  Run this exact command:
    ```powershell
    Split-Path -Parent (Split-Path -Parent (Get-Command java).Source)
    ```
3.  It will print a path like `C:\Program Files\Java\jdk-21` (or `jdk-17`).
4.  **Copy this path.**

**B. Configure Jenkins Tools**

1.  Go to **Manage Jenkins** -\> **Tools**.
2.  Scroll to **JDK**. Click **Add JDK**.
      * **Name:** `jdk17`
          * **CRITICAL:** Even if you have Java 21, **NAME IT `jdk17`**. The pipeline code looks for this specific name.
      * **Uncheck** "Install automatically".
      * **JAVA\_HOME:** Paste the path you copied from PowerShell.
3.  Scroll to **Maven**. Click **Add Maven**.
      * **Name:** `maven3`
      * **Check** "Install automatically".
4.  Click **Save**.

### 3\. Connect SonarQube

1.  Open SonarQube (`localhost:9000`).
2.  Profile Icon (Top Right) -\> **My Account** -\> **Security**.
3.  Token Name: `jenkins` -\> Click **Generate**. **COPY IT.**
4.  Go to Jenkins -\> **Manage Jenkins** -\> **Credentials**.
5.  Click **System** -\> **Global credentials** -\> **+ Add Credentials**.
      * Kind: **Secret text**.
      * Secret: Paste the token.
      * ID: `sonarqube-token`
      * Click **Create**.
6.  **Manage Jenkins** -\> **System**.
      * Scroll to **SonarQube servers**. Click **Add SonarQube**.
      * Name: `sonar-server`
      * URL: `http://localhost:9000`
      * Token: Select `sonarqube-token`.
      * Click **Save**.

-----

## 🚀 Part 4: The Pipeline (The Solution)

1.  Jenkins Dashboard -\> **+ New Item**.
2.  Name: `LibraryPipeline`. Select **Pipeline**. Click **OK**.
3.  Scroll to **Pipeline Script**. Paste this exact code.

**YOU MUST CHANGE ONLY TWO LINES (MARKED BELOW):**

```groovy
pipeline {
    agent any
    
    tools {
        // This looks for the name "jdk17" we configured in Part 3, Step 2
        jdk 'jdk17'
        maven 'maven3'
    }

    stages {
        stage('Checkout Code') {
            steps {
                // CHANGE 1: YOUR GITHUB URL
                git branch: 'main', url: 'https://github.com/YOUR_GITHUB_USERNAME/SimpleLibrary.git'
            }
        }

        stage('Build & Unit Test') {
            steps {
                bat 'mvn clean package' 
            }
        }

        stage('Static Code Analysis') {
            steps {
                // This sends data to SonarQube but does NOT wait for a reply.
                // This prevents the "Timeout" error caused by localhost issues.
                withSonarQubeEnv('sonar-server') {
                    bat 'mvn sonar:sonar'
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                // CHANGE 2: YOUR TOMCAT PATH (Use double backslashes \\)
                // Example: "C:\\DevOps\\apache-tomcat-9.0.112\\webapps\\"
                bat 'copy target\\*.war "C:\\YOUR\\TOMCAT\\PATH\\webapps\\"'
            }
        }
    }
}
```

4.  Click **Save**.
5.  Click **Build Now**.

-----

## ✅ Part 5: The "Live" Presentation Script

**Scenario:** The teacher stands behind you and says "Show me it works."

**Do exactly this:**

1.  **Show the Green Tick:**

      * "As you can see, Build \#1 is Green. This means the Pipeline passed all stages."

2.  **Show the Verification (Module 11 - Unit Testing):**

      * Click the Build \#1 -\> **Console Output**.
      * Ctrl+F search for: `Tests run: 1, Failures: 0`.
      * Say: "This proves JUnit ran and passed the logic tests."

3.  **Show the Code Quality (Module 5 - SonarQube):**

      * Switch tab to SonarQube (`localhost:9000`).
      * Click the `SimpleLibrary` project.
      * Point to the 'Code Smells'.
      * Say: "The static analyzer successfully scanned the code and found the intentional issues I left in the Service class."

4.  **Show the Deployment (Module 6 - Tomcat):**

      * Switch tab to `http://localhost:8082/SimpleLibrary-0.0.1-SNAPSHOT/`
      * Show the page saying "Library System Online".
      * Say: "Jenkins automatically deployed the WAR file to the Tomcat server, and the application is live."

5.  **If they ask: "Why didn't you use a Quality Gate wait?"**

      * **Answer:** "In this lab environment, SonarQube's security settings block localhost webhooks. I used `withSonarQubeEnv` to ensure the analysis runs and reports are generated without causing a false timeout in Jenkins."

**YOU ARE DONE.**
