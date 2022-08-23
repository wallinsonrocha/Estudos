# Resource

A partir daqui, será feito a configuração do Resource server, que enviará alguns resources caso o usuário esteja autorizado através de um token. Esse passo só poderá ser feito caso o anterior tenha sido feito completamente.

Dentro do pacote config criaremos a seguinte classe:

**ResourceServerConfig**

```
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter{

    @Autowired
    private JwtTokenStore tokenStore;

        // Para organizar a autorização de resource para usuário
        // Aqui estão as rotas que todos podem acessar
        private static final String[] PUBLIC = { "/oauth/token" };

        // Aqui estão as rotas que somente o operador e o admin podem acessar.
        // O "**" serve para indicar "todos".
        private static final String[] OPERATOR_OR_ADMIN = { "/products/**", "/categories/**" };

        // Resources para o Admin
        private static final String[] ADMIN = { "/users/**" };



    // Iremos implementar 2 Override. O Configure HttpSecurity e o ResourceServerSecurityConfigurer.
    // Não podermos esquecer que ele está no Source (Alt+Shift+S no STS).

    @Override
    // (ResourceServerSecurityConfigurer resources){
        resources.tokenStore(tokenStore);
    }

    @Override
    // (HttpSecurity http) {
        // Configuração de quem pode acessar tal resource.
        // É através dele que fazemos essas configurações. Baste procurar o método requerido e implementar.
        http.authorizedRequests()
        .antMatchers(PUBLIC).permitAll() // Autorizações
        .antMatchers(HttpMethod.GET, OPERATOR_OR_ADMIN).permitAll() // Apenas os métodos get nessas rostas
        .antMatchers(OPERATOR_OR_ADMIN).hasAnyRoles("OPERATOR", "ADMIN")
        .antMatchers(ADMIN).hasAnyRoles("ADMIN")
        .anyRequest().authenticated(); // Exige que, qualquer outra rota, o usuário esteja logado.
    }
}
```

## Configuração para o H2 Console

Para que seja permitido entrar no H2 console como faziamos antes, devemos configurar sua permissão. Para isso, nas rotas públicas devemos colocar o seu resource lá.

```
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter{
    ...
    @Autowired
    private Environment env;

    private static final String[] PUBLIC = { "/oauth/token", "/h2-console/**" };
    ...

    @Override
    // (HttpSecurity http) {

        // Configuração para liberar o H2
        if(Arrays.asList(env.getActiveProfiles()).contains("test")){
            http.headers().frameOptions().disable();
        }

        http.authorizedRequests()
        .antMatchers(PUBLIC).permitAll() 
        .antMatchers(HttpMethod.GET, OPERATOR_OR_ADMIN).permitAll()
        .antMatchers(OPERATOR_OR_ADMIN).hasAnyRoles("OPERATOR", "ADMIN")
        .antMatchers(ADMIN).hasAnyRoles("ADMIN")
        .anyRequest().authenticated();
    }
}
```

## Para o postman na aba Tests

```
if(responseCode.code >= 200 && responseCode.code >= 300){
    var json = JSON.parse(responseBody);
    postman.setEnvironmentVariable('token', json.access_token);
}
```