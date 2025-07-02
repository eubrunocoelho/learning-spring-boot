# Introdução ao Projeto *Lombok*

## 1. Visão Geral

*Java* é uma ótima linguagem, mas às vezes pode se tornar muito prolixa para tarefas comuns que preccisamos realizar em nosso código ou para atender a algumas *práticas de framework*. Isso muitas vezes não agrega valor real ao lado comercial dos nossos programas, e é ai que o *Lombok* entra para nos tornar mais produtivos.

**Ele funfiona conectando-se ao nosso processo de construção e gerando automaticamente o *bytecode* *Java* em nossos arquivos `.class` de acordo com algumas anotações de projeto que introduzimos em nosso código.**

Podemos conectar a biblioteca *Java* do *Projeto Lombok* a uma ferramenta de construção para automatizar algum código, como métodos `getter/setter` e variáveis de registro.

Neste artigo, discutiremos como usar o *Lombok* com o *Maven* para aproveitar alguns desses recursos.

## 2. Configuração do Projeto *Maven*

Incluir o *Projeto Lombok* em nossas compilações, idependentemente do sistema que estivermos usando, é muito simples. A página do projeto *Lombok* contém instruções detalhadas sobre os detalhes. Podemos remover a dependência do *Maven* *Lombok* no escopo fornecido:

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.36</version>
    <scope>provided</scope>
</dependency>
```

Depender do *Lombok* não fará com que os usuários dos nossos `.jar's` também dependam dele, pois é uma dependência de compilação pura, não de tempo de execução.

## 3. Inferência de Tipo de Variáveis Locais Finais

**A partir do *Lombok 1.16.20*, o *Lombok* suporta o uso de `var` para inferir o tipo de uma variável local a partir da expressão inicializadora. O *Lombok 1.18.22* suporta o uso de `val` para inferência de tipo de variáveis locais finais. Em outras palavras, O *Lombok* substitui `val` por `var` final.** No entanto, não podemos usar esse recurso em campos.

Vamos demonstrar a inferência de tipos de variáveis locais do *Lombok* com um exemplo:

```java
public String lombokTypeInferred() {
    val list = new ArrayList<String>();
    list.add("Hello, Lombok!");
    val listElem = list.get(0);
    return listElem.toLowerCase();
}
```

A geração automática de código torna a *lista* uma variável final do tipo `ArrayList<String>`.

## 4. `Getters/Setters`, Construtores

**Encapsular propriedades de objetos por meio de métodos `getter` e `setter` públicos é uma prática comum em *Java*, e muitas estruturas dependem extensivamente desse padrão `Java Bean` (uma classe com um construtor vazio e métodos `get/set` para *"propriedades"*).**

Isso é tão comum que a maioria dos `IDEs` oferece suporte à geração automática de código para esses padrões (e outros). No entanto, esse código precisa permanecer em nossos códigos-fonte e ser mantido quando uma nova propriedade é adicionada ou um campo é renomeado.

Vamos considerar esta classe que queremos usar como uma entidade `JPA`:

```java
@Entity
public class User implements Serializable {
    private @Id Long id; // will be set when persisting

    private String firstName;
    private String lastName;
    private int age;

    public User() {}

    public User(String firstName, String lastName, int age) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.age = age;
    }

    // getters and setters: ~30 extra lines of code
}
```

Esta é uma classe bastatnet simples, mas imagine se tivéssemos adicionado o código extra para `getters` e `setters`. Teríamos acabado com uma definição com mais código clichê de valor zero do que informações comerciais relevantes: *"Um usuário tem nome, sobrenome e idade"*.

Vamos agora *"Lombokizar"* esta classe:

```java
@Entity
@Getter @Setter @NoArgsConstructor // <--- THIS is it
public class User implements Serializable {
    private @Id Long id; // will be set when persisting

    private String firstName;
    private String lastName;
    private int age;

    public User (String firstName, String lastName, int age) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.age = age;
    }
}
```

Ao adicionar as anotações `@Getter` e `@Setter`, pedimos ao *Lombok* para gerá-las para todos os campos de classe. `@NoArgsConstructor` levará à geração de um construtor vazio.

*Observe que este é o código completo da classe; ao contrário da versão acima com o comentário `// getters and setters`, não estamos omitindo nada. Para uma classe com três atributos relevantes, isso representa uma economia de código significativa!*

