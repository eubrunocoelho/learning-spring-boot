# Guia para `@ConfigurationProperties` no *Spring Boot*

## 1. Introdução

A anotação `@ConfigurationProperties` em *Spring Boot* é **uma ferramenta crucial para vincular configurações externas a um objeto Java**.

## 2. Configuração

Este tutorial usa uma configuração bastante padrão. Começamos adicionando `spring-boot-starter-parent` *como parent* em nosso `pom.xml`:

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.1.5</version>
    <relativePath/>
</parent>
```

Para poder *validar as propriedades* definidas no arquivo, também precisamos de uma implementação do [JSR-380](https://beanvalidation.org/2.0-jsr380/), e o *hibernate-validator* é uma delas, fornecida pela dependência *spring-boot-starter-validation*.

*Adicionamos ela ao nosso `pom.xml`*:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

## 3. Propriedades Simples

**A documentação ofical recomenda isolar as propriedades de configuração em *POJOs separados*.**

```java
@Configuration
@ConfigurationProperties(prefix = "mail")
public class ConfigProperties {
    private String hostName;
    private int port;
    private String from;

    // standard getters and setters
}
```

Usamos `@Configuration` para que o *Spring* crie um Spring `bean` no contexto do aplicativo.

**`@ConfigurationProperties` funciona melhor com propriedades hierárquicas que têm o mesmo prefixo;** portanto, adicionamos o *prefixo `mail`*.

O *framework* usa `setters` Java `bean` padrão, então devemos declarar setters para cada uma das propriedades.

*Observação: se não usarmos `@Configuration` no **POJO**, precisamos adicionar `@EnableConfigurationProperties(ConfigProperties.class)` na classe principal do aplicativo Spring para vincular as **propriedades ao POJO***

```java
@SpringBootApplication
@EnableConfigurationProperties(ConfigProperties.class)
public class EnableConfigurationDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(EnableConfigurationDemoApplication.class, args);
    }
}
```

**O *Spring* vinculará automaticamente qualquer propriedade definida em nosso arquivo de propriedades que tenha o prefixo `mail` e o mesmo nome de um dos campos da classe `ConfigProperties`.**

O *Spring* utiliza algumas regras mais flexíveis para vincular propriedades. Como resultado, as seguintes variações são todas vinculadas à propriedade `hostName`:

```
mail.hostName
mail.hostname
mail.host_name
mail.host-name
mail.HOST_NAME
```

*Portanto, podemos usar o seguinte arquivo de propriedades para definir todos os campos:*

```
#Simple Properties
mail.hostname=host@mail.com
mail.port=9000
mail.from=mailer@mail.com
```

### 3.1. Spring Boot 2.2

**A partir do *Spring Boot 2.2*, o *Spring* encontra e registra classes `@ConfigurationProperties` por meio de *scanning de classpath*.** O *scanning* de `@ConfigurationProperties` precisa ser explicitamente ativada adicionando a anotação `@ConfigurationPropertiesScan`. Portanto, *não precisamos anotar essas classes com `@Component` (e outras meta-anotações como `@Configuration`), **nem mesmo usar `@EnableConfigurationProperties`**:*

```java
@ConfigurationProperties(prefix = "mail")
@ConfigurationPropertiesScan
public class ConfigProperties {
    private String hostName;
    private int port;
    private String from;

    // standard getters and setters
}
```

O *scanner* de *classpath* habilitado por `@SpringBootApplication` encontra a classe `ConfigProperties`, mesmo que não tenhamos anotado essa classe com `@Component`.

Além disso, podemos usar **a anotação `@ConfigurationPropertiesScan` para escanear locais personalizados para classes de propriedades de configuração:**

```java
@SpringBootApplication
@ConfigPropertiesScan("com.bealdung.configurationproperties")
public class EnableConfigurationDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(EnableConfigurationDemoApplication.class, args);
    }
}
```

Dessa forma, o *Spring* procurará classes de propriedades de configuração apenas no *pack* `com.baeldung.properties`.

## 4. Propriedades Aninhadas

*Podemos ter propriedades aninhadas em Lista, Mapas e Classes.*

Vamos criar uma nova classe `Credentials` para usar em algumas propriedades aninhadas:

```java
public class Credentials {
    private String authMethod;
    private String username;
    private String password;

    // standard getters and setters
}
```

Precisamos atualizar a classe `ConfigProperties` para usar uma *Lista, um Mapa e a classe `Credentials`*.

```java
public class ConfigProperties {
    private String hostname;
    private int port;
    private String from;
    private List<String> defaultRecipients;
    private Map<String, String> additionalHeaders;
    private Credentials credentials;

