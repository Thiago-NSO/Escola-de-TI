# Plano Arquitetural e Decisões Técnicas — Zona Azul Digital

Diretrizes de arquitetura, dependências, armazenamento e execução para geração autônoma pelo Kimi 2.8.

## 1. Stack Tecnológica e Justificativas
- **Linguagem**: Python 3.12 (simplicidade sintática e geração determinística sem ambiguidades de tipos).
- **Framework Web**: FastAPI com Pydantic v2 (validação estrita de contratos JSON, regex de placas e serialização de datas).
- **Servidor ASGI**: Uvicorn escutando na porta 8003.
- **Banco de Dados**: SQLite em memória ou arquivo único (elimina dependências externas de infraestrutura e viabiliza testes isolados).
- **Testes**: Pytest com Starlette `TestClient` para validação rápida e síncrona dos endpoints HTTP.

## 2. Arquitetura Modular Proposta
A geração do código deve seguir separação explícita de responsabilidades:
- `main.py`: Inicialização da aplicação FastAPI e configuração dos middlewares.
- `schemas.py`: Modelos Pydantic de entrada, saída e respostas de erro.
- `domain.py`: Funções puras de cálculo de tarifas, arredondamento de frações (15 min / 125 centavos), teto diário (8000) e tolerância (15 min).
- `repository.py`: Acesso e persistência dos bilhetes em SQLite.
- `routers.py`: Definição das rotas (UC1 a UC8) e conversão de exceções em códigos HTTP.

## 3. Gestão Temporal e Determinismo
- Todas as operações utilizam objetos de data/hora associados ao timezone `-03:00` (`zoneinfo.ZoneInfo("America/Sao_Paulo")` ou `timezone(timedelta(hours=-3))`).
- O suporte ao campo `entrada` no `POST /bilhetes` e controle do relógio no encerramento garantem que os testes determinísticos validem cálculos de horas sem necessidade de espera real (`sleep`).

## 4. Representação Monetária em Centavos
- Valores monetários são manipulados exclusivamente como inteiros representando centavos (`int`), prevenindo problemas clássicos de ponto flutuante binário (IEEE 754) e garantindo conformidade com a rubrica.

## 5. Containerização e Execução
O serviço deve ser empacotado via `Dockerfile` expondo a porta 8003:
- Imagem base: `python:3.12-slim`.
- Instalação via `requirements.txt`.
- Comando de execução: `uvicorn main:app --host 0.0.0.0 --port 8003`.

## 6. Artefatos a Gerar
1. `requirements.txt` (fastapi, uvicorn, pydantic, pytest).
2. Código-fonte da aplicação (`app/`).
3. Suíte de testes automatizados (`tests/`).
4. `Dockerfile` funcional.
5. `README.md` com instruções de execução e testes.
