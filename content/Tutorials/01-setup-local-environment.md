+++
title = "1. Set Up Your Local Spring Boot Development Environment"
weight = 1
date = 2025-11-11
draft = false
tags = ["week1", "JIN"]
+++

## Goal

Set up a complete Java development environment and create your first Spring Boot application from scratch.

> **What you'll learn:**
>
> - How to install and configure Java development tools
> - When to use Spring Initializr for project generation
> - Best practices for Spring Boot project structure

## Prerequisites

> **Before starting, ensure you have:**
>
> - ✓ A computer with internet access
> - ✓ Administrator/sudo privileges to install software
> - ✓ Basic familiarity with command line/terminal

## Exercise Steps

### Overview

1. **Install Java Development Kit (JDK)**
2. **Install Apache Maven**
3. **Generate Spring Boot Project**
4. **Explore Project Structure**
5. **Run Your First Spring Boot Application**

### **Step 1:** Install Java Development Kit (JDK)

Install the Java Development Kit to compile and run Java applications. Spring Boot 3.x requires JDK 17 or later to take advantage of modern Java features and long-term support.

1. **Download** the JDK from one of these sources:

   - Oracle JDK: <https://www.oracle.com/java/technologies/downloads/>
   - Eclipse Temurin (recommended): <https://adoptium.net/>

2. **Choose** JDK version 21 (recommended LTS version)

3. **Install** the JDK for your operating system:

   <details>
   <summary><b>Windows Installation</b></summary>

   1. **Download** the `.msi` installer from Eclipse Temurin
   2. **Run** the installer executable
   3. **Follow** the installation wizard:
      - Accept the license agreement
      - Choose installation directory (default is recommended)
      - **Important:** Check "Add to PATH" option
      - **Important:** Check "Set JAVA_HOME variable" option
   4. **Complete** the installation by clicking Finish
   5. **Restart** your terminal/command prompt for PATH changes to take effect

   </details>

   <details>
   <summary><b>macOS Installation</b></summary>

   **Option A: Using Homebrew (recommended)**

   1. **Install** Homebrew if not already installed: <https://brew.sh>
   2. **Run** the following command:

      ```bash
      brew install openjdk@21
      ```

   3. **Link** the JDK (follow the instructions displayed after installation):

      ```bash
      sudo ln -sfn /opt/homebrew/opt/openjdk@21/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-21.jdk
      ```

   **Option B: Manual Installation**

   1. **Download** the `.pkg` installer from Eclipse Temurin
   2. **Run** the installer package
   3. **Follow** the installation wizard prompts
   4. **Complete** the installation

   </details>

   <details>
   <summary><b>Linux Installation</b></summary>

   **Ubuntu/Debian:**

   ```bash
   sudo apt update
   sudo apt install openjdk-21-jdk
   ```

   **Fedora/RHEL/CentOS:**

   ```bash
   sudo dnf install java-21-openjdk-devel
   ```

   **Arch Linux:**

   ```bash
   sudo pacman -S jdk21-openjdk
   ```

   </details>

4. **Verify** the installation by opening a terminal and running:

   ```bash
   java -version
   ```

   Expected output should show Java 21 or later:

   ```bash
   openjdk version "21.0.x" 2024-xx-xx
   ```

> ℹ **Concept Deep Dive**
>
> The JDK (Java Development Kit) includes the Java compiler, runtime environment, and development tools. LTS (Long-Term Support) versions like 17, 21, and 25 receive updates and security patches for several years, making them ideal for production applications. Spring Boot 3.x requires Java 17 as the minimum version and supports up to Java 25. Java 21 is recommended for its stability and widespread adoption.
>
> ⚠ **Common Mistakes**
>
> - Installing JRE (Java Runtime Environment) instead of JDK - JRE cannot compile Java code
> - Not setting JAVA_HOME environment variable on some systems
> - Installing a JDK version older than 17 won't work with Spring Boot 3.x
>
> ✓ **Quick check:** The `java -version` command displays version 21 or higher

### **Step 2:** Install Apache Maven

Install Maven to manage dependencies and build your Spring Boot application. Maven automates the build process and downloads required libraries automatically.

