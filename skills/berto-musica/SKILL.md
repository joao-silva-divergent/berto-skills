---
name: "berto-musica"
description: "Berto, o editor de vídeo da Divergent: põe música nas combinações HOOK x MEAT x CTA, nivela o som das vozes e masteriza para redes sociais. Usar com /berto-musica."
---

# Berto: música e nivelamento de som nas combinações

"Berto" é o editor de vídeo da Divergent. Quando chamarem esta skill, apresenta-te como Berto numa linha e segue os passos abaixo. Qualquer pessoa da equipa a pode usar, para qualquer marca. Não assumas pastas, músicas nem logo: pergunta SEMPRE.

Objetivo: pegar nas combinações feitas pelo /berto-combos (pasta `combinacoes/`) e, em cada vídeo:

- **Nivelar a voz clip a clip.** O hook, o meat e o CTA foram gravados em alturas diferentes, por isso cada um é medido e acertado para -16 LUFS. Depois leva um filtro passa-alto a 70 Hz e uma compressão leve. O logo é medido e acertado da mesma forma.
- **Música a nível fixo, bem por baixo da voz.** Fica 16 dB abaixo da voz. Leva também um corte suave nos 2,5 kHz, para deixar espaço às palavras.
- **Início limpo.** A música começa no primeiro som da faixa, sem o silêncio inicial, ou num segundo escolhido pelo utilizador (nesse caso entra com fade in de 0,4 s).
- **Subir no logo.** Quando começa a animação do logo, a música sobe até ficar 4 dB abaixo da voz (rampa de 0,8 s). Depois faz fade out e termina exatamente no último frame do vídeo.
- **Master final.** Fica a -14 LUFS e -1 dBTP (o padrão de Meta, TikTok e YouTube), em duas passagens e sem compressão extra. Áudio em AAC a 320 kbps e 48 kHz.
- **O vídeo não é recodificado** (stream copy), por isso não perde qualidade. O áudio é cortado à duração exata do vídeo.

Os resultados ficam numa pasta nova, `combinacoes_musica/`, ao lado de `combinacoes/`, com os mesmos nomes de ficheiro, mais um `musicas.csv` que diz que música foi para cada vídeo. Os originais nunca são alterados.

## REGRA OBRIGATÓRIA: perguntar TUDO na janela de opções (AskUserQuestion)

Todas as perguntas desta skill são feitas com **AskUserQuestion**: os caminhos, que música vai para onde, o ponto de início e a aprovação da amostra. Só se pergunta em texto se a ferramenta não existir. Valores próprios (caminhos, regras, segundos) escrevem-se no campo "Other"; diz isso no texto de cada pergunta.

Nunca terminar uma resposta com uma pergunta em texto livre. Qualquer aviso que precise de decisão vai para a janela, para não passar despercebido: falta do `combinacoes.csv` ou dos clips originais, clips muito baixos (< -40 LUFS), música mais curta que o vídeo (loop), vídeos sem música no mapa, erros na verificação.

**Exceção:** quando o /berto-master chama esta skill com as respostas já recolhidas (combinações, músicas, logo, regra das músicas), não voltar a perguntar o que já foi respondido. Se o master estiver em modo "Fazer tudo seguido", saltar a aprovação da amostra e fazer logo o lote completo com os valores por defeito.

No início de cada utilização (mesmo que já se tenha usado na conversa), faz UMA chamada com 3 perguntas:

1. **Combinações** (header "Combinações"): pastas `combinacoes` candidatas.
2. **Músicas** (header "Músicas"): pastas com ficheiros de áudio (.mp3/.wav/.m4a/.aiff) ou um ficheiro de música. Se for um ficheiro, usa a pasta dele e essa faixa.
3. **Logo** (header "Logo"): o vídeo da animação do logo usado nas combinações (serve para saber quanto dura o logo, que é onde a música sobe).

Em cada pergunta, até 4 opções: label curto, description = caminho completo (e n.º de vídeos, faixas ou duração). Como encontrar as candidatas, antes de perguntar:
- os caminhos usados nesta conversa (ex.: o logo do /berto-combos);
- nas pastas já ligadas, com `device_bash`:
  - `find $HOME/mnt -maxdepth 5 -type d -name 'combinacoes'`;
  - `find $HOME/mnt -maxdepth 4 \( -iname '*.mp3' -o -iname '*.wav' -o -iname '*.m4a' \)`;
  - `find $HOME/mnt -maxdepth 5 \( -iname '*logo*' -o -iname '*icon*' \) -iname '*.mp4'`;
