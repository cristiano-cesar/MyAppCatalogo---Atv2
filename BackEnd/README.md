# Catálogo de Produtos — Back-end

API REST para gerenciamento de produtos de tecnologia, desenvolvida com Node.js, TypeScript, Express, Prisma ORM, PostgreSQL e Zod.

## 1. Tecnologias utilizadas

- **Node.js:** ambiente de execução JavaScript.
- **TypeScript:** tipagem estática para o desenvolvimento.
- **Express:** criação dos endpoints da API REST.
- **PostgreSQL 15:** banco de dados relacional.
- **Docker Compose:** execução do banco de dados em contêiner.
- **Prisma ORM:** acesso ao banco de dados e gerenciamento de migrações.
- **Zod:** validação dos dados recebidos pela API.
- **tsx:** execução dos arquivos TypeScript durante o desenvolvimento.
- **Insomnia:** testes das requisições HTTP.

## 2. Pré-requisitos

Antes de executar o projeto, instale:

- Node.js e npm.
- Docker Desktop, com o mecanismo Docker em execução.
- Visual Studio Code ou outro editor de código.
- Insomnia, para testar os endpoints.

## 3. Estrutura do projeto

```text
BackEnd/
├── prisma/
│   ├── migrations/
│   ├── schema.prisma
│   └── seed.ts
├── src/
│   ├── schemas/
│   │   └── produtoSchema.ts
│   ├── prisma.ts
│   └── server.ts
├── .env
├── .gitignore
├── docker-compose.yml
├── package.json
├── package-lock.json
├── prisma.config.ts
└── README.md
```

A pasta `prisma/migrations/` armazena o histórico das alterações do banco de dados. O arquivo `seed.ts` cadastra os produtos iniciais. A pasta `src/` contém o servidor Express, a conexão com o Prisma Client e os esquemas de validação.

## 4. Instalação das dependências

Abra o terminal na pasta do projeto:

```powershell
cd D:\MyAppCatalogo\BackEnd
```

Instale as dependências:

```powershell
npm install
```

## 5. Configuração das variáveis de ambiente

Crie um arquivo `.env` na raiz do projeto, caso ainda não exista.

Configure as variáveis de acordo com o `docker-compose.yml`:

```dotenv
POSTGRES_USER=catalogo_user
POSTGRES_PASSWORD=SUBSTITUA_POR_UMA_SENHA_LOCAL
POSTGRES_DB=catalogo_db

DATABASE_URL="postgresql://catalogo_user:SUBSTITUA_POR_UMA_SENHA_LOCAL@localhost:5432/catalogo_db?schema=public"

ALLOWED_ORIGIN="http://localhost:5173"
```

Substitua o texto de exemplo da senha pela mesma senha nos dois locais. A variável `DATABASE_URL` deve corresponder às credenciais configuradas para o PostgreSQL.

A variável `ALLOWED_ORIGIN` define a origem do front-end autorizada a acessar a API por meio do CORS. Neste projeto, foi configurada para `http://localhost:5173`.

**Importante:** não publique o arquivo `.env` em repositórios públicos. Mantenha-o no `.gitignore` e disponibilize apenas um exemplo sem credenciais reais.

## 6. Inicialização do PostgreSQL com Docker

Com o Docker Desktop em execução, inicialize o banco:

```powershell
docker compose up -d
```

Confira o estado do contêiner:

```powershell
docker compose ps
```

O serviço PostgreSQL deve aparecer como ativo e saudável (`healthy`).

O banco utiliza a porta `5432` do computador e um volume persistente para preservar os dados.

Para interromper os contêineres sem remover os dados:

```powershell
docker compose down
```

Para iniciar novamente:

```powershell
docker compose up -d
```

## 7. Configuração do Prisma e migrações

Valide o esquema do banco:

```powershell
npx prisma validate
```

Gere o Prisma Client:

```powershell
npx prisma generate
```

Para aplicar as migrações existentes em um ambiente configurado:

```powershell
npx prisma migrate deploy
```

Durante o desenvolvimento, quando houver alterações no esquema que precisem de uma nova migração, utilize:

```powershell
npm run prisma:migrate -- --name nome_da_migracao
```

Para abrir a interface de consulta do banco de dados:

```powershell
npm run prisma:studio
```

As migrações registram e aplicam as alterações da estrutura do banco conforme o esquema definido em `prisma/schema.prisma`.

## 8. Cadastro dos produtos iniciais

Para inserir os produtos iniciais no banco de dados, execute:

```powershell
npm run seed
```

O seed cadastra cinco produtos de tecnologia, como notebook, mouse, teclado, monitor e SSD.

O procedimento utiliza o modelo `Produto` do Prisma e evita duplicar os produtos iniciais quando executado novamente, desde que os nomes permaneçam iguais.

## 9. Execução da API

Inicie o servidor em modo de desenvolvimento:

```powershell
npm run dev
```

A API estará disponível em:

`http://localhost:3001`

Para verificar se o servidor está funcionando, acesse:

`http://localhost:3001/`

Resposta esperada:

```json
{
  "status": "ok"
}
```

## 10. Endpoints disponíveis

A API utiliza JSON para receber e retornar os dados dos produtos.