    // standard getters and setters
}
```

*O seguinte arquivo de propriedades definirá todos os campos:*

```
#Simple Properties
mail.hostname=mailter@mail.com
mail.port=9000
mail.from=mailter@mail.com

#List Properties
mail.defaultRecipients[0]=admin@mail.com
mail.defaultRecipients[1]=owner@mail.com

#Map Properties
mail.additionalHeaders.redelivery=true
mail.additionalHeaders.secure=true

#Object Properties
mail.credentials.username=john
mail.credentials.password=123456
mail.credentials.authMethod=SHA1
```

## 5. Usando `@ConfigurationProperties` em um Método `@Bean`

**Também podemos usar a anotação `@ConfigurationProperties` em métodos anotados com `@Bean`.**

*Essa abordagem pode ser particularmente útil quando queremos vincular propriedaeds a um componente de terceiros que está fora do nosso controle.*

Vamos criar uma classe `Item` que usaremos no próximo exemplo:

```java
public class Item {
    private String name;
    private int size;

    // standard getters and setters
}
```

Agora vamos ver como podemos usar `@ConfigurationProperties` em um método `@Bean` para vincular propriedades externalizadas *à instância `Item`*:

```java
@Configuration
public class ConfigProperties {
    @Bean
    @ConfigurationProperties(prefix = "item")
    public Item item() {
        return new Item();
    }
}
```

*Consequentemente, **qualquer propriedade prefixada por item será mapeada para a instância `Item`** gerenciada pelo contexto Spring*.

## 6. Validação de Propriedade

**`@ConfigurationProperties` fornece validação de propriedades usando o formato [JSR-380](https://beanvalidation.org/2.0-jsr380/). Isso permite todo tipo de coisa interessante.**

*Por exemplo, vamos tornar a propriedade `hostName` obrigatória:*

```java
@NotBlank
private String hostName;
```

*Em seguida, vamos fazer com que a propriedade `authMethod` tenha entre 1 a 4 caracteres:*

```java
@Length(max = 4, min = 1)
private String authMethod;
```

*Então a propriedade `port` de 1025 a 65536:*

```java
@Min(1025)
@Max(65536)
private int port;
```

*Por fim, a propriedade `from` deve corresponder a um formato de endereço de e-mail:*

```java
@Pattern(regexp = "^[a-z0-9._%+-]+@[a-z0-9.-]+\\.[a-z]{2,6}$")
private String from;
```

*Isso nos ajuda a reduzir muitas condições `if - else` em nosso código e faz com que ele pareça mais limpo e conciso.*

**Se qualquer uma dessas validações *falhar*, o aplicativo principal não iniciará com uma `IllegalStateException`.**

*O framework de validação do **Hibernate** Usa getters e setters padrão do **Java Bean**, por isso é importante declarar getters e setters para cada uma das propriedades.*

## 7. Conversão de Propriedades

*`@ConfigurationProperties` suporta conversão para vários tipos de vinculação de propriedades aos seus `beans` correspondentes.*

### 7.1. Duração

Começaremos analisando a *conversão de propriedades* em objetos *Duration*.

Temos dois campos do tipo *Duration*:

```java
@ConfigurationProperties(prefix = "conversion")
public class PropertyConversion {
    private Duration timeInDefaultUnit;
    private Duration timeInNano;

    // ...
}
```

Esse é o nosso arquivo de propriedades:

```
conversion.timeInDefaultUnit=10
conversion.timeInNano=9ns
```

Como resultado, o campo `timeInDefaultUnit` terá um *valor de 10 milissegundos*, e `timeInNano` tera um *valor de 9 nanossegundos*.

**As unidades suportadas são *ns, us, ms, s, m, h e d* para *nanossegundos, microssegundos, milissegundos, segundos, minutos, horas e dias*, respectivamente.**

*A unidade padrão é milissegundos, o que significa que se não especificarmos uma unidade ao lado do valor numérico, o Spring converterá o valor para milissegundos.*

Também podemos substituir a unidade padrão usando `@DefaultUnit`:

```java
@DurationUnit(ChronoUnit.DAYS)
private Duration timeInDays;
```

Correspondendo:

```java
conversion.timeInDays=2
```

### 7.2. Tamanho dos Dados

*Da mesma forma, o **Spring Boot** `@ConfigurationProperties` suporta a conversção de tipo `DataSize`.*

Vamos adicionar três campos do tipo `DataSize`:

```java
private DataSize sizeInDefaultunit;

