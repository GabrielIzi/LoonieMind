# Capítulo 3: Treinamento do modelo

> Atualizado em 03/11/2026 com o que foi implementado (`looniemind/models.py`) e medido na versão 1.1. Todos os dados de treinamento e avaliação são **sintéticos**: os resultados não comprovam qualidade em empresas reais. Fontes: [avaliacao-v1.1.json](../relatorios/avaliacao-v1.1.json), [Arquitetura](../ARQUITETURA.md) e [Manual](../MANUAL-SISTEMA-E-IA.md).

## 1. Objetivo do capítulo

Documentar os dois modelos de aprendizado de máquina do projeto: a classificação financeira e a previsão financeira.

## 2. Classificação financeira

- Tipo de modelo: supervisionado, local, sem serviços externos de IA e sem chave de API
- Algoritmo: LightGBM Classifier
- Representação do texto: TF-IDF (Scikit-learn) de palavras (1 a 2 termos, até 12.000 atributos) e de caracteres dentro de palavras (3 a 5 caracteres, até 16.000 atributos)
- Alvo: classe composta por tipo e categoria, por exemplo "Despesa | Aluguel"
- Entrada: somente a descrição do lançamento. Documento, conta e valor ficam no histórico, mas não entram no modelo
- Parâmetros (fixos): 90 árvores, até 15 folhas, profundidade 5, mínimo de 5 exemplos por folha, taxa 0,1, semente 42, 2 threads e ponderação equilibrada de classes

### Como as categorias viram números

O TF-IDF transforma a descrição em números (entrada). O LightGBM prevê um código numérico de classe (saída), que é decodificado de volta para tipo e categoria dentro do próprio artefato do modelo. No banco, o campo `categoria` guarda sempre o nome (capítulo 02).

### Uso e revisão

- Sem modelo treinado, 12 regras por palavra-chave sugerem categorias, identificadas como regra e sem score.
- Com modelo, a sugestão respeita o tipo já conhecido do lançamento.
- O score é a saída bruta do classificador, **sem calibração**: um score alto pode estar errado.
- O usuário confirma ou corrige. Lançamentos já revisados não são substituídos por novas sugestões.
- Confirmar uma categoria **não** treina o modelo na hora. Para incorporar novos exemplos: Classificação → Treinar modelo → Histórico revisado. Isso substitui o artefato anterior da empresa e limpa sugestões pendentes.
- Requisitos do treino com histórico revisado: no mínimo 24 exemplos, 2 classes e 4 descrições distintas por classe.

## 3. Previsão financeira

- Algoritmo: LightGBM Regressor, com Pandas para a série diária
- Séries separadas de receitas e despesas, combinadas com o saldo atual:
  saldo futuro = saldo realizado + entradas estimadas acumuladas − saídas estimadas acumuladas
- Histórico: até 1.095 dias recentes; dias sem registro são tratados como movimento zero (os arquivos precisam cobrir todo o período)
- Com menos de 28 dias de histórico recente, ou com a última movimentação há mais de 7 dias, não há projeção
- A partir de 28 dias: referência semanal (repetição do padrão de 7 dias)
- A partir de 90 dias e 20 dias com valores positivos na série: também avalia o LightGBM Regressor (65 árvores, 10 folhas, profundidade 4, taxa 0,07, semente 42)
- Variáveis: dia da semana, dia/mês, componente semanal, defasagens de 1, 7, 14 e 28 dias e médias móveis anteriores de 7 e 28 dias
- Método: os últimos 14 a 21 dias ficam reservados; a previsão é recursiva e usa só o passado. Compara o MAE do LightGBM com o da referência semanal e escolhe o menor por série; o escolhido é reajustado com todo o histórico
- Horizontes: 7, 30, 60 ou 90 dias. Valores previstos negativos são limitados a zero

## 4. Dados de treinamento

- **Origem:** base sintética v2, gerada de forma reproduzível por `python -m scripts.generate_dataset --end 2026-10-27` (semente 2702026, nomes fictícios, sem CPF, CNPJ, contas ou dados pessoais reais). Manifesto com hashes SHA-256 em `datasets/sintetica-v2/manifesto.json`
- **Classificação:** 14.400 exemplos, 12 classes equilibradas (1.200 por classe), 288 famílias de descrição e 50 variações por família (`classificacao_rotulada.csv`). Esse arquivo é a base do classificador e **não** deve ser importado como caixa
- **Financeiro:** 10.965 movimentações em três cenários, cada um importado em sua própria empresa: Aurora Comércio (4.124), Horizonte Varejo (4.125, com queda recente de receitas e despesa excepcional) e Studio Soluções (2.716, com sazonalidade)
- **Classes (12):** receitas Vendas, Serviços e Outras receitas; despesas Fornecedores, Aluguel, Pessoal, Impostos, Marketing, Logística, Alimentação, Utilidades e Outras despesas
- **Separação:** por famílias de descrição inteiras, para que variações da mesma descrição não apareçam em treino e teste ao mesmo tempo (campo `grupo` preservado)

