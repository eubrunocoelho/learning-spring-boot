# Série *Spring MVC*

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