# Spring Boot + Auth0 End-to-End Setup from Scratch

This document provides a complete, practical, and concept-driven guide for integrating a **Spring Boot application with Auth0 using OIDC and OAuth2**.

It is based on a working setup where:

- Auth0 tenant was created successfully
- Auth0 application was created successfully
- Spring Boot application was configured successfully
- Browser login worked successfully
- Google social login through Auth0 worked successfully
- Secure API access worked successfully
- Deployment and testing on AWS EC2 worked successfully

This guide explains:

- what we are building
- why we are doing it
- the core concepts behind the integration
- how to configure Auth0
- how to configure Spring Boot
- how to run it on an AWS server
- how the login flow works end to end
- what happened in the successful test
- common issues and important clarifications

---

# 1. Why We Are Doing This

Modern applications should not manage usernames, passwords, MFA, social login, and identity lifecycle directly inside application code.

Instead, we integrate with an **Identity Provider (IdP)** such as Auth0.

## Why use Auth0 with Spring Boot?

Because Auth0 gives us:

- centralized authentication
- secure login handling
- MFA support
- social login support such as Google
- token issuance
- standards-based identity using OIDC and OAuth2
- reduced security burden inside application code

## Why use Spring Security with Auth0?

Because Spring Security can:

- redirect users to Auth0 for login
- receive the authorization code after login
- exchange the code for tokens
- validate JWT access tokens
- secure APIs and web endpoints
- create authenticated sessions for logged-in users

## Business Value

This pattern is useful for:

- enterprise applications
- internal portals
- admin dashboards
- microservices
- APIs requiring token-based security
- applications that need Google login or enterprise SSO

---

# 2. What We Are Building

We are building a Spring Boot application that supports both:

## A. Browser Login using OIDC

A user opens the application in a browser.

Spring Boot redirects the user to Auth0.

Auth0 authenticates the user directly or through Google.

After successful login, Auth0 redirects back to Spring Boot.

Spring Security creates an authenticated session.

## B. Secure APIs using JWT Access Tokens

The application also acts as a resource server.

That means:

- public APIs can be accessed without authentication
- secure APIs require a valid Bearer token
- Spring Boot validates the JWT using Auth0 issuer metadata

So the same application acts as:

- **OAuth2 Client** for login
- **OAuth2 Resource Server** for API token validation

---

# 3. Core Concepts You Must Understand

Before implementation, understand these concepts clearly.

## 3.1 Auth0

Auth0 is the **Identity Provider (IdP)**.

It handles:

- login
- signup
- password verification
- MFA
- social login
- token generation
- identity federation

Your application does not authenticate users directly. Auth0 does.

---

## 3.2 OIDC

OIDC stands for **OpenID Connect**.

It is built on top of OAuth2 and is used for **authentication**.

It answers:

- Who is the user?
- Has the user logged in successfully?

OIDC is what enables browser login and user identity claims.

---

## 3.3 OAuth2

OAuth2 is used for **authorization**.

It answers:

- Can this client access this API?
- Does this token allow access?

OAuth2 is the broader authorization framework.

---

## 3.4 JWT

JWT stands for **JSON Web Token**.

It is a signed token format used to carry claims such as:

- subject
- issuer
- audience
- scopes
- expiry
- identity information

Spring Boot validates JWTs using Auth0 metadata and public keys.

---
   
## 3.5 Issuer URI

The issuer URI identifies the authorization server.

In this setup:

```text
https://dev-rxbdreshrv2aylj4.us.auth0.com/
```

Spring Boot uses this issuer URI to:

- discover OAuth2/OIDC endpoints
- validate JWT tokens
- verify token issuer
- fetch signing keys

---

## 3.6 Authorization Code Flow

This is the login flow used by browser-based web applications.

Flow:

1. User opens Spring Boot app
2. Spring Boot redirects to Auth0
3. User logs in
4. Auth0 returns an authorization code
5. Spring Boot exchanges the code for tokens
6. Spring Security creates authenticated session