- com `device_list_dir` em `~/Downloads`, `~/Desktop`, `~/Documents` e `~/Music`.

Texto de cada pergunta: "… Se não estiver na lista, cola o caminho completo em Other (Mac: no Finder, botão direito → Option → 'Copiar … como nome de caminho')." Sem candidatas, as opções são "Colo o caminho em Other" e "Procura em Downloads".

Espera pelas respostas antes de continuar.

Regras:
- NÃO uses ficheiros anexados ao chat: o Berto só trabalha a partir de pastas no computador.
- Pede acesso com UMA chamada `device_request_folder_access` com as pastas necessárias. Se uma estiver dentro de outra, pede só a de cima. Usa reason = "O Berto precisa destas pastas para ler os vídeos, as músicas e o logo e guardar os vídeos com música". Se o acesso for recusado ou o computador não estiver ligado a esta sessão, para e explica o que falta.
- Os caminhos com acentos podem vir noutra forma Unicode: o script resolve isso sozinho. Se mesmo assim não encontrar, lista a pasta-mãe com `device_list_dir`.
- Só escreves em `combinacoes_musica/`.

## Passos

Faz todo o trabalho no computador do utilizador com `device_bash`. Pasta de trabalho na VM: `$HOME/berto` (fora de `mnt/`).

1. **Perguntar os caminhos** (regra acima, na janela de opções) e pedir acesso às pastas.
2. **Verificar as ferramentas** com `which ffmpeg ffprobe python3`. Se faltar o ffmpeg, tenta instalá-lo (apt-get/brew). Se não der, explica e para.
3. **Escrever o script**: `mkdir -p $HOME/berto && rm -rf $HOME/berto/cache_musica` e depois escrever o script abaixo em `$HOME/berto/musica.py` com um heredoc com aspas (`cat > $HOME/berto/musica.py <<'PYEOF' … PYEOF`).
4. **Analisar**: `python3 $HOME/berto/musica.py scan "<combos>" "<músicas>" "<logo>"`. Mostra ao utilizador:
   - quantos vídeos há e se o `combinacoes.csv` e os clips originais foram encontrados. Sem eles, o nivelamento é feito ao vídeo inteiro e não clip a clip; avisa disto;
   - a duração do logo;
   - a **lista numerada das músicas**, com a duração de cada uma.
5. **Perguntar que música vai para onde** (janela de opções, header "Música"). Opções feitas a partir da lista de faixas:
   - **Uma só faixa:** "<faixa> em todos, desde o início (Recommended)" e "<faixa> em todos, a partir de outro segundo (escreve em Other)".
   - **Várias faixas:** até 4 opções, ex.: "<faixa 1> em todos", "<faixa 2> em todos", "Alternar todas as faixas", "Uma faixa por CTA".
   
   Texto da pergunta: "Que música vai para onde? Para outra regra, escreve em Other (ex.: 'CTA2 com a 1 e CTA3 com a 3', 'HOOK16 a HOOK20 com a 4', 'a 2 a partir dos 12 s')." Por defeito, a música começa no primeiro som. Se a resposta for ambígua, volta a perguntar na janela.
6. **Montar o mapa**:
   - `python3 $HOME/berto/musica.py template "<combos>" $HOME/berto/mapa_musica.csv`
   - depois uma ou mais chamadas `python3 $HOME/berto/musica.py assign $HOME/berto/mapa_musica.csv "<padrão>" "<faixa>[,<faixa2>…]" [início_s]`. O padrão é `*` para todos, um texto (ex.: `CTA2`, `HOOK16`) ou um glob sobre o nome do ficheiro (ex.: `00[1-5]_*`). Cada chamada sobrepõe a anterior, por isso vai do geral para o específico. Várias faixas separadas por vírgulas são alternadas.
   - Para regras que o `assign` não cobre (ex.: intervalos de HOOK16 a HOOK20), chama-o uma vez por clip ou edita o CSV com um pequeno python.
   - Mostra o resumo (quantos vídeos com cada música) e confirma que nenhum ficou sem música.
