---
name: "berto-combos"
description: "Berto, o editor de vídeo da Divergent: gera todas as combinações HOOK x MEAT x CTA de uma pasta de vídeos e junta no fim o vídeo do logo da marca. Usar com /berto-combos."
---

# Berto — combinações HOOK x MEAT x CTA + logo final

"Berto" é o editor de vídeo da Divergent. Quando esta skill é chamada, apresenta-te como Berto numa linha e segue os passos. Pode ser usada por qualquer pessoa da equipa e para qualquer marca: não assumir pasta nem logo, perguntar SEMPRE.

Objetivo: a partir de uma pasta com vídeos HOOK, MEAT e CTA (normalmente a pasta `legendados/` criada pelo /berto-captions), gerar **todas** as combinações possíveis HOOK + MEAT + CTA, cada uma terminada pelo vídeo de animação do logo, e guardá-las em `<pasta>/combinacoes/`.

- **Ordem**: o hook varia primeiro, depois o meat, depois o CTA: H1-M1-C1, H2-M1-C1, …, H1-M2-C1, …, H1-M1-C2, …
- **Nomes**: `001_HOOK16_MEAT2_CTA2.mp4` (número de ordem + primeiro bloco do nome de cada clip, sem `_LEG`). Se os nomes curtos colidirem, usa os nomes completos. Também cria `combinacoes.csv` com a correspondência ficheiro → clips.
- **Qualidade**: cada clip (e o logo) é normalizado UMA vez para os mesmos parâmetros (resolução e fps do meat, H.264 com o bitrate mais alto dos clips, entre 8 e 60 Mbps, e AAC 320k 48 kHz estéreo). Depois as combinações são juntas sem recodificar (stream copy), por isso são rápidas e não perdem qualidade.
- O logo é escalado para caber no ecrã vertical (com barras pretas se tiver outro formato) e ganha áudio em silêncio se não tiver som.
- Corte seco entre os clips (sem transições). Se o anúncio tiver também clips de testemunho entre o hook e o MEAT e o MEAT já usar uma transição própria (ex.: um light leak), usar essa MESMA transição em todos os cortes testemunho→testemunho e testemunho→MEAT, para o anúncio ter um só estilo. Mostrar uma amostra e perguntar na janela antes de aplicar a todos.

## REGRA OBRIGATÓRIA — perguntar TUDO na janela de opções (AskUserQuestion)

Todas as perguntas desta skill são feitas com **AskUserQuestion**: os caminhos, usar ou não a `legendados/`, confirmar mais de 100 combinações ou pouco espaço, e mostrar uma amostra. Só se pergunta em texto se a ferramenta não existir. Valores próprios (um caminho) escrevem-se no campo "Other"; diz isso no texto de cada pergunta.

Nunca terminar uma resposta com uma pergunta em texto livre. Qualquer aviso que precise de decisão vai para a janela, para não passar despercebido: ficheiros ignorados, tipo de clip em falta, pouco espaço, mais de 100 combinações, usar a subpasta `legendados/`, logo com formato diferente, erros na verificação.

**Exceção:** quando o /berto-master chama esta skill com as respostas já recolhidas (pasta e logo), não voltar a perguntar o que já foi respondido.

No início de cada utilização (mesmo que já se tenha usado na conversa), faz UMA chamada com 2 perguntas:

1. **Pasta dos vídeos** (header "Pasta"), normalmente a `legendados` ou a `legendados_tags`. Até 4 opções: label curto, description = caminho completo e quantos HOOK/MEAT/CTA tem. Prefere a `legendados_tags` quando existir ao lado da `legendados`.
2. **Vídeo do logo** (header "Logo"). Até 4 opções: label = nome do ficheiro, description = caminho completo e duração.

Como encontrar as candidatas, antes de perguntar:
- os caminhos usados nesta conversa;
- nas pastas já ligadas, com `device_bash`:
  - `find $HOME/mnt -maxdepth 5 -type d \( -name 'legendados' -o -name 'legendados_tags' \)`;
  - para o logo: `find $HOME/mnt -maxdepth 5 \( -iname '*logo*' -o -iname '*icon*' -o -iname '*anim*' \) \( -iname '*.mp4' -o -iname '*.mov' \)`;
