# Matriz de Testes e Casos de Borda — Zona Azul Digital

Especificação dos cenários de teste determinísticos para validação das regras contratuais da variante (Tarifa: 500, Fração: 15, Tolerância: 15, Teto: 8000).

## 1. Casos de Borda de Tempo, Frações e Teto (UC2 e UC7)

| ID | Cenário | Minutos Decorridos | Cálculo de Frações | Valor Esperado (centavos) |
| :--- | :--- | :--- | :--- | :--- |
| TC-01 | Limite exato da tolerância | 15 min | Tolerância aplicada ($\le$ 15 min) | 0 |
| TC-02 | Adjacência de tolerância (+1 min) | 16 min | Cobrança integral: 2 frações de 15 min (30 min cobrados) | 250 |
| TC-03 | Fração exata | 30 min | 2 frações completas | 250 |
| TC-04 | Adjacência de fração (+1 min) | 31 min | 3 frações completas (45 min cobrados) | 375 |
| TC-05 | Uma hora cheia | 60 min | 4 frações de 15 min | 500 |
| TC-06 | Ponto de corte antes do teto | 946 min | 64 frações completas ($64 \times 125$) | 8000 |
| TC-07 | Excedente do teto diário (+1 fração) | 961 min | 65 frações ($65 \times 125 = 8125$, limitado ao teto) | 8000 |
| TC-08 | Permanência extrema | 1440 min (24h) | Valor bruto extrapolado, trava estritamente no teto | 8000 |

> [!WARNING]
> Passou da tolerância de 15 minutos, cobra integralmente desde o primeiro minuto. A tolerância NÃO é abatida do tempo total.

## 2. Validações, Conflitos e Transições de Estado (UC1, UC2, UC5, UC8)

| ID | Operação | Estado Prévio | Entrada | Status HTTP | Resposta Esperada |
| :--- | :--- | :--- | :--- | :--- | :--- |
| TC-09 | Placa com minúsculas | N/A | `{"placa": "abc1d23"}` | 422 | `{"erro": "placa_invalida"}` |
| TC-10 | Placa com formato/hífen inválido | N/A | `{"placa": "ABC-123"}` | 422 | `{"erro": "placa_invalida"}` |
| TC-11 | Entrada com ISO-8601 inválido | N/A | `{"placa": "ABC1D23", "entrada": "2026/10/07 10:00"}` | 422 | `{"erro": "entrada_invalida"}` |
| TC-12 | Conflito de vaga (UC8) | Placa "ABC1D23" já em aberto | `{"placa": "ABC1D23"}` | 409 | `{"erro": "bilhete_em_aberto"}` |
| TC-13 | Reabertura após encerramento | Placa "ABC1D23" encerrada | `{"placa": "ABC1D23"}` | 201 | `status: "aberto"` |
| TC-14 | Reabertura após cancelamento | Placa "ABC1D23" cancelada | `{"placa": "ABC1D23"}` | 201 | `status: "aberto"` |
| TC-15 | Encerramento duplo | Bilhete já encerrado | ID do bilhete | 409 | `{"erro": "bilhete_ja_encerrado"}` |
| TC-16 | Cancelar bilhete já encerrado | Bilhete com status "encerrado" | ID do bilhete | 409 | `{"erro": "bilhete_nao_aberto"}` |
| TC-17 | Cancelar bilhete inexistente | ID 99999 inexistente | ID 99999 | 404 | `{"erro": "bilhete_nao_encontrado"}` |

## 3. Relatório Diário e Arredondamentos de Tempo Médio (UC4)

| ID | Cenário | Dados de Encerramento no Dia | Tempo Médio Calculado | Resposta Esperada |
| :--- | :--- | :--- | :--- | :--- |
| TC-18 | Dia sem movimentações | Nenhum bilhete encerrado na data | 0 | `{"total_bilhetes": 0, "faturamento_centavos": 0, "tempo_medio_minutos": 0}` |
| TC-19 | Arredondamento half-up (.5) | 2 bilhetes encerrados: 15 min e 16 min (Média: 15.5) | Fração .5 arredonda para cima $\rightarrow$ 16 | `tempo_medio_minutos: 16` |
| TC-20 | Contraprova: Arredondamento para baixo (< .5) | 3 bilhetes encerrados: 10 min, 10 min e 11 min (Média: 10.33) | Fração .33 arredonda para baixo $\rightarrow$ 10 | `tempo_medio_minutos: 10` |
| TC-21 | Isolamento de bilhetes abertos | 1 encerrado (20 min) e 2 abertos | Apenas o encerrado conta no cálculo | `total_bilhetes: 1`, `tempo_medio_minutos: 20` |
| TC-22 | Formato de data inválido | Query `?data=2026-13-40` ou `?data=hoje` | Data fora do padrão AAAA-MM-DD | 422 `{"erro": "data_invalida"}` |