This is the correct flow for server-side web applications.

---

## 3.7 Resource Server

A resource server is an application that protects APIs using access tokens.

In this setup, Spring Boot validates Bearer tokens sent to secure endpoints.

---

# 4. Working Environment Used

This setup was tested successfully with:

- Auth0 Developer Dashboard
- Spring Boot 3.x
- Java 17
- Maven
- AWS EC2 Ubuntu server
- Browser-based login
- Google social login through Auth0

---

# 5. Auth0 Tenant Already Created

Create free Okta developer account

https://developer.okta.com/

The Auth0 tenant was created successfully.

Tenant domain:

```text
dev-rxbdreshrv2aylj4(us)
```

This domain is the base identity domain used in the Spring Boot configuration.

---

# 6. Create Spring Boot Application in Auth0

Inside Auth0 Dashboard:

1. Go to **Applications**
2. Click **Applications**
3. Click **Create Application**

## Configure the application

Fill:

- **Name**: `spring-boot-okta`
  - You can use any name, but this was the working example
- **Application Type**: `Regular Web Applications`

Then click:

- **Create**

Why `Regular Web Applications`?

Because Spring Boot is a server-side application that uses authorization code flow and maintains a backend session.

---

# 7. Configure Auth0 Application Settings

After the application is created:

1. Open the application
2. Go to **Settings**

Configure the following values.

## Allowed Callback URLs

For local testing:

```text
http://localhost:8080/login/oauth2/code/auth0
```

For EC2/public testing:

```text
http://YOUR_EC2_PUBLIC_IP:8080/login/oauth2/code/auth0
http://23.20.250.154:8080/login/oauth2/code/auth0
```

## Allowed Logout URLs

For local testing:

```text
http://localhost:8080/
```

For EC2/public testing:

```text
http://YOUR_EC2_PUBLIC_IP:8080/
http://23.20.250.154:8080/
```

## Allowed Web Origins

For local testing:

```text
http://localhost:8080
```

For EC2/public testing:

```text
http://YOUR_EC2_PUBLIC_IP:8080
http://23.20.250.154:8080
```

Then click:

- **Save Changes**

---

# 8. Copy Required Values from Auth0

From the Auth0 application page, copy:

## Domain

```text
dev-rxbdreshrv2aylj4.us.auth0.com
```

## Client ID

Use the value shown in the Auth0 application.

## Client Secret

Use the value shown in the Auth0 application.

These values are required by Spring Boot.

---

# 9. Prepare AWS EC2 Server

On the AWS Ubuntu server, install Java, Maven, and unzip.

Run:

```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
sudo apt install maven -y
sudo apt install unzip -y
java -version
mvn -version
```

---

# 10. Fix Java Version Properly

Ensure both `java` and `javac` point to Java 17.

Run:

```bash
sudo update-alternatives --config java
```

Choose the Java 17 option.

Then run:

```bash
sudo update-alternatives --config javac
```

Choose the Java 17 option.

Why this matters:

- Spring Boot 3.x requires modern Java
- mismatched Java versions can cause build or runtime issues
- Maven may compile with a different JDK if not configured correctly

---

# 11. Generate Spring Boot Project from CLI

Use Spring Initializr directly from command line.

Run:

```bash
curl -L "https://start.spring.io/starter.zip?type=maven-project&language=java&bootVersion=3.5.0&baseDir=auth0-demo&groupId=com.example&artifactId=auth0-demo&name=auth0-demo&javaVersion=17&dependencies=web,security,oauth2-client,oauth2-resource-server" -o auth0-demo.zip
```

Then verify and extract:

```bash
ls -lh auth0-demo.zip
unzip auth0-demo.zip
cd auth0-demo
```

You should see files like:

```text
pom.xml
src/
mvnw
```

Why generate from Spring Initializr?

Because it creates a clean, production-standard Spring Boot project with the correct dependency structure.

---

# 12. Create application.yml

Remove the default properties file and create YAML configuration.

