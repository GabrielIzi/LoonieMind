# LoonieMind

Aplicação de inteligência artificial para organizar movimentações financeiras e prever o fluxo de caixa de pequenas empresas, rodando de forma simples e local.

Status: em desenvolvimento

## Identificação

| Logo do projeto |
|---|
| ![Logo LoonieMind](/docs/logo.jpeg) |

| Integrantes |
|---|
| Gabriel Izidoro Felix dos Santos |
| David Ramos dos Santos |
| Felipe Silva Marcelino |
| Guilherme de Quadros Wendler da Costa |
| Gustavo Araujo |
| Lucas Evangelista |
| Rodrigo dos Anjos |

## Capítulos

 - [01 Prototipação](/docs/01-prototipacao/README.md)
 - [02 Modelagem do banco de dados](/docs/02-modelagem-banco-de-dados/README.md)
 - [03 Treinamento do Modelo](/docs/03-treinamento-do-modelo/README.md)
 - [04 Front End](/docs/04-front-end/README.md)
 - [05 Back End](/docs/05-back-end/README.md)

## 1. Visão geral

A LoonieMind importa planilhas financeiras, padroniza os dados, classifica automaticamente receitas e despesas, prevê o fluxo de caixa futuro e apresenta os resultados em gráficos e em um dashboard. A ideia é manter tudo simples e rodando localmente, deixando espaço para integrar outras fontes de dados no futuro, como CRMs.

## 2. Escopo

- Importar dados por Excel/CSV
- Armazenar o histórico localmente
- Classificar gastos e receitas
- Prever o fluxo de caixa
- Mostrar os resultados em gráficos/dashboard
- Exportar os resultados para Excel/CSV

## 3. Tecnologias

| Tecnologia | Uso no projeto |
|---|---|
| Python | Linguagem de desenvolvimento da aplicação |
| SQLite | Banco de dados para armazenamento local dos dados |
| Pandas | Biblioteca Python para tratamento e organização dos dados |
| Scikit-learn | Biblioteca Python para aprendizado de máquina |
| LightGBM | Framework open-source da Microsoft para aprendizado de máquina, usado pela biblioteca lightgbm (Python) |
| Matplotlib | Biblioteca Python para criação de visualizações de dados estáticas, animadas e interativas |
| Streamlit | Interface da aplicação: biblioteca Python que permite criar um app totalmente local ou uma interface web sem esforço |

## 4. Componentes principais

Além do banco de dados (SQLite) e da interface (Streamlit), o projeto tem três componentes principais:

- **Normalização de dados:** usa regras e aliases pré-cadastrados, junto com o Pandas, para identificar e mapear colunas, padronizar formatos e validar os dados.
- **Classificação financeira:** usa um modelo supervisionado LightGBM com TF-IDF (Scikit-learn) para classificar automaticamente receitas, despesas e categorias.
- **Previsão financeira:** usa o LightGBM Regressor com o Pandas para prever receitas e despesas futuras, com base no histórico e em variáveis temporais.