# Série de *Dependency Injection* de *Spring Boot*

*Injeção de Dependência* (*Dependency Injection*) é um **aspecto fundamental do *Spring Boot***, por meio do qual o contêiner *"injeta"* objetos em outros objetos ou *"dependências"*.

## Introdução à *Inversão de Controle (IoC)* e *Injeção de Dependência (DI)*

### 1. Visão Geral

Neste tutorial, apresentaremos os conceitos de *IoC (Inversão de Controle)* e *DI (Injeção de Dependência)*.

### 2. O que é *Inversão de Controle*?

*Inversão de Controle* é um princípio da engenharia de software que transfere o controle de objetos ou partes de um programa para um *contêiner ou framework*. *Usamos esse princípio com mais frequência no contexto da programação orientada a objetos.*

Em contraste com a programação tradicional, na qual nosso código personalizado faz chamadas de biblioteca, o **IoC** permite que um framework assuma o *controle de fluxo* de um programa e faça chamadas para nosso código personalizado. Para isso, os frameworks usam abstrações com comportamento adicional incorporado. *Se quisermos adicionar nosso próprio comportamento, precisamos estender as classes do framework ou plugar nossas próprias classes.*

Podemos alcançar a Inversão de Controle por meio de vários mecanismos, como:
*Padrão de design de estratégia; Padrão de localizador de serviço; Padrão de fábrica e injeção de dependência (**DI**)*.

### 3. O que é *Injecão de Dependência*?

*Injeção de Dependência* é um padrão que podemos usar para implemnetar **IoC**, onde o controle que está sendo invertido está definindo as dependências de um objeto.

Conectar objetos com outros objetos, ou *"injetar"* objetos em outros objetos, é feito por um montador e não pelos próprios objetos.

Injeção de dependência na programação tradicional:

```java
public class Store {
    private Item item;

    public Store() {
        item = new ItemImpl1();
    }
}
```

No exemplo acima, *precisamos instanciar uma implementação de interface Item dentro da própria classe `Store`*.

Usando *DI*, podemos reescrever o exemplo sem especificar a implementação do *Item* que queremos:

```java
public class Store {
    private Item item;

    public Store(Item item) {
        this.item = item;
    }
}
```

Tanto *IoC* quanto *DI* são conceitos simples, mas têm implicações profundas na maneira como estruturamos nossos sistemas.

### 4. O Contêiner *Spring IoC*

Um contêiner *IoC* é uma característica comum de estruturas que implementam *IoC*.

No *Spring Boot*, a interface `ApplicationContext` representa o contêiner *IoC*. O contêiner do *Spring* é *responsável por instanciar, configurar e montar objetos* conhecidos como *beans*, bem como gerencias seus ciclos de vida.

O *Spring Boot* fornece diversas implementações da interface `ApplicationContext`:
`AnnotationConfigApplicationContext`, `ClassPathXmlApplicationContext` e `FileSystemXmlApplicationContext` para aplicativos independentes, e `WebApplicationContext` para aplicativos web.

Para montar *beans*, o contêiner usa metadados de configuração, que podem estar na forma de configuração `XML` ou *anotações*.

Aqui está uma maneira de instanciar manualmente um contêiner:

```java
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
```

E aqui está um exemplo de instanciação manual de um contêiner usando `AnnotationConfigApplicationContext`:

```java
AnnotationConfigApplicationContext annotationContext = new AnnotationConfigApplicationContext();
```

Ao criar uma instância de `AnnotationConfigApplicationContext` e fornecer a ela uma ou mais classes de configuração, ela verifica essas classes em busca de anotações `@Bean` e outras anotações relevantes. Em seguida, *ele **inicializa** e **gerencia os beans** definidos nessas classes, configurando suas dependências e gerenciado seu ciclo de vida.*

**A injeção de dependência do Spring Boot pode ser feita por meio de *construtores, setters ou campos*.**

### 5. *Injeção de Dependência* Baseada em Construtor

No caso de *injeção de dependência baseada em contrutor*, o contêiner invocará um construtor com argumentos, *cada um representando uma dependência que queremos definir*.

