# PartsSearcher
Procurador de peças para impressoras offset e flexograficas

1. Escopo do Projeto
1.1. Objetivo Principal
O aplicativo "Análise de Peças com IA" é uma ferramenta web front-end projetada para otimizar e automatizar o processo de identificação, análise e busca por peças de reposição para maquinário gráfico (offset e flexográfico). A aplicação permite que um usuário, mesmo sem conhecimento técnico profundo, submeta informações e fotos de uma peça para uma Inteligência Artificial (IA), recebendo uma análise detalhada e contextual.

1.2. Funcionalidades Chave
Análise Multimodal: Aceita entradas de texto (descrição, marcações) e múltiplas imagens simultaneamente.

Integração Multi-IA: Permite ao usuário escolher entre diferentes provedores de IA (Google Gemini, OpenAI) e modelos específicos.

Interface Dinâmica: O usuário pode adicionar campos de informação personalizados para detalhar as características da peça.

Prompt Customizável: Oferece um painel de configurações avançadas onde o usuário pode ajustar a "persona" do assistente de IA e as perguntas específicas a serem feitas, permitindo um ajuste fino para diferentes necessidades de análise.

Conversa Contínua: Após a análise inicial, a interface se transforma em um chat, permitindo que o usuário faça perguntas de acompanhamento, com a IA mantendo o contexto da conversa.

Persistência de Dados: Salva as chaves de API e as customizações de prompt no navegador do usuário (localStorage) para maior conveniência em sessões futuras.

Modo de Depuração: Inclui um modo de pré-visualização que exibe o prompt final e os nomes dos arquivos de imagem antes de serem enviados, facilitando a depuração sem consumir créditos da API.

2. Estrutura e Arquitetura da Aplicação
A aplicação é construída como um Single Page Application (SPA) contido em um único arquivo index.html. Ela não possui back-end; toda a lógica, processamento e comunicação com as APIs ocorrem diretamente no navegador do cliente (client-side).

2.1. Arquitetura Front-End
Estrutura (HTML5): Utiliza HTML semântico para estruturar os principais componentes da página: cabeçalho, formulário principal e a seção de resultados/conversa.

Estilização (Tailwind CSS): A interface é estilizada utilizando o framework de CSS utilitário Tailwind CSS, carregado via CDN. Isso permite um desenvolvimento rápido e a criação de um design moderno e responsivo. Estilos customizados são adicionados em uma tag <style> para componentes mais complexos como a grade de dados e o modal.

Lógica (JavaScript Vanilla): Toda a interatividade é controlada por JavaScript puro (sem frameworks como React ou Vue). O código é organizado em seções:

Inicialização de Elementos DOM: Mapeamento das variáveis para os elementos da página.

Estruturas de Dados: Objetos que contêm as configurações dos provedores de IA, modelos e opções de campos.

Funções de UI: Funções responsáveis por manipular o DOM (adicionar campos, exibir o modal, mostrar/ocultar o loader, etc.).

Funções de API: Funções async dedicadas para fazer as chamadas fetch para as APIs do Gemini e da OpenAI.

Gerenciadores de Eventos: Blocos addEventListener que orquestram a lógica da aplicação em resposta às ações do usuário (submissão de formulário, troca de modelo de IA, etc.).

2.2. Componentes Arquiteturais Chave
a. Web Worker para Processamento de Imagens
Problema: A conversão de múltiplos arquivos de imagem para o formato Base64 é uma tarefa computacionalmente intensiva que pode congelar a interface do usuário (o "main thread" do navegador).

Solução: Um Web Worker é implementado para delegar essa tarefa a um thread em segundo plano.

O script principal envia os arquivos de imagem para o Worker.

O Worker processa os arquivos de forma assíncrona, sem bloquear a interface.

Ao concluir, o Worker retorna os dados das imagens em Base64 para o script principal.

O script principal, ao receber os dados, prossegue com a chamada para a API.

Resultado: A interface permanece 100% responsiva, mesmo durante o upload de imagens grandes.

b. Gerenciamento de Estado e Persistência
Histórico de Conversa: Uma variável de array (conversationHistory) é usada para manter o estado da conversa atual. A cada interação, a pergunta do usuário e a resposta da IA são adicionadas a este array.

Persistência Local (localStorage):

Chaves de API: Para evitar que o usuário insira as chaves a cada uso, uma opção "Salvar chaves" permite armazená-las de forma segura no localStorage do navegador.

Prompts Customizados: As edições feitas na "Especialidade do Assistente" e na "Análise Solicitada" também são salvas no localStorage, garantindo que as preferências do usuário persistam entre as sessões.

c. Modularidade e Flexibilidade
apiProviders Object: A arquitetura para suportar múltiplos provedores de IA é baseada em um objeto de configuração. Adicionar um novo provedor ou novos modelos se resume a atualizar este objeto, tornando o sistema facilmente extensível.

Funções de API Dedicadas: As funções callGeminiAPI e callOpenAIAPI encapsulam a lógica específica de cada serviço (endpoints, estrutura do corpo da requisição, tratamento de resposta), mantendo o código organizado.

Correção de Parâmetros: O código verifica o modelo selecionado para ajustar dinamicamente os parâmetros da requisição (ex: max_tokens vs. max_completion_tokens), garantindo compatibilidade com as diferentes versões da API da OpenAI.

3. Fluxo de Interação do Usuário (UX Flow)
Configuração Inicial: O usuário seleciona o provedor de IA e o modelo desejado. Ele insere sua chave de API, com a opção de salvá-la para usos futuros.

Entrada de Dados: O usuário preenche as informações da peça em uma grade intuitiva, adicionando ou removendo linhas conforme necessário.

Upload de Imagens: O usuário seleciona um ou mais arquivos de imagem, que são exibidos em uma área de pré-visualização.

Análise: Ao clicar em "Analisar com IA":

O Web Worker processa as imagens em segundo plano.

O prompt é montado com base nos dados do formulário e nas configurações personalizadas.

Uma chamada de API é feita com o prompt e as imagens processadas.

A resposta da IA é exibida, e a interface de chat é ativada.

Conversa de Acompanhamento: O usuário pode fazer perguntas adicionais no campo de chat. Cada nova pergunta é enviada com o histórico completo da conversa, permitindo respostas contextuais da IA.

Ajuste Fino (Opcional): A qualquer momento, o usuário pode clicar no ícone de engrenagem para abrir o modal e customizar os prompts de persona e de análise para refinar os resultados futuros.
