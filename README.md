## Criando a Aplicação

Para criar uma API REST com o Quarkus usando o https://code.quarkus.io/, você pode seguir os passos a seguir. Certifique-se de ter o Docker instalado em sua máquina para configurar o PostgreSQL como banco de dados.

1. Acesse o https://code.quarkus.io/ e siga estas etapas:

   a. Selecione as extensões necessárias para sua aplicação. Para criar uma API REST com o PostgreSQL, você precisará selecionar as seguintes extensões:

   - RESTEasy (para suporte a REST)
   - Hibernate ORM with Panache (para persistência de dados)
   - PostgreSQL (para suporte ao banco de dados PostgreSQL)

   b. Clique em "Generate your application" para baixar o projeto gerado.

2. Descompacte o arquivo ZIP baixado e navegue até o diretório do projeto gerado.

3. Configure o banco de dados PostgreSQL no arquivo `src/main/resources/application.properties`. Altere as configurações de banco de dados para se adequar às suas credenciais e preferências. Um exemplo de configuração:

   ```
   quarkus.hibernate-orm.database.generation=update
   quarkus.datasource.jdbc.url=jdbc:postgresql://localhost:5432/secitec
   quarkus.datasource.db-kind=postgresql
   quarkus.datasource.username=postgres
   quarkus.datasource.password=postgres
   ```

4. Crie uma entidade JPA (Java Persistence API) para representar os itens que você deseja armazenar no banco de dados. Por exemplo, crie uma classe chamada `Item`:

   ```java
   import javax.persistence.Entity;
   import io.quarkus.hibernate.orm.panache.PanacheEntity;

   @Entity
   public class Item extends PanacheEntity{
       public String nome;
       public String descricao;
   }
   ```

5. Crie um recurso REST para manipular esses itens. Por exemplo, crie uma classe `ItemResource`:

   ```java
   import java.util.List;
   import jakarta.transaction.Transactional;
   import jakarta.ws.rs.Consumes;
   import jakarta.ws.rs.DELETE;
   import jakarta.ws.rs.GET;
   import jakarta.ws.rs.POST;
   import jakarta.ws.rs.PUT;
   import jakarta.ws.rs.Path;
   import jakarta.ws.rs.PathParam;
   import jakarta.ws.rs.Produces;
   import jakarta.ws.rs.core.MediaType;

   @Path("/itens")
   @Produces(MediaType.APPLICATION_JSON)
   @Consumes(MediaType.APPLICATION_JSON)
   public class ItemResource {

   @GET
    public List<Item> listar() {
        return Item.listAll();
    }

    @POST
    @Transactional
    public void salvar(Item item) {
        item.persist();
    }

    @PUT
    @Path("/{id}")
    @Transactional
    public void atualizar(@PathParam("id") Long id, Item item) {
        Item entity = Item.findById(id);
        if (entity != null) {
            entity.nome = item.nome;
            entity.descricao = item.descricao;
        }
    }

    @DELETE
    @Path("/{id}")
    @Transactional
    public void deletar(@PathParam("id") Long id) {
        Item entity = Item.findById(id);
        if (entity != null) {
            entity.delete();
        }
    }
   }
   ```

6. Inicie o PostgreSQL usando Docker (se você ainda não o fez):

```
docker run --name postgresql -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 postgres

docker exec -it postgresql psql -U postgres

CREATE DATABASE secitec;

\c secitec;

CREATE TABLE item (id SERIAL PRIMARY KEY, nome VARCHAR(255), descricao TEXT);

insert into item (nome, descricao) VALUES ('Cafe', 'Um cafezinho bacana.');

insert into item (nome, descricao) VALUES ('Agua', 'Amana, o patrocinador!');

\q
```

7. Adicione a extensao jackson para cuspir JSON:

```
./mvnw quarkus:add-extensions -Dextensions="io.quarkus:quarkus-resteasy-jackson"
```

8. Compile e execute sua aplicação Quarkus:

```
./mvnw compile quarkus:dev
```

8. Sua API REST estará acessível em `http://localhost:8080/itens`. Você pode usar ferramentas como `curl`, `Postman` ou `Insomnia` para testar os endpoints CRUD (Create, Read, Update, Delete) para itens no banco de dados.

