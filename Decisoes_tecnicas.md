# Documentação de Decisões Técnicas — Projeto Final: Assistente de Investimentos com Automação e IA

## 1. Visão Geral do Projeto

Esta documentação tem como objetivo registrar e justificar as escolhas arquiteturais e de design implementadas no fluxo de automação desenvolvido na ferramenta **n8n**. O projeto consiste em um assistente automatizado capaz de integrar dados de clientes oriundos de uma aplicação web via Webhook, cruzar essas informações com uma tabela de produtos financeiros atualizada e, utilizando inteligência artificial generativa, disparar propostas de investimentos personalizadas por e-mail de forma escalável, segura e controlada.

---

## 2. Decisões Técnicas e Arquiteturais

### 2.1. Abordagem No-Code para Extração de Dados de Investimentos (Substituição de Código Customizado)

- **Decisão:** Optou-se por utilizar o bloco nativo de extração e requisição web (_HTTP Request_ combinado com o nó de extração de arquivos e agregação) para obter a tabela de investimentos a partir de uma fonte externa em formato CSV, em vez de implementar um bloco de código programático (JavaScript ou Python puro) conforme sugerido no material de referência original.
- **Justificativa:** O uso de blocos _no-code_ nativos reduz a superfície de manutenção de código customizado dentro da esteira de integração, promove maior aderência às boas práticas de desenho visual de fluxos de automação e facilita a leitura e auditoria das etapas por profissionais de áreas de negócios ou analistas de suporte, mantendo a flexibilidade sem dependência de scripts complexos de parsing.

### 2.2. Iteração Sequencial Baseada em Lotes (_Loop Over Items_) para Disparos com IA

- **Decisão:** Diferente da proposta didática inicial que processava e disparava blocos inteiros de clientes simultaneamente, o fluxo atual foi estruturado para iterar os envios e as chamadas ao modelo de linguagem de forma unitária e controlada (um cliente por vez), utilizando um mecanismo de laço de repetição (_Split in Batches_) complementado por uma pausa programada (_Wait_).
- **Justificativa:** Em cenários reais de produção escalável (pensando em bases que podem crescer exponencialmente para centenas ou milhares de registros), disparar grandes volumes de requisições simultâneas para APIs de IA e servidores de e-mail pode causar estouros de limite de taxa (_rate limits_), falhas de concorrência ou bloqueios temporários por spam (_throttling_). A abordagem iterativa com espaçamento garante estabilidade operacional, rastreabilidade individual e conformidade com as políticas de envio de e-mails transacionais.

### 2.3. Validação Cruzada de Perfil de Risco e Saldo Mínimo via Motor Lógico

- **Decisão:** A filtragem dos produtos financeiros elegíveis para cada cliente é feita cruzando estritamente o perfil de investidor (Conservador, Moderado, Arrojado) e o saldo monetário disponível convertido para valores numéricos precisos, impedindo que ofertas incompatíveis sejam sugeridas.
- **Justificativa:** Garantir a conformidade regulatória e a adequação do produto ao perfil do cliente (_suitability_) é uma premissa fundamental no mercado financeiro. A automação assegura que o cliente receba apenas recomendações que respeitem o seu capital mínimo de aplicação e tolerância ao risco.

### 2.4. Enriquecimento de Conteúdo Assistido por Inteligência Artificial Estruturada

- **Decisão:** O modelo de linguagem (_LLM_) atua refinando uma mensagem base padronizada, sendo obrigado a retornar estritamente um objeto em formato JSON contendo o assunto, o corpo formatado em HTML limpo e o produto divulgado.
- **Justificativa:** O uso de IA generativa traz um tom humano, persuasivo e estritamente formal às comunicações, sem o risco de alucinações de dados numéricos ou quebra de layout no cliente de e-mail, visto que a estrutura de saída é rigidamente validada e convertida por blocos de normalização subsequentes.

### 2.5. Resiliência Operacional e Notificações de Término

- **Decisão:** Implementação de tratamento de erros individualizados em nós críticos (como o envio de e-mails via Gmail) utilizando a diretriz de continuidade regular (_continueRegularOutput_), além de um relatório automático de conclusão disparado ao final da execução do fluxo.
- **Justificativa:** Garante que falhas pontuais no envio de mensagem para um único cliente não paralisem ou corrompam toda a esteira de processamento, fornecendo visibilidade gerencial imediata sobre o sucesso global da operação.
