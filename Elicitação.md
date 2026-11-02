**Data:** 03 de Outubro de 2026


---

# ELICITAÇÃO

## Questionário para Levantamento de Requisitos (Dados Brutos)

---

### Necessidades

#### 1. Quais requisitos de negócios este sistema atenderá?

* **Redução do Tempo de Máquina Parada:** Através de um melhor controle e planejamento das manutenções.
* **Redução de Custos:** Através de um controle mais rigoroso por meio de relatórios e painéis (dashboards) dinâmicos.
* **Garantia de Aderência e Execução dos Planos de Manutenção:** Assegurar que as revisões programadas sejam cumpridas.
* **Prevenção de Quebras de Equipamentos e Frotas:** Manutenção preventiva e preditiva eficiente para evitar falhas graves.
* **Melhor Rastreabilidade Financeira:** Maior transparência e controle sobre cada centavo gasto na frota.
* **Otimização Operacional e Prescritiva:** Uso de algoritmos de otimização para apoio à tomada de decisão complexa (alocação, estoque e escalonamento).

#### 2. Quais informações você precisa deste sistema que você não tem agora?

* **Status da Fila de Espera/Atendimento:** Saber exatamente quantos veículos estão parados aguardando peça, aguardando box de oficina ou aguardando liberação da operação.
* **Histórico Confiável de Falhas:** Saber se um componente falhou antes do prazo de garantia ou se está apresentando falhas excessivas para investigar a causa raiz.
* **Controle de Gastos (Custo Real x Orçado em Tempo Real):** Visão exata de custos por Ordem de Serviço (OS), por tipo de serviço e por peças.
* **Controle do Consumo de Combustível:** Acompanhamento preciso para identificar desvios e anomalias.

#### 4. Quais funcionalidades você precisa no sistema?

* **Módulo de Abertura de OS e Impressão:** Interface completa para abertura de Ordens de Serviço (OS) com botões integrados para impressão direta do documento.
* **Personalização Livre de Nomenclaturas:** Total flexibilidade para o usuário nomear como quiser frotas, centros de custo, contas contábeis, peças, filtros, óleos e demais insumos.
* **Exportação para Excel:** Opção de exportar dados, tabelas e relatórios em formato Excel (.xlsx) de forma rápida e eficiente.
* **Assistente Virtual e Chat de Análise de Dados (AI RAG):** Interface em linguagem natural onde o usuário pode fazer perguntas sobre relatórios, documentações técnicas, procedimentos de manutenção, histórico de OSs e indicadores operacionais/financeiros sem precisar montar consultas manuais.
* **Dashboards e Relatórios Interativos (Self-Service BI):** Painéis e relatórios totalmente dinâmicos, permitindo que o usuário altere o tipo de gráfico (barras, linhas, pizza, dispersão), altere dimensões/agrupamentos (por centro de custo, família de equipamento, período), filtre dados em tempo real e salve suas visões personalizadas.
* **Módulo Otimizador e Solver Operacional:** Algoritmo de pesquisa operacional/otimização (estilo Solver) aplicado em três frentes principais:
* *Otimização de Compras e Estoque:* Cálculo do lote econômico e ponto de pedido considerando restrições de orçamento, espaço em prateleira e lead time de fornecedores.
* *Dimensionamento do Plano de Manutenção:* Maximização da disponibilidade da frota respeitando a capacidade finita de boxes e mecânicos disponíveis.
* *Alocação e Otimização de Troca de Pneus:* Prescrição do momento e rodízio ideal entre eixos para maximizar a vida útil do conjunto rodante.


* **Módulo de Apontamento Rápido em Oficina:** Rota em tablet/mobile para o mecânico abrir e fechar tarefas da OS sem burocracia.
* **Motor de Planos de Manutenção Flexível:** Disparo de preventivas por Odômetro, Horímetro, Calendário ou Consumo de Combustível (o que vencer primeiro), com alertas/mensagens automáticas para os usuários diretamente na tela.
* **Gestão de Pneus Integrada:** Controle de vida útil, sulcagem, recapagens e rodízio por posição de eixo.
* **Sistema de Checklist:** Para verificação prévia dos veículos antes de operarem.
* **Gestão de OS de Terceiros (Serviços Externos):** Acompanhamento de orçamento, aprovação e status de oficina credenciada.

#### 5. Os dados ou funcionalidades são compartilhados por outras áreas de negócios? Se sim, quais?

*(Definição de compartilhamento com Operação, Almoxarifado, Compras e Financeiro conforme perfil dos usuários).*

#### 6. Quanta informação histórica é necessária?

*(A definir com base nas necessidades de auditoria e análise de frota).*

#### 7. Quais são as integrações necessárias com outros sistemas ou aplicações externas?

