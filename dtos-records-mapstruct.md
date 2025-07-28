# Como criar `DTOs` com `records` e `MapStruct` no *Spring Boot*

> Eu não diria que os `DTOs` são uma parte essencial de qualquer aplicativo. Mas, caso você não consiga viver sem eles, existe uma maneira de reduzir o código repetitivo e simplificar o processo de transmissão de dados por `DTOs` - tudo graças aos `records` *Java* e ao *framework MapStruct*.

Abaixo, você encontrará um tutorial sobre como integrar essas duas soluções poderosas ao seu projeto.

## O que são `records`

Os `records` (registros) foram introduzidos pela primeira vez no **Java 14** e finalizados no **Java 16**. Eles representam portadores de dados imutáveis que permitem aos desenvolvedores omitir muito código clichê ao definir uma classe de dados simples. *Por padrão, todos os campos definidos em uma classe de `record` são privados e finais.* Os métodos (como `equals()`, `hashCode()`, `toString()`) já estão lá, assim como o *construtor*.

Mas porque precisamos de registros quando temos o *Lombok*? É verdade que o *Lombok* também ajuda a reduzir o *boilerplate*. Mas você precisa adicionar inúmeras anotações à classe ao usar o *Lombok*, *"e Deus nos livre de squeçer uma delas"*. Certa vez, esqueci de adicionar a anotação `@Getter` a um de meus `DTOs` e passei meia hora com um depurador tentando entender porque os campos do `DTO` são nulos.

NO entanto, em alguns casos, *temos que* nos ater ao *Lombok*. Entidades *JPA*, por exemplo, não podem ser transformadas em registros, porque, entre outros aspectos, não podem ser finais e devem fornecer `getters`, `setters` e um construtor sem parâmetros.

Mas os `DTOs` são uma combinação perfeita para registros, o que veremos a seguir:

## Criar `DTOs` usando registros

Vamos primeiro dar uma olhada na classe `Film` para a qual vamos criar `DTOs`:

```java
@NoArgsConstructor
@AllArgsConstructor
@Getter
@Setter
@Builder
@Entity
@Table(name = "film")
public class Film {

    @Id
    @Column(name = "id", nullable = false)
    @GeneratedValue(strategy = GeneretionType.IDENTITY)
    private Long id;

    @Column(name = "name")
    @NotNull
    private String name;

    @Column(name "description", columnDefinition = "TEXT", length = 200)
    @NotNull
    private String description;

    @Column(name = "premiere_date", columnDefinition = "DATE")
    @DateTimeFormat(pattern = "yyyy-MM-dd")
    @NotNull
    private LocalDate premiereDate;

    @ManyToMany(fetch = FetchType.LAZY, 
        cascade = {CascadeType.DETACH, CascadeType.MERGE, CascadeType.PERSIST, CascadeType.REFRESH}
    )
    @JoinTable(name = "film_genre",
        joinColumns = @JoinColumn(name = "film_id"),
        inverseJoinColumns = @JoinColumn(name = "genre_id")
    )
    private Set<Genre> genres = new HashSet<>();

    @Column(name = "duration")
    @NotNull
    @Positive
    private int duration;

    @ManyToOne(
        cascade = {CascadeType.DETACH, CascadeType.MERGE, CascadeType.PERSIST, CascadeType.REFRESH}
    )
    @JoinColumn(name = "mpaa_id")
    private Mpaa mpaa;

    @ManyToMany(fetch = FetchType.LAZY,
        cascade = {CascadeType.DETACH, CascadeType.MERGE, CascadeType.PERSIST, CascadeType.REFRESH}
    )
    @JoinTable(name = "film_actor",
        joinColumns = @JoinColumn(name = "film_id"),
        inverseJoinColumns = @JoinColumn(name = "star_id")
    )
    private Set<FilmPerson> actors = new HashSet<>();

    @OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "film_id")
    private Set<Rating> ratings = new HashSet<>();

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Film film = (Film) o;
        return Objects.equals(id, film.id);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}
```

