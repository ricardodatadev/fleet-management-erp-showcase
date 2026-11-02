# ESPECIFICAÇÃO TÉCNICA 01: STACK TECNOLÓGICA & INFRAESTRUTURA

**Sistema:** GOF — Gestão Operacional de Frotas (antes SIGOF-M; o nome vem de fonte única, ver F1-33)  
**Versão Documental:** 1.0.0  
**Data:** 05 de Outubro de 2026  
**Status:** Aprovado para Arquitetura  
**Documento PAI:** Especificação de Requisitos de Software (SRS v2.0.0)

---

## 1. VISÃO GERAL DA ARQUITETURA

O **GOF** adota uma **Arquitetura Modular Centrada no Laravel (Core Monolith Clean Architecture)**, complementada por **dois microserviços especializados de escopo delimitado** (Python para IA/Pesquisa Operacional e Node.js para Telemetria/Real-Time).

Esta abordagem garante que o **Core Transacional (OLTP)** permaneça coeso, seguro e de fácil manutenção, enquanto demandas de altíssima concorrência I/O (WebSockets/Telemetria) e processamento científico/IA rodam em runtimes nativamente otimizados para esses domínios.

                           ┌───────────────────────────────────────────┐
                           │   SPA / Mobile PWA (React + TypeScript)   │
                           └─────────────────────┬─────────────────────┘
                                                 │
                                        HTTPS / REST / WebSockets
                                                 │
                                                 ▼
                           ┌───────────────────────────────────────────┐
                           │     CORE BACKEND (PHP 8.3 / Laravel 11)   │
                           │   • OLTP & Regras de Negócio             │
                           │   • Data Agent (Queries Determinísticas)  │
                           │   • Orquestração & RBAC                   │
                           └───────┬───────────────────┬───────────────┘
                                   │                   │
                          REST / HTTP                  │ Async Queue
                          (Interno)                    │ (Redis / RabbitMQ)
                                   │                   │
                                   ▼                   ▼
       ┌───────────────────────────────┐   ┌───────────────────────────────┐
       │  PYTHON SERVICE (FastAPI)     │   │  NODE.JS SERVICE (TypeScript) │
       │  • Pipeline RAG (LangChain)   │   │  • Worker de Telemetria / IoT │
       │  • Solver Matemático (OR-Tools)│   │  • Sincronização Real-time    │
       └───────────────────────────────┘   └───────────────────────────────┘
                                   │                   │
                                   └─────────┬─────────┘
                                             │
                                             ▼
                           ┌───────────────────────────────────────────┐
                           │       INFRAESTRUTURA & PERSISTÊNCIA      │
                           │   • PostgreSQL 16 (Dados Transacionais)  │
                           │   • pgvector (Embeddings para RAG)        │
                           │   • Redis 7 (Cache / Queues / PubSub)     │
                           └───────────────────────────────────────────┘


---

## 2. MATRIZ DE TECNOLOGIAS E JUSTIFICATIVA ARQUITETURAL

| Componente | Tecnologia / Runtime | Biblioteca / Framework Principal | Justificativa Técnica & Fronteira de Responsabilidade |
| :--- | :--- | :--- | :--- |
| **Frontend Web / PWA** | Node.js 20 (Build) | **React 18 + TypeScript + Vite** | Interface SPA com tipagem estática. Responsável pelo gerenciamento de estado local, visualizações interativas (Drag-and-Drop do Dashboard via `React DND`, Gêmeo Digital de Pneus via `Konva.js` em SVG Canvas) e operação PWA/Offline local via Service Workers e IndexedDB. |
| **Core Backend (OLTP)** | **PHP 8.3** | **Laravel 11** | Espinha dorsal transacional do sistema. Responsável por regras de negócio de OS, Almoxarifado, Controle Financeiro, Autenticação (OAuth2/Sanctum), RBAC, Auditoria e pelo **Data Agent** (execução de queries SQL determinísticas para KPIs/Relatórios). |
| **Microserviço AI & Solver** | **Python 3.12** | **FastAPI + LangChain + Google OR-Tools** | Processamento científico e IA. Isolado para isolamento de CPU/Memória. Responsável pelo pipeline de **RAG** (ingestão e busca em manuais técnicos) e pelos **Solvers de Otimização** (Planejamento de Manutenção - PMP, Lote Econômico e Escalonamento). |
| **Microserviço Real-Time / IoT** | **Node.js 20** | **TypeScript + Fastify + Socket.io** | Ingestão I/O não-bloqueante de alta concorrência. Responsável pelo parsing de pacotes de telemetria de frota e pela manutenção de conexões bidirecionais WebSockets para o Kanban de OS e alertas do pátio. |
| **Banco de Dados Relacional** | **PostgreSQL 16** | **Extensão `pgvector`** | Banco de dados central ACID. Armazena a estrutura relacional OLTP e estende suporte nativo a buscas por similaridade vetorial (embeddings de RAG) na mesma instância. |
| **Cache, Filas & Pub/Sub** | **Redis 7** | **Redis Server** | Gestão de sessões, cache de consultas de leitura, mensageria e *broker* das filas assíncronas do Laravel e eventos Pub/Sub para o serviço Node.js. |

---

## 3. CONTRATOS E PROTOCOLOS DE COMUNICAÇÃO

### 3.1 Frontend ↔ Core Laravel
* **Protocolo:** HTTPS (RESTful API).
* **Autenticação:** Tokens Bearer via Laravel Sanctum (OAuth2).
* **Formato:** JSON no padrão `JSON:API` com tratamento padronizado de exceções (`status`, `message`, `errors`, `data`).

