# ESPECIFICAÇÃO DE REQUISITOS DE SOFTWARE (SRS)

**Sistema:** GOF — Gestão Operacional de Frotas (antes SIGOF-M; o nome vem de fonte única, ver F1-33)  
**Versão:** 2.0.0 *(Revisão Operacional de Negócio - PCM)*  
**Data:** 05 de Outubro de 2026  
**Status:** Aprovado para Arquitetura e Modelagem de Banco de Dados  
**Autores:** Ricardo Pereira / Analista Sênior de PCM  

---

## 1. INTRODUÇÃO

### 1.1 Propósito
Este documento especifica os Requisitos Funcionais (RF), Requisitos Não Funcionais (RNF) e Regras de Negócio (RN) para o desenvolvimento do **GOF**. Seu objetivo é servir como contrato funcional e técnico definitivo para as fases de Arquitetura de Software, Modelagem de Dados, APIs e Engenharia de Qualidade (QA), alinhando o rigor técnico com a flexibilidade exigida pelo chão de fábrica.

### 1.2 Escopo do Sistema
O GOF é uma plataforma de gestão de frotas e oficinas agroindustriais e corporativas que centraliza o ciclo de vida das Ordens de Serviço (OS), planos de manutenção preventiva/corretiva, gestão de pneus e controle de almoxarifado. 

O sistema integra:
* **Módulo Otimizador (Solver Prescritivo):** Atua no Planejamento de Manutenção de Médio/Longo Prazo (PMP) e no escalonamento de oficina/compras.
* **Assistente Virtual e Chat de Dados:** Arquitetura híbrida que combina **RAG (Retrieval-Augmented Generation)** para documentação técnica e **Data Agent** sobre consultas determinísticas/queries nativas para relatórios e indicadores.
* **Painel Parâmetro-Centrado:** Permite flexibilizar travas operacionais, aprovações e obrigatoriedades conforme a maturidade de cada filial ou unidade operacional.

---

## 2. REGRAS DE NEGÓCIO (RN)

+--------------------------------------------------------------------------------------------------+
|                                PAINEL DE PARÂMETROS DO GESTOR                                   |
+--------------------------------------------------------------------------------------------------+
| [RN-001] Modo de Disparo de Preventivas:    ( ) Automático          (X) Painel de Sugestão (PCM) |
| [RN-002] Travar Fechamento sem Mão de Obra: ( ) Sim                 (X) Não (Conforme Categoria) |
| [RN-003] Permitir Baixa com Pendência Fiscal: (X) Sim                 ( ) Não                      |
| [RN-004] Comportamento de Alerta de Garantia: (X) Aviso Não Bloqueante ( ) Bloqueio Rígido          |
+--------------------------------------------------------------------------------------------------+


### RN-001 – Gatilho de Preventiva, Tolerância Parametrizada e Modo de Disparo
1. **Critérios de Monitoramento:** O sistema deve monitorar continuamente os critérios do Plano de Manutenção: Odômetro (km), Horímetro (h), Calendário (dias) e Consumo Acumulado (litros).
2. **Painel de Oportunidades (Pré-Alerta):** Ao atingir a Margem de Antecedência Parametrizada por Família de Equipamento (padrão: 90% da meta ou X dias/horas antes), o ativo entra no status *"Preventiva Apta para Agendamento"*, permitindo reserva prévia de peças no almoxarifado.
3. **Modo de Disparo Parametrizável (Geral ou por Família):**
   * **Modo Sugestão (Padrão Operacional):** Ao atingir a meta do primeiro critério, o equipamento é listado no *Painel de Oportunidades do PCM*. A OS só é gerada quando o PCM confirmar a programação/liberação junto à Operação.
   * **Modo Automático:** A OS Preventiva é aberta automaticamente no status *"Programada"* assim que o primeiro critério atingir a meta.
4. **Margem de Tolerância e Aderência Dinâmica:** A janela de tolerância para considerar a manutenção como *"Aderente ao Plano"* deixará de ser um valor global fixo e será cadastrada por **Família de Equipamento / Criticidade do Ativo** (ex: Colhedoras no pico da safra possuem margem maior que veículos de apoio). Execuções fora da janela serão auditadas como *"Antecipada"* ou *"Atrasada/Estourada"*.
5. **Tratamento de Inconsistência de Telemetria:** Leituras fora do padrão operacional (ex: salto de horímetro > 24h em um único dia ou variação negativa de odômetro) devem ser isoladas em uma *Fila de Inconsistência*, gerando alerta sem disparar preventivas indevidas.