Como você pode ver, esta classe inclui vários campos, além de entidades aninhadas. Mas, quando exibimos informações sobre um filme, não queremos incluir todas as informações sobre atores, gêneros e classificação de MPAA, apenas os nomes. Por outro lado, não queremos passar conjuntos de objetos inteiros no lado do cliente para preencher um novo filme: passar um `id` deve ser suficiente. Então, vamos criar nossos `DTOs` personalizados de acordo.

```java
@Builder
public record FilmResponse(
    
    String name,
    
    String description,

    LocalDate premiereDate,
    
    int duration,

    List<String> genres,

    String mpaa,

    List<String> actors,

    double rationg
) {
}
```

A princípio, você pode não notar a diferença a uma lista padrão de variáveis. Mas há uma diferença. *Veja bem*, sem *modificadores de acesso*, sem *construtores*, sem *boilerplate*, sem *inúmeras anotações do Lombok* *(exceto a anotação `@Builder`)* - muito lacônico e bonito! E se você não tiver instruções `if` *complexas* em seus métodos de mapeamento, `@Builder` também pode se livrar delas e, posteriormente, criar objetos `DTO` usando um construtor fornecido com *registros*.

Que tal uma nova solicitação de filme? Queremos adicionar anotações de validação para delegar a verificação de dados ao *Spring Boot*. *A boa notícia é que os registros ou `records` nos permitem adicionar anotações aos campos!*

```java
@Builder
public record NewFilmRequest(
    
    @NotBlank
    String name.

    @NotBlank
    @Size(max = 200)
    String description,

    @NotNull
    LocalDate premiereDate,

    @NotNull
    @Positive
    int duration,

    @NotNull
    Set<Long> genreIds,

    @NotNull
    Set<Long> actorIds,

    @NotNull
    long mpaaId   
) {
}
```

*Você pode criar um `DTO` para atualizar um filme ou qualquer outro `DTO` necessário para um filme de maneira semelhante.*

Os `DTOs` estão prontos, vamos mapeá-los!

## Use `MapStruct` para converter *Entidades* em *DTOs* e vice-versa

## Adicionar dependências do `MapStruct`

Primeiro, precisamos adicionar as dependências necessárias do `MapStruct`.
Se você usa *Maven*, adicione a seguinte dependência do `MapStruct` ao `pom.xml`:

```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.5.5.Final</version>
</dependency>
```

Também precisamos adicionar o processador `MapStruct` *ao plugin* *Maven*, que gera implementações do mapeador durante a fase de compilação. Além disso, devemos adicionar a dependência `lombok-mapstruct-binding` *ao plugin* se quisermos usar o *Lombok*:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.13.0</version>
    <configuration>
        <annotationProcessorPaths>
            <path>
                <groupId>org.mapstruct</groupId>
                <artifactId>mapstruct-processor</artifactId>
                <version>1.5.5.Final</version>
            </path>
            <path>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>1.18.32</version>
            </path>
            <path>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok-mapstruct-binding</artifactId>
                <version>0.2.0</version>
            </path>
        </annotationProcessorPaths>
    </configuration>
</plugin>
```

> Estou usando as versões mais recentes do `MapStruct` e seu processador no momento em que escrevo este artigo. Você pode sempre verificar os repositórios *Maven* do [MapStruct Core](https://mvnrepository.com/artifact/org.mapstruct/mapstruct) e do [MapStruct Processor](https://mvnrepository.com/artifact/org.mapstruct/mapstruct-processor) para obter as atualizações.

## Mapeamento básico com *interface* `MapStruct`

Vamos deixar de lado nossa entidade `Film` por um minuto e analisar mais exemplos básicos de mapeamento.

Em um mundo perfeito, os campos `DTO` são os mesmos da nossa entidade.
Por exemplo, temos uma entidade `Post`:

```java
public class Post {
    
    private Long id;

    private String text;