| Método | Endpoint | Descrição |
|---|---|---|
| GET | `/` | Verifica o estado da API |
| GET | `/api/products` | Lista os produtos |
| GET | `/api/products/:id` | Consulta um produto por ID |
| POST | `/api/products` | Cadastra um produto |
| PUT | `/api/products/:id` | Atualiza um produto |
| DELETE | `/api/products/:id` | Exclui um produto |

### 10.1. Listar produtos

**Requisição:**

```http
GET http://localhost:3001/api/products
```

Retorna a lista de produtos cadastrados.

### 10.2. Consultar um produto

**Requisição:**

```http
GET http://localhost:3001/api/products/1
```

Retorna o produto correspondente ao ID informado.

- `200 OK`: produto encontrado.
- `400 Bad Request`: identificador inválido.
- `404 Not Found`: produto não encontrado.
- `500 Internal Server Error`: erro interno do servidor.

### 10.3. Cadastrar um produto

**Requisição:**

```http
POST http://localhost:3001/api/products
Content-Type: application/json
```

Corpo da requisição:

```json
{
  "nome": "Webcam Logitech C920",
  "descricao": "Webcam Full HD para videoconferências e aulas on-line.",
  "preco": 349.90,
  "imagemUrl": "https://exemplo.com/webcam.jpg"
}
```

Em caso de sucesso, a API retorna `201 Created` e os dados do produto criado, incluindo o identificador gerado pelo banco.

Validações aplicadas:

- `nome`: texto com pelo menos 3 caracteres.
- `descricao`: texto com pelo menos 10 caracteres.
- `preco`: número maior que zero.
- `imagemUrl`: campo opcional; quando informado, não pode ser uma string vazia.

O nome do produto deve ser único. Dados inválidos resultam em `400 Bad Request`, e um nome duplicado resulta em `409 Conflict`.

### 10.4. Atualizar um produto

**Requisição:**

```http
PUT http://localhost:3001/api/products/1
Content-Type: application/json
```

Corpo da requisição:

```json
{
  "nome": "Webcam Logitech C920 Atualizada",
  "preco": 329.90
}
```

A atualização é parcial: podem ser enviados somente os campos que precisam ser alterados.

Em caso de sucesso, a API retorna `200 OK` e o produto atualizado.

- `400 Bad Request`: ID ou dados inválidos, ou corpo sem campos para atualização.
- `404 Not Found`: produto não encontrado.
- `409 Conflict`: conflito com um nome já cadastrado.
- `500 Internal Server Error`: erro interno do servidor.

### 10.5. Excluir um produto

**Requisição:**

```http
DELETE http://localhost:3001/api/products/1
```

Em caso de sucesso, a API retorna `204 No Content`, sem corpo na resposta.

Se o produto não existir, retorna `404 Not Found`.

## 11. Validação de dados com Zod

O projeto utiliza o Zod para validar os dados recebidos nas operações de criação e atualização.

A validação verifica o tamanho mínimo do nome e da descrição, além de exigir que o preço seja positivo. O esquema de atualização utiliza `produtoSchema.partial()`, permitindo alterações parciais.

As validações são realizadas antes da gravação no banco de dados, e as requisições inválidas recebem `400 Bad Request`.

## 12. Configuração do CORS

O CORS (*Cross-Origin Resource Sharing*) é configurado pelo middleware `cors` do Express.

A origem permitida é definida pela variável `ALLOWED_ORIGIN`, cujo valor padrão no servidor é `http://localhost:5173`.

A configuração permite os métodos HTTP `GET`, `POST`, `PUT` e `DELETE`, além do cabeçalho `Content-Type`, necessário para requisições que enviam dados JSON.

Essa configuração permite que o front-end local acesse a API, restringindo as origens autorizadas conforme a configuração do servidor.

## 13. Testes com o Insomnia

Utilize a URL-base:

`http://localhost:3001`

Verifique os seguintes cenários:

- Consulta do estado da API.
- Listagem de produtos.
- Consulta de produto existente.
- Consulta com ID inválido.
- Consulta com ID inexistente.
- Cadastro de produto válido.
- Cadastro com dados inválidos.
- Cadastro com nome duplicado.
- Atualização parcial de produto existente.
- Atualização com preço negativo.
- Atualização de produto inexistente.
- Exclusão de produto existente.
- Exclusão de produto inexistente.

### Principais códigos HTTP

| Código | Significado |
|---|---|
| `200 OK` | Requisição concluída com sucesso |
| `201 Created` | Recurso criado |
| `204 No Content` | Exclusão concluída sem corpo na resposta |
| `400 Bad Request` | Dados ou identificador inválidos |
| `404 Not Found` | Produto não encontrado |
| `409 Conflict` | Conflito, como nome duplicado |
| `500 Internal Server Error` | Erro interno do servidor |

## 14. Observações

- O banco de dados é executado localmente por meio do Docker Compose.
- O Prisma Client é gerado a partir do esquema do Prisma.
- O campo `preco` utiliza o tipo `Decimal` para representar valores monetários.
- O campo `imagemUrl` permite armazenar o endereço de uma imagem; o upload de arquivos não faz parte da implementação atual.
- O arquivo `.env` contém configurações locais e não deve ser publicado com credenciais reais.

## 15. Licença

Projeto desenvolvido para fins acadêmicos e de aprendizagem em desenvolvimento de APIs REST.
