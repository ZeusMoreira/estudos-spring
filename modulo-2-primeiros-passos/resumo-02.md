# SQL

Antes de começarmos a realizar o **Spring Data JPA** para mexermos com as operações dos endpoints apresentados no resumo anterior (`@PostMapping`, `@GetMapping`, `@PutMapping` e `@DeleteMapping`), assim como criamos uma classe que representa o **Produto** para podermos gerar um **JSON** com os atributos necessários, criaremos também um script de uma tabela `produtos` em SQL que rodará sempre que nossa aplicação subir, pois estamos utilizando o **H2** como objeto de estudo no momento.  

Isso permitirá que possamos salvar o que o usuário passar no JSON e também consultar, editar e deletar dados reais.

---

### `data.sql`
```sql
CREATE TABLE produto (
    id VARCHAR(255) NOT NULL PRIMARY KEY,
    nome VARCHAR(50) NOT NULL,
    descricao VARCHAR(300),
    preco NUMERIC(18,2)
);
```

Nesse momento, ao subir a aplicação, a tabela **produto** será criada no nosso banco **H2**.

---

# Mapeamento básico JPA (DDL)

Agora com a tabela **produto** criada e a classe `Produto` também, devemos encontrar uma forma de ao receber o JSON com os dados do Produto, utilizar a classe para persistir os dados no banco. Para isso precisamos realizar um **mapeamento JPA**.

---

## Dependência
A dependência que permite fazer esse mapeamento é o starter do **JPA**, presente no `pom.xml` da aplicação.

---

## Entidade
O que faz o JPA entender que a classe `Produto` criada é de fato uma tabela no banco de dados é a annotation `@Entity`.

```java
@Entity
public class Produto {
    // aqui terá os atributos ditados pela regra de negócio do sistema
}
```

---

## Colunas
Mas como o JPA sabe que os atributos da classe serão as colunas da tabela?  
Temos 2 alternativas:  

1. Se o nome dos atributos for **exatamente igual** ao do `data.sql`, o JPA infere automaticamente.  
2. Utilizar a annotation `@Column`.

```java
@Entity
public class Produto {

    @Column
    private String id;

    @Column
    private String nome;

    @Column
    private String descricao;

    @Column
    private String preco;

    // getters e setters (não falaremos de Lombok ainda)
}
```

O `@Column` é desnecessário quando os nomes já batem, mas é útil para fins didáticos.  
Quando forem diferentes, utilizamos o atributo `name`:

```java
@Column(name = "aqui virá o nome da coluna do data.sql")
private String id;
```

---

## Tabela
Assim como `@Column`, temos a annotation `@Table` para mapear a tabela caso o nome não bata.

```java
@Entity
@Table(name = "aqui virá o nome da tabela do data.sql")
public class Produto {
    // atributos...
}
```

---

## Identificador
A annotation `@Id` indica para o JPA qual o atributo que será o identificador (chave primária).

```java
@Entity
public class Produto {

    @Id
    @Column
    private String id;
}
```

---

# Spring Data JPA (DML)

Agora temos no projeto:
- Um **controller** responsável por receber requisições REST.  
- Uma **classe mapeada** com JPA.  
- Um `data.sql` que define a tabela.

Mas... e as operações? Quem faz? Como?

---

## Repository
O **Spring Data JPA** entra aqui.  
A interface `JpaRepository` nos permite realizar operações SQL de forma simples e encapsulada.  

Exemplo:

```java
public interface ProdutoRepository extends JpaRepository<Produto, String> {
}
```

Não é necessário adicionar annotations: por estender `JpaRepository`, o Spring entende que é uma classe gerenciável.

---

# Operações

Agora, com um `Repository`, conseguimos realizar operações de persistência.  
Seguindo o conceito de **injeção de dependências**, podemos injetar o `ProdutoRepository` no `ProdutoController`:

```java
@RestController
@RequestMapping("produtos")
public class ProdutoController {
    private ProdutoRepository produtoRepository;

    public ProdutoController(ProdutoRepository produtoRepository) {
        this.produtoRepository = produtoRepository;
    }

    // métodos @PostMapping, @GetMapping, etc.
}
```

A injeção funciona porque `ProdutoController` também é gerenciado pelo Spring.  
Se fosse uma classe sem `@Component`, `@Service`, `@Repository`, `@Controller` ou `@RestController`, não seria possível.

---

# Nossa primeira operação

Com o `Repository` injetado, podemos usar os métodos do `JpaRepository<Produto, String>`.

```java
@RestController
@RequestMapping("produtos")
public class ProdutoController {
    private ProdutoRepository produtoRepository;

    public ProdutoController(ProdutoRepository produtoRepository) {
        this.produtoRepository = produtoRepository;
    }

    @PostMapping
    public Produto salvar(@RequestBody Produto produto) {
        var id = UUID.randomUUID().toString();
        produto.setId(id);

        produtoRepository.save(produto);
        return produto;
    }
}
```

Como o `id` foi mapeado como obrigatório e não usamos autoincremento, precisamos gerar o `UUID` antes de salvar.  

A operação `save()` é herdada do `JpaRepository` e usa `<Produto, String>` como base.  

Basta chamar o endpoint enviando um JSON **sem o id**, que a persistência ocorrerá.