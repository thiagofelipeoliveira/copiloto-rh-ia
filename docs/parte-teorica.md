# Parte Teórica — Copiloto de Comunicação Interna com IA Generativa

> Trabalho da disciplina **Fundamentos de IA com foco em IA Generativa** — UniFECAF.

## 1. Contextualização do Desafio


O desafio proposto parte de um cenário comum em empresas de médio e grande porte: o setor de Recursos Humanos concentra uma quantidade elevada de demandas de comunicação escrita — e-mails corporativos, resumos de reunião, mensagens para grupos de WhatsApp institucional e avisos internos — que, embora repetitivas em estrutura, exigem tempo e atenção para manter um padrão de qualidade e identidade organizacional.

Essa sobrecarga operacional reduz o tempo que o RH poderia dedicar a atividades estratégicas e aumenta o risco de inconsistência no tom e na clareza das comunicações, já que diferentes colaboradores escrevem com estilos e níveis de formalidade distintos.

A proposta deste projeto foi idealizar e prototipar um assistente conversacional — um copiloto de IA — que recebe, em linguagem natural via chat, o pedido do colaborador (o tipo de texto, o tom de voz e os pontos principais) e entrega automaticamente um texto corporativo pronto no canal apropriado (e-mail, Slack ou Notion), sem exigir conhecimento técnico para operar.

## 2. Justificativa para o Uso de IA Generativa


A IA Generativa foi escolhida como núcleo da solução porque o problema central não é apenas organizar informação — é produzir linguagem natural de qualidade a partir de um pedido informal e muitas vezes incompleto. Modelos de linguagem de grande porte (LLMs) são, hoje, a tecnologia mais adequada para esse tipo de tarefa, pois conseguem:
- Interpretar um pedido em linguagem natural e identificar informações faltantes antes de gerar o texto final;
- Adaptar automaticamente o registro linguístico (formal, direto, amigável, empático) conforme instrução;
- Aplicar convenções de gênero textual (estrutura de e-mail, de resumo de ata, de aviso institucional) sem que o usuário precise conhecê-las;

Raciocinar sobre qual ferramenta acionar (e-mail, Slack, Notion) a partir do próprio pedido do usuário, funcionando como um agente autônomo, e não apenas como um gerador de texto passivo.

Alternativas sem IA Generativa — como modelos de texto fixos (templates) preenchidos automaticamente — não resolveriam o problema da mesma forma, pois exigiriam que o RH mantivesse dezenas de modelos prontos e ainda assim não se adaptariam a variações de contexto e tom trazidas por cada solicitação, nem seriam capazes de decidir sozinhas para onde enviar cada texto.

## 3. O Modelo de Linguagem (LLM) Utilizado


A solução foi construída sobre o recurso Make AI Agent, disponível nativamente na plataforma de automação Make.com. Trata-se de uma camada de agente que orquestra um modelo de linguagem de grande porte (LLM) do tipo transformer — configurado neste projeto para usar o modelo de maior capacidade disponível ("Recommended: Large") — e permite equipá-lo com ferramentas (tools) que ele pode acionar de forma autônoma durante a execução.

Diferente do primeiro protótipo do projeto (que usava uma chamada simples ao modelo de linguagem via módulo "Ask Anything"), esta versão final utiliza um agente com raciocínio orientado a ferramentas (tool use): a cada execução, o modelo decide, com base no pedido do colaborador, quais das ferramentas disponíveis — consulta à base de conhecimento, envio por e-mail, publicação no Slack e/ou criação de página no Notion — deve acionar, sem que isso precise ser programado como um fluxo fixo de decisões (if/else).

Essa arquitetura de agente é um passo além de um simples "prompt único": ela demonstra na prática o conceito de tool-augmented LLM, em que o modelo de linguagem deixa de apenas gerar texto e passa a executar ações no mundo real (enviar e-mails, publicar mensagens, criar documentos) de forma autônoma.

## 4. Como o Prompt Foi Elaborado


O prompt foi construído como um prompt de sistema (Instructions) fixo, definindo a persona e as regras de formatação do agente, combinado com uma base de conhecimento externa (Knowledge) e um conjunto de ferramentas (tools) que o próprio modelo decide quando acionar. A estrutura segue quatro camadas de prompt engineering:

### 4.1 Persona e papel


O modelo é instruído a assumir o papel de "Copiloto de Comunicação do RH", fixando o domínio de atuação, o tom institucional esperado e a lista de tipos de texto que deve saber produzir.

### 4.2 Coleta de informações faltantes


O prompt instrui o agente a identificar três informações essenciais (tipo de texto, tom de voz e pontos principais) e a perguntar educadamente por qualquer uma que estiver faltando, em vez de gerar um texto genérico ou inventar informação — comportamento observado e validado durante os testes reais do protótipo.

### 4.3 Instruções condicionais de formatação


O prompt inclui regras de formatação específicas por tipo de texto (emojis com moderação no WhatsApp, sugestão de assunto no e-mail, estrutura de tópicos no resumo de reunião, título e prazo em destaque no aviso institucional).

### 4.4 Base de conhecimento (Knowledge) e ferramentas (Tools)


