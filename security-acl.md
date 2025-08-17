# Introdução ao Spring Security ACL

## 1. Introdução

O *Access Control List* *(ACL)* é uma lista de permissões anexadas a um objeto. Uma *ACL* especifica quais identidades recebem quais operações em um determinado objeto.

O *Access Control List* do *Spring Security* é **um componente do *Spring* que oferece supoerta à Segurança de Objetos de Domínio**. Simplificando, a *ACL* do *Spring* ajuda a definir permissões para usuários/funções específicos em um único objeto de domínio - em vez de permissões gerais, no nível típico por operação.

Por exemplo, um usuário com a função *Administrador* pode ver e editar todas as mensagens em uma *Caixa de Avisos Central*, mas o usuário normal só pode ver as mensagens, relcionar-se com elas e não pode editá-las. Enquanto isso, outros usuários com a função *Editor* podem ver e editar algumas mensagens específicas.

Portanto, diferentes usuários/funções têm permissões diferentes para cada objeto específico. Nesse caso, o *Spring ACL* é capaz de realizar a tarefa.

## 2. Configuração

### 2.1. Banco de Dados ACL

Para usar o *Spring Security ACL*, precisamos criar quatro tabelas obrigatórias em nosso banco de dados.

A primeira tabela é `ACL_CLASS`, que armazena o nome da classe do objeto de domínio, as colunas incluem:

- `ID`;
- `CLASS`: o nome da classe de objetos de domínio protegidos, por exemplo: `com.baeldung.acl.persistence.entity.NoticeMessage`.

Em segundo lugar, precisamos da tabela `ACL_SID`, que nos permite identificar universalmente qualquer princípio ou autoridade do sistema. A tabela precisa:

- `ID`;
- `SID`: que é o nome de usuário ou nome da função. `SID` significa *Identidade de Segurança*;
- `PRINCIPAL`: *`0` ou `1`*, para indicar que o `SID` correspondente é um principal *(usuário, como mary, mike, jack...)* ou uma autoridade *(função, como `ROLE_ADMIN`, `ROLE_USER`, `ROLE_EDITOR`...)*.

A próxima tabela é `ACL_OBJECT_IDENTITY`, que armazena informações para cada objeto de domínio exclusivo:

- `ID`;
- `OBJECT_ID_CLASS`: define a classe do objeto de domínio, links para a tabela `ACL_CLASS`;
- `OBJECT_ID_IDENTITY`: objetos de domínio podem ser armazenados em diversas tabelas, dependendo da classe. Portanto, este campo armazena a chave primária do objeto de destino;
- `PARENT_OBJECT`: especifica o pai desta *identidade de objeto* dentro desta tabela;
- `OWNER_SID`: `ID` do proprietário do objeto, links para a tabela `ACL_SID`;
- `ENTRIES_INHERITING`: se as entidades *ACL* deste objeto herdam do objeto pai *(as entradas ACL são definidas na tabela `ACL_ENTRY`)*.

Por fim, a permissão individual de armazenamento `ACL_ENTRY` é atribuída a cada `SID` em uma *Identidade de Objeto*:

- `ID`;
- `ACL_OBJECT_IDENTITY`: especifica a identidade do objeto, links para a tabela `ACL_OBJECT_IDENTITY`;
- `ACL_ORDER`: a ordem da entrada atual na lista de *entradas ACL* da *Identidade de Objeto* correspondente;
- `SID`: o `SID` de destino ao qual a permissão é concedida ou negada, links para a tabela `ACL_SID`;
- `MASK`: a máscara de *bits inteira* que representa a permissão real que está sendo *concedida ou negada*;
- `GRANTING`: valor `1` significa concessão, valor `0` significa negação;
- `AUDIT_SUCCESS` e `AUDIT_FAILURE`: para fins de auditoria.

### 2.2. Dependência