7. **Preparar**: `python3 $HOME/berto/musica.py prep "<combos>" "<músicas>" "<logo>" $HOME/berto/mapa_musica.csv $HOME/berto/cache_musica --budget 140` com `timeout_ms` 180000. Repete até aparecer "prep completo". Se algum clip medir abaixo de -40 LUFS (pode ser um clip sem som ou muito baixo; o ganho máximo aplicado é 24 dB), pergunta na janela: "Continuar na mesma (Recommended)" ou "Parar para ver esse clip".
8. **Amostra primeiro**: `python3 $HOME/berto/musica.py mix "<combos>" "<logo>" $HOME/berto/mapa_musica.csv $HOME/berto/cache_musica "<combos>/../combinacoes_musica" --limit 1`. Diz que vídeo ficou pronto e pergunta na janela de opções (header "Amostra"). Opções:
   - "Está bem: fazer todos (Recommended)";
   - "Música mais baixa" (`--gap` +3);
   - "Música mais alta" (`--gap` −3);
   - "Enviar-me a amostra para ouvir" (Other para outros pedidos). Se ele não estiver ao computador, faz staging desse vídeo e envia-o com SendUserFile. Se pedir mais ou menos música, ajusta `--gap`: maior baixa a música, menor sobe. O padrão é 16, e o intervalo razoável vai de 12 a 22. Para mudar a subida no logo, ajusta `--gap-logo` (padrão 4). Para refazer a amostra, apaga o ficheiro dela (com `device_request_delete_permission` se for preciso).
9. **Lote completo**: o mesmo comando sem `--limit` (e com os mesmos `--gap`/`--gap-logo` se tiverem sido mudados), `--budget 140`, `timeout_ms` 180000. Repete até aparecer "N/N vídeos com música". Se aparecer "AVISO … entra em loop", a música é mais curta que o vídeo: pergunta na janela — "Usar outra faixa", "Começar a música mais cedo" ou "Aceitar o loop" (Other para outra regra).
10. **Verificar** 2–3 vídeos com ffprobe e ffmpeg:
    - a duração do áudio é igual à do vídeo (diferença < 0,05 s);
    - `ffmpeg -i <f> -af loudnorm=print_format=json -f null -` dá input_i ≈ -14 (±1) e input_tp ≤ -1;
    - medindo o hook, o meat e o CTA em separado (`-ss`/`-t` com as durações dos clips), ficam a ±2 LU uns dos outros;
    - `ffmpeg -v error -i <f> -f null -` descodifica sem erros.
11. **Limpar e entregar**: `rm -rf $HOME/berto/cache_musica`. Resposta final curta: quantos vídeos, em que pasta, tamanho total, e que o `musicas.csv` tem a lista. Não envies os ficheiros para o chat a não ser que o utilizador peça.

## Notas

- Para refazer só alguns vídeos (outra música, por exemplo): muda o mapa com `assign`, apaga esses vídeos em `combinacoes_musica/` (pede permissão de apagar) e corre `prep` + `mix` de novo. Só é gerado o que falta.
- O nível da voz (`--voz -16`) e o master (`--lufs -14 --tp -1`) raramente precisam de mudar. Para TV/broadcast, usa `--lufs -23`.
- A música entra com loop se for mais curta que o vídeo. Evita-o, porque o ponto de loop pode notar-se.
- Se um vídeo for alterado depois de ter música (ex.: acrescentar meio segundo de fala no início de um clip), refazer a música desse vídeo a partir da combinação alterada, e não colar áudio por cima: o nivelamento é feito clip a clip e o pedaço novo tem de receber o ganho do clip a que pertence. Depois confirmar que o n.º de frames e o `start_time` da imagem são os esperados (a imagem não pode ficar à frente do som).

## Script `musica.py`

