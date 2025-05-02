---
title: "Understanding Spring Security and OAuth 2.0 with a project"
seoTitle: "Implement OAuth 2.0 Authorization Code Flow with Google Sign-In and Cu"
seoDescription: "Learn how to secure your Java-based applications using Spring Security and implement authentication with OAuth 2.0 in a practical project"
datePublished: Fri Jan 10 2025 23:58:33 GMT+0000 (Coordinated Universal Time)
cuid: cm5rf4ina000109lg7woigoxe
slug: understanding-spring-security-and-oauth-20-with-a-project
tags: authentication, java, security, springboot, oauth2, springsecurity-authenticationfilter-springboot-javasecurity-customfilters-websecurity-apiauthentication

---

In this article we will explore Spring security and will build a authentication system with OAuth 2.0.

### Spring Security is a powerful, highly customizable framework for implementing robust authentication and access control mechanisms in Java-based applications. It is a core component of the Spring ecosystem, widely used to secure web applications, REST APIs, and other backend services. With Spring Security, you gain a solid foundation to build and enforce secure practices in your application.

---

## How Spring Security Works

Before diving into how Spring Security operates, it's crucial to understand the **request-handling lifecycle** in a Java-based web server. Spring Security seamlessly integrates into this lifecycle to secure incoming requests.

---

### **Request-Handling Lifecycle with Spring Security**

The lifecycle of handling an HTTP request in a Spring-based application with Spring Security involves several stages, each playing a critical role in processing, validating, and securing the request.

---

### 1\. **Client Request**

The lifecycle begins when a client (e.g., browser, mobile app, or API tool like Postman) sends an HTTP request to the server.

**Example:**  
`GET /api/admin/dashboard HTTP/1.1`

---

### 2\. **Servlet Container**

The servlet container (e.g., Tomcat) receives the request and delegates it to the `DispatcherServlet`, the front controller in a Spring application. This is where the application’s processing pipeline starts.

---

### 3\. **Spring Security Filter Chain**

Before the `DispatcherServlet` processes the request, **Spring Security's Filter Chain** intercepts it. The filter chain is a sequence of filters, each responsible for handling specific security tasks. These filters ensure the request meets authentication and authorization requirements before it reaches the application logic.

#### **Key Filters in the Chain**:

1. **Authentication Filters**:  
    These filters verify if the request contains valid credentials, such as a username/password, a JWT, or session cookies.
    
2. **Authorization Filters**:  
    After authentication, these filters ensure the authenticated user has the necessary roles or permissions to access the requested resource.
    
3. **Other Filters**:
    
    * **CsrfFilter**: Validates CSRF tokens to prevent Cross-Site Request Forgery attacks.
        
    * **CorsFilter**: Manages Cross-Origin Resource Sharing (CORS) rules for secure API access from different domains.
        
    * **ExceptionTranslationFilter**: Handles security-related exceptions (e.g., invalid credentials) and sends appropriate responses to the client.
        

---

### 4\. **Security Context**

If authentication is successful, Spring Security creates an `Authentication` object and stores it in the **SecurityContext**. This object, often stored in a thread-local storage, is accessible throughout the request lifecycle.

#### **The Authentication Object**:

* **Principal**: Represents the authenticated user (e.g., username).
    
* **Credentials**: Includes authentication details like JWT tokens or passwords.
    
* **Authorities**: Contains roles and permissions assigned to the user.
    

#### **Example Flow in the Filter Chain**:

* A request passes through the authentication filters.
    
* If the credentials are valid, the `Authentication` object is created and added to the `SecurityContext`.
    
* If the credentials are invalid, the `ExceptionTranslationFilter` sends a `401 Unauthorized` response to the client.
    

---

### 5\. **DispatcherServlet**

Once the request successfully passes through the Spring Security Filter Chain, the `DispatcherServlet` takes over:

1. **Handler Mapping**:  
    It maps the incoming request to the appropriate controller method based on the URL and HTTP method.
    
2. **Controller Invocation**:  
    The mapped controller processes the request and returns the appropriate response, often with help from other Spring components like services and repositories.
    

### **How Spring Security Fits Into the Lifecycle**

Spring Security integrates itself into this lifecycle through its filters, intercepting requests at the earliest stage. By the time a request reaches the application logic, it has already been authenticated and authorized, ensuring only legitimate traffic gets processed by the core application.

---