    private Long userId;
}
```

E o registro *DTO* correspondente:

```java
public record PostDto(

    Long id,

    String text,

    Long userId
) {
}
```

Neste caso, podemos criar uma interface `Mapper` com a anotação `@Mapper` e dois métodos simples:

```java
@Mapper
public interface PostMapper {

    PostMapper INSTANCE = Mappers.getMapper(PostMapper.class);
    PostDto mapPostToDto(Post post);
    Post mapDtoTOPost(PostDto dto);
}
```

O campo `INSTANCE` pode ser usado quando você precisa executar o mapeamento em suas classes de serviço *(ou onde você prefirir fazer o mapeamento)*.

**Pronto!** Você não precisa criar uma implemnetação do `Mapper`, pois ela será gerada automaticamente ao executar o aplicativo. Abaixo está a implementação que o `MapStruct` gerou para nós em `target/annotations/com/example/demo/PostMapperImpl.java`:

```java
@Generated(
    value = "org.mapstruct.ap.MappingProcessor",
    date = "2024-06-10T13:04:45+0300",
    comments = "version: 1.5.5.Final, compiler: javac, environment: Java 21.0.3 (BellSoft)"
)
public class PostMapperImpl implements PostMapper {

    @Override
    public PostDto mapPostToDto(Post post) {
        if (post == null) {
            return null;
        }

        Long id = null;
        String text = null;

        id = post.getId();
        text = post.getText();

        Long userId = null;

        PostDto postDto = new PostDto(id, text, userId);

        return postDto;
    }

    @Override
    public Post mapDtoToPost(PostDto dto) {
        if (dto == null) {
            return null;
        }

        Post post = new Post();

        post.setId(dto.id());
        post.setText(dto.text());

        return post;
    }
}
```

*Fácil, né?* Vamos ver como o `MapStruct` lida com outros casos de uso.

Por exemplo, em um mundo menos perfeito, não queremos mapear certos campos como `id`, então nosso `PostDto` será um pouco diferente:

```java
public record PostDto(

    String text,

    Long userId
) {
}
```

Nesse caso, podemos excluir o campo ausente do mapeamento adicionando `unmappedTargetPolicy` à anotação `@Mapper`:

```java
@Mapper(unmappedTargetPolicy = ReportingPolicy.IGNORE)
public interface PostMapper {

    PostMapper INSTANCE = Mappers.getMapper(PostMapper.class);

    PostDto mapPostToDto(Post post);

    Post mapDtoToPost(PostDto dto);
}
```

Como resultado, o `MapStruct` ignorará propriedades não mapeadas e mapeará apenas o que pode ser mapeado. Além disso, não emitirá nenhum aviso durante a compilação.

E se alguns campos em `DTO` e `Entity` tiverem nomes diferentes? Por exemplo, `userId` em `Post` e `authorId` em `PostDto`:

```java
public record PostDto(
    
    String text,
    
    Long authorId
) {
}
```

Para mapear campos com nomes diferentes, precisamos adicionar a anotação `@Mapping` aos métodos de `PostMapper` e configurar os campos de origem e destino da seguinte maneira:

```java
@Mapper(unmappedTargetPolicy = ReportingPolicy.IGNORE)
public interface PostMapper {

    PostMapper INSTANCE = Mappers.getMapper(PostMapper.class);

    @Mapping(target = "authorId", source = "userId")
    PostDto mapPostToDto(Post post);

