# Série Anotações *Spring MVC*

O *Spring MVC* fornece ferramentas que controlam tanto aplicativos web típicos quanto *APIs REST*.

## Anotações da Web do *Spring*

### 1. Visão Geral

Neste tutorial, exploraremos as anotações do *Spring Web* do *pacote (package)* `org.springframework.web.bind.annotation`.

### 2. `@RequestMapping`

Simplificando, `@RequestMapping` *marca métodos de manipuladores de solicitações* dentro de classes `@Controller`; *ele pode ser configurado usando:*

- *path, ou seu aliases, nome e valor*: para qual URL o método é mapeado
- *method*: métodos `HTTP` compatíveis
- *params*: filtra solicitações com base na presença, ausência ou valor de *cabeçalhos `HTTP`*
- *consumes*: quais tipos de mídia o método pode consumir no corpo da solicitação `HTTP`
- *produces*: quais tipos de mídia o método pode produzir no corpo da resposta `HTTP`

Aqui está um exemplo rápido de como isso se parece:

```java
@Controller
class VehicleController {
    @RequestMapping(value = "/vehicles/home", method = RequestMethod.GET)
    String home() {
        return "home";
    }
}
```

Podemos fornecer **configurações padrão para todos os métodos manipuladores em uma classe `@Controller`** se aplicarmos esta anotação no nível da classe. A única **exceção é a `URL`, que o Spring não substitui** pelas configurações no nível do método, mas anexa as duas partes do caminho.

Por exemplo, a configuração a seguir tem o mesmo efeito que a anterior:

```java
@Controler
@RequestMapping(value = "/vehicles", method = RequestMethod.GET)
class VehicleController {
    @RequestMapping("/home")
    String home() {
        return "home";
    }
}
```

Além disso, `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping` e `@PathMapping` são variantes diferentes de `@RequestMapping` com o  *método `HTTP`* já definido como `GET`, `POST`, `PUT`, `DELETE` e `PATCH`.

### 3. `@RequestBody`

*`@RequestBody` mapeia o corpo da solicitação HTTP para um objeto:*

```java
@PostMapping("/save")
void saveVehicle(@RequestBody Vehicle vehicle) {
    // ...
}
```

A desserialização é automática e depende do tipo de conteúdo da solicitação.

### 4. `@PathVariable`

*Esta anotação indica que um argumento de método está vinculado a uma variável de modelo de `URI`.* Podemos especificar o *modelo de `URI`* com a anotação `@RequestMapping` e vincular um argumento de método a uma das partes do modelo com `@PathVariable`.

```java
@RequestMapping("/{id}")
Vehicle getVehicle(@PathVariable("id") long id) {
    // ...
}
```

Se o nome da parte no modelo corresponder ao nome do argumento do método, não precisamos especificá-lo na anotação:

```java
@RequestMapping("/{id}")
Vehicle getVehicle(@PathVariable long id) {
    // ...
}
```

Além disso, podemos marcar uma *variável de caminho (path variable)* como opcional, definindo o argumento `required` como `false`:

```java
@RequestMapping("/{id}")
Vehicle getVehicle(@PathVariable(required = false) long id) {
    // ...
}
```

### 5. `@RequestParam`

*Usamos `@RequestParam` para acessar parâmetros de solicitação `HTTP`.*

```java
@RequestMapping
Vehicle getVehicleByParam(@RequestParam("id") long id) {
    // ...
}
```

Ele tem as mesmas opções de configuração que a anotação `@PathVariable`.

Além dessas configurações, com `@RequestParam` podemos especificar um *valor injetado* quando o Spring não encontra valor algum ou encontra um valor vazio na requisição. Para isso, precisamos definir o argumento `defaultValue`.

*Fornecer um valor padrão define implicitamente `required` como `false`:*

```java
@RequestMapping("/buy")
Car buyCar(@RequestParam(defaultValue = "5") int seatCount) {
    // ...
}
```

Além dos parâmetros, existem **outras partes da *solicitação `HTTP`* que podemos acessar: `cookies` e `headers` *(cabeçalhos)***. *Podemos acessá-los com as anotações `@CookieValue` e `@RequestHeader`, respectivamente.*

Podemos configurá-los da mesma forma que `@RequestParam`.

### 6. Anotações de Tratamento de Resposta

Nas próximas seções, veremos as anotações mais comuns para manipular *respostas `HTTP`* no *Spring MVC*.

#### 6.1. `@ResponseBody`

Se marcamos um método de manipulador de solicitação com `@ResponseBody`, *o Spring tratará o resultado do método como a própria resposta*:

```java
@ResponseBody
@RequestMapping("/hello")
String hello() {
    return "Hello, world!";
}
```

Se anotarmos uma classe `@Controller` com esta anotação, todos os métodos do manipulador de solicitações a usarão.

#### 6.2. `@ExceptionHandler`

Com esta anotação, podemos declarar um **método *manipulador de erros* personalizado**. O Spring chama este método qunado um *método manipulador* de requisições gera qualquer uma das exceções especificadas.

A exceção capturada pode ser passada ao método como um argumento:

```java
@ExceptionHandler(IllegalArgumentException.class)
void onIllegalArgumentException(IllegalArgumentException exception) {
    // ...
}
```

#### 6.3. `@ResponseStatus`

Podemos especificar o ***status `HTTP`* desejado na resposta** anotando um *método do manipulador de solicitações* com esta anotação. Podemos declarar o *código de status* com o argumento `code` ou seu alias, o argumento `value`.

*Também podemos fornecer uma razão usando o argumento `reason`.*

*Também podemos usá-lo junto com `@ExceptionHandler`:*

```java
@ExceptionHandler(IllegalArgumentException.class)
@ResponseStatus(HttpStatus.BAD_REQUEST)
void onIllegalArgumentExeption(IllegalArgumentException exception) {
    // ...
}
```

### 7. Outras Anotações da Web

Algumas anotações não gerenciam *solicitações ou respostas `HTTP`* diretamente.

#### 7.1. `@Controller`

Podemos definir um `controller` *Spring MVC* com `@Controller`.

#### 7.2. `@RestController`

*O `@RestController` combina `@Controller` e `@ResponseBody`.*

As seguintes declarações são equivalentes:

```java
@Controller
@ResponseBody
class VehicleRestController {
    // ...
}
```

```java
@RestController
class VehicleRestController {
    // ...
}
```

#### 7.3. `@ModelAttribute`

Com esta anotação podemos **acessar elementos que já estão no modelo** de um *MVC* `@Controller`, fornecendo a chave do modelo:

```java
@PostMapping("/assemble")
void assembleVehicle(@ModelAttribute("vehicle") Vehicle vehicleInModel) {
    // ...
}
```

Assim como com `@PathVariable` e `@RequestParam`, não precisamos especificar a chave do modelo se o argumento tiver o mesmo nome:

