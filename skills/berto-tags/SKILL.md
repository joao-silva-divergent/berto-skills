---
name: "berto-tags"
description: "Berto, o editor de vídeo da Divergent: põe a tag com o nome do médico e o n.º da ordem (OM/OMD) nos clips legendados em que ele fala, logo a seguir ao /berto-captions. Usar com /berto-tags."
---

# Berto: tag com o nome do médico + n.º da ordem

"Berto" é o editor de vídeo da Divergent. Quando chamarem esta skill, apresenta-te como Berto numa linha e segue os passos abaixo. Qualquer pessoa da equipa a pode usar, para qualquer marca. Não assumas pastas, fonte nem números: pergunta SEMPRE.

Lugar no fluxo: **/berto-captions → /berto-tags → /berto-combos → /berto-musica**.

Objetivo: nos clips legendados (pasta `legendados/`), escrever em cima, à esquerda, duas linhas: o nome do médico e o n.º da ordem.

```
Dra. Paula Rodrigues
OMD 00935
```

- **Estilo de referência:** texto branco (95 % de opacidade) com sombra suave para baixo e para a direita, letra condensada e leve (Oswald Light no exemplo).
  - Posição: 6,9 % da largura a partir da esquerda e 13 % da altura a partir de cima.
  - Altura das maiúsculas: 2 % da altura do vídeo (39 px em 1080x1920). A 2.ª linha fica a 1,5 alturas de maiúscula da 1.ª.
  - Com outras resoluções, o texto escala na mesma proporção.
- **Só nos clips do médico.** O nome do ficheiro diz quem fala (ex.: `MEAT2_ENGRAVIDAR_LIA_LEG.mp4` → LIA). Normalmente é o MEAT e o CTA, mas a regra é o nome, não o tipo de clip. Os clips de outras pessoas (ex.: hooks de pacientes) são copiados sem alterações.
- **Qualidade.** O texto fica no vídeo durante o clip todo. O vídeo é recodificado em H.264 com o mesmo bitrate do original (entre 8 e 60 Mbps) e o áudio é copiado sem mexer.
- **Resultado.** Uma pasta nova ao lado da pasta dos clips, com o sufixo `_tags` (ex.: `legendados` → `legendados_tags`), com TODOS os clips e os mesmos nomes de ficheiro. É essa pasta que se usa depois no /berto-combos. Os originais nunca são alterados.

## REGRA OBRIGATÓRIA: perguntar TUDO na janela de opções (AskUserQuestion)

Todas as perguntas desta skill são feitas com **AskUserQuestion**: fonte, peso, pasta, nome e n.º de cada médico, e a aprovação da pré-visualização. Nunca as faças em texto livre (só se a ferramenta não existir). Valores próprios, como um caminho, um nome ou um número, o utilizador escreve-os no campo "Other". Diz isso no texto de cada pergunta.

Nunca terminar uma resposta com uma pergunta em texto livre. Qualquer aviso que precise de decisão vai para a janela, para não passar despercebido: ficheiro com dois médicos no nome, tag por cima de algo importante (cara, logotipo, legenda), fonte inexistente, clip sem pessoa reconhecível no nome, erros na verificação.

**Exceção:** quando o /berto-master chama esta skill com as respostas já recolhidas (fonte, peso, pasta, médicos), não voltar a perguntar o que já foi respondido. Se o master estiver em modo "Fazer tudo seguido", saltar a aprovação da pré-visualização (usar os valores por defeito), mas confirmar na mesma com Read que a tag está legível e não tapa a cara; se tapar, perguntar na janela.

**A. No início de cada utilização, mesmo que já se tenha usado na conversa: UMA chamada com 3 perguntas.**

1. **Fonte** (header "Fonte"). Opções iguais às do /berto-captions:
   - **Roboto**;
   - **Montserrat**;
   - **Poppins**;
   - **Inter**.
   
   Texto da pergunta: "Que Google Font para a tag? Podes escrever outra em Other (ex.: Oswald, a do estilo de referência)."
2. **Peso** (header "Peso"). Opções:
   - **Light 300 (Recommended)**, como no exemplo;
   - **Regular 400**;
   - **Medium 500**.