1. **Install** Maven for your operating system:

   <details>
   <summary><b>Windows Installation</b></summary>

   1. **Download** Maven 3.9.11 from: <https://maven.apache.org/download.cgi>
      - Select `apache-maven-3.9.11-bin.zip`

   2. **Extract** the archive to a directory (e.g., `C:\Program Files\Apache\maven`)

   3. **Set environment variables:**
      - Open System Properties → Environment Variables
      - **Add** new system variable:
        - Variable name: `MAVEN_HOME`
        - Variable value: `C:\Program Files\Apache\maven` (your Maven directory)
      - **Edit** the `Path` system variable:
        - Add new entry: `%MAVEN_HOME%\bin`

   4. **Restart** your terminal/command prompt for changes to take effect

   </details>

   <details>
   <summary><b>macOS Installation</b></summary>

   **Option A: Using Homebrew (recommended)**

   1. **Install** Maven with a single command:

      ```bash
      brew install maven
      ```

      This automatically installs the latest version and configures the PATH.

   **Option B: Manual Installation**

   1. **Download** Maven 3.9.11 from: <https://maven.apache.org/download.cgi>
      - Select `apache-maven-3.9.11-bin.tar.gz`

   2. **Extract** to `/opt`:

      ```bash
      sudo tar -xzvf apache-maven-3.9.11-bin.tar.gz -C /opt
      ```

   3. **Add** Maven to your shell profile (`~/.zshrc` for zsh or `~/.bash_profile` for bash):

      ```bash
      export M2_HOME=/opt/apache-maven-3.9.11
      export PATH=$M2_HOME/bin:$PATH
      ```

   4. **Reload** your shell configuration:

      ```bash
      source ~/.zshrc  # or source ~/.bash_profile
      ```

   </details>

   <details>
   <summary><b>Linux Installation</b></summary>

   **Option A: Using Package Manager (if available)**

   **Ubuntu/Debian:**

   ```bash
   sudo apt update
   sudo apt install maven
   ```

   **Fedora/RHEL/CentOS:**

   ```bash
   sudo dnf install maven
   ```

   **Option B: Manual Installation (latest version)**

   1. **Download** Maven 3.9.11:

      ```bash
      wget https://dlcdn.apache.org/maven/maven-3/3.9.11/binaries/apache-maven-3.9.11-bin.tar.gz
      ```

   2. **Extract** to `/opt`:

      ```bash
      sudo tar -xzvf apache-maven-3.9.11-bin.tar.gz -C /opt
      ```

   3. **Create** symbolic link:

      ```bash
      sudo ln -s /opt/apache-maven-3.9.11 /opt/maven
      ```

   4. **Add** Maven to PATH by editing `/etc/profile.d/maven.sh`:

      ```bash
      sudo nano /etc/profile.d/maven.sh
      ```

      Add these lines:

      ```bash
      export M2_HOME=/opt/maven
      export PATH=$M2_HOME/bin:$PATH
      ```

   5. **Make** the script executable and load it:

      ```bash
      sudo chmod +x /etc/profile.d/maven.sh
      source /etc/profile.d/maven.sh
      ```

   </details>

2. **Verify** the installation:

   ```bash
   mvn -version
   ```

   Expected output:

   ```bash
   Apache Maven 3.9.11
   Maven home: /path/to/maven
   Java version: 21.0.x
   ```

> ℹ **Concept Deep Dive**
>
> Maven is a build automation tool that manages project dependencies, compiles code, runs tests, and packages applications. Spring Boot projects include a Maven Wrapper (`mvnw`), which means you can skip this step if you prefer - the wrapper will download Maven automatically when you first run it.
>
> ⚠ **Common Mistakes**
>
> - Forgetting to add Maven to PATH means you can't run `mvn` from any directory
> - Maven requires JAVA_HOME to be set correctly to find the JDK - it should point to JDK 21
> - Not restarting the terminal after PATH changes means the new settings won't take effect
> - Using package manager Maven might install an older version - check the version after installation
>
> ✓ **Quick check:** Running `mvn -version` shows Maven 3.9.x and displays Java version 21.0.x

### **Step 3:** Generate Spring Boot Project

Use Spring Initializr to create a new Spring Boot project with all necessary configuration files. This online tool generates a complete project structure following Spring Boot best practices.

<details>
<summary><b>Option A: Using Web Interface (Recommended)</b></summary>

1. **Navigate to** <https://start.spring.io> in your web browser

2. **Configure** the project settings in the left panel:

   **Project:**
   - Select: **Maven**

   **Language:**
   - Select: **Java**

   **Spring Boot:**
   - Select: **3.5.7** (the currently selected stable version)

   **Project Metadata:**
   - Group: `com.jin`
   - Artifact: `hellojava`
   - Name: `HelloJava`
   - Description: `Spring Boot Demo Application for Azure`
   - Package name: `com.jin.hellojava`

   **Packaging:**
   - Select: **Jar** (should be selected by default)

   **Configuration:**
   - Keep: **Properties** (default)

   **Java:**
   - Select: **21** (change from 17 if needed)

