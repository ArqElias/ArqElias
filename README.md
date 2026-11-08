## Hi there 👋

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...

- #!/usr/bin/env python3
"""
Gera o skyline.svg: um construtor animado + grid de quadradinhos verdes
(estilo gráfico de contribuições do GitHub) que vão formando prédios,
andar por andar, enquanto o guindaste trabalha.

Uso:  python gerar_skyline.py
Saída: skyline.svg  (coloque na raiz do repositório de perfil)
"""
import random

random.seed(7)  # troque o número para mudar a distribuição das cores

# ---------------------------------------------------------------- Config
W, H = 900, 340
GROUND_Y = 280          # linha do chão
GRID_X = 135            # onde o grid começa
CELL, PITCH = 10, 13    # tamanho do quadradinho / passo
CYCLE = 20              # duração do ciclo da animação (segundos)

# Prédios: (coluna_inicial, largura_em_colunas, altura_em_andares)
BUILDINGS = [
    (0, 5, 5),
    (6, 4, 8),
    (11, 6, 11),
    (19, 4, 6),
    (24, 5, 9),
    (30, 3, 4),
    (34, 6, 10),
    (41, 4, 7),
    (46, 5, 5),
    (51, 3, 8),
]

# Paleta (tons de verde do GitHub) + janelas mais claras
GREENS = ["#0e4429", "#006d32", "#26a641"]
WINDOW = "#39d353"
BG = "#0d1117"
PANEL = "#161b22"
TEXT = "#c9d1d9"
MUTED = "#8b949e"

