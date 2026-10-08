# Matriz de Testes e Casos de Borda — Zona Azul Digital

Especificação dos cenários de teste determinísticos para validação das regras contratuais da variante (Tarifa: 500, Fração: 15, Tolerância: 15, Teto: 8000).

## 1. Casos de Borda de Tempo e Cobrança (UC2 e UC7)

| ID | Cenário | Minutos Decorridos | Cálculo de Frações | Valor Esperado (centavos) |
| :--- | :--- | :--- | :--- | :--- |
| TC-01 | Limite exato da tolerância | 15 min | Tolerância aplicada ($\le$ 15 min) | 0 |
| TC-02 | Adjacência de tolerância (+1 min) | 16 min | Cobrança integral: 2 frações de 15 min (30 min cobrados) | 250 |
| TC-03 | Fração exata | 30 min | 2 frações de 15 min | 250 |
| TC-04 | Adjacência de fração (+1 min) | 31 min | 3 frações de 15 min (45 min cobrados) | 375 |
| TC-05 | Uma hora cheia | 60 min | 4 frações de 15 min | 500 |
| TC-06 | Permanência longa / Teto diário | 1200 min (20h) | Frações superam teto ($80 \times 125 = 10000$) | 8000 |

> [!WARNING]
> Passou da tolerância de 15 minutos, cobra integralmente desde o primeiro minuto. A tolerância NÃO é abatida do tempo total.

## 2. Validações e Estados (UC1, UC5, UC8)

| ID | Operação | Estado Prévio | Entrada | Status HTTP | Resposta Esperada |
| :--- | :--- | :--- | :--- | :--- | :--- |
| TC-07 | Abertura com placa inválida | N/A | `{"placa": "ABC12"}` | 422 | `{"erro": "placa_invalida"}` |
| TC-08 | Abertura com entrada inválida | N/A | `{"placa": "ABC1D23", "entrada": "data_ruim"}` | 422 | `{"erro": "entrada_invalida"}` |
| TC-09 | Conflito de vaga (UC8) | Placa "ABC1D23" já em aberto | `{"placa": "ABC1D23"}` | 409 | `{"erro": "bilhete_em_aberto"}` |
| TC-10 | Reabertura após encerramento | Placa "ABC1D23" encerrada | `{"placa": "ABC1D23"}` | 201 | `status: "aberto"` |
| TC-11 | Reabertura após cancelamento | Placa "ABC1D23" cancelada | `{"placa": "ABC1D23"}` | 201 | `status: "aberto"` |
| TC-12 | Cancelar bilhete encerrado | Bilhete com status "encerrado" | ID do bilhete | 409 | `{"erro": "bilhete_nao_aberto"}` |
| TC-13 | Cancelar bilhete inexistente | Bilhete ID 99999 | ID 99999 | 404 | `{"erro": "bilhete_nao_encontrado"}` |

## 3. Relatório Diário e Arredondamento (UC4)

| ID | Cenário | Dados de Encerramento no Dia | Tempo Médio Calculado | Resposta Esperada |
| :--- | :--- | :--- | :--- | :--- |
| TC-14 | Dia sem movimentações | Nenhum bilhete encerrado na data | 0 | `{"total_bilhetes": 0, "faturamento_centavos": 0, "tempo_medio_minutos": 0}` |
| TC-15 | Arredondamento half-up (.5) | Bilhete 1: 15 min; Bilhete 2: 16 min (Média: 15.5) | 15.5 $\rightarrow$ 16 min | `tempo_medio_minutos: 16` |
| TC-16 | Bilhetes abertos ignorados | 1 encerrado (20 min, 250 centavos) e 2 abertos | Apenas o encerrado conta | `{"total_bilhetes": 1, "tempo_medio_minutos": 20}` |
| TC-17 | Parâmetro de data inválido | Query `?data=07-10-2026` | Formato fora de AAAA-MM-DD | 422 `{"erro": "data_invalida"}` |
