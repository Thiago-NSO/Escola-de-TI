# Especificação Funcional — Zona Azul Digital

Especificação técnica dos endpoints, regras de negócio e critérios de aceite mensuráveis.

## 1. Parâmetros da Variante
- **TARIFA_HORA_CENTAVOS**: 500
- **FRACAO_MINUTOS**: 15 (Valor por fração: 500 ÷ (60 ÷ 15) = 125 centavos)
- **TETO_DIARIO_CENTAVOS**: 8000
- **TOLERANCIA_MINUTOS**: 15
- **PORTA_SERVICO**: 8003

## 2. Modelo de Dados
- **Bilhete**: `id` (int sequencial), `placa` (string, 7 caracteres alfanuméricos maiúsculos), `entrada` (ISO-8601 com `-03:00`), `saida` (ISO-8601 opcional), `status` ("aberto", "encerrado", "cancelado"), `minutos` (int opcional), `valor_centavos` (int opcional).
- **Relatório**: `data` (AAAA-MM-DD), `total_bilhetes` (int), `faturamento_centavos` (int), `tempo_medio_minutos` (int).

## 3. Casos de Uso (UC)

### UC1 — Abrir Bilhete
- **Rota**: `POST /bilhetes`
- **Body**: `{"placa": "ABC1D23", "entrada": "2026-10-07T10:00:00-03:00"}` (`entrada` é opcional).
- **Critério de Aceite**: Retorna status **201** com `id`, `placa`, `entrada` e `status: "aberto"`. Placa inválida ou ausente retorna **422** `{"erro": "placa_invalida"}`. `entrada` inválida retorna **422** `{"erro": "entrada_invalida"}`.

### UC2 — Encerrar Bilhete
- **Rota**: `POST /bilhetes/{id}/encerramento`
- **Critério de Aceite**: Retorna status **200** com `id`, `placa`, `entrada`, `saida`, `minutos` e `valor_centavos`. 
- **Cálculo**:
  - Duração $\le$ 15 min: `valor_centavos: 0`.
  - Duração $> 15$ min: cobra todas as frações de 15 min arredondando para cima a 125 centavos por fração desde o início.
  - O valor final não pode ultrapassar o teto diário de 8000 centavos.

### UC3 — Listar Ativos
- **Rota**: `GET /bilhetes/ativos`
- **Critério de Aceite**: Retorna status **200** com array de bilhetes em status `"aberto"`, ordenados por `entrada` decrescente (mais recentes primeiro). Retorna `[]` se não houver ativos.

### UC4 — Relatório Diário
- **Rota**: `GET /relatorios/diario?data=AAAA-MM-DD`
- **Critério de Aceite**: Retorna status **200** contendo `data`, `total_bilhetes`, `faturamento_centavos` e `tempo_medio_minutos`. Considera apenas bilhetes com `saida` na data informada. `tempo_medio_minutos` arredonda 0.5 para cima. Data fora do padrão retorna **422** `{"erro": "data_invalida"}`.

### UC5 — Cancelar Bilhete
- **Rota**: `POST /bilhetes/{id}/cancelamento`
- **Critério de Aceite**: Retorna status **200** com `status: "cancelado"`. Não gera campos `saida` nem `valor_centavos`. Apenas bilhetes `"aberto"` podem ser cancelados; caso contrário retorna **409** `{"erro": "bilhete_nao_aberto"}`.

### UC6 — Histórico por Placa
- **Rota**: `GET /bilhetes?placa=ABC1D23`
- **Critério de Aceite**: Retorna status **200** com lista de todos os bilhetes daquela placa (qualquer status), ordenados por `entrada` decrescente. Placa sem bilhetes retorna `[]`.

### UC7 — Tolerância Gratuita
- **Critério de Aceite**: Bilhetes com até 15 minutos de permanência têm `valor_centavos: 0`. A partir de 16 minutos, a tolerância deixa de existir e cobra-se integralmente desde o minuto 1 (ex: 16 min = 2 frações = 250 centavos).

### UC8 — Uma Vaga por Placa
- **Critério de Aceite**: Tentativa de abrir bilhete para placa com bilhete em status `"aberto"` retorna status **409** `{"erro": "bilhete_em_aberto"}`.

## 4. Tabela Canônica de Erros
| Situação | Status HTTP | Resposta JSON |
| :--- | :--- | :--- |
| Placa ausente ou inválida | 422 | `{"erro": "placa_invalida"}` |
| Entrada fora de ISO-8601 | 422 | `{"erro": "entrada_invalida"}` |
| Data fora de AAAA-MM-DD | 422 | `{"erro": "data_invalida"}` |
| Bilhete inexistente | 404 | `{"erro": "bilhete_nao_encontrado"}` |
| Encerrar bilhete já encerrado | 409 | `{"erro": "bilhete_ja_encerrado"}` |
| Cancelar bilhete não aberto | 409 | `{"erro": "bilhete_nao_aberto"}` |
| Abrir bilhete com placa já ocupada | 409 | `{"erro": "bilhete_em_aberto"}` |