```java
@PostMapping("/assemble")
void assembleVehicle(@ModelAttribute Vehicle vehicle) {
    // ...
}
```

Além disso, `@ModelAttribute` tem outro uso: *se anotarmos um método com ele, o Spring **adicionará automaticamente o valor de retorno do método ao modelo**:*

```java
@ModelAttribute("vehicle")
Vehicle getVehicle() {
    // ...
}
```

Como antes, não precisamos especificar a chave do modelo, o *Spring* usa o nome do método por padrão:

```java
@ModelAttribute
Vehicle vehicle() {
    // ...
}
```

Antes de o Spring chamar um método de manipulador de solicitação, ele invoca todos os métodos anotados `@ModelAttribute` na classe.

#### 7.4 `@CrossOrigin`

`@CrossOrigin` **permite a comunicação entre domínios** para os métodos do manipulador de solicitações anotados:

```java
@CrossOrigin
@RequestMapping("/hello")
String hello() {
    return "Hello, world!";
}
```

Se marcamos uma classe com ele, ele se aplicará a todos os métodos de manipulação de solicitações nela contidos.

*Podemos ajustar o comportamento do `CORS` com os argumentos desta anotação.*

### Fonte:

- Artigo: [Spring Web Annotations](https://www.baeldung.com/spring-mvc-annotations)

## Spring `@RequestMapping`

### 1. Visão Geral

A anotação `@RequestMapping` é usada para mapear solicitações da web para métodos do *Spring* `Controller`.

### 2. Noções Básicas de `@RequestMapping`

Começamos com um exemplo simples: mapear uma solicitação `HTTP` para um método usando alguns critérios básicos. Vamos considerar que o *Spring* serve conteúdo no `path` *(caminho do contexto)* raiz (`"/"`) por padrão. Todas as solicitações *`CURL`* neste artigo dependem do caminho do contexto padrão.

#### 2.1. `@RequestMapping` - Por Caminho

```java
@RequestMapping(value = "/ex/foos", method = RequestMethod.GET)
@ResponseBody
public String getFoosBySimplePath() {
    return "Get some Foos";
}
```

Para testar esse mapeamento com um comando `curl` simples, execute:

```
curl -i http://localhost:8080/ex/foos
```

#### 2.2 `@RequestMapping` - O Método `HTTP`

O parâmetro do *método `HTTP`* **não tem um valor padrão**. Portanto, se não especificarmos um valor, ele será mapeado para *qualquer solicitação `HTTP`*.

Aqui está um exemplo simples, semelhante ao anterior, mas desta vez mapeado para uma *solicitação `HTTP` `POST`*:

```java
@RequestMapping(value = "/ex/foos", method = POST)
@ResponseBody
public String postFoos() {
    return "Post some Foos";
}
```

Para testar o `POST` por meio de um comando `curl`:

```
curl -i -X POST http://localhost:8080/ex/foos
```

### 3. `@RequestMapping` e `HTTP` *Headers* (*Cabeçalhos `HTTP`*)

#### 3.1. `@RequestMapping` com o Atributo `headers`

O mapeamento pode ser ainda mais restringido *especificando um cabeçalho* para a *solicitação*:

```java
@RequestMapping(value = "/ex/foos", headers = "key=val", method = GET)
@ResponseBody
public String getFoosWithHeader() {
    return "Get some Foos with Header";
}
```

Para testar a operação, usaremos o suporte ao cabeçalho `curl`:

```
curl -i -H "key:val" http://localhost:8080/ex/foos
```

e até mesmo vários cabeçalhos por meio do atributo `headers` de `@RequestMapping`:

```java
@RequestMapping(
    value = "/ex/foos",
    headers = { "key1=val1", "key2=val2" },
    method = GET
)
@ResponseBody
public String getFoosWithHeaders() {
    return "Get some Foos with Header";
}
```

Para testar usamos o comando:

```
curl -i -H "key1:val1" -H "key2:val2" http://localhost:8080/ex/foos
```

Observe que, para a *sintaxe `curl`*, dois pontos separam a chave do cabeçalho e o valor do cabeçalho, o mesmo que na *especificação `HTTP`*, enquanto no *Spring*, o sinal de igual é usado.

#### 3.2. `@RequestMapping` *Consumes* e *Produces*

O mapeamento de *tipos de mídia produzidos por um método* de `controller` merece atenção.

Podemos mapear uma solicitação com base em seu cabeçalho `Accept` por meio do atributo de cabeçalho `@RequestMapping` apresentado acima:

```java
@RequestMapping(
    value = "/ex/foos",
    method = GET,
    headers = "Accept=application/json"
)
@ResponseBody
public String getFoosAsJsonFromBrowser() {
    return "Get some Foos with Header Old";
}
```

A correspondência para essa maneira de definir o cabeçalho `Accept` *é flexível* - ela usa `contains` em vez de `equals`, então uma solicitação como a seguinte ainda seria mapeada corretamente:

```
curl -H "Accept:application/json,text/html" http://localhost:8080/ex/foos
```

*A partir do **Spring 3.1**, a anotação `@RequestMapping` agora tem os atributos `produces` e `consumes`, especficamente para essa finalidade:*

```java
@RequestMapping(
    value = "/ex/foos",
    method = RequestMethod.GET,
    produces = "application/json"
)
@ResponseBody
public String getFoosAsJsonFromREST() {
    return "Get some Foos with Header New";
}
```

Além disso, o antigo tipo de mapeamento com o atributo `headers` será automaticamente convertido para o novo mecanismo `produces` *a partir do **Spring 3.1***, então os resultados serão idênticos.

```
curl -H "Accept:application/json" http://localhost:8080/ex/foos
```

Além disso, *`produces` também suporta vários valores*:

```java
@RequestMapping(
    value = "/ex/foos",
    method = GET,
    produces = { "aplication/json", "application/xml" }
)
```

Tenha em mente que essas - as formas antiga e nova de especificar o cabeçalho `Accept` - são basicamente o mesmo mapeamento, então *o Spring não as permitirá juntas*.

Ter ambos os métodos ativos resultaria em:

```
Caused by: java.lang.IllegalStateException: Ambiguous mapping found. 
Cannot map 'fooController' bean method 
java.lang.String 
org.baeldung.spring.web.controller
  .FooController.getFoosAsJsonFromREST()
to 
{ [/ex/foos],
  methods=[GET],params=[],headers=[],
  consumes=[],produces=[application/json],custom=[]
}: 
There is already 'fooController' bean method
java.lang.String 
org.baeldung.spring.web.controller
  .FooController.getFoosAsJsonFromBrowser() 
mapped.
```

*Uma nota final sobre os novos mecanismos de `produces` e `consumes`, que se comportam de forma diferente da na maioria das outras anotações: quando especificadas no nível do tipo, **as anotações no nível do método não complementam, mas substituem** as informações no nível de tipo.*

### 4. `@RequestMapping` com `@PathVariable`

Partes do `URI` de mapeamento podem ser vinculadas a variáveis por meio da anotação `@PathVariable`.

#### 4.1 `@PathVariable` Única

Um exemplo simples com uma única `@PathVariable`:

```java
@RequestMapping(value = "/ex/foos/{id}", method = GET)
@ResponseBody
public String getFoosBySimplePathWithPathVariable(
    @PathVariable("id") long id
) {
    return "Get a specific Foo with id=" + id;
}
```

```
curl http://localhost:8080/ex/foos/1
```

Se o nome do parâmetro do método corresponder exatamente ao nome da *variável de caminho*, isso pode ser simplficado *usando `@PathVariable`* sem valor:

```java
@RequestMapping(value = "/ex/foos/{id}", method = GET)
@ResponseBody
public String getFoosBySimplePathWithPathVariable(@PathVariable String id) {
    return "Get a specific Foo with id=" + id;
}
```

Observe que `@PathVariable` se beneficia da conversão automática de tipo, então poderiámos ter declarado `id` como:

```java
@PathVariable long id
```

#### 4.2. Múltiplas `@PathVariable`

Um *`URI` mais complexo* pode precisar mapear várias partes do `URI` para *vários valores*:

```java
@RequestMapping(value = "/ex/foos/{fooid}/bar/{barid}", method = GET)
@ResponseBody
public String getFoosBySimplePathWithPathVariables (
    @PathVariable long fooid, @PathVariable long barid
) {
    return "Get a specific Bar with id=" + barid +
        " from a Foo with id=" + fooid;
}
```

Isso é facilmente testado com um `curl` da mesma maneira:

```
curl http://localhost/foos/1/bar/2
```

#### 4.3. `@PathVariable` com `Regex`

*Expressões regulares* também podem ser usadas ao mapear `@PathVariable`.

```java
@RequestMapping(value = "/ex/bars/{numericId:[\\d+]}", method = GET)
@ResponseBody
public String getBarsBySimplePathWithPathVariable(
    @PathVariable long numericId
) {
    return "Get a specific Bar with id" + numericId;
}
```

Os seguintes `URIs` corresponderão:

```
http://localhost:8080/ex/bars/1
```

Mas não à:

```
http://localhost:8080/ex/bars/abc
```

### 5. `@RequestMapping` com `@RequestParam`

`@RequestMapping` permite o **mapeamento fácil de parâmetros de `URL` com a anotação `@RequestParam`**.

Agora estamos mapeando uma solicitação para um `URI`:

```
http://localhost:8080/ex/bars?id=100
```

```java
@RequestMappring(value = "/ex/bars", method = GET)
@ResponseBody
public String getBarBySimplePathWithRequestParam(
    @RequestParam("id") long id
) {
    return "Get a specific Bar with id=" + id;
}
```

Estamos então extraindo o valor do parâmetro `id` usando a anotação `@RequestParam("id")` na assinatura do método do `controller`.

Para enviar uma solicitação com parâmetro `id`, usaremos o suporte a parâmetros em `curl`:

```
curl -i http://localhost:8080/ex/bars --get -d id=100
```

Neste exemplo, o parâmetro foi vinculado diretamente, sem ter sido declarado primeiro.

Para *cenários mais avançados*, *`@RequestMapping` pode opcionalmente definir os parâmetros* como outra maneira de restringir o mapeamento da solicitação:

```java
@RequestMapping(value = "/ex/bars", params = "id", method = GET)
@ResponseBody
public String getBarBySimplePathWithExplicitRequestParam(
    @RequestParam("id") long id
) {
    return "Get a specific Bar with id=" + id;
}
```

Mapeamentos ainda *mais flexíveis* são permitidos. Vários valores de *parâmetros* podem ser definidos, e nem todos precisam ser usados:

```java
@RequestMapping(
    value = "/ex/bars",
    params = { "id", "second" },
    method = GET
)
@ResponseBody
public String getBarBySimplePathWithExplicitRequestParams(
    @RequestParam("id") long id
) {
    return "Narrow Get a specific Bar with id=" + id;
}
```

E claro, uma solicitação para um `URI` como:

```
http://localhost:8080/ex/bars?id=100&second=something
```

sempre será mapeado para a melhor correspondência - que é a correspondência mais restrita, o que define o `id` e o `second`.

### 6. Casos Extremos de Mapeamento de Solicitações

#### 6.1. `@RequestMappoing` - Vários Caminhos Mapeados para o mesmo Método do `Controller`

Embora um único valor de caminho `@RequestMapping` seja normalmente usado para um único método de `controller` *(apenas uma boa prática, não uma regra rígida)*, há alguns casos em que *pode ser necessário mapear várias solicitações para um mesmo método*.

Neste caso, **o atributo value de `@RequestMapping` aceita múltiplos mapeamentos**.

```java
@RequestMapping(
    value = { "/ex/advanced/bars", "/ex/advanced/foos" },
    method = GET
)
@ResponseBody
public String getFoosOrBarsByPath() {
    return "Advanced - Get some Foos or Bars";
}
```

Agora ambos os comandos `curl` devem atingir o mesmo método:

```
curl -i http://localhost:8080/ex/advanced/foos
curl -i http://localhost:8080/ex/advanced/bars
```

#### 6.2. `@RequestMapping` - Múltiplos Métodos de Solicitação `HTTP` para o mesmo Método de `Controller`

Várias solicitações usando diferentes *verbos `HTTP`* podem ser mapeados para o *mesmo método* de `controller`:

```java
@RequestMapping(
    value = "/ex/foos/multiple",
    method = { RequestMethod.PUT, RequestMethod.POST }
)
@RepsonseBody
public String putAndPostFoos() {
    return "Advanced - PUT and POST within single method";
}
```

Com `curl`, ambos agora atingirão o mesmo método:

```
curl -i -X POST http://localhost:8080/ex/foos/multiple
curl -i -X PUT http://localhost:8080/ex/foos/multiple
```

#### 6.3. `@RequestMapping` - um `fallback` para todas as Solicitações

Para implementar um `fallback` simples para todas as solicitações usando um método `HTTP` específico, por exemplo, para um `GET`:

```java
@RequestMapping(value = "*", method = RequestMethod.GET)
@ResponseBody
public String getFallback() {
    return "Fallback for GET Requests";
}
```

Ou mesmo, para todas as solicitações:

```java
@RequestMapping(
    value = "*",
    method = { RequestMethod.GET, RequestMethod.POST ... }
)
@ResponseBody
public String allFallback() {
    return "Fallback for All Requests";
}
```

#### 6.4. Erro de Mapeamento Ambíguo

O erro de mapeamento ambíguo ocorre quando o *Spring* avalia dois ou mais mapeamentos de requisição como sendo iguais para métodos de `controller` diferentes. Um mapeamento de requisição é o mesmo quando possui o mesmo método `HTTP`, `URL`, parâmetros, cabeçalhos e tipo de mídia.

Por exemplo:

```java
@GetMapping(value = "foos/duplicate")
public String duplicate() {
    return "Duplicate";
}

@GetMapping(value = "foos/duplicate")
public String duplicateEx() {
    return "Duplicate";
}
```

A exceção lançada geralmente tem mensagens de erro semelhantes a isto:

```
Caused by: java.lang.IllegalStateException: Ambiguous mapping.
  Cannot map 'fooMappingExamplesController' method 
  public java.lang.String org.baeldung.web.controller.FooMappingExamplesController.duplicateEx()
  to {[/ex/foos/duplicate],methods=[GET]}:
  There is already 'fooMappingExamplesController' bean method
  public java.lang.String org.baeldung.web.controller.FooMappingExamplesController.duplicate() mapped.
```

Uma leitura cuidadosa da mensagem de erro aponta para o fato de que o *Spring* não consegue mapear o método `org.baeldung.web.controller.FooMappingExamplesController.duplicateEx()`, pois ele tem um mapeamento conflitante com um `org.baeldung.web.controller.FooMappingExamplesController.duplicate()` já mapeado.

**O trecho de código abaixo não resultará em erro de mapeamente ambíguo porque ambos os métodos retornam tipos de conteúdo diferentes:**

```java
@GetMapping(value = "foos/duplicate", produces = MediaType.APPLICATION_XML_VALUE)
public String duplicateXml() {
    return "<message>Duplicate</message>";
}

@GetMapping(value = "foos/duplicate", produces = MediaType.APPLICATION_JSON_VALUE)
public String duplicateJson() {
    return "{\"message\":\"Duplicate\"}";
}
```

Essa diferenciação permite que nosso `controller` retorne a representação de dados correta com base no cabeçalho `Accepts` fornecido na solicitação.

Outra maneira de resolver isso é *atualizar a `URL`* atribuída a qualquer um dos dois métodos envolvidos.

### 7. Novos Atalhos de Mapeamento de Solicitações

O **Spring Framework 4.3** introduzir algumas novas anotações de *mapeamento `HTTP`*, todas baseadas em `@RequestMapping`:

- `@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@DeleteMapping`
- `@PatchMapping`

Essas novas anotações podem melhorar a legibilidade e reduzir a verbosidade do código.

Vamos analisar essas novas anotações em ação criando uma *`API RESTful`* que suporta *operações `CRUD`*:

```java
@GetMapping("/{id}")
public ResponseEntity<?> getBazz(@PathVariable String id){
    return new ResponseEntity<>(new Bazz(id, "Bazz"+id), HttpStatus.OK);
}

@PostMapping
public ResponseEntity<?> newBazz(@RequestParam("name") String name){
    return new ResponseEntity<>(new Bazz("5", name), HttpStatus.OK);
}

@PutMapping("/{id}")
public ResponseEntity<?> updateBazz(
  @PathVariable String id,
  @RequestParam("name") String name) {
    return new ResponseEntity<>(new Bazz(id, name), HttpStatus.OK);
}

@DeleteMapping("/{id}")
public ResponseEntity<?> deleteBazz(@PathVariable String id){
    return new ResponseEntity<>(new Bazz(id), HttpStatus.OK);
}
```

### 8. Configuração *Spring*

A configuração do *Spring MVC* é bastante simples, considerando que nosso `FooController` está definido no seguinte pacote:

```java
package org.baeldung.spring.web.controller;

@Controller
public class FooController {...}
```

Precisamos apenas de uma classe `@Configuration` para habilitar o suporte completo ao `MVC` e configurar o escaneamento de `classpath` para o `controller`.

```java
@Configuration
@EnableWebMvc
@ComponentScan({ "org.baeldung.spring.web.controller" })
public class MvcConfig {
    // ...
}
```

### Fonte:

- Artigo: [Spring @RequestMapping](https://www.baeldung.com/spring-requestmapping)

## Anotação Spring `@RequestParam`

### 1. Visão Geral

Neste artigo, exploraremos a anotação `@RequestParam` do *Spring* e seus atributos.

**Simplificando, podemos usar `@RequestParam` para extrair parâmetros de consulta, parâmetros de formulário e até mesmo arquivos da solicitação.**

### 2. Um Mapeamento Simples

Digamos que temos um endpoint `/api/foos` que recebe um parâmetro de consulta chamado `id`:

```java
@GetMapping("/api/foos")
@ResponseBody
public String getFoos(@RequestParram String id) {
    return "ID: " + id;
}
```

Neste exemplo, usamos `@RequestParam` para extrair o parâmetro de consulta `id`.

Uma simples solicitação `GET` invocaria `getFoos`:

```
http://localhost:8080/spring-mvc-basics/api/foos?id=abc
----
ID: abc
```

Em seguida, **vamos dar uma olhada nos atributos da anotação: `name`, `value`, `required`, `defaultValue`**.

### 3. Especificando o Nome do Parâmetro de Solicitação

No exemplo anterior, tanto o nome da variável quanto o nome do parâmetro são iguais.

**Ás vezes, queremos que sejam diferentes.** Ou, se não estivermos usando o *Spring Boot*, talvez precisamos fazer uma *configuração especial* em tempo de compilação, ou os nomes dos parâmetros não estarão no `bytecode`.

*Podemos configurar o nome `@RequestParam` usando o atributo `name`.*

```java
@PostMapping("/api/foos")
@ResponseBody
public String addFoo(@RequestParam(name = "id") String fooId, @RequestParam String name) {
    return "ID: " + fooId + " Name: " + name;
}
```
Também podemos fazer `@RequestParam(value = "id")` ou apenas `@RequestParam("id")`.

### 4. Parâmetros de Solicitação Opcionais

*Parâmetros de métodos anotados com `@RequestParam` são necessários por padrão.*

Isso significa que se o parâmetro não estiver presente na solicitação, receberemos um erro:

```
GET /api/foos HTTP/1.1
----
400 Bad Request
Required String parameter 'id' is not present
```

**Podemos configurar nosso `@RequestParam` para ser opcional, com o atributo `required`:**

```java
@GetMapping("/api/foos")
@ResponseBody
public String getFoos(@RequestParam(required = false) String id) {
    return "ID: " + id;
}
```

Neste caso, ambos:

```
http://localhost:8080/spring-mvc-basics/api/foos?id=abc
----
ID: abc
```

e

```
http://localhost:8080/spring-mvc-basics/api/foos
----
ID: null
```

invocará o método corretamente.

*Quando o parâmetro não é especificado, o parâmetro do método é vinculado a `null`.*

#### 4.1. Usando `Optional` em *Java 8*

Alternativamente, podemos encapsular o parâmetro em `Optional`.

```java
@GetMapping("/api/foos")
@ResponseBody
public String getFoos(@RequestParam Optional<String> id) {
    return "ID: " + id.orElseGet(() -> "not provided");
}
```

*Neste caso, não precisamos especificar o atributo `required`.*

```
http://localhost:8080/spring-mvc-basics/api/foos 
---- 
ID: not provided
```

### 5. Um Valor Padrão para `@RequestParam`

Também podemos definir um valor padrão para `@RequestParam` usando o atributo `defaultValue`:

```java
@GetMapping("/api/foos")
@ResponseBody
public String getFoos(@RequestParam(defaultValue = "test") String id) {
    return "ID: " + id;
}
```

*Isso é como `required = false`, pois o usuário não precisa mais fornecer o parâmetro:*

```
http://localhost:8080/spring-mvc-basics/api/foos
----
ID: test
```

Embora ainda estejamos autorizados a fornecê-lo:

```
http://localhost:8080/spring-mvc-basics/api/foos?id=abc
----
ID: abc
```

*Observe que quando definimos o atributo `defaultValue`, `required` é de fato definido como `false`.*

### 6. Mapeando Todos os Parâmetros

**Também podemos ter vários parâmetros sem definir seus nomes** ou contagem usando apenas um `Map`:

```java
@PostMapping("/api/foos")
@ResponseBody
public String updateFoos(@RequestParam Map<String, String> allParams) {
    return "Parameters are " + allParams.entrySet();
}
```

que então refletirá quaisquer parâmetros enviados:

```
curl -X POST -F 'name=abc' -F 'id=123' http://localhost:8080/spring-mvc-basics/api/foos
----
Parameters are {[name=abc], [id=123]}
```

### 7. Mapeando um Parâmetro Multivalor

Um *único `@RequestParam`* pode ter vários valores:

```java
@GetMapping("/api/foos")
@ResponseBody
public String getFoos(@RequestParam List<String> id) {
    return "IDs are " + id;
}
```

*E o **Spring MVC** mapeará um parâmetro `id` delimitado por vírgulas:*

```
http://localhost:8080/spring-mvc-basics/api/foos?id=1,2,3
----
IDs are [1,2,3]
```

**ou uma lista de parâmetros de `id`** separados:

```
http://localhost:8080/spring-mvc-basics/api/foos?id=1&id=2
----
IDs are [1,2]
```

### Fonte:

- Artigo: [Spring @RequestParam Annotation](https://www.baeldung.com/spring-request-param)

## *Spring MVC* e a anotação `@ModelAttribute`

### 1. Visão Geral

**Uma das anotações mais importantes do *Spring MVC* é a anotação `@ModelAttribute`.**

`@ModelAttribute` é uma anotação que vincula um parâmetro de método ou valor de retorno de método a um atributo de modelo nomeado e, em seguida, o expôe a uma visualização da web.

Neste artigo, demonstraremos a usabilidade e a funcionalidade desta anotação por meio de um conceito comum: *um formulário enviado por um funcionário da empresa*.

### 2. `@ModelAttribute` em Detalhes

Como o parágrafo introdutório revelou, podemos usar `@ModelAttribute` como um parâmetro de método ou no nível do método.

#### 2.1. No Nível do Método

Quando usamos a anotação no nível do método, indica que o objetivo do método é adicionar um ou mais atributos do modelo. Esses métodos suportam os mesmos tipos de argumento que os métodos `@RequestMapping`, mas não podem ser mapeados diretamente para solicitações.

Vejamos um exemplo rápido:

```java
@ModelAttribute
public void addAttributes(Model model) {
    model.addAttriute("msg", "Welcome to the Netherlands!");
}
```

No exemplo acima, vemos um método que adiciona um atributo chamado `msg` a todos *os modelos* definidos na classe de `controller`.

Em geral, o *Spring MVC* sempre fará uma chamada para esse método primeiro, antes de chamar qualquer método do manipulador de requições. Basicamente, os métodos **`@ModelAttribute` são invocados antes dos métodos do `controller` anotados com `@RequestMapping`**. Isso ocorre porque o objeto do modelo precisa ser criado antes que qualquer processamento seja iniciado dentro dos métodos de `controller`.

*Também é importante anotar a classe respectiva como `@ControllerAdvice`. Assim, podemos adicionar valores no modelo que serão identificados como **globais**. Isso significa que, para cada solicitação, existe um valor padrão para cada método na resposta.*

#### 2.2. Como um Argumento de Método

Quando usamos a anotação como argumento de método, isso indica que devemos recuperar o argumento do modelo. Quando a anotação não estiver presente, ela deve primeiro ser instanciada e, em seguida, adicionada ao modelo. Uma vez presentes no modelo, os campos de argumentos devem ser preenchidos com todos os parâmetros da solicitação que tenham os nomes correspondentes.

No código a seguir, preencheremos o atributo do modelo `employee` com dados de um formulário enviado ao *endpoint* `addEmployee`. O *Spring MVC* faz isso em segundo plano antes de invocar o método `submit`:

```java
@RequestMapping(value = "/addEmployee", method = RequestMethod.POST)
public String submit(@ModelAttribute("employee") Employee employee) {
    // Code that uses the employee object

    return "employeeView";
}
```

Mais adiante neste artigo, veremos um exemplo completo de como usar o objeto `employee` para preencher o modelo `employeeView`.

Ele vincula os dados do formulário a um `bean`. **O `controller` anotado com `@RequestMapping` pode ter agumentos de classe personalizados anotados com `@ModelAttribute`.**

No *Spring MVC*, chamamos isso de vinculação de dados, um mecanismo comum que nos poupa de ter que analisar cada campo de formulário individualmente.

### 3. Exemplo de Formulário

Nesta seção, veremos o exemplo descrito na seção de visão geral, um formulário muito básico que solicita que um usuário (*especificamente um funcionário da empresa*) insira algumas informações pessoais (*nome e ID*). Após a conclusão do envio, e sem erros, o usuário espera ver os dados enviados anteriormente exibidos em outra tela.

#### 3.1. A `View`

Vamos primeiro criar um formulário simples com os campos `id` e `name`:

```jsp
<form:form method="POST" action="/spring-mvc-basics/addEmployee" modelAttribute="employee">
    <form:label path="name">Name</form:label>
    <form:input path="name" />
    <form:label path="id">Id</form:label>
    <form:input path="id" />
    <input type="submit" value="Submit" />
</form:form>
```

#### 3.2. O `Controller`

Aqui está a classe de `controller`, onde implementaremos a lócia para a visualização mencionada anteriormente:

```java
@Controller
@ControllerAdvice
public class EmployeeController {
    private Map<Long, Employee> employeeMap = new HashMap<>();

    @RequestMapping(value = "/addEmployee", method = RequestMethod.POST)
    public String submit(
        @ModelAttribute("employee") Employee employee,
        BindingResult result, ModelMap model
    ) {
        if (result.hasErrors()) {
            return "error";
        }

        model.addAttribute("name", employee.getName());
        model.addAttribute("id", employee.getId());

        employeeMap.put(employee.getId(), employee);

        return "employeeView";
    }

    @ModelAttribute
    public void addAttribute(Model model) {
        model.addAttribute("msg", "Welcome to the Netherlands!");
    }
}
```

No método `submit()`, temos um objeto `Employee` vinculado à nossa `View`. Podemos mapear os campos do nosso formulário para um modelo de objeto de forma simples. No método, estamos buscando valores do formulário e definindo-os como `ModelMap`.

**No final, retornamos employeeView, o que significa que chamamos o respectivo arquivo `JSP` como um representante da `View`.**

Além disso, há também um método `addAttributes()`. Sua finalidade é adicionar valores no `Model` que serão identificados globalmente. Ou seja, cada solicitação para cada método do `controller` retornará um valor padrão como resposta. Também precisamos anotar a classe específica como `@ControllerAdvice`.

#### 3.3. O `Model`

Como mencionado anteriormente, o objeto `Model` é muito simples e contém tudo o que os atributos de *"front-end"* exigem. Agora, vejamos um exemplo:

```java
@XmlRootElement
public class Employee {
    private long id;
    private String name;

    public Employee(long id, String name) {
        this.id = id;
        this.name = name;
    }

    // standard getters and setters removed
}
```

#### 3.4. Conclusão

`@ControllerAdvice` auxilia um `controller` e, em particular, os métodos `@ModelAttribute` que se aplicam a todos os métodos `@RequestMapping`. Obviamente, nosso método `addAttributes()` será o primeiro a ser executado, antes dos demais métodos `@RequestMapping`.

Tendo isso em mente, e depois que `submit()` e `addAttributes()` forem executados, podemos referir a eles na `View` retornada da classe `Controller` mencionando seus nomes dentro de um duo de chaves dolarizadas, como `${name}`.

#### 3.5. Visualização de Resultados

Agora vamos imprimir o que recebemos do formulário:

```jsp
<h3>${msg}</h3>
Name: ${name}
ID: ${id}
```

### Fonte:

- Artigo: [Spring MVC and the @ModelAttribute Annotation](https://www.baeldung.com/spring-mvc-and-the-modelattribute-annotation)

## As Anotações Spring `@Controller` e `@RestController`

### 1. Visão Geral

Neste artigo, discutiremos a diferença entre as anotações `@Controller` e `@RestController` no *Spring MVC*.

Podemos usar a primeira anotação para *controladores* *Spring* tradicionais, e ela faz parte do framework há muito tempo.

O **Spring 4.0** introduziu a anotação `@RestController` para simplificar a *criação de serviços web `RESTful`*. **É uma anotação conveniente que combina `@Controller` e `@ResponseBody`**, eliminando a necessidade de anotar todos os métodos de tratamento de solicitações da classe `controller` com a anotação `@ResponseBody`.

### 2. *Spring MVC* `@Controller`

Podemos anotar `controllers` clássicos com a anotação `@Controller`. Esta é simplesmente uma especialização da classe `@Component`, que nos permite detectar automaticamente classes de implementação por meio de escaneamento do `classpath`.

Normalmente usamos `@Controller` em combinação com uma anotação `@RequestMapping` para métodos de tratamento de solicitações.

Vamos ver um exemplo rápiudo do `controller` *Spring MVC*:

```java
@Controller
@RequestMapping("books")
public class SimpleBookController {
    @GetMapping("/{id}", produces = "application/json")
    public @ResponseBody Book getBook(@PathVariable int id) {
        return findBookById(id);
    }

    private Book findBookById(int id) {
        // ...
    }
}
```

Anotamos o método de tratamento de solicitações com `@ResponseBody`. Essa anotação permite a serialização automática do objeto de retorno no `HttpResponse`.

### 3. *Spring MVC* `@RestController`

`@RestController` é uma versão especializada de `controller`. Inclui as anotações `@Controller` e `@ResponseBody` e, como resultado, simplifica a implementação de `controller`.

```java
@RestController
@RequestMapping("books-rest")
public class SimpleBookRestController {
    @GetMapping("/{id}", produces = "application/json")
    public Book getBook(@PathVariable int id) {
        return findBookById(id);
    }

    private Book findBookById(int id) {
        // ...
    }
}
```

*O `controller` é anotado com a anotação `@RestController`; portanto, o `@ResponseBody` não é necessário.*

Cada método de tratamento de solicitação da classe do `controller` serializa automaticamente os objetos de retorno em `HttpResponse`.

### Fonte:

- Artigo: [The Spring @Controller and @RestController](https://www.baeldung.com/spring-controller-vs-restcontroller)

## Anotaçoes `@RequestBody` e `@ResponseBody` do *Spring*

### 1. Introdução

Neste artigo, fornecemos uma visão geral concisa das anotações `@RequestBody` e `@ResponseBody` do *Spring*.

### 2. `@RequestBody`

Simplificando, **a anotação `@RequestBody` mapeia o corpo do `HttpRequest` para um objeto de transferência ou domínio, permitindo a desserialização automática** do corpo do `HttpRequest` de entrada em um objeto **Java**.

Primeiro, vamos dar uma olhada em um método de `controller` do *Spring*:

```java
@PostMapping("/request")
public ResponseEntity postController(
    @RequestBody LoginForm loginForm
) {
    exampleService.fakeAuthenticate(loginForm);
    
    return ResponseEntity.ok(HttpStatus.OK);
}
```

O *Spring* desserializa automaticamente o `JSON` em um tipo **Java**, supondo que um tipo apropriado seja especificado.

Por padrão, **o tipo que anotamos com a anotação `@RequestBody` deve corresponder ao `JSON` enviado do nosso `controller` do lado do cliente**:

```java
public class LoginForm {
    private String username;
    private String password;

    // ...
}
```

Aqui, o objeto que usamos para representar o corpo do `HttpRequest` é mapeado para nosso objeto `LoginForm`.

Vamos testar isso usando `CURL`:

```
curl -i \
-H "Accept: application/json" \
-H "Content-Type:application/json" \
-X POST --data 
  '{"username": "johnny", "password": "password"}' "https://localhost:8080/spring-boot-rest/post/=request"
```

Isso é tudo o que precisamos para uma **API REST** do *Spring* e um cliente *Angular* usando a anotação `@RequestBody`.

### 3. `@ResponseBody`

A anotação `@ResponseBody` informa ao `controller` que o objeto retornado é automaticamente serializado em `JSON` e passado de volta para o objeto `HttpResponse`.

Suponha que temos um objeto `Response` personalizado:

```java
public class RespondeTransfer {
    private String text;

    // standard getters/setters
}
```

Em seguida, o `controller` associado pode ser implementado:

```java
@Controller
@RequestMapping("/post")
public class ExamplePostController {
    @Autowired
    ExampleService exampleService;

    @PostMapping("/response")
    @ResponseBody
    public ResponseTransfer postResponseController(
        @RequestBody LoginForm loginForm
    ) {
        return ResponseTransfer("Thanks For Posting!!!");
    }
}
```

No console do desenvolvedor do nosso navegador ou usando uma ferramenta como o *Postman*, podemos ver a seguinte resposta:

```json
{
    "text": "Thanks For Posting!!!"
}
```

*Lembre-se, não precisamos anotar os `controllers` anotados com `@RestController` com a anotação `@ResponseBody`, pois o **Spring** faz isso por padrão.*

#### 3.1. Definindo o Tipo de Conteúdo

Quando usamos a anotação `@ResponseBody`, ainda podemos definir explicitamente o tipo de conteúdo que nosso método retorna.

Para isso, *podemos usar o atributo `produces` do `@RequestMapping`*. Observe que as anotações como `@PostMapping`, `@GetMapping`, etc. definem aliases para esse parâmetro.

Vamos agora adicionar um novo `endpoint` que envia uma resposta `JSON`:

```java
@PostMapping(value = "/content", produces = MediaType.APPLICATION_JSON_VALUE)
@ResponseBody
public ResponseTransfer postResponseJsonContent(
    @RequestBody LoginForm loginForm
) {
    return new ResponseTransfer("JSON Content!");
}
```

No exemplo, usamos a constante `MediaType.APPLICATION_JSON_VALUE`. Como alternativa, podemos usar `application/json` diretamente.

Em seguida, vamos implmentar um novo método, mapeado para o mesmo caminho `/content`, mas retornando conteúdo `XML`:

```java
@PostMapping(value = "/content", produces = MediaType.APPLICATION_XML_VALUE)
@ResponseBody
public ResponseTransfer postResponseXMLContent(
    @RequestBody LoginForm loginForm
) {
    return new ResponseTransfer("XML Content!");
}
```

*Dependendo do valor de um parâmetro `Accept` enviado no cabeçalho da solicitação, obteremos respostas diferentes.*

Vamos ver isso em ação:

```
curl -i \ 
-H "Accept: application/json" \ 
-H "Content-Type:application/json" \ 
-X POST --data 
  '{"username": "johnny", "password": "password"}' "https://localhost:8080/spring-boot-rest/post/content"
```

O comando `CURL` retorna uma resposta `JSON`:

```
HTTP/1.1 200
Content-Type: application/json
Transfer-Encoding: chunked
Date: Thu, 20 Feb 2020 19:43:06 GMT

{"text":"JSON Content!"}
```

Agora, vamos alterar o parâmetro `Accept`:

```
curl -i \
-H "Accept: application/xml" \
-H "Content-Type:application/json" \
-X POST --data
  '{"username": "johnny", "password": "password"}' "https://localhost:8080/spring-boot-rest/post/content"
```

Desta vez obtemos um conteúdo `XML`:

```
HTTP/1.1 200
Content-Type: application/xml
Transfer-Encoding: chunked
Date: Thu, 20 Feb 2020 19:43:19 GMT

<ResponseTransfer><text>XML Content!</text></ResponseTransfer>
```

### 4. Conclusão

Criamos um cliente *Angular* simples para o aplicativo *Spring* que demonstra como usar as anotações `@RequestBody` e `@ResponseBody`.

Além disso, mostramos como definir um tipo de contéudo ao usar `@ResponseBody`.

### Fonte:

- Artigo: [Spring's RequestBody and ResponseBody Annotations](https://www.baeldung.com/spring-request-response-body)

## Usando *Spring* `@ResponseStatus` para Definir o Código de Status `HTTP`

### 1. Introdução

No *Spring MVC*, temos muitas maneiras de **definir o código de status de uma resposta `HTTP`**.

Neste artigo, veremos a maneira mais direta: *usando a anotação `@ResponseStatus`*.

### 2. Sobre os Métodos de `controller`

Quando um `endpoint` retorna com sucesso, o *Spring* fornece uma resposta `HTTP 200` (*OK*).

Se quisermos especificar o **status de resposta de um método `controller`**, podemos marcá-lo com `@ResponseStatus`. Ele possui dois argumentos intercambiáveis para o status de resposta desejado: *código* e *valor*. Por exemplo, *podemos indicar que o servidor se recusa a preparar café poque é um bule de chá*:

```java
@ResponseStatus(HttpStatus.I_AM_TEAPOT)
void teaPot() {}
```

Quando queremos *sinalizar um erro*, podemos fornecer uma mensagem de erro por meio do argumento `reason`:

```java
@ResponseStatus(HttpStatus.BAD_REQUEST, reason = "Some parameters are invalid")
void onIllegalArgumentException(IllegalArgumentException exception) {}
```

Observe que, quando definimos `reason`, o *Spring* chama `HttpServletResponse.sendError()`. Portanto, ele enviará unma **página de erro `HTML` para o cliente, o que o torna inadequado para `endpoints` *REST***.

Observe também que o *Spring* só usa `@ResponseStatus` quando **o método marcado é concluído com sucesso** (sem lançar uma exceção).

### 3. Com Manipuladores de Erros

Temos *três maneiras* de usar `@ResponseStatus` para converter uma *exceção* em um status de resposta `HTTP`:

- usando `@ExceptionHandler`
- usando `@ControllerAdvice`
- marcando a classe `Exception`

Para usar as *duas primeiras* soluções, precisamos definir um método de tratamento de erros.

Podemos usar `@ResponseStatus` com esses métodos de tratamento de erros **da mesma forma que fizemos com métodos MVC regulares** na seção anterior.

*Quando não precisamos de resposta de erro dinâmicas, a **solução mais direta é a terceira**: marcar a classe `Exception` com `@ResponseStatus`:*

```java
@ResponseStatus(code = HttpStatus.BAD_REQUEST)
class CustomException extends RuntimeException {}
```

Quando o *Spring* captura essa exceção, **ele usa as configurações que fornecemos em `@ResponseStatus`**.

Observe que quando marcamos uma classe `Exception` com `@ResponseStatus`, o *Spring* sempre chama `HttpServletResponse.sendError()`, *independentemente de definirmos o motivo ou não*.

Observe também que o *Spring* usa a mesma configuração para *subclasses*, a menos que as marquemos com `@ResponseStatus` também.

### Fonte:

- Artigo: [Using @ResponseStatus to Set HTTP Status Code](https://www.baeldung.com/spring-response-status)

## Novas Anotações de Atalho do *Spring* `@RequestMapping`

### 1. Visão Geral

O **Spring 4.3** introduziu algumas anotações compostas em nível de métodos muito interessantes para suavizar o tratamento de `@RequestMapping` em projetos típicos do *Spring MVC*.

### 2. Novas Anotações

Normalmente, se quiséssemos *implementar o manipulador de `URL`* usando a anotação `@RequestMapping` tradicional, seria algo assim:

```java
@RequestMapping(value = "/get/{id}", method = RequestMethod.GET)
```

A nova abordagem torna possível encurtar isso simplesmente para:

```java
@GetMapping("/get/{id}")
```

*Atualmente, o **Spring** suporta cinco tipos de anotações integradas para lidar co diferentes tipos de métodos de solicitação `HTTP` de entrada: `GET`, `POST`, `PUT`, `DELETE` e `PATH`.* Essas anotações são:

- `@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@Deleteapping`
- `@PathMapping`

Pela convenção de nomenclatura, podemos ver que cada anotação serve para manipular o respectivo tipo de método de solicitação de entrada, ou seja, `@GetMapping` é usado para manipular o tipo de método de solicitação `GET`, `@PostMapping` é usado para manipular o tipo de método de solicitação `POST`, etc.

### 3. Como Funciona

*Todas as anotações acima já estão antoadas internamente com `@RequestMapping` e o respectivo valor no elemento do método.*

Por exemplo, se observamos o *código-fonte* da anotação `@GetMapping`, podemos ver que ela já está anotada com `RequestMethod.GET` da seguinte maneira:

```java
@Target({ java.lang.annotation.ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@RequestMapping(method = { RequestMethod.GET })
public @interface GetMapping {
    // abstract codes
}
```

*Todas as anotações são criadas da mesma maneira, ou seja, `@PostMapping` é anotado com `RequestMethod.POST`, `@PutMapping` é anotado com `RequestMethod.PUT`, etc.*

### 4. Implementação

Vamos tentar usar essas anotações para criar um aplicativo *REST* rápido.

Observe que, como usaremos o **Maven** para construir o projeto e o *Spring MVC* para criar nosso aplicativo, precisamos adicionar as dependências necessárias no `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.2.RELEASE</version>
</dependency>
```

*A versão mais recente do spring-webmvc* está disponível no [Repositório Central Maven](https://mvnrepository.com/artifact/org.springframework/spring-webmvc).

Agora, precisamos criar o `controller` para mapear a `URL` da solicaitação de entrada. Dentro desse `controller`, usaríamos todas essas anotações, uma por uma.

#### 4.1. `@GetMapping`

```java
@GetMapping("/get")
public @ResponseBody ResponseEntity<String> get() {
    return new ResposeEntity<String>("GET Response", HttpStatus.OK);
}
```

```java
@GetMapping("/get/{id}")
public @ResponseBody ResponseEntity<String> getById(@PathVariable String id) {
    return new ResponseEntity<String>("GET Response: " + id, HttpStatus.OK);
}
```

#### 4.2. `@PostMapping`

```java
@PostMapping("/post")
public @ResponseBody ResponseEntity<String> post() {
    return new ResponseEntity<String>("POST Response", HttpStatus.OK);
}
```

#### 4.3. `@PutMapping`

```java
@PutMapping("/put")
public @ResponseBody ResponseEntity<String> put() {
    return new ResponseEntity<String>("PUT Response", HttpStatus.OK);
}
```

#### 4.4. `@DeleteMapping`

```java
@DeleteMapping("/delete")
public @ResponseBody ResponseEntity<String> delete() {
    return new ResponseEntity<String>("DELETE Response", HttpStatus.OK);
}
```

#### 4.5. `@PatchMapping`

```java
@PatchMapping("/patch")
public @ResponseBody ResponseEntity<String> patch() {
    return new ResponseEntity<String>("PATCH Response", HttpStatus.OK);
}
```

**Pontos a serem observados:**

- Utilizamos as anotações necessárias para manipular métodos `HTTP` de entrada com `URI`. Por exemplo, `@GetMapping` para manipular o `URI` `/get`, `@PostMapping` para manipular o `URI` `/post` e assim por diante.
- Como estamos criando uma aplicação baseada em `REST`, estamos retornando uma string constante *(exclusiva para cada solicitação)* com *código 200 de resposta* para simplificar a aplicação. Usamos a anotação `@ResponseBody` do *Spring* neste caso.
- Se tivéssemos que manipular qualquer variável de caminho de `URL`, poderíamos fazê-lo de uma maneira muito menos comum do que costumávamos fazer no caso de usar `@RequestMapping`.

### 5. Testando a Aplicação

Para testar a aplicação, precisamos criar alguns casos de testes usando `JUnit`. Usaríamos `SpringJUnit4ClassRunner` para iniciar a classe de teste. Criaríamos cinco casos de teste diferentes para testar cada anotação e cada manipulador que declaramos no `controller`.

Vamos simplificar o caso de teste de exemplo de `@GetMapping`:

```java
@Test
public void givenUrl_whenGetRequest_thenFindGetResponse() throws Exception {
    MockHttpServletRequestBuilder builder = MockMvcRequestBuilders.get("/get");

    ResultMatcher contentMatcher = MockMvcResultMatchers.content().string("GET Response");

    this.mockMvc.perform(builder).addExcept(contentMatcher).andExcept(MockMvcResultMatchers.status().isOk());
}
```

Como podemos ver, estamos esperando uma *string constante* `GET Response`, quando acessamos o `GET` com a `URL` `/get`.

Agora, vamos criar o caso de teste para testar `@PostMapping`:

```java
@Test
public void givenUrl_whenPostRequest_thenFindPostResponse() throws Exception {
    MockHttpServletRequestBuilder builder = MockMvcRequestBuilders.post("/post");

    ResultMatcher contentMatcher = MockMvcResultMatchers.content().string("POST Response");

    this.mockMvc.perform(builder).andExcept(contentMatcher).andExcept(MockMvcResultMatchers.status().isOk());
}
```

**Da mesma forma, criamos o restante dos casos de teste par testar todos os métodos `HTTP`.**

Como alternativa, podemos sempre usar quaisquer *cliente `REST` comum*, por exemplo, *PostMan*, *RESTClient* etc., par testar nossa aplicação. Nesse caso, precisamos ter um pouco de cuidado ao escolher o tipo de *método `HTTP`* correto ao usar o *cliente `REST`*. Caso contrário, ele retornaria o *status de erro **405***.

### Fonte:

- Artigo: [Spring @RequestMapping New Shortcut Annotations](https://www.baeldung.com/spring-new-requestmapping-shortcuts)