### 3.2 Core Laravel ↔ Python FastAPI (AI & Solver)
* **Protocolo:** HTTP/2 REST Interno (Rede privada Docker).
* **Padrão de Chamada (RAG):** Síncrono (`POST /api/v1/rag/query`). O Laravel valida a permissão do usuário e repassa a consulta tokenizada com os parâmetros de escopo.
* **Padrão de Chamada (Solver):** Assíncrono (`POST /api/v1/solver/pmp/run`). O Laravel envia os dados de demanda e capacidade, recebe um `job_id`, e o Python notifica a conclusão via *Webhook* de callback no Laravel.

### 3.3 Core Laravel ↔ Node.js Real-Time Service
* **Protocolo:** Pub/Sub via Redis.
* **Fluxo:** Quando uma regra transacional altera o estado do sistema no Laravel (ex: mudança de status da OS no banco), um evento é publicado no Redis (`channel:os-updates`). O serviço Node.js escuta a mensagem e faz o *broadcast* via WebSocket (`Socket.io`) apenas para as salas/usuários autorizados no frontend.

---

## 4. TOPOLOGIA DE INFRAESTRUTURA EM CONTÊINERES (DOCKER)

O ambiente é totalmente contêinerizado via **Docker Compose** para ambiente de desenvolvimento/staging, com manifestos preparados para orquestração **Kubernetes (K8s)** em produção.

### 4.1 Estrutura do Repositório (`docker/`)

fleet-management/
├── docker-compose.yml
├── .env.example
├── docker/
│   ├── nginx/
│   │   └── default.conf            # Proxy reverso central e roteamento
│   ├── laravel/
│   │   ├── Dockerfile              # PHP 8.3 FPM
│   │   └── php.ini
│   ├── python-ai/
│   │   ├── Dockerfile              # Python 3.12 + FastAPI / Uvicorn
│   │   └── requirements.txt
│   ├── node-realtime/
│   │   ├── Dockerfile              # Node.js 20 + TypeScript (Production build)
│   │   └── package.json
│   └── postgres/
│       └── init-pgvector.sql       # Script de ativação da extensão pgvector


### 4.2 Mapeamento dos Serviços no `docker-compose.yml`

```yaml
version: '3.8'

services:

  # Proxy Reverso / Servidor Web
  gof-web:
    image: nginx:alpine
    container_name: gof-web
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./docker/nginx/default.conf:/etc/nginx/conf.d/default.conf
      - ./frontend/dist:/var/www/frontend
    depends_on:
      - gof-app
      - gof-node
    networks:
      - gof-network

  # Core Backend - PHP/Laravel FPM
  gof-app:
    build:
      context: .
      dockerfile: docker/laravel/Dockerfile
    container_name: gof-app
    volumes:
      - ./backend:/var/www/html
    environment:
      - DB_HOST=gof-db
      - REDIS_HOST=gof-redis
      - PYTHON_SERVICE_URL=http://gof-python:8000
    depends_on:
      - gof-db
      - gof-redis
    networks:
      - gof-network

  # Worker de Filas do Laravel (Horizon / Queue)
  gof-worker:
    build:
      context: .
      dockerfile: docker/laravel/Dockerfile
    container_name: gof-worker
    command: php /var/www/html/artisan horizon
    volumes:
      - ./backend:/var/www/html
    depends_on:
      - gof-app
      - gof-redis
    networks:
      - gof-network

  # Microserviço Python (RAG + Solver)
  gof-python:
    build:
      context: ./services/python-ai
      dockerfile: Dockerfile
    container_name: gof-python
    ports:
      - "8000:8000"
    environment:
      - DB_HOST=gof-db
    depends_on:
      - gof-db
    networks:
      - gof-network

  # Microserviço Node.js (WebSockets & Telemetria)
  gof-node:
    build:
      context: ./services/node-realtime
      dockerfile: Dockerfile
    container_name: gof-node
    ports:
      - "3000:3000"
    environment:
      - REDIS_HOST=gof-redis
    depends_on:
      - gof-redis
    networks:
      - gof-network

  # Banco de Dados PostgreSQL 16 + pgvector
  gof-db:
    image: pgvector/pgvector:pg16
    container_name: gof-db
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: gof_db
      POSTGRES_USER: gof_user
      POSTGRES_PASSWORD: secret_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./docker/postgres/init-pgvector.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - gof-network

  # Redis Cache & Broker
  gof-redis:
    image: redis:7-alpine
    container_name: gof-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - gof-network

networks:
  gof-network:
    driver: bridge

volumes:
  postgres_data:
  redis_data:

5. ESTRATÉGIA DE SEGURANÇA E AMBIENTES

    Gestão de Segredos: Todas as chaves e credenciais sensíveis (chaves de APIs de IA, senhas de banco, segredos JWT) devem ser injetadas exclusivamente via variáveis de ambiente (.env), proibindo hardcode no código-fonte.

    Isolamento de Redes: Apenas o container gof-web (Nginx) expõe portas públicas (80/443). As comunicações entre o Laravel, Python, Node.js e bancos de dados ocorrem dentro da rede privada Docker (gof-network).

    Persistência de Dados: Todos os volumes de banco de dados e arquivos de mídia/manuais (postgres_data, redis_data, e storage local) são mantidos persistentes fora do ciclo de vida dos contêineres.