* **ERP Corporativo (Financeiro/Compras/Fiscal):** Envio de requisições de compras e espelhamento de Notas Fiscais/Contas a Pagar.
* **Telemetria / Módulo de Rastreamento:** Importação diária e automática de posição, odômetro e horímetro.
* **Sistemas de Abastecimento Terceirizados (Tickets/Bombas Próprias):** Leitura automática de litragem, valor e km/h no ato do abastecimento.
* **Catálogos Eletrônicos / APIs de Montadoras (se aplicável):** Para sincronização de códigos de peças e manuais para alimentação do assistente de consulta (RAG).

---

### Usuários do Sistema

#### 1. Quem usará o sistema?

Desde a gestão executiva até a operação de campo (mecânicos no chão de oficina e motoristas na cabine).

#### 2. Quais são as áreas e funções das pessoas que usarão o sistema?

* **PCM (Planejamento e Controle de Manutenção):** Analistas e programadores de manutenção (usuários intensivos / Power Users).
* **Gestão de Frota e Manutenção:** Gerentes, coordenadores e supervisores.
* **Oficina Interna:** Chefes/líderes de oficina, mecânicos, eletricistas, funileiros e lubrificadores.
* **Almoxarifado:** Almoxarifes e compradores de peças.
* **Operação:** Motoristas, operadores de máquinas e fiscais de frota.
* **Financeiro/Controladoria:** Analistas financeiros e auditores.

#### 3. Quais são seus níveis de especialização?

* **Alta Especialização em Sistemas e Processos (Analistas de PCM e Controladoria):** Conseguem manusear telas mais detalhadas e complexas, envolvendo relatórios, planejamentos, parâmetros do Solver, filtros avançados, exportações e montagem de visões customizadas nos dashboards.
* **Média Especialização (Gestores de Frota, Almoxarifes e Chefes de Oficina):** Precisam de dashboards executivos interativos e prontos, além de consultas diretas via assistente virtual (chat de relatórios/indicadores) para tomada de decisão ágil.
* **Baixa / Básica Especialização Tecnológica (Mecânicos de chão de fábrica, Lubrificadores, Motoristas e Operadores):** Exige interface extremamente intuitiva (mobile/tablet), com botões grandes, ícones explicativos e o mínimo de digitação possível.

---

### Problemas Atuais

#### 1. Quais são os problemas atuais que você enfrenta sem o sistema?

* **Controle Financeiro e Relatórios Financeiros:** Dificuldade em visualizar o Orçado x Realizado; rateio de valores para frotas e equipamentos; acompanhamento de gastos diários, semanais, mensais e anuais por frota e por serviço; custo médio por tipo de manutenção (preventiva, corretiva, preditiva); custos detalhados por serviço (lubrificação, borracharia, mecânica); custos com pneus; custos com serviços de terceiros; custo por quilômetro (km); gestão de contas a pagar/receber e fluxo de caixa diário/mensal; além de controle por centro de custo, plano de contas e conta contábil.
* **Relatórios e Dashboards Rígidos:** Os sistemas legados entregam relatórios engessados em formato PDF ou tabelas fixas, sem flexibilidade para alterar gráficos, comparar dimensões ou reorganizar informações visualmente.
* **Planejamento de Manutenções sem Otimização:** Dificuldade em estruturar o sequenciamento ideal das manutenções quando há restrição de mão de obra ou verba, gerando gargalos na oficina e concorrência por recursos.
* **Dificuldade na Busca de Informações Técnicas e Históricas:** Perda de tempo ao navegar por múltiplos menus e pastas para encontrar um manual, uma regra de garantia de peça ou um indicador histórico consolidado.
* **Troca de Óleo, Filtros e Abastecimentos:** Ausência de controle claro e visual da frequência de trocas e custos associados; falta de visibilidade dos componentes de cada frota e do tempo de uso de filtros e sistemas para prever o tempo restante até a próxima troca considerando os km rodados.
* **Consultar Catálogos e Manuais:** Falta de uma base de consulta rápida e centralizada para manuais de cada equipamento.
* **Relatórios e Indicadores Precisos:** Dificuldade para extrair métricas confiáveis como Disponibilidade Física, MTBF, MTTR, Consumo Médio e afins.
* **Controle de Estoque:** Necessidade de um controle rigoroso de entrada e saída de peças, quantidades disponíveis, solicitações de reposição e devoluções para evitar perdas ou extravios.
* **Apontamento de Mecânicos:** Falta de acompanhamento e controle das horas e tarefas apontadas pelos mecânicos.

#### 2. Que problemas este sistema deve resolver?

