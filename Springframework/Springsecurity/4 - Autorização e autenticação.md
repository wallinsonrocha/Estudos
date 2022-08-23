# Autorização e autenticação

A partir daqui, será feito a configuração do Authorization server, que enviar as credencias através de um token para o usuário.

Dependências: **spring security test e spring security oauth2 autoconfigure**

Para poder fazer a comunicação com o servidor e conseguir um token de acesso.

## Interfaces

### UserDetails

É uma interface que será implementada na entidate do Usuário. Quando implementarmos, ele irá criar os métodos que faltam.

```
public class User implements UserDetails, Serializable{
    ...

    @Override
    public String getUsername(){
        return email;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities(){
        return roles.stream().map(role -> new SimpleGrantedAuthority(role.getAuthority())).collect(Collectors.toList());
    }

    // O role aqui é o papel de cada usuário de acordo com o exemplo visto na video aula.
    // O role é uma entidade criada.

    ...
}
```

De restante dos métodos, a fins de testes, podemos retornar true para todos.

### UserDetailsService

É uma interface que será implementada no UserService.

```
public class UserService implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // Já criado no nosso repositório
        User user = repository.findByEmail(username);
        if(user == null){
            logger.error("User not found: " + username);
            throw new UsernameNotFoundException("Email not found");
        }

        logger.info("User found: " + username);
        return user;
    }
}
```

**Implementação de um Logger**

```
private static Logger logger = LoggerFactory.getLogger(UserService.class);
```

## Classes

É o mesmo SecurityConfig que criamos, porém, podemos renomear para WebSecurityConfig se quisermos. Iremos fazer algumas alterações.

```
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter{

    @Autowired
    private BCryptPasswordEncoder passwordEncoder;

    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    public void configure(WebSecurity web) throws Exception{
        web.ignoring().antMatchers("/actuator/**");
    }

    // Precisamos criar uma sobrecarga para o mesmo método configure. Nas configurações de geração, como o construtor e getters e setters, tem o de override. Iremos escolher o configure(AuthenticationManagerBuilder). Nele iremos usar as configurações como BCrypt e o UserDetailsSerice.
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception{
        auth.userDetailsService(userDetailsService).passwordEncoder(passwordEncoder);
    }

    // Chamaremos mais um Override
    @Override
    @Bean
    protected AuthenticationManager authenticationManager() throws Exception {
        return super.authenticationManager();
    }
}
```

---

# Autorização e autenticação - OAuth2

## Configuração para externalizar alguns valores

Nós usaremos o arquivo application.properties para criar as variáveis. Adicionaremos as seguintes linhas:

```
spring.profiles.active=${APP_PROFILE:test}

spring.jpa.open-in-view=false

security.oauth2.client.client-id=${CLIENT_ID:dscatalog}
security.oauth2.client.client-secret=${CLIENT_SECRET:dscatalog123}

jwt.secret=${JWT_SECRET:MY-JWT-SECRET}
jwt.duration=${JWT_DURATION:86400}
```

## Beans para Token JWT

Pode ser colocado no AppConfig. Eles serão capazes de acessar o token.

```
@Value("${jwt.secret}")
    private String jwtSecret;

@Bean
public JwtAccessTokenConverter accessTokenConverter() {
	JwtAccessTokenConverter tokenConverter = new JwtAccessTokenConverter();
	tokenConverter.setSigningKey(jwtSecret);
	return tokenConverter;
}

@Bean
public JwtTokenStore tokenStore() {
	return new JwtTokenStore(accessTokenConverter());
}
```

## Classe

Nós iremos criar ela no pacote de configuração.

