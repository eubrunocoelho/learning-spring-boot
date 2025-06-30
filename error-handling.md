# Tratamento de Erros para *REST* com *Spring*

## 1. Visão Geral

Este artigo ilustrará *como implementar o Tratamento de Exceções com **Spring** para uma `API REST`*. Aprenderemos que existem várias possibilidades para isso. Todas elas têm uma coisa em comum: *lidam muito bem com a **separação de responsabilidades***. *O aplicativo pode lançar exceções normalmente para indicar algum tipo de falha, que será tratada separadamente.*

## 2. `@ExceptionHandler`

Podemos usar `@ExceptionHandler` para anotar métodos que o *Spring* invoca automaticamente quando a exceção fornecida ocorre. Podemos especificar a exceção com a anotação ou declarando-a como um parâmetro de método, o que nos permite ler detalhes do objeto de exceção para tratá-la corretamente. O método em si é tratado como um método do `Controller`, portanto:

- Ele pode retornar um objeto que é renderizado no corpo da resposta ou uma `ResponseEntity` completa. A `Content Negotation` *(Negociação de Conteúdo)* é permitida aqui desde o **Spring 6.2**.
- Ele pode retornar um objeto `ProblemDetail`. O *Spring* definirá o cabeçalho `Content-Type` automaticamente como `application/problem+json`.
- Podemos especificar um código de retorno com `@ResponseStatus`.

O *exception handler* *(manipulador de exceção)* mais simples que retorna um código de status 400 poderia ser:

```java
@ResponseStatus(HttpStatus.BAD_REQUEST)
@ExceptionHandler(CustomException1.class)
public void handleException1() {}
```

