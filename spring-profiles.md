# Learning Spring-Boot

## Spring Profiles

### 1. Visão Geral

Os *Spring Profiles* são um recurso central do *framework* - **permitindo-nos mapear nossos *beans* para diferentes perfis**.

### 2. Use *@Profile* em um *Bean*

Usando a `annotation @Profile` - **estamos mapeando o *bean* para esse perfil específico**.

```java
@Component
@Profile("dev")
public class DevDatasourceConfig
```

Os nomes de perfil também podem ser prefixados com *um operador NOT* - por exemplo, `!dev` - para excluí-los de um perfil.

```java
@Component
@Profile("!dev")
public class DevDatasourceConfig
```

Também podemos usar o `&` para o aperador `AND` *e combinar vários perfis*.

```java
@Component
@Profile("a & b & c")
public class ThreeProfilesComponent
```

Neste exemplo, o `ThreeProfilesComponent` é ativado somente se os perfis `a, b e c` estiverem todos ativos.

Podemos combinar os operadores `AND` e `NOT`.

```java
@Component
@Profiles("!a & !b & !c")
public class NoneOfThreeProfilesComponent
```

**O `NoneOfThreeProfilesComponent` será ativado qunado os perfis ativos atuais não forem nenhum dos seguintes: *a, b e c.***

### 3. Perfis em *XML*

Os perfis também podem ser configurados em *XML*.

```xml
<beans profile="dev">
    <bean
        id="devDatasourceConfig"
        class="org.bealdung.profiles.DevDatasourceConfig"
    />
</beans>
```

### 4. Definindo Perfis

O próximo passo é ativar e definir os perfil para que os respectivos *beans* sejam registrado no *container*.

#### 4.1. Via inteface *WebApplicationInitializer*

Em aplicativos web, o `WebApplicationInitializer` pode ser usado para configura o `ServletContext`.

```java
@Configuration
public class MyWebApplicationInitializer
    implements WebApplicationInitializer {

        @Override
        public void onStartup(ServletContext servletContext) throws ServletException {
            servletContext.setInitParameter(
                "spring.profiles.active", "dev"
            );
        }
    }
```

#### 4.2. Via *ConfigurableEnvironment*

Podemos definir perfis diretamente no ambiente:

```java
@Autowired
private COnfigurableEnvironment env;
// ...
env.setActiveProfiles("someProfile");
```

#### 4.3. Parâmetro de contexto em *web.xml*

Podemos definir os perfis ativos no arquivo `web.xml` usando um parâmetro de contexto:

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/app-config.xml</param-value>
</context-param>
<context-param>
    <param-name>spring.profiles.active</param-name>
    <param-value>dev</param-value>
</context-param>
```

#### 4.4. Parâmetro do Sistema *JVM*

Os nomes dos perfis também podem ser passados por meio de um parâmetro da *JVM*.

```
-Dspring.profiles.active=dev
```

#### 4.5. Variável de Ambiente

Em um ambiente *Unix*, **os perfis também podem ser ativados por meio da variável de ambiente**:

```
export spring_profiles_active=dev
```

#### 4.6. Maven

Os perfis do *Spring* também podem ser ativados por meio de perfis Maven **especificando a propriedade de configuração `spring.profiles.active`**.

```xml
<profiles>
    <profile>
        <id>dev</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <spring.profiles.active>dev</spring.profiles.active>
        </properties>
    </profile>
    <profile>
        <id>prod</id>
        <properties>
            <spring.profiles.active>prod</spring.profiles.active>
        </properties>
    </profile>
</profiles>
```

Seu valor será usado para substituir o espaço reservado `@spring.profiles.active@` em `application.properties`:

```
spring.profiles.active=@spring.profiles.active@
```

Precisamos habilitar a filtragem de recursos em `pom.xml`:

```xml
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
    <!-- ... -->
</build>
```

adicionando um parâmtro `-P` para alternar qual perfil será aplicado:

```
mvn clean package -Pprod
```

Este comando empacotará o aplicativo para o perfil *prod*. Ele também aplica o valor `spring.profiles.active prod` para este aplicativo quando ele stiver em execução.

#### 4.7. *@ActiveProfile* em Testes

Os testes facilitam a especificação de quais perfis estão ativos usando a `annotation` `@ActiveProfile` para habilitar perfis específicos:

```java
@ActiveProfiles("dev")
```

- Prioridades se usarmos mais de uma maneira de ativar perfis:
    1. Parâmetro de contexto em `web.xml`
    2. *Inicializador* de *Aplicativo Web*
    3. Parâmetro do sistema *JVM*
    4. Variável de ambiente
    5. Perfil *Maven*


### 5. *Default Profile*

Qualquer *bean* que não especifique um perfil pertente ao *default profile*.

O *Spring* também fornece uma maneira de definir o *"perfil padrão"* quando nenhum outro perfil estiver ativo - *usando a propriedade `spring.profiles.default`*.

### 6. Obtenha Perfis Ativos

Os perfis ativos do *Spring* controlam o comportamento da *"annotation"* `@Profile` para habilitar/desabilitar *beans*.

Também podemos acessar a lista de *perfis ativos*. Tendo duas maneiras de fazer isso, **usando *Environment* ou `spring.profiles.active`**.

#### 6.1. Usando o *Ambiente*

Podemos acessar os perfis ativos do objeto *Environment* injetando-o:

```java
public class ProfileManager {
    @Autowired
    private Environment environment;