# ---------------------------------------------------------------- Células
cells = []   # (x, y, cor, percentual_de_aparição)
n_b = len(BUILDINGS)
for b, (c0, w, h) in enumerate(BUILDINGS):
    start = 3 + b * (62 / n_b)          # prédios são erguidos da esq. p/ dir.
    for r in range(h):
        for c in range(w):
            x = GRID_X + (c0 + c) * PITCH
            y = GROUND_Y - (r + 1) * PITCH
            if r > 0 and r % 2 == 1 and (c % 2 == 1) and w > 3:
                color = WINDOW                    # janelas acesas
            else:
                color = random.choice(GREENS)
            p = start + r * 1.7 + c * 0.25
            p = round(p * 2) / 2                  # arredonda a 0,5%
            cells.append((x, y, color, p))
    # antena no topo dos prédios mais altos
    if h >= 10:
        cx = GRID_X + (c0 + w // 2) * PITCH
        cells.append((cx, GROUND_Y - (h + 1) * PITCH, WINDOW,
                      round((start + h * 1.7 + 1) * 2) / 2))

# ---------------------------------------------------------------- CSS
used = sorted({p for *_, p in cells})
css = [
    "text{font-family:'Segoe UI',Ubuntu,Helvetica,Arial,sans-serif}",
    ".c{transform-box:fill-box;transform-origin:center;opacity:0}",
]
for p in used:
    k = str(p).replace(".", "_")
    p1 = p + 1.5
    css.append(
        f".a{k}{{animation:k{k} {CYCLE}s ease-out infinite}}"
        f"@keyframes k{k}{{0%,{p}%{{opacity:0;transform:scale(.2)}}"
        f"{p1}%{{opacity:1;transform:scale(1)}}"
        f"93%{{opacity:1;transform:scale(1)}}"
        f"100%{{opacity:0;transform:scale(1)}}}}"
    )
css += [
    # martelo do construtor
    ".hammer{transform-origin:78px 214px;animation:bate .9s ease-in-out infinite}"
    "@keyframes bate{0%,100%{transform:rotate(-35deg)}50%{transform:rotate(25deg)}}",
    # respiração leve do construtor
    ".corpo{animation:resp 2.4s ease-in-out infinite}"
    "@keyframes resp{0%,100%{transform:translateY(0)}50%{transform:translateY(-1.5px)}}",
    # balão de fala piscando
    ".fala{animation:fala 4s ease-in-out infinite}"
    "@keyframes fala{0%,100%{opacity:.55}50%{opacity:1}}",
    # luz do guindaste
    ".luz{animation:luz 1.2s steps(2,start) infinite}"
    "@keyframes luz{0%{opacity:1}100%{opacity:.15}}",
    "@media (prefers-reduced-motion:reduce){*{animation:none!important}.c{opacity:1}}",
]

# ---------------------------------------------------------------- SVG
svg = []
svg.append(f'<svg xmlns="http://www.w3.org/2000/svg" width="{W}" height="{H}" '
           f'viewBox="0 0 {W} {H}" role="img" '
           f'aria-label="Status dos repositórios: obra em andamento">')
svg.append("<style>" + "".join(css) + "</style>")
svg.append(f'<rect width="{W}" height="{H}" rx="14" fill="{BG}" stroke="#30363d"/>')

# Título
svg.append(f'<text x="28" y="40" font-size="20" font-weight="700" fill="{TEXT}">'
           f'Obra em andamento</text>')
svg.append(f'<text x="28" y="60" font-size="12" fill="{MUTED}">'
           f'Cada quadradinho é um commit · a cidade cresce com o seu código</text>')

# Chão / calçada
total_w = (max(c0 + w for c0, w, _ in BUILDINGS)) * PITCH
svg.append(f'<rect x="20" y="{GROUND_Y}" width="{W-40}" height="6" rx="3" fill="#30363d"/>')
for i in range(0, W - 60, 28):
    svg.append(f'<rect x="{30+i}" y="{GROUND_Y+12}" width="14" height="3" rx="1.5" fill="#21262d"/>')

# Quadradinhos
for x, y, color, p in cells:
    k = str(p).replace(".", "_")
    svg.append(f'<rect class="c a{k}" x="{x}" y="{y}" width="{CELL}" height="{CELL}" '
               f'rx="2" fill="{color}"/>')

# ----- Guindaste (mastro na coluna 17, entre o 3º e o 4º prédio)
mx = GRID_X + 17 * PITCH + 2        # x do mastro
top = 84
svg.append(f'<g id="guindaste">')
svg.append(f'<rect x="{mx}" y="{top}" width="6" height="{GROUND_Y-top}" fill="#f2b705"/>')
for yy in range(top + 6, GROUND_Y, 14):    # treliça
    svg.append(f'<path d="M{mx} {yy} L{mx+6} {yy+14} M{mx+6} {yy} L{mx} {yy+14}" '
               f'stroke="#8a6d03" stroke-width="1"/>')
svg.append(f'<rect x="{mx-96}" y="{top}" width="196" height="5" fill="#f2b705"/>')            # lança
svg.append(f'<rect x="{mx+70}" y="{top+5}" width="26" height="12" fill="#6e7681"/>')         # contrapeso
svg.append(f'<polygon points="{mx-8},{top} {mx+3},{top-22} {mx+14},{top}" fill="#f2b705"/>')  # torre
svg.append(f'<circle class="luz" cx="{mx+3}" cy="{top-24}" r="3" fill="#ff4d4d"/>')
# gancho (SMIL: sobe e desce)
hx = mx - 52
svg.append(f'<line x1="{hx}" y1="{top+5}" x2="{hx}" y2="130" stroke="#c9d1d9" stroke-width="1.5">'
           f'<animate attributeName="y2" values="110;134;110" dur="6s" repeatCount="indefinite"/></line>')
svg.append(f'<g><animateTransform attributeName="transform" type="translate" '
           f'values="0 0;0 24;0 0" dur="6s" repeatCount="indefinite"/>'
           f'<path d="M{hx-5} 110 q5 12 10 0" stroke="#c9d1d9" stroke-width="2" fill="none"/>'
           f'<rect x="{hx-6}" y="119" width="12" height="12" rx="2" fill="{WINDOW}"/></g>')
svg.append("</g>")

# ----- Construtor (personagem original)
svg.append('<g id="construtor">')
# sombra
svg.append(f'<ellipse cx="62" cy="{GROUND_Y+2}" rx="30" ry="4" fill="#000" opacity=".35"/>')
# pernas + botas
svg.append('<rect x="49" y="248" width="11" height="30" rx="3" fill="#1f3a68"/>')
svg.append('<rect x="64" y="248" width="11" height="30" rx="3" fill="#1f3a68"/>')
svg.append('<rect x="46" y="272" width="16" height="8" rx="3" fill="#3b2a1a"/>')
svg.append('<rect x="62" y="272" width="16" height="8" rx="3" fill="#3b2a1a"/>')
svg.append('<g class="corpo">')
# braço esquerdo (segurando a planta)
svg.append('<rect x="30" y="210" width="10" height="32" rx="5" fill="#f78c1f"/>')
svg.append('<circle cx="35" cy="244" r="5" fill="#f1c27d"/>')
svg.append('<rect x="22" y="232" width="18" height="12" rx="2" fill="#4cc3ff" opacity=".9"/>'
           '<path d="M26 236h10M26 240h7" stroke="#fff" stroke-width="1"/>')
# tronco: colete laranja com faixas refletivas
svg.append('<rect x="40" y="206" width="44" height="46" rx="9" fill="#f78c1f"/>')
svg.append('<rect x="40" y="222" width="44" height="5" fill="#ffe27a"/>')
svg.append('<rect x="40" y="234" width="44" height="5" fill="#ffe27a"/>')
svg.append('<rect x="59" y="206" width="6" height="46" fill="#d9740f"/>')
# pescoço e cabeça
svg.append('<rect x="56" y="198" width="12" height="10" rx="3" fill="#e0ac69"/>')
svg.append('<circle cx="62" cy="184" r="16" fill="#f1c27d"/>')
svg.append('<circle cx="56" cy="184" r="1.8" fill="#222"/><circle cx="68" cy="184" r="1.8" fill="#222"/>')
svg.append('<path d="M55 191 q7 7 14 0" stroke="#7a3b1d" stroke-width="2" fill="none" stroke-linecap="round"/>')
# capacete amarelo
svg.append('<path d="M44 178 a18 16 0 0 1 36 0 z" fill="#ffd23f"/>')
svg.append('<rect x="40" y="176" width="44" height="6" rx="3" fill="#f2b705"/>')
svg.append('<rect x="59" y="162" width="6" height="14" rx="2" fill="#f2b705"/>')
# braço direito com martelo (animado)
svg.append('<g class="hammer">'
           '<rect x="76" y="208" width="10" height="34" rx="5" fill="#f78c1f" '
           'transform="rotate(-70 78 214)"/>'
           '<rect x="96" y="190" width="5" height="30" rx="2" fill="#8b5a2b" '
           'transform="rotate(-20 98 205)"/>'
           '<rect x="86" y="184" width="24" height="11" rx="2" fill="#8b949e" '
           'transform="rotate(-20 98 190)"/>'
           '</g>')
svg.append("</g>")
# balão de fala
svg.append('<g class="fala">'
           '<rect x="6" y="116" width="104" height="26" rx="8" fill="#21262d" stroke="#30363d"/>'
           '<path d="M44 142 l6 9 l6 -9 z" fill="#21262d" stroke="#30363d"/>'
           f'<text x="58" y="133" text-anchor="middle" font-size="11" fill="{TEXT}">Construindo...</text>'
           '</g>')
svg.append("</g>")

# Legenda (Menos ... Mais)
lx = W - 200
svg.append(f'<text x="{lx}" y="{H-16}" font-size="11" fill="{MUTED}">Menos</text>')
for i, col in enumerate(["#161b22"] + GREENS + [WINDOW]):
    svg.append(f'<rect x="{lx+40+i*16}" y="{H-26}" width="11" height="11" rx="2" fill="{col}" stroke="#30363d"/>')
svg.append(f'<text x="{lx+40+5*16+4}" y="{H-16}" font-size="11" fill="{MUTED}">Mais</text>')
svg.append(f'<text x="28" y="{H-16}" font-size="11" fill="{MUTED}">Arquitetura · BIM · Construção civil</text>')

svg.append("</svg>")

with open("skyline.svg", "w", encoding="utf-8") as f:
    f.write("\n".join(svg))

print(f"skyline.svg gerado: {len(cells)} quadradinhos, {len(used)} tempos de animação")