Também poderíamos declarar a exceção tratada como um parâmetro de método, por exemplo, para ler detalhes da exceção e criar um objeto de detalhes do problema *compatível com **[RFC-8457](https://www.rfc-editor.org/rfc/rfc9457.html)***:

```java
@ResponseStatus(HttpStatus.BAD_REQUEST)
@ExceptionHandler
public ProblemDetail handleException2(CustomException2 ex) {
    // ...
}
```

Desde o **Spring 6.2**, podemos escrever diferentes manipuladores de exceção para diferentes tipos de conteúdo:

```java
@ResponseStatus(HttpStatus.BAD_REQUEST)
@ExceptionHandler(produces = MediaType.APPLICATION_JSON_VALUE)
public CustomExceptionObject handleException3Json(CustomException3 ex) {
    // ...
}

@ResponseStatus(HttpStatus.BAD_REQUEST)
@ExceptionHandler(produces = MediaType.TEXT_PLAIN_VALUE)
public String handleException3Text(CustomException3 ex) {
    // ...
}
```

E também poderíamos escrever manipuladores de exceção para diferentes tipos de exceção. Se precisarmos de detalhes no método manipulador, usamos a *superclasse* compartilhada de todos os tipos de exceção como parâmetro do método:

```java
@ResponseStatus(HttpStatus.BAD_REQUEST)
@ExceptionHandler({
    CustomException4.class,
    CustomException5.class
})
public ResponseEntity<CustomExceptionObject> handleException45(Exception ex) {
    // ...
}
```

### 2.1. Tratamento de Exceções Locais (Nível de `Controller`)

Podemos colocar esses métodos de manipulação na classe do `controller`.

```java
@RestController
public class FooController {
    // ...

    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(CustomException1.class)
    public void handleException() {
        // ...
    }
}
```

Poderíamos usar essa abordagem sempre que precisarmos de tratamento de exceções específico para um `controller`. Mas ela tem a desvantagem de **não podermos usá-la em vários `controllers`, a menos que a coloquemos em uma classe base e usemos herança**. Há, porém, outra abordagem que se encaixa melhor no sentido de composição em vez de herança.

### 2.2. Tratamento de Exceções Globais

Um `@ControllerAdvice` contém código compartilhado entre vários `controllers`. É um tipo especial de componente *Spring*. Especialmente para `APIs REST`, onde o valor de retorno de cada método deve ser renderizado no corpo da resposta, existe um `@RestControllerAdvice`.

Então, para lidar com uma exceção específica para todos os `controllers` do aplicativo, poderíamos escrever uma classe simples:

```java
@RestControllerAdvice
public class MyGlobalExceptionHandler {
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(CustomException1.class)
    public void handleException() {
        // ...
    }
}
```

Devemos saber que também existe uma classe base (`ResponseEntityExceptionHandler`) da qual podemos herdar para usar funcionalidades predefinidas comuns, como a geração de `ProblemDetails`. Também podemos herdar métodos para lidar com exceções típicas do `MVC`:

```java
@ControllerAdvice
public class MyCustomResponseEnityExceptionHandler extends ResponseEntityExceptionHandler {
    @ExceptionHandler({
        IllegalArgumentException.class,
        IllegalStateException.class
    })
    ResponseEntity<Object> handleConflict(RuntimeException ex, WebRequest request) {
        String bodyOfResponse = "This should be application specific";

        return super.handleExceptionInternal(ex, bodyOfResponse, new HttpHeaders(), HttpStatus.CONFLICT, request);
    }

    @Override
    protected ResponseEntity<Object> handleHttpMediaTypeNotAcceptable(
        HttpMediaTypeNotAcceptableException ex, HttpHeaders headers, HttpStatusCode status, WebRequest request
    ) {
        // ... (customization, maybe invoking the overridden method)
    }
}
```

Observe que, no exemplo acima, não há necessidade de adicionar a anotação `@RestControllerAdvice` à classe, já que todos os métodos retornam uma `ResponseEntity`, então usamos a anotação `@ControllerAdvice` aqui.

## 3. Anotar Exceções Diretamente

Outra abordagem simples é anotar diretamente nossa exceção personalizada com `@ResponseStauts`:

```java
@ResponseStatus(value = HttpStatus.NOT_FOUND)
public class MyResourceNotFoundException extends RuntimeException {
    // ...
}
```

Assim como `DefaultHandlerExceptionResolver`, este *resolvedor* é limitado na forma como lida com o corpo da resposa - ele mapeia o código de `Status` na resposta, mas o corpo ainda é *nulo*. Só podemos usá-lo para nossas exceções personalizadas, pois não podemos anotar classes existentes já compiladas. E, em uma *arquitetura em camadas*, **devemos usar essa abordagem apenas para exceções específicas de limites**.

A propósito, devemos observar que **exceções neste contexto normalmente são derivadas de `RuntimeException`**, pois não precisamos de verificações do compilador aqui. Caso contrário, isso levaria a declarações de `throws` desnecessárias em nosso código.

## 4. Exceção de Status de Resposta

Um `controller` também pode lançar uma `ResponseStatusException`. Podemos criar uma instância dele, fornecendo um `HttpStauts` e, opcionalmente, um `reason` e uma `cause`:

```java
@GetMapping(value = "/{id}")
public Foo findById(@PathVariable("id") Long id) {
    try {
        // ...
    } catch (MyResourceNotFoundException ex) {
        throw new ResponseStatusException(
            HttpStatus.NOT_FOUND, "Foo Not Found", ex
        );
    }
}
```

Quais são os benefícios de usar `ResponseStatusException`?

- Excelente para prototipagem: podemos implemnetar uma solução básica muito rapidamente.
- Um tipo, vários códigos de status: um tipo de exceção pode levar várias respostas diferentes. *Isso reduz o aclopamento rígido em comparação com `@ExceptionHandler`*.
- Não precisaremos criar tantas classes de exceção personalizadas.
- Temos *mais controle sobre o tratamento de exceções*, pois elas podem ser criadas programaticamente.

E quanto às compensações?

- Não há uma maneira unificada de lidar com exceções: *é mais difícil impor algumas convenções em todo o aplicativo do que `@ControllerAdvice`, que fornece uma abordagem global*.
- Duplicação de código: podemos acabar replicando código em vários `controllers`.
- Em arquitetura em camadas, devemos lançar apenas essas exceções do `controller`. Como mostrado no exemplo de código, podemos precisar de encapsulamento de exceções para exceções de camadas subjacentes.

## 5. `HandlerExceptionResolver`

Outra soliução é definir um `HandlerExceptionResolver` personalizado. Isso resolverá qualquer exceção gerada pela aplicação. Também nos permitirá implementar um *mecanismo uniforme de tratamento de exceções* em nossa `API REST`.

### 5.1. Implementações Existentes

Já existem implementações existentes que são habilitadas por padrão no `DispatcherServlet`:

- `ExceptionHandlerExceptionResolver` é na verdade o componente principal de como o mecanismo `@ExceptionHandler` apresentado anteriormente funciona.
- `DefaultHandlerExceptionResolver` é usado para resolver exceções padrão do *Spring* para seus *códigos de status `HTTP`* correspondentes, ou seja, os códigos de status de erro cliente **4xx** e erro do servidor **5xx**. [Aqui está a lista completa](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/mvc/support/DefaultHandlerExceptionResolver.html) das exceções do *Spring* que ele manipula e como elas se relacionam com os códigos de status. Embora defina o código de status da resposta corretamente, uma **limitação é que não define nada para o corpo da resposta**.

### 5.2. `HandlerExceptionResolver` Personalizado

A combinação de `DefaultHandlerExceptionResolver` e `ResponseStatusExceptionResolver` contribui significativamente para fornecer um bom mecanismo de tratamento de erros para um serviço `Spring RESTful`. A desvantagem, como mencionado anteriormente, é que **não temos controle sobre o corpo da resposta**.

O ideal seria podermos gerar a saída em `JSON` ou `XML`, depnedendo do formato solicitado pelo cliente *(por meio de cabeçalho `Accept`)*.

Isso por si só justifica a criação de *um novo `resolver` de exceção personalizado*:

```java
@Component
public class RestResponseStatusExceptionResolver extends AbstractHandlerExceptionResolver {
    @Override
    protected ModalAndView doResolverException(
        HttpServletRequest request,
        HttpServletResponse response,
        Object handler,
        Exception ex
    ) {
        try {
            if (ex instanceof IllegalArgumentException) {
                return handleIllegalArgument(
                    (IllegalArgumentException) ex, response, handler
                );
            }

            // ...
        } catch (Exception handlerException) {
            logger.warn(
                "Handling of [" + ex.getClass().getName() + "]
                    resulted in Exception", handlerException
                );
        }

        return null;
    }

    private ModelAndView handleIllegalArgument(
        IllegalArgumentException ex, HttpServletResponse response
    ) throws IOException {
        response.sendError(HttpServletResponse.SC_CONFLICT);
        String accept = request.getHeader(HttpHeaders.ACCEPT);

        // ...

        return new ModelAndView();
    }
}
```

Um detalhe a ser observado aqui é que temos à *requisição* em si, então podemos considerar o valor do cabeçalho `Accept` enviado pelo cliente.

Por exemplo, se o cliente solicitar `application/json`, então no caso de uma condição de erro, gostaríamos de ter certeza de retornar um corpo de resposta codificado com `application/json`.

O outro detalhe importante da implementação é que **retornamos um `ModalAndView` - este é o corpo da resposta** e nos permitirá definir o que for necessário nele.

Essa abordagem é um *mecanismo consistente e facilmente configurável* para o tratamento de erros de um serviço *Spring `REST`*.
No entanto, ele *tem limitações*: ele interage com o `HttpServletResponse` de *baixo nível* e se encaixa no antigo modelo `MVC` que usa `ModelAndView`.

## 6. Outras Notas

### 6.1. Lidando com Exceções Existentes

Há diversas exceções com as quais frequentemente lidamos em implementações *`REST` típicas*:

- `AccessDeniedException` ocorre quando um usuário autenticado tenta acessar recursos para os quais não tem autoridade suficiente. Por exemplo, isso pode acontecer quando usamos *anotações de segurança* em nível de método, como `@PreAuthorize`, `@PostAuthorize` e `@Secure`.
- `ValidationException` e `ConstraintViolationException` ocorrem quando usamos [`Bean Validation`](https://www.baeldung.com/spring-boot-bean-validation).
- `PersistenceException` e `DataAccessException` ocorrem quando usamos [`Spring Data JPA`](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa).

É claro que usaremos o mecanismo global de tratamento de exceções que discutimos anteriormente para tratar também a `AccessDeniedException`:

```java
@RestControllerAdvice
public class MyGlobalExceptionHandler {
    @ResponseStatus(value = HttpStauts.FORBIDDEN)
    @ExceptionHandler(AccessDeniedException.class)
    public void handleAccessDeniedException() {
        // ...
    }
}
```

### 6.2. Suporte para *Spring Boot*

**O *Spring Boot* fornece uma implementação `ErrorController` para lidar com erros de maneira sensata.**

Em poucas palavras, ele serve como uma página de erro de *fallback* para navegadores (também conhecida como Página de Erro `Whitelabel`) e uma resposta `JSON` para solicitações `RESTful` e não `HTML`:

```json
{
    "timestamp": "2019-01-17T16:12:45.977+0000",
    "status": 500,
    "error": "Internal Server Error",
    "message": "Error processing the request!",
    "path": "/my-endpoint-with-exceptions"
}
```

Como de costume, o *Spring Boot* permite configurar esses recursos com propriedades:

- `server.error.whitelabel.enabled`: pode ser usado para desabilitar a página de erro `Whitelabel` e contar com o contêiner de `servlet` para fornecer uma mensagem de erro `HTML`.
- `server.error.include-stacktrace`: com um valor `always`, inclui o *rastreamento de pilha* *(stacktrace)* tanto no `HTML` quanto na resposta padrão `JSON`.
- `server.error.include-message`: desde a **versão 2.3**, o *Spring Boot* oculta o campo de *mensagem* na resposta para evitar vazamento de informações confidenciais; podemos usar essa propriedade com um valor `always` para habilitá-lo.

Além dessas propriedades, **podemos fornecer nosso próprio mapeamento de resolução de exibição para  erro, substituindo a página `Whitelabel`**.

Também podemos personalizar os atributos que queremos mostrar na resposta incluindo um `bean` `ErrorAttributes` no contexto. Podemos estender a classe `DefaultErrorAttributes` fornecida pelo *Spring Boot* para facilitar as coisas:

```java
@Component
public class MyCustomErrorAttributes extends DefaultErrorAttributes ?{
    @Override
    public Map<String, Object> getErrorAttributes(
        WebRequest webRequest, ErrorAttributeOptions options
    ) {
        Map<String, Object> errorAttributes = super.getErrorAttributes(webRequest, options);

        errorAttributes.put("locale", webRequest.getLocale().toString());
        errorAttributes.remove("error");

        // ...

        return errorAttributes;
    }
}
```

Se quisermos ir mais longe e definir (ou substituir) como o aplicativo tratará erros para um tipo de conteúdo específico, podemos registrar um `bean` `ErrorController`.

Novamente, podemos usar o `BasicErrorController` padrão pelo *Spring Boot* para nos ajudar.

Por exemplo, imagine que queremos personalizar como nossa aplicação lida com erros acionados em `endpoints` `XML`. tudo que precisamos fazer é definir um método público usando `@RequestMapping` e declarar que ele produz o tipo de mídia `application/xml`:

```java
@Component
public class MyErrorController extends BasicErrorController {
    public MyErrorController(
        ErrorAttributes errorAttributes, ServerProperties serverProperties
    ) {
        super(errorAttributes, serverProperties.getError());
    }

    @RequestMapping(produces = MediaType.APPLICATION_XML_VALUE)
    public ResponseEntity<Map<String, Object>> xmlError(HttpServletRequest request) {
        // ...
    }
}
```

***Observação:* aqui, ainda estamos contando com as propriedades `server.error`. do *Spring Boot* que podemos ter definido em nosso projeto e que estão vinculadas ao `bean` `ServerProperties`.**

## 7. Conclusão

Neste artigo, discutimos diversas maneiras de implementar um mecanismo de tratamento de exceções para uma `API REST` no *Spring*. Comparamos cad uma delas em termos de seus casos de uso.

Devemos observar que é possível combinar diferentes abordagens em uma aplicação. Por exemplo, podemos implementar um *`@ControllerAdvice` globalmente*, mas também *`@ResponseStatusExceptions` localmente*.

## Fonte:

- Artigo: [Error Handling for REST with Spring](https://www.baeldung.com/exception-handling-for-rest-with-spring)