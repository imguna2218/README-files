# DevOps Lab Environment Documentation

**Date:** November 27, 2025
**System:** Windows 10/11 (64-bit)
**Java Version:** OpenJDK 21 (LTS)

## 1\. Environment Preparation (The Java Fix)

**Objective:** Establish a clean Java 21 environment and remove legacy conflicts.

  * **Conflict:** System originally had Java 1.8 (Eclipse Temurin) taking priority in `%PATH%`.
  * **Action 1:** Uninstalled "Eclipse Temurin JDK with Hotspot 8u472".
  * **Action 2:** Verified Java 21 installation path: `C:\Program Files\Java\jdk-21`.
  * **Action 3:** Configured System Environment Variables:
      * `JAVA_HOME`: Set to `C:\Program Files\Java\jdk-21`.
      * `Path`: Removed Java 1.8 entries. Added `C:\Program Files\Java\jdk-21\bin` to the top of the list.
  * **Verification:**
      * Command: `java -version`
      * Result: `java version "21.0.8" 2025-07-15 LTS`

## 2\. Directory Structure

**Base Directory:** `C:\DevOps` (Created to bypass `Program Files` permission issues).

**Contents:**

1.  `C:\DevOps\jenkins.war` (Executable)
2.  `C:\DevOps\apache-tomcat-9.0.98` (Extracted Folder)
3.  `C:\DevOps\sonarqube-10.7.0.96327` (Extracted Folder)

## 3\. Tool Installation & Configuration

### A. Jenkins (Automation Server)

  * **Source:** Generic Java WAR (v2.479.1).
  * **Port Conflict:** Default Port `8080` was occupied (likely Oracle DB). Port `8081` failed binding.
  * **Resolution:** Moved to Port **9999**.
  * **Setup:**
      * Unlocked using initial admin password from console.
      * Installed "Suggested Plugins".
      * Created Admin User (`admin`/`admin`).

### B. Apache Tomcat (Deployment Server)

  * **Source:** Tomcat 9 Core Zip (64-bit Windows).
  * **Port Conflict:** Default Port `8080` conflicted with Oracle/Jenkins.
  * **Configuration Change:**
      * File: `C:\DevOps\apache-tomcat-9.0.98\conf\server.xml`
      * Action: Changed `Connector port="8080"` to `Connector port="8082"`.

### C. SonarQube (Static Analysis)

  * **Source:** SonarQube Community Edition (v10.7.0).
  * **Java 21 Compatibility Issue:** SonarQube failed to start due to `java.lang.UnsupportedOperationException: The Security Manager is deprecated`.
  * **Configuration Change:**
      * File: `C:\DevOps\sonarqube-10.7.0.96327\conf\sonar.properties`
      * Action: Appended the following lines to the end of the file to allow Security Manager:
        ```properties
        sonar.web.javaOpts=-Xmx512m -Xms128m -XX:+HeapDumpOnOutOfMemoryError -Djava.security.manager=allow
        sonar.ce.javaOpts=-Xmx512m -Xms128m -XX:+HeapDumpOnOutOfMemoryError -Djava.security.manager=allow
        sonar.search.javaOpts=-Xmx512m -Xms512m -XX:+HeapDumpOnOutOfMemoryError -Djava.security.manager=allow
        ```
  * **Setup:**
      * Logged in with default `admin`/`admin`.
      * Forced password change to `admin123`.

## 4\. Operational Startup Commands

To bring the lab online, open **three separate** PowerShell (or CMD) windows and execute:

**1. Jenkins (Port 9999)**

```powershell
cd C:\DevOps
java -jar jenkins.war --httpPort=9999
```

*Access:* `http://localhost:9999`

**2. Tomcat (Port 8082)**

```powershell
cd C:\DevOps\apache-tomcat-9.0.98\bin
.\startup.bat
```

*Access:* `http://localhost:8082`

**3. SonarQube (Port 9000)**

```powershell
cd C:\DevOps\sonarqube-10.7.0.96327\bin\windows-x86-64
.\StartSonar.bat
```

*Access:* `http://localhost:9000`

## 5\. Application "Payload" (Spring Boot)

**Project:** SimpleLibrary
**Location:** `C:\Users\DELL\IdeaProjects\SimpleLibrary`
**Configuration:**

  * **Source:** start.spring.io
  * **Build Tool:** Maven
  * **Language:** Java 21
  * **Packaging:** **WAR** (Critical for Tomcat deployment)
  * **Dependencies:** Spring Web, Spring Boot DevTools.
  * **Status:** Project generated, extracted, and currently open in IntelliJ IDEA.

-----

**End of Report.**