Para poder usar o *Spring ACL* em nosso projeto, vamos primeiro definir nossas dependências:

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-acl</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
</dependency>
```

O *Spring ACL* requer um cache para armazenar a *identidade do objeto* e as entradas de *ACL*, então usaremos o `ConcurrentMapCache` fornecido pelo `spring-context`.

Quando não estivermos trabalhando com o *Spring Boot*, precisamos adicionar versões explicitamente. Elas podem ser verificadas em *Maven Central*: [spring-security-acl](https://mvnrepository.com/artifact/org.springframework.security/spring-security-acl), [spring-security-config](https://mvnrepository.com/artifact/org.springframework.security/spring-security-config), [spring-context-support](https://mvnrepository.com/artifact/org.springframework/spring-context-support).

### 2.3. Configuração Relacionada à ACL

Premisamos proteger todos os métodos que retornam objetos de domínio protegidos ou fazer alterações no objeto, habilitando `Global Method Security`:

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
public class AclMethodSecurityConfiguration
    extends GlobalMethodSecurityConfiguration {

        @Autowired
        MethodSecurityExpressionHandler
            defaultMethodSecurityExpressionHandler;

        @Override
        protected MethodSecurityExpressionHandler createExpressionHandler() {
            return defaultMethodSecurityExpressionHandler;
        }
}
```

Vamos também habilitar o *Controle de Acesso Baseado em Expressão* definindo `prePostEnabled` como `true` para usar a *Linguagem de Expressão Spring* *(`SpEL`)*. Além disso, precisamos de um manipulador de expressões com suporte a *ACL*:

```java
@Bean
public MethodSecurityExpressionHandler defaultMethodSecurityExpressionHandler() {
    DefaultMethodSecurityExpressionHandler expressionHandler = new
        DefaultMethodSecurityExpressionHandler();

        AclPermissionEvaluator permissionEvaluator = new AclPermissionEvaluator(aclService());

        expressionHandler.setPermissionEvaluator(permissionEvaluator);

        return expressionHandler;
}
```

Portanto, atribuímos `AclPermissionEvaluator` ao `DefaultMethodSecurityExpressionHandler`. O avaliador precisa de um `MutableAclService` para carregar as configurações de permissão e as definiões de objetos de domínio do banco de dados.

Para simplificar, usamos o `JdbcMutableAclService` fornecido:

```java
@Bean
public JdbcMutableAclService aclService() {
    return new JdbcMutableAclService(dataSource, lookupStrategy(), aclCache());
}
```

Como o próprio nome sugere, o `JdbcMutableAclService` utiliza o `JDBCTemplete` para simplificar o acesso ao banco de dados. Ele precisa de um `DataSource` *(para `JDBCTemplate`)*, `LookupStrategy` *(que fornece uma consulta otimizada ao consultar o banco de dados)* e um `AclCache` *(que armazena em cache entradas de ACL e identidade de objetos)*.

Novamente, para simplificar, usamos `BasicLookupStrategy` e `EhCacheBasedAclCache` fornecidos.

```java
@Autowired
DataSource dataSource;

@Bean
public AclAuthorizationStrategy aclAuthorizationStrategy() {
    return new AclAuthorizationStrategyImpl(new SimpleGrantedAuthority("ROLE_ADMIN"));
}

@Bean
public PermissionGrantingStrategy permissionGrantingStrategy() {
    return new DefaultPermissionGrantingStrategy(new ConsoleAuditLogger());
}

@Bean
public SpringCacheBasedAclCache aclCache() {
    final ConcurrentMapCache aclCache = new ConcurrentMapCache("acl_cache");

    return new SpringCacheBasedAclCache(aclCache, permissionGrantingStrategy(), aclAuthorizationStrategy());
}

@Bean
public LookupStrategy lookupStrategy() {
    return new BasicLookupStrategy(
        dataSource,
        aclCache(),
        aclAuthorizationStrategy(),
        new ConsoleAuditLogger()
    );
}
```

Aqui, a `AclAuthorizationStrategy` é responsável por concluir se um usuário atual possui todas as permissões necessárias em determinados objetos ou não.

Ele precisa do suporte de `PermissionGrantingStrategy`, que define a lógica para determinar se uma permissão é concedida a um `SID` específico.

## 3. Segurança de Método com Spring ACL

Até agora, fizemos as configurações necessárias. Agora podemos aplicar a regra de verificação necessária aos nossos métodos protegidos.

