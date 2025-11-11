+++
title = "2. Build Your First Spring Boot Web Application"
weight = 2
date = 2025-11-11
draft = false
tags = ["week1", "JIN"]
+++

## Goal

Create a simple web application with a static landing page and a dynamic "Hello World" page using Spring MVC and Thymeleaf templates.

> **What you'll learn:**
>
> - How to serve static content with Spring Boot
> - When to use server-side rendering with Thymeleaf
> - Best practices for the Model-View-Controller pattern

## Prerequisites

> **Before starting, ensure you have:**
>
> - ✓ Completed the "Set Up Your Local Spring Boot Development Environment" tutorial
> - ✓ Spring Boot project running successfully
> - ✓ Basic understanding of HTML

## Exercise Steps

### Overview

1. **Create Static Landing Page**
2. **Add Thymeleaf Dependency**
3. **Create Hello Controller**
4. **Create Hello Template**
5. **Test Your Application**

### **Step 1:** Create Static Landing Page

Build a simple landing page with Bootstrap styling as the entry point to your application. Spring Boot automatically serves static content from the `static` directory, making it easy to add HTML, CSS, JavaScript, and images without any configuration.

1. **Navigate to** the `src/main/resources/static` directory

2. **Create** a new file named `index.html`

3. **Add** the following HTML code:

   > `src/main/resources/static/index.html`

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>HelloJava - Home</title>
       <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
   </head>
   <body>
       <div class="container">
           <div class="row min-vh-100 align-items-center">
               <div class="col-lg-8 mx-auto text-center">
                   <h1 class="display-1 fw-bold mb-4">Welcome to HelloJava</h1>
                   <p class="lead mb-4">
                       A simple Spring Boot web application demonstrating static and dynamic content.
                   </p>
                   <a href="/hello" class="btn btn-primary btn-lg">Hello</a>
               </div>
           </div>
       </div>
   </body>
   </html>
   ```

> ℹ **Concept Deep Dive**
>
> Spring Boot automatically serves static resources from `src/main/resources/static` at the root path (`/`). This means `index.html` becomes your default homepage at `http://localhost:8080/`. Static content is served directly by the embedded Tomcat server without involving Spring MVC controllers, making it very fast. This approach is perfect for landing pages, CSS files, JavaScript, and images that don't require server-side processing.
>
> ⚠ **Common Mistakes**
>
> - Placing `index.html` in the wrong directory (like `templates/`) prevents it from being served
> - Using `src/main/webapp` instead of `src/main/resources/static` won't work with Spring Boot's embedded server
> - Not including the viewport meta tag makes the page look poor on mobile devices
>
> ✓ **Quick check:** File created at `src/main/resources/static/index.html` with proper HTML structure

### **Step 2:** Add Thymeleaf Dependency

Add the Thymeleaf template engine to enable server-side HTML rendering with dynamic content. Thymeleaf integrates seamlessly with Spring Boot and allows you to create dynamic web pages using natural HTML templates.

1. **Open** the `pom.xml` file in your project root

2. **Locate** the `<dependencies>` section (around line 32)

3. **Add** the Thymeleaf starter dependency after the existing `spring-boot-starter-web` dependency:

   > `pom.xml`

   ```xml
   <dependencies>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-web</artifactId>
       </dependency>

       <!-- Add this dependency -->
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-thymeleaf</artifactId>
       </dependency>

       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-test</artifactId>
           <scope>test</scope>
       </dependency>
   </dependencies>
   ```

4. **Save** the file and let Maven download the dependency

> ℹ **Concept Deep Dive**
>
> Thymeleaf is a server-side template engine that processes HTML templates on the server before sending them to the browser. Unlike static HTML, Thymeleaf templates can include dynamic data from your Java code using special attributes (like `th:text`). Spring Boot auto-configures Thymeleaf when it detects the dependency, setting up default locations (`src/main/resources/templates/`) and file extensions (`.html`). This zero-configuration approach is a key benefit of Spring Boot's convention-over-configuration philosophy.
>
> ⚠ **Common Mistakes**
>
> - Forgetting to save `pom.xml` means Maven won't download the dependency
> - Typos in the artifactId will cause "artifact not found" errors
> - If your IDE doesn't auto-import, you may need to manually refresh Maven dependencies
>
> ✓ **Quick check:** Maven successfully downloads the dependency (check for errors in the Maven output or IDE)