3. **Pasta dos clips** (header "Pasta"). Até 4 opções com pastas `legendados` candidatas. Em cada uma, o label é curto (ex.: "GRÁVIDAS/legendados") e a description é o caminho completo. As candidatas, por ordem:
   - as usadas nesta conversa (ex.: a do último /berto-captions);
   - as que encontrares nas pastas já ligadas, com `device_bash`: `find $HOME/mnt -maxdepth 5 -type d -name 'legendados' 2>/dev/null`;
   - as que encontrares com `device_list_dir` em `~/Downloads`, `~/Desktop` e `~/Documents` (sem acesso dá só os nomes, o que chega para ver se existe uma `legendados`).
   
   Texto da pergunta: "Que pasta de clips legendados? Se não estiver na lista, cola o caminho completo em Other (Mac: no Finder, botão direito → Option → 'Copiar … como nome de caminho')." Sem candidatas, as opções são "Colo o caminho em Other" e "Procura em Downloads" (nesse caso, lista `~/Downloads` e volta a perguntar com o que encontrares).

Espera pelas respostas antes de continuar.

Regras:
- NÃO uses ficheiros anexados ao chat: o Berto só trabalha a partir de pastas no computador.
- Pede acesso com UMA chamada `device_request_folder_access` com a pasta-mãe da pasta dos clips (é lá que se cria a `_tags`) e a pasta da fonte. Se uma estiver dentro da outra, pede só a de cima. Usa reason = "O Berto precisa destas pastas para ler os clips e a fonte e guardar os clips com a tag do médico". Se o acesso for recusado ou o computador não estiver ligado a esta sessão, para e explica o que falta.
- Os caminhos com acentos podem vir noutra forma Unicode: o script resolve isso sozinho. Se mesmo assim não encontrar, lista a pasta-mãe com `device_list_dir`.
- **Obter a fonte:** `python3 $HOME/berto/tags.py font "<Família>" <peso> $HOME/berto/fontes`. O script descarrega a Google Font com `npm pack @fontsource/<nome>`, escolhe o peso mais próximo e imprime `FONTE=<caminho>`: usa esse caminho como `<fonte>`.
  - Se não houver npm ou rede no computador, corre o mesmo comando na cloud com `Bash`, envia o .woff com SendUserFile e grava-o com `device_commit_files` em `<pasta-mãe>/fontes/`. Diz ao utilizador que o deixaste aí.
  - Se a fonte não existir no Google Fonts, volta a perguntar a fonte com AskUserQuestion.
  - Se o utilizador escrever em Other o caminho de um ficheiro .ttf/.otf/.woff do computador, usa esse ficheiro (incluindo a pasta dele no pedido de acesso).

## Passos

Faz todo o trabalho no computador do utilizador com `device_bash`. Pasta de trabalho na VM: `$HOME/berto` (fora de `mnt/`).

1. **Perguntar a fonte, o peso e a pasta** (regra A, numa janela de opções) e pedir acesso.
2. **Verificar as ferramentas:**
   - `which ffmpeg ffprobe python3 && python3 -c "import PIL; print(PIL.__version__)"`;
   - se faltar o Pillow: `pip install --break-system-packages pillow` (ou `pip install --user pillow`);
   - se faltar o ffmpeg, tenta apt-get/brew. Se não der, explica e para.