- com `device_list_dir` em `~/Downloads`, `~/Desktop` e `~/Documents`.

Texto de cada pergunta: "… Se não estiver na lista, cola o caminho completo em Other (Mac: no Finder, botão direito → Option → 'Copiar … como nome de caminho'; Windows: botão direito → 'Copiar como caminho')." Sem candidatas, as opções são "Colo o caminho em Other" e "Procura em Downloads".

Espera pelas respostas antes de continuar.

Regras:
- NÃO usar vídeos anexados ao chat: o Berto só trabalha a partir de pastas no computador.
- Pedir acesso com UMA chamada `device_request_folder_access` com a pasta dos vídeos e a pasta onde está o logo (se for a mesma ou estiver dentro da outra, só a de cima), reason = "O Berto precisa destas pastas para ler os vídeos e o logo e guardar as combinações". Se recusado ou se o computador não estiver ligado a esta sessão, parar e explicar o que falta.
- Os caminhos com acentos (ex.: "GRÁVIDAS") podem vir numa forma Unicode diferente da do disco; o script resolve isso sozinho. Se mesmo assim não encontrar, listar a pasta-mãe com `device_list_dir` e mostrar o que existe.
- Os originais nunca são alterados. Só se escreve em `<pasta>/combinacoes/`.

## Passos

Todo o trabalho é feito no computador do utilizador com `device_bash` (os ficheiros finais são grandes: cerca de 40 Mbps, ou seja ~240 MB por combinação de 45 s, e transferi-los para a cloud e de volta seria muito lento). Pasta de trabalho na VM: `$HOME/berto` (fora de `mnt/`, invisível ao utilizador).

1. **Perguntar os caminhos** (regra acima, na janela de opções) e pedir acesso às pastas.
2. **Verificar ferramentas**: `which ffmpeg ffprobe python3` no `device_bash`. Se faltar o ffmpeg, tentar instalar (`pip install --break-system-packages imageio-ffmpeg` não chega para o ffprobe; tentar `apt-get`/`brew` se existir). Se não der, explicar e parar: fazer isto na cloud obrigaria a transferir gigas de vídeo.
3. **Escrever o script**: com `device_bash`, `mkdir -p $HOME/berto` e escrever o script abaixo em `$HOME/berto/combinar.py` com um heredoc com aspas (`cat > $HOME/berto/combinar.py <<'PYEOF' … PYEOF`). Começar sempre com cache limpa: `rm -rf $HOME/berto/cache` (a cache guarda os parâmetros do trabalho anterior).
4. **Analisar**: `python3 $HOME/berto/combinar.py scan "<pasta>" "<logo>"`. Dizer numa linha: quantos HOOKs, MEATs e CTAs, o logo (resolução, duração, se tem áudio) e o total de combinações.
   - Se a pasta tiver uma subpasta `legendados/` e os vídeos de fora não tiverem legendas, perguntar se é para usar a `legendados/`.
   - Se faltar algum tipo (0 meats, por exemplo), parar e avisar. Ficheiros que não começam por HOOK/MEAT/BODY/CTA: perguntar na janela, um por pergunta (até 4 por chamada), com as opções "Ignorar (Recommended)", "HOOK", "MEAT", "CTA".
   - Estimar o espaço: nº de combinações × (duração média hook + meat + CTA + logo) × bitrate ÷ 8. Ver o espaço livre com `df -h "<pasta>"`. Se não couber, ou se forem mais de 100 combinações, confirmar com o utilizador antes de continuar.
