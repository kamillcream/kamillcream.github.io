# 🔐 Spring Security 코드 Develop

---

## 2025.01.30 
### SecurityConfig
```
@Configuration
@EnableSecurity
@RequiredArgsConstructor
public class SecurityConfig{
    private final AuthenticationConfiguration authenticationConfiguration;
    private final JwtUtil;
    
    @Bean
    public AuthenticationManager authentication(AuthenticationConfiguration configuration) throws Exception {
        AuthenticationManager manager = configuration.getAuthenticationManager();
        return manager;
    }    
    
    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder(){
        return new BCryptPasswordEncoder();
    }
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.csrf(csrf -> csrf.disable());
        http.cors(Customizer.withDefaults());
        
        http.formLogin((auth) -> auth.disable());
        http.httpBasic((auth) -> auth.disable());
        
        http
              .authorizeHttpRequests(requests -> requests
                    .requestMatchers().permitAll()
                .anyRequest().authenticated()
              );
        http.
            addFilterAt(new LoginFilter(authenticationManager(authenticationConfiguration), jwtUtil),
            UsernamePasswordAuthenticationFilter.class
            )
            .addFilterBefore(new JwtFilter(jwtUtil, LoginFilter.class);
        http.sessionManagement(
            session -> session.sessionCreationFolicy(SeesionCreationPolicy.STATELESS));
            
        
        return http.build;
    }
}      
```
### CustomUserDetails
```
@RequiredArgsConstructor
public class CustomUserDetails implements UserDetails{
    private final Entity entity;
    
    //getter
    
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        List<GrantedAuthority> authorities = new ArrayList();
        authorities.add(new SimpleGrantedAuthority(""));
        return authorities;
    }
    @Override
    public String getPassword() {
        return customer.getPassword();
    }

    @Override
    public String getUsername() {
        return customer.getName();
    }
}    
```
### CustomUserDetailsService
```
@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService{
    private final UserRepository repository;
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUserName(username);
        if (user != null) {
            return new CustomUserDetails(user);
        }
        return null;
    }
}
```