Spring Security’s design ensures that authentication, authorization, and other security measures are handled declaratively, giving developers the flexibility to customize or extend its behavior as needed. It not only enforces best practices but also simplifies the implementation of complex security requirements in modern applications.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1736449644430/75fba25f-7339-4003-9dab-59503993ef0c.png align="center")

### Spring Security Components: Beyond the Filter Chain

Having explored the **Filter Chain** in Spring Security, let’s delve into some other key components that play a pivotal role in the authentication and authorization process.

## AuthenticationManager

`AuthenticationManager` is an interface that defines a single method , `authenticate(Authentication authentication)` , which is used to verify the credentials of a user and determine if they are valid. You can think of `AuthenticationManager` as a coordinator where you can register multiple providers, and based on the request type, it will deliver an authentication request to the correct provider.

## AuthenticationProvider

An `AuthenticationProvider` is an interface that defines a contract for authenticating users based on their credentials. It represents a specific authentication mechanism, such as username/password, OAuth, or LDAP. Multiple `AuthenticationProvider` implementations can coexist, allowing the application to support various authentication strategies.

#### **Core Concepts**:

1. **Authentication Object**:  
    The `AuthenticationProvider` processes an `Authentication` object, which encapsulates the user’s credentials (e.g., username and password).
    
2. **authenticate Method**:  
    Each `AuthenticationProvider` implements the `authenticate(Authentication authentication)` method, where the actual authentication logic resides. This method:
    
    * Validates the user’s credentials.
        
    * Returns an authenticated `Authentication` object upon success.
        
    * Throws an `AuthenticationException` if authentication fails.
        
3. **supports Method**:  
    The `supports(Class<?> authentication)` method indicates whether the `AuthenticationProvider` can handle the given type of `Authentication`. This allows Spring Security to determine the correct provider to handle specific authentication requests.
    

#### **Example**:

* A database-backed `AuthenticationProvider` validates usernames and passwords.
    
* An OAuth-based `AuthenticationProvider` validates tokens issued by an external identity provider.
    

## UserDetailsService

`UserDetailsService` is described as a core interface that loads user-specific data in the Spring documentation It contains a single method `loadUserByUsername` which accepts username as parameter and returns the ==User== identity object . Basically we create and implementation class of `UserDetailsService` in which we override the `loadUserByUsername` method.

```java
package com.oauth.backend.services;

import com.oauth.backend.entities.User;
import com.oauth.backend.repositories.UserRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Component;


@Component
public class CustomUserDetailsService implements UserDetailsService {
    private final UserRepository userRepository;

    public CustomUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {

        User user = userRepository.findByUsername(username);
        if(user==null){
            throw new UsernameNotFoundException(username);
        }
        return new UserDetailsImpl(user);
    }
    public UserDetails loadUserByEmail(String email) throws UsernameNotFoundException {
        User user = userRepository.findByEmail(email);
        if(user==null){
            throw new UsernameNotFoundException(email);
        }
        return new UserDetailsImpl(user);
    }
}
```

Now how all these three works together is AuthenticationManager will ask AuthenticationProvider to carry on the authentication according to the type of Provider specified and the UserDetailsService implementation will help the AuthenticationProvider in proving the userdetails .

#### **Now before moving to the configuration and all stuff here's a concise flow of Spring Security for JWT-based authentication:**

### 1\. **User Request**

* The user sends a request to the authenticated endpoint with their credentials (username and password) or a JWT token (in the header) and the request is passed to the Authentication Filter
    
* `AuthenticationFilter` (e.g., `UsernamePasswordAuthenticationFilter`):
    
    * Handles user authentication based on credentials submitted (typically in the form of a username and password). This is where the `UsernamePasswordAuthenticationFilter` comes into play.
        
    * It listens for the request , extracts the username and password, and passes them to the `AuthenticationManager`.
        
    * But we are not passing the username and password , we are giving only the token hence there should be a filter before this AuthenticationFilter which will tell the Authentication Process that the user is authenticated and no need to check for Username and password and this is done by creating a JWTFilter
        

### 2\. JWTFilter

This custom filter extends OncePerRequestFilter and is placed before the UsernamePasswordAuthenticationFilter , and what it do is it extracts the token from request and validates it.

If the token is valid it creates a `UsernamePasswordAuthenticationToken` and sets that token into the Security Context which tells the spring security that the request is authenticated and when this request passes to the `UsernamePasswordAuthenticationFilter` it just passes as it has the `UsernamePasswordAuthenticationToken`