3. **Escrever o script**: `mkdir -p $HOME/berto` e depois escrever o script abaixo em `$HOME/berto/tags.py` com um heredoc com aspas (`cat > $HOME/berto/tags.py <<'PYEOF' … PYEOF`). Apaga qualquer `$HOME/berto/pessoas_tags.csv` antigo. Depois obtém a fonte com `tags.py font` (ver Regras).
4. **Analisar**: `python3 $HOME/berto/tags.py scan "<pasta>"`. O script mostra os clips agrupados pela pessoa que sugere (o último bloco do nome) e todos os blocos que aparecem nos nomes.
5. **Pedir o nome e o n.º de cada médico (janela de opções).** Faz UMA pergunta por pessoa encontrada (no máximo 4 por chamada; se houver mais pessoas, faz outra chamada a seguir).
   - **Header:** a chave (ex.: "LIA").
   - **Pergunta:** "Tag para LIA (MEAT2, CTA2, CTA3)? Se for médico/a, escreve em Other: Dra. Nome Apelido | OMD 00000".
   - **Opções:**
     - "Sem tag": não é médico/a nem paciente a dar testemunho (ex.: atriz de um hook);
     - "É médico/a: escrevo em Other";
     - "É paciente (testemunho)": tag "Nome / Paciente <Marca>" (ex.: "Ana" / "Paciente Clínica Exemplo"), no mesmo estilo da tag do médico. O nome escreve-se em Other; por defeito só o primeiro nome. Se a marca ainda não tiver sido dita, perguntar.
     - se já houver nesta conversa um nome e n.º para essa pessoa, junta essa opção primeiro, ex.: "Dra. Lia Costa | OMD 12345 (Recommended)".
   
   Interpretar a resposta:
   - **Linha 1** = título e nome (ex.: "Dra. Paula Rodrigues");
   - **Linha 2** = ordem e número (ex.: "OMD 00935" ou "OM 12345"); numa paciente, "Paciente <Marca>".

   **A mesma pessoa leva sempre a mesma tag.** Se um/a médico/a também aparece a dar testemunho (ex.: a Dra. Sofia no MEAT e a Sofia num clip de testemunho, com outro nome de ficheiro como `V2_SOFIA_SANTOS_03…`), esse testemunho leva a tag de médico/a ("Dra. Sofia Santos / OM 00000"), nunca "Paciente": no mesmo anúncio, a mesma pessoa não pode aparecer como paciente e depois como médica. Antes de gravar, cruzar os blocos dos nomes de todos os clips (o scan mostra-os) e juntar as chaves que são a mesma pessoa.
   
   O texto pode vir separado por "|", vírgula ou mudança de linha. Se vier só o número, pergunta numa nova janela: "OMD (dentistas)" ou "OM (médicos)". Se escolher "É médico/a" sem escrever nada, ou se faltar o nome, volta a perguntar só por essa pessoa. Quem tiver "Sem tag" fica sem tag. Se o nome do médico estiver noutro bloco do nome do ficheiro (não no último), usa esse bloco como chave.
6. **Gravar as pessoas:**
   - `python3 $HOME/berto/tags.py template "<pasta>" $HOME/berto/pessoas_tags.csv`;
   - depois, para cada médico, `python3 $HOME/berto/tags.py set $HOME/berto/pessoas_tags.csv <CHAVE> "<linha 1>" "<linha 2>"`. A chave é o bloco do nome do ficheiro, ex.: LIA.
   - Escreve os textos exatamente como o utilizador os deu (pontos, acentos, maiúsculas).
7. **Pré-visualização:**
   - `python3 $HOME/berto/tags.py preview "<pasta>" "<fonte>" $HOME/berto/pessoas_tags.csv "<pasta-mãe>/tags_preview.png"` (gera um frame de um clip do médico; `--file` escolhe outro e `--t` outro segundo);
   - faz staging desse PNG, abre-o com Read para confirmar que o texto está legível e bem posicionado, e envia-o com SendUserFile;
   - pergunta se está bem, com AskUserQuestion (header "Tag"). Opções:
     - "Está bem: fazer todos (Recommended)";
     - "Maior";
     - "Mais pequena";
     - "Mudar posição" (com Other para outros pedidos).
     
     Ajustes possíveis: `--cap` (tamanho, padrão 0.0205; maior/mais pequena = ±15 %), `--x`/`--y` (posição, padrão 0.069/0.130), `--sombra` (0 a 1, padrão 0.45), `--cor` (hex, padrão FFFFFF), `--pitch` (espaço entre linhas, padrão 1.5);
   - se a tag ficar por cima de algo importante (cara, logotipo, legenda), sugere mudar a posição. Repete até o utilizador aprovar.
   - O PNG da pré-visualização fica na pasta-mãe. Nas pastas ligadas não se pode apagar ficheiros sem autorização, por isso, no fim, diz ao utilizador que o pode apagar (ou pede permissão de apagar e apaga-o). Os ficheiros temporários do script ficam fora das pastas do utilizador.
