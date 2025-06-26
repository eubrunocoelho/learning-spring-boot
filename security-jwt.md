# Spring Security com *`JWT`* para *`API REST`*

O *Spring Security* é a estrutura de fato para proteger aplicativos *Spring*, mas pode ser conplicado de configurar.

Este artigo do *Spring Security* destaca uma solução `JWT` eficiente.

**Aviso: O *Spring Security 5+* lançou o suporte para `OAuth JWT`. Recomenda-se usar a versão mais recente do *OAuth* para suporte a `JWT` em vez de usar segurança ou filtros personalizados.**

O *Spring* é considerado um *framework confiável* no ecossistema *Java* e é amplamente utilizado. Não é mais válido se referir ao *Spring* como um framework, pois é mais um termo abrangente que abrange vários frameworks. Um desses frameworks é o *Spring Security*, um framework de autenticação e autorização poderoso e personalizável. Ele é considerado o padrão de fato para proteger aplicativos baseados em *Spring*. Portanto, se você deseja implemnetar uma *solução de token `JWT Spring`*, faz sentido baseá-lo no *Spring Security*.
Apesar de sua popularidade, devo admitir que, quando se trata de aplicativos de página única, o *Spring* não é simples e direto de configurar. Suspeito que o motivo seja que ele começou mais como um *framework orientado a aplicativos MVC*, onde a renderização de páginas da web ocorre no lado do servidor e a *comunicação é baseada em sessão*.

Se o `back-end` for baseado em *Java* e *Spring*, faz sentido usar o *Spring Security* com `JWT` para autenticação/autorização e configurá-lo para comunicação sem estado. Embora existam muitos artigos explicando como isso é feito, para mim, ainda foi frustante configurá-lo pela primeira vez, e tive que ler e somar informações de várias fontes. É por isso que decedi escrever este tutorial do *Spring Security*, onde tentarei resumir e abordar todos os detalhes sutis e pontos fracos que você pode encontrar durante o processo de configuração.

## Definindo Terminologia

Antes de mergulhar nos detalhes técnicos, quero definir explicitamente a terminologia usada no contexto do *Spring Security* apenas para ter certeza de que todos falamos a mesma língua.

Estes são os termos que precisamos abordar:

- **Authentication *(Autenticação)*** refere-se ao processo de verificação da identidade de um usuário, com base nas credenciais fornecidas. Um exemplo comum é inserir um nome de usuário e uma senha ao fazer login em um site. Você pode pensar nisso como uma resposta à pergunta *"QUem é você?"*.
- **Authorization *(Autorização)*** refere-se ao processo de determinar se um usuário tem permissão adequada para executar uma ação específica ou ler dados específicos, supondo que o usuário seja autenticado com sucesso. Você pode pensar nisso como uma resposta à pergunta *"Um usuário pode fazer/ler isto?"*.
- **Principle *(Princípio)*** refere-se ao usuário atualmente autenticado.
- **Granted authority *(Autoridade concedida)*** refere-se à permissão do usuário autenticado.
- **Role *(Função)*** refere-se a um grupo de permissões do usuário autenticado.

## Criando um aplicativo Spring Básico

