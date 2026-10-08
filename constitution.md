# Constitution — Zona Azul Digital

Regras permanentes, invariantes inegociáveis e diretrizes globais para a API de bilhetes de estacionamento rotativo.

## 1. Tratamento Monetário Estrito
- **Centavos Inteiros**: Todos os valores monetários (`valor_centavos`, `faturamento_centavos`, tarifas) DEVEM ser manipulados, calculados e serializados como números inteiros (`integer`). É proibido o uso de tipos de ponto flutuante (`float`) para dinheiro.
- **Formatação Limpa**: A API nunca retorna prefixos de moeda (ex: "R$") ou números decimais com vírgula/ponto em valores monetários.

## 2. Padrões Temporais e Fuso Horário
- **Fuso Horário Obrigatório**: Todas as datas/horas (`entrada`, `saida`) DEVEM ser formatadas em ISO-8601 estrito com deslocamento `-03:00` (exemplo: `2026-10-07T14:30:00-03:00`).
- **Gancho de Testabilidade**: A API DEVE aceitar o campo opcional `entrada` no `POST /bilhetes`. Quando informado, esse carimbo substitui o horário atual do sistema.
- **Agrupamento Diário**: O relatório diário (`UC4`) considera estritamente a data civil local (`-03:00`) do momento de encerramento (`saida`) dos bilhetes.

## 3. Conformidade Contratual e Respostas de Erro
- **Formato Canônico de Erro**: Toda resposta de erro deve retornar um JSON com chave única `erro` contendo a string exata definida no contrato: `{"erro": "<codigo_erro>"}`.
- **Códigos HTTP Imutáveis**: Utilizar estritamente os códigos 200, 201, 404, 409 e 422 conforme definidos no contrato. Não adicionar chaves adicionais como `message`, `details` ou `status`.

## 4. Regras Aritméticas e Arredondamento
- **Fração de Cobrança**: O tempo faturado é sempre calculado em frações de 15 minutos arredondadas para cima (`ceil`). Fração exata cobra 1 fração; 1 minuto a mais cobra a fração subsequente.
- **Tempo Médio**: O campo `tempo_medio_minutos` no relatório diário aplica arredondamento aritmético de meio ponto para cima (half-up: $\ge 0.5$ arredonda para cima). É proibido utilizar arredondamento bancário (`half-even`).

## 5. Qualidade do Entregável
- A aplicação gerada deve ser autônoma, conter testes unitários e de integração determinísticos, manifesto de dependências e configuração de container na porta 8003.