8. **Render**: `python3 $HOME/berto/tags.py render "<pasta>" "<fonte>" $HOME/berto/pessoas_tags.csv "<pasta>_tags" --budget 140 [ajustes aprovados]` com `timeout_ms` 180000. Repete até aparecer "N/N clips prontos".
9. **Verificar:**
   - os clips sem tag são iguais aos originais (`cmp`);
   - em 1–2 clips com tag, a duração, a resolução e o fps são iguais aos do original (ffprobe);
   - `ffmpeg -v error -i <f> -f null -` descodifica sem erros;
   - extrai um frame de um clip final e confirma com Read que a tag lá está.
10. **Entregar**: resposta curta.
    - Quantos clips levaram tag e de quem.
    - A pasta `_tags` criada.
    - Que, no /berto-combos, deve indicar essa pasta (`…/legendados_tags`) em vez da `legendados`.

## Notas

- Para corrigir um nome ou número: `set` de novo, apaga os clips dessa pessoa em `<pasta>_tags/` (pede permissão de apagar) e corre `render`. Só é refeito o que falta.
- Se o nome de um ficheiro tiver o nome de dois médicos, é usado o primeiro, com um AVISO. Pergunta na janela de opções qual é o certo (uma opção por médico).
- `--limit N` no render faz só N clips (útil para testar).
- Num clip de testemunho, a tag fica só enquanto a pessoa está no ecrã: sai antes de um B-roll ou de uma transição e volta depois (num clip inteiro de uma só pessoa, o render normal chega).

## Script `tags.py`

