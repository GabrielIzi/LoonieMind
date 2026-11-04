# Capítulo 5: Back-end

> Atualizado em 03/11/2026. O back-end está implementado em `looniemind/` com FastAPI e SQLite. Detalhes e justificativas em [Arquitetura operacional](../ARQUITETURA.md), [Decisões e escopo](../DECISOES-E-ESCOPO.md) e [Validação](../VALIDACAO.md).

## 1. Objetivo do capítulo

Documentar a lógica da aplicação: tratamento dos dados, classificação, previsão e armazenamento.

## 2. Tecnologias

| Tecnologia | Uso |
|---|---|
| Python 3.12 | Linguagem da aplicação |
| FastAPI / Uvicorn | API e servidor da interface local (`127.0.0.1:8765`) |
| SQLite | Armazenamento local do histórico, com valores em centavos inteiros |
| Pandas | Leitura e preparação de dados; séries temporais |
| Openpyxl / Xlrd | Leitura e escrita de Excel atual (XLSX) e antigo (XLS) |
| Scikit-learn | TF-IDF e avaliação |
| LightGBM | Classificação e previsão |
| Joblib | Salvar o artefato do modelo de cada empresa |

Versões fixadas em `requirements-lock.txt` (execução) e `requirements-dev-lock.txt` (testes e verificação).

## 3. Componentes

### Normalização e importação (`imports.py`, `domain.py`)

Lê CSV (UTF-8/BOM e CP1252, vários separadores), XLS e XLSX; localiza o cabeçalho nas primeiras linhas; mapeia as colunas (aliases em `imports.py`); padroniza datas e valores em formatos brasileiro e decimal; e valida os dados. Importação em **duas fases**: a prévia não grava nada; a confirmação grava de forma transacional. Mesma empresa, arquivo e aba não podem ser confirmados duas vezes; lançamentos semelhantes exigem confirmação; importação parcial exige escolha explícita; é possível desfazer a importação inteira. Limites: 10 MB, 10.000 linhas, 100 colunas e 30 abas. Fórmulas e macros não são executadas.

### Serviços financeiros (`services.py`)

Empresas, lançamentos, revisão, consultas, indicadores do dashboard, exportação e backup. Saldo atual = saldo inicial + receitas − despesas realizadas desde a data de referência. Lançamento sem tipo fica fora dos totais e é sinalizado.

### Classificação financeira (`models.py`)

LightGBM com TF-IDF, treino explícito, artefato por empresa e sugestões que respeitam revisões humanas. Detalhes no capítulo 3.

### Previsão financeira (`models.py`)

Séries diárias de receitas e despesas, comparação entre LightGBM e referência semanal em janela cronológica. Detalhes no capítulo 3.

### Armazenamento (`db.py`)

SQLite com esquema v2 versionado e migração a partir da v1. Modelo no capítulo 2.

### Relatórios e planilhas (`workbooks.py`)

Gera o modelo de importação (13 campos), o Excel financeiro (Resumo, Mensal, Categorias, Centros_de_custo, Lancamentos) e o Excel de projeção (Resumo, Previsao, Metodos). Textos que começam com caracteres de fórmula são protegidos como texto.

### Dados sintéticos (`synthetic.py`) e backup (`backup.py`)

Geração reproduzível das bases de demonstração e restauração validada de backups em pasta nova.

## 4. Estrutura do código

```text
run.py                  inicialização local e abertura do navegador
looniemind/
  api.py                rotas, contratos de entrada, erros e interface estática
  db.py                 esquema SQLite, conexões, versão e auditoria
  domain.py             dinheiro, datas, tipos e normalização de texto
  imports.py            leitura CSV/XLS/XLSX, prévia e confirmação
  services.py           empresas, lançamentos, consultas, exportação e backup
  models.py             TF-IDF/LightGBM, regras e previsão temporal
  synthetic.py          geração das bases rotulada e financeira
  workbooks.py          modelos de importação e relatórios
  backup.py             restauração validada
web/                    interface (capítulo 4)
scripts/                treinamento, base sintética, restauração e empacotamento dos materiais
tests/                  testes automatizados
datasets/sintetica-v2/  base sintética e manifesto
docs/                   documentação
plataforma/database/    banco original vazio, mantido como referência histórica
outputs/                apresentação, roteiro, manual e planilhas
```

## 5. API

Contrato completo em `/api/openapi.json`; referência legível em `/api/docs`, ambos servidos localmente. Principais grupos de rotas (prefixo `/api/companies/{company_id}` nas operações por empresa): empresas, indicadores (`/summary`), lançamentos (`/transactions`), importações (`/imports`, com `/preview`, `/confirm` e `/undo`), categorias, modelo (`/model`, `/model/train`), classificação (`/classify`, `/classify/accept`), previsão (`/forecast`), exportação (`/export`), auditoria (`/audit`), modelo de arquivo (`/api/template`) e backup (`/api/backup`).

## 6. Qualidade e operação

- **Testes:** 41 testes automatizados passaram na entrega de 28/10/2026 (domínio financeiro, importação, exportação, backup e restauração, modelos, isolamento entre empresas e materiais). Ruff sem erros e verificação de sintaxe dos módulos JavaScript.
- **CI:** `.github/workflows/tests.yml` executa Ruff, pytest e `node --check` em Windows e Linux (Python 3.12). A validação local foi feita em Windows x64; a execução em Linux ainda precisa ser confirmada.
- **Segurança local:** servidor em `127.0.0.1`, Host restrito, escritas de outra origem recusadas, filtros SQL parametrizados, uploads guardados sob UUID e artefatos de modelo enviados pelo usuário nunca carregados.
- **Limites:** não há login, permissões por usuário, criptografia própria do banco nem sincronização. Não expor o servidor na internet.

## 7. Pontos em aberto

- Empacotamento nativo (`.exe`, `.deb`, Flatpak): a v1 usa scripts de inicialização (`Iniciar LoonieMind.cmd` e `iniciar.sh`); instaladores ficam para uma evolução.
- Validar a execução em Linux (a CI está configurada, mas o resultado não foi registrado).
- Autenticação, permissões, proteção dos arquivos e política de retenção antes de qualquer hospedagem compartilhada.
- Contas programadas e vencimentos futuros.
- Integração com outras fontes de dados, como CRMs, ainda não planejada em detalhe.

[Voltar ao índice](../../README.md)