Além do prompt de sistema, o agente consulta uma base de conhecimento com a identidade da empresa e exemplos de bons textos antes de responder, e tem acesso a quatro ferramentas: busca na base de conhecimento, atualização da base de conhecimento, envio de mensagem no Slack e criação de página no Notion (o envio por e-mail foi mantido como uma etapa fixa do fluxo, executada sempre após o agente).

## 5. Benefícios Percebidos e Desafios Enfrentados


### 5.1 Benefícios percebidos

- Interação totalmente conversacional: o colaborador só precisa mandar uma mensagem de chat no Telegram, sem preencher formulários;
- Autonomia do agente: o próprio modelo decide quais ferramentas acionar (e-mail, Slack, Notion) a partir do pedido, sem fluxo condicional programado manualmente;
- Padronização do tom institucional, reforçada por uma base de conhecimento consultável pelo agente;
- Zero necessidade de programação: toda a automação foi construída de forma visual no Make.com;
- Rastreabilidade: o agente reporta, no próprio e-mail de resposta, o status de cada ferramenta acionada, incluindo eventuais falhas.

### 5.2 Desafios enfrentados

- Gatilho desconectado do fluxo: ao adicionar o módulo do Telegram Bot separadamente, ele ficou como um módulo "órfão" — existia no cenário, mas não estava de fato conectado ao Agente. Foi necessário reestruturar o fluxo para que o Telegram fosse reconhecido como o primeiro módulo (gatilho) do cenário;
- Mapeamento da variável de entrada: foi preciso mapear corretamente o campo de texto da mensagem do Telegram (message.text) como entrada do agente, em meio a múltiplos campos parecidos (mensagens editadas, publicações de canal, etc.) disponíveis na estrutura de dados do Telegram;
- Erro de validação na ferramenta do Notion: em uma das execuções de teste, o agente tentou montar uma estrutura de blocos de conteúdo (títulos, parágrafos) mais complexa do que o esperado pela API do Notion, gerando um erro de validação. A correção foi ajustar a descrição da ferramenta para instruir o agente a sempre usar um único bloco de parágrafo, simplificando a saída e eliminando o erro;
- Configuração de conexões externas: Slack e Notion exigiram criação de conexões OAuth (incluindo a criação de um workspace de teste no Slack) antes de poderem ser usados como ferramentas do agente;

Equilíbrio entre autonomia e previsibilidade: dar ao agente liberdade para decidir quais ferramentas usar aumenta a naturalidade da interação, mas também exige testes mais cuidadosos, já que o comportamento não é 100% determinístico como em um fluxo condicional tradicional.

## 6. Limites Éticos e de Segurança


### 6.1 Privacidade e LGPD


O fluxo trafega dados fornecidos voluntariamente pelo colaborador (mensagem de chat e e-mail de destino) por múltiplos serviços de terceiros (Telegram, Make, provedor de IA, Gmail, Slack, Notion). Sob a ótica da LGPD, isso exige: (i) informar aos colaboradores que as mensagens enviadas ao bot são processadas por provedores de IA externos; (ii) orientar para que não sejam inseridos dados sensíveis ou confidenciais no chat; e (iii) definir política de retenção dos registros de execução do Make.com e do histórico de conversas do Telegram.

### 6.2 Vazamento de dados


Como o conteúdo do chat é processado por uma cadeia de serviços externos, existe risco de informações confidenciais serem expostas a provedores de terceiros. Recomenda-se treinamento dos colaboradores para não inserir dados sigilosos no chat, e avaliação futura de contratos de não retenção de dados com os provedores envolvidos.

### 6.3 Viés, alucinação e autonomia do agente


Diferente de um gerador de texto simples, um agente com acesso a ferramentas (enviar e-mail, publicar no Slack, criar página no Notion) pode executar ações reais de forma autônoma. Isso amplia o risco de um texto impreciso ou tendencioso ser efetivamente publicado sem revisão humana. Recomenda-se, em um cenário de produção, manter uma etapa de aprovação humana antes do envio de comunicados institucionais mais sensíveis, e restringir o escopo de ferramentas que o agente pode acionar sem confirmação.

### 6.4 Segurança de acesso


As credenciais de conexão (Telegram, Google, Slack, Notion) ficam armazenadas na conta do Make.com vinculada ao criador do fluxo. Em um cenário real de produção, recomenda-se o uso de contas de serviço corporativas, controle de acesso por papéis, e rotação periódica de tokens e chaves de API — especialmente o token do bot do Telegram, que concede controle total sobre o bot a quem o possuir.

## 7. Conclusão


O protótipo evoluiu de uma automação simples baseada em formulário para um agente de IA conversacional capaz de interpretar pedidos em linguagem natural, consultar uma base de conhecimento e decidir autonomamente quais ferramentas acionar. Essa evolução ilustra, na prática, a diferença entre usar um LLM como um gerador de texto isolado e usá-lo como um agente com ferramentas (tool use) — um passo relevante na direção de assistentes de IA Generativa verdadeiramente úteis no contexto corporativo. Os principais aprendizados envolveram tanto aspectos técnicos (conexão correta entre gatilhos e módulos, mapeamento de variáveis, validação de esquemas de API de terceiros) quanto éticos (a necessidade de equilibrar autonomia do agente com supervisão humana e segurança de dados).