### RN-002 – Regras e Obrigatoriedades Configuráveis de Fechamento de OS
1. **Matriz de Obrigatoriedades Parametrizável:** O Gestor Geral/PCM pode configurar via painel de administração as exigências para o encerramento operacional da OS:
   * Exigir ao menos 1 apontamento de mão de obra (Sim/Não - por Categoria de Serviço).
   * Exigir requisição/aplicação de material (Sim/Não - por Categoria de Serviço).
   * Exigir preenchimento de causa/defeito (Sim/Não).
2. **Exceções Nativas:**
   * **Serviços sem Aplicação de Peças:** Categorias como *"Inspeção"*, *"Ajuste/Regulagem"*, *"Diagnóstico"*, *"Lavagem"* e *"Socorro Volante"* podem ser encerradas sem requisição de almoxarifado mediante parecer técnico.
   * **Serviços Terceirizados:** Exigem a vinculação da OS Terceirizada/Pedido de Compra e o registro/anexo do Documento Fiscal (NF) do prestador, dispensando apontamento de mão de obra interna.
3. **Cancelamento Auditado:** O cancelamento de OSs exige justificativa obrigatória registrada em histórico de auditoria.

### RN-003 – Fluxo de Almoxarifado, Baixa de Estoque e Trativa de Pendência Fiscal
1. **Separação de Papéis:** A OS registra a *Solicitação/Aplicação de Peças*. A *Baixa Física e Fiscal* do saldo de estoque ocorre estritamente dentro do Módulo de Almoxarifado/Depósito através da efetivação da requisição.
2. **Atendimento com Saldo Zerado no Sistema (Pendência Fiscal / Recebimento Emergencial):**
   * Caso a peça exista fisicamente na oficina (chegada de emergência/plantão), mas o saldo no sistema esteja zerado por falta de lançamento da NF/XML pelo Compras/Recebimento, o Almoxarife/PCM pode selecionar a opção **"Baixa Emergencial por Pendência Fiscal"**.
   * O sistema realiza a baixa do saldo (gerando provisoriamente saldo negativo controlado ou registro em conta transitória), libera o item para a OS sem travar o veículo no box e gera um alerta automático no Módulo de Compras/Recebimento: *"Item [X] aplicado na OS [123] aguardando escrituração da NF"*.
   * Na entrada posterior da NF, o sistema realiza a reconciliação e recalcula o Custo Médio Apropriado na OS de forma retroativa.
3. **Indisponibilidade Física (Falta de Peça):** Se o item não existir fisicamente, o almoxarife sinaliza a falta. A OS muda para status *"Aguardando Peça (Imobilizado)"*, interrompendo a contagem do tempo de reparo (MTTR) imputável à oficina e gerando necessidade de compra.

### RN-004 – Rastreabilidade de Componentes e Alerta Não Bloqueante de Garantia
1. **Controle de Itens Críticos:** Peças classificadas como *"Componentes de Garantia"* (ex: motores, câmbios, turbinas, compressores, alternadores) exigem o registro de número de série/lote na aplicação.
2. **Alerta Visual de Reincidência (Garantia Ativa):** Ao solicitar/atender um item crítico para uma OS, o sistema pesquisa o histórico do equipamento. Se o mesmo item tiver sido aplicado dentro da janela de garantia cadastrada (dias, km ou horas):
   * Exibir um **Banner de Alerta Crítico (Amarelo/Vermelho)** no Almoxarifado e no PCM:  
     > **⚠️ POSSÍVEL GARANTIA ATIVA:** *Item aplicado há 28 dias / 320h na OS nº 008452 (Garantia vigora até 90 dias).*
3. **Caráter Não Bloqueante:** O alerta **não impede** a liberação imediata da peça para a OS de socorro. O sistema disponibiliza um ação rápida opcional: `[ Gerar Ficha de Garantia / Sinistro ]`. Se acionada, a ficha é encaminhada ao setor de Compras/Fornecedores para tratativa posterior, enviando a peça substituída para a quarentena de garantia sem atrasar a liberação da frota.

