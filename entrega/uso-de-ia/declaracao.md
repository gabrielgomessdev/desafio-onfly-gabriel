# Declaração de Uso de Inteligência Artificial

Em conformidade com as diretrizes do desafio técnico, este documento formaliza o uso de ferramentas de assistência por inteligência artificial durante a realização da atividade.

---

## 1. Ferramenta e Modelo Utilizados
* **Plataforma:** Google Gemini
* **Modelo:** Gemini 3.8 Flash
* **Link público da sessão compartilhada:** [Link público da conversa](https://share.gemini.google/up5xvGHoLR44)

---

## 2. Divisão de Atuação e Responsabilidades

### O que a Inteligência Artificial fez:
* **Diagnóstico e mapeamento de falhas:** Auxiliou no agrupamento e leitura analítica do output do `pytest`, destacando o que cada teste falho estava esperando versus o que foi retornado (sem fornecer a solução pronta de imediato).
* **Orientação de sintaxe e ordenação composta:** Esclareceu a mecânica do método `sorted()` do Python para ordenação decrescente por valor financeiro e ascendente por critério de desempate alfabético (`-item[1]["amount"], item[0]`).
* **Suporte operacional de ambiente:** Sugeriu comandos do PowerShell para execução da CLI e configuração de variáveis de ambiente (`$env:PYTHONPATH`).
* **Estruturação de documentação:** Forneceu modelos de formatação em Markdown para o relatório de entrega e checklist das etapas concluídas.

### O que o desenvolvedor fez:
* **Configuração e gerenciamento do ambiente:** Criação, ativação do ambiente virtual (`.venv`) e instalação das dependências de desenvolvimento no VS Code.
* **Investigação da base de código:** Localização dos arquivos e serviços responsáveis pelas regras de negócio afetadas dentro de `src/onfly_report/services/`.
* **Implementação e refatoração das soluções:**
  * Correção da fórmula de conversão cambial em `currency_converter.py` (multiplicação de moeda estrangeira pela cotação).
  * Refatoração do cálculo de estadias em `stay_calculator.py` para utilizar objetos de data completos e respeitar viradas de mês/ano.
  * Ajuste da política de elegibilidade em `status_policy.py`, excluindo reservas `CANCELLED` e `NO_SHOW` dos totais financeiros.
  * Aplicação do critério composto de ordenação em `report_builder.py`.
* **Validação prática:** Execução dos testes via `pytest`, geração do relatório completo (`report.json`) e validação de contrato com o dataset de referência (`exemplo-validacao.csv`).
* **Versionamento e entrega:** Revisão do repositório, redação final da documentação e submissão do código.