# Spring Security

## Agenda

### Spring Security Basics 

- Create `HomeController.java` and `AdminController.java`
- Try to access `http://localhost:8080/`

```java
@RestController
public class HomeController {

    @GetMapping("")
    public String home() {
        return "Hello 👋🏻 O'Reilly!";
    }

}
```

```java
@RestController
public class AdminController {

    @GetMapping("/admin")
    public String admin() {
        return "Hello, Admin!";
    }

}
```

Spring Security is secure by default. Spring Boot provides a default configuration where the username is `user` and the password is randomly generated and printed to the console. This is to get you up and running quickly but you will need to override this configuration with your own.

### Spring Security Configuration

In prior versions of Spring Security you might have written code that looked like this: 

```java
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
@Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/actuator/**").permitAll()
                .anyRequest().authenticated()
                .and()
            .oauth2Login()
                .and()
            .oauth2ResourceServer()
                .jwt();
    }
    
}
```

Spring Security 6 introduces a component based model for configuration: 

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {}
```

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    return http
            .authorizeHttpRequests( auth -> {
                auth.requestMatchers("/").permitAll();
                auth.requestMatchers("/admin/**").authenticated();
                auth.anyRequest().authenticated();
            })
            .formLogin(Customizer.withDefaults())
            .build();
}
```

Show an example of mutliple security filter chains

```java
@Bean
@Order(1)
SecurityFilterChain apiSecurityFilterChain(HttpSecurity http) throws Exception {
    return http
            .securityMatcher("/api/**")
            .authorizeHttpRequests(auth -> {
                auth.anyRequest().authenticated();
            })
            .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .httpBasic(withDefaults())
            .build();
}

@Bean
@Order(2)
SecurityFilterChain h2ConsoleSecurityFilterChain(HttpSecurity http) throws Exception {
    return http
            .securityMatcher(AntPathRequestMatcher.antMatcher("/h2-console/**"))
            .authorizeHttpRequests( auth -> {
                auth.requestMatchers(AntPathRequestMatcher.antMatcher("/h2-console/**")).permitAll();
            })
            .csrf(csrf -> csrf.ignoringRequestMatchers(AntPathRequestMatcher.antMatcher("/h2-console/**")))
            .headers(headers -> headers.frameOptions().disable())
            .build();
}

@Bean
@Order(3)
SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    return http
            .authorizeHttpRequests(auth -> {
                    auth.requestMatchers("/").permitAll();
                    auth.requestMatchers("/error").permitAll();
                    auth.anyRequest().authenticated();
                }
            )
            .formLogin(withDefaults())
            .build();
}
```

#### Defining Users

Password Storage - https://docs.spring.io/spring-security/reference/features/authentication/password-storage.html 

```java
@Bean
public UserDetailsService userDetailsService() {
    return new InMemoryUserDetailsManager(
            User.withUsername("dvega")
                    .password("password")
                    .build()
    );
}
```

```java
@Bean
public UserDetailsService userDetailsService() {
    return new InMemoryUserDetailsManager(
            User.withUsername("dvega")
                    .password(passwordEncoder().encode("password"))
                    .build()
    );
}

@Bean
PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

```java
@Bean
public UserDetailsService userDetailsService() {
    return new InMemoryUserDetailsManager(
            User.withUsername("dvega")
                    .password("{noop}password")
                    .build()
    );
}
```

### User Roles

```java
@Bean
public UserDetailsService userDetailsService() {
    UserDetails admin = User.withUsername("dvega")
            .password("{noop}password")
            .roles("ADMIN")
            .build();
    UserDetails user = User.withUsername("user")
            .password("{noop}password")
            .roles("USER")
            .build();
    return new InMemoryUserDetailsManager(admin,user);
}
```

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    return http
            .authorizeHttpRequests( auth -> {
                auth.requestMatchers("/").permitAll();
                auth.requestMatchers("/admin/**").hasRole("ADMIN");
            })
            .formLogin(Customizer.withDefaults())
            .build();
}
```

You can enable method security by adding the `@EnableMethodSecurity` annotation to your `SecurityConfig` class.

https://docs.spring.io/spring-security/reference/servlet/authorization/method-security.html 

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity(securedEnabled = true)
public class SecurityConfig {
```

You can use the `@Secured` annotation to secure a method.

```java
@RestController
public class AdminController {

    @Secured("ROLE_ADMIN")
    @GetMapping("/admin")
    public String admin() {
        return "Hello, Admin!";
    }

}
```

or the `@PreAuthorize` annotation.

### Authentication 

Username and Password
https://docs.spring.io/spring-security/reference/servlet/authentication/index.html 

- form login
- basic authentication
- digest authentication

Change to basic authentication 

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    return http
            .authorizeHttpRequests( auth -> {
                auth.requestMatchers("/").permitAll();
                auth.requestMatchers("/admin/**").hasRole("ADMIN");
                auth.anyRequest().authenticated();
            })
            .httpBasic(Customizer.withDefaults())
            .build();
}
```

#### Username and Password with Database

https://github.com/danvega/jdbc-users

#### Social Login

### JWT

### CORS

## Resources

- [Spring Security Reference Documentation](https://docs.spring.io/spring-security/reference/index.html)
- [Migrating to Spring Security 6.0](https://docs.spring.io/spring-security/reference/migration/index.html)
- [Spring Security Architecture](https://docs.spring.io/spring-security/reference/servlet/architecture.html)