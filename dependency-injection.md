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

Ao criar uma instância de `AnnotationConfigApplicationContext` e fornecer a ela uma ou mais classes de configuração, ela verifica essas classes em busca de anotações `@Bean` e outras anotações relevantes. Em seguida, *ele inicializa e **gerencia os beans** definidos nessas classes, configurando suas dependências e gerenciado seu ciclo de vida.*

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

- ***no**:** O valor padrão - isso significa que nenhuma conexão automática é usada para o *bean* e temos que nomear explicitamente as dependências.
- ***byName**:** A conexão automática é feita com base no nome da propriedade, portanto, o *Spring Boot* procurará um *bean* com o mesmo nome da propriedade que precisa ser definida.
- ***byType**:** Semlhante à *autowiring ByName*, mas com base no tipo da propriedade. Isso significa que o *Spring Boot* procurará um *bean* com o mesmo tipo da propriedade para definir. Se houver *mais de um bean* desse tipo, o framework *lançará uma exceção*.
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