5. **Normalizar**: `python3 $HOME/berto/combinar.py prep "<pasta>" "<logo>" $HOME/berto/cache --budget 140` com `timeout_ms` 180000. É incremental: repetir a chamada até aparecer "prep completo".
6. **Juntar**: `python3 $HOME/berto/combinar.py combine "<pasta>" "<logo>" $HOME/berto/cache "<pasta>/combinacoes" --budget 140` com `timeout_ms` 180000, e repetir até aparecer "N/N combinações prontas". Cada combinação demora poucos segundos.
7. **Verificar**: com ffprobe, confirmar em 2–3 combinações que a duração ≈ soma das partes e que áudio e vídeo têm a mesma duração (diferença < 0,1 s). Correr `ffmpeg -v error -i <ficheiro> -f null -` numa delas para garantir que descodifica sem erros.
8. **Limpar e entregar**: `rm -rf $HOME/berto/cache`. Resposta final curta: quantas combinações, em que pasta, tamanho total, e que o `combinacoes.csv` tem a lista. Os ficheiros já estão no computador do utilizador, não é preciso enviá-los para o chat.

## Notas

- Para refazer só algumas (ex.: depois de corrigir um hook), apagar essas combinações em `combinacoes/` (pedir permissão de apagar com `device_request_delete_permission`) e o clip correspondente na cache, e correr `prep` + `combine` de novo: só gera o que falta.
- `--limit N` no combine gera só N combinações novas (útil para mostrar uma amostra antes do lote completo).
- **Nunca usar `-shortest`** em ffmpeg nestes vídeos: corta os últimos frames. Limitar com `-frames:v N` (imagem) ou `-t` (com `apad` no áudio), e em entradas `-loop 1` usar sempre `-t` para o ffmpeg não ficar a correr para sempre.
- **Acrescentar ou trocar um pedaço num vídeo já montado**: as combinações começam a imagem em pts 0,04 s. Manter os tempos originais (`settb=1/25000`, sem `setpts=PTS-STARTPTS` no primeiro segmento; `tpad=start=1:start_mode=clone` se houver um buraco no corte), senão a imagem fica 1 frame à frente do som. Confirmar com ffprobe que o `start_time` do vídeo e o n.º de frames são os esperados.
- Se o utilizador quiser outro vídeo no fim (ou nenhum), o argumento do logo é obrigatório no script: para "sem logo", pedir que indique um clip curto ou adaptar o script.

## Script `combinar.py`