```java

@Component
public class JWTFilter extends OncePerRequestFilter {

    private final JWTService jwtService;
    private final UserDetailsService userDetailsService;
    public JWTFilter(JWTService jwtService,UserDetailsService userDetailsService) {
        this.jwtService = jwtService;
        this.userDetailsService = userDetailsService;
    }
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        final String authHeader = request.getHeader("Authorization");

        if(authHeader == null || !authHeader.startsWith("Bearer")) {
            filterChain.doFilter(request,response);
            return;
        }

        final String jwt = authHeader.substring(7);
        final String userName = jwtService.extractUserName(jwt);

        Authentication authentication
                = SecurityContextHolder.getContext().getAuthentication();

        if(userName !=null  && authentication == null) {
            //Authenticate
            UserDetails userDetails
                    = userDetailsService.loadUserByUsername(userName);

            if(jwtService.isTokenValid(jwt,userDetails)) {
                UsernamePasswordAuthenticationToken authenticationToken
                        = new UsernamePasswordAuthenticationToken(
                        userDetails,
                        null,
                        userDetails.getAuthorities()
                );

                SecurityContextHolder.getContext()
                        .setAuthentication(authenticationToken);
            }
        }

        filterChain.doFilter(request,response);
    }



}
```

this UsernamePasswordAuthenticationToken is generated with the help of the AuthenticationManager and AuthenticationProvider if we have passed username and password instead of the token after authenticating the username and password with the help of out UserDetails class.

### 3\. **AuthenticationManager**

* **AuthenticationManager**: This receives the authentication request and delegates it to the appropriate `AuthenticationProvider` which we configure.
    

```java
@Bean  
public AuthenticationManager authenticationManager(AuthenticationConfiguration config) throws Exception{  
return config.getAuthenticationManager();  
}
```

### 4\. **AuthenticationProvider**

* **UserDetailsService**: The `AuthenticationProvider` uses `UserDetailsService` to load user details based on the username. And we provide this with a implementation of `UserDetailsService`
    
* **Credential Validation**: It compares the provided password with the one stored in the user details (typically using a `PasswordEncoder`).
    

```java
@Bean  
public AuthenticationProvider authenticationProvider(){  
DaoAuthenticationProvider authenticationProvider = new DaoAuthenticationProvider();  
authenticationProvider.setUserDetailsService(userDetailsServiceImpl);  
authenticationProvider.setPasswordEncoder(passwordEncoder);  
return authenticationProvider;  
}
```

Now all these different filters and beans are needed to be configured so that Spring security knows what to do hence we create a configuration class where we specify all the configuration.

```java
package com.oauth.backend.config;


import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@EnableWebSecurity
@Configuration
public class SecurityConfig {
    private final UserDetailsService userDetailsService;
    private final JWTFilter jwtFilter;
    public SecurityConfig(UserDetailsService userDetailsService, JWTFilter jwtFilter) {
        this.userDetailsService = userDetailsService;
        this.jwtFilter = jwtFilter;
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {

        http.cors(Customizer.withDefaults()).csrf(csrf->csrf.disable()).authorizeHttpRequests((authorize) -> authorize
                        .requestMatchers(HttpMethod.OPTIONS,"/**").permitAll()
                        .requestMatchers("/api/v1/auth/register","/api/v1/auth/login","/api/v1/oauth/google").permitAll()
                        .anyRequest().authenticated()
                ).sessionManagement(session -> session
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS) // Stateless sessions
                )
                .authenticationProvider(authenticationProvider())
                .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    @Bean
    public AuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration configuration) throws Exception {
        return configuration.getAuthenticationManager();
    }
    @Bean
    public BCryptPasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }

}
```

till now we have understand and configured our authentication with the help of spring security now it’s time to test it.

We will create a simple app with two controllers AuthController (handles login and register) and ProductController (dummy protected controller)

