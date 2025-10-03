# O que é o start.spring.io

É um site essencial que o Spring disponibiliza para podermos criar um projeto Spring com um template, escolher as dependências e etc.

Quando for criar um projeto, geralmente o tipo de **Project** mais utilizado é o **Maven** e a linguagem **Java**.

Quando for escolher a versão do **Spring Boot** podemos escolher as versões **SNAPSHOT**, onde é possível testar funcionalidades, mas ainda sujeitas a mudanças. Via de regra, escolha uma versão já estável.

Quando for escolher o **Packaging**, via de regra escolha **Jar**, que é o mais moderno, já vem com o servidor Tomcat embutido e facilita muito o uso com Docker. Só escolha **War** se for um projeto legado mesmo.

Em **Dependencies (bibliotecas)** você deve escolher as que serão úteis para seu projeto. Caso for criar uma API é bem provável que a dependency **Spring Web** será útil, assim como o **Lombok**, que ajuda a evitar repetição boba de código, como construtores e getters/setters.  

---

# Um pouco sobre Spring Web, WebServer e REST

Quando adicionamos a dependência do **Spring Web**, conseguimos utilizar a annotation `@RestController` acima de nossa classe.  
Isso permite que criemos métodos dentro da classe com:

- `@GetMapping`
- `@PutMapping`
- `@PostMapping`
- `@DeleteMapping`

Essas annotations permitem que nossos métodos façam requisições HTTP via REST.  
Elas recebem uma string indicando a URL da requisição, por exemplo:

```java
@GetMapping("/produtos")
```

Por padrão, o Spring possui o **webserver Tomcat**, o que nos permite subir a aplicação e acessar uma porta.  
Exemplo: acessar `http://localhost:8080/produtos` para consumir o método acima.

Podemos também, junto da annotation `@RestController`, adicionar:

```java
@RequestMapping("/produtos")
```

Assim, todos os métodos da classe terão a URL base `/produtos`.  
Se colocarmos um `@PostMapping` sem nenhum caminho dentro, o Spring entenderá que a rota será **POST em `/produtos`**.

---

### Pergunta:
> Quando queremos realizar um POST devemos realizar o post de algo, mas o que seria esse algo? Como representar esse algo?

### Resposta:
Quando necessitamos realizar o **POST**, recebemos um **JSON** da requisição contendo algumas informações.  
Para padronizar e definir como esse JSON deve ser montado, podemos criar uma **Classe Java** com os atributos necessários que serão recebidos.

Mais para frente entenderemos um pouco mais sobre regra de negócios e **Spring Data JPA**, que permitirá modelar esses dados.

---

### Exemplo básico:

```java
public class Produto {
    // aqui terá os atributos ditados pela regra de negócio do sistema
}
```

```java
@RestController
@RequestMapping("/produtos")
public class ProdutoController {

    @PostMapping
    public void salvar(@RequestBody Produto produto) {
        // coisas acontecerão aqui...
    }
}
```

O JSON enviado para esse método deverá conter chaves exatamente iguais aos atributos da classe `Produto`.  

Exemplo:

```json
{
    "preco": 100.0
}
```

Caso seja necessário retornar dados na requisição, basta adicionar um retorno ao método, e por padrão o Spring retornará em **JSON**.

---

# application.properties / application.yml

As dependências adicionadas no `pom.xml` muitas vezes podem precisar de configuração.  
Para isso, o Spring disponibiliza os arquivos:

- `application.properties`
- `application.yml`

Por padrão, recebemos dentro desse arquivo algo como:

```properties
spring.application.name="nome da aplicação definido quando estavamos no start.spring.io"
```

A diferença entre os dois formatos é **apenas a sintaxe**. Ambos têm o mesmo propósito.

---

# E o banco de dados?

No `application.yml` podemos definir parâmetros de conexão com banco de dados, como:

```yaml
spring:
    application:
        name: Aqui vai o nome da sua aplicação
    datasource:
        url: url que varia de acordo com a dependência de banco adicionada
        username: nome de usuario
        password: senha
```

### Exemplo com banco em memória **H2**:

```yaml
spring:
    application:
        name: Aqui vai o nome da sua aplicação
    datasource:
        url: jdbc:h2:mem:produtos
        username: sa
        password: password
```

Explicando:
- **`jdbc:`** → obrigatório para fazer o Java se conectar ao banco de dados.  
- **`h2:`** → driver do banco.  
- **`mem:`** → indica que é um banco em memória RAM (para testes).  

Podemos também habilitar o console do H2 para acessá-lo no navegador:

```yaml
spring:
    application:
        name: Aqui vai o nome da sua aplicação
    datasource:
        url: jdbc:h2:mem:produtos
        username: sa
        password: password
    h2:
        console:
            enabled: true
            path: /h2-console
```

---

# Sobre Spring Data JPA

A biblioteca responsável por facilitar operações SQL (DDL e DML) é o **Spring Data JPA**.  
Ele funciona com qualquer banco SQL e abstrai muitas operações repetitivas.

---

### Por que citar o JPA agora?

Quando temos uma API, geralmente o intuito é **receber, buscar e enviar dados**.  
Como utilizaremos o **H2**, é necessário indicar para o **Spring Data JPA** qual **dialeto** do banco usar, já que cada SQL tem suas particularidades.

No `application.yml` fica assim:

```yaml
spring:
    application:
        name: Aqui vai o nome da sua aplicação
    datasource:
        url: jdbc:h2:mem:produtos
        username: sa
        password: password
    jpa:
        database-platform: org.hibernate.dialect.H2Dialect
    h2:
        console:
            enabled: true
            path: /h2-console
```