    public void getActiveProfiles() {
        for (String profileName : environment.getActiveProfiles()) {
            System.out.printpl("Currently active profile - " + profileName);
        }
    }
}
```

#### 6.2. Usando `spring.profiles.active`

Podemos acessar os perfis injetando a propriedade `spring.profiles.active`:

```java
@Value("${spring.profiles.active}")
private String activeProfile;
```

Nossa variável `activeProfile` **conterá o nome do perfil que está ativo no momento** e, se houver vários, ela conterá seus nomes separados por vírgula.

No entanto, devemos **considerar o que aconteceria se não houvesse nenhum perfil ativo**. Com o código acima, a ausência de um perfil ativo impediria a criação do contexto do aplicativo. Isso resultaria em uma `IllegalArgumentException` devido à ausência do espaço reservado para a injeção na variável.

Podemos evitar **definindo um valor padrão**:

```java
@Value("${spring.profiles.active:}")
private String activeProfile;
```

Se nenhum perfil estiver ativo, nosso `activeProfile` conterá apenas uma string vazia.

E se quisermos acessar a lista deles como no exemplo anterior, podemos fazer isso *dividindo* a variável `activeProfile`:

```java
public class ProfileManager {
    @Value("${spring.profiles.active:}")
    private String activeProfiles;

    public String getActiveProfiles() {
        for (String profileName : activeProfiles.split(",")) {
            System.out.println("Currently active profile - " + profileName);
        }
    }
}
```

### 7. Configurações de *"datasources"* Separadas usando Perfis

#### 7.1. Perfil Único

Considere um cenário em que **precisamos manter a configuração da *"datasource"* para os ambientes de desenvolvimento e produção**.

Vamos criar uma *interface* comum `DatasourceConfig` que precisa ser implemnetada por ambas as implementações de fonte de dados:

```java
public interface DatasourceConfig {
    public void setup();
}
```

Em seguida, vamos escrever a configuração para o ambiente de desenvolvimento:

```java
@Component
@Profile("dev")
public class DevDataSourceConfig implements DatasourceConfig {
    @Override
    public void setup() {
        System.out.println("Setting up datasource for DEV environment.")
    }
}
```

Agora a configuração para o ambiente de produção:

```java
@Component
@Profile("production")
public class ProductionDatasourceConfig implements DatasourceConfig {
    @Override
    public void setup() {
        System.out.println("Setting up datasource for PRODUCTION environment.");
    }
}
```

Vamos criar dois testes e injetar nossa interface *DatasourceConfig*. **Dependendo do perfil ativo, o *Spring* injetará o *bean* `DevDatasourceConfig` ou `ProductionDatasourceConfig`**:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ActiveProfiles("dev")
@ContextConfiguration(classes = { SpringProfilesConfig.class }, loader = 
AnnotationConfigContextLoader.class)
public class DevProfileWithAnnotationIntegrationTest {
    @Autowired
    DatasourceConfig datasourceConfig;

    @Test
    public void testSpringProfiles() {
        Assert.assertTrue(datasourceConfig instanceof DevDatasourceConfig);
    }
}

@RunWith(SpringJUnit4ClassRunner.class)
@ActiveProfiles("production")
@ContextConfigration(classes = { SpringProfilesConfig.class }, loader = 
AnnotationConfigContextLoader.class)
public class ProductionProfileWithAnnotationIntegrationTest {
    @Autowired
    DatasourceConfig datasourceConfig;

    @Test
    public void testSpringProfiles() {
        Assert.assertTrue(datasourceConfig instanceof ProductionDatasourceConfig);
    }
}
```

#### 7.2. Combinando Vários Perfis

Digamos que queremos injetar um *bean* `DatasourceConfig` apenas para o banco de dados *MySQL* e a plataforma *Tomcat*. Podemos definir o *bean* como:

```java
@Component
@Profile("tomcat & mysql")
public class TomcatAndMySQLDatasourceConfig implemnets DatasourceConfig {
    @Override void setup() {
        System.out.println("Setting up MySQL datasource for Tomcat environment.");
    }
}
```

Como mostra o exemplo, **somente se os perfis *tomcat* e *mysql* estiverem ativos, uma instância `TomcatAndMySQLDatasourceConfig` será o *bean* `DatasourceConfig` no *app***.

