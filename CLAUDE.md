# CLAUDE.md — Contexto e regras do projeto **fvm**

> Portfólio audiovisual de **Flávio Costalonga** (filmmaker, fotógrafo e artista
> visual). Este arquivo guarda o contexto e as regras acumuladas para que
> qualquer sessão futura continue o trabalho sem reler todo o histórico.

---

## 0. Regras rápidas (leia primeiro)

- **Idioma:** o usuário fala **português (Brasil)**. Responda em português.
- **Branch de trabalho:** `claude/intelligent-euler-1uhqa9`. **Nunca** faça push
  para outra branch sem autorização. Repositório: `jwilliangp/fvm`.
- **O site é UM único arquivo:** `index.html` (HTML + CSS inline + JS vanilla).
  **Sem dependências externas / sem requisições de rede** (fontes do sistema,
  imagens locais). Mantenha assim.
- **Commits em português**, descritivos. Sempre **commit + push** ao concluir
  uma alteração (com retry/backoff se a rede falhar). **Não** abra Pull Request
  sem o usuário pedir.
- **Antes de commitar:** validar o JS (`node --check`), conferir o balanço de
  chaves do CSS e checar que os caminhos de imagem referenciados existem.

---

## 1. Estética e sistema de design (regras imutáveis)

Referências do usuário: **hickduarte.com**, **dimitrow.com.br**, **nixonfreire.com**.
Princípio central: **a mídia domina; o texto é discreto.**

- **Tipografia minimalista:** UMA família (grotesca neutra — `--font`:
  Helvetica Neue/Inter + fallback de sistema), **um único peso** (`--fw: 400`)
  e **um único tracking** (`--tracking: 0.04em`), definidos uma vez no `body` e
  herdados. **A hierarquia vem só do tamanho** (`--fs-*`). Nada de fontes
  display pesadas ou monoespaçadas.
- **Dark mode, alto contraste:** fundo `--noir: #070707`, texto `--white`,
  secundário `--grey: #8a8a8a`, linhas `--line: #333`.
- **Accent único:** `--red: #FE0000`, usado com parcimônia (estado ativo de
  filtro, hover, seleção, e o brilho do PLAY REEL).
- **Border-radius binário:** `0` para mídia/blocos, `999px` (pill) para botões.
- **Botões sem borda** (decisão do usuário). Foco visível por `outline` global.
- **Grão de filme:** existe um grão atmosférico sutil no hero (original) e um
  grão **só no brilho do botão PLAY REEL** (efeito foto antiga). O grão global
  foi **removido** a pedido do usuário — não reintroduza sem pedir.

### PLAY REEL (hero)
Botão de texto; no hover surge um **foco de luz vermelha** em formato de losango
difuso (contra-luz). Implementação: recorte (`clip-path` losango) no `::before`
de `.btn-reel__glow` e o **desfoque (`blur`) no elemento-pai** (assim o blur
"penteia" as bordas — sem linha dura). Por cima, um **grão** nítido em
`soft-light` confinado ao miolo (`.btn-reel::after`, mascarado). Já está afinado;
mexer só se o usuário pedir.

---

## 2. Estrutura do site

Seções em `index.html`: header/nav fixo · hero (PLAY REEL) · portfólio (filtros +
grid de cards) · seção INFO/SOBRE · footer · **3 overlays/modais**.

### Abas — nome exibido ≠ chave interna (NÃO confundir)
A filtragem usa `data-filter` (nos links/botões) e `data-category` (nos cards).
O **texto exibido** foi renomeado, mas as **chaves internas continuam as antigas**:

| Aba (exibida) | chave interna (`data-filter`/`data-category`) | Modal ao clicar no card |
|---|---|---|
| **AUDIOVISUAL** | `filmes` | Modal de **vídeo** (`#film-modal`) |
| **FOTOGRAFIA** | `fotos` | **Galeria** horizontal (`#gallery-modal`) |
| **CONCEITOS** | `conceitos` | Modal de **projeto** (`#project-modal`) — placeholder |

➡️ Ao mexer em texto de aba, **mude só o conteúdo visível**, nunca os
`data-filter`/`data-category`.

### Roteamento do clique no card (JS)
Cada card tem um dataset que define o destino:
`data-film` → `openFilm()` · `data-gallery` → `openGallery()` · `data-project` →
`openProject()`. O helper `makeOverlay(modalEl, onClose?)` cuida de
abrir/fechar/ESC/foco-preso para os três.