Run:

```bash
rm -f src/main/resources/application.properties
vi src/main/resources/application.yml
```

Add:

```yaml
server:
  port: 8080

spring:
  security:
    oauth2:
      client:
        registration:
          auth0:
            client-id: PHTS7cNTRdPzmSN38ovt9vPWBprxdi2s
            client-secret: ydVKuJ_N1UXxX8y1BZC5ZTcAjMJTbgBKToxDSZBng-bBZGAbonq9CGvSKuFnnoxY
            scope:
              - openid
              - profile
              - email
            authorization-grant-type: authorization_code
            redirect-uri: "{baseUrl}/login/oauth2/code/{registrationId}"

        provider:
          auth0:
            issuer-uri: https://dev-rxbdreshrv2aylj4.us.auth0.com/

      resourceserver:
        jwt:
          issuer-uri: https://dev-rxbdreshrv2aylj4.us.auth0.com/
```

## Important

Replace the values with your actual values if they change:

- client-id
- client-secret
- issuer-uri

## Why this configuration is needed

### OAuth2 Client section
Used for browser login.

### registration.auth0
Defines the client application details.

### provider.auth0
Defines the Auth0 issuer metadata.

### resourceserver.jwt
Enables JWT validation for secure APIs.

---

# 13. Recommended Secure Version Using Environment Variables

For real projects, do not hardcode secrets.

Use:

```yaml
server:
  port: 8080

spring:
  security:
    oauth2:
      client:
        registration:
          auth0:
            client-id: ${AUTH0_CLIENT_ID}
            client-secret: ${AUTH0_CLIENT_SECRET}
            scope:
              - openid
              - profile
              - email
            authorization-grant-type: authorization_code
            redirect-uri: "{baseUrl}/login/oauth2/code/{registrationId}"

        provider:
          auth0:
            issuer-uri: https://${AUTH0_DOMAIN}/

      resourceserver:
        jwt:
          issuer-uri: https://${AUTH0_DOMAIN}/
```

Set variables:

```bash
export AUTH0_CLIENT_ID=your_client_id
export AUTH0_CLIENT_SECRET=your_client_secret
export AUTH0_DOMAIN=dev-rxbdreshrv2aylj4.us.auth0.com
```

This is safer than storing secrets in source code.

---

# 14. Create SecurityConfig.java

Run:

```bash
mkdir -p src/main/java/com/example/auth0_demo/config
nano src/main/java/com/example/auth0_demo/config/SecurityConfig.java
```

Add:

```java
package com.example.auth0_demo.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {

        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/api/public/**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2Login(Customizer.withDefaults())
            .oauth2ResourceServer(oauth2 ->
                oauth2.jwt(Customizer.withDefaults()));

        return http.build();
    }
}
```

## Why we need this class

This class defines the security behavior of the application.

### `requestMatchers("/", "/api/public/**").permitAll()`
Allows public access to:

- home page
- public APIs

### `anyRequest().authenticated()`
All other endpoints require authentication.

### `oauth2Login(Customizer.withDefaults())`
Enables browser login using Auth0.

### `oauth2ResourceServer(oauth2 -> oauth2.jwt(...))`
Enables JWT validation for Bearer token-based API access.

---

# 15. Create Controller

Run:

```bash
mkdir -p src/main/java/com/example/auth0_demo/controller
nano src/main/java/com/example/auth0_demo/controller/TestController.java
```

Add:

```java
package com.example.auth0_demo.controller;

import org.springframework.security.core.Authentication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TestController {

    @GetMapping("/")
    public String home() {
        return "Application Running";
    }

    @GetMapping("/api/public/test")
    public String publicApi() {
        return "Public API Working";
    }

    @GetMapping("/api/secure/test")
    public String secureApi(Authentication auth) {
        return "Secure API Working for " + auth.getName();
    }
}
```

## Why this controller is useful

It gives us three simple test endpoints:

### `/`
Confirms the application is running.

