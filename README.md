# Desafio-Dados-e-Seguran-a

Para integrar a API Spring Boot com o banco de dados PostgreSQL usando Spring Data JPA e implementar a segurança com Spring Security, siga os seguintes passos:

1. Configuração do Projeto
Criar um novo projeto Spring Boot:

1 - Usar o Spring Initializr para gerar um projeto Spring Boot com as dependências "Spring Web", "Spring Data JPA", "Spring Security" e "PostgreSQL Driver".

2 - Configuração do "application.properties":

spring.datasource.url=jdbc:postgresql://localhost:5432/nome_do_banco
spring.datasource.username=seu_usuario
spring.datasource.password=sua_senha
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect

2. Configuração do Spring Data JPA

1 - Entidades:
Crie as classes de entidade para representar as tabelas do banco de dados. Por exemplo, para usuários e pacotes de viagem:

@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;
    private String password;
    private String role;
    
    // Getters e Setters
}

@Entity
public class TravelPackage {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String destination;
    private String description;
    private double price;
    
    // Getters e Setters
}

2 - Repositórios:
Crie interfaces de repositório para acessar os dados das entidades.

public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
}

public interface TravelPackageRepository extends JpaRepository<TravelPackage, Long> {
}


3. Configuração do Spring Security

1 - Serviço de Usuário:
Crie um serviço para carregar detalhes do usuário a partir do banco de dados.

@Service
public class CustomUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found"));

        return new org.springframework.security.core.userdetails.User(user.getUsername(), user.getPassword(),
                AuthorityUtils.createAuthorityList(user.getRole()));
    }
}

2 - Configuração de Segurança:
Configure o Spring Security para autenticação e autorização.

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private CustomUserDetailsService userDetailsService;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService).passwordEncoder(passwordEncoder());
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
            .authorizeRequests()
            .antMatchers("/admin/**").hasRole("ADMIN")
            .antMatchers("/user/**").hasAnyRole("USER", "ADMIN")
            .anyRequest().authenticated()
            .and()
            .httpBasic();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}

4. Controladores

1 - Controladores de API:
Implemente os controladores para gerenciar os endpoints da API.

@RestController
@RequestMapping("/api/packages")
public class TravelPackageController {

    @Autowired
    private TravelPackageRepository travelPackageRepository;

    @GetMapping
    public List<TravelPackage> getAllPackages() {
        return travelPackageRepository.findAll();
    }

    @PostMapping
    @PreAuthorize("hasRole('ADMIN')")
    public TravelPackage createPackage(@RequestBody TravelPackage travelPackage) {
        return travelPackageRepository.save(travelPackage);
    }

    @GetMapping("/{id}")
    public TravelPackage getPackageById(@PathVariable Long id) {
        return travelPackageRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Package not found"));
    }
}

5. Execução do Projeto
1 - Executar a Aplicação:
Execute o projeto Spring Boot e certifique-se de que a aplicação está conectada corretamente ao banco de dados PostgreSQL e que a segurança está funcionando conforme esperado.

Conclusão:
Seguindo esses passos, você conseguirá integrar a API Spring Boot com o PostgreSQL usando Spring Data JPA e implementar a segurança com Spring Security, garantindo autenticação e autorização baseadas em perfis de usuário.
