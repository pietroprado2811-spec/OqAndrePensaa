# Oque o André pensa?
Analisador recreativo de conversas exportadas do WhatsApp. App estático (JS puro, módulos ES, sem build, sem backend).

## Rodar
```
npm start        # servidor estático local → http://localhost:8080
npm test         # node --test (Node 18+)
```
Exporte a conversa no WhatsApp ("sem mídia", .txt). Gere sua chave em https://aistudio.google.com/apikey e cole no app. O modelo padrão (`gemini-2.5-flash`) é editável no app: confirme nomes, limites e política de dados na [documentação oficial](https://ai.google.dev/gemini-api/docs).

## Estrutura
`src/parser.js` (parser) · `src/importFile.js` + `src/zip.js` (importa .zip ou .txt) · `src/media.js` (classifica mídia) · `src/stats.js` (estatísticas) · `src/privacy.js` (anonimização, amostragem, payload) · `src/gemini.js` (único módulo com rede + prompt de sistema) · `src/schema.js` (validação runtime do JSON) · `tests/` (fixture fictícia + testes).

## Decisões
- Sem framework: dependência zero, bundle sem chave. A CSP em `index.html` só permite conexão com `generativelanguage.googleapis.com`.
- Limite de arquivo: 5 MB (`MAX_BYTES` em `app.js`). Amostragem: 400 mensagens (100 início, 100 fim, 200 espalhadas no meio), 300 caracteres por mensagem.
- Regras de estatística: curta ≤ 20 caracteres, longa ≥ 140, nova conversa = pausa ≥ 6 h; intervalo médio ignora pausas ≥ 6 h. Datas ambíguas assumem dia/mês/ano.
- Resposta do Gemini: JSON via `responseMimeType`, validado em runtime; uma nova tentativa se inválido. Texto renderizado só com `textContent`.

## Privacidade e limitações
Nada é salvo em storage/cookies/logs. A chave fica só em memória e é apagada ao reiniciar ou fechar a aba. O app tenta remover identificadores antes do envio (nomes dos participantes, telefones, e-mails, links, CPF, @usuários), mas a anonimização automática pode não ser perfeita: nomes de terceiros, locais e escolas não são detectados automaticamente, por isso a prévia é editável e o envio exige consentimento. Uma chave no navegador pode ser vista por quem usa a máquina; use chave própria e com cota limitada.


## Importação .zip e mídias
Aceita o `.zip` do WhatsApp (com ou sem mídia) ou só o `.txt`. O zip é lido no navegador: só o diretório central e o `.txt` do chat são lidos; fotos, áudios e vídeos nunca são abertos nem enviados ao Gemini, só contados (por nome de arquivo no zip e, por pessoa, pelos marcadores no texto). Sem zip64 (zips >4 GB ou >65 mil arquivos pedem exportar "sem mídia"). O WhatsApp pode exportar menos arquivos do que os marcadores no texto, então os dois números podem diferir.

## Interface (Stitch)
O design vem do Stitch (`ui/DESIGN.md` e `ui/ref-inicio.png`). O script `cdn.tailwindcss.com` do HTML original foi removido: o Tailwind é compilado uma vez para `styles.css` (já incluído). Para alterar o visual: `npm i` e `npm run build:css` (config em `tailwind.config.cjs`). As fotos de exemplo do Stitch foram removidas. A única origem externa é o Google Fonts, para os ícones Material Symbols (a CSP libera só estilo e fonte dessa origem, nenhum script).