Lembre-se de substituir as informações de configuração do banco de dados (URL, usuário e senha) de acordo com suas configurações específicas do PostgreSQL. Este é um exemplo básico de como criar uma API RESTful com Quarkus e PostgreSQL. Você pode personalizar e expandir conforme suas necessidades específicas.

Para testar o CRUD via REST usando o `curl`, execute na linha de comando:

    curl -X GET http://localhost:8080/itens

    curl -X GET http://localhost:8080/itens/1

    curl -X POST -H "Content-Type: application/json" -d '{
    "nome": "Nome do Item",
    "descricao": "Descrição do Item"
    }' http://localhost:8080/itens

    curl -X PUT -H "Content-Type: application/json" -d '{
    "nome": "Um novo cafezis!",
    "descricao": "Modificando o cafezis!"
    }' http://localhost:8080/itens/1

    curl -X DELETE http://localhost:8080/itens/1

## Executando em Container

Para compilar e executar a aplicação Quarkus em um contêiner Docker localmente, siga os passos abaixo:

1. Certifique-se de que você já tenha criado a aplicação Quarkus e configurado o banco de dados PostgreSQL conforme descrito nas etapas anteriores.

2. Abra um terminal e navegue até o diretório raiz do seu projeto Quarkus.

3. Compile sua aplicação Quarkus usando o comando Maven Wrapper (que está incluído por padrão no projeto Quarkus):

```

./mvnw clean package

```

Isso irá criar um arquivo JAR executável da sua aplicação no diretório `target`.

4. Agora, você precisa criar um arquivo Dockerfile para criar uma imagem Docker da sua aplicação Quarkus. Crie um arquivo chamado `Dockerfile` no diretório raiz do seu projeto com o seguinte conteúdo:

```Dockerfile
FROM adoptopenjdk/openjdk11:alpine-jre
WORKDIR /opt/app

COPY target/*-runner.jar app.jar

EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

Esse Dockerfile usa uma imagem base do OpenJDK 11 e copia o JAR executável gerado durante a compilação para dentro do contêiner.

5. Agora, você pode criar a imagem Docker da sua aplicação Quarkus. No terminal, navegue até o diretório do projeto (onde o Dockerfile está localizado) e execute o seguinte comando:

   ```
   docker build -t nome-da-sua-imagem .
   ```

   Substitua `nome-da-sua-imagem` pelo nome que deseja dar à imagem Docker.

6. Após a conclusão da criação da imagem Docker, você pode executar um contêiner com a aplicação Quarkus usando o seguinte comando:

   ```
   docker run -d -p 8080:8080 --name nome-do-seu-container nome-da-sua-imagem
   ```

   Substitua `nome-do-seu-container` pelo nome que deseja dar ao contêiner.

7. Sua aplicação Quarkus agora estará em execução em um contêiner Docker localmente e estará acessível em `http://localhost:8080`.

Lembre-se de que, para que a aplicação se comunique com o banco de dados PostgreSQL, você também precisa garantir que o contêiner Docker do PostgreSQL esteja em execução, conforme as etapas mencionadas anteriormente. Certifique-se de que a configuração de banco de dados no arquivo `application.properties` aponte para o endereço correto do banco de dados PostgreSQL dentro do contêiner Docker.

O `target/*-runner.jar` é uma referência ao arquivo JAR executável gerado quando você compila seu projeto Quarkus. Vou explicar o que cada parte desse caminho significa:

- `target`: É o diretório onde o Maven coloca os artefatos de construção (como arquivos JAR) por padrão durante o processo de compilação.

- `*-runner.jar`: O asterisco (`*`) é um curinga que representa o nome do projeto. O Quarkus gera o arquivo JAR com um nome que inclui o nome do seu projeto Quarkus. Portanto, o curinga é usado para pegar o nome exato do arquivo JAR, que pode variar de projeto para projeto.

Então, quando você vê `target/*-runner.jar`, está se referindo ao arquivo JAR gerado na pasta `target` com o nome específico do seu projeto seguido por `-runner.jar`. Esse arquivo JAR contém a aplicação Quarkus compilada e pode ser executado diretamente para iniciar sua aplicação.

Ao criar o Dockerfile com essa referência, você está copiando esse arquivo JAR para dentro do contêiner Docker para que a aplicação seja executada quando o contêiner for iniciado. O Dockerfile inclui o comando `CMD ["java", "-jar", "app.jar"]`, que inicia a aplicação Java contida no arquivo JAR quando o contêiner é executado.