* **Eliminação da Redundância de Dados e Democratização do Acesso:** Centralização de todas as informações em uma única base, permitindo consultas dinâmicas por chat ou gráficos configuráveis.
* **Garantia de Rastreabilidade:** Saber exatamente onde as peças foram aplicadas, atrelando o componente ao número da OS ou à placa do veículo.
* **Redução do Tempo de Parada e Atendimento:** Agilizar o ciclo de vida da manutenção, usando algoritmos de otimização para melhor uso da infraestrutura da oficina.

#### 3. Você precisa fazer coisas manualmente que gostaria de automatizar?

* Consultas a relatórios longos automatizadas por assistente de IA em linguagem natural.
* Cálculo do plano de programação da oficina otimizado via algoritmo Solver considerando recursos limitados.
* Geração automática de OS de preventiva e manutenção quando o horímetro/odômetro atingir o limite parametrizado por usuário autorizado.
* Atualização automática de km/h via telemetria.
* Baixa automática de estoque pelo sistema na execução da OS.
* Envio automático de relatórios diários de disponibilidade e indicadores para a operação e gestores.
* Exportação ágil de relatórios para Excel e facilidade para emissão/impressão de OSs.

#### 4. Você tem problemas de desempenho que precisam ser alterados?

* Lentidão ao consultar históricos em sistemas legados ou planilhas (atualmente leva minutos para carregar dados de 1 ano; o novo sistema precisa responder em segundos, inclusive nos gráficos interativos e no assistente virtual).
* Demora excessiva no processo de importação de dados.
* Demora para exportar grandes volumes de dados para o Excel.

#### 5. Você tem limitações funcionais que gostaria de alterar?

* Impossibilidade de customizar relatórios/gráficos em tempo real sem dependência da equipe de TI.
* Necessidade de aplicativos que funcionem offline e sincronizem os dados automaticamente quando houver conexão.
* Dificuldade no ajuste e flexibilização dos planos de manutenção.

#### 7. Quais relatórios você usa atualmente? Quais dados no relatório são importantes?

* **Boletim Diário de Disponibilidade.**
* **Curva ABC de Custos por Frota.**
* **Aderência ao Plano de Preventiva.**
* **Indicadores MTBF e MTTR.**
* **Relatórios Financeiros:** Visão Orçado x Realizado, rateios por frota/equipamento, custos diários/semanais/mensais/anuais, custos por tipo de manutenção, por tipo de serviço, com pneus e terceiros, custo/km, contas a pagar/receber, fluxo de caixa e centro de custo/conta contábil.
* **Relatório de Apontamento de Mecânicos.**
* *Premissa obrigatória de dados:* Todos os relatórios e componentes visuais de dashboards devem permitir filtros e agrupamentos por identificação da frota, modelo e categoria de serviço (ex: Tratores, Colhedoras, Caminhões, Ônibus).

#### 8. Existem gargalos específicos para obter informações?

* Dificuldade e demora na coleta de dados na ponta para abertura da OS ou para preenchimento de fichas com os dados necessários.

---

### Critérios de Sucesso

#### 1. O que é mais importante para o sucesso da aplicação?

* **Facilidade de Uso no Chão de Fábrica e na Gestão (Adoção):** Interface operacional ágil e simples no mobile; dashboards dinâmicos e assistente virtual intuitivo para os gestores.
* **Confiabilidade nos Dados de Entrada:** Garantir que odômetros e horímetros registrados estejam corretos e sem adulterações.
* **Efetividade nos Modelos de Otimização (Solver):** Apresentar recomendações de compras, estoques e agendamento de oficina que efetivamente reduzam o tempo ocioso e o custo operacional.

#### 2. O que precisamos realizar para tornar este projeto bem-sucedido?

* Mapear e simplificar os processos operacionais antes de iniciar o desenvolvimento.
* Garantir a integração fluida com o ERP corporativo e sistemas de telemetria.
* Projetar um sistema de fácil integração com outras tecnologias e softwares futuros da empresa.

#### 5. Está faltando algum elemento crítico, como orçamento, alocação de recursos ou suporte?

* A empresa está aberta à aquisição de hardwares adequados (tablets industriais/robustos para a oficina) e melhorias na infraestrutura (instalação de Wi-Fi de alta qualidade no pátio e na oficina).

---

### Premissas, Pendências e Riscos

#### 1. Liste as principais premissas para realização desse projeto

* As frotas devem possuir equipamentos de telemetria ativos para transmissão de horímetro, odômetro e localização.
* A empresa fornecerá a infraestrutura necessária (rede Wi-Fi na oficina e dispositivos móveis para a operação do aplicativo).
* Os demais sistemas e ERPs internos devem possuir APIs abertas e devidamente documentadas para integração.
* Os manuais em PDF e documentos técnicos das máquinas serão disponibilizados em repositório acessível para indexação no assistente virtual (RAG).