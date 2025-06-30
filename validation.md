# Validação no *Spring Boot*

## 1. Visão Geral

Quando se trata de validar entrada do usuário, o *Spring Boot* fornece forte suporte para essa tarefa comum, porém crítica, imediatamente.

Embora o *Spring Boot* suporte integração perfeita com validadores personalizados, **o padrão de fato para executar validação é o [Hibernate Validator](http://hibernate.org/validator/)**, a implementação de referência do [*framework Bean Validation*](https://www.baeldung.com/javax-validation).

## 2. As Dependências do *Maven*

Neste caso, aprenderemos como validar objetos de domínio no *Spring Boot* **construindo um `controller` `REST` básico**.

O `controller` primeiro pegará um objeto de domínio, depois o validará com o *Hibernate Validator* e, finalmente, o persistirá em um banco de dados `H2` na memória.

*As dependências do projeto são bastante padronizadas:*

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency> 
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency> 
<dependency> 
    <groupId>com.h2database</groupId> 
    <artifactId>h2</artifactId>
    <version>2.1.214</version> 
    <scope>runtime</scope>
</dependency>
```

Como mostrado acima, incluímos [`spring-boot-starter-web`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web) em nosso arquivo `pom.xml` porque precisamos dele para criar o `controller` `REST`. Além disso, vamos verificar as versões mais recentes de [`spring-boot-starter-jpa`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-jpa) e do [banco de dados `H2`](https://mvnrepository.com/artifact/com.h2database/h2) no *Maven Central*.

A partir do **Spring Boot 2.3**, também precisamos adicionar explicitamente a dependência [`spring-boot-starter-validation`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-validation):

## 3. Uma Classe de Domínio Simples

Com as dependências do nosso projeto já definidas, precisamos definir uma classe de entidade `JPA` de exemplo, cuka função será exclusivamente *modelar usuários*.

Vamos dar uma olhada nesta aula:

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;

    @NotBlank(message = "Name is mandatory")
    private String name;

    @NotBlank(message = "Email is mandatory")
    private String email;

    // standard constructors / setters / getters / toString
}
```

A implementação da nossa *classe de entidade* `User` é bastante anêmica, mas mostra resumidamente como usar as restrições do `Bean Validation` para restringir os campos `name` e `email`.

Para simplificar, restringimos os campos de destino usando apenas a restrição `@NotBlank`. Além disso, especificamos as mensagens de erro com o atributo `message`.

Portanto, quando o *Spring Boot* valida a instância da classe, os campos restritos **não devem ser nulos e seu comprimento aparado deve ser maior que zero**.

Além disso, o `Bean Validation` fornece muitas outras restrições úteis além de `@NotBlank`. Isso nos permite aplicar e combinar diferentes regras de validação às classes restritas.

Como usaremos o [`Spring Data JPA`](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) para salvar usuários no *banco de dados `H2`* na memória, também precisamos definir uma *interface* de *repositório* simples para ter funcionalidades `CRUD` básica em objetos `User`:

```java
@Repository
public interface UserRepository extends CrudRepository<User, Long> {}
```

## 4. Implentando um `Controller` `REST`

Claro, precisamos implementar uma camada que nos permitira obter os valores atribuídos aos campos restritos do nosso objeto `User`.

Portanto, podemos validá-los e executar algumas tarefas adicionais, dependendo dos resultados da validação.

O *Spring Boot* torna **esse processo aparentemente complexo realmente simples** por meio da implementaão de um `controller` `REST`.

Vamos dar uma olhada na implementação do `controller` `REST`:

```java
@RestController
public class UserController {
    @PostMapping("/users")
    ResponseEntity<String> addUser(@Valid @RequestBody User user) {
        // persisting the user
        return ResponseEntity.ok("User is valid");
    }

    // standard constructors / other methods
}
```

Em um *contexto `Spring REST`*, a implementação do método `addUser()` é bastante padrão.

Claro, a parte mais relevante é o uso da anotação `@Valid`.

**Quando o *Spring Boot* encontra um argumento anotado com `@Valid`, ele *inicializa automaticamente* a implementação padrão do `JSR 380` - `Hibernate Validator` - e valida o argumento.**

*Quando o argumento de destino não passa na validação, o **Spring Boot** lança uma exceção `MethodArgumentNotValidException`.*

## 5. A Anotação `@ExceptionHandler`

Embora seja muito útil ter o *Spring Boot* validando o obejto `User` passado para o método `addUser()` automaticamente, a faceta que falta nesse processo é como processamos os resultados da validação.

*A anotação `@ExceptionHandler` **nos permite manipular tipos específicos de exceções por meio de um único método***.

Portanto, podemos usá-lo para processar os *erros de validação*:

```java
@ResponseStatus(HttpStatus.BAD_REQUEST)
@ExceptionHandler(MethodArgumentNotValidException.class)
public Map<String, String> handleValidationExceptions(
    MethodArgumentNotValidException ex
) {
    Map<String, String> errors = new HashMap<>();

    ex.getBindingResult().getAllErrors().forEach((error) -> {
        String fieldName = ((FieldError) error).getField();
        String errorMessage = error.getDefaultMessage();

        errors.put(fieldName, errorMessage);
    });

    return errors;
}
```

Especificamos a exceção `MethodArgumentNotValidException` como a exceção a ser tratada. Consequentemente, o *Spring Boot* chamará esse método **quando o objeto `User` especificado for inválido**.

O método armazena o nome e a mensagem de erro *pos-validação* de cada campo inválido em um `Map`. Em seguida, ele envia o `Map` de volta ao *cliente* como uma *representação `JSON`* para o processamento posterior.

Simplificando, o `controller` `REST` nos permite processar facilmente solicitações para diferentes `endpoints`, validar objetos `User` e enviar as respostas no formato `JSON`.

O *design* é flexível o suficiente para lidar com respostas do `controller` por meio de várias camadas da *web*, desde mecanismos de modelo como o [*Thymeleaf*](https://www.baeldung.com/spring-boot-crud-thymeleaf) até uma estrutura `JavaScript` completa como o [*Angular*](https://angular.io/).

## 6. Testando o `Controller` `REST`

Podemos testar facilmente a funcionalidade do nosso `controller` `REST` com um *teste de integração*.

Vamos começar a *simular/conectar* automaticamente a *implementação da interface* `UserRepository`, juntamente com a instância `UserController` e um objeto `MockMvc`:

```java
@RunWith(SpringRunner.class)
@WebMvcTest
@AutoConfigureMockMvc
public class UserControllerIntegrationTest {
    @MockBean
    private UserRepository userRepository;

    @Autowired
    UserController userController

    @Autowired
    private MockMvc mockMvc;

    // ...
}
```

Como estamos testando apenas a *camada web*, usamos a anotação `@WebMvcTest`. Ela nos permite testar facilmente soliciatações e respostas usando o conjunto de métodos estáticos implemnetados pelas classes `MockMvcRequestBuilders` e `MockMvcResultMatchers`.

Agora vamos testar o método `addUser()` com um objeto `User` *válido e um inválido* passados no corpo da solicitação:

```java
@Test
public void whenPostRequestToUsersAndValidUser_thenCorrectResponse() throws Exception {
    MediaType textPlainUtf8 = new MediaType(MediaType.TEXT_PLAIN, Charset.forName("UTF-8"));

    String user = "{\"name\": \"bob\", \"email\" : \"bob@domain.com\"}";

    mockMvc.perform(MockMvcRequestBuilders.post("/users"))
        .content(user)
        .contentType(MediaType.APPLICATION_JSON_UTF8)
        .andExpect(MockMvcResultMatchers.status().isOk())
        .andExpect(
            MockMvcResultMatchers.content()
                .contentType(textPlainUtf8)
        );
}

@Test
public void whenPostRequestToUsersAndInValidUser_thenCorrectResponse() throws Exception {
    String user = "{\"name\": \"\", \"email\": \"bob@domain.com\"}";

    mockMvc.perform(MockMvcRequestBuilders.post("/users"))
        .content(user)
        .contentType(MediaType.APPLICATION_JSON_UTF8)
        .andExpect(MockMvcResultMatchers.status().isBadRequest())
        .andExpect(MockMvcResultMatchers.jsonPath("$.name", Is.is("Name is mandatory")))
        .andExpect(
            MockMvcResultMatchers.content()
                .contentType(MediaType.APPLICATION_JSON_UTF8)
        );
}
```

Além disso, podemos testar a `API` do `controller` `REST` *usando um aplicativo gratuíto de teste de ciclo de vida da `API`*, como o [**Postman**](https://www.getpostman.com/).

## 7. Executando o Aplicativo de Exemplo

Por fim, podemos executar nosso projeto de exemplo com um método `main()` padrão:

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    public CommandLineRunner run(UserRepository userRepository) throws Exception {
        return (String[] args) -> {
            User user1 = new User("Bob", "bob@domain.com");
            User user2 = new User("Jenny", "jenny@domain.com");

            userRepository.save(user1);
            userRepository.save(user2);
            userRepository.findAll().forEach(System.out::println);
        }
    }
}
```

Como esperado, devemos ver alguns objetos `User` impressos no *console*.

Uma solicitação `POST` para o `endpoint` `http://localhost:8080/users` com um objeto `User` válido retornará a `String` `"User is valid"`.

Da mesma forma, uma solicitação `POST` com um objeto `User` sem valores de `name` e `email` retornará a seguinte resposta:

```json
{
  "name":"Name is mandatory",
  "email":"Email is mandatory"
}
```

## Fonte:

- Artigo: [Validation in Spring Boot](https://www.baeldung.com/spring-boot-bean-validation)