O **Spring Boot** resolve cada argumento principalmente por tipo, seguido pelo nome do atributo e índice para desambiguação.

```java
@Configuration
public class AppConfig {
    @Bean
    public Item item1() {
        return new ItemImpl1();
    }

    @Bean
    public Store store() {
        return new Store(item1());
    }
}
```

A *anotação `@Configuration`* *indica que a classe é uma fonte de definições de **bean***. Também podemos *adicioná-la várias classes de configuração*.

*Usamos a anotação `@Bean` em um método para definir um bean.* Se não especificarmos um nome personalizado, *o nome do **bean** será o nome do método padrão*.

Para um *bean* como o escopo *singleton* padrão, o *Spring Boot* primeiro verifica se já existe uma instância em *cache do bean* e só cria uma nova *caso isso não aconteça*. Se estivermos usando o *escopo de protótipo*, o contêiner *retorna uma nova instância de **bean*** para cada chamada de método.

Outra maneira de criar a configuração dos beans é através de *configuração `XML`*:

```xml
<bean id="item1" class="org.bealdung.store.ItemImpl1" />
<bean id="store" class="org.bealdung.store.Store">
    <constructor-arg type="ItemImpl1" index="0" name="item" ref="item1" />
</bean>
```

### 6. *Injeção de Dependência* Baseada em Setter

Para ***DI*** *baseada em setter*, o contêiner chamará os métodos da nossa classe após invocar um construtor sem argumentos ou um método de fábrica estático sem argumentos para instanciar o *bean*. Vamos criar esta configuração usando anotações:

```java
@Bean
public Store store() {
    Store store = new Store();

    store.setItem(item1());

    return store;
}
```

Podemos usar `XML` para a mesma configuração de *beans*:

```xml
<bean id="store" class="org.bealdung.store.Store">
    <property name="item" ref="item1" />
</bean>
```

Podemos *combinar tipos de injeção* baseados em construtor e setter para o mesmo *bean*.

### 7. *Injeção de Dependência* Baseada em Campo

No caso de **Field-Based DI**, podemos injetar as dependências marcando-as com uma anotação `@Autowired`:

```java
public class Store {
    @Autowired
    private Item item;
}
```

Ao construir *o objeto `Store`*, se não houver um *método construtor ou setter* para injetar o *bean* ´Item´, o contêiner usará reflexão para *injetar `Item` em `Store`*.

Essa abordagem pode parercer mais simples e limpa, *mas não recomendamos usá-la porque tem desvantagens*:

- *Este método usa reflexão para injetar as dependências, o que é mais custoso do que a injeção baseada em construtor ou setter.*
- *É muito fácil continuar adicionando múltiplas dependências usando essa abordagem. Se estivéssemos usando injeção de construtor, ter múltiplos argumentos nos faria pensar que a classe faz mais de uma coisa, o que pode violar o **Princípio de Responsabilidade Única**.*

### 8. Dependências de *Autowiring*

**Wiring** permite que o *contêiner do **Spring Boot*** resolva automaticamente dependências entre *beans colaborativos* inspecionando os *beans que foram definidos*.

Existem quatro modos de *autowiring* de um *bean* usando uma configuração `XML`:

- ***no*:** O valor padrão - isso significa que nenhuma conexão automática é usada para o *bean* e temos que nomear explicitamente as dependências.
- ***byName*:** A conexão automática é feita com base no nome da propriedade, portanto, o *Spring Boot* procurará um *bean* com o mesmo nome da propriedade que precisa ser definida.
- ***byType*:** Semlhante à *autowiring ByName*, mas com base no tipo da propriedade. Isso significa que o *Spring Boot* procurará um *bean* com o mesmo tipo da propriedade para definir. Se houver *mais de um bean* desse tipo, o framework *lançará uma exceção*.
- ***constructor*:** A conexão automática é feita com base nos argumentos do construtor, o que significa que o Spring Boot procurará por beans com o mesmo tipo de argumentos do construtor.

Por exemplo, vamos conectar *automaticamente o bean* de `item` definido acima por tipo no *bean* de `store`:

```java
@Bean(autowire = Autowire.BY_TYPE)
public class Store {
    private Item item;

    public setItem(Item item) {
        this.item = item;
    }
}
```

*Observe que a propriedade **autowire** está obsoleta a partir do Spring 5.1.*

Também podemos injetar *beans* usando a anotação `@Autowired` para *autowiring* por tipo:

```java
public class Store {
    @Autowired
    private Item item;
}
```

Se houver *mais de um bean* do mesmo tipo, podemos usar a anotação `@Qualifier` para *referenciar um **bean** pelo nome*:

```java
public class Store {
    @Autowired
    @Qualifier("item1")
    private Item item;
}
```

Agora vamos conectar automaticamente os *beans* por tipo por meio da configuração `XML`:

```xml
<bean id="store" class="org.bealdung.store.Store" autowire="byType"></bean>
```

Em seguida, vamos injetar um bean chamado `item` na propriedade `item` do *bean* *armazenado* por nome por meio de `XML`:

```xml
<bean id="item" class="org.bealdung.store.ItemImpl1" />
<bean id="store" class="org.bealdung.store.Store" autowire="byName"></bean>
```

*Podemos substituir a conexão automática definindo dependências explicitamente por meio de argumentos do construtor ou setters.*

### 9. *Lazy Initialized Beans*

*Por padrão, o contêiner cria e configura todos os singleton beans durante a inicialização. Para evitar isso, podemos usar o atributo `lazy-init` com o valor `true` na configuração do bean:*

```xml
<bean id="item1" class="org.bealdung.store.ItemImpl1" lazy-init="true" />
```

Consequentemente, *o bean `item1`* só será inicializado quando for solicitado pela *primeira vez*, e não na inicialização. A vantagem é o tempo de inicialização mais rápido, mas a desvantagem é que não descobriremos nunhum erro de configuração até que o *bean* seja solicitado, o que pode levar várias horas ou até dias após a aplicação já estar em execução.

### Fonte:

- Artigo: [Inversion of Control and Dependency Injection with Spring](https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring)

## Guia Rápido para Escopos *Spring Bean*

### 1. Visão Geral

O escopo de um *bean* define o *ciclo de vida* e a *visibilidade* desse *bean* nos contextos em que o usamos.

A versão mais recente do *Spring Boot* define 6 tipos de escopo:

- *singleton*
- *prototype*
- *request*
- *session*
- *application*
- *websocket*

*Os últimos quatro escopos mencionados, request, session, application e websocket, estão disponíveis somente em aplicativos com suporte web.*

### 2. Escopo *Singleton*

Quando definimos um *bean* com o escopo *singleton*, contêiner cria uma única instância desse *bean*; todas as requisições para esse nome de *bean* retornarão o mesmo objeto, que é armazenado em cache. *Quaisquer modificações no objeto serão refletidas em todas as referências ao bean.* Este escopo é o valor padrão se nenhum outro escopo for especificado.

Vamos criar uma entidade `Person` para exemplificar o conteito de escopos:

```java
public class Person {
    private String name;

    // standard contructor, getters and setters
}
```

Depois, definimos o *bean* com o escopo `singleton` usando a anotação `@Scope`:

```java
@Bean
@Scope("singleton")
public Person personSingleton() {
    return new Person();
}
```

Também podemos usar uma constante em vez do valor `String` da seguinte forma:

```java
@Scope(value = ConfigurableBeanFactory.SCOPE_SINGLETON)
```

Agora podemos prosseguir para escrever um teste que mostra que dois objetos que fazem referência ao mesmo *bean* terão os mesmos valores, *mesmo que apenas um deles altere seu estado, já que ambos estão referenciando a mesma instância de bean.*

```java
private static final String NAME = "John Smith";

@Test
public void givenSingletonScope_whenSetName_thenEqualNames() {
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("scopes.xml");

    Person personSingletonA = (Person) applicationContext.getBean("personSingleton");
    Person personSingletonB = (Person) applicationContext.getBean("personSingleton");

    personSingleTonA.setName(NAME);
    Assert.assertEquals(NAME, personSingletonB.getName());

    ((AbstractApplicationContext) applicationContext).close();
}
```

