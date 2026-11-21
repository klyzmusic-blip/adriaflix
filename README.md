# AdriaFlix 🌊

> Seu streaming favorito, do jeitinho da Ponyo 🌊 — filmes clássicos, séries e documentários em domínio público.

Addon de [Stremio](https://www.stremio.com/) **100% estático**: roda no **GitHub Pages**, é atualizado pelo **GitHub Actions** e não precisa de VPS, servidor nem cartão de crédito. As fontes são legais e abertas: **Internet Archive**, catálogos JSON seus e playlists M3U que você mesmo cadastra.

- **ID do addon:** `org.adriaflix.legal`
- **Você só mexe em um arquivo:** `fontes.json`

```mermaid
flowchart LR
    A["✏️ Você edita<br/>fontes.json"] --> B["⚙️ GitHub Actions<br/>build_addon.py"]
    C["⏰ 1x por dia<br/>(cron)"] --> B
    B --> D["📁 docs/<br/>manifest + catálogos"]
    D --> E["🌐 GitHub Pages"]
    E --> F["📺 Stremio no iPad"]
```

---

## Estrutura do repositório

```
adriaflix/
├── fontes.json                  ← ✏️ o ÚNICO arquivo que você edita
├── README.md
├── scripts/
│   ├── build_addon.py           ← gerador estático (só Python padrão, sem pip)
│   └── gerar_assets.py          ← (opcional) refaz logo e fundo
├── .github/workflows/update.yml ← atualização automática (push + diária)
├── exemplos/                    ← modelos de custom_json, m3u e fontes avançadas
└── docs/                        ← 📦 gerado automaticamente → vira o site
    ├── assets/logo.png
    ├── assets/background.png
    └── (manifest.json, catalog/, meta/, stream/, index.html — criados pelo Actions)
```

---

## Passo a passo

### 1) Crie o repositório

1. Entre em <https://github.com/new>.
2. **Repository name:** por exemplo `adriaflix`.
3. Marque **Public** (o GitHub Pages gratuito exige repositório público).
4. Clique em **Create repository**.

### 2) Suba os arquivos

Na página do repositório vazio, clique em **uploading an existing file** e arraste **todo o conteúdo** desta pasta (`fontes.json`, `README.md`, `scripts/`, `docs/`, `exemplos/`, `.gitignore`).

> ⚠️ **Pasta oculta `.github`:** alguns navegadores não enviam pastas que começam com ponto. Depois do upload, confira se existe `.github/workflows/update.yml`. Se não existir, clique em **Add file → Create new file**, digite exatamente `.github/workflows/update.yml` no nome e cole o conteúdo do arquivo.

Clique em **Commit changes**.

### 3) Libere o Actions para gravar no repositório

**Settings → Actions → General → Workflow permissions** → escolha **Read and write permissions** → **Save**.

### 4) Ative o GitHub Pages apontando para `/docs`

1. **Settings → Pages**.
2. Em **Build and deployment → Source**, escolha **Deploy from a branch**.
3. **Branch:** `main` e a pasta **`/docs`** → **Save**.

Seu site ficará em `https://<usuario>.github.io/<repo>/`.

### 5) Rode a primeira geração

O workflow roda sozinho quando o `fontes.json` é enviado. Para conferir (ou forçar):

1. Aba **Actions → Atualizar AdriaFlix 🌊**.
2. **Run workflow** → **Run workflow**.
3. Aguarde o ✅ verde (costuma levar 1–3 minutos) e mais ~1 minuto para o Pages publicar.

Teste no navegador: `https://<usuario>.github.io/<repo>/manifest.json` deve mostrar um JSON.

### 6) Instale no Stremio do iPad 📱

**Jeito mais fácil:**

1. No **Safari do iPad**, abra `https://<usuario>.github.io/<repo>/`.
2. Toque em **⭐ Instalar no Stremio** e confirme.

**Jeito manual:** no Stremio, abra a área de **Addons** e cole o link
`https://<usuario>.github.io/<repo>/manifest.json` no campo de instalação por URL.

Pronto: as fileiras 🎬 📺 📚 aparecem na tela inicial e em **Descobrir**. 🎉

---

## Como editar o `fontes.json` (pelo navegador, sem código)

1. No repositório, clique em `fontes.json` → ícone de **lápis ✏️**.
2. Adicione, altere ou remova entradas.
3. **Commit changes**. O Actions refaz tudo automaticamente em poucos minutos.

Se houver erro de vírgula ou campo faltando, a aba **Actions** mostra uma mensagem clara em português e o addon continua funcionando com a última versão.

### Campos de cada fonte

| Campo | Obrigatório | Descrição |
|---|---|---|
| `id` | ✅ | Identificador único (letras, números, `_` e `-`). |
| `nome` | ✅ | Nome da fileira. |
| `icone` | | Emoji na frente do nome. |
| `tipo` | ✅ | `movie`, `series` ou `tv`. |
| `provedor` | ✅ | `internet_archive`, `custom_json` ou `m3u`. |
| `query` | IA | Busca do Internet Archive. |
| `url` / `arquivo` | JSON/M3U | URL http(s) **ou** caminho de um arquivo dentro do repositório. |
| `limite` | | Máximo de títulos (padrão 30; teto 300). |
| `origem` | | Nome exibido como origem (custom_json/m3u). |

### Provedor `internet_archive`

Usa `archive.org/advancedsearch.php` + `archive.org/metadata/{id}` e monta o link direto `.mp4` (com alternativas leve e `.ogv`/`.webm` quando existem), sem propaganda. Itens de **empréstimo/restritos** são descartados automaticamente.

Opções extras:

- `"exigir_licenca_aberta": true` — só entram itens cujo cadastro no Internet Archive traz licença de domínio público ou Creative Commons.
- `"rotulo_origem": "..."` — texto de origem que **você** afirma para a fonte inteira.
- `"ordenar": "date desc"` — ordenação da busca (padrão: `downloads desc`).

Exemplos de `query`:

```
collection:feature_films AND mediatype:movies
collection:classic_tv AND mediatype:movies
collection:opensource_movies AND subject:documentary
collection:prelinger AND mediatype:movies
collection:animationandcartoons AND mediatype:movies
```

### Provedor `custom_json` (seu catálogo próprio)

Aponte `url` para qualquer JSON seu (ou `arquivo` para um JSON dentro do repositório). Modelo completo em [`exemplos/catalogo_exemplo.json`](exemplos/catalogo_exemplo.json):

```json
{
  "itens": [
    {
      "id": "meu-filme",
      "nome": "Meu Filme",
      "ano": 1925,
      "descricao": "Sinopse.",
      "poster": "https://.../poster.jpg",
      "fanart": "https://.../fanart.jpg",
      "generos": ["Drama"],
      "duracao": "90 min",
      "origem": "Meu acervo",
      "licenca": "https://creativecommons.org/publicdomain/mark/1.0/",
      "streams": [{ "url": "https://.../filme.mp4", "titulo": "Legendado" }]
    }
  ]
}
```

Para `tipo: "series"`, troque `streams` por `episodios`: `[{ "temporada": 1, "episodio": 1, "titulo": "...", "url": "..." }]`.

### Provedor `m3u` (playlists legais)

Aponte `url` ou `arquivo` para uma playlist `#EXTM3U`. São lidos `tvg-logo`, `group-title` (vira gênero) e o nome. Modelo em [`exemplos/playlist_exemplo.m3u`](exemplos/playlist_exemplo.m3u). Veja mais combinações em [`exemplos/fontes_avancadas.json`](exemplos/fontes_avancadas.json).

---

## Atualização automática

O workflow [`update.yml`](.github/workflows/update.yml) roda:

- **a cada push** que altere `fontes.json` (ou o gerador);
- **1 vez por dia** (06:17 UTC = 03:17 em Brasília);
- **manualmente** pelo botão *Run workflow*.

Ele executa `python scripts/build_addon.py` e faz commit automático da pasta `docs/` (só se algo mudou).

---

## O que o addon faz — e o que não faz

Como tudo é arquivo estático (sem servidor), alguns recursos do Stremio **não existem** aqui:

- ❌ **Busca por texto:** exigiria um servidor para responder cada pesquisa. No lugar, cada fileira tem **filtro por gênero** (em *Descobrir*), gerado a partir dos assuntos dos títulos.
- ❌ **Paginação infinita:** cada fileira mostra até o `limite` que você configurou.
- ✅ Filmes, séries com temporadas/episódios, canais (`tv`), pôster, fanart, sinopse com origem e links diretos.

---

## Aviso legal ⚖️

O AdriaFlix **não hospeda nenhum vídeo**: ele só organiza links de fontes que **você** cadastra.

- O Internet Archive abriga muito material em domínio público, mas **não garante** que toda a coleção seja livre. A sinopse mostra a licença quando o item a declara; sem licença declarada, aparece *"coleção aberta — confira a licença"*. Use `exigir_licenca_aberta` se quiser ser mais restrito.
- Você é responsável por só cadastrar fontes que tenha o direito de usar.

---

## Solução de problemas

| Sintoma | O que fazer |
|---|---|
| `manifest.json` dá 404 | Confira **Settings → Pages** (branch `main`, pasta `/docs`) e se o Actions terminou com ✅. |
| Actions falha com *permission denied* | **Settings → Actions → General → Read and write permissions**. |
| Falta a pasta `.github` | Crie o arquivo manualmente (passo 2). |
| Fileira sumiu | A fonte falhou ou não achou vídeos tocáveis; veja o log em **Actions**. Ajuste a `query` ou rode de novo. |
| Atualização diária parou | O GitHub pausa agendamentos de repositórios sem atividade por ~60 dias. Rode **Run workflow** uma vez para reativar. |
| Vídeo não toca | Tente o stream “MP4 Direto (leve)”; arquivos `.ogv`/`.webm` podem não tocar no iPad. |
| Fez mudanças no addon | No Stremio, remova e reinstale o addon para ele reler o manifest. |

## Testando no seu computador (opcional)

```bash
python scripts/build_addon.py --base-url http://localhost:8000
cd docs && python -m http.server 8000
```

Depois use `http://localhost:8000/manifest.json` no Stremio do computador.