Antes de passar para a configuração do framework *Spring Security*, vamos criar uma aplicação web *Spring* báscia. Para isso, podemos usar um [Spring Initializr](https://start.spring.io/) e gerar um projeto modelo. Para uma aplicação web simples, apenas uma dependência do framework web *Spring* é suficiente:

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

Depois de criar o projeto, podemos adicionar um `controller` *REST* simples a ele da seguinte maneira:

```java
@RestController @RequestMapping("hello")
public class HelloRestController {
    @GetMapping("user")
    public String helloUser() {
        return "Hello User";
    }

    @GetMapping("admin")
    public String helloAdmin() {
        return "Hello Admin";
    }
}
```

Depois disso, se construirmos e executarmos o projeto, poderemos acessar as seguintes `URLs` no navegador:

- `http://localhost:8080/hello/user` retornará `Hello User`.
- `http://localhost:8080/hello/admin` retornará `Hello Admin`.

Agora, podemos adicionar o framework *Spring Security* ao nosso projeto, e podemos fazer isso adicionando a seguinte dependência ao nosso arquivo `pom.xml`:

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
</dependencies>
```

Adicionar routras dependências do framework *Spring* normalmente não tem efeito imediato em um aplicativo até que fornecamos a configuração correspondente, mas o *Spring Security* é diferente, pois tem *efeito imediato*, o que geralmente confunde novos usuários. Após adicioná-lo, se reconstruirmos e executarmos o projeto e, em seguida, tentarmos acessar uma das `URLs` mencionadas em vez de visualizar o resultado, seremos redirecionados para `http://localhost:8080/login`. Este é o comportamento padrão, pois o framework *Spring Security* exige autenticação pronta para uso em todas `URLs`.

Para passar a autenticação, podemos usar o nome de usuário padrão `user` e encontrar uma senha gerada automaticamente em nosso console:

```
Using generated security password: 1fc15145-dfee-4bec-a009-e32ca21c77ce
```

Lembre-se de que a senha muda sempre que executamos o aplicativo novamente. Se quisermos alterar esse comportamento e tornar a senha estática, podemos adicionar a seguinte configuração ao nosso arquivo `application.properties`:

```
spring.security.user.password=Test12345_
```

Agora, se inserirmos as credenciais no formulário de login, seremos redirecionados para nossa `URL` e veremos o resultado correto. Observe que o processo de autenticação padrão é baseado em sessão e, se quisermos sair, podemos acessar a seguinte `URL`: `http://localhost:8080/logout`.

Esse comportamento pronto para uso pode ser útil para *aplicações web MVC clássicas*, nas quais temos *autenticação baseada em sessão*, mas, no caso de aplicações de página única, geralmente não é útil, pois, na maioria dos casos de uso, temos renderização do lado do cliente e *autenticação sem estado baseada em `JWT`*. Nesse caso, teremos que personalizar bastante o framework *Spring Security*, o que faremos no restante do artigo.

Com exemplo, implementaremos um aplicativo web de livrária clássico e criaremos um `back-end` que fornecerá `APIs CRUD` para criar autores e livros, além de `APIs` para gerenciamento e autenticação de usuários.

## Visão Geral da Arquitetura do *Spring Security*

Antes de começarmos a personalizar a configuração, vamos primeiro discutir como a autenticação do *Spring Security* funciona nos bastidores.

*O driagrama a seguir apresenta o fluxo e mostra como as solicitações de autenticação são processadas:*

### Arquitetura do *Spring Security*

![Spring Security Architecture](./.github/assets/images/spring-security-architecture.png)

Agora, vamos dividir este diagrama em componentes e discutir cada um deles separadamente.

### Filtros do *Spring Security*

Ao adicionar o framework *Spring Security* ao seu aplicativo, ele registra automaticamente uma *cadeia de filtros* *(filters chain)* que intercepta todas as aplicações recebidas. Essa cadeia consiste em vários filtros, e cada um deles lida com um caso de uso específico.

Por exemplo:

- Verifique se a `URL` solicitada é acessível publicamente, com base na configuração.
- Em caso de *autenticação baseada em sessão*, verifique se o usuário já está autenticado na sessão atual.
- Verifique se o usuário está autorizado a executar a ação solicitada e assim por diante.

*Um detalhe importante que quero mencionar é que os filtros do **Spring Security** são registrados com a ordem mais baixa e são os primeiros a serem invocados. Em alguns casos de uso, se você quiser colocar seu filtro personalizado na frente deles, precisará adicionar preenchimento à ordem. Isso pode ser feito com a seguinte configuração:*

```
spring.security.filter.order=10
```

Depois de adicionarmos essa configuração ao nosso arquivo `application.properties`, teremos espaço para 10 fultros personalizados na frente dos filtros do *Spring Security*.

### `AuthenticationManager`

Você pode pensar em `AuthenticationManager` *(Gerenciador de Autenticação)* como um coordenador onde é possível registrar vários provedores e, com base no tipo de solicitação, ele entregará uma solicitação de autenticação ao provedor correto.

### `AuthenticationProvider`

`AuthenticationProvider` *(Provedor de Autenticação)* processa tipos específicos de autenticação. Sua interface expõe apenas duas funções:

- `authenticate` realiza autenticação com a solicitação.
- `supports` verifica se este provedor suporta o tipo de autenticação indicado.

Uma implementação importante da interfacce que estamos usando em nosso projeto de exemplo é `DaoAuthenticationProvider`, que recupera detalhes do usuário de um `UserDetailsfService`.

### `UserDetailsService`

`UserDetailsService` é descrito como uma interface central que carrega dados específicos do usuário na documentação do *Spring*.

Na maioria dos casos de uso, os provedores de autenticação extraem informações de identidade do usuário com base em *credencias de um banco de dados* e, em seguida, realizam a validação. Como esse caso de uso é tão comum, os desenvolvedores do *Spring* decidiram *extraí-lo* como uma *interface* separada, que expõe a função única:

- `loadUserByUsername` aceita nome de usuário como parâmetro e retorna o objeto de identidade do usuário.

## Autenticação Usando `JWT` com *Spring Security*

Depois de discutir os aspectos internos do framework *Spring Security*, vamos configurá-lo para *autenticação sem estado* com um *token `JWT`*.

Para personalizar o *Spring Security* para uso com `JWT`, precisamos de uma classe de configuração anotada com a anotação `@EnableWebSecurity` em nosso `classpath`. Além disso, para simplificar o processo de personalização, o framework expõe uma classe `WebSecurityConfigurerAdapter`. Estenderemos esse adaptador e substituiremos ambas as funções para:

1. Configurar o gerenciador de autenticação com o provedor correto.
2. Configurar a segurança da web *(URLs públicas, URLs privadas, autorização, etc.)*.

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        // TODO configure authentication manager
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // TODO configure web security
    }
}
```

Em nosso aplicativo de exemplo, armazenamos identidades de usuários em um banco de dados `MongoDB`, na coleção `users`. Essas identidades são mapeadas pela entidade `User` e suas operações `CRUD` são definidas pelo repositório `UserRepo` do *Spring Data*.

Agora, quando aceitarmos a solicitação de autenticação, precisamos recuperar a identidade correta do banco de dados usando as credencias fornecidas e, em seguida, verificá-la. Para isso, precisamos da implementação da interface `UserDetailsService`, que é definida da seguinte forma:

```java
public interface UserDetailsService {
    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

Aqui, podemos ver que é necessário retornar o objeto que implementa a inteface a `UserDetails`, e nossa entidade `User` o implementa *(para detalhes de implemnetação, consulte o repositório do projeto de exemplo)*. Considerando que ele expõe apenas o protótipo de função única, podemos tratá-lo como uma interface funcional e fornecer a implementação com uma expressão `lambda`.

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    private final UserRepo userRepo;

