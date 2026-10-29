# Quiz da Bio | Viver de Vendas Diretas

Quiz de captura de leads em um único arquivo (`index.html`). Sem frameworks, sem backend, sem IA. Só usa as fontes do Google Fonts.

Fluxo: tela inicial, 10 perguntas de Sim ou Não, captura (nome, WhatsApp, e-mail e consentimento), resultado com nota, dicas, PDF e ofertas.

## 1. Editar o CONFIG

Abra o `index.html` e procure o bloco `CONFIG` no começo do `<script>`:

```js
var CONFIG = {
  WEBHOOK_URL: "",                                       // quem recebe os leads ("" = modo de teste)
  SALES_URL: "https://EXEMPLO.com/apareca-e-venda",      // página do Apareça e Venda
  ANALISE_URL: "https://EXEMPLO.com/analise-completa",   // página da Análise Completa
  PDF_URL: "https://EXEMPLO.com/3-modelos-de-bio.pdf",   // PDF com os 3 modelos
  PRIVACY_URL: "https://EXEMPLO.com/politica-de-privacidade",
  FACEBOOK_PIXEL_ID: "",                                 // só números; "" = sem Pixel
  NOME_MARCA: "Viver de Vendas Diretas"
};
```

Troque todos os `https://EXEMPLO.com/...` pelos links reais antes de publicar.

## 2. Testar localmente

1. Dê dois cliques no `index.html` (abre direto no navegador).
2. Abra o console (F12, aba Console).
3. Faça o quiz. Com `WEBHOOK_URL` vazio você verá `Modo de teste` e o JSON que seria enviado.
4. Teste também: e-mail sem @, WhatsApp com 8 dígitos, caixa de consentimento desmarcada e o botão Voltar.
5. Para testar UTMs, abra com `index.html?utm_source=instagram&utm_campaign=bio`.

## 3. Publicar no WordPress

**Opção A: Elementor**
1. Crie uma página nova e escolha o modelo **Elementor Canvas** (página em branco, sem cabeçalho e rodapé do tema).
2. Arraste o widget **HTML**.
3. Cole o conteúdo inteiro do `index.html`.
4. Publique e teste no celular.

Se o tema alterar fontes ou cores, prefira a opção B.

**Opção B: subdomínio ou pasta (mais limpo)**
1. Crie um subdomínio na hospedagem (ex.: `quiz.seudominio.com.br`) ou uma pasta (ex.: `seudominio.com.br/quiz`).
2. Envie o `index.html` pelo Gerenciador de Arquivos ou FTP para essa pasta.
3. Acesse o endereço e teste.

## 4. Conectar o WEBHOOK_URL

O quiz faz um `POST` em JSON com:

```json
{
  "nome": "Maria", "whatsapp": "5511912345678", "email": "maria@email.com",
  "nota": 6, "faixa": "Bio no meio do caminho",
  "respostas": [true, false, true, true, false, true, true, false, true, true],
  "consentimento": true, "data_hora": "2026-10-29T12:00:00.000Z",
  "pagina": "https://...", "utm_source": "instagram"
}
```

Se o envio falhar, a pessoa vê o resultado mesmo assim e o quiz tenta de novo uma vez em segundo plano.

- **Make**: crie um cenário com o módulo **Webhooks > Custom webhook**, copie a URL e cole em `WEBHOOK_URL`. Faça um envio de teste para o Make reconhecer os campos. Depois conecte em Google Sheets, RD Station, ActiveCampaign etc.
- **Zapier**: gatilho **Webhooks by Zapier > Catch Hook**. Copie a URL, cole em `WEBHOOK_URL` e envie um teste.
- **ManyChat**: o ManyChat não recebe webhook direto de página. Use o Make ou Zapier no meio: recebe o lead e chama a ação do ManyChat (criar ou atualizar contato pelo WhatsApp e adicionar uma tag).
- **Google Apps Script** (grava em planilha, grátis): veja abaixo.

### Exemplo: Google Apps Script gravando em planilha

1. Crie uma Planilha Google. Vá em **Extensões > Apps Script**.
2. Apague o que tiver e cole:

```js
function doPost(e) {
  var aba = SpreadsheetApp.getActiveSpreadsheet().getSheets()[0];
  var d = JSON.parse(e.postData.contents);
  if (aba.getLastRow() === 0) {
    aba.appendRow(['data_hora', 'nome', 'whatsapp', 'email', 'nota', 'faixa', 'respostas',
                   'consentimento', 'pagina', 'utm_source', 'utm_medium', 'utm_campaign', 'utm_content', 'utm_term']);
  }
  aba.appendRow([d.data_hora, d.nome, d.whatsapp, d.email, d.nota, d.faixa, (d.respostas || []).join(','),
                 d.consentimento, d.pagina, d.utm_source || '', d.utm_medium || '', d.utm_campaign || '',
                 d.utm_content || '', d.utm_term || '']);
  return ContentService.createTextOutput(JSON.stringify({ ok: true }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

3. Clique em **Implantar > Nova implantação**, tipo **App da Web**. Executar como: **Eu**. Quem pode acessar: **Qualquer pessoa**.
4. Autorize, copie a URL que termina em `/exec` e cole em `WEBHOOK_URL`.

Observação: para URLs do Google Apps Script o quiz envia o JSON como `text/plain` (exigência do Google para aceitar o envio vindo do navegador). O script acima lê normalmente.

## 5. Conferir o Pixel da Meta

1. Coloque o ID em `FACEBOOK_PIXEL_ID` (ex.: `"123456789012345"`).
2. Instale a extensão **Meta Pixel Helper** no Chrome.
3. Abra o quiz: deve aparecer **PageView**. Clique em Começar: **ViewContent**. Envie a captura: **Lead**.
4. Confira também em **Gerenciador de Eventos > Testar eventos**.
5. Com o ID vazio nada é carregado; os eventos aparecem só no console como `[Quiz Bio] evento: ...`.

O evento **Lead** dispara quando o envio é confirmado (ou no modo de teste). Se a primeira tentativa falhar e a segunda der certo, ele dispara na segunda.
