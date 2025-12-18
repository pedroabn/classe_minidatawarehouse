# 📦 Documentação – Classe `GDriveWarehouse`

## 1. Visão Geral

A classe `GDriveWarehouse` implementa um **Data Warehouse orientado a objetos** utilizando o **Google Drive como camada de armazenamento**. Seu objetivo é abstrair a complexidade de autenticação, leitura, cache e organização de arquivos armazenados no Google Drive, oferecendo uma interface simples para consumo de dados em formato de `DataFrame` (pandas).

A classe suporta arquivos nos seguintes formatos:

* CSV
* Excel (`.xlsx`, `.xls`)
* JSON
* Google Sheets

Além disso, possui **cache em memória com controle de TTL**, indexação preguiçosa (lazy loading) e metadados automáticos sobre os dados carregados.

---

## 2. Arquitetura e Conceitos

A classe foi projetada seguindo princípios de **Programação Orientada a Objetos**, com:

* **Encapsulamento** da lógica de acesso ao Google Drive
* **Abstração** da leitura de diferentes formatos de arquivos
* **Responsabilidade única**: acesso e organização de dados

O `GDriveWarehouse` atua como uma *facade* sobre as APIs do Google Drive e Google Sheets.

---

## 3. Pré-requisitos

### 3.1 Dependências

* Python 3.9+
* pandas
* google-api-python-client
* google-auth
* python-dotenv

### 3.2 Credenciais

É necessário possuir um **Service Account** do Google com permissões de leitura para:

* Google Drive
* Google Sheets

As credenciais devem ser armazenadas em um arquivo `credentials.json`.

---

## 4. Configuração via `.env`

A classe pode carregar automaticamente variáveis de ambiente:

```env
GDRIVE_FOLDER_ID=seu_folder_id
GDRIVE_CREDENTIALS_PATH=/caminho/credentials.json
```

---

## 5. Inicialização da Classe

```python
from utils.load import GDriveWarehouse

wh = GDriveWarehouse(
    folder_id=None,
    credentials_path=None,
    cache_ttl_minutes=30,
    sheet_name=0,
    load_env=True
)
```

### Parâmetros

| Parâmetro           | Descrição                              |
| ------------------- | -------------------------------------- |
| `folder_id`         | ID da pasta raiz no Google Drive       |
| `credentials_path`  | Caminho do arquivo de credenciais      |
| `cache_ttl_minutes` | Tempo de validade do cache em memória  |
| `sheet_name`        | Nome ou índice da aba padrão           |
| `load_env`          | Se `True`, carrega variáveis do `.env` |

---

## 6. Cache e Indexação

* O índice de arquivos é construído apenas quando necessário (*lazy loading*)
* O cache utiliza:

  * Hash do arquivo
  * Timestamp de leitura
  * TTL configurável

Arquivos só são baixados novamente se:

* O cache expirar
* O arquivo for modificado
* `force_refresh=True`

---

## 7. Métodos Públicos

### 7.1 `list_tables()`

Lista todas as tabelas disponíveis sem realizar download.

```python
tabelas = wh.list_tables()
```

---

### 7.2 `get_table()`

Carrega uma tabela específica e retorna um `DataFrame`.

```python
df = wh.get_table("empresas")
```

Parâmetros:

* `name`: nome do arquivo (sem extensão)
* `sheet_name`: aba específica (opcional)
* `force_refresh`: ignora cache

---

### 7.3 `get_tables()`

Carrega múltiplas tabelas de uma vez.

```python
dfs = wh.get_tables(["empresas", "produtos"])
```

Retorno:

```python
{
  "empresas": DataFrame,
  "produtos": DataFrame
}
```

---

### 7.4 `search_tables()`

Busca tabelas pelo nome.

```python
wh.search_tables("emp")
```

---

### 7.5 `join_tables()`

Realiza `join` entre duas tabelas.

```python
df = wh.join_tables(
    left_table="empresas",
    right_table="cidades",
    on="cidade_id",
    how="left"
)
```

---

### 7.6 `get_metadata()`

Retorna metadados das tabelas carregadas.

```python
wh.get_metadata("empresas")
```

Metadados incluem:

* Nome do arquivo
* Tipo
* Dimensões
* Colunas
* Data de modificação

---

### 7.7 `clear_cache()`

Limpa todo o cache em memória.

```python
wh.clear_cache()
```

---

### 7.8 `get_cache_info()`

Retorna informações sobre o cache atual.

```python
wh.get_cache_info()
```

---

## 8. Tratamento de Erros

* Erros de leitura retornam `None`
* Mensagens de erro são exibidas via `print`
* Falhas críticas (credenciais, autenticação) lançam exceções

---

## 9. Boas Práticas de Uso

* Sempre valide retornos `None`
* Use `list_tables()` antes de `get_table()`
* Utilize `force_refresh` apenas quando necessário

---

## 10. Considerações Finais

A classe `GDriveWarehouse` oferece uma solução robusta e extensível para consumo de dados no Google Drive, sendo adequada para:

* Pipelines de dados
* Análises exploratórias
* Dashboards
* Ambientes de BI e Ciência de Dados

Sua estrutura orientada a objetos facilita manutenção, testes e evolução futura.