Se adicionarmos atributos (propriedades) à nossa classe `User`, o mesmo acontecerá; aplicamos as anotações ao próprio tipo para que eles ocupem todos os campos por padrão.

E se quisermos refinar a visibilidade de algumas propriedades? Por exemplo, se quisermos manter os modificadores de campo `id` das nossas entidades, `package` ou `protected`, visíveis porque devem ser lidos, mas não explicitamente definidos pelo código do aplicativo, podemos simplesmente usar um `@Setter` mais refinado para este campo específico:

```java
private @Id @Setter(AccessLevel.PROTECTED) Long id;
```

## 5. Lazy Getter

Os aplicativos geralmente precisam executar operações caras e salvar os resultados para uso posterior.

Por exemplo, digamos que precisamos ler *dados estáticos* de um *arquivo ou banco de dados*. Geralmente, é uma boa prática recuperar esses dados uma vez e armazená-los em *cahce* para permitir leituras na memória dentro do aplicativo. Isso evita que o aplicativo repita a operação custosa.

**Outro padrão comum é recuperar esses dados somente qunado forem necessários pela primeira vez. Em outras palavras, só obtemos os dados qunado chamamos o `getter` correspondente pela primeira vez. Chamamos isso de `lazy-loading`.**

Suponhamos que esses dados sejam armazenados em *cache* como um campo dentro de uma classe. A classe agora precisa garantir que qualquer acesso a esse campo retorne os dados armazenados em *cahce*. Uma maneira possível de implementar essa classe é usar o método `getter` para recuperar os dados somente se o campo for `nulo`. **Chamamos isso de *Lazy Getter*.**

*O **Lombok** torna isso possível com o parâmetro `lazy` na anotação `@Getter` que vimos acima.*

Por exemplo, considere esta classe simples:

```java
public class GetterLazy {
    @Getter(lazy = true)
    private final Map<String, Long> transactions = getTransactions();

    private Map<String, Long> getTransactions() {
        final Map<String, Long> cache = new HashMap<>();

        txnRows.forEach(s -> {
            String[] txnIdValueTuple = s.split(DELIMETER);
            cache.put(txnIdValueTuple[0], Long.parseLong(txnIdValuetuple[1]));
        });

        return cache;
    }
}
```

Isso lê algumas transações de um arquivo para um `Map`. Como os dados no arquivo não mudam, vamos armazená-los em *cache* uma vez e permitir o acesso por meio de um `getter`.

*Se agora olharmos para o código compilado desta classe, veremos um método `getter` que atualiza o *cache* se ele for `nulo` e então retorna os dados armazenados em <ins>cache</ins>:*

```java
public class GetterLazy {
    private final AtomicReference<Object> transactions = new AtomicReference();

    public GetterLazy() {}

    // other methods

    public Map<String, Long> getTransactions() {
        Object value = this.transactions.get();

        if (value == null) {
            synchronized(this.transactions) {
                value = this.transactions.get();

                if (value == null) {
                    Map<String, Long> actualValue = this.readTxnsFromFile();
                    value = actualValue = null ? this.transactions : actualValue;
                    this.transactions.set(value);
                }
            }
        }

        return (Map) ((Map)(value == this.transactions ? null : value));
    }
}
```

É interessante ressaltar o ***Lombok* encapsula o campo de dados uma `AtomicReference`**. Isso garante *atualizações atômicas* no campo `transactions`. O método `getTransactions()` também garante a leitura do arquivo se `transactions` for `nulo`.

*Não recomendamos o uso de campo de transações `AtomicReference` diretamente de dentro da classe. **Recomendadmos o uso do método `getTransactions()` para acessar o campo**.*

Por esse motivo, se usarmos outra anotação do *Lombok* como `ToString` na mesma classe, ela usará `getTransactions()` em vez de acessar diretamente o campo.

## 6. `Setter` Imutável com um Campo Alterado

Além disso, *podemos usar a anotação `@With` para gerar automaticamente um método que pode clonar um objeto com um campo alterado*. Por exemplo, quando queremos clonar um objeto `User` com um novo valor de `age`, podemos anotar o campo com `@With`:

```java
@AllArgsConstructor
public class User implements Serializable {
    private @Id Long id;
    
    private final String firstName;
    private final String lastName;
    @With private final int age;
}
```

Posteriormente, podemos usar o método de instância `withAge(int newAge)` gerado automaticamente para clonar um `User`, mas com *uma idade diferente*:

```java
User user = new User("John", "Smith", 40);
User user_upadted = user.withAge(41);
```

## 7. Classes de `valor/DTO's`

**Existem muitas situações em que queremos definir um tipo de dado com o único propósito de representar *"valores"* complexos como *"Objetos de Transferência de Dados (DTO's)"*.** Na maioria das vezes, na forma de estruturas de dados imutáveis que construímos uma vez e nunca queremos alterar.

Projetamos uma classe para representar uma *operação de login* bem-sucedida. Queremos que todos os campos não sejam `nulos` e os objetos sejam *imutáveis* para que possamos acessar suas propriedades com segurança em `threads`:

```java
public class LoginResult {
    private final Instant loginTs;

    private final String authToken;
    private final Duration tokenValidaty;

    private final URL tokenRefreshUrl;

    // constructor taking every field and checking nulls

    // read-only accessor, not necessarily as get*() form
}
```

Novamente, a quantidade de código que teríamos que escrever para as seções comentadas seria muito maior do que a quantidade necessária de informações que queremos encapsular. Podemos usar o *Lombok* para melhorar isso:

```java
@RequiredArgsConstructor
@Accessors(fluent = true) @Getter
public class LoginResult {
    private final @NonNull Instant loginTs;

    private final @NonNull String authToken;
    private final @NonNull Duration tokenValidity;

    private final @NonNull URL tokennRefreshUrl;
}
```

Após adicionarmos a anotação `@RequiredArgsConstructor`, obteremos um construtor para todos os campos finais da classe, exatamente como os declararmos. Adicionar `@NonNull` aos atributos faz com que nosso construtor verifique a nulidade e lance `NullPointerException` de acordo. Isso também aconteceria se os campos fossem não finais e adicionássemos `@Setter` para eles.

**Queremos o velho e chato formato `get*()` para nossas propriedades? Como adicionamos `@Accessors(fluent=true)` neste exemplo, `getters` teria o mesmo nome de método que as propriedades; `getAuthToken()` simplesmente se torna `authToken()`.**

Esta forma `fluent` se aplica a campos não finais para definidores de atributos, *além de permitir chamadas encadeadas*:

```java
// Imagine fields were no longer final now
return new LoginResult()
    .loginTs(Instant.now())
    .authToken("asdasd")
    // and so on
```

## 8. Modelo Básico do *Java*

**Outra situação em que acabamos escrevendo código que precisamos manter é ao gerar os métodos `toString()`, `equals()` e `hashCode()`.** `IDEs` tentam ajudar com modelos para geração automática desses métodos em termos de nossos atributos de classe.

Podemos automatizar isso usando outras anotações de *nível de classe* do *Lombok*:

- `@ToString`: gerará um método `toString()` incluindo todos os atributos da classe. Não é necessário escrever um e mantê-lo enquanto enriquecemos nosso modelo de dados.
- `@EqualsAndHashCode`: gerará os métodos `equals()` e `hashCode()` *por padrão*, considerando todos os campos relevantes e de acordo com [*uma semântica muito bem pensada*](http://www.artima.com/lejava/articles/equality.html).

Esses geradores oferecem *opções de configuração* muito úteis. Por exemplo, se as nosass *classes anotadas* fizerem parte de uma hierarquia, podemos simplesmente usar o parâmetro `callSuper=true` e os resultados dos pais serão considerados ao gerar o código do método.

### 8.1. Uma Demonstração

Para demonstar isso, digamos que nosso exemplo de *entidade `JPA`* do *usuário* inclui uma *referência a eventos associados a este usuário*:

```java
@OneToMany(mappedBy = "user")
private List<UserEvent> events;
```

*Não gostaríamos que toda a lista de eventos fosse despejada sempre que chamássemos o método `toString()` de nosso `User`, só porque usamos a anotação `@ToString`. Em vez disso, podemos parametrizá-la assim, `@ToString(exclude = {"events"})`, e isso não acontecerá. Isso também é útil para evitar referências circulares se, por exemplo, `UserEvents` tivessem uma referência a um `User`.*

*Para o exemplo `LoginResult`, podemos definir a igualdade e o cáluclo de código `hash` apenas em termos do `token` em si e não dos outros atributos finais da nossa classe. Então, podemos simplesmente escrever algo como `@EqualsAndHashCode(of = {"authToken"})`.*

Se os recursos das anotações até agora forem de seu interesse, talvez seja interessante examinar também as anotações [`@Data`](https://projectlombok.org/features/Data.html) e [`@Value`](https://projectlombok.org/features/Value.html), pois elas se comportam como se um conjunto delas tivesse sido aplicados às nossas classes. Afinal, esses usos discutidos são comumente combinados em muitos casos.

### 8.2. *(Não)* usar `@EqualsAndHashCode` com Entidades `JPA`

Se devemos usar os métodos padrão `equals()` e `hashCode()` ou criar métodos personalizados para as *entidades `JPA`* é um tópico frequentemnte discutido entre desenvolvedores. Existem [diversas abordagens](https://www.baeldung.com/jpa-entity-equality) que podemos seguir, cada uma com seus prós e contras.

**Por padrão, `@EqualsAndHashCode` inclui todas as propriedades não finais da classe de entidade.** Podemos tentar *"consertar"* isso usando o atributo `onlyExplicityIncluded` de `@EqualsAndHashCode` para fazer o *Lombok* usar apenas a chave primária da entidade. Ainda assim, o método `equals()` gerado pode causar alguns problemas. *Thorben Janssen* explica esse cenário com mais detalhes em [uma de suas postagens no blog](https://thorben-janssen.com/lombok-hibernate-how-to-avoid-common-pitfalls).

*Em geral, **devemos evitar usar o <ins>Lombok</ins> para gerar os métodos `equals()` e `hashCode()` para nossas entidades `JPA`**.*

## 9. O Padrão do Construtor

O seguinte poderia ser uma classe de configuração de exemplo para um cliente de `API REST`:

```java
public class ApiClientConfiguration {
    private String host;
    private int port;
    private boolean useHttps:

    private long connectTimeout;
    private long readTimeout;

    private String username;
    private String password;

    // Whataver other options you may thing.

    // Empty constructor? All combinations?

    // getters... and setters?
}
```

Poderiamos adotar uma abordagem inicial baseada no uso do construtor vazio padrão da classe e no fornecimento de métodos `setter` para cada campo; no entanto, o ideal é que as configuraçõpes não sejam redefinidas *após* serem construídas *(instanciadas)*, tornando-as efetivamente imutáveis. Portanto, queremos evitar `setters`, *mas escrever um construtor com argumentos potencialmente longos é um antipadrão*.

Em ves disso, podemos dizer à ferramenta para gerar um padrão de *construtor*, o que nos impede de ter que escrever uma classe `Builder` extra e os métodos associados do tipo `setter` flutente, simplesmente adicionando a anotação `@Builder` à nossa `ApiClientConfiguration`:

```java
@Builder
public class ApiClientConfiguration {
    // ... everything else remains the same
}
```

Deixando a definição da classe como acima *(sem declarar construtores ou `setters` + `@Builder`)*, poderiamos acabar usando-a como:

```java
ApiClientConfiguration config = 
    ApiClientConfiguration.builder()
        .host("api.server.com")
        .port(443)
        .useHttps(true)
        .connectTimeout(15_000L)
        .readTimeout(5_000L)
        .username("myusername")
        .password("secret")
        .build();
```

## 10. Exceções Verificadas

*Muitas `APIs` <ins>Java</ins> são projetadas para lançar uma ou mais das várias exceções verificadas; o código do cliente é forçado a capturar ou declarar para `throws`.* Quantas vezes transformamos essas exceções que sabemos que não acontecerão em algo assim:

```java
public String resourceAsString() {
    try (InputStream is = this.getClass().getResourceAsStream("sure_in_my_jar.txt")) {
        BufferedReader br = new BufferedReader(new InputStreamReader(is, "UTF-8"));

        return bt.lines().collect(Collectors.joining("/n"));
    } catch (IOException | UnsupportedCharsetException ex) {
        // If this ever happens, then its a bug.

        throw new RuntimeException(ex); // <--- encapsulate into a Runtime ex.
    }
}
```

Se quisermos evitar esse padrão de código porque o compilador não ficará feliz *(e sabemos que os erros verificados não podem acontecer)*, use o apropriadamente chamado `@SneakyThrows`:

```java
@SneakyThrows
public String resourceAsString() {
    try (InputStream is = this.getClass().getResourceAsStream("sure_in_my_jar.txt")) {
        BufferedReader bt = new BufferedReader(new InputStreamReader(is, "UTF-8"));

        return br.lines().collect(Collectors.joining("\n"));
    }
}
```

## 11. Garantir que nossos Recursos sejam Liberados

O **Java 7** introduziu o bloco `try-with-resources` para garantir que os recursos mantidos por instâncias da implementação `java.lang` e `AutoCloseable` sejam liberados ao sair.

O *Lombok* oferece uma maneira alternativa e mais flexível de fazer isso via `@Cleanup`. Podemos usá-lo para qualquer variável local cujos recursos queremos garantir que sejam liberados. Não há necessidade de implementar nenhuma interface específica; basta chamar o método `close()`:

```java
@Cleanup InputStream is = this.getClass().getResourceAsStream("res.txt");
```

Nosso  método de liberação tem um nome diferente? Sem problemas, apenas personalizamos a anotação:

```java
@Cleanup("dispose") JFrame mainFrame = new JFrame("Main Window");
```

## 12. Anotar nossa Classe para Obter um Registrador

Muitos de nós adicionamos instruções de registro ao nosso código com moderação, criando uma instância de um `Logger` a partir do framework de nossa escolha. Digamos `SLF4J`:

```java
public class ApiClientConfiguration {
    private static Logger LOG = LoggerFactory.getLogger(ApiClientConfiguration.class);

    // LOG.debug(), LOG.info(), ...
}
```

Esse é um padrão tão comum que os desenvolvedores do *Lombok* o simplificaram para nós:

```java
@Slf4j // or: @Log @CommonsLog @Log4j @Log4j2 @XSLf4j
public class ApiClientConfiguration {
    // log.debug(), log.info(), ...
}
```

Muitas [estruturas de registro](https://projectlombok.org/features/log) são suportadas e, claro, podemos personalizar o nome da instância, o tópico, etc.

## 13. Escreva Métodos mais Seguros para `threads`

Em *Java*, podemos usar a palavra-chave `synchronized` para implementar *seções críticas*; no entanto, essa **não é um abordagem 100% segura**. *Outros códigos de cliente também podem eventualmente sincronizar em nossa instância, o que pode levar a `deadlocks` inesperados.*

**É aqui que entra o [`@Synchronized`](https://projectlombok.org/features/Synchronized.html), Podemos anotar nossos métodos *(tanto de instância quanto estáticos)* com ele, e obteremos um campo privado, não exposto e gerado automaticamente, que nossa implementação usará para bloqueio:**

```java
@Synchronized
public /* better than: synchronized */ void putValueInCache(String key, Object value) {
    // whatever here will be a thread-safe code
}
```

Entretanto, ao usar [`threads` virtuais](https://www.baeldung.com/java-virtual-thread-vs-thread), introduzidas no **Java 21**, devemos usar as anotações `@Locked`, `@Locked.Read` e `@Locked.Write` para obter um `ReentrantLock`.

## 14. Automatize a Composição de Objetos

***Java* não possui construção em *nível de linguagem* para facilitar uma abordagem de *"favorecer a heranã da composição"*.** Outras linguagens possuem conceitos integrados, como `Traits` ou `Mixins`, para alcançar esse objetivo.

*O `@Delegate` do <ins>Lonbok</ins> é muito útil quando queremos usar esse padrão de programação.* Vejamos um exemplo em que:

- deseja que *usuários* e *clientes* compartilhem alguns *atributos comuns* para nomenclatura e número de telefone.
- defina uma *interface* e uma *classe* de adaptador para esses campos.
- faça com que nossos modelos implementem a *interface* e `@Delegate` em seu adaptador, compondo-os efetivamente com nossa informações de contato.

Primeiro, vamos definir uma interface:

```java
public interface HasContactInformation {
    String getFirstName();
    void setFirstName(String firstName);

    String getFullName();

    String getLastName();
    void setLastName(String lastName);

    String getPhoneNr();
    void setPhoneNr(String phoneNr);
}
```

Agora um adaptador como *classe de suporte*:

```java
@Data
public class ContactInformationSupport implements HasContactInformation {
    private String firstName;
    private String lastName;
    private String phoneNr;

    @Override
    public String getFullName() {
        return getFirstName() + " " + getLastName();
    }
}
```

*Agora, a parte interessante, vamos ver como é fácil compor informações de contato em ambas as classes de modelo:*

```java
public class User implements HasContactInformation {
    // Whichever other User-specific attributes

    @Delegate(types = {HasContactInformation.class})
    private final ContactInformationSupport contactInformation = new ContactInformationSupport();

    // User itself will implement all contact information by delegation
}
```

O caso do `Cliente` seria tão semelhante que podemos omitir o exemplo por brevidade.

## 15. Contantes para Nomes de Campos

Ao lidar com classes com muitos campos, especialmente ao trabalhar com reflexão, frameworks de serialização ou construção de consultas dinâmicas, frequentemente criamos constantes de string representando nomes de campos. Gerenciar essas constantes manualmente é repetitivo e propenso a erros.

*O <ins>Lombok</ins> simplifica essa tarefa fornecendo a anotação `@FieldNameConstants`, que gera automaticamente constantes de `string` para os nomes de campo de uma classe.*

Vamos considerar a seguinte classe:

```java
@Getter
@FieldNameConstants
public class Person {
    private final String firstName;
    private final String lastName;
    private final int age;

    public Person(String firstName, String lastName, int age) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.age = age;
    }
}
```

Quando usamos a anotação `@FieldNameConstants`, o *Lombok* gera uma classe estática aninhada conveniente contendo nomes de campos constantes. Isso nos permite referenciar campos com segurança e evitar a codificação de literais de `string` em nosso código.

Por exemplo, o *Lombok* gera uma classe aninhada `Fields` com constantes:

```java
public class Person {
    public static final class Fields {
        public static final String firstName = "firstName";
        public static final String lastName = "lastName";
        public static final String age = "age";
    }

    // Existing code
}
```

Agora, sempre que precisamos nos referir a campos pelo nome, podemos usar essas constantes, evitando erros de digitação e simplificando a refatoração:

```java
CriteriaBuilder cb = entityManager.getCriteriaBuilder();
CriteriaQuery<Person> query = cb.createQuery(Person.class);
Root<Person> root = query.from(Person.class);
query.select(root).where(cb.equals(root.get(Person.Fields.lastName), "Doe"));
```

Esse recurso nos ajuda a evitar literais de `string` espalhados pelo código, tornando-o mais seguro, mais legível e mais fácil de manter.

Tambémm podemos personalizar o nome da classe interna:

```java
@FieldNameConstants(innerTypeName = "FieldConstants")
```

## 16. Rolando *Lombok* para Trás?

*Resposta curta: De jeito nenhum, na verdade.*

Pode haver a preocupação de que, se usarmos o *Lombok* em um de nossos projetos, possamos posteriormente reverter essa decisão. O problema em potencial pode ser o grande de número de classes anotadas para ele. Nesse caso, estamos cobertos graças a ferramenta `delombok` do mesmo projeto.

Ao *"delombokizar"* nosso código, obtemos código-fonte *Java* gerado automaticamente com exatamente os mesmos recursos do *bytecode* criado pelo *Lombok*. Podemos então simplesmente substituir nosso código anotado orignial por esses novos arquivos *"delombokizados"* e não depender mais dele.

Isso é algo que podemos [integrar em nossa construção](https://projectlombok.org/features/delombok.html).

## 17. Conclusão

Ainda há alguns outros recursos que não apresentamos neste artigo. Podemos nos aprofundar na [visão geral dos recursos](https://projectlombok.org/features/index.html) para obter mais detalhes e casos de uso.

Além disso, a maioria das funções que mostramos possui diversas opções de personalização que podem ser úteis. O sistema de configuração integrado também pode nos ajudar com isso.

Agora que podemos dar ao *Lombok* a chance de acessar nosso conjunto de ferramentas de desenvolvimento *Java*, podemos aumentar nossa produtividade.

## Fonte:

- Artigo: [Introduction to Project Lombok](https://www.baeldung.com/intro-to-project-lombok)