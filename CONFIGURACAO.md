# Guia de configuração — LP Ebook Scanner da Saúde

Você vai fazer **3 coisas**, em ordem:

1. Criar a planilha do Google Sheets que vai receber os leads (com um pequeno script)
2. Colar a URL do script dentro do arquivo `index.html`
3. Subir o projeto na sua Vercel como **um novo projeto separado** (sem mexer no Scanner da Saúde que já está lá)

Tempo total: cerca de 15 minutos.

---

## Parte 1 — Google Sheets + Apps Script (recebe os leads)

### 1.1 Criar a planilha

1. Acesse https://sheets.new (cria uma planilha nova)
2. Renomeie para **"Leads Ebook Scanner da Saúde"**
3. Na linha 1, cole estes títulos (cada um em uma coluna):

   `Timestamp | Nome | Email | WhatsApp | Origem`

### 1.2 Criar o Apps Script

1. Na mesma planilha, clique em **Extensões → Apps Script**
2. Apague tudo que estiver lá e cole o código abaixo:

```javascript
const SHEET_NAME = 'Página1'; // Ou o nome da aba da sua planilha

function doPost(e) {
  try {
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(SHEET_NAME);
    const params = e.parameter;

    sheet.appendRow([
      new Date(),
      params.nome || '',
      params.email || '',
      params.whatsapp || '',
      params.origem || ''
    ]);

    return ContentService
      .createTextOutput(JSON.stringify({ ok: true }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ ok: false, error: String(err) }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet() {
  return ContentService.createTextOutput('Endpoint ativo.');
}
```

3. Clique em **Salvar** (ícone do disquete)
4. Clique em **Implantar → Nova implantação**
5. No ícone de engrenagem, escolha **Aplicativo da Web**
6. Configure:
   - **Descrição:** Leads Ebook
   - **Executar como:** Eu (seu email)
   - **Quem tem acesso:** **Qualquer pessoa** (importante!)
7. Clique em **Implantar**
8. O Google vai pedir para autorizar — autorize com a sua conta Google
9. Copie a **URL do aplicativo da Web** que aparece (formato: `https://script.google.com/macros/s/XXXXXXXX/exec`)

> ⚠️ Se depois você alterar o código, precisa criar uma **NOVA implantação** (não editar a existente), para não quebrar a URL.

### 1.3 Teste rápido

Abra a URL no navegador — deve aparecer "Endpoint ativo." Se aparecer, está tudo OK.

---

## Parte 2 — Colar a URL no site

1. Abra o arquivo `index.html` em um editor de texto (ou peça pra mim)
2. Procure por `COLE_AQUI_A_URL_DO_APPS_SCRIPT`
3. Substitua por sua URL real, por exemplo:

```js
const GOOGLE_SHEETS_ENDPOINT = "https://script.google.com/macros/s/AKfycbxxxxxxx/exec";
```

4. Salve o arquivo.

---

## Parte 3 — Adicionar seus arquivos (PDF + Logo)

Na pasta do projeto, coloque:

- `ebook.pdf` → o seu ebook (já em PDF)
- `logo.png` → a logo do Scanner da Saúde (fundo transparente é o ideal; altura 42px é o tamanho renderizado)

> Os nomes precisam ser **exatamente** `ebook.pdf` e `logo.png`.
> Se sua logo estiver em outro formato (SVG, JPG), me avisa que eu ajusto o HTML.

---

## Parte 4 — Publicar na sua Vercel (SEM mexer no seu projeto atual)

Na Vercel, cada projeto é totalmente isolado. Você vai criar **um novo projeto** — o Scanner da Saúde original continua intacto.

### Opção A — Deploy arrastando a pasta (mais rápido, 2 minutos)

1. Compacte a pasta `lp-ebook` em um `.zip` (ou deixe descompactada)
2. Acesse https://vercel.com/new
3. Clique em **"Other"** ou arraste a pasta direto
4. Dê um nome novo ao projeto — sugestão: **`ebook-scannersaude`**
5. Clique em **Deploy**
6. Pronto! Você recebe uma URL tipo `ebook-scannersaude.vercel.app`

> ✅ Esse projeto é **independente** do seu Scanner da Saúde — não compartilha nada, não sobrescreve nada.

### Opção B — Conectar ao seu domínio scannersaude.com.br

Se você quiser usar um **subdomínio** do seu domínio principal (ex: `ebook.scannersaude.com.br`), é o caminho mais profissional:

1. No **novo projeto** na Vercel (criado na Opção A)
2. Vá em **Settings → Domains**
3. Adicione `ebook.scannersaude.com.br`
4. A Vercel vai pedir pra você criar um registro **CNAME** no seu provedor de DNS (onde você comprou o domínio):
   - **Nome:** `ebook`
   - **Valor:** `cname.vercel-dns.com`
5. Após alguns minutos o subdomínio fica ativo e aponta pra LP.

> ⚠️ **Nunca** adicione o domínio raiz `scannersaude.com.br` no novo projeto — isso tiraria do ar o seu site principal. Apenas o subdomínio `ebook.scannersaude.com.br`.

### Opção C — Fazer pelo CLI (para devs)

```bash
npm i -g vercel
cd lp-ebook
vercel --prod
```

---

## Parte 5 — Testar de ponta a ponta

1. Abra a URL final da Vercel
2. Preencha o formulário com dados de teste
3. Veja se o ebook aparece e baixa corretamente
4. Volte ao Google Sheets e confirme que o lead apareceu na planilha

Pronto! ✨

---

## Dúvidas comuns

**"A URL do Apps Script quebra?"**
Sempre que alterar o código, faça **Nova implantação** (não edite a existente).

**"Os leads não estão chegando na planilha."**
- Confirme que a permissão do Apps Script está como "Qualquer pessoa"
- Confirme que a URL colada no HTML termina em `/exec`
- Abra o console do navegador (F12) na LP e veja se aparece algum erro

**"E se quiser receber por email também?"**
No Apps Script, depois do `sheet.appendRow`, adicione:
```js
MailApp.sendEmail('clinicanutrisecrets@gmail.com',
  'Novo lead — Ebook',
  `Nome: ${params.nome}\nEmail: ${params.email}\nWhatsApp: ${params.whatsapp}`);
```
Faça uma nova implantação.
