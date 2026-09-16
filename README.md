# Name: MADESWARAN M
# Reg no: 212223040106

# EXP05-Setting-Up-Spring-Security-in-a-Spring-Boot-Project
## AIM:
To write a program for setting up Spring Security in a Spring Boot project to secure endpoints with basic authentication and role-based access control.

## ALGORITHM:
Create a Spring Boot Project with the following dependencies:

Spring Web

Spring Security

Spring Boot DevTools (optional)

Add Spring Security dependency in pom.xml (if not using Spring Initializr).

Create a configuration class extending WebSecurityConfigurerAdapter (or using SecurityFilterChain for newer Spring versions).

Define an in-memory user with username, password, and roles using UserDetailsService.

Secure your REST endpoints using annotations or in the security config class.

Run and test the app using a browser or Postman:

Secure endpoints will prompt for username and password.

## PROGRAM CODE:
### pom.xml (Dependencies)
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
</dependencies>
```
## Ex5Application.java
```
package com.example.ex5;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Ex5Application {

	public static void main(String[] args) {
		SpringApplication.run(Ex5Application.class, args);
	}

}


````
### SecurityConfig.java (Spring Boot 3.x / Spring Security 6+)
```
package com.example.ex5;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http)
            throws Exception {

        http.csrf(csrf -> csrf.disable());

        http.authorizeHttpRequests(auth -> auth
                .requestMatchers("/login").permitAll()
                .requestMatchers("/verify").permitAll()
                .anyRequest().permitAll()
        );

        return http.build();
    }
}

```

## JwtService.java:
```
package com.example.ex5;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import io.jsonwebtoken.security.Keys;
import org.springframework.stereotype.Service;

import java.nio.charset.StandardCharsets;
import java.security.Key;
import java.util.Date;

@Service
public class JwtService {

    // In real projects, store this in application.properties
    private final String SECRET_KEY =
            "my-super-secret-key-my-super-secret-key-123456";

    // 1 hour
    private final long EXPIRATION_TIME = 1000 * 60 * 60;

    private Key getSigningKey() {
        return Keys.hmacShaKeyFor(
                SECRET_KEY.getBytes(StandardCharsets.UTF_8)
        );
    }

    // =========================
    // Generate JWT
    // =========================
    public String generateToken(String username) {

        return Jwts.builder()
                .setSubject(username)
                .setIssuedAt(new Date())
                .setExpiration(
                        new Date(System.currentTimeMillis()
                                + EXPIRATION_TIME)
                )
                .signWith(getSigningKey(), SignatureAlgorithm.HS256)
                .compact();
    }

    // =========================
    // Extract username
    // =========================
    public String extractUsername(String token) {

        Claims claims = Jwts.parser()
                .verifyWith((javax.crypto.SecretKey) getSigningKey())
                .build()
                .parseSignedClaims(token)
                .getPayload();

        return claims.getSubject();
    }

    // =========================
    // Validate JWT
    // =========================
    public boolean isTokenValid(String token, String username) {

        try {

            String tokenUsername = extractUsername(token);

            return tokenUsername.equals(username)
                    && !isTokenExpired(token);

        } catch (Exception e) {
            return false;
        }
    }

    // =========================
    // Check expiration
    // =========================
    private boolean isTokenExpired(String token) {

        Claims claims = Jwts.parser()
                .verifyWith((javax.crypto.SecretKey) getSigningKey())
                .build()
                .parseSignedClaims(token)
                .getPayload();

        return claims.getExpiration().before(new Date());
    }
}


```

## AuthController:

```
package com.example.ex5;

import org.springframework.web.bind.annotation.*;

@RestController
public class AuthController {

    private final JwtService jwtService;

    public AuthController(JwtService jwtService) {
        this.jwtService = jwtService;
    }

    // =========================
    // LOGIN - Generate JWT
    // =========================
    @PostMapping("/login")
    public String login(
            @RequestParam String username,
            @RequestParam String password) {

        // Simple username/password check
        if (username.equals("admin")
                && password.equals("1234")) {

            // Generate JWT
            return jwtService.generateToken(username);
        }

        return "Invalid login";
    }

    // =========================
    // VERIFY JWT
    // =========================
    @GetMapping("/verify")
    public String verifyToken(
            @RequestHeader(
                    value = "Authorization",
                    required = false
            ) String authorizationHeader) {

        // Check Authorization header
        if (authorizationHeader == null) {
            return "Authorization header is missing";
        }

        // Check Bearer token
        if (!authorizationHeader.startsWith("Bearer ")) {
            return "Invalid Authorization header";
        }

        // Remove "Bearer "
        String token = authorizationHeader.substring(7);

        try {

            // Extract username from JWT
            String username = jwtService.extractUsername(token);

            // Validate JWT
            if (jwtService.isTokenValid(token, username)) {
                return "Token is valid. User: " + username;
            }

            return "Token is invalid";

        } catch (Exception e) {

            // JWT expired / invalid / malformed
            return "JWT Error: " + e.getMessage();
        }
    }
}


```

## Output:

<img width="1917" height="927" alt="image" src="https://github.com/user-attachments/assets/16106875-5101-446b-9cd1-8e6e08ecafdc" />


<img width="1917" height="937" alt="image" src="https://github.com/user-attachments/assets/3e95f347-98a7-4f15-9c5f-a6e31dfa2a32" />

## Result:
Thus the program is completed successfully.