### **Step 3:** Create Hello Controller

Build a Spring MVC controller to handle requests to the `/hello` endpoint and pass data to the view. Controllers are the entry point for HTTP requests in Spring MVC, acting as the "C" in the Model-View-Controller pattern.

1. **Navigate to** `src/main/java/com/jin/hellojava`

2. **Create** a new directory named `controller`

3. **Create** a new file named `HelloController.java` inside the `controller` directory

4. **Add** the following code:

   > `src/main/java/com/jin/hellojava/controller/HelloController.java`

   ```java
   package com.jin.hellojava.controller;

   import org.springframework.stereotype.Controller;
   import org.springframework.ui.Model;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RequestParam;

   @Controller
   public class HelloController {

       @GetMapping("/hello")
       public String hello(@RequestParam(value = "name", defaultValue = "World") String name, Model model) {
           // Add the name to the model so it's available in the view
           model.addAttribute("name", name);

           // Return the view name (Spring resolves this to templates/hello.html)
           return "hello";
       }
   }
   ```

> ℹ **Concept Deep Dive**
>
> The `@Controller` annotation marks this class as a Spring MVC controller, making it eligible for component scanning and request mapping. The `@GetMapping("/hello")` annotation maps HTTP GET requests to this method. The `@RequestParam` annotation extracts the `name` query parameter from the URL (e.g., `/hello?name=Alice`) with a default value of "World" if not provided. The `Model` object acts as a container for data passed from the controller to the view. Returning "hello" tells Spring to render the `templates/hello.html` template.
>
> **Code Formatting Note:** This code follows the K&R style (Kernighan and Ritchie style), which is the standard Java formatting convention. Method parameters are kept on the same line when they fit comfortably (under ~90-120 characters), and the opening curly brace `{` stays on the same line as the method signature rather than on a new line. This style is used by the Oracle Java Style Guide, Google Java Style Guide, and Spring Framework Code Style.
>
> ⚠ **Common Mistakes**
>
> - Using `@RestController` instead of `@Controller` causes the method to return the string "hello" as plain text instead of rendering the template
> - Forgetting `defaultValue` means the parameter becomes required, causing errors when accessing `/hello` without parameters
> - Placing the controller outside the `com.jin.hellojava` package prevents Spring Boot from discovering it
> - Not adding data to the Model means the view won't have access to it
>
> ✓ **Quick check:** No compilation errors and IntelliSense recognizes Spring annotations

### **Step 4:** Create Hello Template

Create a Thymeleaf template to display the dynamic greeting message. Templates combine static HTML structure with dynamic data from your controller, enabling server-side rendering.

1. **Navigate to** `src/main/resources/templates`

2. **Create** a new file named `hello.html`

3. **Add** the following template code:

   > `src/main/resources/templates/hello.html`

   ```html
   <!DOCTYPE html>
   <html xmlns:th="http://www.thymeleaf.org">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>Hello - HelloJava</title>
       <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
   </head>
   <body>
       <div class="container mt-5">
           <div class="row">
               <div class="col-lg-8 mx-auto text-center">
                   <h1 class="display-4 mb-4" th:text="|Hello, ${name}!|">Hello, World!</h1>
                   <p class="lead mb-4">
                       This message is dynamically generated using Spring MVC and Thymeleaf.
                   </p>
                   <a href="/" class="btn btn-secondary">Back to Home</a>
               </div>
           </div>
       </div>
   </body>
   </html>
   ```

> ℹ **Concept Deep Dive**
>
> The `xmlns:th="http://www.thymeleaf.org"` namespace declaration enables Thymeleaf attributes in your HTML. The `th:text` attribute replaces the element's text content with the dynamic value. The pipe syntax `|Hello, ${name}!|` is Thymeleaf's literal substitution, combining static text with variable values. The `${name}` expression accesses the "name" attribute from the Model. The content "Hello, World!" inside the tag is shown only when viewing the template directly (not processed by Thymeleaf), making it a natural template that designers can preview.
>
> ⚠ **Common Mistakes**
>
> - Forgetting the Thymeleaf namespace declaration causes `th:` attributes to be ignored
> - Using `templates` directory without the `.html` extension prevents Spring from finding the template
> - Misspelling the Model attribute name (e.g., `${Name}` instead of `${name}`) shows empty or error text
> - Placing template in `static/` instead of `templates/` causes it to be served as static HTML
>
> ✓ **Quick check:** File created at `src/main/resources/templates/hello.html` with valid HTML and Thymeleaf syntax