    public SecurityCOnfig(UserRepo userRepo) {
        this.userRepo = userRepo;
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(username -> userRepo
            .findByUsername(username)
            .orElseThrow(
                () -> new UsernameNotFoundException(
                    format("User: %s, not found", username)
                )
            ));
    }

    // Details omitted for brevity
}
```

Aqui, a chamada de função `auth.userDetailsService` inciará a instância a `DaoAuthenticationProvider` usando nossa implementação da interface `UserDetailsService` e a registrará no gerenciador de autenticação.

Juntamente com o provedor de autenticação, precisamos configurar um gerenciador de autenticação com o esquema de codificação de senha correto que será usado para verificação de credenciais. Para isso, precisamos expor a implementação preferencial da interface `PasswordEncoder` como um `bean`.

Em nosso projeto de exemplo, usaremos o algoritmo de *hash de senha* `bcrypt`.

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    private final UserRepo userRepo;

    public SecurityConfig(UserRepo userRepo) {
        this.userRepo = userRepo;
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(username -> userRepo
        (
            .findByUsername(username)
            .orElseThrow(
                () -> new UsernameNotFoundException(
                    format("User: %s, not found", username)
                )
            )
        ));
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    // Details omitted for brevity
}
```

Após configurar o gerenciador de autenticação, precisamos agora configurar a segurança web. Estamos implementando uma `API REST` e precisamos de autenticação sem estado com um *token `JWT`*; portanto, precisamos definir as seguintes opções:

- Habilite `CORS` e desabilite `CSRF`.
- Defina o gerenciamento de sessão como *sem estado*.
- Defina o manipulador de exceções para solicitações não autorizadas.
- Defina permissões em `endpoints`.
- Adicionar filtro de *token `JWT`*