```java
//AuthController.java
package com.oauth.backend.controller;

import com.oauth.backend.dto.LoginResponse;
import com.oauth.backend.dto.UserDTO;
import com.oauth.backend.entities.User;
import com.oauth.backend.services.AuthService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;


@RestController
@CrossOrigin("*")
@RequestMapping("/api/v1/auth")
public class AuthController {
    private final AuthService authService;

    public AuthController(AuthService authService) {
        this.authService = authService;
    }

    @PostMapping("/login")
    public ResponseEntity<LoginResponse> login(@RequestBody UserDTO userDTO){
        return ResponseEntity.ok(new LoginResponse(authService.login(userDTO.username(), userDTO.password())));
    }
    @PostMapping("/register")
    public String register(@RequestBody UserDTO userDTO){
        return authService.register(userDTO);
    }
    @GetMapping("/me")
    public ResponseEntity<?> getCurrentUser(){
        return ResponseEntity.ok(authService.me());
    }
}
```

```java
// AuthService.java
package com.oauth.backend.services;

import com.oauth.backend.dto.UserDTO;
import com.oauth.backend.entities.User;
import com.oauth.backend.repositories.UserRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.security.authentication.BadCredentialsException;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.stereotype.Service;

@Service

public class AuthService {
    private final UserRepository userRepository;
    private final BCryptPasswordEncoder bCryptPasswordEncoder;
    private final JWTService jwtService;

    public AuthService(UserRepository userRepository, BCryptPasswordEncoder bCryptPasswordEncoder, JWTService jwtService) {
        this.userRepository = userRepository;
        this.bCryptPasswordEncoder = bCryptPasswordEncoder;
        this.jwtService = jwtService;
    }

    public String register(UserDTO userDTO) {
        System.out.println(userDTO);
    User user = new User();
    user.setUsername(userDTO.username());
    user.setPassword(bCryptPasswordEncoder.encode(userDTO.password()));
    user.setEmail(userDTO.email());
    userRepository.save(user);
//        user.setPassword(bCryptPasswordEncoder.encode(user.getPassword()));
        return "User registered successfully";
    }
    public String login(String username, String password) {
        User user = userRepository.findByUsername(username);
        if (user == null) {
           throw new UsernameNotFoundException("User not found");
        }
        if (!bCryptPasswordEncoder.matches(password, user.getPassword())) {
            throw new BadCredentialsException("Bad credentials");
        }
        return jwtService.generateToken(username);
    }

    public UserDTO me(){
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        UserDetailsImpl userDetails = (UserDetailsImpl) authentication.getPrincipal();

        return new UserDTO(userDetails.getUsername(), "",userDetails.getEmail() );
    }
}
```

```java
//User.java (Entity)

package com.oauth.backend.entities;

import jakarta.persistence.*;
import lombok.Data;

@Entity
@Table(name = "_user")

public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    private String id;
    private String username;
    private String password;
    private String email;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```

```java
//UserDetailsImpl.java

package com.oauth.backend.services;

import com.oauth.backend.entities.User;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import java.util.Collection;
import java.util.Collections;
import java.util.List;

public class UserDetailsImpl implements UserDetails {
    private User user;

    UserDetailsImpl(User user) {
        this.user = user;
    }
    @Override
    public boolean isEnabled() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public String getUsername() {
        return user.getUsername();
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }
    public String getEmail(){
        return user.getEmail();
    }
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return Collections.singleton(new SimpleGrantedAuthority("USER"));
    }
}
```

```java
//JWTService

package com.oauth.backend.services;

import com.oauth.backend.entities.User;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.io.Decoders;
import io.jsonwebtoken.security.Keys;
import lombok.RequiredArgsConstructor;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Service;

import javax.crypto.SecretKey;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.function.Function;

@Service

public class JWTService {
    private String secretKey = null;

    public String generateToken(String username) {
        Map<String, Object> claims
                = new HashMap<>();
        return Jwts
                .builder()
                .claims()
                .add(claims)
                .subject(username)
                .issuer("DCB")
                .issuedAt(new Date(System.currentTimeMillis()))
                .expiration(new Date(System.currentTimeMillis()+ 60*10*1000))
                .and()
                .signWith(generateKey())
                .compact();
    }

    private SecretKey generateKey() {
        byte[] decode
                = Decoders.BASE64.decode(getSecretKey());

        return Keys.hmacShaKeyFor(decode);
    }


    public String getSecretKey() {
        return secretKey = "RqxPOuVfHoBA8Uq40MhJvfY6qEHOOWWvg6N9W9vt23s=";
    }

    public String extractUserName(String token) {
        return extractClaims(token, Claims::getSubject);
    }

    private <T> T extractClaims(String token, Function<Claims,T> claimResolver) {
        Claims claims = extractClaims(token);
        return claimResolver.apply(claims);
    }

    private Claims extractClaims(String token) {
        return Jwts
                .parser()
                .verifyWith(generateKey())
                .build()
                .parseSignedClaims(token)
                .getPayload();
    }

    public boolean isTokenValid(String token, UserDetails userDetails) {
        final String userName = extractUserName(token);
        return (userName.equals(userDetails.getUsername()) && !isTokenExpired(token));
    }

    private boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }

    private Date extractExpiration(String token) {
        return extractClaims(token, Claims::getExpiration);
    }
}
```