### RN-005 – Apropriação Financeira Automática
1. **Herança Transparente de Dados:** Toda apropriação de custos de mão de obra, peças e serviços externos é 100% automatizada em background:
   * **Centro de Custo:** Herdado automaticamente do cadastro ativo do Equipamento ou da Operação/Unidade alocada.
   * **Conta Contábil / Elemento de Custo:** Herdado do Grupo/Família da Peça ou da Categoria do Serviço.
2. **Usabilidade:** Os formulários operacionais do mecânico e do almoxarife exibem apenas Identificação do Veículo (Prefixo/Placa) e Categoria, eliminando a digitação de códigos contábeis no chão de fábrica.

### RN-006 – Capacidade Dinâmica do Solver no PMP e na Oficina
1. **Atuação no Planejamento de Médio e Longo Prazo (PMP):** O Solver calcula o nivelamento da carga de manutenção preventiva para os próximos 1 a 12 meses, projetando a utilização da frota (km/h) em relação à janela de safra/produção e à capacidade das oficinas, recomendando antecipações ou postergações seguras para evitar gargalos nos picos operacionais.
2. **Atuação no Escalonamento Diário da Oficina:**
   * Utiliza como linha base a capacidade nominal (Boxes x Mão de Obra x Turno).
   * **Flexibilização:** Permite simular e alocar Ordens de Serviço em *Boxes Virtuais* (Pátio/Pista), agrupar múltiplos mecânicos na mesma OS para redução de MTTR e prever projeções de Horas Extras em situações de pico.
   * **Gargalo e Sobrealocação:** OSs classificadas como *"Emergência / Parada Operacional"* possuem prioridade e reposicionam as demais OSs programadas no painel visual do PCM.

### RN-007 – Matriz de Perfis de Acesso (RBAC) e Alçada de Plantão
1. **Motorista / Operador:** Acesso via app móvel para Checklists Pré-operacionais, consulta de solicitações e registro de telemetria manual.
2. **Mecânico / Técnico:** Acesso ao terminal touch/tablet de oficina para iniciar, pausar e encerrar etapas de OS e solicitar peças ao almoxarifado.
3. **Líder de Oficina / Plantonista:** Permissão para abertura de OSs emergenciais, alteração manual da sequência da fila, autorização de baixas com pendência fiscal (RN-003) e requisições fora do expediente comercial.
4. **PCM / Gestor / Administrador:** Acesso integral ao planejamento (PMP), parametrização das regras de negócio, aprovações financeiras, BI e configurações gerais do sistema.

### RN-008 – Sincronização Offline Resiliente e Tratamento de Conflitos
1. **Arquitetura Offline-First:** O app móvel opera localmente com banco de dados criptografado e registra eventos marcados com carimbo de data/hora (Timestamp UTC) no momento do clique.
2. **Sincronização em Lote:** Ao reconectar à rede, os dados são transmitidos em ordem cronológica estrita.
3. **Tratamento de Conflitos Críticos:** Em caso de divergências estruturais (ex: apontamento de tempo em OS que foi reatribuída ou cancelada no servidor central enquanto o dispositivo estava offline), os dados offline não são descartados. Eles são registrados em uma *Fila de Análise de Conflitos do PCM* para validação manual com 1 clique.

---

## 3. REQUISITOS FUNCIONAIS (RF)

                              ┌─────────────────────────────────────────┐
                              │                   GOF                   │
                              └────────────────────┬────────────────────┘
                                                   │
  ┌──────────────────┬──────────────┼──────────────┬──────────────────┐
  │                  │              │              │                  │
┌─────┴────────┐ ┌───────┴──────┐ ┌─────┴──────┐ ┌─────┴────────┐ ┌───────┴──────┐
│  1. Gestão   │ │  2. Pneus e  │ │  3. Otimiz.│ │ 4. Assistente│ │ 5. Dashboards│
│  de OS/Ofic. │ │  Checklist   │ │  (Solver)  │ │   Chat (AI)  │ │ (Self-Serv.) │
└──────────────┘ └──────────────┘ └────────────┘ └──────────────┘ └────────────┘