---

## 3. FOTOGRAFIA — galerias (estado atual ✅)

Cada card de FOTOGRAFIA abre uma **galeria em tela cheia, rolagem horizontal**.
Cada foto ocupa a altura toda; título pequeno no canto inferior direito com
**fundo preto 33%** (`background: rgba(0,0,0,0.33)`). Rolagem com **inércia
(lerp em `requestAnimationFrame`)** — não usar `scroll-snap` (foi removido por
travar). A roda do mouse é convertida em rolagem horizontal.

### Dados: objeto `galleries` (no `<script>`)
```js
'chave-da-galeria': {
  name: 'Nome exibido',
  seg: 'Foto',
  desc: '...',
  shuffle: true,            // opcional: embaralha a ordem a cada abertura (Fisher–Yates)
  photos: [
    { title: 'Título', src: 'images/fotografia/<chave>/NN-slug.jpg', w: <largura>, h: <altura> }
  ]
}
```
- `openGallery()` aplica `shuffle` se presente; a função genérica `shuffle()`
  já existe. (Houve um campo `cover` para fixar a 1ª foto do modal — **foi
  removido**; "capa" = capa do **card**, ver abaixo.)
- **Placeholders:** galerias começam com `skin`/`ratio` (gradientes). Ao receber
  fotos reais, **substitua** os placeholders (não misture).

### Capa do card
Para usar uma foto real como capa do quadradinho no grid, troque o `<img>` do
card: remova a classe `skin-N` e aponte `src` para a foto, com `width/height`
reais. Ex.: `<img class="card__media" src="images/fotografia/<chave>/NN.jpg" ...>`.

### Estado das 3 galerias de FOTOGRAFIA
| Galeria | chave | nº fotos | shuffle | capa do card |
|---|---|---|---|---|
| A Câmera Escondida | `camera-escondida` | **20** | sim | `07-chuva-roxa.jpg` |
| Velocidade, Brechas e Disparo | `velocidade-brechas` | **13** | sim | `07-m3.jpg` |
| E se tudo for cinema? | `se-tudo-cinema` | **15** | **não** | gradiente (skin-5) — sem capa real ainda |

---

## 4. AUDIOVISUAL — modal de vídeo (estrutura pronta, mídia pendente)

Os 3 cards (`data-film`: `penumbra`, `mare-alta`, `cidade-cinza`) abrem
`#film-modal`: **vídeo em tela cheia + botão de play**; rolando para baixo
aparece o **briefing** (título, texto, ficha técnica). Dados no objeto `films`.
Para colocar vídeo real, preencher `src` (e `poster` opcional) no objeto:
```js
'penumbra': { name:'Penumbra', seg:'Filme', skin:'skin-1', src:'', poster:'', brief:'...' }
```
Enquanto `src` está vazio, o palco mostra o gradiente da `skin`. O `PLAY REEL`
do hero ainda usa o modal de **projeto** (poderia migrar para o player de vídeo).

---

## 5. CONCEITOS — **a fazer por último** (pedido do usuário)