3. **Add** dependencies by clicking "ADD DEPENDENCIES..." button in the top right:

   - Search for and add: **Spring Web**
   - This enables building web applications and REST APIs

4. **Click** the "GENERATE" button at the bottom to download `hellojava.zip`

5. **Navigate** to your project root directory in terminal:

   ```bash
   cd /Users/lasse/Developer/JIN_Development/HelloJava
   ```

6. **Move** the downloaded zip file to the current directory if needed

</details>

<details>
<summary><b>Option B: Using curl (Command Line)</b></summary>

1. **Navigate** to your project root directory in terminal:

   ```bash
   cd /Users/lasse/Developer/JIN_Development/HelloJava
   ```

2. **Generate and download** the project directly using curl:

   ```bash
   curl https://start.spring.io/starter.zip \
     -d type=maven-project \
     -d language=java \
     -d bootVersion=3.5.7 \
     -d baseDir=hellojava \
     -d groupId=com.jin \
     -d artifactId=hellojava \
     -d name=HelloJava \
     -d description='Spring Boot Demo Application for Azure' \
     -d packageName=com.jin.hellojava \
     -d packaging=jar \
     -d javaVersion=21 \
     -d dependencies=web \
     -o hellojava.zip
   ```

   This creates `hellojava.zip` directly in your current directory with all the correct settings.

</details>

7. **Extract** the zip file and move contents to project root (to preserve hidden files):

   **On Mac/Linux:**

   ```bash
   unzip hellojava.zip -d . && mv hellojava/{*,.*} . 2>/dev/null; rmdir hellojava && rm hellojava.zip
   ```

   **On Windows (PowerShell):**

   ```bash
   Expand-Archive -Path hellojava.zip -DestinationPath .
   Get-ChildItem -Path hellojava -Force | Move-Item -Destination . -Force
   Remove-Item -Recurse -Force hellojava
   Remove-Item hellojava.zip
   ```

   This extracts the zip, moves all files (including hidden files) to the project root, removes the empty directory, and deletes the zip file

> ℹ **Concept Deep Dive**
>
> Spring Initializr generates a complete Maven project with `pom.xml` (dependency configuration), main application class, test class, and configuration files. The Spring Web dependency includes embedded Tomcat server and Spring MVC framework, allowing you to build web applications without installing a separate application server. The Maven Wrapper files (`mvnw` and `mvnw.cmd`) allow running Maven commands even without Maven installed.
>
> ⚠ **Common Mistakes**
>
> - Wrong group/package naming prevents Spring Boot from finding components
> - Choosing "War" packaging instead of "Jar" requires additional servlet container setup
> - Forgetting to add Spring Web dependency means no web capabilities
>
> ✓ **Quick check:** The extracted directory contains `pom.xml`, `mvnw`, and `src` folder

### **Step 4:** Explore Project Structure

Understand the standard Spring Boot project layout to know where to place your code and resources. This conventional structure helps developers quickly navigate any Spring Boot project.

1. **Open** the project folder in your preferred text editor or IDE

2. **Review** the key directories and files:

   ```text
   project-root/
   ├── .mvn/
   │   └── wrapper/
   │       └── maven-wrapper.properties
   ├── src/
   │   ├── main/
   │   │   ├── java/com/jin/hellojava/
   │   │   │   └── HelloJavaApplication.java
   │   │   └── resources/
   │   │       ├── application.properties
   │   │       ├── static/
   │   │       └── templates/
   │   └── test/
   │       └── java/com/jin/hellojava/
   │           └── HelloJavaApplicationTests.java
   ├── .gitattributes    (Git line ending configuration)
   ├── .gitignore        (Git ignore rules)
   ├── HELP.md           (Spring Initializr help documentation)
   ├── mvnw              (Maven wrapper for Mac/Linux)
   ├── mvnw.cmd          (Maven wrapper for Windows)
   └── pom.xml           (Maven project configuration)
   ```

3. **Examine** the main application class at `src/main/java/com/jin/hellojava/HelloJavaApplication.java`:

   > `src/main/java/com/jin/hellojava/HelloJavaApplication.java`

   ```java
   package com.jin.hellojava;

   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;

   @SpringBootApplication
   public class HelloJavaApplication {

       public static void main(String[] args) {
           SpringApplication.run(HelloJavaApplication.class, args);
       }
   }
   ```

