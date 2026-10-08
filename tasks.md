# Plano de Tarefas de Implementação — Zona Azul Digital

Guia sequencial de tarefas para o modelo Kimi 2.8 executar a geração e verificação da aplicação.

## Diretrizes Operacionais para a IA
1. Ler e respeitar integralmente as restrições de `constitution.md` e o contrato de `spec.md`.
2. Executar as tarefas em ordem estrita de dependência.
3. Não truncar código ou omitir implementações essenciais.
4. Manter mensagens e logs sucintos para não consumir desnecessariamente a janela de contexto de 256k tokens.

## Tarefas Decompostas

### Tarefa 1: Setup do Ambiente e Manifesto de Dependências
- **Ações**: Criar a estrutura de diretórios e o arquivo `requirements.txt` contendo `fastapi`, `uvicorn`, `pydantic` e `pytest`.
- **Critério de Conclusão**: Ambiente configurado com dependências especificadas e sem conflitos de versão.

### Tarefa 2: Módulo de Domínio e Regras de Negócio
- **Ações**: Implementar as regras puras de cálculo de frações de 15 minutos, tarifa proporcional (125 centavos), teto diário de 8000 centavos, regra de tolerância de 15 minutos e arredondamento aritmético de tempo médio.
- **Artefato**: Módulo de domínio com funções puras desacopladas de frameworks web.
- **Critério de Conclusão**: Funções matemáticas cobrindo com exatidão todos os casos de borda descritos em `tests.md` (TC-01 a TC-08, TC-19 e TC-20).

### Tarefa 3: Esquemas Pydantic e Persistência
- **Ações**: Definir modelos Pydantic com validação de placa (7 caracteres maiúsculos) e datas ISO-8601 (-03:00). Criar repositório SQLite para persistência dos bilhetes.
- **Artefatos**: Esquemas de requisição/resposta e repositório de dados.
- **Critério de Conclusão**: Operações de salvar bilhete, buscar por placa e consultar por data funcionando com integridade de tipos.

### Tarefa 4: Endpoints HTTP e Tratamento de Erros
- **Ações**: Implementar as rotas da API em FastAPI (UC1 a UC8) respeitando rigorosamente os códigos HTTP (200, 201, 404, 409, 422) e payloads de erro `{"erro": "<codigo>"}`.
- **Artefato**: Routers da API integrados na aplicação principal.
- **Critério de Conclusão**: Todos os endpoints respondendo exatamente aos formatos canônicos definidos em `spec.md`.

### Tarefa 5: Suíte de Testes, Containerfile e Documentação
- **Ações**: Implementar a suíte completa de testes com Pytest cobrindo os IDs de TC-01 a TC-22. Criar `Dockerfile` expondo a porta 8003 e `README.md` com instruções de execução.
- **Critério de Conclusão**: Suíte de testes executando com 100% de sucesso e container pronto para inicialização na porta 8003.
