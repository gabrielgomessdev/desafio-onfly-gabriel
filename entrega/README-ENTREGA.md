# Relatório de Entrega — Desafio Técnico Onfly

Este documento descreve a resolução dos bugs identificados na CLI de geração de relatórios de reservas de hotéis, detalhando a causa raiz de cada falha, as correções adotadas, a validação por testes automatizados e a declaração de uso de ferramentas de assistência.

---

## 1. Diagnóstico e Correção de Bugs

### Bug 1: Conversão Invertida de Câmbio (USD para BRL)
* **Arquivo:** `src/onfly_report/services/currency_converter.py` 
* **Sintoma:** O teste `test_currency_conversion_to_brl` falhou acusando valor menor do que o original (`118.45` em vez de `642.00`).
* **Causa Raiz:** A lógica de conversão estava dividindo o valor em moeda estrangeira pela taxa de câmbio (`amount / rate`), em vez de multiplicá-lo (`amount * rate`). Para $ 100,00 USD em uma taxa de câmbio de 5.42 BRL/USD, a operação resultava em aproximadamente R$ 18,45, em vez dos R$ 542,00 corretos.
* **Solução:** A operação foi ajustada para multiplicar o montante em moeda estrangeira pela taxa correspondente da cotação (`amount * rate`), garantindo a conversão correta para reais antes do somatório.

---

### Bug 2: Cálculo Incorreto de Diárias com Virada de Mês
* **Arquivo:** `src/onfly_report/services/stay_calculator.py` 
* **Sintoma:** O teste `test_nights_across_month_boundary` resultava em `-28` diárias para uma reserva de `2025-01-30` a `2025-02-02`, quando o esperado eram `3` diárias.
* **Causa Raiz:** O cálculo de noites estava sendo feito por subtração ingênua dos atributos de dia (`checkout.day - checkin.day`, resultando em `2 - 30 = -28`), desconsiderando o mês e ano das datas.
* **Solução:** A lógica foi refatorada para calcular a diferença absoluta entre objetos de data completos (`checkout_date - checkin_date`), obtendo os dias através de `(checkout - checkin).days`, respeitando a passagem dos meses e anos civis e multiplicando pela quantidade de quartos (`rooms`).

---

### Bug 3: Inclusão de Reservas `NO_SHOW` nos Totais Financeiros
* **Arquivo:** `src/onfly_report/services/status_policy.py`
* **Sintoma:** O teste `test_status_rules_exclude_cancelled_and_no_show` resultou em `120.00` em vez de `100.00`, somando o valor de uma reserva `NO_SHOW`.
* **Causa Raiz:** A regra de filtro excluía apenas reservas com status `CANCELLED`, permitindo que reservas `NO_SHOW` continuassem compondo o somatório financeiro (`total_amount_brl`) e a média de diárias.
* **Solução:** O critério de agregação financeira foi restringido explicitamente para considerar apenas reservas com status `CONFIRMED` (`status == "CONFIRMED"`), garantindo que tanto `CANCELLED` quanto `NO_SHOW` sejam contabilizados no resumo quantitativo, mas excluídos do faturamento seguindo: `return status not in ("CANCELLED", "NO_SHOW")`.

---

### Bug 4: Ordenação Crescente no Ranking de Hotéis (Top N)
* **Arquivo:** `src/onfly_report/services/report_builder.py`
* **Sintoma:** O teste `test_top_hotels_are_ranked_descending` retornava hotéis com menores faturamentos (`['H04', 'H03', 'H02']`) em vez dos maiores (`['H01', 'H02', 'H03']`).
* **Causa Raiz:** A função `sorted()` utilizava uma chave ascendente por padrão (`key=lambda item: (item[1]["amount"], item[0])`). Ao aplicar a fatia `[: self._top_n]`, eram selecionados os primeiros hotéis de menor receita.
* **Solução:** A ordenação foi alterada para aplicar sinal negativo ao valor financeiro na tupla da chave de ordenação:
`ranked_hotels = sorted(hotels.items(), key=lambda item: (-item[1]["amount"], item[0]))`.

## 2. Validação e Execução

### Suíte de Testes Automatizados
Após a implementação das correções em cada um dos serviços afetados, a suíte de testes unitários e de regressão foi executada com o ambiente virtual ativo, obtendo 100% de cobertura nos cenários avaliados.

```bash
Comando: 

python -m pytest -v

Resultado:

================================== test session starts ==================================
platform win32 -- Python 3.11.9, pytest-9.2.0, pluggy-1.7.0 -- .venv\Scripts\python.exe
cachedir: .pytest_cache
rootdir: C:\Users\gomes\Documents\Projetos\desafio-onfly-gabriel
configfile: pyproject.toml
testpaths: tests
collected 5 items

tests/test_currency.py::test_currency_conversion_to_brl PASSED                    [ 20%]
tests/test_nights.py::test_nights_across_month_boundary PASSED                    [ 40%]
tests/test_regression_report.py::test_regression_report_on_sample PASSED          [ 60%]
tests/test_status_rules.py::test_status_rules_exclude_cancelled_and_no_show PASSED [ 80%]
tests/test_top_hotels.py::test_top_hotels_are_ranked_descending PASSED            [100%]

================================== 5 passed in 0.78s ==================================