### `/api/public/test`
Confirms public API access works without authentication.

### `/api/secure/test`
Confirms secure API access works only after authentication.

---

# 16. Build the Application

Run:

```bash
mvn clean package
```

If successful, Maven builds the application JAR.

Why build first?

Because it validates:

- dependencies
- Java version
- package structure
- code compilation
- Spring Boot configuration loading

---

# 17. Run the Application

Start the application:

```bash
mvn spring-boot:run
```

Expected logs should include something similar to:

```text
Started Auth0DemoApplication
Tomcat started on port(s): 8080
```

This confirms the application started successfully.

---

# 18. Test Locally on EC2

## Home endpoint

Run:

```bash
curl http://localhost:8080
```

Expected:

```text
Application Running
```

## Public API

Run:

```bash
curl http://localhost:8080/api/public/test
```

Expected:

```text
Public API Working
```

## Secure API without token

Run:

```bash
curl http://localhost:8080/api/secure/test
```

Expected:

```text
401 Unauthorized
```

Why 401?

Because the endpoint is protected and no authentication was provided.

---

# 19. Enable Public Access from Internet

To test from browser externally, allow port 8080 in the EC2 security group.

Why?

Because without opening port 8080, the application is reachable only inside the server.

After allowing the port, you can access:

```text
http://23.20.250.154:8080/
```

<img width="1322" height="375" alt="Screenshot 2026-06-07 at 12 20 48 AM" src="https://github.com/user-attachments/assets/e3d1c937-7e0c-4bf7-ac9e-7f5afa4b9ec7" />

---

# 20. Configure Auth0 for EC2 Public Access

In Auth0 application settings, add the EC2 public URLs.

## Allowed Callback URLs

```text
http://YOUR_EC2_PUBLIC_IP:8080/login/oauth2/code/auth0
http://23.20.250.154:8080/login/oauth2/code/auth0
```

## Allowed Logout URLs

```text
http://YOUR_EC2_PUBLIC_IP:8080/
http://23.20.250.154:8080/
```

## Allowed Web Origins

```text
http://YOUR_EC2_PUBLIC_IP:8080
http://23.20.250.154:8080
```

Why is this required?

Because Auth0 only redirects to URLs explicitly allowed in the application settings.

If these URLs are missing, login will fail with callback or redirect errors.

---

# 21. Start Login Flow from Browser

Open:

```text
http://YOUR_EC2_PUBLIC_IP:8080/oauth2/authorization/auth0
```

or:

```text
http://23.20.250.154:8080/oauth2/authorization/auth0
```

<img width="496" height="750" alt="Screenshot 2026-06-07 at 12 32 39 AM" src="https://github.com/user-attachments/assets/9a5d0bce-85c0-4d04-9e21-90c12e9d23f7" />



What happens:

1. Spring Boot detects OAuth2 client configuration
2. Spring Boot redirects browser to Auth0
3. Auth0 login page loads
4. User logs in using Auth0 credentials or social login such as Google

This confirms:

- OAuth2 client configuration is correct
- issuer URI is correct
- redirect URI is correct
- Auth0 application is correct
- callback URL is correct
- EC2 networking is correct

---

# 22. What Happened in the Successful Working Test

During the successful test:

1. Browser opened:
   ```text
   http://23.20.250.154:8080/oauth2/authorization/auth0
   ```

2. Spring Boot redirected to Auth0

3. Auth0 presented login options

4. User selected Google signup/login

5. Google authenticated the user

6. Google returned identity to Auth0

7. Auth0 returned authorization code to Spring Boot

8. Spring Boot exchanged the code for tokens

9. Spring Security created authenticated session

10. Secure endpoint returned:

```text
Secure API Working for google-oauth2|114409360300857680925
```

This proves the integration worked end to end.

---

# 23. Why the Application Later Showed "Application Running"

After successful login, when the URL:

```text
http://23.20.250.154:8080/oauth2/authorization/auth0
```

was opened again, the application showed:

```text
Application Running
```

<img width="403" height="121" alt="Screenshot 2026-06-07 at 12 48 47 AM" src="https://github.com/user-attachments/assets/6b55c08a-c1ba-421f-87e4-627933a7fc93" />


Why?

Because the user session was already authenticated.

Spring Security did not need to force login again.

The application already knew the user was logged in.

So instead of repeating the login flow, it continued with the authenticated session and returned the application response.

---

# 24. Full End-to-End Flow Explanation

Here is the exact working flow.

```text
Browser
-> Spring Boot

Spring Boot
-> redirects to Auth0

Auth0
-> redirects to Google login

Google
-> authenticates user

Google
-> returns identity to Auth0

Auth0
-> returns authorization code to Spring Boot

Spring Boot
-> exchanges code for tokens

Spring Security
-> creates authenticated session
```

This is the complete OIDC authorization code flow with social login federation.

---

# 25. Why Google Appeared in the Flow

Google appeared because Auth0 was configured to allow Google as an identity provider.

That means:

- Auth0 is still the main identity platform for the application
- Google is only one upstream login provider
- Spring Boot does not integrate directly with Google
- Spring Boot integrates only with Auth0
- Auth0 handles the federation to Google

This is an important architecture concept.

---

# 26. What Spring Boot Actually Receives

Spring Boot does not receive the user's password.

Spring Boot receives:

- authorization code
- tokens after code exchange
- authenticated principal
- user identity claims

This is more secure than handling passwords directly.

---

# 27. Why Secure API Returned `google-oauth2|...`

The secure endpoint returned:

```text
Secure API Working for google-oauth2|114409360300857680925
```

Why?

Because `auth.getName()` returned the authenticated principal identifier from Auth0.

Since the user logged in through Google federation, the principal identifier reflected the Google-linked identity.

This is expected behavior.

---

# 28. How to Force Login Again

If you want to see the login page again, use one of these methods:

## Option 1: Open browser in incognito/private mode

This starts a fresh browser session.

## Option 2: Clear cookies/session

This removes the existing authenticated session.

## Option 3: Implement logout

Currently logout is not configured in a complete way.

---

# 29. Important Logout Clarification

Current behavior only clears the Spring session if local logout is added.

It does **not** fully log out from:

- Auth0 SSO session
- Google SSO session

Why?

Because enterprise logout is more advanced and usually requires **OIDC RP-Initiated Logout**.

So if the user is still logged into Auth0 or Google, login may appear automatic on the next attempt.

---

# 30. Optional Basic Logout Example

You can add local logout like this:

```java
http
    .logout(logout -> logout
        .logoutSuccessUrl("/"));
```

This only logs out from the Spring Boot session.

It does not guarantee full Auth0 or Google logout.

---

# 31. Why We Need Both OAuth2 Client and Resource Server

This is one of the most important concepts.

## OAuth2 Client
Needed for browser login.

Without it, Spring Boot cannot redirect users to Auth0 and complete OIDC login.

## Resource Server
Needed for JWT validation on APIs.

Without it, Spring Boot cannot validate Bearer tokens sent to secure endpoints.

So in this application:

- browser login uses OAuth2 client
- API protection uses resource server

---

# 32. Security Design of This Application

Current security design is:

- `/` is public
- `/api/public/**` is public
- all other endpoints require authentication

This is a clean and common starting point for enterprise applications.

---

# 33. Common Errors and Their Meaning

## 401 Unauthorized

Meaning:

- token missing
- token invalid
- token expired
- user not authenticated

## 403 Forbidden

Meaning:

- authentication succeeded
- but authorization failed

## Invalid issuer

Meaning:

- wrong Auth0 domain or issuer URI

## Redirect mismatch / callback error

Meaning:

- callback URL in Auth0 does not exactly match Spring Boot callback URL

## CORS error

Meaning:

- frontend origin not allowed
- usually relevant when frontend is separate from backend

---

# 34. Production Recommendations

For production use:

- use HTTPS instead of HTTP
- never hardcode client secret in files
- store secrets in AWS Secrets Manager, Vault, or environment variables
- use custom domain if required
- restrict callback URLs carefully
- configure proper logout
- configure roles/scopes/permissions
- add monitoring and audit logging
- avoid exposing port 8080 directly without reverse proxy
- place application behind ALB, Nginx, or API gateway if needed

---

# 35. Interview Explanation

If asked in interview, explain like this:

> We integrated Spring Boot with Auth0 using Spring Security OAuth2.  
> Auth0 acts as the identity provider and handles authentication, social login, and token issuance.  
> In Spring Boot, we configured OAuth2 client for browser-based OIDC login and resource server for JWT validation.  
> Public endpoints were allowed without authentication, while secure endpoints required a valid authenticated session or token.  
> During testing, Auth0 federated login to Google, and after successful authentication Spring Security created the user session successfully.

---

# 36. Final Working Commands Summary

## Install dependencies on AWS server

```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
sudo apt install maven -y
sudo apt install unzip -y
java -version
mvn -version
sudo update-alternatives --config java
sudo update-alternatives --config javac
```

## Generate project

```bash
curl -L "https://start.spring.io/starter.zip?type=maven-project&language=java&bootVersion=3.5.0&baseDir=auth0-demo&groupId=com.example&artifactId=auth0-demo&name=auth0-demo&javaVersion=17&dependencies=web,security,oauth2-client,oauth2-resource-server" -o auth0-demo.zip
unzip auth0-demo.zip
cd auth0-demo
```

## Build and run

```bash
mvn clean package
mvn spring-boot:run
```

## Local tests

```bash
curl http://localhost:8080
curl http://localhost:8080/api/public/test
curl http://localhost:8080/api/secure/test
```

## Browser login

```text
http://localhost:8080/oauth2/authorization/auth0
```

## EC2 browser login

```text
http://23.20.250.154:8080/oauth2/authorization/auth0
```

---

# 37. Final Code Summary

## application.yml

```yaml
server:
  port: 8080

spring:
  security:
    oauth2:
      client:
        registration:
          auth0:
            client-id: PHTS7cNTRdPzmSN38ovt9vPWBprxdi2s
            client-secret: ydVKuJ_N1UXxX8y1BZC5ZTcAjMJTbgBKToxDSZBng-bBZGAbonq9CGvSKuFnnoxY
            scope:
              - openid
              - profile
              - email
            authorization-grant-type: authorization_code
            redirect-uri: "{baseUrl}/login/oauth2/code/{registrationId}"

        provider:
          auth0:
            issuer-uri: https://dev-rxbdreshrv2aylj4.us.auth0.com/

      resourceserver:
        jwt:
          issuer-uri: https://dev-rxbdreshrv2aylj4.us.auth0.com/
```

## SecurityConfig.java

```java
package com.example.auth0_demo.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {

        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/api/public/**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2Login(Customizer.withDefaults())
            .oauth2ResourceServer(oauth2 ->
                oauth2.jwt(Customizer.withDefaults()));

        return http.build();
    }
}
```

## TestController.java

```java
package com.example.auth0_demo.controller;

import org.springframework.security.core.Authentication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TestController {

    @GetMapping("/")
    public String home() {
        return "Application Running";
    }

    @GetMapping("/api/public/test")
    public String publicApi() {
        return "Public API Working";
    }

    @GetMapping("/api/secure/test")
    public String secureApi(Authentication auth) {
        return "Secure API Working for " + auth.getName();
    }
}
```

---

# 38. Conclusion

This setup is now working successfully end to end.

You have successfully implemented:

- Auth0 tenant usage
- Auth0 regular web application
- Spring Boot OAuth2 client integration
- Spring Boot resource server integration
- browser-based login
- Google social login through Auth0
- secure API protection
- EC2 deployment and public testing

This is a strong real-world authentication integration pattern and a solid foundation for enterprise-grade application security.