Por padrão, o *Spring ACL* se refere à classe `BasePermission` para todas as permissões disponíveis. Basicamente, temos as permissões de *READ, WRITE, CREATE, DELETE* e *ADMINISTRATION*.

Vamos tentar definir algumas regras de segurança:

```java
@PostFilter("hasPermission(filterObject, 'READ')")
List<NoticeMessage> findAll();

@PostAuthorize("hasPermission(returnObject, 'READ')")
NoticeMessage findById(Integer id);

@PreAuthorize("hasPermission(#noticeMessage, 'WRITE')")
NoticeMessage save(@Param("noticeMessage")NoticeMessage noticeMessage);
```

Após a execução do método `findAll()`, `@PostFilter` será acionado. A regra obrigatória `hasPermission(filterObject, 'READ')` retorna apenas o `NoticeMessage` para as quais o usuário atual tem permissão de `READ`.

Da mesma forma, `@PostAuthorize` é acionado após a execução do método `findById()`. Certifique-se de retornar o objeto `NoticeMessage` somente se o usuário atual tiver permissão de `READ` sobre ele. Caso contrário, o sistema lançará uma `AccessDeniedException`.

Por outro lado, o sistema aciona a anotação *`@PreAuthorize` antes de invocar o método `save()`*. Ele decidirá onde o método correspondente poderá ser executado ou não. Caso contrário, `AccessDeniedException` será lançada.

## 4. Em Ação

Agora, vamos testar todas essas configurações usando o *JUnit*. Usaremos o banco de dados *H2* para manter a configuração o mais simples possível.

Precisamos adicionar:

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>
```

### 4.1. O Cenário

Neste cenário, teremos dois usuários *(manager, hr)* e uma função de usuário *(ROLE_EDITOR)*, então nosso `acl_sid` será:

```sql
INSERT INTO acl_sid (principal, sid) VALUES
    (1, 'manager'),
    (1, 'hr'),
    (0, 'ROLE_EDITOR');
```

Em seguida, precisamos declarar a classe `NoticeMessage` em `acl_class`. E três instâncias da classe `NoticeMessage` serão inseridas em `system_message`.

Além disso, os registros correspondentes para essas 3 instâncias devem ser declaradas em `acl_object_identity`:

```sql
INSERT INTO acl_class (id, class) VALUES
    (1, 'com.baeldung.acl.persistence.entity.NoticeMessage');

INSERT INTO system_message(id,content) VALUES 
    (1,'First Level Message'),
    (2,'Second Level Message'),
    (3,'Third Level Message');

INSERT INTO acl_object_identity 
    (object_id_class, object_id_identity, parent_object, owner_sid, entries_inheriting) 
    VALUES
    (1, 1, NULL, 3, 0),
    (1, 2, NULL, 3, 0),
    (1, 3, NULL, 3, 0);
```

Inicialmente, concedemos permissões de `READ` e `WRITE` no primeiro objeto *(`id = 1`)* ao usuário `manager`. Enquanto isso, qualquer usuário com `ROLE_EDITOR` terá permissões de `READ` em todos os três objetos, mas apenas de `WRITE` no terceiro obejto *(`id = 3`)*. Além disso, o usuário `hr` terá apenas permissão de `READ` no segundo objeto.

Aqui, como usamos a classe `BasePermission` padrão do *Spring ACL* para verificação de permissão, o valor da máscara da permissão `READ` será `1`, e o valor da máscara de permissão `WRITE` será `2`. Nossos dados em `acl_entry` serão:

```sql
INSERT INTO acl_entry 
    (acl_object_identity, ace_order, sid, mask, granting, audit_success, audit_failure) 
    VALUES
    (1, 1, 1, 1, 1, 1, 1),
    (1, 2, 1, 2, 1, 1, 1),
    (1, 3, 3, 1, 1, 1, 1),
    (2, 1, 2, 1, 1, 1, 1),
    (2, 2, 3, 1, 1, 1, 1),
    (3, 1, 3, 1, 1, 1, 1),
    (3, 2, 3, 2, 1, 1, 1);