Em seguida, criamos um teste para verificar isso:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ActiveProfiles({"tomcat", "mysql"})
@ContextConfiguration(classes = { SpringProfilesConfig.class }, loader = 
@AnnotationConfigContextLoader.class)
public class TomcatAndMySQLWithAnnotationIntegrationTest ?{
    @Autowired
    DatasourceConfig datasourceConfig;

    @Test
    public void testSpringProfiles() {
        Assert.assertTrue(datasourceConfig instanceof TomcatAndMySQLDatasourceConfig);
    }
}
```

Neste exemplo, ativamos ambos os perfis usando a *annotation* `@ActiveProfiles`.

Digamos que o perfil ativo no nosso *app Spring* não seja nenhum dos mencionados acima, precisamos injetar um *bean especial* `DatasourceConfig`:

```java
@Component
@Profile("!dev & !prodction & !mysql & !tomcat")
public class SpecialDatasourceConfig implements DatasourceConfig {
    @Override
    public void setup() {
        System.out.println("Setting up a very special datasource.");
    }
}
```

**Usamos o comando `("!a & !b & !c ...")` na anotação `@Profile` para conseguir isso.**

Vamos verificar o comportamento em um teste simples:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ActiveProfiles("something-very-special")
@ContextConfiguration(classes = { SpringProfileConfig.class }, loader = 
AnnotationConfigContextLoader.class)
public class SpecialProfileWithAnnotationIntegrationTest {
    @Autowired
    DatasourceConfig datasourceConfig;

    @Test
    public void testSpringProfiles() {
        Assert.assertTrue(datasourceConfig instanceof SpecialDatasourceConfig);
    }
}
```

Neste teste, ativamos o perfil `something-very-special` e obtivemos o bean `DatasourceConfig` esperado.

### 8. Perfis no Spring Boot

O *Spring Boot* suporta todas as configurações de perfil descritas até agora, com alguns recursos adicionais.

#### 8.1. Ativando ou Definindo um Perfil

O parâmetro de inicialização `spring.profiles.active`, também pode ser configurado como uma propriedade no *Spring Boot* para definir perfis ativos no momento.

```
spring.profiles.active=dev
```

A partir do *Spring Boot 2.4*, essa propriedade não pode ser usada em conjunto com `spring.config.active.on-profile`, pois  isso pode gerar uma `ConfigDataException` *(como uma `InvalidConfigDataPropertyException` ou uma `InactiveConfigDataAccessException`)*.

Para definir perfis programaticamente, também podemos usar a classe `SpringApplication`:

```
SpringApplication.setAdditionalProfiles("dev");
```

Para definir perfis usando *Maven* no *Spring Boot*, podemos especifcar em `spring-boot-maven-plugin` em `pom.xml`:

```xml
<plugins>
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
            <profiles>
                <profile>dev</profile>
            </profiles>
        </configuration>
    </plugin>
</plugins>
```

e executar a o comando:

```
mvn spring-boot:run
```

#### 8.2. Arquivos de Propriedades Especificas do Perfil

O recurso mais importante relacionado a perfis que o *Spring Boot* oferece são **os arquivos de propriedades específicos de perfil**. *Estes devem ser nomeados no formato `application-{profile}.properties`*.

Por exemplo, podemos configurar diferentes *datasources* para perfis de desenvolvimento e produção usando dois arquivos chamados `application-dev.properties` e `application-production.properties`:

No arquivo `application-production.properties`, podemos configurar um banco de dados *MySQL*:

```
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/db
spring.datasource.username=root
spring.datasource.password=root
```

E no arquivo `application-dev.properties`, podemos configurar um banco de dados *H2*:

```
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:mem:db;DB_CLOSE_DELAY=-1
spring.datasource.username=sa
spring.datasource.password=sa
```

#### 8.3. Arquivos *Multi-Files*

*Podemos agrupar todas as propriedades no mesmo arquivo e usar um separador para indicar o perfil*.

O arquivo `application.properties` ficaria assim:

```
my.prop=used-always-in-all-profiles
#---
spring.config.activate.on-profile=dev
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/db
spring.datasource.username=root
spring.datasource.password=root
#---
spring.config.activate.on-profile=production
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:mem:db;DB_CLOSE_DELAY=-1
spring.datasource.username=sa
spring.datasource.possword=sa
```

Este arquivo é lido pelo *Spring Boot* em ordem de cima para baixo. Se alguma propriedade, como `my.prop`, ocorrer novamente no final do exemplo acima, *o valor ao final será considerado*.

#### 8.4. Grupos de Perfis

Como o nome sugere, *eles nos permitem agrugar perfis semelhantes*.

Vamos considerar um caso de uso em que teriamos vários perfis de configuração para o ambiente de produção. Digamos, um *proddb* para o banco de dados e um *prodquartz* para o *scheduler* no *ambiente de produção*.

No nosso arquivo `application.properties` poderiamos especificar:

```
spring.profiles.group.production=proddb,prodquartz
```

## Fonte

- Artigo: [Baeldung](https://www.baeldung.com/spring-profiles)