```python
#!/usr/bin/env python3
"""Combinações HOOK x MEAT x CTA + vídeo final (logo).

Uso:
  python3 combinar.py scan    PASTA_VIDEOS FINAL                -> mostra o que encontrou e quantas combinações
  python3 combinar.py prep    PASTA_VIDEOS FINAL CACHE [--budget 150]
  python3 combinar.py combine PASTA_VIDEOS FINAL CACHE OUT [--budget 150] [--limit N]

- Tipo pelo nome: HOOK... = hook; MEAT.../BODY... = meat; CTA... = cta. Outros ficheiros são ignorados.
- prep: normaliza cada clip (e o vídeo final) para os MESMOS parâmetros (resolução, fps, H.264, AAC 48k
  estéreo, timescale), uma única vez, para depois juntar sem recodificar. Incremental: corre de novo até dizer "prep completo".
- combine: junta (stream copy, sem perda) HOOK + MEAT + CTA + FINAL para todas as combinações.
  Incremental: salta as que já existem. Escreve OUT/combinacoes.csv.
- Ordem: o hook varia primeiro, depois o meat, depois o CTA (H1-M1-C1, H2-M1-C1, ..., H1-M2-C1, ...).
"""
import json, os, re, subprocess, sys, time, csv, unicodedata

def resolve(path):
    """Encontra o caminho mesmo que os acentos venham noutra forma Unicode (macOS usa NFD)."""
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

EXT = (".mp4", ".mov", ".m4v")

def kind(name):
    n = os.path.basename(name).upper()
    if n.startswith("HOOK"): return "hook"
    if n.startswith(("MEAT", "BODY")): return "meat"
    if n.startswith("CTA"): return "cta"
    return None

def natkey(s): return [int(t) if t.isdigit() else t.lower() for t in re.split(r"(\d+)", s)]

def scan(vdir):
    g = {"hook": [], "meat": [], "cta": []}
    for f in sorted(os.listdir(vdir), key=natkey):
        p = os.path.join(vdir, f)
        if f.startswith(".") or not f.lower().endswith(EXT) or not os.path.isfile(p): continue
        k = kind(f)
        if k: g[k].append(p)
    return g

def probe(p):
    j = json.loads(subprocess.run(["ffprobe", "-v", "error", "-show_entries",
        "stream=codec_type,width,height,r_frame_rate,bit_rate,duration:format=duration,bit_rate", "-of", "json", p],
        capture_output=True, text=True).stdout)
    v = next((s for s in j["streams"] if s["codec_type"] == "video"), None)
    a = any(s["codec_type"] == "audio" for s in j["streams"])
    num, den = v["r_frame_rate"].split("/")
    br = v.get("bit_rate") or j["format"].get("bit_rate")
    return dict(w=int(v["width"]), h=int(v["height"]), fps=float(num) / float(den), audio=a,
                dur=float(j["format"]["duration"]), vdur=float(v.get("duration") or j["format"]["duration"]),
                br=int(br) if br and br != "N/A" else 0)

def short(p):
    b = os.path.splitext(os.path.basename(p))[0]
    return re.sub(r"_LEG$", "", b, flags=re.I)

def target(g):
    """Parâmetros comuns: resolução/fps do primeiro meat (ou hook); bitrate = maior dos clips (8–60 Mbps)."""
    ref = probe((g["meat"] or g["hook"] or g["cta"])[0])
    brs = [probe(p)["br"] for k in g for p in g[k]]
    fps = round(ref["fps"], 3)
    return dict(w=ref["w"], h=ref["h"], fps=fps, br=min(60_000_000, max(8_000_000, max(brs or [0]))))

def cache_name(p): return re.sub(r"[^\w.-]+", "_", short(p)) + ".mp4"

def normalize(src, dst, t):
    info = probe(src)
    W, H, fps, br = t["w"], t["h"], t["fps"], t["br"]
    vf = (f"scale={W}:{H}:force_original_aspect_ratio=decrease,pad={W}:{H}:(ow-iw)/2:(oh-ih)/2:color=black,"
          f"setsar=1,fps={fps},format=yuv420p")
    cmd = ["ffmpeg", "-y", "-v", "error", "-i", src]
    if not info["audio"]:
        cmd += ["-f", "lavfi", "-t", f"{info['vdur']:.3f}", "-i", "anullsrc=r=48000:cl=stereo"]
    cmd += ["-map", "0:v:0", "-map", "0:a:0" if info["audio"] else "1:a:0", "-vf", vf,
            "-c:v", "libx264", "-preset", "medium", "-profile:v", "high", "-level", "4.2",
            "-b:v", str(br), "-maxrate", str(int(br * 1.3)), "-bufsize", str(br * 2),
            "-g", str(int(round(fps * 2))), "-video_track_timescale", "90000",
            # sem -shortest (cortava os últimos frames quando o áudio era mais curto): o áudio é completado com
            # silêncio e cortado na duração exata da imagem, para não perder frames nem desacertar no concat
            "-af", "aresample=48000,aformat=channel_layouts=stereo,apad", "-c:a", "aac", "-b:a", "320k", "-ar", "48000",
            "-t", f"{info['vdur']:.3f}", "-movflags", "+faststart", dst + ".part.mp4"]
    subprocess.run(cmd, check=True)
    os.replace(dst + ".part.mp4", dst)

def prep(vdir, final, cache, budget):
    g = scan(vdir); os.makedirs(cache, exist_ok=True)
    tpath = os.path.join(cache, "target.json")
    if os.path.exists(tpath): t = json.load(open(tpath))
    else: t = target(g); json.dump(t, open(tpath, "w"))
    todo = [p for k in ("hook", "meat", "cta") for p in g[k]] + [final]
    t0 = time.time(); left = 0
    for p in todo:
        dst = os.path.join(cache, "FINAL.mp4" if p == final else cache_name(p))
        if os.path.exists(dst): continue
        if time.time() - t0 > budget: left += 1; continue
        normalize(p, dst, t); print("ok", os.path.basename(dst), flush=True)
    print("prep completo" if left == 0 else f"faltam {left} — correr prep outra vez")

def combine(vdir, final, cache, out, budget, limit=None):
    g = scan(vdir); os.makedirs(out, exist_ok=True)
    combos = [(h, m, c) for c in g["cta"] for m in g["meat"] for h in g["hook"]]
    tok = lambda p: short(p).split("_")[0]
    names = [f"{tok(h)}_{tok(m)}_{tok(c)}" for h, m, c in combos]
    if len(set(names)) != len(names):  # nomes curtos colidem -> nomes completos
        names = [f"{short(h)}__{short(m)}__{short(c)}" for h, m, c in combos]
    width = max(3, len(str(len(combos))))
    rows = []; t0 = time.time(); made = left = 0
    for i, ((h, m, c), n) in enumerate(zip(combos, names), 1):
        fn = f"{i:0{width}d}_{n}.mp4"; dst = os.path.join(out, fn)
        rows.append([fn, os.path.basename(h), os.path.basename(m), os.path.basename(c), os.path.basename(final)])
        if os.path.exists(dst) or (limit and made >= limit): continue
        if time.time() - t0 > budget: left += 1; continue
        parts = [os.path.join(cache, cache_name(x)) for x in (h, m, c)] + [os.path.join(cache, "FINAL.mp4")]
        for x in parts:
            if not os.path.exists(x): sys.exit(f"falta {x}: correr prep primeiro")
        lst = os.path.join(cache, "list.txt")
        with open(lst, "w") as f:
            for x in parts: f.write("file '" + x.replace("'", "'\\''") + "'\n")
        subprocess.run(["ffmpeg", "-y", "-v", "error", "-f", "concat", "-safe", "0", "-i", lst, "-c", "copy",
                        "-movflags", "+faststart", dst + ".part.mp4"], check=True)
        os.replace(dst + ".part.mp4", dst); made += 1; print("ok", fn, flush=True)
    with open(os.path.join(out, "combinacoes.csv"), "w", newline="") as f:
        w = csv.writer(f); w.writerow(["ficheiro", "hook", "meat", "cta", "final"]); w.writerows(rows)
    done = sum(os.path.exists(os.path.join(out, r[0])) for r in rows)
    print(f"{done}/{len(rows)} combinações prontas" + ("" if done == len(rows) else " — correr combine outra vez"))

if __name__ == "__main__":
    a = sys.argv[1:]
    if len(a) < 3: print(__doc__); sys.exit(1)
    opt = lambda k, d: type(d)(a[a.index(k) + 1]) if k in a else d
    cmd, vdir, final = a[0], resolve(a[1]), resolve(a[2])
    if not os.path.isdir(vdir): sys.exit(f"pasta não encontrada: {vdir}")
    if not os.path.isfile(final): sys.exit(f"vídeo final não encontrado: {final}")
    if cmd == "scan":
        g = scan(vdir); n = len(g["hook"]) * len(g["meat"]) * len(g["cta"])
        for k in g: print(k, len(g[k]), [os.path.basename(p) for p in g[k]])
        fi = probe(final); print("final", os.path.basename(final), f"{fi['w']}x{fi['h']} {fi['fps']:.2f}fps {fi['dur']:.1f}s áudio={fi['audio']}")
        print("combinações:", n)
    elif cmd == "prep": prep(vdir, final, resolve(a[3]), opt("--budget", 150))
    elif cmd == "combine":
        out = a[4]; parent = resolve(os.path.dirname(os.path.abspath(out)))
        out = os.path.join(parent, os.path.basename(os.path.abspath(out)))
        combine(vdir, final, resolve(a[3]), out, opt("--budget", 150), opt("--limit", 0) or None)
```