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
    import java.util.List;

    @Path("/itens")
    @Produces(MediaType.APPLICATION_JSON)
    @Consumes(MediaType.APPLICATION_JSON)
    public class ItemResource {

    @GET
    public List<Item> listar() {
        return Item.listAll();
    }

    @GET
    @Path("/{id}")
    public Item getItemPorId(@PathParam("id") Long id) {
        return Item.findById(id);
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
docker run -d -it -p 5432:5432 -e POSTGRES_PASSWORD=postgres -e POSTGRES_HOST_AUTH_METHOD=trust --name postgresql postgres


docker exec -it postgresql psql -U postgres

CREATE DATABASE secitec;

\c secitec;

CREATE TABLE item (id SERIAL PRIMARY KEY, nome VARCHAR(255), descricao TEXT);

insert into item (nome, descricao) VALUES ('Cafe', 'Um cafezinho bacana.');

insert into item (nome, descricao) VALUES ('Agua', 'Amana, o patrocinador!');

ALTER SEQUENCE item_seq RESTART WITH 3;

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

    ### ATENÇAO! Vai dar pau!
    curl -X POST -H "Content-Type: application/json" -d '{
    "nome": "Um bebida misteriosa",
    "descricao": "Provavelmente levemente altamente alcoolica"
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

3. Agora, você precisa criar um arquivo Dockerfile para criar uma imagem Docker da sua aplicação Quarkus. Adicione o JIB ao projeto, ele criará o Dockerfile para você:

   ```
   ./mvnw quarkus:add-extension -Dextensions='container-image-jib'
   ```

4. Altere o arquivo `src/main/resources/application.properties`:

   ```
    quarkus.container-image.build=true
    quarkus.container-image.group=xico
    quarkus.container-image.name=secitec
    quarkus.container-image.tag=latest
    quarkus.jib.ports=8080

   ```

5. Compile sua aplicação Quarkus usando o comando Maven Wrapper (que está incluído por padrão no projeto Quarkus):

   ```

   ./mvnw clean package -DskipTests

   ```

A imagem deve aparecer ao fazer:

    docker image ls

6. Após a conclusão da criação da imagem Docker, você pode executar um contêiner com a aplicação Quarkus usando o seguinte comando:

   ```
   docker run -d -it -p 8080:8080 --name secitec_container "xico/secitec"
   ```

7. Sua aplicação Quarkus agora estará em execução em um contêiner Docker localmente e estará acessível em `http://localhost:8080`.

Lembre-se de que, para que a aplicação se comunique com o banco de dados PostgreSQL, você também precisa garantir que o contêiner Docker do PostgreSQL esteja em execução, conforme as etapas mencionadas anteriormente. Certifique-se de que a configuração de banco de dados no arquivo `application.properties` aponte para o endereço correto do banco de dados PostgreSQL dentro do contêiner Docker.
