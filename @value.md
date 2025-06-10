# Guia Rápido para Spring `@Value`

## 1. Visão Geral

Esta anotação pode ser usada para injetar valores em campos em *beans* gerenciados pelo **SPring** e pode ser aplicada no vível de parâmetro de campo ou de *construtor/método*.

## 2. Configurando o Aplicativo

Para descrever diferentes tipos de uso desta anotação, precisamos configurar uma classe de configuração de aplicativo *Spring*.

**Precisamos de um arquivo de propriedades** para definir os valores que queremos injetar com a anotação `@Value`. Primeiro precisaremos definr uma `@PropertySouce` em nossa *classe de configuração* - com o *nome do arquivo de propriedades*.

```
value.from.file=Value got from the file
priority=high
listOfValues=A,B,C
```

## 3. Exemplos de Uso

Como exemplo básico, podemos apenas injetar *"valor de string"* da anotação no campo:

```java
@Value("string value")
private String stringValue
```

Usar a anotação `@PropertySouce` nos permite trabalhar com valores de arquivos de propriedades com a anotação `@Value`.

```java
@Value("${value.from.file}")
private String valueFromFile;
```

Podemos também definir o valor das propriedades do sistema com a mesma sintaxe.

Vamos supor que definimos uma propriedade do sistema chamada `systemValue`:

```java
@Value("${systemValue}")
private String systemValue;
```

Valores padrão podem ser fornecidos para propriedaes que podem não estar definidas.

```java
@Value("${unknown.param:some default}")
private String someDefault;
```

Se a mesma propriedade for definida como uma propriedade do sistema e no arquivo de propriedades, então a propriedade do sistema será aplicada.

Às vezes, precisamos injetar vários valores. Seria conveniente defini-los como valores separados por vírgula para uma única propriedade no arquivo de propriedades ou como uma propriedade do sistema e injetá-los em um array.

Na primeira seção, definimos valores separados por vírgula na *lista de valores* do arquivo de propriedades, então os *valores do array* seriam `["C","B", "C"]`:

```java
@Value("${listOfValues}")
private String[] valuesArray;
```

## 4. Exemplos Avançados com SpEL

Também podemos usar expressões **SpEL** para obter o valor.

Se tivermos uma propriedade do sistema chamada *priority*, seu valor será aplicada ao campo:

```java
@Value("${systemProperties['priority']}")
private String spelValue;
```

Se não tivermos definido a propriedade do sistema, o valor *nulo* será atribuído.

Para evitar isso, podemos fornecer um valor padrão na expressão *SpEL*. Obteremos *um valor padrão* para o campo se a propriedade do sistema não estiver definida:

```java
@Value("#{systemProperties['unknown'] ?: 'some default'}")
private String spelSomeDefault;
```

Podemos usar um *valor de campo* de *outros **beans***. Suponha que temos um bean chamado *someBean* com um campo *someValue* igual a *10*. Então, *10* será atribuído:

```java
@Value("#{someBean.someValue}")
private Integer someBeanValue;
```

Podemos manipular propriedades para obter uma *lista de valores*, aqui, uma *lista de valores de string A, B, C*:

```java
@Value("#{'${listOfValues}'.split(',')}")
private List<String> valuesList;
```

## 5. Usando `@Value` com Mapas

Podemos usar a *anotação `@Value`* para injetar uma *propriedade Map*.

Primeiro, precisamos definir a propriedade no formato `{key: 'value'}` em nosso arquivo de propriedades:

```
valuesMap={key1: '1', key2: '2', key3: '3'}
```

**Observe que os valores no *Mapa* devem estar entre aspas simples.**

Podemos injetar esse valor do arquivo de propriedades como um *Map*:

```java
@Value("#{${valuesMap}}")
private Map<String, Integer> valuesMap;
```

Se precisarmos **obter o valor de uma *chave específica*** no *Map*, tudo o que precisamos fazer é **adicionar o nome da chave na expressão**:

```java
@Value("#{${valuesMap}.key1}")
private Integer valuesMapKey1;
```

Se não tivermos certeza se o *Mapa* contém uma determinada chave, devemos escolher uma *expressão mais segura que não gere uma exceção*, mas defina o valor como *nulo* quando a chave não for encontrada:

```java
@Value("#{${valueMap}['unknownKey']}")
private Integer unknownMapKey;
```

Podemos **definir valores padrões para as propriedades ou chaves que podem não existir**:

```java
@Value('#{${unknownMap: {key1: '1', key2: '2'}}}')
private Map<String, Integer> unknownMap;

@Value("#{${valuesMap}['unknownKey'] ?: 5}")
private Integer unknownMapKeyWithDefaultValue;
```

*As entradas do mapa também podem ser filtradas* antes da injeção.

Vamos supor que precisamos obter apenas as entradas *cujo valores são maiores que um*:

```java
@Value("#{${valuesMap}.?[value>'1']}")
private Map<String, Integer> valuesMapFiltered;
```

Podemos usar a anotação `@Value` para *injetar todas as propriedades atuais do **sistema***:

```java
@Value("#{systemProperties}")
private Map<String, String> systemPropertiesMap;
```

## 6. Usando `@Value` com *Injeção de Construtor*

Quando usamos a *anotação `@Value`*, **não estamos limitados à injeção de campo. Também podemos usá-la em conjunto com a *injeção de construtor***.

```java
@Component
@PropertySource("classpath:values.properties")
public class PriorityProvider {
    private String priority;

    @Autowired
    public PriorityProvider(@Value("${priority:normal}") String priority) {
        this.priority = priority;
    }

    // standard getter
}
```

## 7. Usando `@Value` com *Injeção de `setter`*

Analogamente à *injeção do construtor*, *também podemos usar @Value com injeção do setter*.

```java
@Component()
@PropertySource("classpath:value.properties")
public class CollectionProvider {
    private List<String> values = new ArrayList<>();

    @Autowired
    public void setValues(@Value("#{'${listOfValues}'.split(',')}") List<String> values) {
        this.values.addAll(values);
    }

    // standard getter
}
```

Usamos a *expressão **SpEL*** para injetar uma lista de valores no método `setValues`.

## 8. Usando `@Value` com Registros

O **Java 14** *introduziu registros para facilitar a criação de uma classe imutável*. O **framework Spring** suporta `@Value` para *injeção de registros*.

```java
@Component
@PropertySource("classpath:values.properties")
public record PriorityRecord(@Value("${priority.normal}") String priority) {}
```

## Fonte

- Artigo: [A Quick Guide to Spring @Value](https://www.baeldung.com/spring-value-annotation)