```

### 4.2. Case de Teste

Primeiro, tentamos chamar o método `findAll`.

Conforme nossa configuração, o método retorna apenas aquelas `NoticeMessage` mas quais o usuário tem permissão de `READ`.

Portanto, esperamos que a lista de resultados contenha penas a primeira mensagem:

```java
@Test
@WithMockUser(username = "manager")
public void
    givenUserManager_whenFindAllMessage_thenReturnFirstMessage() {
    List<NoticeMessage> details = repo.findAll();

    assertNotNull(details);
    assertEquals(1, details.size());
    assertEquals(FIRST_MESSAGE_ID, details.get(0).getId());
}
```

Em seguida, tentamos chamar o mesmo método com qualquer usuário que tenha a função - `ROLE_EDITOR`. Observe que, neste caso, esses usuários têm permissão de `READ` em todos os três objetos.

Portanto, esperamos que a lista de resultados contenha todas as três mensagens:

```java
@Test
@WithMockUser(roles = {"EDITOR"})
public void givenRoleEditor_whenFindAllMessage_thenReturn3Message() {
    List<NoticeMessage> details = repo.findAll();

    assertNotNull(details);
    assertEquals(3, details.size());
}
```

Em seguida, usando o usuário `manager`, tentaremos obter a primeira mensagem pelo `id` e atualizar o seu conteúdo - o que deve funcionar bem:

```java
@Test
@WithMockUser(username = "manager")
public void
    givenUserManager_whenFind1stMessageByIdAndUpdateItsContent_thenOK() {
        NoticeMessage firstMessage = repo.findById(FIRST_MESSAGE_ID);
        assertNotNull(firstMessage);
        assertEquals(FIRST_MESSAGE_ID, firstMessage.getId());

        firstMessage.setContent(EDITTED_CONTENT);
        repo.save(firstMessage);

        NoticeMessage editedFirstMessage = repo.findById(FIRST_MESSAGE_ID);

        assertNotNull(editedFirstMessage);
        assertEquals(FIRST_MESSAGE_ID, editedFirstMessage.getId());
        assertEquals(EDITTED_CONTENT, editedFirstMessage.getContent());
}
```

Mas se qualquer usuário com a função `ROLE_EDITOR` atualizar o conteúdo da primeira mensagem, nosso sistema lançará uma `AccessDeniedException`:

```java
@Test(expected = AccessDeniedException.class)
@WithMockUser(roles = {"EDITOR"})
public void
    givenRoleEditor_whenFind1stMessageByIdAndUpdateContent_thenFail() {
        NoticeMessage firstMessage = repo.findById(FIRST_MESSAGE_ID);

        assertNotNull(firstMessage);
        assertEquals(FIRST_MESSAGE_ID, firstMessage.getId());

        firstMessage.setContent(EDDITED_CONTENT);
        repo.save(firstMessage);
    }
```

Da mesma forma, o usuário `hr` pode encontrar a segunda mensagem pelo `id`, mas não consiguirá atualizá-la:

```java
@Test
@WithMockUser(username = "hr")
public void givenUsernameHr_whenFindMessageById2_thenOK() {
    NoticeMessage secondMessage = repo.findById(SECOND_MESSAGE_ID);
    assertNotNull(secondMessage);
    assertEquals(SECOND_MESSAGE_ID, secondMessage.getId());
}

@Test(expected = AccessDeniedException.class)
@WithMockUser(username = "hr")
public void givenUsernameHr_whenUpdateMessageWithId2_thenFail() {
    NoticeMessage secondMessage = new NoticeMessage();
    secondMessage.setId(SECOND_MESSAGE_ID);
    secondMessage.setContent(EDITTED_CONTENT);
    repo.save(secondMessage);
}
```

## 5. Conclusão

> Neste artigo, abordamos a configuração básica e o uso do *Spring ACL*.

> Como sabemos, o *Spring ACL* exigia tabelas específicas para gerenciar objetos, princípios/autoridades e configurações de permissões. Todas as interações com essas tabelas, especialmente as ações de atualização, devem passar pelo `AclService`.

> Por padrão, estamos restritos à permissão predefinida na classe `BasePermission`.

## Fonte:

- Artigo: [An Introduction to Spring Security ACL](https://www.baeldung.com/spring-security-acl)