O arquivo `scopes.xml` deve conter as definições `XML` dos *beans* usados:

```xml
<?xml version="1.0" enconding="UTF-8"?>
<beans 
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd"   
>
    <bean id="personSingleton" class="org.bealdiung.scopes.Person" scope="singleton"/>
</beans>
```

### 3. Escopo *Prototype*

Um *bean* com o escopo *prototype* retornará uma instância diferente sempre que for solicitado do contêiner. Isso é definido definindo o valor `prototype` para a anotação `@Scope` na definição do *bean*:

```java
@Bean
@Scope("prototype")
public Person personPrototype() {
    return new Person();
}
```

Podemos usar uma constatnte:

```java
@Scope(value = ConfigurableBeanFactore.SCOPE_PROTOTYPE)
```

Escreveremos um teste semelhante ao anterior, que mostra dois objetos solicitando o mesmo nome de *bean* com o escopo de `prototype`. Eles terão estados diferentes, *pois não estão mais se referindo à mesma instância de bean*:

```java
private static final String NAME = "John Smith";
private static final String NAME_OTHER = "Anna Jones";

@Test
public void givenPrototypeScope_whenSetNames_thenDifferentNames() {
    ApplicationContexxt applicationContext = new ClassPathApplicationContext("scopes.xml");

    Person personPrototypeA = (Person) applicationContext.getBean("personPrototype");
    Person personPrototypeB = (Person) applicationContext.getBean("personPrototype");

    personPrototypeA.setName(NAME);
    personPrototypeB.setName(NAME_OTHER);

    Assert.assertEquals(NAME, personPrototypeA.getName());
    Assert.assertEquals(NAME_OTHER, personPrototypeB.getName());

    ((AbstractApplicationContext) applicationContext).close();
}
```

O arquivo `scopes.xml` é semelhante ao apresentado na seção anterior, ao adicionar a *definição xml* para o *bean* com o escopo de `prototype`:

```xml
<bean id="personPrototype" class="org.bealdung.scopes.Person" scope="prototype"/>
```

### 4. Escopos com reconhecimento da Web

Como mencionado anteriormente, existem quatro escopos adicionais que estão disponíveis apenas em um contexto de aplicativo com suporte à web.

O escopo `request` cria uma instância de *bean* para uma única solicitação `HTTP`, enquanto o escopo `session` cria uma instância de *bean* para o ciclo de vida de um `ServletContext`, e o escopo de `websocket` cria para uma sessão específica do `WebSocket`.

Criamos uma classe para usar na *instanciação dos beans*:

```java
public class HelloMessageGenerator {
    private String message;

    // standard getter and setter
}
```

#### 4.1.1 Escopo de `request`

Podemos definir o bean com o escopo de `request` usando a anotação `@Scope`:

```java
@Bean
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS);
public HelloMessageGenerator requestScopedBean() {
    return new HelloMessageGenerator();
}
```

O atributo `proxyMode` é necessário porque, no momento da instanciação do *contexto da aplicação web*, não há nenhuma requisição ativa. O *Spring* cria um *proxy* para ser injetado como dependência e instancia o *bean* de destino quando necessário em uma requisição.

Também podemos usar uma anotação composta `@RequestScope` que atua como um atalho para a definição acima:

```java
@Bean
@RequestScope
public HelloMessageGenerator requestScopedBean() {
    return new HelloMessageGenerator();
}
```

Em seguida, podemos definir um `controller` que tenha uma referência injetada ao `requestScopedBean`. Precisamos acessar a mesma solicitação duas vezes para testar os escopos específicos da web.

Se exibirmos a mensagem sempre que a solicitação for executada, podemos ver que o valor é redefinido para *nulo*, mesmo que seja alterado posteriormente no método. Isso ocorre porque uma instância de *bean* diferente é retornada para cada solicitação.

