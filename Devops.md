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