Esta configuração é implementada da seguinte maneira:

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    private final UserRepo userRepo;
    private final JwtTokenFilter jwtTokenFilter;

    public SecurityConfig(UserRepo userRepo, JwtTokenFilter jwtTokenFilter) {
        this.userRepo = userRepo;
        this.jwtTokenFilter = jwtTokenFilter;
    }

    // Details omitted for brevity

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // Enable CORS and disable CSRF
        http = http.cors().and().csrf().disable();

        // Set session management to stateless
        http = http
            .sessionManagement()
            .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and();

        // Set unauthorized requests exception handler
        http = http
            .exceptionHandling()
            .authenticationEntryPoint(
                (request, response, ex) -> {
                    response.sendError(
                        HttpServletResponse.SC_UNAUTHORIZED,
                        ex.getMessage()
                    );
                }
            )
            .and();

            // Set permission on endpoints
            http.authorizeRequests()
                // Our public endpoints
                .antMatchers("/api/public/**").permitAll()
                .antMatchers(HttpMethod.GET, "/api/author/**").permitAll()
                .antMatchers(HttpMethod.POST, "/api/author/search").permitAll()
                .antMatchers(HttpMethod.GET, "/api/book/**").permitAll()
                // Our private endpoints
                .anyRequest().authenticated();

            // Add JWT token filter
            http.addFilterBefore(
                jwtTokenFilter,
                UsernamePasswordAuthenticationFilter.class
            );
    }

    // Used by Spring Security if CORS is enabled.
    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source =
            new UrlBasedCorsConfigurationSource();

        CorsConfiguration config = new CorsConfiguration();
        
        config.setAllowCredentials(true);
        config.addAllowedOrigin("*");
        config.addAllowedHeader("*");
        config.addAllowedMethod("*");

        source.registerCorsConfiguration("/**", config);

        return new CorsFilter(source);
    }
}
```

Observe que adicionamos o `JwtTokenFilter` antes do `UsernamePasswordAuthenticationFilter` interno do *Spring Security*. Estamos fazendo isso porque precisamos acessar a identidade do usuário neste momento para realizar a autenticação/autorização, e sua extração ocorre dentro do filtro de *token `JWT`* com base no *token `JWT`* fornecido. Isso é implementado da seguinte maneira:

```java
@Component
public class JwtTokenFilter extends OncePerRequestFilter {
    private final JwtTOkenUtil jwtTokenUtil;
    private final UserRepo userRepo;

    public JwtTokenFilter(JwtTokenUtil jwtTokenUtil, UserRepo userRepo) {
        this.jwtTokenUtil = jwtTokenUtil;
        this.userRepo = userRepo;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws ServletException, IOException {
        // Get authorization header and validate
        final String header = request.getHeader(HttpHeaders.AUTHORIZATION);

        if (isEmpty(header) || !header.startsWith("Bearer ")) {
            chain.doFilter(request, response);

            return;
        }

        // Get JWT token and validate
        final String token = header.split(" ")[1].trim();

        if (!jwtTokenUtil.validate(token)) {
            chain.doFilter(request, response);

            return;
        }

        // Get user identity and set it on the sprint security context
        UserDetails userDetaiuls = userRepo
            .findBtUsername(jwtTokenUtil.getUsername(token))
            .orElse(null);

        UsernamePasswordAuthenticationToken
            authentication = new UsernamePasswordAuthenticationToken(
                userDetails, null,
                userDetails == null ?
                    List.of() : userDetails.getAuthorities()
            );

        authentication.setDetails(
            new WebAuthenticationDetailsSource().bindDetails(request)
        );

        SecurityContextHolder.getContext().setAuthentication(authentication);
        chain.doFilter(request, response);
    }
}
```

Antes de implementar nossa função de *`API` de login*, precisamos cuidar de mais uma etapa: precisamos acessar o gerenciador de autenticação. Por padrão, ele *não é acessível publicamente*, e precisamos *expô-lo explicitamente* como um `bean` em nossa classe de configuração.

Isso pode ser feito da seguinte forma:

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    // Details omitted for brevity
    @Override @Bean
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
}
```

E agora, estamos prontos para implementar nossa *função de login na `API`*:

```java
@Api(tags = "Authentication")
@RestController @RequestMapping(path = "api/public")
public class AuthApi {
    private final AuthenticationManager authenticationManager;
    private final JwtTokenUtil jwtTokenUtil;
    private final UserViewMapper userViewMapper;

    public AuthApi(AuthenticationManger authenticationManager, JwtTokenUtil jwtTokenUtil, UserViewMapper userViewMapper) {
        this.authenticationManager = authenticationManager;
        this.jwtTokenUtil = jwtTokenUtil;
        this.userViewMapper = userViewMapper;
    }

    @PostMapping("login")
    public ResponseEntity<UserView> login(@RequestBody @Valid AuthRequest request) {
        try {
            Authentication authenticate = authenticationManager
                .authenticate(
                    new UsernamePasswordAuthenticationToken(
                        request.getUsername(), request.getPassword()
                    )
                );

            User user = (User) authenticate.getPrincipal();

            return ResponseEntity.ok()
                .header(
                    HttpHeaders.AUTHORIZATION,
                    jwtTokenUtil.generateAccessToken(user)
                )
                .body(userViewMapper.toUserView(user));
        } catch (BadCredentialsException ex) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
        }
    }
}
```

Aqui verificamos as credenciais fornecidas usando o gerenciador de autenticação e, em caso de sucesso, geramos o *token `JWT`* e o retornamos com um cabeçalho `HTTP` de resposta junto com as informações de identidade do usuário no corpo da resposta.