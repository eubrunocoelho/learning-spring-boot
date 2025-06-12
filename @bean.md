# O que é o `Bean` no Spring Boot?

## 1. Visão Geral

`Bean` é um conceito-chave do *Framework Spring*. Portanto, entender essa noção é crucial para entender o framework e usá-lo de forma eficaz.

## 2. Definição de `Bean`

*No **Spring**, os objetos que formam a espinha dorsal da sua aplicação e que são gerenciados pelo contêiner **Spring IoC** são chamados de beans. Um bean é um objeto que é instanciado, montado e gerenciado por um contêiner **Spring IoC**.*

Esta definição é concisa e vai direto ao ponto, **mas não aborda um elemento importante: o contêiner Spring IoC**. Vamos dar uma olhada mais de perto para entender o que ele é e quais são os benefícios que ele traz.

## 3. Inversão de Controle

Simplificando, **Inversão de Controle (IoC)** *é um processo no qual um objeto define suas dependências sem criá-las*. Esse objeto delega a tarefa de construir tais dependências a um contêiner de *IoC*.

### 3.1. Classe de Domínio

Suponha que temos uma declaração de classe:

```java
public class Company {
    private Address address;

    public Company(Address address) {
        this.address = address;
    }

    // getter, setter and other properties
}
```

Esta classe precisa de um colaborador do tipo *Address*:

```java
public class Address {
    private String street;
    private int number;

    public Address(String street, int number) {
        this.street = street;
        this.number = number;
    }

    // getters and setters
}
```

### 3.2. Abordagem Tradicional

Normalmente, criamos objetos com os construtores de suas classes:

```java
Address address = new Address("Rua XV de Novembro", 666);
Company company = new Company(address);
```

Não há nada de errado com essa abordagem, mas não seria bom gerenciar as dependências de uma maneira melhor?

Imagine um aplicativo com dezenas ou até centenas de classes. Às vezes, queremos compartilhar uma única instância de uma classe em todo o aplicativo, outras vezes precisamos de um objeto separado para cada caso de uso, e assim por diante.

Gerenciar tantos objetos é um verdadeiro pesadelo. **É aí que a inversão de controle entra em ação.**

Em vez de construir dependências por si só, um objeto pode recuperá-las de um contêiner IoC. **Tudo o que precisamos fazer é fornecer ao contêiner os *metadados de configuração apropriados*.**

### 3.3. Configuração do `Bean`

Primeiro, vamos decorar a classe `Company` com a anotação `@Component`:

```java
@Component
public class Company {
    // this body is the same as before
}
```

Aqui está uma classe de configuração que fornece metadados de *bean* para um *contêiner IoC*:

```java
@Configuration
@ComponentScan(basePackageClasses = Company.class)
public class Config {
    @Bean
    public Address getAddress() {
        return new Address("Rua XV de Novembro", 666);
    }
}
```

A classe de configuração produz em *bean* do tipo `Address`. Ela também carrega a anotação `@ComponentScan`, que instrui o contêkiner a procurar *beans* no `package` que contém a classe `Company`.

**Quando um contêiner *Spring IoC* contrói objetos desses tipos, todos os objetos são chamados de *Spring Beans*, pois são gerenciados pelo contêiner *Ioc*.**

### 3.4. *IoC* em Ação

Como definimos *beans* em uma classe de configuração, *precisamos de uma instância da classe `AnnotationConfigApplicationContext` para construir um contêiner*:

```java
ApplicationContext context = new AnnotationConfigApplicationContext(Config.class);
```

Um teste rápido  verifica a *existência e os valores* de propriedades dos nossos *beans*:

```java
Company company = context.getBean("company", Company.class);
assertEquals("Rua XV de Novembro", company.getAddress().getStreet());
assertEquals(666, company.getAddress().getNumber());
```

*O resultado prova que o contêiner **IoC** criou e inicializou os beans corretamente.*

## Fonte:

- Artigo: [What Is a Spring Bean](https://www.baeldung.com/spring-bean)