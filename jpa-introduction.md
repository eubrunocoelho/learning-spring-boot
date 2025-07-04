# Introdução ao *Spring Data JPA*

## 1. Visão Geral

Este artigo se concentrará na *introdução do **Spring Data JPA** em um projeto **Spring*** e na configuração completa da camada de persistência.

## 2. O `DAO` Gerado por dados da *Spring* - Chega de Implementações de `DAO`

Como discutimos em um artigo anterior, [*a camada `DAO`*](https://www.baeldung.com/simplifying-the-data-access-layer-with-spring-and-java-generics) geralmente consiste em muito código `boilerplate` que pode e deve ser simplificado. As vantagens dessa simplificação são muitas: *redução do número de artefatos que precisamos definir e manter, consistência dos padrões de acesso a dados e consistência da configuração.*

O *Spring Data* leva essa simplificação um passo adiante e ***possibilita a remoção completa das implementações do `DAO`***. A *interface do `DAO`* agora é o único artefato que precisamos definir explicitamente.

Para começar a aproveitar o modelo de programação do *Spring Data com `JPA`*, uma *interface `DAO`* precisa estender a *interface `Repository`* específica do `JPA`, **`JpaRepository`**. Isso permitirá que o *Spring Data* encontre essa interface e crie automaticamente uma implementação para ela.

Ao estender a interface, obtemos os *métodos `CRUD`* mais relevantes para acesso a dados padrão disponíveis em um `DAO` padrão.

## 3. Método de Acesso e Consultas Personalizados

Conforme discutido, **ao implementar uma das interfaces do *Repositório*, o `DAO` já terá alguuns métodos `CRUD` básicos *(e consultas)* definidos e implementados**.

Para definir métodos de acesso mais específico, o *Spring JPA* suporta algumas opções:

- basta *definir um novo método* na interface.
- fornça a *consulta `JPQL`* real usando a anotação `@Query`.
- use o *suporte mais avançado de especificação e `Querydsl`* no *Spring Data*.
- definir *consultas personalizadas* por meio de consultas nomeadas `JPA`.

A [*terceira opção*](http://spring.io/blog/2011/04/26/advanced-spring-data-jpa-specifications-and-querydsl/), Especificações e suporta a `QueryDSL`, é semelhante aos *Critérios JPA*, mas utiliza *uma `API` mais flexível e conveniente*. Isso torna toda a operação muito mais legível e reutilizável. As vantagens dessa `API` se tornarão mais evidentes ao lidar com um *grande número de consultas fixas*, pois poderíamos expressá-las de forma mais concisa por meio de um número menor de blocos reutilizáveis.

A última opção tem a desvantagem de envolver `XML` ou sobrecarregar a classe de domínio com as consultas.

### 3.1. Consultas Personalizadas Automáticas

Quando o *Spring Data* cria uma nova implementação de *Repositório*, ele analisa todos os métodos definidos pelas interfaces e tenta **gerar consultas automaticamente a partir dos nomes dos métodos**. Embora isso tenha algumas limitações, é uma meneira muito poderosa e elegante de definir novos métodos de acesso personalizados com pouquíssimo esforço.

Vejamos um exemplo. Se a entidade tiver um campo de nome *(e os métodos `getName` e `setName` padrão do `Java Bean`)*, **definiremos o método `findByName` na interface `DAO`**. Isso gerará automaticamente a consulta correta:

```java
public interface IFooDAO extends JpaRepository<Foo, Long> {
    Foo findByName(String name);
}
```

Este é um exemplo relativamente simples. O mecanismo de criação de consultas suporta [um conjunto muito maior de palavras-chave](https://docs.spring.io/spring-data/jpa/reference/repositories/query-methods-details.html#repositories.query-methods.query-creation).

Caso o analisador não consiga corresponder a propriedade com o campo do objeto de domínio, veremos a seguinte exceção:

```
java.lang.IllegalArgumentException: No property nam found for type class com.baeldung.jpa.simple.model.Foo
```

### 3.2. Consultas Personalizadas Manuais

Agora, vamos analisar uma consulta personalizada que definiremos por meio da anotação `@Query`:

```java
@Query("SELECT f FROM Foo f WHERE LOWER(f.name) = LOWER(:name)")
Foo retrieveByName(@Param("name") String name);
```

Para um controle ainda mais preciso sobre a criação de consultas, com usar parâmetros nomeados ou modificar consultas existentes, [a referência](https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html#jpa.named-parameters) é um bom lugar para começar.

## 4. Configuração da Transação

A implementação real do `DAO` gerenciado pelo *Spring* está oculta, pois não trabalhamos com ele diretamente. No entanto, é uma implementação bastante simples, **o `SimpleJpaRepository` que define a semântica da transação usando anotações**.

*Mais explicitamente, isso usa uma anotação `@Transactional`* somente leitura no nível da classe, que é então substituída pelos métodos que não são somente leitura. O restante da semântica da transação é padrão, mas pode ser facilmente substituída manualmente por método.

### 4.1. A Tradução de Exceçoes está Viva e Bem

A questão agora é: como o *Spring Data JPA* não depende dos antigos modelos `ORM` *(`JpaTemplate`, `HibernateTemplate`)* e eles foram removidos desde o **Spring 5**, ainda teremos nossas exceções `JPA` traduzidas para a hierarquia `DataAccessException` do *Spring*?

A reposta é, claro, sim. **A tradução de exceções ainda é habilitada peço uso da anotação `@Repository` no `DAO`.** Essa anotação permite que um pós-processador do *Spring Bean* informe todos os `beans` `@Repository` com todas as instâncias de `PersistenceExceptionTranslator` encontradas no contêiner e forneça a tradução de exceções como antes.

Vamos verificar a tradução de exceções com um teste de integração:

```java
@Test(expected = DataIntegrityViolationException.class)
public void whenInvalidEntityIsCreated_thenDataException() {
    service.create(new Foo());
}
```

Lembre-se de que **a tradução de exceções é feita por meio de `proxies`**. Para que o *Spring* possa criar `proxies` em torno das classes `DAO`, estas não devem ser declaradas `final`.

## 5. Configuração do Repositório `JPA` do *Spring Data*

Para ativar o suporte ao repositório *Spring JPA*, podemos usar a anotação `@EnableJpaRepositories` e especificar o pacote que contém as *interfaces `DAO`*:

```java
@EnableJpaRepositories(basePackages = "com.baeldung.jpa.simple.repository")
public class PersistenceConfig {
    // ...
}
```

Podemos fazer o mesmo com uma configuração `XML`:

```xml
<jpa:repositories base-package="com.baeldung.jpa.simple.repository" />
```

## 6. Configuração *Java* ou `XML`

O *Spring Data* também aproveita o suporte do *Spring* à anotação `@PersistenceContext` do `JPA`. Ele a utiliza para conectar o `EntityManger` ao `bean factory` do *Spring* responsável por criar as implementações `DAO` reais, o `JpaRepositoryFactoryBean`.

Além da configuração já discutida, também precisamos incluir o *Spring Data XML Config* se estivermos usando `XML`:

```java
@Configuration
@EnableTransactionManagement
@ImportResource("classpath*:*springDataConfig.xml")
public class PersistenceJPAConfig {
    // ...
}
```

## 7. Dependência do *Maven*

Além da configuração do *Maven* para `JPA`, adicionaremos a dependência `spring-data-jpa`:

```xml
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-jpa</artifactId>
</dependency>
```

## 8. Usando *Spring Boot*

**Também podemos usar a dependência *JPA* do *Spring Boot Starter Data*, que configurará automaticamente o `DataSource` para nós.**

Precisamos garantir que o banco de dados que queremos usar esteja presente no `classpath`. No nosso exemplo, adicionamos o banco de dados `H2` na memória:

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```

Como resultado, apenas realizando essas dependências, nosso aplicativo estará funcionando e poderemos usá-lo para outras operações de banco de dados.

**A configuração explícita para um aplicativo *Spring* padrão agora está incluída como parte da configuração automática do *Spring Boot*.**

Podemos, é claro, modificar a autoconfiguração adicionando nossa configuração explícita personalizada.

*O <ins>Spring Boot</ins> oferece uma maneira fácil de fazer isso usando propriedades no arquivo `application.properties`.* Vejamos um exemplo de alteração da `URL` de conexão e das credenciais:

```
spring.datasource.url=jdbc:h2:mem:db;DB_CLOSE_DELAY=-1
spring.datasource.username=sa
spring.datasource.password=sa
```

## 9. Ferramentas Úteis para *Spring Data JPA*

O *Spring Data JPA* é suportado por todos os principais `IDEs` *Java*. Vamos ver quais ferramentas úteis estão disponíveis para *Eclipse* e *IntelliJ IDEA*.

**Se você usa o *Eclipse* como `IDE`, pode instalar o *plugin* *Dali Java Persistence Tools*.** Ele fornece diagramas `ER` para entidades `JPA`, geração de `DDL` para inicializar esquemas e recursos básicos de *engenharia reversa*. Você também pode usar o *Eclipse Spring Tools Suite <ins>(STS)</ins>*. Ele ajudará a validar nomes de métodos de consulta em repositórios `JPA` do *Spring Data*.

Caso você use *IntelliJ IDEA*, há duas opções.

O *IntelliJ IDEA Ultimate* permite diagramas `ER`, um console `JPA` para testar instruções `JPQL` e inspeções valiosas. No entanto, esses recursos não estão disponíveis na *Community Edition*.

**Para aumentar a produtividade no *IntelliJ*, você pode instalar o plugin *JPA Buddy***, que fornece muitos recursos, incluindo geração de *entidades `JPA`*, *repositórios Spring Data `JPA`*, *`DTOs`*, *scripts `DDL` de inicialização*, *migrações versionadas do <ins>Flyway</ins>*, &, etc. Além disso, o *JPA Buddy* fornece uma ferramenta avançada para engenharia reversa.

Por fim, o plugin *JPA Buddy* funciona com as edições *Community* e *Ultimate*.

## 10. Conclusão

Neste artigo, abordamos a configuração e a implementação da camada de persistência com **Spring 5**, *JPA 2* e *Spring Data JPA* usando configurações baseada em `XML` e *Java*.

Discutimos maneiras de definir *consultas personalizadas mais avançadas*, bem como *semântica transacional* e uma *configuração com o novo `namespace` `JPA`*. O resultado final é uma abordagem nova e elegante ao acesso a dados com *Spring*, com quase nenhum trabalho de implementação.

## Fonte:

- Artigo: [Introduction to Spring Data JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)