    @Mapping(target = "userId", source = "authorId")
    Post mapDtoToPost(PostDto dto);
}
```

*Depois disso, é só sentar, relaxar e deixar o `MapStruct` fazer sua <ins>mágica</ins>!*

## Mapeamento avançado com injeção de dependência

Voltemos a `Film` e seus `DTOs`. O `DTO` `FilmResponse` difere significativamente do `DTO Entity`: em vez de retornar conjunto de entidades aninhadas *(`Genre`, `FilmPerson`)*, ele retorna uma lista com seus nomes. Além disso, precisamos calcular uma classificação média para o filme com base nas classificações fornecidas como um conjunto em nossa entidade.

`NewFilmRequest` é ainda mais complicado: inclui apenas `IDs` de *Entidades*, que precisamos procurar no repositório e adicionar suas referências ao objeto `Film` para poder salvá-lo. Isso significa que precisamos injetar a instância do repositório no *Mapeador*.

*Parece muita dificuldade para configurar o mapeador, mas, na verdade, nada que o `MapStruct` não consiga resolver.*

Para poder usar o *Spring `CDI` e o `IOC`*, precisamos aprimorar nosso `Mapper` da seguinte maneira:

- Alterar `interface` para `abstract class`. Isso também nos permitirá personalizar os métodos de mapeamento;
- Adicione à anotação `@Mapper` a configuração `componentModel = "spring"` para transformar nosso `Mapper` em `Spring Bean` e poder injetá-lo via `@Autowired`;
- Remova o campo `INSTANCE` porque o `Mapper` agora é um `Spring Bean` normal e deve ser adicionado *diretamente às classes que o utilizam*;
- Injete a instância do repositório no `Mapper` via `@Autowired`. Observe que o `MapStruct` *não suporta injeção de construtor*, então você precisa *executar a injeção de campo*, que <ins>não é recomendada</ins>, ou a *injeção de setter*.

Como resultado, nossa classe `FimlMapper` ficará assim:

```java
@Mapper(unmappedTargetPolicy = org.mapstruct.ReportingPolicy.IGNORE, componentModel = "spring")
public abstract class FilmMapper {

    protected ReferenceFinderRepository repository;

    @Autowired
    protected void setReferenceFinderRepository(ReferenceFinderRepository repository) {
        this.repository = repository;
    }

    public FilmResponse mapToFilmResponse(Film film) {
        return FilmResponse.builder()
            .name(film.getName())
            .description(film.getDescription())
            .duration(film.getDuration())
            .premiereDate(film.getPremiereDate())
            .language(film.getLanguage().getName())
            .mpaa(film.getMpaa().getName())
            .genres(film.getGenres().stream().map(Genre::getName).toList())
            .actors(film.getActors().stream().map(FilmPerson::getName).toList())
            .build();
    }

    public Film mapToFilm(NewFilmRequest filmRequest) {
        Film.FilmBuilder film = Film.builder()
            .name(filmRequest.name())
            .description(filmRequest.description())
            .premiereDate(filmRequest.premiereDate())
            .duration(filmRequest.duration())
            .mpaa(repository.getMpaaReference(filmRequest.mpaaId()));

        film.genres(filmRequest.genreIds()
            .stream()
            .map(repository::getGenreReference)
            .collect(Collectors.toSet()));

        film.actors(filmRequest.actorIds()
                .stream()
                .map(repository::getFilmPersonReference)
                .collect(Collectors.toSet()));

            double meanRating = 0.0;

            if (!film.getRatings().isEmpty()) {
                meanRating = film.getRatings()
                    .stream()
                    .map(Rating::getPoints)
                    .mapToint(Integer::intValue)
                    .summaryStatistics()
                    .getAverage();
            }

            response.rating(round(meanRating, 2));

            return film.build();
    }
}
```

## Resumo

Como você pode ver, usar `records` *(registros)* e o `MapStruct` para trabalhar com `DTOs` nos permite reduzir a quantidade de código clichê propenso a erros. Além disso, o `MapStruct` é muito flexível e pode ser usado até mesmo para casos de mapeamento complexos. O `MapStruct` pode fazer muito mais: *descrever todos os seus recuros seria demais para um post de blog.* Consulte a [documentação do `MapStruct`](https://mapstruct.org/documentation/) para saber mais sobre seus recursos.

## Fonte:

- Autor: [Catherine Edelveis](https://medium.com/@cat.edelveis)
- Artigo: [How to create DTOs with records and MapStruct in Spring Boot](https://medium.com/@cat.edelveis)