private DataSize sizeInGB;

@DataSizeUnit(DataUnit.TERABYTES)
private DataSize sizeInTB;
```

Correspondendo:

```java
conversion.sizeInDefaultUnit=300
conversion.sizeInGB=2GB
conservion.sizeInTB=4
```

**Nesse caso, *o valor de `sizeInDefaultUnit` será 300 bytes*, pois a *unidade padrão é bytes*.**

*As unidades suportadas são `B, KB, MB, GB e TB`.* Também podemos substituir a unidade padrão usando `@DataSizeUnit`.

### 7.3. Conversor Personalizado

Também *podemos adicionar nosso próprio conversor personalizado* para dar suporta à conversão de uma propriedade em um tipo de classe específico.

Vamos adicionar uma classe `Employee`:

```java
public class Employee {
    private String name;
    private double salary;
}
```

*Em seguida, criaremos um conversor personalizado para converter esta propriedade:*

```
conversion.employee=john,2000
```

Vamos convertê-lo em um arquivo do tipo `Employee`:

```java
private Employee employee;
```

Precisaremos implementar a *interface `Converter`* e, em seguida, **usar a *anotação* `@ConfigurationPropertiesBinding` para registrar nosso `Converter`** personalizado.

```java
@Component
@ConfigurationPropertiesBinding
public class EmployeeConverter implements Converter<String, Employee> {
    @Overrite
    public Employee convert(String, from) {
        String[] data = from.split(",");
        return new Employee(data[0], Double.parseDouble(data[1]));
    }
}
```

## 8. Vinculação Imutável `@ConfigurationProperties`

A partir do **Spring Boot 2.2**, *podemos usar a **anotação `@ConstructorBinding`** para vincular nossas propriedades de condfiguração*, em vez da *antiga injeção de setter*.

*Isso significa essencialmente que as classes anotadas com `@ConfigurationProperties` agora podem ser imutáveis.*

No **Spring Boot 3**, se houver *um único construtor parametrizado*, *a vinculação do construtor é implícita* e *não precisamos usar a anotação*. Mas, no caso de vários construtores, precisamos *anotar um preferido*.

```java
@ConfigurationProperties(prefix = "mail.credentials")
public class ImmutableCredentials {
    private final String authMethod;
    private final String username;
    private final String password;

    @ConstructorBinding
    public ImmutableCredentials(String authMethod, String username, String password) {
        this.authMethod = authMethod;
        this.username = username;
        this.password = password;
    }

    public ImmutableCredentials(String username, String password) {
        this.username = username;
        this.password = password;
        this.authMethod = "Default";
    }

    public String getAuthMethod() {
        return authMethod;
    }

    public String getUsername() {
        return username;
    }

    public String getPassword() {
        return password;
    }
}
```

Ao usar `@ConstructorBinding`, *precisamos fornecer ao construtor todos os parâmetros que gostaríamos de vincular*.

Observe que todos os campos de `ImmutableCredentials` **são finais**. *Além disso, não há métodos setter.*

É *importante enfatizar* que para usar a vinculação do construtor, *precisamos habilitar explicitamente nossa classe de configuração com `@EnableConfigurationProperties` ou com `@ConfigurationPropertiesScan`*.

## 9. `Records` *Java 16*

O *Java 16* introduziu os tipos de `record` como parte da [JEP 365](https://openjdk.org/jeps/395). `Records` *são classes que atuam como portadores transparentes de dados imutáveis*. Isso os torna candidatos perfeitos para *detentores de configuração* e *DTOs*. **Aliás, podemos definir `records` *Java* como propriedades de configuração no *Spring Boot*.** Por exemplo, o exemplo anterior pode ser reescrito como:

```java
@ConstructorBinding
@ConfigurationProperties(prefix = "mail.credentials")
public record ImmutableCredentials(String authMethod, String username, String password) {
}
```

Obviamente, é mais conciso em comparação a todos aqueles *getters e setters barulhentos*.

Além disso, a partir do **Spring Boot 2.6**, **para registros com um único construtor, podemos remover a anotação `@ConstructorBinding`**. Se o nosso registro tiver vários construtores, no entando, *`@ConstructorBinding` ainda deverá ser usado para identificar o construtor a ser usado para a vinculação de propriedades*.

## Fonte

- Artigo: [Guide to @ConfigurationProperties in Spring Boot](https://www.baeldung.com/configuration-properties-in-spring-boot)