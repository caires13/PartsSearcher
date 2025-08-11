# PartsSearcher
Procurador/analisador de peças para impressoras offset e flexográficas (SPA 100% client-side)

## Estado atual (simplificado)
- Provedor único: OpenAI GPT‑5 (chat/completions)
- Multimodal: texto + imagens (imagens enviadas como `image_url` com `data:` URI)
- Página única: tudo em `index.html` (sem backend)
- Segurança: renderização Markdown com DOMPurify
- Robustez: processamento de imagens via Web Worker com fallback, timeouts de rede, tratamento de erros amigável

## Como usar
1. Abra `index.html` em um navegador moderno (Chrome/Edge/Firefox). Para evitar limitações de `file://`, prefira servir com um servidor local simples.
2. Insira sua chave da OpenAI (campo “Chave da API OpenAI”). Opcional: marque “Salvar chaves no navegador”.
3. Preencha ao menos um campo nas “Informações da Peça”.
4. Adicione 1+ imagens da peça.
5. Clique em “Analisar com IA”. Após a resposta, use o campo de acompanhamento para continuar a conversa.

## Requisitos
- Navegador com suporte a Web Workers, Fetch e ES2020+
- Chave da OpenAI com acesso ao modelo `gpt-5`

## Arquitetura resumida
- UI: Tailwind via CDN + pequeno CSS custom (sem `@apply` em runtime)
- Lógica: JavaScript vanilla em módulos lógicos (seções) dentro do mesmo arquivo
  - Config/Constantes (`CONFIG`)
  - Tratamento de erros (`AppError`, `APIError`, `ValidationError`, `ErrorHandler`)
  - Logs (`Logger`)
  - Validações (`Validator`)
  - UI/DOM helpers (pré-visualização, status, modal, campos dinâmicos)
  - Imagens (Worker + fallback) com limite de arquivos e tamanho
  - Provedor OpenAI (`callOpenAIAPI`) com timeout e normalização de conteúdo
  - Fluxo do formulário e do chat (histórico em `conversationHistory`)

## Principais decisões técnicas (e por quê)
- Apenas GPT‑5: reduz superfície de bugs enquanto estabilizamos. A arquitetura continua preparada para reativar múltiplos provedores.
- Worker com fallback: evita “travamentos” durante leitura de imagens; se o Worker falhar/atrasar, há leitura no main thread.
- AbortController em requisições: evita esperas longas e retorna mensagem “Tempo de resposta esgotado” ao usuário.
- DOMPurify: respostas em Markdown sanitizadas antes de inserir no DOM.

## Configurações e limites
- Tamanho máximo de arquivo: 10 MB
- Tipos aceitos: `jpeg`, `png`, `webp`, `gif`
- Timeout de API: 60 s (configurável em `CONFIG.API_TIMEOUT_MS`)
- Máximo de arquivos: 10

## Depuração
- Marque “Modo de Depuração (Pré-visualizar prompt)” para ver o prompt e os nomes das imagens sem chamar a API.
- Use o botão “Exportar Config” no modal para copiar um snapshot do ambiente (útil ao reportar problemas).
- “Limpar Logs” zera o console e mostra um toast confirmando.

## Solução de problemas
- “Chave inválida”: verifique a chave no painel da OpenAI e se há acesso ao `gpt-5`.
- “Tempo de resposta esgotado”: verifique a rede; tente novamente. Aumente `CONFIG.API_TIMEOUT_MS` se necessário.
- “Erro ao processar imagens”: cheque formato/tamanho; tente com menos arquivos.
- Sem resposta na conversa: abra o console do navegador e copie a primeira entrada “🚨 [ERROR] …” ao abrir uma issue.

## Roadmap curto
- Modularizar em namespaces internos (`App.*`) mantendo arquivo único
- Reintroduzir provedores e modelos adicionais (Gemini, OpenAI o3/mini) atrás de um toggle
- Upload direto (sem base64) quando o provedor suportar URLs blob temporárias

## Estrutura do projeto
```
PartsSearcher/
  ├─ index.html   # SPA única com UI + lógica + workers
  └─ README.md    # este arquivo
```

## Observações de privacidade
- Todas as chaves ficam no seu navegador (opcional via localStorage). Não há backend neste projeto.
- As imagens são convertidas para `data:` URL e enviadas diretamente à API da OpenAI.

## Licença
Este projeto é disponibilizado “como está”. Verifique os termos de uso das APIs de terceiros (OpenAI) antes de usar em produção.