### **Step 5:** Test Your Application

Verify that both the static landing page and dynamic hello page work correctly. Testing ensures all components are properly connected and functioning as expected.

1. **Run** the application using Maven wrapper:

   ```bash
   ./mvnw spring-boot:run
   ```

   On Windows, use:

   ```bash
   mvnw.cmd spring-boot:run
   ```

2. **Wait for** the application to start. Look for the "Started HelloJavaApplication" message

3. **Test the landing page:**

   - Open your browser and navigate to: `http://localhost:8080/`
   - Verify the Bootstrap hero section displays correctly
   - Confirm the "Hello" button is visible

4. **Test the Hello button:**

   - Click the "Hello" button on the landing page
   - Verify you're redirected to `/hello`
   - Confirm the page displays "Hello, World!"

5. **Test with query parameters:**

   - Navigate to: `http://localhost:8080/hello?name=Alice`
   - Verify the page displays "Hello, Alice!"
   - Try different names: `http://localhost:8080/hello?name=Spring`
   - Confirm each shows the correct personalized greeting

6. **Test navigation:**

   - Click "Back to Home" on the hello page
   - Verify you return to the landing page
   - Confirm the navigation flow works both directions

> ✓ **Success indicators:**
>
> - Landing page loads at `http://localhost:8080/` with proper Bootstrap styling
> - "Hello" button navigates to `/hello` endpoint
> - Default greeting shows "Hello, World!" at `/hello`
> - Custom name parameter displays correctly (e.g., "Hello, Alice!" at `/hello?name=Alice`)
> - Navigation between pages works smoothly
> - No errors in browser console or application logs

> ✓ **Final verification checklist:**
>
> - ☐ `index.html` exists in `src/main/resources/static/`
> - ☐ Thymeleaf dependency added to `pom.xml`
> - ☐ `HelloController.java` exists in `controller` package
> - ☐ `hello.html` template exists in `src/main/resources/templates/`
> - ☐ Application runs without errors
> - ☐ Static landing page displays correctly
> - ☐ Dynamic hello page shows personalized greeting
> - ☐ Query parameters work as expected

## Common Issues

> **If you encounter problems:**
>
> **"Whitelabel Error Page" at /hello:** Controller not found - verify `HelloController.java` is in the `com.jin.hellojava.controller` package and has `@Controller` annotation
>
> **Template not found error:** Check that `hello.html` is in `src/main/resources/templates/` (not `static/`) and Thymeleaf dependency is in `pom.xml`
>
> **"Hello, World!" doesn't change with name parameter:** Verify `th:text` attribute syntax is correct and uses `${name}` to reference the Model attribute
>
> **404 error on landing page:** Confirm `index.html` is in `src/main/resources/static/` directory
>
> **Bootstrap styles not loading:** Check internet connection (CDN link requires network access) or download Bootstrap locally
>
> **Still stuck?** Check the console logs for specific error messages and stack traces

## Summary

You've successfully created a Spring Boot web application with both static and dynamic content:

- ✓ Static landing page served from the `static` directory
- ✓ Dynamic hello page using Spring MVC and Thymeleaf
- ✓ Query parameter handling with default values

> **Key takeaway:** Spring Boot makes it simple to serve both static content (for performance) and dynamic content (for personalization) in the same application. Static resources are served directly, while controllers and templates enable server-side rendering of dynamic pages. This pattern is fundamental to traditional web applications and provides excellent SEO and initial page load performance.

## Going Deeper (Optional)

> **Want to explore more?**
>
> - Try adding CSS styling in `static/css/style.css` and linking it in your templates
> - Add multiple request parameters (e.g., `?name=Alice&greeting=Welcome`)
> - Create additional controller methods for different pages
> - Explore Thymeleaf's conditional rendering with `th:if`
> - Add a form that submits data to a controller method

## Done! 🎉

Excellent work! You've learned the basics of Spring MVC, server-side rendering with Thymeleaf, and how Spring Boot serves static content. You can now build web applications that combine static pages for performance with dynamic pages for personalized content.