```java
@Controller
public class ScopeController {
    @Resource(name = "requestScopedBean")
    HelloMessageGenerator requestScopedBean;

    @RequestMapping("/scopes/request")
    public String getRequestScopeMessage(final Model model) {
        model.addAttribute("previousMessage", requestScopedBean.getMessage());
        requestScopedBean.setMessage("Good morning!");
        model.addAttribute("currentMessage", requestScopedBean.getMessage());

        return "scopesExample";
    }
}
```

#### 4.2. Escopo de `session`

Podemos definir o *bean* com o escopo de `session` de maneira semelhante:

```java
@Bean
@Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)
public HelloMessageGenerator sessionScopedBean() {
    return new HelloMessageGenerator();
}
```

Há também uma *anotação composta* dedicada que podemos usar para simplificar a definição do *bean*:

```java
@Bean
@SessionScope
public HelloMessageGenerator sessionScopedBean() {
    return new HelloMessageGenerator();
}
```

Em seguida, definimos um `controller` com uma referência ao `sessionScopedBean`. *Novamente, precisamos executar duas requisições para mostrar que o valor do campo de mensagem é o mesmo para a sessão.*

Quando a solicitação é feita pela primeira vez, o valor `message` é *nulo*. *No entanto, uma vez alterado, esse valor é mantido para solicitações subsequentes, pois a mesma instância do **bean** é retornada durante toda a sessão.*

```java
@Controller
public class ScopesController {
    @Resource(name = "sessionScopedBean")
    HelloMessageGenerator sessionScopedBean;

    @RequestMapping("/scopes/session")
    public String getSessionScopeMessage(final Model model) {
        model.addAttribute("previousMessage", sessionScopedBean.getMessage());
        sessionScopedBean.setMessage("Good afternoon!");
        model.addAttribute("currentMessage", sessionScopedBean.getMessage());

        return "scopesExample";
    }
}
```

#### 4.3. Escopo de `application`

O escopo de `application` cria instância do *bean* para o ciclo de vida de um `ServletContext`.

Isso é semelhante ao escopo `singleton`, mas há uma *diferença muito importante* em relação ao escopo do *bean*.

Quando os *beans* têm o escopo `application`, *a mesma instância do **bean** é compartilhada entre vários aplicativos baseados em `servlet` em execução no mesmo `ServletContext`, enquanto os **beans** com escopo `singleton` têm escopo para apenas um único contexto do aplicativo.*

```java
@Bean
@Scope(value = WebApplicationContext.SCOPE_APPLICATION, proxyMode = ScopedProxyMode.TARGET_CLASS)
public HelloMessageGenerator applicationScopedBean() {
    return new HelloMessageGenerator();
}
```

Analogamente aos escopos de `request` e `session`, podemos usar uma versão mais curta:

```java
@Bean
@ApplicationScope
public HelloMessageGenerator applicationScopedBean() {
    return new HelloMessageGenerator();
}
```

Criamos um `controller` que faça referência a este *bean*:

```java
@Controller
public class ScopesController {
    @Resource(name = "applicationScopedBean")
    HelloMessageGenerator applicationScopedBean;

    @RequestMapping("/scopes/application")
    public String getApplitationScopeMessage(final Model model) {
        model.addAttribute("previousMessage", applicationScopedBean.getMessage());
        applicationScopedBean.setMessage("Good afternoon!");
        model.addAttribute("currentMessage", applicationScopedBean.getMessage());

        return "scopesExample";
    }
}
```

Uma vez definido no `applicationScopedBean`, o valor `message` será retido para todas as solicitações e sessões subsequentes e até mesmo para diferentes aplicativos `servlet` que acessarão este *bean*, desde que ele esteja sendo executado no mesmo `ServletContext`.

#### 4.4. Escope de `WebSocket`

Por fim, vamos criar o *bean* com o escopo `websocket`:

```java
@Bean
@Scope(scopeName = "websocket", proxyMode = ScopedProxyMode.TARGET_CLASS)
public HelloMessageGenerator websocketScopedBean() {
    return new HelloMessageGenerator();
}
```

No primeiro acesso, os *beans* com escopo `WebSocket` são armazenados nos atributos de sessão do `WebSocket`. A mesma instância do *bean* é retornada sempre que o *bean* é acessado durante toda a sessão do `WebSocket`.

