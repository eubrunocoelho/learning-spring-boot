# `@Component` vs `@Repository` e `@Service` no *Spring*

## 1. Introdução

Neste artigo, aprenderemos sobre as diferençãs entre as anotações `@Component`, `@Repository` e `@Service` no *Spring Framework*.

## 2. Anotações do *Spring*

Na maioria das aplicações típicas, temos camadas distintas, como acesso a dados, apresentação, serviço, negócios, etc.

Além disso, em cada camada, temos vários `beans`. Para detectar esses `beans` automaticamente, **o *Spring* usa anotações de escaneamento de `classpath`**.

Em seguida, ele registra cada `bean` no `ApplicationContext`.

Aqui está uma rápida visão geral de algumas dessas anotações:

- `@Component` é um estereótipo genérico para qualquer componente gerenciado pelo *Spring*.
- `@Service` anota as classes na camada de serviço.
- `@Repository` anota classes na camada de persistência, que atuará como um repositório de banco de dados.

## 3. O que é Diferente?

**A principal diferenã entre esses estereótipos é que eles são usados para classificações diferentes.** Ao anotar uma classe para *autodetecção*, devemos usar o estereótipo correspondente.

### 3.1. `@Compomnent`

**Podemos usar `@Component` em toda a aplicação para marcar os `beans` como componentes gerenciados do *Spring*.** O *Spring* só selecionará e registrará `beans` com `@Component`, e não procurará por `@Service` e `@Repository` em geral.

*Eles são registrados no `ApplicationContext` porque são anotados com `@Component`:*

```java
@Component
public @interface Service {
}
```

```java
@Component
public @interface Repository {
}
```

`@Service` e `@Repository` são casos especiais de `@Component`. Tecnicamente, são a mesma coisa, mas usamos cada um para propósitos diferentes.

### 3.2. `@Repository`

**O trabalho do `@Repository` é capturar exceções específicas de persistêmcia e relançá-las como uma das exceções unificadas não verificadas do *Spring*.**

Para isso, o *Spring* fornece `PersistenceExceptionTranslationPostProcessor`, que precisamos adicionar no contexto do nosso aplicativo *(já incluído se estivermos usando o **Spring Boot**)*:

```xml
<bean class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor" />
```

Este *pós-processador de `bean`* adiciona um consultor a qualquer `bean` anotado com `@Repository`.

### 3.3. `@Service`

**Marcamos `beans` com `@Service` para indicar que eles contêm a lógica de negócios.** Além de ser usado na camada de serviço, não há nenhum outro uso especial para essa anotação.

## Fonte:

- Artigo: [@Component vs @Repository and @Service](https://www.baeldung.com/spring-component-repository-service)