```
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter{

    @Value("${security.oauth2.client.client-id}")
    private String clientId;

    @Value("${security.oauth2.client.client-secret}")
    private String clientSecret;

    @Value("${jwt.duration}")
    private Integer jwtDuration;

    @Autowired
    private BCryptPasswordEncoder passwordEncoder;

    @Autowired
    private JwtAccessTokenConverter accessTokenConverter;

    @Autowired
    private JwtTokenStore tokenStore;

    // O bean criado no SecurityConfig
    @Autowired
    private AuthenticationManager authenticationManager;

    // Sobscreveremos os 3 métodos Override configure
    @Override
    protected void configure(AuthorizationServerSecurityConfigurer security) throws Exception{
        security.tokenKeyAccess("permitAll()").checkTokenAccess("isAuthenticated()");
    }

    @Override
    protected void configure(ClientDetailsServiceConfigurer clients) throws Exception{
        clients.inMemory()
        .withClient(clientId)
        .secret(passwordEncoder.encode(clientSecret))
        .scopes("read", "write")
        .authorizedGrantTypes("password")
        .accessTokenValiditySeconds(jwtDuration);
    }

    @Override
    protected void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception{
        endpoints.authenticationManager(authenticationManager)
        .tokenStore(tokenStore)
        .accessTokenConverter(accessTokenConverter);
    }
}
```

---

# Autorização e autenticação - Acesso na url

Quando vamos fazer o acesso, o endpoint será "/oauth/token". Além disso, no postman, iremos colocar o método post, pois faremos um loggin. Ao invés de fazermos pela aba do "Params", iremos na aba "Authorization". E o tipo de autorização que escolheremos é o basic.

![Direcionamento segundo a instrução acima](../../Fotos/Captura%20de%20tela%202022-08-06%20101622.png)

Logo após isso, iremos colocar o Usuário e a senha de acordo com o que colocamos no nosso banco. No caso do código acima foram "usuario" e "senha123".

Após isso, precisaremos colocar as credenciais do usuário. Iremos na aba Body e escolheremos o formato x-www-form-urlencoded.

![Direcionamento segundo a instrução acima](../../Fotos/credenciais%20do%20usuario.png)

Logo após, iremos colocar as chaves e os valores para as credenciais. O usuário e a senha devem estar registradas no banco de dados.

![Direcionamento segundo a instrução acima](../../Fotos/chaves%20e%20valores.png)

---

# Adicionando informações no Token

Caso a nossa aplicação precise de algumas informações, como o nome, id, email ou outras coisas nescessárias de alguém que foi autenticado, podemos fazer isso da seguinte forma:

Criaremos um pacote chamado **Components** para por os nossos componentes.

**Classe JwtTokenEnhancer**

```
@Component
public class JwtTokenEnhancer implements TokenEnhancer{
    
    // Esse repositório irá auxiliar o método abaixo
    @Autowired
    private UserRepository userRepository;

    @Override
    // Método OAuth2AccessToken exigido após implantar o TokenEnhancer (accessToken, authentication) {
        User user = userRepository.findByEmail(authentication.getName());

        // Adicionando objetos no token. Eles estarão dentro do token
        Map<String, Object> map = new HasMap<>();
        map.put("userFirstName", user.getFisrtName());
        map.put("userId", user.getId());

        // Consolidando os objetos adicionados.
        // Esse Default é bem mais específico do que o OAuth2AccessToken. Ele possui o método para adicionar.
        DefaultOAuth2AccessToken token = (DefaultOAuth2AccessToken) accessToken;
        token.setAdditionalInformation(map);

        return accessToken; // Ou token.
    }
}
```

**Injetando na classe** no AuthorizationServerConfig.

```
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter{
    ...
    @Autowired
    private JwtTokenEnhancer tokenEnhancer;
    ...

     @Override
    protected void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception{

        TokenEnhancerChain chain = new TokenEnhancerChain();
        // Esse objeto espera uma lista
        chain.setTokenEnhancer(Arrays.asList(accessTokenConverter, tokenEnhancer));

        endpoints.authenticationManager(authenticationManager)
        .tokenStore(tokenStore)
        .accessTokenConverter(accessTokenConverter)
        //Adicional
        .tokenEnhancer(chain);
}
```
