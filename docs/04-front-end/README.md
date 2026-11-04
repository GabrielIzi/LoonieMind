# Capítulo 4: Front-end

> Atualizado em 03/11/2026. A proposta inicial usava Streamlit e Matplotlib; a versão 1.1 usa uma interface web própria, servida pela API local. A troca está justificada nas decisões D02 e D03 de [Decisões e escopo](../DECISOES-E-ESCOPO.md).

## 1. Objetivo do capítulo

Documentar a interface da aplicação, que roda localmente no navegador em `http://127.0.0.1:8765`.

## 2. Tecnologias

| Tecnologia | Uso |
|---|---|
| HTML, CSS e JavaScript (módulos ES nativos) | Toda a interface, sem etapa de build e sem Node para executar |
| Gráficos SVG | Evolução do saldo, visão mensal, projeção e despesas por categoria, sem imagens remotas ou CDN |
| FastAPI | Serve a interface e a API na mesma origem |

## 3. Arquivos

| Arquivo | Função |
|---|---|
| `web/index.html` | Estrutura da página e menu lateral |
| `web/app.js` | Rotas por hash, telas, chamadas à API e ações do usuário |
| `web/charts.js` | Gráficos SVG |
| `web/icons.js` | Ícones |
| `web/styles.css` | Estilos e responsividade |
| `web/api.html`, `web/api-reference.js` | Referência da API servida localmente em `/api/docs` |

## 4. Funcionalidades da interface

- Cadastro de empresas, saldo de referência e categorias; alternância entre empresas sem misturar dados.
- Importação em duas fases (prévia e confirmação), com histórico e opção de desfazer.
- Lançamentos com busca, filtros, cadastro manual, edição e exclusão.
- Classificação com sugestões, confirmação em lote e treinamento do modelo.
- Dashboard com saldo consolidado, entradas/saídas/resultado, evolução diária, projeção de 30 dias, qualidade do histórico e alertas.
- Previsão de 7, 30, 60 ou 90 dias.
- Exportação em Excel/CSV, backup completo e download dos materiais do projeto.

## 5. Telas implementadas

| Tela | Rota | Status | Observação |
|---|---|---|---|
| Dashboard | `#/dashboard` | Implementada | O saldo independe do filtro de período; a visão mensal usa sempre os últimos 6 meses |
| Importação | `#/import` | Implementada | Até 10 MB, 10.000 linhas, 100 colunas e 30 abas |
| Lançamentos | `#/transactions` | Implementada | Edições e exclusões vão para a auditoria |
| Classificação | `#/classification` | Implementada | O score exibido não é calibrado |
| Previsão de caixa | `#/forecast` | Implementada | Mostra método escolhido e erro de validação |
| Exportação | `#/export` | Implementada | CSV com UTF-8 BOM, ponto e vírgula e decimais brasileiros |
| Minha empresa | `#/settings` | Implementada | Inclui base ampliada de demonstração e histórico de alterações |

## 6. Segurança e comportamento

- Todo texto importado é escapado antes de montar o HTML.
- O servidor aceita apenas Host `localhost`/`127.0.0.1` e recusa escritas de outra origem.
- Atalhos: `/` foca a busca; `Esc` fecha o menu e o painel de alertas.
- Verificada em desktop (1440 px) e em tela de 390 px, com menu móvel e sem rolagem horizontal do documento.

## 7. Pontos em aberto

- Não há registro de revisão de acessibilidade (leitores de tela, contraste).
- Testes automatizados do front-end se limitam à checagem de sintaxe (`node --check`); não há testes de interface automatizados.

[Voltar ao índice](../../README.md)