### 3.1 Módulo de Ordens de Serviço (OS) e Oficina
* **RF-001 – Abertura, Gestão e Impressão Profissional de OS:** Permitir a abertura manual e automática de OS (Preventiva, Corretiva, Preditiva, Inspeção, Socorro Volante). Gerar o documento espelho em formato PDF profissional otimizado para impressão contendo:
  * Cabeçalho institucional, dados do ativo, localização e medidores atuais (km/h).
  * Histórico recente (data e horímetro da última manutenção executada).
  * Lista de tarefas/checklist de execução, peças e insumos requisitados/entregues.
  * Tabela para apontamento manual de mão de obra (reserva para contingência no papel).
  * Campo para diagnóstico técnico e bloco de assinaturas (Executante e Aceite da Operação).
* **RF-002 – Apontamento Otimizado em Tablet/Mobile (Chão de Fábrica):** Interface touch-first para mecânicos iniciarem, pausarem, solicitarem peças e concluírem serviços em no máximo 3 toques.
* **RF-003 – Motor de Planos de Manutenção:** Processar telemetria e inserções manuais, gerando alertas no *Painel de Oportunidades* e disparando OSs conforme o modo configurado na RN-001.
* **RF-004 – Gestão de Serviços de Terceiros:** Acompanhar orçamentos, aprovações, envio, execução e entrega de frotas em oficinas credenciadas/externas.
* **RF-005 – Parametrização Dinâmica de Nomenclaturas:** Permitir que o administrador renomeie cadastros e agrupamentos de frota, tipos de insumos e centros de custo.

### 3.2 Módulo de Pneus e Checklists
* **RF-006 – Controle do Ciclo de Vida de Pneus:** Rastrear pneus individualmente por número de fogo/série, posição nos eixos, medição de sulcagem (mm), histórico de rodízios, recapeamentos e Custo por Quilômetro (CPK).
* **RF-007 – Checklist Operacional Móvel:** Aplicação móvel para verificação pré-operacional por motoristas/operadores. A reprovação de um item classificado como *"Crítico"* gera automaticamente uma solicitação/OS Corretiva no painel do PCM.

### 3.3 Módulo Otimizador / Prescritivo (Solver)
* **RF-008 – Otimização do Planejamento de Manutenção (PMP):** Executar algoritmos para simular e sugerir o nivelamento do plano de manutenção preventiva de médio/longo prazo, equilibrando a disponibilidade operacional necessária para a colheita/transporte com a capacidade física das oficinas.
* **RF-009 – Otimização de Compras e Estoque (Lote Econômico):** Recomendar quantidades e momentos ideais de compra de peças baseando-se no consumo histórico, lead time do fornecedor, espaço físico e limite orçamentário.
* **RF-010 – Escalonamento da Oficina e Rodízio de Pneus:** Sugerir o sequenciamento otimizado de OSs nos boxes (físicos e pátio) e emitir planos automáticos de rodízio de pneus para maximizar a vida útil da carcaça.

### 3.4 Módulo de Assistente Virtual Híbrido e Chat de Dados (IA)
* **RF-011 – Arquitetura Híbrida de Atendimento por IA:**
  * **Motor RAG (Busca Não Estruturada):** Utilizado exclusivamente para consultar manuais técnicos em PDF, catálogos de fabricantes, boletins de serviço e campo de observações históricas de OS.
  * **Data Agent / Consultas Determinísticas (Busca Estruturada):** Para perguntas sobre indicadores, totais financeiros e listagens quantitativas (ex: *"Qual o MTTR da frota de caminhões no mês passado?"*), a IA mapeia a intenção e consome **queries/APIs nativas preexistentes**, formatando os resultados sem recalcular dados, garantindo precisão matemática de 100% e baixo consumo de tokens.
* **RF-012 – Sugestão de Diagnóstico Técnico:** Analisar sintoma digitado na abertura da OS e sugerir procedimentos com base nos manuais indexados e histórico de reparos da frota.

