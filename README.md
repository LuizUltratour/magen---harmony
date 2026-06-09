# Modern Galeria — 360 Home • Office • Mall

Galeria de imagens para injeção via script no 3DVista, hospedada no AWS S3.

**Projeto:** 360 Home • Office • Mall — Indaiatuba, SP  
**Realização:** HINC + HCON Engenharia

---

## URLs de produção

| Arquivo | URL |
|---------|-----|
| Galeria | `https://skylineip.s3.sa-east-1.amazonaws.com/Tour+Virtual/hcon/360-home-office-mall/ferramentas/modern-galery-360/index.html` |
| Script  | `https://skylineip.s3.sa-east-1.amazonaws.com/Tour+Virtual/hcon/360-home-office-mall/ferramentas/modern-galery-360/inject.js` |

**S3 path:** `s3://skylineip/Tour Virtual/hcon/360-home-office-mall/ferramentas/modern-galery-360/`

---

## Estrutura de arquivos

```
modern-galery-360/
├── index.html              ← galeria completa (auto-suficiente)
├── inject.js               ← loader leve para injeção no 3DVista
├── generate_thumbs.py      ← gerador de thumbnails
└── assets/
    ├── fachada/
    ├── lazer/
    ├── torre residencial/
    ├── torre office/
    ├── torre corporativa/
    ├── plantas/
    │   ├── gerais/
    │   ├── apartamentos/
    │   └── duplex/
    └── thumbs/             ← gerado automaticamente
        ├── fachada/
        ├── lazer/
        ├── torre-residencial/
        ├── torre-office/
        ├── torre-corporativa/
        └── plantas/
            ├── gerais/
            ├── apartamentos/
            └── duplex/
```

---

## Categorias da galeria

### Modo `imagens`

| Categoria | Label | Pasta |
|-----------|-------|-------|
| `fachada` | Fachada | `assets/fachada/` |
| `lazer` | Lazer | `assets/lazer/` |
| `torre-residencial` | Torre Residencial | `assets/torre residencial/` |
| `torre-office` | Torre Office | `assets/torre office/` |
| `torre-corporativa` | Torre Corporativa | `assets/torre corporativa/` |

### Modo `plantas`

| Sub-categoria | Label | Pasta |
|---------------|-------|-------|
| `pl-gerais` | Gerais | `assets/plantas/gerais/` |
| `pl-apartamentos` | Apartamentos | `assets/plantas/apartamentos/` |
| `pl-duplex` | Duplex | `assets/plantas/duplex/` |

---

## Cards especiais (`cobertura-plan`)

Usados para plantas com dois pavimentos — abas alternáveis + botão de dual-view:

- **Pavimentos** (pl-gerais): Pavimento 19 ↔ Pavimento Ático
- **Planta Duplex** (pl-duplex): Piso Inferior ↔ Piso Superior (332,49 m²)

```js
{ id:51, type:'cobertura-plan', category:'plantas', subCategory:'pl-gerais',
  title:'Pavimentos', area:'', viewBothLabel:'Ver Pav. 19 e Ático', floors:[
    { label:'Pavimento 19',    src:'assets/plantas/gerais/PAVIMENTO 19.png',    thumb:'assets/thumbs/plantas/gerais/PAVIMENTO 19.png' },
    { label:'Pavimento Ático', src:'assets/plantas/gerais/PAVIMENTO ATICO.png', thumb:'assets/thumbs/plantas/gerais/PAVIMENTO ATICO.png' },
]},
```

---

## Thumbnails

Gerados com `generate_thumbs.py` (Python + Pillow). Tamanho: **900×900 px**, qualidade JPEG **85**.

**Regra de formato:**
- Fonte PNG com canal alfa (transparência) → thumb salvo como **PNG** (plantas técnicas)
- Demais fontes → thumb salvo como **JPEG** (renders fotorrealistas)

```bash
# Gerar apenas os novos (sem sobrescrever)
python generate_thumbs.py

# Forçar regeneração de todos
python generate_thumbs.py --force
```

---

## Deploy AWS S3

### Sync completo

```bash
aws s3 sync . "s3://skylineip/Tour Virtual/hcon/360-home-office-mall/ferramentas/modern-galery-360/" \
  --exclude ".git/*" --exclude ".claude/*" --exclude "*.py" \
  --exclude "README.md" --exclude ".gitattributes" --exclude "*.pdf" \
  --delete
```

### Atualizar só index.html e inject.js

```bash
aws s3 cp index.html "s3://skylineip/Tour Virtual/hcon/360-home-office-mall/ferramentas/modern-galery-360/index.html" \
  --cache-control "no-cache,no-store,must-revalidate"

aws s3 cp inject.js "s3://skylineip/Tour Virtual/hcon/360-home-office-mall/ferramentas/modern-galery-360/inject.js" \
  --cache-control "no-cache,no-store,must-revalidate"
```

> **`--cache-control "no-cache,no-store,must-revalidate"`** — essencial para garantir que o browser nunca sirva uma versão cacheada do script ou da galeria.

---

## Integração 3DVista

### Passo 1 — Loader (colocar no JavaScript global do projeto)

```js
(function(){
  var s = document.createElement('script');
  s.src = 'https://skylineip.s3.sa-east-1.amazonaws.com/Tour+Virtual/hcon/360-home-office-mall/ferramentas/modern-galery-360/inject.js?v=' + Date.now();
  document.head.appendChild(s);
})();
```

> **`?v=` + `Date.now()`** — cache-busting: força o browser a baixar sempre a versão mais recente do script, evitando que o 3DVista sirva uma versão antiga em cache.  
> Esse detalhe foi o que resolveu o problema de redirecionamento para a galeria de outro projeto.

### Passo 2 — Acionar nos hotspots/botões

```js
// Abre galeria de imagens
GaleriaImagens(1);

// Fecha galeria de imagens
GaleriaImagens(0);

// Abre galeria de plantas
GaleriaPlantas(1);

// Fecha galeria de plantas
GaleriaPlantas(0);
```

---

## Cores e tipografia

| Token CSS | Valor |
|-----------|-------|
| `--bg` (fundo) | `#FFFFFF` |
| `--dark` (texto/UI) | `#000000` |
| `--accent` | `#000000` |
| Fonte títulos | Cormorant Garamond |
| Fonte UI | Inter |
