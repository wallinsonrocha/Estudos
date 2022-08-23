# Springsecurity

O srping security trabalha com a segurança da nossa API. Através dela nós podemos criptografar senhas, dar autorização a alguns usuários e assim deixá-lo mais segura.

---

##  SecurityConfig - Autorização para usar a API.

Quando nós criamos alguma aplicação com springsecurity, normalmente ele gera um código que deve ser implementado na requisição de uma aplicação para que o se usos seja altorizado. Mas, podemos configurar para que o código rode sem precisar dessa autorização. Para isso, devemos criar uma **classe de configuração** no pacote **config**.

```
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter{
    @Override
    public void configure(WebSecurity web) throws Exception{
        web.ignoring().antMatchers("/**");
    }
}
```