### Fonte:

- Artigo: [Quick Guide to Spring Bean Scopes](https://www.baeldung.com/spring-bean-scopes)

## Escaneamento de componentes *Spring*

### 1. Visão Geral

Neste tutorial, abordaremos o *escanemaneto de componentes* do *Spring*. Ao trabalhar com o *Spring*, podemos anotar classes para transformá-las em *Spring beans*. Além disso, **podemos informar ao *Spring* onde procurar por essas classes anotadas**, já que nem todas precisam se tornar *beans* nesta execução específica.

### 2. `@ComponentScan` sem Argumentos

#### 2.1. Usando `@ComponentScan` em uma Aplicação *Spring*

Com o *Spring*, **usamos a anotação `@ComponentScan` junto com a anotação `@Configuration` para especificar os `packages` que queremos escanear.** `@ComponentScan` sem argumentos diz ao *Spring* para escanear o `package` atual e todos os seus `subpackages`.

Digamos que temos a seguinte `@Configuration` no *package* `com.bealdung.componentscan.springapp`:

```java
@Configuration
@ComponentScan
public class SpringComponentScanApp {
    private static ApplicationContext applicationContext;

    @Bean
    public ExampleBean exampleBean() {
        return new ExampleBean();
    }

    public static void main(String[] args) {
        applicationContext = new AnnotationConfigApplicationContext(SpringComponentScanApp.class);

        for (String beanName = applicationContext.getBeanDefinitionNames()) {
            System.out.println(beanName);
        }
    }
}
```

Além disso, temos os componentes `Cat` e `Dog` no *package* `com.bealdung.componentscan.springapp.animals`:

```java
package com.bealdung.componentscan.springapp.animals;

// ...
@Component
public class Cat {}
```

```java
package com.baeldung.componentscan.springapp.animals;

// ...
@Component
public class Dog {}
```

Por fim, temos o componente `Rose` no *package* `com.baeldung.componentscan.springapp.flowers`:

```java
package com.baeldung.componentscan.springapp.flowers;

// ...
@Component
public class Rose {}
```

A saída do método `main()` conterá todos os *beans* do *package* `com.baeldung.componentscan.springapp` e seus *subpackages*:

```
springComponentScanApp
cat
dog
rose
exampleBean
```

Observe que a classe principal do aplicativo também é um bean, pois é anotada com `@Configuration`, que é um `@Component`.

*Devemos também observar que a classe principal do aplicativo e a classe de configuração não são necessariamente as mesmas.* Se forem diferentes, não importa onde colocamos a classe principal do aplicativo. **Apenas a localização da classe de configuração importa, pois o escaneamento de componentes começa a partir do seu pacote (*package*) por padrão.**

Por fim, observe que em nosso exemplo, `@ComponentScan` é equivalente a:

```java
@ComponentScan(basePackages = "com.baeldiung.componentscan.springapp")
```

#### 2.2. Usando `@ComponentScan` em um aplicativo *Spring Boot*

O truque do *Spring Boot* é que muitas coisas acontecem implicitamente. Usamos a anotação `@SpringBootApplication`, *mas ela é uma combinação de três anotações*:

```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
```

Vamos criar uma estrutura semelhante no pacote `com.baeldung.componentscan.springapp`. Desta vez, a aplicação principal será:

```java
package com.baeldung.componentscan.springbootapp;

// ...
public class SpringBootComponentScanApp {
    private static ApplicationContext applicationContext;

    @Bean
    public ExampleBean exampleBean() {
        return new ExampleBean();
    }

    public static void main(String[] args) {
        applicationContext = SpringApplication.run(SpringBootComponentScanApp.class, args);

        checkBeansPresence(
            "cat", "dog", "rose", "exampleBean", "springBootComponentScanApp"
        );
    }

    private static void checkBeansPresence(String... beans) {
        for (String beanName : beans) {
            System.out.println(
                "Is " 
                + beanName
                + " in ApplicationContext: "
                + applicationContext.containsBeans(beanName)
            );
        }
    }
}
```

Todos os outros pacotes e classes permanecem os mesmos, vamos apenas copiá-los para o pacote `com.baeldung.componentscan.springbootapp` próximo.

O *Spring Boot* verifica os pacotes de forma semelhante ao nosso exemplo anterior.

```
Is cat in ApplicationContext: true
Is dog in ApplicationContext: true
Is rose in ApplicationContext: true
Is exampleBean in ApplicationContext: true
Is springBootComponentScanApp in ApplicationContext: true
```

O motivo pelo qual estamos apenas verificando a existência dos beans em nosso segundo exemplo (em vez de imprimir todos os beans) é que a saída seria muito grande.

Isso ocorre por causa da anotação implícita `@EnableAutoConfiguration`, que faz o *Spring Boot* criar muitos *beans* automaticamente, contando com as dependências do arquivo `pom.xml`.

### 3. `@ComponentScan` com Argumentos

Agora, vamos personalizar os caminhos para a varredura. Por exemplo, digamos que queremos excluir o *bean* `Rose`.

#### 3.1. `@ComponentScan` para Pacotes Específicos

Podemos fazer isso de algumas maneiras diferentes. Primeiro, podemos alterar o pacote base:

```java
@ComponentScan(basePackages = "com.baeldung.componentscan.springapp.animals")
@Configuration
public class SpringComponentScanApp {
    // ...
}
```

A saída será:

```springComponentScanApp
cat
dog
exampleBean
```

O que está por trás disso?

- `springComponentScanApp` é criado porque é uma configuração passada como um argumento para `AnnotationConfigApplicationContext`
- `exampleBean` é um bean configurado dentro da configuração
- `cat` e `dog` estão no pacote `com.baeldung.componentscan.springapp.animals` específicado

Todas as personalizações listadas acima também são aplicáveis ao *Spring Boot*. Podemos usar `@ComponentScan` junto com `@SpringBootApplication` e o resultado será o mesmo:

```java
@SpringBootApplication
@ComponentScan(basePackages = "com.baeldung.componentscan.springbootapp.animals")
```

#### 3.2. `@ComponentScan` com Vários Pacotes

O *Spring* oferece uma maneira conveniente de especificar vários nomes de pacotes. Para isso, precisamos usar um `array` de `strings`.

Cada sequência do `array` denota um nome de *pacote*:

```java
@ComponetnScan(basePackages = {"com.baeldung.compontentscan.apringapp.animals", "com.baeldung.componentscan.springapp.flowers"})
```

Alternativamente, *desde o **Spring 4.1.1***, *podemos usar uma vírgula, um ponto e vírgula ou um espaço para separar a lista de pacotes*:

```java
@ComponentScan(basePackages = "com.baeldung.componentscan.springapp.animals;com.baeldung.componentscan.springapp.flowers")
@ComponentScan(basePackages = "com.baeldung.componentscan.springapp.animals,com.baeldung.componentscan.springapp.flowers")
@ComponentScan(basePackages = "com.baeldung.componentscan.springapp.animals com.baeldung.componentscan.springapp.flowers")
```

#### 3.3. `@ComponentScan` com Exclusões

Outra maneira é usar um filtro especificando o padrão para as classes excluídas:

```java
@ComponentScan(
    excludeFilters =
        @ComponentScan.Filter(type=FilterType.REGEX,
        pattern="com\\.baeldung\\.componentscan\\.springapp\\.flowers\\..*"
    )
)
```

Também podemos escolher um tipo de filtro diferente, pois *a anotação suporta várias opções flexíveis para filtrar as classes escaneadas*:

```java
@ComponentScan(
    excludeFilters =
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, value = Rose.class)
)
```

### 4. O Pacote Padrão

Devemos evitar colocar a classe `@Configuration` no *pacote padrão* (ou seja, não especificando o pacote). Se fizermos isso, *o **Spring** verificará todas as classes em todos os `jars` de um `classpath`, o que causa erros e o aplicativo provavelmente não iniciará*.

### Fonte:

- Artigo: (Spring Component Scanning)[https://www.baeldung.com/spring-component-scanning]

## *Injeção de Dependências* de Construtor no *Spring Boot*

### 1. Introdução

Provavelmente um dos princípios do desenvolvimento mais importantes do design de software moderno é a *Injeção de Dependêcnia (**DI**)*, que naturalmente decorre de outro princípio extremamente importante: a *Modularidade*.

Este tutorial rápido explorará um tipo específico de técnica de *DI* no *Spring Boot* chamada de *Injeção de Dependência Baseada em Construtor*, que, em termos simples, significa que passamos os componentes necessários para uma classe no momento de instanciação.

Para começar, precisamos importar a dependência `spring-boot-starter-web` em nosso `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Em seguida, precisamos configurar um arquivo de configuração. Este arquivo pode ser um arquivo `POJO` ou `XML`, conforme a preferência.

### 2. Configuração Baseada em Anotação

Os arquivos de configuração *Java* são semelhantes aos objetos *Java* com algumas anotações adicionais:

```java
@Configuration
@ComponentScan("com.baeldung.constructordi")
public class Config {
    @Bean
    public Engine engine() {
        return new Engine("v8", 5);
    }

    @Bean
    public Transmission transmission() {
        return new Transmission("sliding");
    }
}
```

Aqui, estamos usando anotações para notificar o tempo de execução do *Spring* de que esta classe fornece *definições de bean* (anotação `@Bean`) e que o pacote (`package`) `com.baeldung.spring` precisa executar um escaneamento de contexto em busca de *beans* adicionais. Em seguida, definimos uma classe `Car`:

```java
@Component
public class Car {
    @Autowired
    public Car(Engine engine, Transmission transmission) {
        this.engine = engine;
        this.transmission = transmission;
    }
}
```

O *Spring* encontrará nossa classe `Car` ao fazer um escaneamento de *package* e inicializará sua instância chamando o construtor anotado `@Autowired`.

Chamando os métodos anotados `@Bean` da classe `Config`, obteremos instâncias de `Engine` e `Transmission`. Por fim, precisamos inicializar um `ApplicationContext` usando nossa configuração `POJO`.

```java
ApplicationContext context = new AnnotationConfigApplicationCOntext(Config.class);
Car car = context.getBean(Car.class);
```

### 3. Injeção de Construtor Implícito

*Classes com um único construtor podem omitir a anotação `@Autowired`.* Isso é uma pequena vantagem prática e uma remoção de clichês.

Além disso, a *partir da versão **4.3***, *podemos aproveitar a injeção baseada em construtor em classes anotadas com `@Configuration`*. Além disso, *se uma classe tiver apenas um construtor, também podemos omitir a anotação `@Autowired`*.

### 4. Prós e Contras

*A injeção de construtor tem algumas vantagens em comparação à injeção de campo.*

**O primeiro benefício é a testabilidade.** Suponha que vamos testar a unidade de um *bean* *Spring* que usa injeção de campo:

```java
public class UserService {
    @Autowired
    private UserRepository userRepository;
}
```

Durante a construção de uma instância de `UserService`, não podemos inicializar o estado `userRepository`. A única maneira de fazer isso é por meio da **API de Reflexão**, *que quebra* completamente o encapsulamento. Além disso, o código resultante será menos seguro em comparação com uma simples chamada de construtor.

Além disso, **com a injeção de campo, não podemos impor invariantes em nível de classe**, *então é possível* ter uma instância de `UserService` sem um `userRepository` inicializado corretamente. Portanto, podemos encontrar `NullPointerExceptions` aleatórias aqui e ali. Além disso, com a injeção de construtor, é mais fácil construir componentes imutáveis.

Além disso, *usar construtores para criar instâncias de objetos é mais natural do ponto de vista da **POO***.

Por outro lado, a principal desvantagem da *injeção de construtor* é sua verbosidade, especialmente quando um *bean* possui poucas dependências. Ás vezes, pode ser uma *bênção disfarçada*, pois podemos nos esforçar mais para manter o número de dependências mínimo.

### Fonte:

- Artigo: [Constructor Dependency Injection in Spring](https://www.baeldung.com/constructor-injection-in-spring)