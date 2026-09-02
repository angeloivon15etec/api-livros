[Read.md](https://github.com/user-attachments/files/31757982/Read.md)

# API da Loja — Turma 3F

API REST desenvolvida em **Python**, utilizando **FastAPI**, **SQLAlchemy**, **PyMySQL** e **MySQL/MariaDB**. O projeto foi produzido nas aulas do 3º bimestre de Sistemas Web II e implementa operações CRUD para dois recursos: **produtos** e **professores**.

A aplicação recebe requisições HTTP, valida os dados recebidos, consulta ou altera registros no banco de dados e devolve respostas em JSON. O projeto também possui testes automatizados com `pytest`, `TestClient` e `MagicMock`, permitindo testar a API sem depender de uma conexão real com o MySQL.

> **Fluxo principal:** cliente → requisição HTTP → rota FastAPI → dependência `get_db` → sessão SQLAlchemy → MySQL → schema Pydantic → resposta JSON.

## Conteúdo

| Seção | Descrição |
|---|---|
| [Objetivo](#objetivo) | O que a API faz. |
| [Tecnologias](#tecnologias) | Bibliotecas e ferramentas usadas. |
| [Arquitetura](#arquitetura-do-projeto) | Função de cada arquivo. |
| [Instalação](#instalação-no-windows) | Preparação do ambiente virtual. |
| [Banco de dados](#configuração-do-banco-de-dados) | Criação e importação do banco `loja`. |
| [Execução](#executando-a-api) | Inicialização com Uvicorn. |
| [Rotas](#rotas-da-api) | Endpoints de produtos e professores. |
| [Testes](#testes-automatizados) | Execução e cobertura atual. |
| [Frontend](#consumo-pelo-frontend) | Como consumir a API em HTML/JavaScript. |
| [Correções importantes](#correções-importantes-no-código-atual) | Ajustes necessários antes de executar. |
| [Conceitos](#conceitos-principais) | Resumo para estudo e prova. |

## Objetivo

O objetivo do projeto é demonstrar como construir uma API REST conectada a um banco de dados relacional. A aplicação possui duas entidades:

| Recurso | Tabela | Campos |
|---|---|---|
| Produto | `produtos` | `id`, `nome`, `preco`, `quantidade` |
| Professor | `professores` | `id`, `nome`, `email`, `materia`, `idade` |

Cada recurso possui endpoints para listar, consultar por ID, criar, atualizar e remover registros. Os dados são persistidos no MySQL/MariaDB, portanto não são perdidos quando o servidor é reiniciado.

## Tecnologias

| Tecnologia | Função no projeto |
|---|---|
| Python | Linguagem de programação. |
| FastAPI | Framework utilizado para criar a API e registrar as rotas. |
| Uvicorn | Servidor ASGI que executa a aplicação. |
| SQLAlchemy | ORM que representa tabelas como classes e registros como objetos Python. |
| PyMySQL | Driver utilizado pelo SQLAlchemy para conversar com o MySQL. |
| MySQL/MariaDB | Banco de dados relacional. |
| Pydantic | Validação dos dados de entrada e saída. |
| pytest | Execução dos testes automatizados. |
| `unittest.mock.MagicMock` | Simulação da sessão do banco nos testes. |
| Swagger/OpenAPI | Documentação interativa gerada em `/docs`. |
| HTML/JavaScript | Frontend simples para consumir a API. |

## Arquitetura do projeto

```text
api-loja/
├── venv/                 # Ambiente virtual Python; não deve ser versionado
├── main.py               # Aplicação FastAPI e endpoints
├── database.py           # Engine, sessões e dependência get_db
├── models.py             # Modelos ORM das tabelas
├── schemas.py            # Schemas Pydantic de entrada e saída
├── test_produtos.py      # Testes automatizados da API
├── loja.sql              # Banco, tabelas e dados iniciais
├── relatorio.html        # Relatório gerado pelo pytest-html
├── requirements.txt      # Dependências do projeto
└── README.md             # Esta documentação
```

### Responsabilidade de cada arquivo

`database.py` configura a URL do banco, cria o `engine`, cria a fábrica de sessões e define `get_db`, que entrega uma sessão para cada requisição e garante seu fechamento.

`models.py` contém as classes `ProdutoDB` e `ProfessoresDB`. Cada classe herda de `Base` e representa uma tabela do banco por meio do SQLAlchemy.

`schemas.py` contém os modelos Pydantic usados para validar os dados recebidos e formatar os dados devolvidos pela API. O schema de criação não exige `id`, pois o banco gera esse valor; o schema de resposta inclui `id`.

`main.py` cria o objeto `app`, configura CORS, registra as rotas e implementa as operações CRUD.

`test_produtos.py` cria requisições internas com `TestClient` e substitui `get_db` por um `MagicMock`, evitando a necessidade de um MySQL ligado durante os testes.

## Instalação no Windows

Abra o Prompt de Comando dentro da pasta do projeto:

```bat
cd /d C:\caminho\para\api-loja
```

Crie o ambiente virtual:

```bat
python -m venv venv
```

Ative-o:

```bat
venv\Scripts\activate
```

Quando o ambiente estiver ativo, o terminal exibirá `(venv)` no início da linha. Confirme qual Python está sendo utilizado:

```bat
where python
python --version
```

O primeiro caminho deve apontar para:

```text
...\api-loja\venv\Scripts\python.exe
```

Instale todas as dependências:

```bat
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Se ainda não houver um `requirements.txt` correto, instale diretamente:

```bat
python -m pip install fastapi uvicorn sqlalchemy pymysql pydantic httpx pytest pytest-html
```

Use `python -m pip` e `python -m pytest` para garantir que os pacotes sejam instalados e executados no mesmo ambiente Python.

## Configuração do banco de dados

O projeto usa a URL abaixo como configuração padrão:

```text
mysql+pymysql://root:@localhost/loja
```

A URL é composta por:

```text
mysql+pymysql://usuario:senha@servidor/nome_do_banco
```

Neste projeto, `mysql` identifica o banco, `pymysql` identifica o driver, `root` é o usuário, `localhost` é o servidor local e `loja` é o nome do banco.

### Criar o banco

Abra o MySQL Workbench, o cliente MySQL ou o MariaDB e execute:

```sql
CREATE DATABASE loja;
```

O arquivo `loja.sql` já contém a criação do banco, as tabelas, dados iniciais, chaves primárias e configuração de auto incremento. Para importar o arquivo pelo terminal:

```bat
mysql -u root -p < loja.sql
```

Se o comando `mysql` não estiver no PATH, importe o arquivo pelo MySQL Workbench.

### Conferir as tabelas

```sql
USE loja;
SHOW TABLES;
DESCRIBE produtos;
DESCRIBE professores;
SELECT * FROM produtos;
SELECT * FROM professores;
```

A tabela `produtos` possui `id`, `nome`, `preco` e `quantidade`. A tabela `professores` possui `id`, `nome`, `email`, `materia` e `idade`.

### Segurança da senha

Não coloque senhas reais no README nem publique credenciais no GitHub. Em um projeto mais seguro, a URL deve ser lida por variável de ambiente, por exemplo:

```bat
set DATABASE_URL=mysql+pymysql://root:SUA_SENHA@localhost/loja
```

O código pode usar essa configuração com `os.getenv("DATABASE_URL", URL_PADRAO)`. Em produção, também é necessário restringir CORS, usar usuário de banco com permissões limitadas e armazenar segredos fora do código.

## Executando a API

Com o ambiente virtual ativo e o MySQL ligado, execute:

```bat
python -m uvicorn main:app --reload
```

A expressão `main:app` significa que o Uvicorn deve importar o arquivo `main.py` e procurar nele a variável chamada `app`. O parâmetro `--reload` reinicia o servidor automaticamente quando um arquivo do projeto é alterado.

A API ficará disponível em:

| Endereço | Função |
|---|---|
| `http://127.0.0.1:8000` | Endereço base do servidor. |
| `http://127.0.0.1:8000/docs` | Swagger UI interativo. |
| `http://127.0.0.1:8000/redoc` | Documentação alternativa ReDoc. |
| `http://127.0.0.1:8000/openapi.json` | Contrato OpenAPI em JSON. |

Acesse `/docs` para testar as rotas sem Postman. No Swagger, clique em **Try it out**, preencha os dados e clique em **Execute**.

## Rotas da API

### Produtos

| Método | Rota | Status de sucesso | Função |
|---|---|---:|---|
| `GET` | `/produtos` | `200` | Lista todos os produtos. |
| `GET` | `/produtos/{produto_id}` | `200` | Busca um produto pelo ID. |
| `POST` | `/produtos` | `201` | Cria um produto. |
| `PUT` | `/produtos/{produto_id}` | `200` | Atualiza todos os dados de um produto. |
| `DELETE` | `/produtos/{produto_id}` | `204` | Remove um produto. |

### Professores

| Método | Rota | Status de sucesso | Função |
|---|---|---:|---|
| `GET` | `/professores` | `200` | Lista todos os professores. |
| `GET` | `/professores/{professores_id}` | `200` | Busca um professor pelo ID. |
| `POST` | `/professores` | `201` | Cria um professor. |
| `PUT` | `/professores/{professores_id}` | `200` | Atualiza todos os dados de um professor. |
| `DELETE` | `/professores/{professores_id}` | `204` | Remove um professor. |

### Corpo do POST de produtos

```json
{
  "nome": "Teclado",
  "preco": 89.90,
  "quantidade": 15
}
```

Resposta esperada:

```json
{
  "id": 1,
  "nome": "Teclado",
  "preco": 89.9,
  "quantidade": 15
}
```

### Corpo do POST de professores

```json
{
  "nome": "Ana Silva",
  "email": "ana@example.com",
  "materia": "Sistemas Web II",
  "idade": 35
}
```

Resposta esperada:

```json
{
  "id": 1,
  "nome": "Ana Silva",
  "email": "ana@example.com",
  "materia": "Sistemas Web II",
  "idade": 35
}
```

### Corpo do PUT

O PUT recebe os campos completos do recurso. Para produto:

```json
{
  "nome": "Teclado mecânico",
  "preco": 149.90,
  "quantidade": 10
}
```

Para professor:

```json
{
  "nome": "Ana Silva",
  "email": "ana.silva@example.com",
  "materia": "Sistemas Web II",
  "idade": 36
}
```

### Erro de registro inexistente

Quando o ID não existe, a API deve responder `404`:

```json
{
  "detail": "Produto não encontrado"
}
```

ou:

```json
{
  "detail": "Professor não encontrado"
}
```

## Testando com cURL

Listar produtos:

```bat
curl http://127.0.0.1:8000/produtos
```

Buscar produto por ID:

```bat
curl http://127.0.0.1:8000/produtos/1
```

Criar produto no Windows CMD:

```bat
curl -X POST http://127.0.0.1:8000/produtos -H "Content-Type: application/json" -d "{\"nome\":\"Mouse\",\"preco\":59.90,\"quantidade\":20}"
```

Atualizar produto:

```bat
curl -X PUT http://127.0.0.1:8000/produtos/1 -H "Content-Type: application/json" -d "{\"nome\":\"Mouse sem fio\",\"preco\":79.90,\"quantidade\":18}"
```

Excluir produto:

```bat
curl -X DELETE http://127.0.0.1:8000/produtos/1
```

No PowerShell, pode ser mais simples usar `Invoke-RestMethod` ou executar os comandos cURL com aspas adaptadas ao shell utilizado.

## Testes automatizados

Execute os testes com:

```bat
python -m pytest -v
```

O parâmetro `-v` significa **verbose** e mostra o nome de cada teste.

O arquivo atual `test_produtos.py` testa principalmente:

| Teste | O que verifica |
|---|---|
| `test_listar_produtos_com_mock` | GET `/produtos`, status `200` e nome retornado. |
| `test_criar_produto_com_mock` | POST `/produtos`, status `201`, chamada de `add` e chamada de `commit`. |

A sessão é simulada com `MagicMock`:

```python
db_mock = MagicMock()
app.dependency_overrides[get_db] = lambda: db_mock
```

Isso faz o FastAPI utilizar o mock no lugar de `get_db`. A documentação do FastAPI recomenda registrar a dependência original como chave e uma função substituta como valor [1].

No teste de listagem, a cadeia do mock é configurada assim:

```python
db_mock.query.return_value.all.return_value = [produto]
```

A leitura mental é: `query()` retorna um objeto que possui `all()`, e `all()` retorna a lista simulada.

No teste de criação, o `refresh` recebe um `side_effect` para imitar a geração do ID pelo banco:

```python
def simular_refresh(produto):
    produto.id = 1

db_mock.refresh.side_effect = simular_refresh
```

Sem isso, o objeto poderia permanecer com `id=None`, mas o `ProdutoResponse` exige `id` inteiro.

Ao final de cada teste, limpe os overrides:

```python
app.dependency_overrides.clear()
```

Sem essa limpeza, o mock de um teste pode afetar o teste seguinte.

### Relatório HTML

Instale o plugin, se necessário:

```bat
python -m pip install pytest-html
```

Gere o relatório:

```bat
python -m pytest -v --html=relatorio.html --self-contained-html
```

Abra `relatorio.html` no navegador. Para cobertura de linhas:

```bat
python -m pip install pytest-cov
python -m pytest --cov=main --cov-report=html -v
```

A pasta `htmlcov` mostra quais linhas foram executadas. A suíte atual não cobre todas as rotas: faltam testes específicos para PUT, DELETE, busca por ID e os endpoints de professores.

## Consumo pelo frontend

A API pode ser consumida por uma página HTML usando JavaScript:

```html
<script>
  fetch('http://127.0.0.1:8000/produtos')
    .then(resposta => resposta.json())
    .then(produtos => {
      produtos.forEach(produto => {
        console.log(produto.nome, produto.preco);
      });
    })
    .catch(erro => console.error('Erro ao buscar produtos:', erro));
</script>
```

O `fetch` faz a requisição, `resposta.json()` converte o corpo para um objeto JavaScript, `forEach` percorre os produtos e `catch` trata falhas. O CORS configurado no FastAPI permite que o frontend esteja em outra origem durante o desenvolvimento.

Para tabelas dinâmicas, o frontend deve obter as chaves dos objetos retornados em vez de fixar somente `nome`, `preco` e `quantidade`. Assim, ele também consegue exibir novos recursos da API.

## Correções importantes no código atual

Antes de executar a versão atual do projeto, aplique estas correções em `main.py`.

### 1. Remover a importação circular

Remova do `main.py`:

```python
from main import app, get_db
```

Essa linha pertence ao `test_produtos.py`. Dentro do próprio `main.py`, ela faz o módulo importar a si mesmo e impede o Uvicorn de encontrar `app`.

O teste pode manter:

```python
from main import app, get_db
```

### 2. Importar `HTTPException`

Como as rotas usam `HTTPException`, o import deve ser:

```python
from fastapi import FastAPI, Depends, HTTPException
```

### 3. Corrigir a capitalização de `ProfessoresDB`

Python diferencia letras maiúsculas e minúsculas. A função atual usa `professoresDB`, mas a classe foi declarada como `ProfessoresDB`.

Troque:

```python
return db.query(ProfessoresDB).filter(professoresDB.id == professores_id).first()
```

por:

```python
return db.query(ProfessoresDB).filter(ProfessoresDB.id == professores_id).first()
```

### 4. Atualizar `.dict()` em Pydantic v2

O código atual usa:

```python
produto.dict()
```

Em Pydantic v2, prefira:

```python
produto.model_dump()
```

O mesmo vale para `Professores.model_dump()`.

### 5. Não devolver corpo com status 204

As rotas DELETE estão declaradas com status `204`, mas retornam o objeto excluído. Uma resposta 204 deve ser sem corpo. Remova o `return produto` e o `return professores`, ou use explicitamente:

```python
from fastapi import Response

return Response(status_code=204)
```

### 6. Controlar a criação das tabelas

A instrução abaixo tenta acessar o banco imediatamente quando o módulo é importado:

```python
Base.metadata.create_all(bind=engine)
```

Isso pode impedir o pytest de iniciar quando o MySQL está desligado. A criação deve ficar em uma função de inicialização controlada, executada somente quando a API for iniciada com banco disponível. Testes que usam mock não devem exigir que o MySQL esteja rodando.

## Conceitos essenciais

### API REST

API é uma interface para comunicação entre aplicações. REST organiza essa comunicação por recursos, URLs e métodos HTTP. No projeto, `produtos` e `professores` são recursos.

### Injeção de dependência

`Depends(get_db)` informa ao FastAPI que o parâmetro `db` deve ser fornecido pela função `get_db`. O endpoint recebe a sessão pronta e não precisa conhecer os detalhes de sua criação.

### CRUD

CRUD é a sigla para Create, Read, Update e Delete. Na API, POST representa criação, GET representa leitura, PUT representa atualização e DELETE representa remoção.

### ORM

ORM permite usar classes e objetos Python para representar tabelas e registros relacionais. `ProdutoDB` representa a tabela `produtos`; um objeto `ProdutoDB` representa uma linha.

### Sessão SQLAlchemy

A sequência de criação é:

```text
objeto ORM → db.add() → db.commit() → db.refresh()
```

`add` adiciona o objeto à sessão, `commit` confirma a transação e `refresh` atualiza o objeto com os dados do banco, como o ID gerado.

### Model e schema

O model representa a estrutura persistida no banco. O schema representa o formato de entrada ou saída da API. Eles podem possuir campos parecidos, mas têm responsabilidades diferentes.

### Teste unitário com mock

O mock substitui a sessão real. O teste verifica o comportamento da rota sem gravar no MySQL. `return_value` define retornos fixos, `side_effect` executa um comportamento e `assert_called_once` verifica uma chamada.

## Checklist de execução

Antes de iniciar:

- O ambiente `venv` está ativado.
- `where python` aponta para `venv\Scripts\python.exe`.
- As dependências foram instaladas com `python -m pip install -r requirements.txt`.
- O MySQL/MariaDB está ligado para executar a API real.
- O banco `loja` foi criado ou o arquivo `loja.sql` foi importado.
- A linha de importação circular foi removida do `main.py`.
- `HTTPException` foi importado.
- `ProfessoresDB` está escrito com a capitalização correta.

Com o projeto corrigido, execute em terminais separados:

```bat
python -m pytest -v
```

```bat
python -m uvicorn main:app --reload
```

Depois abra:

```text
http://127.0.0.1:8000/docs
```

## Referências

[1]: https://fastapi.tiangolo.com/advanced/testing-dependencies/ "FastAPI — Testing Dependencies with Overrides"

[2]: https://docs.sqlalchemy.org/en/20/orm/session_basics.html "SQLAlchemy 2.0 — Session Basics"

[3]: https://docs.python.org/3/library/unittest.mock.html "Python 3.14 — unittest.mock"

[4]: https://github.com/ProfAndersonVanin/SW-II_2026/tree/62884a08a794db4e8b8092351775693f8107c543/aulas/3%20BIMESTRE "Materiais do 3º bimestre — Sistemas Web II"

## Créditos

Projeto desenvolvido pela turma **3F** na disciplina de **Sistemas Web II**.

Documentação revisada para refletir a estrutura e o funcionamento da API enviada pela turma.