> ℹ **Concept Deep Dive**
>
> The `src/main/java` directory contains your application code. The `src/main/resources` directory holds configuration files, static assets (CSS, JavaScript), and templates. The `@SpringBootApplication` annotation enables auto-configuration, component scanning, and configuration properties. The `pom.xml` file defines project dependencies and build configuration. Maven Wrapper scripts ensure everyone uses the same Maven version - use `mvnw` on Mac/Linux or `mvnw.cmd` on Windows. The wrapper files (`mvnw`, `mvnw.cmd`, and `.mvn/wrapper/`) should be committed to version control so others can build the project without installing Maven.
>
> ⚠ **Common Mistakes**
>
> - Placing classes outside the main package prevents Spring Boot's component scanning
> - Modifying the main application class name without updating configuration
> - Confusing `resources` (for config/assets) with `java` (for code)
> - Adding Maven wrapper files to `.gitignore` - they should be committed
>
> ✓ **Quick check:** All standard directories exist and the main application class has `@SpringBootApplication` annotation

### **Step 5:** Run Your First Spring Boot Application

Start the Spring Boot application to verify everything is configured correctly. Even without adding custom code, Spring Boot will start an embedded web server.

1. **Run** the application using Maven wrapper:

   ```bash
   ./mvnw spring-boot:run
   ```

   On Windows, use:

   ```bash
   mvnw.cmd spring-boot:run
   ```

2. **Wait for** the startup to complete. Look for this message in the console:

   ```text
   Started HelloJavaApplication in X.XXX seconds
   ```

3. **Stop** the application by pressing `Ctrl+C` in the terminal

> ℹ **Concept Deep Dive**
>
> Spring Boot starts an embedded Tomcat server on port 8080 by default. The `spring-boot:run` Maven goal compiles your code and starts the application in development mode with automatic classpath setup. The startup time includes initializing the Spring context, auto-configuration, and starting the embedded server. You haven't created any endpoints yet, but the application successfully starts and is ready to handle HTTP requests.
>
> ⚠ **Common Mistakes**
>
> - Port 8080 already in use: change the port in `application.properties` with `server.port=8081`
> - Build failures: ensure JDK 17+ is being used by Maven
> - Permission denied on `mvnw`: run `chmod +x mvnw` on Linux/Mac
>
> ✓ **Quick check:** Application starts successfully and displays "Started HelloJavaApplication" message

> ✓ **Success indicators:**
>
> - Application starts without errors
> - Console shows Spring Boot banner and startup logs
> - Application can be stopped cleanly with Ctrl+C
>
> ✓ **Final verification checklist:**
>
> - ☐ JDK 21 installed and verified
> - ☐ Maven installed or wrapper scripts work
> - ☐ Spring Boot project generated and extracted
> - ☐ Project structure matches expected layout
> - ☐ Application starts and stops successfully

## Common Issues

> **If you encounter problems:**
>
> **"java: command not found":** JDK not installed or not in PATH. Reinstall JDK and verify PATH configuration
>
> **"mvnw: Permission denied":** Run `chmod +x mvnw` on Linux/Mac to make the wrapper executable
>
> **"Port 8080 already in use":** Another application is using the port. Create `src/main/resources/application.properties` and add `server.port=8081`
>
> **"UnsupportedClassVersionError":** Java version mismatch. Verify JDK 21 is installed and Maven is using it
>
> **Build errors:** Delete the `.m2/repository` folder in your home directory and rerun to download fresh dependencies
>
> **Still stuck?** Check the full error stack trace in the console output for specific issues

## Summary

You've successfully set up a Spring Boot development environment which:

- ✓ Includes all necessary tools (JDK, Maven)
- ✓ Generated a proper Spring Boot project structure
- ✓ Runs a working Spring Boot application

> **Key takeaway:** Spring Boot's auto-configuration and embedded server make it incredibly fast to start building web applications. No complex server setup or XML configuration required - just download the JDK, generate a project, and run. This streamlined development experience is why Spring Boot is the most popular Java framework for modern applications.

## Going Deeper (Optional)

> **Want to explore more?**
>
> - Install an IDE like IntelliJ IDEA Community Edition or Eclipse
> - Explore the generated `pom.xml` to understand Maven dependencies
> - Try the command-line alternative: generate projects using curl to Spring Initializr API
> - Add Spring Boot DevTools dependency for automatic restart on code changes
> - Explore Spring Boot Actuator for production-ready features

## Done! 🎉

Excellent work! You've set up a complete Java development environment and created your first Spring Boot application. This foundation will support all your future Spring Boot development. You're now ready to add controllers, services, and build real applications.