```java
//Product Controller.java
package com.oauth.backend.controller;

import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

import java.util.ArrayList;
import java.util.List;


@RestController
@CrossOrigin("*")
@RequestMapping("/api/v1/product")
public class ProductController {
    private List<Product> products;
    ProductController(){
        products = new ArrayList<>();
    }
    private record Product(Integer productId,
                           String productName,
                           double price) {}

    @GetMapping("/view")
    public List<Product> getProducts() {
        return products;
    }
    @PostMapping("/add")
    public String addProduct(@RequestBody Product product) {
        products.add(product);
        return "Product added";
    }
}
```

till now we have implemented a registration , login and verification but what if i also want to add Login With Google/Github fuctionality then we can do it with the help of OAuth2.0

### OAuth 2.0

OAuth 2.0 is a protocol made for authorization through which enables users to grant third party applications access to resources stored on other platforms (e.g Google Drive , Github) without sharing the credentials of those platforms.

It is mostly used in enabling social logins like “Login with google” , “Login with github ” .

Platforms like Google , Facebook , Github provides Authorization server which implements OAuth 2.0 protocol for this social sign in or authorizing access .

## Key Concepts of OAuth 2.0

* ### Resource Owner
    
* ### Client
    
* ### **Authorization Server**
    
* ### Resource Server
    
* ### Access Token
    
* ### Scopes
    
* ### Grants
    

Now we will look into each concept one by one

## Resource Owner

Resource owner is the user who wants to authorize the third party application (Your Application).

## Client

It is your (third party) application which wants to access the data or resource from the resource server.

## Resource Server

It is the server where the user’s data is stored and is to be accessed by the third party application.

## Authorization Server

The server that authenticates the resource owner and issues access tokens to the client (e.g., Google Accounts).

## Access Token

A credential issued by the authorization server to the client, allowing it to access the resource server on behalf of the user. It is generally short lived that is expires very soon so a refresh token is also provided in order to refresh this access token so that user doesn’t need to authorize again.

## Scopes

Specific permissions granted by the user, defining what the client can and cannot do with the user's data. For example for authorization we only need user info like profile , name etc. but for file access different scope is required.

## Grants

It refers to the methods by which Client application can obtain the access token from the Authorization Server. A grant defines the process and conditions under which the client application is authorized to access a resource owner's protected data.

It is secure as we do not need to expose our client secret and other credentials to the browser

There are two mostly used Grant types provided by OAuth 2.0

1. ### Authorization Code Grant
    
    It is the most used type of grant/method , most secure and is for server-side applications
    
    In this a authorization code is given by the client to the backend and the backend gives the access token to the client.
    
    **Process:**
    
    1. The client redirects the user to the authorization server.
        
    2. The user logs in and consents.
        
    3. The authorization server issues an **authorization code**.
        
    4. The client exchanges the authorization code with the backend for an **access token**.
        
    
2. ### Implicit Grant
    
    Used by single-page apps (SPAs) or applications without a backend. In this the access token is directly generated and issued in the browser itself.
    
    **Process:**
    
    1. The client redirects the user to the authorization server.
        
    2. The user logs in and consents.
        
    3. The authorization server directly issues an **access token**.
        
    

We will implement both separately for complete understanding , but first we will implement the Authorization Code Grant for that we will need

1. Authorization Server
    
    It can be either of a platform (like google , github) or you can create your own also using KeyCloak or can also build your own adhering to OAuth 2.0 standards ( we may do this in next blog 😌)
    
2. Spring Boot Application
    
    This will be our main backend application/service which will handle all the operations like code exchange, verification , saving user details and assigning JWT tokens
    
3. React Application (Frontend)
    
    This will be our client which will redirect the user to Authorization Server for authorization.
    

So in our implementation what we will be doing is the frontend(web/app) will redirect our user to google login with redirect uri to our backend endpoint which will take control further we will talki about it later and along with the redirect\_url we also be passing the client id of our app all these will be sent in the query parameters.

