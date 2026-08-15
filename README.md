# Project-Tabela-FIPE

Repositório dedicado ao estudo e à análise das variações de preços de bens móveis ao longo do tempo, com dados da [Tabela FIPE](https://veiculos.fipe.org.br/).

> Coleta automatizada dos preços médios de veículos diretamente do site da FIPE, armazenamento em MySQL e base pronta para análise de séries temporais e modelos de Machine Learning.

---

## Funcionalidades

- **Coleta de veículos** — extrai automaticamente todas as combinações de marca, modelo e ano disponíveis no site da FIPE (mais de 12 mil registros).
- **Consulta de preços por período** — para cada veículo, varre os meses de referência disponíveis e captura o preço médio de cada um.
- **Persistência em MySQL** — os dados são organizados em duas tabelas (`tabela_fipe` e `valores_fipe`) prontas para consulta e análise.
- **Base para análise temporal** — a cada consulta fica registrado o mês de referência, permitindo acompanhar a evolução do preço no tempo.

---

## Pipeline

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ FIPE (web)   │────▶│  Selenium    │────▶│   MySQL      │────▶│  Análise / ML│
│ (site oficial)│     │  (coleta)    │     │  (armazenamento)│    │  (futuro)    │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
```

1. `codigo_extract_maquinas.ipynb` — varre o site e popula a tabela `tabela_fipe` com os veículos disponíveis.
2. `codigo_extract_valores.ipynb` — para cada veículo salvo no banco, itera pelos meses de referência, consulta o preço médio e grava na tabela `valores_fipe`.
3. `tabela_final_v2.ipynb` — notebooks de exploração e apoio ao scraping (marcas, modelos, anos).

---

## Tecnologias

| Tecnologia | Uso |
|---|---|
| Python 3.12 | Linguagem principal |
| Selenium | Automação e scraping do site da FIPE |
| Pandas | Manipulação dos dados coletados |
| SQLAlchemy + PyMySQL | Conexão e persistência no MySQL |
| MySQL | Banco de dados |
| python-dotenv | Gestão de credenciais |

---

## Estrutura do repositório

```
Project-Tabela-FIPE/
├── codigo_extract_maquinas.ipynb   # Coleta de marcas, modelos e anos
├── codigo_extract_valores.ipynb    # Consulta de preços por mês de referência
├── tabela_final_v2.ipynb           # Exploração e apoio ao scraping
├── tabela.sql                      # Scripts de criação das tabelas (histórico)
├── senha.env                       # Credenciais do banco (NÃO versionado)
├── .gitignore                      # Ignora arquivos sensíveis
└── README.md
```

---

## Créditos

- **Dados pré-extraídos (`dados_ja_extraidos/`)** — arquivos mensais de preços FIPE (janeiro a agosto/2026), com o respectivo código de mapeamento em `analitico_ml.ipynb`, foram extraídos e exportados por [**alanwgt**](https://github.com/alanwgt). Toda a coleta, mapeamento e exportação desses arquivos é de autoria dele; este repositório apenas consome e analisa esses dados.

---

## Como executar

### Pré-requisitos

- Python 3.12+
- MySQL Server local (porta 3306)
- Chrome (para o Selenium WebDriver)

### 1. Instalar dependências

```bash
pip install selenium pandas sqlalchemy pymysql python-dotenv
```

### 2. Configurar credenciais

Crie um arquivo `senha.env` na raiz (ele **não** deve ser versionado):

```
MYSQL_HOST=localhost
MYSQL_PORT=3306
MYSQL_DATABASE=FIPE
MYSQL_USER=root
MYSQL_PASSWORD=sua_senha
```

### 3. Criar o banco de dados

Rode o `tabela.sql` no MySQL Workbench, ou execute:

```sql
CREATE DATABASE IF NOT EXISTS FIPE;
```

### 4. Executar os notebooks

Abra os notebooks no Jupyter e execute as células em ordem:

1. **`codigo_extract_maquinas.ipynb`** — preenche `tabela_fipe` com os veículos.
2. **`codigo_extract_valores.ipynb`** — consulta e salva os preços na `valores_fipe`.
   - No início da célula principal, ajuste a lista `lista = [...]` com os meses de referência desejados (ex.: `'janeiro/2026'`).

---

## Banco de dados

Schema atual (MySQL, banco `FIPE`):

### `tabela_fipe`

| Coluna | Tipo | Descrição |
|---|---|---|
| `ID` | INT (PK) | Identificador |
| `Marca` | TEXT | Marca do veículo |
| `Modelo` | TEXT | Modelo do veículo |
| `Ano` | TEXT | Ano do modelo + combustível (ex.: `1998 Gasolina`) |

### `valores_fipe`

| Coluna | Tipo | Descrição |
|---|---|---|
| `ID` | INT (PK) | Identificador |
| `Mês de referência` | VARCHAR | Mês completo (ex.: `setembro de 2023`) |
| `Código Fipe` | VARCHAR | Código FIPE do veículo |
| `Marca` | VARCHAR | Marca do veículo |
| `Modelo` | VARCHAR | Modelo do veículo |
| `Ano Modelo` | VARCHAR | Ano + combustível (ex.: `1998 Gasolina`) |
| `Autenticação` | VARCHAR | Token da consulta |
| `Data da consulta` | VARCHAR | Data/hora da coleta |
| `Preço Médio` | VARCHAR | Preço médio (ex.: `R$ 26.762,00`) |
| `Mês` | VARCHAR | Mês de referência (ex.: `setembro/2023`) |

---

## Status das fases

| Fase | Status |
|---|---|
| Coleta de veículos (marcas, modelos e anos) | ✅ Concluída |
| Consulta de preços por mês de referência | ✅ Concluída |
| Persistência em MySQL (`tabela_fipe` / `valores_fipe`) | ✅ Concluída |
| Documentação do projeto | ✅ Concluída |
| Normalização dos dados (ano, combustível, preço numérico) | 🚧 Em andamento |
| Modelo de Machine Learning (predição de preço) | ⏳ Pendente |
| Análise de séries temporais (previsão de variação mensal) | ⏳ Pendente |
| Curva de depreciação por modelo | ⏳ Pendente |
| Detecção de anomalias de preço | ⏳ Pendente |

**Legenda:** ✅ Concluída · 🚧 Em andamento · ⏳ Pendente

## Próximos passos (roadmap)

1. **Normalização** — separar ano e combustível em colunas próprias e transformar `Preço Médio` em valor numérico (base para o ML).
2. **Machine Learning** — modelo de regressão para predição do preço FIPE a partir de marca, modelo, ano e combustível.
3. **Séries temporais** — previsão da variação mensal de preço com base no histórico coletado.
4. **Curva de depreciação** — análise de como cada modelo desvaloriza com a idade.
5. **Anomalias** — identificar veículos com preço fora do esperado para suas características.

---

## Licença

Distribuído sob a licença MIT. Veja o arquivo [LICENSE](LICENSE).