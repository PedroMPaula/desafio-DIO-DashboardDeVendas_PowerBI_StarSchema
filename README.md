# Desafio de Modelagem Dimensional — Star Schema

## Objetivo

Criar um **esquema em estrela (star schema)** focado na análise dos dados de **professores**, com base em um diagrama relacional fornecido como referência.

O modelo financeiro disponibilizado no arquivo `star_schema_financial_sample.pbix` foi utilizado como **referência de estrutura e boas práticas** de modelagem dimensional no Power BI — não como fonte de dados do desafio.

---

## Contexto

O foco analítico do modelo é o **professor**: seus cursos ministrados, disciplinas ofertadas, departamento ao qual pertence e o período de atuação. Dados relacionados a alunos estão fora do escopo deste desafio.

---

## Estrutura do Star Schema

### Tabela Fato — `F_Professor`

Registra os eventos mensuráveis relacionados à atuação do professor. Cada linha representa um professor ministrando uma disciplina em um curso, em um determinado período.

| Coluna            | Tipo    | Descrição                                  |
|-------------------|---------|--------------------------------------------|
| `id_fato`         | PK      | Chave primária surrogate                   |
| `id_professor`    | FK      | Referência a `D_Professor`                 |
| `id_disciplina`   | FK      | Referência a `D_Disciplina`                |
| `id_curso`        | FK      | Referência a `D_Curso`                     |
| `id_departamento` | FK      | Referência a `D_Departamento`              |
| `id_data`         | FK      | Referência a `D_Data`                      |
| `carga_horaria`   | Medida  | Carga horária da disciplina no período     |
| `salario`         | Medida  | Salário do professor no período            |

---

### Tabelas Dimensão

#### `D_Professor`
Atributos descritivos do professor.

| Coluna           | Descrição                              |
|------------------|----------------------------------------|
| `id_professor`   | PK surrogate                           |
| `nome`           | Nome completo                          |
| `titulacao`      | Graduação, mestrado, doutorado etc.    |
| `regime_trabalho`| Integral, parcial, horista             |
| `email`          | E-mail institucional                   |
| `data_admissao`  | Data de entrada na instituição         |

---

#### `D_Disciplina`
Detalhes sobre as disciplinas ministradas.

| Coluna              | Descrição                          |
|---------------------|------------------------------------|
| `id_disciplina`     | PK surrogate                       |
| `nome_disciplina`   | Nome da disciplina                 |
| `codigo`            | Código interno                     |
| `carga_horaria_tot` | Carga horária total da disciplina  |
| `nivel`             | Graduação ou pós-graduação         |
| `periodo_oferta`    | Semestre de oferta (ex: 2024-1)    |

---

#### `D_Curso`
Informações sobre os cursos aos quais as disciplinas pertencem.

| Coluna              | Descrição                              |
|---------------------|----------------------------------------|
| `id_curso`          | PK surrogate                           |
| `nome_curso`        | Nome do curso                          |
| `modalidade`        | Presencial, EAD, híbrido               |
| `turno`             | Manhã, tarde, noite, integral          |
| `duracao_semestres` | Duração total do curso em semestres    |
| `area_conhecimento` | Ex: Exatas, Humanas, Saúde             |

---

#### `D_Departamento`
Dados do departamento acadêmico ao qual o professor está vinculado.

| Coluna              | Descrição                          |
|---------------------|------------------------------------|
| `id_departamento`   | PK surrogate                       |
| `nome_departamento` | Nome completo do departamento      |
| `sigla`             | Sigla (ex: DINFO, DMAT)            |
| `centro`            | Centro acadêmico                   |
| `campus`            | Campus da instituição              |
| `chefe_departamento`| Nome do chefe do departamento      |

---

#### `D_Data`
Dimensão de tempo criada para compensar a ausência de dados de datas no modelo relacional de origem. Permite análises em múltiplos níveis de granularidade.

| Coluna          | Descrição                                  |
|-----------------|--------------------------------------------|
| `id_data`       | PK surrogate                               |
| `data_completa` | Data no formato `YYYY-MM-DD`              |
| `ano`           | Ano (ex: 2024)                             |
| `semestre`      | 1 ou 2                                     |
| `mes`           | Número do mês (1–12)                       |
| `trimestre`     | 1, 2, 3 ou 4                               |
| `ano_semestre`  | Chave legível (ex: 2024-1)                 |

> **Nota:** Os campos de data foram criados com base em suposição de acesso aos dados reais, conforme orientação do desafio. As datas representam eventos como data de oferta da disciplina, data de início do curso, entre outros.

---

## Relacionamentos

Todos os relacionamentos seguem o padrão **1 (dimensão) para N (fato)**:

```
D_Professor    (1) ──→ (N) F_Professor
D_Disciplina   (1) ──→ (N) F_Professor
D_Curso        (1) ──→ (N) F_Professor
D_Departamento (1) ──→ (N) F_Professor
D_Data         (1) ──→ (N) F_Professor
```

---

## Diagrama

O diagrama abaixo representa visualmente o star schema construído:

```
                    ┌──────────────┐
                    │  D_Disciplina│
                    └──────┬───────┘
                           │
     ┌──────────────┐      │      ┌──────────────┐
     │  D_Professor │──────┤      │   D_Curso    │
     └──────────────┘      │      └──────┬───────┘
                           ▼             │
                    ┌──────────────┐     │
                    │  F_Professor │◄────┘
                    └──────┬───────┘
                    ▲      │      ▲
                    │      ▼      │
         ┌──────────┴──┐  ┌──────┴──────────┐
         │   D_Data    │  │ D_Departamento  │
         └─────────────┘  └─────────────────┘
```

---

## Referência de Boas Práticas

O arquivo `star_schema_financial_sample.pbix` foi utilizado como modelo de referência. Ele demonstra:

- Uso do prefixo `D_` para tabelas dimensão e `F_` para tabela fato
- Chaves surrogate (`ID_Produto`) para relacionamentos entre fato e dimensões
- Separação entre tabela de origem bruta (`financials_origem`) e tabelas do modelo analítico
- Relacionamentos `1:N` entre dimensões e tabela fato no Power BI

---

## Observações Finais

- Dados de **alunos não foram incluídos** no modelo, conforme restrição do desafio.
- A tabela `D_Data` foi **criada do zero**, pois o modelo relacional de origem não continha campos de data explícitos.
- Chaves surrogate foram adotadas em todas as tabelas para garantir integridade e desempenho no ambiente analítico.