No when the user will successfully login in the google the authrization server(google’s) will redirect our request to the backend enpoint and there what we will be doing is exchanging Authorization code with authorization server to get access token and refresh token and then we can handle the auth as we want and finally we will send a response back to our frontend which will have a cookie and redirect to our dashboard or may be a protected page.

Now we will look into the code , but make sure that you add the url of your backend endpoint in authorized redirect urls in Google console dashboard for OAuth client.

```java
package com.oauth.backend.controller;

import com.oauth.backend.entities.User;
import com.oauth.backend.repositories.UserRepository;
import com.oauth.backend.services.CustomUserDetailsService;
import com.oauth.backend.services.JWTService;
import com.oauth.backend.services.UserDetailsImpl;
import jakarta.servlet.http.Cookie;
import jakarta.servlet.http.HttpServletResponse;
import org.antlr.v4.runtime.misc.MultiMap;
import org.springframework.http.*;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.util.LinkedMultiValueMap;
import org.springframework.util.MultiValueMap;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.client.RestTemplate;
import org.springframework.web.servlet.view.RedirectView;

import java.util.HashMap;
import java.util.Map;
import java.util.UUID;

@RestController
@CrossOrigin("*")
@RequestMapping("/api/v1/oauth")
public class OAuthController {
    private final CustomUserDetailsService customUserDetailsService;
    private final BCryptPasswordEncoder bCryptPasswordEncoder;
    private final UserRepository userRepository;
    private final JWTService jWTService;

    public OAuthController(CustomUserDetailsService customUserDetailsService, BCryptPasswordEncoder bCryptPasswordEncoder, UserRepository userRepository, JWTService jWTService) {
        this.customUserDetailsService = customUserDetailsService;
        this.bCryptPasswordEncoder = bCryptPasswordEncoder;
        this.userRepository = userRepository;
        this.jWTService = jWTService;
    }

    @GetMapping("/google")
    public void google(@RequestParam String code, HttpServletResponse response2) {
        RestTemplate restTemplate = new RestTemplate();
//        restTemplate
        try{
            String tokenEndpoint = "https://oauth2.googleapis.com/token";
            String clientId = "ANJALI_HI_SHEETAL_HAI";
            String clienSecret = "SHEETAL_HI_MUNNI_HAI";
            MultiValueMap<String,String> map = new LinkedMultiValueMap<>();
            map.add("client_id", clientId);
            map.add("client_secret", clienSecret);
            map.add("code", code);
            map.add("grant_type", "authorization_code");
            map.add("redirect_uri", "http://localhost:8080/api/v1/oauth/google");
            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
            HttpEntity<MultiValueMap<String,String>> request = new HttpEntity<>(map,headers);
            ResponseEntity<Map> tokenResponse = restTemplate.postForEntity(tokenEndpoint, request, Map.class);
            String token = tokenResponse.getBody().get("id_token").toString();
            String url = "https://oauth2.googleapis.com/tokeninfo?id_token=" + token;
            ResponseEntity<Map> response = restTemplate.getForEntity(url,Map.class);
            User user = null;
        if(response.getStatusCode()== HttpStatus.OK){
            UserDetails userDetails =null;
            String email = response.getBody().get("email").toString();;
            try {
                userDetails = customUserDetailsService.loadUserByEmail(email);
            }
            catch(Exception e){
                user=new User();
                user.setEmail(email);
                user.setPassword(bCryptPasswordEncoder.encode(UUID.randomUUID().toString()));
                user.setUsername(email.split("@")[0]);
                userRepository.save(user);
                userDetails=customUserDetailsService.loadUserByUsername(user.getUsername());
            }



           String jwt_token = jWTService.generateToken(userDetails.getUsername());
            Cookie cookie = new Cookie("accessToken",jwt_token);
            cookie.setPath("/");
            cookie.setMaxAge(3600);
            cookie.setSecure(true);
            response2.addCookie(cookie);
            response2.sendRedirect("http://localhost:5172/products");
        }
        }
        catch (Exception e){
            e.printStackTrace();
    throw new RuntimeException(e.getMessage());
        }

    }
}
```

and that’s it this will work fine and for testing you can made a simple frontend application which will nothing but having a context and yout know login and registration functions.

Thanks for reading till this long , and if you have any suggestion , please drop it in the comments