```python
#!/usr/bin/env python3
"""Tag com o nome do médico + n.º da ordem (OM/OMD) sobre os clips legendados do Berto.

Uso:
  python3 tags.py font     "FAMÍLIA" PESO DIR_FONTES                    (Google Font via npm; imprime FONTE=<caminho>)
  python3 tags.py scan     PASTA
  python3 tags.py template PASTA PESSOAS.csv
  python3 tags.py set      PESSOAS.csv CHAVE "LINHA 1" "LINHA 2"      (LINHA 1 vazia = sem tag nesse clip)
  python3 tags.py preview  PASTA FONTE PESSOAS.csv OUT.png [--file NOME] [--t 1.0] [estilo]
  python3 tags.py render   PASTA FONTE PESSOAS.csv OUT_DIR [--budget 140] [--limit N] [estilo]

Estilo (proporções do frame; o padrão imita o exemplo da Divergent):
  --x 0.069  --y 0.130   canto superior esquerdo da 1.ª linha (fração da largura / altura)
  --cap 0.0205           altura das maiúsculas (fração da altura do vídeo)
  --pitch 1.5            distância entre linhas (em alturas de maiúscula)
  --opac 0.95            opacidade do texto branco
  --sombra 0.45          opacidade da sombra (0 = sem sombra)
  --cor FFFFFF           cor do texto (hex)

- Só levam tag os clips cujo nome contém (como bloco separado por _) a chave de uma pessoa com
  texto no PESSOAS.csv (ex.: chave LIA -> MEAT2_ENGRAVIDAR_LIA_LEG e CTA2_ENGRAVIDAR_LIA_LEG).
  Os outros (ex.: hooks de pacientes) são copiados sem alterações. O scan sugere como pessoa o
  último bloco do nome, mas qualquer bloco serve de chave (set cria a chave se não existir).
- render: só os ficheiros HOOK/MEAT/BODY/CTA. Vídeo recodificado em H.264 com o bitrate do original
  (8-60 Mbps), áudio copiado. Pessoas sem tag: o clip é copiado tal como está. Incremental.
"""
import csv, os, re, subprocess, sys, time, json, unicodedata, tempfile
from PIL import Image, ImageDraw, ImageFont, ImageFilter

EXT = (".mp4", ".mov", ".m4v")

def resolve(path):
    path = os.path.abspath(os.path.expanduser(path))
    if os.path.exists(path): return path
    cur = "/"
    for part in path.strip("/").split("/"):
        nxt = os.path.join(cur, part)
        if not os.path.exists(nxt):
            want = unicodedata.normalize("NFC", part)
            hit = [e for e in (os.listdir(cur) if os.path.isdir(cur) else []) if unicodedata.normalize("NFC", e) == want]
            if not hit: return path
            nxt = os.path.join(cur, hit[0])
        cur = nxt
    return cur

def natkey(s): return [int(t) if t.isdigit() else t.lower() for t in re.split(r"(\d+)", s)]

def clips(d):
    return [f for f in sorted(os.listdir(d), key=natkey) if not f.startswith(".") and f.lower().endswith(EXT)
            and os.path.isfile(os.path.join(d, f)) and f.upper().startswith(("HOOK", "MEAT", "BODY", "CTA"))]

def tokens(f):
    return [t.upper() for t in re.split(r"[_\- ]+", re.sub(r"_LEG$", "", os.path.splitext(f)[0], flags=re.I)) if t]

def person(f):
    t = tokens(f); return t[-1] if len(t) > 1 else "?"

def probe(p):
    j = json.loads(subprocess.run(["ffprobe", "-v", "error", "-select_streams", "v:0", "-show_entries",
        "stream=width,height,r_frame_rate,bit_rate:format=bit_rate,duration", "-of", "json", p],
        capture_output=True, text=True).stdout)
    s = j["streams"][0]; br = s.get("bit_rate") or j["format"].get("bit_rate")
    n, d = s["r_frame_rate"].split("/")
    return dict(w=int(s["width"]), h=int(s["height"]), fps=float(n) / float(d),
                br=int(br) if br and br != "N/A" else 0, dur=float(j["format"]["duration"]))

def read_people(p):
    with open(p, newline="") as f: return {r["chave"]: r for r in csv.DictReader(f)}

def write_people(p, rows):
    with open(p, "w", newline="") as f:
        w = csv.DictWriter(f, fieldnames=["chave", "linha1", "linha2", "clips"]); w.writeheader(); w.writerows(rows)

def style(a):
    opt = lambda k, d: type(d)(a[a.index(k) + 1]) if k in a else d
    return dict(x=opt("--x", 0.069), y=opt("--y", 0.130), cap=opt("--cap", 0.0205), pitch=opt("--pitch", 1.5),
                opac=opt("--opac", 0.95), sombra=opt("--sombra", 0.45), cor=opt("--cor", "FFFFFF"))

def font_for(fontpath, cap_px):
    """Escolhe o tamanho da fonte para que a altura das maiúsculas seja cap_px."""
    size = max(8, int(cap_px * 1.4))
    for _ in range(6):
        f = ImageFont.truetype(fontpath, size)
        b = f.getbbox("H"); h = b[3] - b[1]
        if h <= 0: break
        new = max(8, round(size * cap_px / h))
        if new == size: break
        size = new
    return ImageFont.truetype(fontpath, size)

def overlay_png(W, H, lines, fontpath, st, out):
    cap = st["cap"] * H; f = font_for(fontpath, cap)
    capoff = f.getbbox("H")[1]  # distância do topo do "em" ao topo das maiúsculas
    x = st["x"] * W; y0 = st["y"] * H
    txt = Image.new("L", (W, H), 0); d = ImageDraw.Draw(txt)
    for i, line in enumerate([l for l in lines if l]):
        d.text((x, y0 + i * st["pitch"] * cap - capoff), line, font=f, fill=255)
    rgb = tuple(int(st["cor"][i:i + 2], 16) for i in (0, 2, 4))
    img = Image.new("RGBA", (W, H), (0, 0, 0, 0))
    if st["sombra"] > 0:
        off = max(1, round(0.0018 * H))
        sh = Image.new("L", (W, H), 0); sh.paste(txt, (off, round(off * 1.4)))
        sh = sh.filter(ImageFilter.GaussianBlur(max(1, 0.0028 * H)))
        sh = sh.point(lambda v: int(v * st["sombra"]))
        img = Image.composite(Image.new("RGBA", (W, H), (0, 0, 0, 255)), img, sh)
        img.putalpha(sh)
    fg = Image.new("RGBA", (W, H), rgb + (255,)); fg.putalpha(txt.point(lambda v: int(v * st["opac"])))
    img = Image.alpha_composite(img, fg)
    img.save(out)

# ---------- comandos ----------

def cmd_font(family, weight, outdir):
    """Descarrega a Google Font (npm @fontsource/<nome>) e devolve o .woff latin normal do peso mais próximo."""
    slug = re.sub(r"[^a-z0-9]+", "-", family.lower()).strip("-")
    d = os.path.join(os.path.abspath(os.path.expanduser(outdir)), slug); os.makedirs(d, exist_ok=True)
    files = os.path.join(d, "package", "files")
    if not os.path.isdir(files):
        r = subprocess.run(["npm", "pack", "--silent", f"@fontsource/{slug}"], cwd=d, capture_output=True, text=True)
        if r.returncode != 0: sys.exit(f"Fonte '{family}' não encontrada no Google Fonts (@fontsource/{slug}).")
        subprocess.run(["tar", "xzf", r.stdout.strip().splitlines()[-1]], cwd=d, check=True)
    ws = sorted({int(m.group(1)) for f in os.listdir(files) if (m := re.match(rf"{re.escape(slug)}-latin-(\d+)-normal\.woff$", f))})
    if not ws: sys.exit(f"Fonte '{family}' sem ficheiros latin normais.")
    w = min(ws, key=lambda x: (abs(x - weight), x))
    p = os.path.join(files, f"{slug}-latin-{w}-normal.woff")
    print(f"fonte {family} peso {w} (pedido {weight}; disponíveis {ws})"); print("FONTE=" + p)

def cmd_scan(d):
    cs = clips(d); groups = {}
    for f in cs: groups.setdefault(person(f), []).append(f)
    print("clips:", len(cs))
    if cs: p = probe(os.path.join(d, cs[0])); print(f"formato: {p['w']}x{p['h']} {p['fps']:.2f}fps")
    for k, v in groups.items(): print(f"pessoa {k}: {len(v)} clips -> {', '.join(v)}")
    print("blocos nos nomes:", sorted({t for f in cs for t in tokens(f)[1:]}))
    other = [f for f in os.listdir(d) if f.lower().endswith(EXT) and f not in cs]
    if other: print("ignorados:", other)

def cmd_template(d, pp):
    groups = {}
    for f in clips(d): groups.setdefault(person(f), []).append(f)
    old = read_people(pp) if os.path.exists(pp) else {}
    rows = [{"chave": k, "linha1": old.get(k, {}).get("linha1", ""), "linha2": old.get(k, {}).get("linha2", ""),
             "clips": len(v)} for k, v in groups.items()]
    write_people(pp, rows); print(f"{len(rows)} pessoas em {pp}: {[r['chave'] for r in rows]}")

def cmd_set(pp, key, l1, l2):
    ppl = read_people(pp); key = key.upper()
    if key not in ppl: ppl[key] = {"chave": key, "linha1": "", "linha2": "", "clips": ""}
    ppl[key]["linha1"] = l1; ppl[key]["linha2"] = l2
    write_people(pp, list(ppl.values()))
    for r in ppl.values(): print(f"{r['chave']}: {r['linha1'] or '(sem tag)'} | {r['linha2']}")

def lines_for(ppl, f):
    """Texto do clip: a 1.ª chave com texto que aparece como bloco no nome do ficheiro (ex.: LIA em
    MEAT2_ENGRAVIDAR_LIA_LEG). Clips sem nenhuma chave com texto ficam sem tag."""
    t = tokens(f)
    hits = [r for k, r in ppl.items() if r.get("linha1") and k.upper() in t]
    if len(hits) > 1: print(f"AVISO {f}: várias pessoas no nome ({[h['chave'] for h in hits]}), usada {hits[0]['chave']}")
    return [hits[0]["linha1"], hits[0]["linha2"]] if hits else None

def cmd_preview(d, font, pp, out, a, st):
    ppl = read_people(pp); cs = clips(d)
    f = a[a.index("--file") + 1] if "--file" in a else next((c for c in cs if lines_for(ppl, c)), None)
    if not f: sys.exit("nenhum clip com tag")
    t = float(a[a.index("--t") + 1]) if "--t" in a else 1.0
    p = probe(os.path.join(d, f)); png = os.path.join(tempfile.gettempdir(), "berto_tag_preview.png")
    overlay_png(p["w"], p["h"], lines_for(ppl, f) or [], font, st, png)
    subprocess.run(["ffmpeg", "-y", "-v", "error", "-ss", str(t), "-i", os.path.join(d, f), "-i", png,
                    "-filter_complex", "[0:v][1:v]overlay=0:0", "-frames:v", "1", out], check=True)
    os.remove(png); print("preview", f, "->", out)

def cmd_render(d, font, pp, out, a, st):
    ppl = read_people(pp); os.makedirs(out, exist_ok=True)
    com = [f for f in clips(d) if lines_for(ppl, f)]
    print(f"{len(com)} clips com tag, {len(clips(d)) - len(com)} sem tag (copiados tal como estão)")
    if not com: sys.exit("nenhum clip corresponde a uma pessoa com texto: ver o pessoas_tags.csv")
    opt = lambda k, dft: type(dft)(a[a.index(k) + 1]) if k in a else dft
    budget = opt("--budget", 140.0); limit = opt("--limit", 0)
    t0 = time.time(); made = left = 0; cs = clips(d)
    for f in cs:
        src = os.path.join(d, f); dst = os.path.join(out, f)
        if os.path.exists(dst) or (limit and made >= limit): continue
        if time.time() - t0 > budget: left += 1; continue
        ln = lines_for(ppl, f)
        if not ln:
            subprocess.run(["cp", "-p", src, dst + ".part"], check=True); os.replace(dst + ".part", dst)
            made += 1; print("copiado (sem tag)", f, flush=True); continue
        p = probe(src); br = min(60_000_000, max(8_000_000, p["br"]))
        png = os.path.join(tempfile.gettempdir(), "berto_tag_" + f + ".png")
        overlay_png(p["w"], p["h"], ln, font, st, png)
        subprocess.run(["ffmpeg", "-y", "-v", "error", "-i", src, "-i", png, "-filter_complex",
                        "[0:v][1:v]overlay=0:0:format=auto,format=yuv420p[v]", "-map", "[v]", "-map", "0:a?",
                        "-c:v", "libx264", "-preset", "medium", "-profile:v", "high", "-b:v", str(br),
                        "-maxrate", str(int(br * 1.3)), "-bufsize", str(br * 2), "-c:a", "copy",
                        "-movflags", "+faststart", "-f", "mp4", dst + ".part"], check=True)
        os.replace(dst + ".part", dst); os.remove(png); made += 1; print("ok", f, flush=True)
    done = sum(os.path.exists(os.path.join(out, f)) for f in cs)
    print(f"{done}/{len(cs)} clips prontos" + ("" if done == len(cs) else " — correr render outra vez"))

if __name__ == "__main__":
    a = sys.argv[1:]
    if len(a) < 2: print(__doc__); sys.exit(1)
    c = a[0]
    if c == "font": cmd_font(a[1], int(a[2]), a[3])
    elif c == "scan": cmd_scan(resolve(a[1]))
    elif c == "template": cmd_template(resolve(a[1]), a[2])
    elif c == "set": cmd_set(a[1], a[2], a[3], a[4] if len(a) > 4 else "")
    elif c == "preview": cmd_preview(resolve(a[1]), resolve(a[2]), a[3], a[4], a, style(a))
    elif c == "render":
        out = a[4]; parent = resolve(os.path.dirname(os.path.abspath(out)))
        cmd_render(resolve(a[1]), resolve(a[2]), a[3], os.path.join(parent, os.path.basename(os.path.abspath(out))), a, style(a))
    else: print(__doc__); sys.exit(1)
```