Ainda usa o **modal de projeto** (`#project-modal`) com 3 cards placeholder
(`data-project="6/7/8"` → "Arquitetura da Luz", "Espaços Vazios", "Concreto e
Vidro", gradientes `skin-7/8/9`). Estrutura/estética a definir com o usuário.

---

## 6. ⚠️ Como ADICIONAR IMAGENS enviadas pelo usuário (workflow essencial)

**Imagens coladas no chat NÃO viram arquivo em disco** — elas ficam embutidas em
**base64 no transcript da sessão**. Arquivos anexados (clipe), sim, vão para
`/root/.claude/uploads/...`.

Passo a passo:
1. **Localizar o transcript da sessão atual:** o `.jsonl` mais recente em
   `/root/.claude/projects/-home-user-fvm/`.
2. **Extrair** os blocos `{"type":"image","source":{"type":"base64",...}}` com
   Python: decodificar o base64, **deduplicar por hash** (sha256), e pegar o
   **lote do maior `msg` index** (a mensagem mais recente). Mantenha um conjunto
   de hashes já salvos para não reprocessar lotes antigos.
3. **Atenção aos formatos:** além de `image/jpeg`, podem vir **`image/webp`** e
   anexos **`.heic`**. Converta webp/heic → **JPG (q95)** com Pillow
   (`pip install pillow-heif` para HEIC; `pillow_heif.register_heif_opener()`).
   Navegadores (Chrome/Firefox) **não exibem HEIC** — sempre converter.
4. **Ordem = ordem da mensagem.** O usuário fornece os títulos "respectivamente";
   confira o conteúdo (dimensões batem com a ordem) antes de nomear. **Evite
   abrir muitas imagens com Read** (estoura o contexto — ver §8).
5. **Salvar** em `images/fotografia/<chave-da-galeria>/NN-slug.jpg` (numeração
   sequencial, slug sem acento/espaço). **Não recomprimir** os JPEGs originais.
6. **Ligar na galeria:** adicionar os objetos `{title, src, w, h}` no array
   `photos` da galeria certa — **anexando** às existentes (ou substituindo
   placeholders, se ainda forem gradientes). `w/h` = dimensões reais (fixam a
   proporção e evitam "pulo" no carregamento).
7. Validar, commit, push.

`openGallery()` já suporta foto real (`src` + `w/h`) **e** placeholder
(`skin` + `ratio`) — não quebre essa compatibilidade.

---

## 7. Preview para o usuário

`index.html` aponta para **arquivos locais**, então abrir o HTML sozinho não
mostra as fotos. Gere uma **cópia de preview com as imagens embutidas** e envie
via `SendUserFile`:
- Inline as imagens como `data:image/jpeg;base64,...`, **reduzidas** com Pillow
  (`im.thumbnail((1000,1000)); save(JPEG, quality~72)`) só no preview — o
  repositório mantém as originais.
- Substitua tanto `'caminho'` (aspas simples, no JS) quanto `"caminho"` (aspas
  duplas, no `<img>` do card).
- Mantém o arquivo leve (~3–4 MB). Sem Pillow, instale: `pip install Pillow`.

---

## 8. ⚠️ Limite de 32 MB (contexto da conversa)

A API recusa requisições > 32 MB. Como cada imagem (colada ou aberta com Read)
fica no histórico, sessões longas e com muitas fotos chegam ao limite e **novos
envios de imagem falham** ("Request too large").

Para evitar:
- **Não use Read em muitas imagens.** Confie em dimensões/ordem para mapear; abra
  no máximo 1 para conferência pontual.
- Peça ao usuário **poucas fotos por mensagem** (2–3).
- Quando a conversa ficar pesada, **abra uma sessão nova** apontando para a
  branch `claude/intelligent-euler-1uhqa9`. Este `CLAUDE.md` é justamente para
  isso: começar leve com todo o contexto.

---

## 9. Validação e commit (checklist)

```bash
# 1) sintaxe do JS embutido
awk '/<script>/{f=1;next}/<\/script>/{f=0}f' index.html > /tmp/app.js && node --check /tmp/app.js
# 2) chaves do CSS balanceadas
o=$(grep -o '{' index.html|wc -l); c=$(grep -o '}' index.html|wc -l); echo "$o $c"
# 3) caminhos de imagem existem
grep -oE 'images/[^"'\'' ]+\.jpg' index.html | while read p; do [ -f "$p" ] || echo "FALTA $p"; done
```
Commit em português (ex.: `Galeria X: + N fotos`). Push:
`git push -u origin claude/intelligent-euler-1uhqa9` (retry com backoff se falhar).
Siga o trailer de commit que o harness da sessão instruir (Co-Authored-By /
Claude-Session).

---

## 10. Pendências / TODO

- [ ] **CONCEITOS** — montar a aba (por último). Definir estrutura/modal com o usuário.
- [ ] **AUDIOVISUAL** — inserir os vídeos reais (`src`/`poster` no objeto `films`);
      avaliar migrar o PLAY REEL do hero para o player de vídeo.
- [ ] **E se tudo for cinema?** — definir **capa do card** (sugestões: *Bailarina*
      ou *Casa do Flávio de Carvalho*) e decidir se liga `shuffle`.
- [x] **Recebidas:** *Equilíbrio*, *Espetáculo*, *FIM*, *Fruteira* e *Joia*
      adicionadas a *E se tudo for cinema?* (fotos 11–15 → galeria com 15).
- [ ] Possível: usar fotos reais como capa de mais cards; revisar legendas
      antigas ("[ Foto ]", "[ Filme ]") se o usuário quiser alinhar ao novo vocabulário.