### 3.5 Módulo de Dashboards Interativos (Self-Service BI) e Relatórios
* **RF-013 – Painéis Customizáveis Drag-and-Drop:** Permite ao usuário organizar widgets visuais, cartões de KPI e tabelas.
* **RF-014 – Alternância Dinâmica de Visualização:** Permitir a troca de tipo de gráfico e alteração nos agrupamentos (por unidade, família, centro de custo e período).
* **RF-015 – Exportação Rápida:** Exportar dados tabulares para os formatos `.xlsx` e `.csv` em no máximo 3 cliques.
* **RF-016 – Indicadores Nativos com Foco em Decisão:**
  * Disponibilidade Física (%) e Taxa de Utilização.
  * MTBF (Tempo Médio Entre Falhas) e MTTR (Tempo Médio para Reparo).
  * Aderência ao Plano de Manutenção Preventiva.
  * Curva ABC de Custos por Ativo / Custo por Quilômetro ou Hora (CPK/CPH).
  * Relatório de Acompanhamento de Pendências Fiscais de Estoque (RN-003).

---

## 4. REQUISITOS NÃO FUNCIONAIS (RNF)

| Código | Categoria | Descrição e Métrica Operacional |
| :--- | :--- | :--- |
| **RNF-001** | **Desempenho** | Consultas a históricos de dados de até 12 meses devem retornar em menos de 2,0 segundos. Consultas do Data Agent e RAG devem entregar a primeira resposta visual em até 3,0 segundos. |
| **RNF-002** | **Disponibilidade**| O sistema web e APIs de integração devem apresentar disponibilidade (_Uptime_) mínima de 99,5% no regime 24/7. |
| **RNF-003** | **Usabilidade (Oficina)** | Interface móvel/tablet projetada sob diretrizes Touch-First (botões de no mínimo 48 x 48px), permitindo a conclusão de um apontamento operacional em no máximo 3 toques. |
| **RNF-004** | **Operação Offline** | O aplicativo móvel deve operar em modo Offline-First com banco local criptografado, permitindo a execução de checklists e apontamentos de OS sem conectividade, sincronizando os dados automaticamente ao detectar rede (RN-008). |
| **RNF-005** | **Segurança** | Autenticação via OAuth2/OpenID Connect com suporte a Controle de Acesso Baseado em Funções (RBAC, RN-007). Todo tráfego de dados deve ser criptografado via TLS 1.3 (em trânsito) e AES-256 (em repouso). |
| **RNF-006** | **Integração (APIs)** | Disponibilizar endpoints RESTful/OpenAPI 3.0 documentados para integração com ERPs corporativos (SAP, TOTVS, etc.), sistemas de telemetria e gateways de abastecimento. |
| **RNF-007** | **Escalabilidade** | Arquitetura baseada em microsserviços/contêineres (Docker/Kubernetes), suportando escalabilidade horizontal automática para absorver picos de sincronização e importação de telemetria. |
| **RNF-008** | **Auditoria** | Manter logs inalteráveis de auditoria (_Audit Trail_) para todas as alterações em dados financeiros, movimentações de estoque, cancelamentos de OS e liberações emergenciais por pelo menos 5 anos. |

---

## 5. MATRIZ DE RASTREABILIDADE (REQUISITOS x USUÁRIOS)

| Requisito / Módulo | PCM / Analista | Gestor / Coordenador | Mecânico / Oficina | Líder / Plantonista | Almoxarife | Financeiro / Compras |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **RF-001 / RF-003** *(OS e Planos)* | **X** | **X** | **X** | **X** | | |
| **RF-002** *(Apontamento Mobile)* | | | **X** | **X** | | |
| **RN-003 / RF-001** *(Almoxarifado e Pendência Fiscal)* | **X** | | | **X** | **X** | **X** |
| **RF-006 / RF-007** *(Pneus e Checklist)* | **X** | | **X** | **X** | | |
| **RF-008 a RF-010** *(Solver / PMP / Estoque)* | **X** | **X** | | | **X** | **X** |
| **RF-011 / RF-012** *(Assistente IA / RAG / Data Agent)* | **X** | **X** | **X** | **X** | | |
| **RF-013 a RF-016** *(Self-Service BI e Relatórios)* | **X** | **X** | | | **X** | **X**
| **RF-013 a RF-016** *(Self-Service BI e Relatórios)* | **X** | **X** | | | **X** | **X**
| **RF-013 a RF-016** *(Self-Service BI e Relatórios)* | **X** | **X** | | | **X** | **X**