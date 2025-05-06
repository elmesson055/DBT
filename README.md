# Projeto DBT com Docker Compose

Este projeto utiliza Docker Compose para executar um ambiente DBT conectado a um banco de dados PostgreSQL. Ele também fornece instruções para gerar e servir o site de documentação do DBT.

## Pré-requisitos

*   **Docker Desktop**: Certifique-se de que o Docker Desktop esteja instalado e em execução no seu sistema. Você pode baixá-lo no [site oficial do Docker](https://www.docker.com/products/docker-desktop/). O Docker Compose está incluído no Docker Desktop.

## Estrutura do Projeto

Certifique-se de que o diretório do seu projeto (`c:\Users\*********\Desktop\DBT\`) tenha os seguintes arquivos e diretórios principais:




*   **`dbt_project.yml`**: Este arquivo define o seu projeto dbt. Certifique-se de que esteja configurado corretamente.
*   **`dbt_profiles/profiles.yml`**: Este arquivo contém os perfis de conexão com o banco de dados. O `docker-compose.yml` fornecido define variáveis de ambiente que são usadas pelo `profiles.yml` para se conectar ao contêiner PostgreSQL. Um exemplo de `profiles.yml` é:
    ```yaml
    default:
      target: dev
      outputs:
        dev:
          type: postgres
          host: "{{ env_var('POSTGRES_HOST') }}"
          user: "{{ env_var('POSTGRES_USER') }}"
          password: "{{ env_var('POSTGRES_PASSWORD') }}"
          port: "{{ env_var('POSTGRES_PORT') | int }}"
          dbname: "{{ env_var('POSTGRES_DB') }}"
          schema: public # Ou o schema desejado
          threads: 1
          connect_timeout: 10
    ```
*   **`models/`**: Coloque seus modelos dbt (arquivos SQL ou Python) neste diretório.

## Executando o Ambiente

1.  **Navegue até o Diretório do Projeto**:
    Abra seu terminal ou prompt de comando e navegue até a raiz deste projeto:
    ```bash
    cd c:\Users\*********\Desktop\DBT
    ```

2.  **Inicie os Contêineres**:
    Para construir e iniciar os contêineres DBT e PostgreSQL, execute:
    ```bash
    docker-compose up --build
    ```
    A flag `--build` é recomendada para a primeira execução ou se você fez alterações no `docker-compose.yml` ou nos arquivos do projeto.
    Este comando irá:
    *   Baixar as imagens Docker necessárias.
    *   Construir os serviços definidos no `docker-compose.yml`.
    *   Iniciar o contêiner `postgres`.
    *   Iniciar o contêiner `dbt`, que então executará `dbt docs generate` seguido por `dbt docs serve`.

3.  **Acessando a Documentação do DBT**:
    Assim que os contêineres estiverem em execução e o serviço dbt tiver iniciado o servidor de documentação (você deverá ver uma saída como `Running on http://0.0.0.0:8080/`), você poderá acessar o site da documentação do DBT em seu navegador web em:
    [http://localhost:8080](http://localhost:8080)

## Executando Outros Comandos DBT

Se você quiser executar outros comandos dbt (por exemplo, `dbt run`, `dbt test`), você pode fazê-lo executando-os no serviço `dbt`.

1.  **Abra um novo terminal** (deixe o comando `docker-compose up` em execução se quiser manter o servidor de documentação e o PostgreSQL ativos).
2.  Navegue até o diretório do projeto:
    ```bash
    cd c:\Users\*********\Desktop\DBT
    ```
3.  Use `docker-compose run --rm dbt <seu_comando_dbt>`:
    *   Para executar seus modelos dbt:
        ```bash
        docker-compose run --rm dbt dbt run
        ```
    *   Para executar testes dbt:
        ```bash
        docker-compose run --rm dbt dbt test
        ```
    *   Para compilar seu projeto:
        ```bash
        docker-compose run --rm dbt dbt compile
        ```
    A flag `--rm` garante que o contêiner seja removido após a conclusão do comando, o que é bom para tarefas únicas.

    **Nota**: Se você alterar o `command` no `docker-compose.yml` de volta de servir docs para apenas `["run"]` ou outro comando, então `docker-compose up` executará esse comando diretamente. A configuração atual é otimizada para servir a documentação.

## Parando o Ambiente

Para parar os contêineres em execução:
1.  Vá para o terminal onde `docker-compose up` está em execução.
2.  Pressione `Ctrl+C`.
3.  Para garantir que os contêineres sejam parados e removidos (se você não usou `--rm` para os comandos `run`), você também pode executar:
    ```bash
    docker-compose down
    ```
    Este comando para e remove contêineres, redes e volumes criados por `up`.

## Solução de Problemas

*   **Conflitos de Porta**: Se a porta `8080` (para dbt docs) ou `5432` (para PostgreSQL) já estiver em uso na sua máquina host, você precisará alterar o mapeamento da porta do lado do host no `docker-compose.yml`. Por exemplo, altere `ports: - "8080:8080"` para `ports: - "8081:8080"` para o serviço dbt.
*   **`dbt_project.yml` não encontrado**: Certifique-se de que este arquivo exista na raiz do seu projeto (`c:\Users\*********\Desktop\DBT\`) e esteja corretamente montado no contêiner.
*   **Perfil não encontrado**: Certifique-se de que `dbt_profiles/profiles.yml` exista e esteja configurado corretamente com um perfil que corresponda ao especificado em `dbt_project.yml`.
*   **Problemas de Conexão com o Banco de Dados**: Verifique as credenciais e o host do PostgreSQL em `profiles.yml` (usando variáveis de ambiente) correspondem às configurações em `docker-compose.yml`. Verifique os logs do contêiner `postgres` para quaisquer erros de banco de dados: `docker-compose logs postgres`.