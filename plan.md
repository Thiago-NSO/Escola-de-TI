# Plano Arquitetural e Decisões Técnicas — Zona Azul Digital

Diretrizes de arquitetura, dependências, armazenamento e execução para geração autônoma pelo Kimi 2.8.

## 1. Stack Tecnológica e Justificativas
- **Linguagem**: Python 3.12 (sintaxe direta, ampla familiaridade do modelo gerador e determinismo de execução).
- **Framework Web**: FastAPI com Pydantic v2 (validação estrita de contratos via schemas, tipagem estática e serialização de ISO-8601).
- **Servidor ASGI**: Uvicorn escutando na porta 8003.
- **Banco de Dados**: SQLite com persistência em arquivo único local ou memória (elimina dependências de rede/serviços externos e facilita testes isolados).
- **Testes**: Pytest com Starlette `TestClient` para execução síncrona, veloz e sem necessidade de subprocessos.

## 2. Arquitetura Modular Proposta
A geração do código deve seguir separação explícita de responsabilidades:
- `main.py`: Inicialização da aplicação FastAPI e configuração dos routers.
- `schemas.py`: Modelos Pydantic para validação de entrada, saída e formato canônico de erros.
- `domain.py`: Funções puras de cálculo de tarifas, arredondamento para cima em frações de 15 min, teto de 8000 centavos, tolerância de 15 min e média com arredondamento 0.5 para cima.
- `repository.py`: Camada de acesso a dados e consultas SQLite.
- `routers.py`: Implementação dos endpoints (UC1 a UC8) e conversão de erros de domínio em códigos HTTP.

## 3. Gestão Temporal e Determinismo
- **Tratamento de Fuso Horário**: Decisão de utilizar exclusivamente `datetime.timezone(datetime.timedelta(hours=-3))` nativo da biblioteca padrão `datetime` do Python. Isso elimina a dependência do pacote `tzdata` em containers Docker minimalistas (`python:3.12-slim`) e assegura formatação estrita em ISO-8601 com offset `-03:00`.
- **Determinismo nos Testes**: O endpoint `POST /bilhetes` aceita o campo opcional `entrada` para permitir que cenários de teste simulem o passado sem depender de esperas reais (`time.sleep`) ou de mocks externos de relógio.
- **Data do Relatório**: O filtro `?data=AAAA-MM-DD` do relatório diário considera a data civil local (`-03:00`) registrada no carimbo de `saida` dos bilhetes encerrados.

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
