# LoonieMind

Aplicação de inteligência artificial para organizar movimentações financeiras e prever o fluxo de caixa de pequenas empresas, rodando de forma simples e local.

| Informação | Detalhe |
|---|---|
| Disciplina | Inteligência Artificial, 2º semestre de 2026 |
| Professor | Edson Ceroni |
| Nome do grupo | LoonieMind |
| Status | Em desenvolvimento |
| Repositório | https://github.com/GabrielIzi/LoonieMind |

## Identificação

| Logo do projeto |
|---|
| ![Logo LoonieMind](docs/logo.jpeg) |

| Integrantes |
|---|
| Gabriel Izidoro Felix dos Santos |
| David Ramos dos Santos |
| Felipe Silva Marcelino |
| Guilherme de Quadros Wendler da Costa |
| Gustavo Araujo |
| Lucas Evangelista |
| Rodrigo dos Anjos |

## Sumário

| Seção | Onde |
|---|---|
| Capítulo 01: Prototipação | [docs/01-prototipacao](docs/01-prototipacao/README.md) |
| Capítulo 02: Modelagem do banco de dados | [docs/02-modelagem-banco-de-dados](docs/02-modelagem-banco-de-dados/README.md) |
| Capítulo 03: Treinamento do modelo | [docs/03-treinamento-do-modelo](docs/03-treinamento-do-modelo/README.md) |
| Capítulo 04: Front-end | [docs/04-front-end](docs/04-front-end/README.md) |
| Capítulo 05: Back-end | [docs/05-back-end](docs/05-back-end/README.md) |

## 1. Objetivo do trabalho

### Objetivo geral

Criar uma aplicação local que usa inteligência artificial para classificar as movimentações financeiras de uma empresa e prever o seu fluxo de caixa.

### Objetivos específicos

- Importar dados por Excel/CSV
- Armazenar o histórico localmente
- Classificar gastos e receitas
- Prever o fluxo de caixa
- Mostrar os resultados em gráficos/dashboard
- Exportar os resultados para Excel/CSV

A ideia é manter tudo simples e rodando localmente, deixando espaço para integrar outras fontes de dados no futuro, como CRMs.

## 2. Funcionamento da aplicação

1. O usuário importa uma planilha Excel ou CSV.
2. O sistema identifica as colunas, padroniza os formatos e valida os dados.
3. Os lançamentos são guardados no histórico local.
4. O modelo classifica cada lançamento como receita ou despesa e define a categoria.
5. O modelo de previsão estima receitas e despesas futuras a partir do histórico.
6. O dashboard mostra os resultados em gráficos.
7. O usuário exporta os resultados para Excel ou CSV.

O detalhamento de cada etapa está nos capítulos 03 e 05.

## 3. Softwares e bibliotecas utilizados

### Softwares
 
| Software | Versão | Como é usado | Quando é usado |
|---|---|---|---|
| Python | (versão) | Linguagem em que toda a aplicação é escrita | No desenvolvimento e sempre que a aplicação roda |
| SQLite | (versão) | Banco de dados local que guarda empresas, planilhas e lançamentos | Ao gravar os dados importados e ao ler o histórico para classificar, prever e exportar |
 
### Bibliotecas
 
| Biblioteca | Versão | Como é usada | Quando é usada |
|---|---|---|---|
| Pandas | (versão) | Lê os arquivos Excel/CSV, mapeia colunas, padroniza formatos, valida dados, monta as variáveis temporais e gera os arquivos de exportação | Na importação, na normalização, na preparação da previsão e na exportação |
| Scikit-learn | (versão) | Fornece o TF-IDF, que transforma as descrições dos lançamentos em números | No treinamento e na classificação de cada lançamento |
| LightGBM | (versão) | Classificador (receita, despesa e categoria) e regressor (previsão de receitas e despesas) | No treinamento dos modelos, na classificação dos dados importados e na geração da previsão |
| Matplotlib | (versão) | Cria os gráficos do dashboard | Ao mostrar os resultados da classificação e da previsão |
| Streamlit | (versão) | Interface da aplicação, local ou web | Durante toda a execução: importar, ver resultados e exportar |
 
### Ferramentas de distribuição
 
| Biblioteca | Versão | Como é usada | Quando é usada |
|---|---|---|---|
| Briefcase (BeeWare) | (versão) | Empacota o projeto Python como aplicativo instalável: MSI no Windows, .deb, .rpm, Flatpak e AppImage no Linux, DMG no macOS | Só no final, depois que a aplicação estiver pronta, rodando o empacotamento em cada sistema operacional de destino (a validar com o Streamlit) |

## 4. Sistemas operacionais compatíveis

| Sistema | Compatível | Versão testada |
|---|---|---|
| Windows | Sim | (a preencher) |
| Linux | Sim | (a preencher) |
| Android | Não | - |
| iOS | Não | - |
| Outros (macOS) | Não | - |

## 5. Elementos de Inteligência Artificial

| Princípio ou elemento | Utilizado | Justificativa |
|---|---|---|
| Aprendizado de máquina supervisionado | Sim | A classificação (LightGBM) e a previsão (LightGBM Regressor) são treinadas com dados históricos rotulados |
| Aprendizado de máquina não supervisionado | Não | Nenhum modelo busca agrupamentos sem rótulos |
| Aprendizado por reforço | Não | Não há agente que aprende por recompensa |
| Reconhecimento de padrão | Sim | O modelo reconhece padrões nas descrições dos lançamentos e nas variáveis temporais do histórico |
| Linguagem natural | Sim | O TF-IDF transforma as descrições dos lançamentos em números para o modelo |
| Banco de conhecimentos | Sim | O histórico fica no SQLite e as regras e aliases de normalização ficam pré-cadastrados |
| Poda | Não | Não é usada |
| Mecanismo de inferência | Não | Não há motor de regras com inferência |
| Encadeamento para frente | Não | Não é usado |
| Encadeamento para trás | Não | Não é usado |
| Sistema especialista | Não | As regras servem só para normalizar os dados |
| Sistema heurístico | Não | Não é usado |
| Modelo de busca | Não | Não é usado |
| IA generativa | Não | Não é usada |

## 6. Regras e quadro negro

### Quantidade de regras utilizadas

(a preencher: número de regras e aliases pré-cadastrados na normalização)

### Domínio de conhecimento

Finanças de pequenas empresas: lançamentos de receitas e despesas, categorias financeiras e fluxo de caixa ao longo do tempo.

### Árvore de decisão

Os modelos LightGBM são conjuntos de árvores de decisão. A árvore e as categorias finais serão apresentadas no capítulo 03 (a preencher).
