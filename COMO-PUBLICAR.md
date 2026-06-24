# Dashboard FP&A — Como publicar no GitHub Pages

## O que é este projeto
Um dashboard que lê suas planilhas Excel **direto no navegador** e mostra duas coisas:

**1. Orçado vs Real (agregado, por mês)** — quando você carrega também o orçamento:
- KPIs: Orçado, Realizado, Variação (R$ e %), Execução Orçamentária
- Gráfico Orçado vs Realizado por mês + tabela mensal
- Seletor de escopo (Tipo do orçamento; padrão: Custos Variáveis)

**2. Análise de Custos Realizados (SAP KSB1)** — detalhe dos lançamentos:
- KPIs: Custo Total, Nº de Lançamentos, Média Mensal, Centros de Custo, Maior Classe
- Custo por Classe de Custo / por Centro de Custo / Evolução mensal / Composição por Objeto
- Tabela detalhada com filtros (mês / centro / classe / objeto / origem)

Você pode carregar **vários arquivos de uma vez** (serviços + materiais + orçamento) — o
sistema reconhece sozinho qual é qual.

> **Privacidade:** os dados do Excel NÃO são enviados para a internet. Tudo é processado
> no navegador de quem abre o site. O link é público, mas os números ficam locais.

## Formato das planilhas
- **Realizado:** extração do KSB1 (cabeçalho na 1ª linha). Reconhece automaticamente
  *Valor variável/MObj.*, *Denom.classe custo*, *Centro custo*, *Denominação objeto*, *Data do documento*.
- **Orçamento:** a planilha "Consolidado CV-CF-SG&A" (cabeçalho na linha 6, com colunas
  mensais jan–dez). Reconhece *Tipo*, *Conta 4\**, *Descrição Conta 4\** e os meses.

> ⚠️ O Orçado vs Real é comparado **no total e por mês** (escopo Custos Variáveis), apenas
> nos meses que têm orçado E realizado. A abertura por conta NÃO é comparada porque os
> códigos de conta do orçamento diferem dos códigos de lançamento do SAP — precisaria de
> um de-para para isso.

---

## Passo a passo para publicar (gratuito)

### 1. Criar conta no GitHub
- Acesse https://github.com e crie uma conta (gratuita).

### 2. Criar um repositório
- Clique no **+** (canto superior direito) → **New repository**.
- Nome: `dashboard-fpea` (ou o que preferir).
- Marque **Public**.
- Clique em **Create repository**.

### 3. Subir o arquivo index.html
- Na página do repositório, clique em **uploading an existing file**.
- Arraste o arquivo **index.html** desta pasta.
- Clique em **Commit changes**.

### 4. Ativar o GitHub Pages
- No repositório, vá em **Settings** (aba no topo).
- No menu lateral, clique em **Pages**.
- Em **Source**, escolha **Deploy from a branch**.
- Em **Branch**, selecione **main** e a pasta **/ (root)**. Clique em **Save**.

### 5. Pegar o link
- Aguarde ~1 minuto e recarregue a página de Pages.
- Vai aparecer: **"Your site is live at https://SEU-USUARIO.github.io/dashboard-fpea/"**
- Esse é o link que você compartilha com qualquer pessoa. 🎉

---

## Como atualizar depois
Sempre que mudar o `index.html`, suba a nova versão pelo mesmo caminho
(**Add file → Upload files**) e o site atualiza sozinho em ~1 minuto.