```python
#!/usr/bin/env python3
"""Música + nivelamento de som nas combinações do Berto.

Uso:
  python3 musica.py scan     COMBOS PASTA_MUSICAS LOGO
  python3 musica.py template COMBOS MAPA.csv
  python3 musica.py assign   MAPA.csv PADRAO FAIXA[,FAIXA2,...] [INICIO_s]
  python3 musica.py prep     COMBOS PASTA_MUSICAS LOGO MAPA.csv CACHE [--budget 140]
  python3 musica.py mix      COMBOS LOGO MAPA.csv CACHE OUT [--budget 140] [--limit N]
                             [--voz -16] [--gap 16] [--gap-logo 4] [--lufs -14] [--tp -1]

- COMBOS: pasta com as combinações e o combinacoes.csv (do /berto-combos). Os clips originais
  (HOOK/MEAT/CTA) têm de estar na pasta-mãe de COMBOS, como o /berto-combos os deixa.
- template: cria MAPA.csv (ficheiro, hook, meat, cta, musica, inicio) com a música em branco.
- assign: PADRAO = texto contido no nome do ficheiro ou nos nomes dos clips (ex.: CTA2, HOOK16), ou *
  para todos. Várias faixas separadas por vírgulas = alternadas pela ordem dos vídeos. Chamadas
  posteriores sobrepõem as anteriores. INICIO = segundo da faixa onde a música começa (opcional;
  sem início, salta o silêncio inicial da faixa).
- prep: mede o volume de cada clip original e do logo e prepara as faixas (48 kHz, EQ para abrir
  espaço à voz, normalizadas a -20 LUFS). Incremental.
- mix: para cada vídeo: voz nivelada clip a clip (hook/meat/cta/logo a --voz LUFS) + compressão leve;
  música a --gap dB abaixo da voz, sobe para --gap-logo dB abaixo durante o logo, fade in no início
  e fade out a terminar exatamente no fim do vídeo; master final a --lufs LUFS / --tp dBTP (2 passagens).
  O vídeo é copiado sem recodificar. Incremental.
"""
import csv, json, math, os, re, subprocess, sys, time, unicodedata, fnmatch

MUSIC_EXT = (".mp3", ".wav", ".m4a", ".aac", ".aif", ".aiff", ".flac", ".ogg", ".mp4", ".mov")
MUSIC_REF = -20.0  # LUFS das faixas preparadas

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

def vdur(p):
    """Duração da faixa de vídeo (a referência para cortar o áudio)."""
    out = subprocess.run(["ffprobe", "-v", "error", "-select_streams", "v:0", "-show_entries", "stream=duration",
                          "-of", "csv=p=0", p], capture_output=True, text=True).stdout.strip()
    try: return float(out)
    except ValueError: return dur(p)

def dur(p):
    out = subprocess.run(["ffprobe", "-v", "error", "-show_entries", "format=duration", "-of", "csv=p=0", p],
                         capture_output=True, text=True).stdout.strip()
    return float(out)

def has_audio(p):
    return bool(subprocess.run(["ffprobe", "-v", "error", "-select_streams", "a", "-show_entries", "stream=index",
                                "-of", "csv=p=0", p], capture_output=True, text=True).stdout.strip())

def loud(p, pre="", extra_in=()):
    """Mede com loudnorm. Devolve dict com input_i, input_tp, input_lra, input_thresh, target_offset."""
    af = (pre + "," if pre else "") + "loudnorm=I=-14:TP=-1:LRA=11:print_format=json"
    r = subprocess.run(["ffmpeg", "-hide_banner", "-nostats", *extra_in, "-i", p, "-vn", "-af", af, "-f", "null", "-"],
                       capture_output=True, text=True)
    m = re.findall(r"\{[^{}]*\}", r.stderr, re.S)
    if not m: raise RuntimeError(f"não consegui medir {p}: {r.stderr[-400:]}")
    j = json.loads(m[-1])
    return {k: float(v) if v not in ("-inf", "inf") else -99.0 for k, v in j.items() if k != "normalization_type"}

def gain_db(measured, target, lim=24.0):
    if measured < -60: return 0.0
    return max(-lim, min(lim, target - measured))

def amix_has_normalize():
    return "normalize" in subprocess.run(["ffmpeg", "-hide_banner", "-h", "filter=amix"], capture_output=True, text=True).stdout

def read_csv(p):
    with open(p, newline="") as f: return list(csv.DictReader(f))

def write_csv(p, rows, cols):
    with open(p, "w", newline="") as f:
        w = csv.DictWriter(f, fieldnames=cols); w.writeheader(); w.writerows(rows)

def musics(mdir):
    return sorted([f for f in os.listdir(mdir) if not f.startswith(".") and f.lower().endswith(MUSIC_EXT)
                   and os.path.isfile(os.path.join(mdir, f))], key=natkey)

def safe(name): return re.sub(r"[^\w.-]+", "_", os.path.splitext(name)[0])

# ---------- comandos ----------

def cmd_scan(combos, mdir, logo):
    vids = sorted([f for f in os.listdir(combos) if f.lower().endswith(".mp4") and not f.startswith(".")], key=natkey)
    print("vídeos:", len(vids))
    csvp = os.path.join(combos, "combinacoes.csv")
    if os.path.exists(csvp):
        rows = read_csv(csvp); parent = os.path.dirname(combos)
        miss = sorted({r[k] for r in rows for k in ("hook", "meat", "cta") if not os.path.exists(os.path.join(parent, r[k]))})
        print("combinacoes.csv: ok" + (f" — clips originais em falta na pasta-mãe: {miss}" if miss else " (clips originais encontrados)"))
    else:
        print("combinacoes.csv: NÃO existe — nivelamento por vídeo inteiro em vez de clip a clip")
    print(f"logo: {os.path.basename(logo)} {dur(logo):.2f}s áudio={has_audio(logo)}")
    if vids: print(f"duração dos vídeos: {min(dur(os.path.join(combos, v)) for v in vids[:50]):.1f}–{max(dur(os.path.join(combos, v)) for v in vids[:50]):.1f}s")
    print("músicas:")
    for i, m in enumerate(musics(mdir), 1):
        print(f"  {i}. {m}  ({dur(os.path.join(mdir, m)):.1f}s)")

def cmd_template(combos, mapa):
    csvp = os.path.join(combos, "combinacoes.csv")
    if os.path.exists(csvp):
        rows = [{"ficheiro": r["ficheiro"], "hook": r["hook"], "meat": r["meat"], "cta": r["cta"], "musica": "", "inicio": ""}
                for r in read_csv(csvp) if os.path.exists(os.path.join(combos, r["ficheiro"]))]
    else:
        rows = [{"ficheiro": f, "hook": "", "meat": "", "cta": "", "musica": "", "inicio": ""}
                for f in sorted(os.listdir(combos), key=natkey) if f.lower().endswith(".mp4") and not f.startswith(".")]
    write_csv(mapa, rows, ["ficheiro", "hook", "meat", "cta", "musica", "inicio"])
    print(f"{len(rows)} vídeos em {mapa}")

def cmd_assign(mapa, pat, tracks, inicio=""):
    rows = read_csv(mapa); tracks = [t.strip() for t in tracks.split(",") if t.strip()]
    n = 0
    for r in rows:
        hay = " ".join([r["ficheiro"], r["hook"], r["meat"], r["cta"]]).upper()
        ok = pat == "*" or (fnmatch.fnmatch(r["ficheiro"].upper(), pat.upper()) if any(c in pat for c in "*?[") else pat.upper() in hay)
        if ok:
            r["musica"] = tracks[n % len(tracks)]; r["inicio"] = inicio; n += 1
    write_csv(mapa, rows, ["ficheiro", "hook", "meat", "cta", "musica", "inicio"])
    from collections import Counter
    print(f"{n} vídeos atualizados. Resumo:", dict(Counter(r["musica"] or "(sem música)" for r in rows)))

def cmd_prep(combos, mdir, logo, mapa, cache, budget):
    os.makedirs(os.path.join(cache, "music"), exist_ok=True)
    rows = read_csv(mapa); parent = os.path.dirname(combos)
    sem = [r["ficheiro"] for r in rows if not r["musica"]]
    if sem: sys.exit(f"{len(sem)} vídeos sem música no mapa (ex.: {sem[:3]}). Usar assign primeiro.")
    jp = os.path.join(cache, "clips.json")
    clips = json.load(open(jp)) if os.path.exists(jp) else {}
    t0 = time.time(); left = 0
    todo = sorted({os.path.join(parent, r[k]) for r in rows for k in ("hook", "meat", "cta") if r[k]
                   and os.path.exists(os.path.join(parent, r[k]))}, key=natkey) + [logo]
    for p in todo:
        key = "__LOGO__" if p == logo else os.path.basename(p)
        if key in clips: continue
        if time.time() - t0 > budget: left += 1; continue
        m = loud(p) if has_audio(p) else {"input_i": -99.0}
        clips[key] = {"dur": dur(p), "lufs": m["input_i"]}
        json.dump(clips, open(jp, "w"), indent=1); print("medido", key, f"{m['input_i']:.1f} LUFS", flush=True)
    mp = os.path.join(cache, "music.json")
    mus = json.load(open(mp)) if os.path.exists(mp) else {}
    for name in sorted({t for r in rows for t in [r["musica"]]}, key=natkey):
        if name in mus: continue
        src = os.path.join(mdir, name)
        if not os.path.exists(src): sys.exit(f"música não encontrada: {src}")
        if time.time() - t0 > budget: left += 1; continue
        tmp = os.path.join(cache, "music", safe(name) + ".tmp.wav")
        subprocess.run(["ffmpeg", "-y", "-v", "error", "-i", src, "-vn", "-af",
                        "aresample=48000,aformat=channel_layouts=stereo,equalizer=f=2500:t=o:w=1.5:g=-3",
                        "-c:a", "pcm_f32le", tmp], check=True)
        m = loud(tmp); g = gain_db(m["input_i"], MUSIC_REF, 30)
        dst = os.path.join(cache, "music", safe(name) + ".wav")
        subprocess.run(["ffmpeg", "-y", "-v", "error", "-i", tmp, "-af", f"volume={g:.2f}dB", "-c:a", "pcm_f32le", dst], check=True)
        os.remove(tmp)
        r = subprocess.run(["ffmpeg", "-hide_banner", "-nostats", "-i", dst, "-af", "silencedetect=n=-45dB:d=0.05",
                            "-f", "null", "-"], capture_output=True, text=True).stderr
        lead = 0.0
        s = re.findall(r"silence_start: ([\d.\-e]+)", r); e = re.findall(r"silence_end: ([\d.]+)", r)
        if s and e and abs(float(s[0])) < 0.01: lead = float(e[0])
        mus[name] = {"wav": dst, "dur": dur(dst), "lead": round(lead, 3), "lufs_orig": m["input_i"]}
        json.dump(mus, open(mp, "w"), indent=1)
        print("música", name, f"{mus[name]['dur']:.1f}s, silêncio inicial {lead:.2f}s", flush=True)
    print("prep completo" if left == 0 else f"faltam {left} — correr prep outra vez")

def cmd_mix(combos, logo, mapa, cache, out, budget, limit, voz, gap, gap_logo, lufs, tp):
    os.makedirs(out, exist_ok=True)
    rows = read_csv(mapa)
    clips = json.load(open(os.path.join(cache, "clips.json")))
    mus = json.load(open(os.path.join(cache, "music.json")))
    logo_d = clips["__LOGO__"]["dur"]
    norm = amix_has_normalize()
    t0 = time.time(); made = left = 0; avisos = []
    for r in rows:
        src = os.path.join(combos, r["ficheiro"]); dst = os.path.join(out, r["ficheiro"])
        if os.path.exists(dst) or (limit and made >= limit): continue
        if time.time() - t0 > budget: left += 1; continue
        D = vdur(src); LS = max(0.0, D - logo_d)
        # --- ganhos da voz, clip a clip ---
        lin = lambda db: 10 ** (db / 20)
        if all(r[k] in clips for k in ("hook", "meat", "cta")):
            b1 = clips[r["hook"]]["dur"]; b2 = b1 + clips[r["meat"]]["dur"]
            g = [gain_db(clips[r[k]]["lufs"], voz) for k in ("hook", "meat", "cta")]
            gexpr = f"if(lt(t,{b1:.3f}),{lin(g[0]):.5f},if(lt(t,{b2:.3f}),{lin(g[1]):.5f},if(lt(t,{LS:.3f}),{lin(g[2]):.5f},{{GL}})))"
        else:  # sem csv: um ganho para a parte falada
            m = loud(src, extra_in=["-t", f"{LS:.3f}"])
            gexpr = f"if(lt(t,{LS:.3f}),{lin(gain_db(m['input_i'], voz)):.5f},{{GL}})"
        gexpr = gexpr.replace("{GL}", f"{lin(gain_db(clips['__LOGO__']['lufs'], voz)):.5f}")
        # --- música ---
        mu = mus[r["musica"]]
        start = float(r["inicio"]) if r.get("inicio") else mu["lead"]
        if mu["dur"] - start < D: avisos.append(f"{r['ficheiro']}: música mais curta que o vídeo, entra em loop")
        fin = 0.4 if r.get("inicio") else 0.05
        fout = min(1.5, logo_d * 0.5)
        gb = lin(voz - gap - MUSIC_REF); gl = lin(voz - gap_logo - MUSIC_REF)
        rs = max(0.0, LS - 0.2)
        mexpr = f"{gb:.5f}+({gl:.5f}-{gb:.5f})*min(max((t-{rs:.3f})/0.8,0),1)"
        mixf = "amix=inputs=2:duration=first:normalize=0" if norm else "amix=inputs=2:duration=first,volume=2"
        fc = (f"[0:a]aresample=48000,aformat=channel_layouts=stereo,volume='{gexpr}':eval=frame,highpass=f=70,"
              f"acompressor=threshold=0.125:ratio=2.5:attack=10:release=200,apad=whole_dur={D:.3f}[v];"
              f"[1:a]atrim=start={start:.3f}:duration={D:.3f},asetpts=PTS-STARTPTS,volume='{mexpr}':eval=frame,"
              f"afade=t=in:d={fin},afade=t=out:st={D - fout:.3f}:d={fout:.3f}[m];"
              f"[v][m]{mixf},atrim=duration={D:.3f}[a]")
        tmp = os.path.join(cache, "mix.wav")
        subprocess.run(["ffmpeg", "-y", "-v", "error", "-i", src, "-stream_loop", "-1", "-i", mu["wav"],
                        "-filter_complex", fc, "-map", "[a]", "-c:a", "pcm_f32le", tmp], check=True)
        m = loud(tmp)
        ln = (f"loudnorm=I={lufs}:TP={tp}:LRA=11:measured_I={m['input_i']}:measured_TP={m['input_tp']}:"
              f"measured_LRA={m['input_lra']}:measured_thresh={m['input_thresh']}:offset={m['target_offset']}:linear=true,"
              f"aresample=48000")
        subprocess.run(["ffmpeg", "-y", "-v", "error", "-i", src, "-i", tmp, "-map", "0:v:0", "-map", "1:a:0",
                        "-c:v", "copy", "-af", ln, "-t", f"{D:.3f}", "-c:a", "aac", "-b:a", "320k", "-ar", "48000",
                        "-movflags", "+faststart", dst + ".part.mp4"], check=True)
        os.replace(dst + ".part.mp4", dst); made += 1; print("ok", r["ficheiro"], "←", r["musica"], flush=True)
    write_csv(os.path.join(out, "musicas.csv"), rows, ["ficheiro", "hook", "meat", "cta", "musica", "inicio"])
    for a in sorted(set(avisos)): print("AVISO", a)
    done = sum(os.path.exists(os.path.join(out, r["ficheiro"])) for r in rows)
    print(f"{done}/{len(rows)} vídeos com música" + ("" if done == len(rows) else " — correr mix outra vez"))

if __name__ == "__main__":
    a = sys.argv[1:]
    if not a: print(__doc__); sys.exit(1)
    opt = lambda k, d: type(d)(a[a.index(k) + 1]) if k in a else d
    c = a[0]
    if c == "scan": cmd_scan(resolve(a[1]), resolve(a[2]), resolve(a[3]))
    elif c == "template": cmd_template(resolve(a[1]), a[2])
    elif c == "assign": cmd_assign(a[1], a[2], a[3], a[4] if len(a) > 4 and not a[4].startswith("--") else "")
    elif c == "prep": cmd_prep(resolve(a[1]), resolve(a[2]), resolve(a[3]), a[4], a[5], opt("--budget", 140.0))
    elif c == "mix":
        out = a[5]; parent = resolve(os.path.dirname(os.path.abspath(out)))
        out = os.path.join(parent, os.path.basename(os.path.abspath(out)))
        cmd_mix(resolve(a[1]), resolve(a[2]), a[3], a[4], out, opt("--budget", 140.0), opt("--limit", 0) or None,
                opt("--voz", -16.0), opt("--gap", 16.0), opt("--gap-logo", 4.0), opt("--lufs", -14.0), opt("--tp", -1.0))
    else: print(__doc__); sys.exit(1)
```