| Partição | Exemplos | Famílias | Proporção |
|---|---|---|---|
| Treino | 9.600 | 192 | 66,7% |
| Validação | 2.400 | 48 | 16,7% |
| Teste | 2.400 | 48 | 16,7% |

O vocabulário do TF-IDF é aprendido só no treino durante a avaliação. Depois de medir validação e teste, o modelo de uso é reajustado na base completa e salvo por empresa em `data/models/` (versão, hash dos dados, bibliotecas e métricas).

Comandos de treino:

```powershell
.\.venv\Scripts\python.exe -m scripts.train_models --company ID --source synthetic
.\.venv\Scripts\python.exe -m scripts.train_models --company ID --csv datasets/sintetica-v2/classificacao_rotulada.csv
```

## 5. Avaliação

### Classificação (base v2)

F1 macro: **0,6034 na validação** e **0,6766 no teste**. A média macro dá o mesmo peso a cada classe; não é a porcentagem geral de acertos nem uma probabilidade de acerto financeiro. Resultado por classe no teste (200 exemplos por classe):

| Classe | Precisão | Recall | F1 |
|---|---|---|---|
| Receita / Vendas | 0,938 | 0,750 | 0,833 |
| Receita / Serviços | 1,000 | 0,260 | 0,413 |
| Receita / Outras receitas | 0,576 | 1,000 | 0,731 |
| Despesa / Aluguel | 1,000 | 0,750 | 0,857 |
| Despesa / Marketing | 1,000 | 0,750 | 0,857 |
| Despesa / Utilidades | 1,000 | 0,750 | 0,857 |
| Despesa / Alimentação | 1,000 | 0,500 | 0,667 |
| Despesa / Fornecedores | 0,512 | 0,880 | 0,647 |
| Despesa / Pessoal | 0,474 | 1,000 | 0,643 |
| Despesa / Impostos | 0,667 | 0,500 | 0,571 |
| Despesa / Logística | 0,667 | 0,500 | 0,571 |
| Despesa / Outras despesas | 0,444 | 0,500 | 0,471 |

O desempenho é desigual: Serviços teve o pior F1 (0,413). Mais variações do mesmo padrão não substituem a diversidade de exemplos reais.

### Previsão (validação cronológica de 7 a 27/10/2026)

| Cenário / série | MAE LightGBM | MAE referência | Escolhido |
|---|---|---|---|
| Aurora / receitas | R$ 135,89 | R$ 145,59 | LightGBM |
| Aurora / despesas | R$ 126,68 | R$ 2.078,73 | LightGBM |
| Horizonte / receitas | R$ 71,16 | R$ 130,54 | LightGBM |
| Horizonte / despesas | R$ 125,04 | R$ 1.797,11 | LightGBM |
| Studio / receitas | R$ 86,43 | R$ 86,42 | Semanal |
| Studio / despesas | R$ 151,67 | R$ 2.390,39 | LightGBM |

MAE é o erro absoluto médio em reais por dia. No cenário Aurora, o WAPE do método escolhido foi 12,21% nas receitas e 30,01% nas despesas. WAPE não equivale a "precisão" (87,79% não é acerto) nem mede o erro acumulado do saldo.

### Limitações da avaliação

- Os dados são sintéticos e os padrões foram definidos pelo gerador.
- A mesma janela recente compara e escolhe o método da previsão; não é uma avaliação externa independente.
- A previsão não lê contratos nem agenda de vencimentos e não tem intervalo de confiança.
- Os scores do classificador não são calibrados.

## 6. Pontos em aberto

- Reunir descrições reais revisadas, com origem e autorização de uso, e medir o erro por classe separando empresas e períodos de teste.
- Comparar modelos e regras, calibrar scores e avaliar rejeição de sugestões incertas.
- Avaliar a previsão em várias janelas temporais e separar agenda de pagamentos de previsão estatística.
- Treinar explicitamente os classificadores de Horizonte Varejo e Studio Soluções (hoje usam regras até o treino).

[Voltar ao índice](../../README.md)
