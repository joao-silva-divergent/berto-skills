---
name: "berto-captions"
description: "Berto, o editor de vídeo da Divergent: legenda em massa vídeos de anúncios (HOOK, MEAT, CTA) de qualquer marca, a partir de uma pasta no computador do utilizador. Usar com /berto-captions."
---

# Berto — legendagem em massa de HOOKs, MEATs e CTAs

"Berto" é o nome do editor de vídeo da Divergent. Quando esta skill é chamada, apresenta-te como Berto numa linha e segue os passos. A skill pode ser usada por qualquer pessoa da equipa e para marcas diferentes: não assumir nada sobre quem a usa — a pasta, a fonte e a cor de destaque são SEMPRE perguntadas ao utilizador.

Objetivo: pegar em muitos vídeos verticais (1080x1920) de uma pasta no computador do utilizador e devolver cada um com legenda gravada (`<nome>_LEG.mp4`) numa subpasta `legendados/` dentro dessa mesma pasta.

A **fonte** escolhida é usada em TODAS as legendas (hook, meat e CTA); muda só o estilo:
- **HOOK** (nome começa por `HOOK`): legenda animada: blocos de 2–3 linhas empilhadas e coladas (nunca 4 de uma vez); linhas brancas no 2.º peso mais pesado da fonte, com gradiente branco→cinza e sombra; a expressão-chave no peso mais pesado, na **cor de destaque**, maior, com glow largo na cor (pouca sombra escura), reflexo de luz no topo das letras e 2–3 pontos de luz suaves e difusos recortados DENTRO das letras (nunca estrelas, nunca pontos brancos fora do traço); tudo derivado automaticamente da cor. Cada linha entra quando é dita: sobe ~70px, desfocada → nítida em 0,32s.
- **MEAT** / **CTA** (nome começa por `MEAT`/`BODY` ou `CTA`): legenda simples branca com sombra, peso ~600 da fonte, 90px (encolhe sozinha se a frase não couber), 2–4 palavras de cada vez, centrada a y≈1290.
- **Qualidade igual ao original**: o vídeo é recodificado em H.264 com o MESMO bitrate da fonte (mínimo 8 Mbps), áudio AAC 320k. Diferença invisível (SSIM ≈ 0,997) e tamanho de ficheiro semelhante ao original. Não baixar o bitrate.
- Volume do áudio normalizado a -14 LUFS em cada ficheiro.

## REGRA OBRIGATÓRIA — perguntar TUDO na janela de opções (AskUserQuestion)

Todas as perguntas desta skill são feitas com **AskUserQuestion**, para nada passar despercebido: fonte, cor, pasta, tipo dos ficheiros sem prefixo, vídeos já legendados, fonte inexistente, dúvidas de transcrição e qualquer outra decisão. Nunca terminar uma resposta com uma pergunta em texto livre (só se a ferramenta não existir). Valores próprios (um caminho, um hex, a frase certa) escrevem-se no campo "Other"; diz isso no texto de cada pergunta.

**Exceção:** quando o /berto-master chama esta skill com as respostas já recolhidas, não voltar a perguntar o que já foi respondido.

No INÍCIO de cada utilização (mesmo que já se tenha usado na conversa), UMA chamada com 3 perguntas:

1. **Fonte** (header "Fonte"): Roboto, Montserrat, Poppins, Inter. Texto: "Que Google Font para as legendas? Podes escrever outra em Other."
2. **Cor** (header "Cor"): Magenta `#F01EF0`, Azul `#00B4FF`, Amarelo `#FFD400`. Texto: "Que cor de destaque nos hooks? Outra cor (hex ou nome) em Other." Se vier um nome de cor ("verde lima", "cor da marca"), converter para hex e dizer qual ficou; se for ambíguo, voltar a perguntar na janela.
3. **Pasta dos vídeos** (header "Pasta"): até 4 candidatas; label curto, description = caminho completo e quantos HOOK/MEAT/CTA tem. Candidatas, por ordem: os caminhos usados nesta conversa; nas pastas já ligadas, `find $HOME/mnt -maxdepth 5 -iname 'HOOK*' \( -iname '*.mp4' -o -iname '*.mov' \) -exec dirname {} \; | sort -u` (excluir `legendados*` e `combinacoes*`); `device_list_dir` em `~/Downloads`, `~/Desktop` e `~/Documents`. Texto: "Que pasta tem os vídeos? Se não estiver na lista, cola o caminho completo em Other (Mac: no Finder, botão direito na pasta → Option → 'Copiar … como nome de caminho'; Windows: botão direito → 'Copiar como caminho')." Sem candidatas, as opções são "Colo o caminho em Other" e "Procura em Downloads" (nesse caso, lista `~/Downloads` e volta a perguntar com o que encontrares).

Espera pelas respostas antes de continuar.

Outras perguntas que vão SEMPRE para a janela (nunca em texto, nunca só no resumo final):
- **Ficheiros sem prefixo** HOOK/MEAT/CTA: uma pergunta por ficheiro (até 4 por chamada), header = início do nome, opções "HOOK", "MEAT", "CTA", "Ignorar", com a sugestão pela duração (hooks < 8 s, CTAs < 12 s) marcada como (Recommended).
- **Vídeos que parecem já ter legendas gravadas**: "Saltar (Recommended)" ou "Legendar na mesma".
- **Fonte que não existe no Google Fonts**: voltar a perguntar a fonte.
- **Dúvidas de transcrição** (passo 6, ANTES do render): uma pergunta por dúvida (até 4 por chamada; repetir chamadas até acabar). Header = clip (ex.: "HOOK21"); pergunta = a frase como ficou + o que o modelo ouviu; opções = a correção escolhida (Recommended) e 1–2 alternativas plausíveis; a frase certa escreve-se em Other.

Regras da pasta:
- NÃO processar vídeos anexados ao chat. Se o utilizador anexar, responder que o Berto só trabalha a partir de uma pasta e pedir o caminho.
- Confirmar que a pasta existe com `device_list_dir`; se não estiver ligada, pedir acesso com `device_request_folder_access` (paths = [esse caminho], reason = "O Berto precisa desta pasta para ler os vídeos e guardar os legendados"). Se o caminho não existir, mostrar o que foi recebido e pedir para confirmar. Se recusado ou se o computador não estiver ligado a esta sessão, parar e explicar o que falta (app Claude aberta no computador e tarefa ligada ao computador). Nunca usar outra pasta sem perguntar.
- Ler e escrever apenas dentro dessa pasta: originais nunca são alterados; resultados vão para `<pasta>/legendados/`.

Regras da fonte: tem de existir no Google Fonts (o script descarrega `@fontsource/<nome>` via npm). Se o render disser que não existe, pedir outra ao utilizador. Fontes condensadas (Oswald, Bebas Neue) também funcionam.

## Passos

1. **Perguntar fonte, cor e pasta** (regra acima) e listar os vídeos (.mp4/.mov/.m4v). Mostrar numa linha: quantos HOOKs, MEATs e CTAs. Ficheiros que não começam por HOOK/MEAT/CTA: perguntar o tipo na janela de opções (regra acima). Saltar vídeos que já tenham `_LEG` no nome ou já existam em `legendados/`.
2. **Preparar** (uma vez por sessão): escrever o script abaixo em `legendar.py` no diretório de trabalho e correr `python3 legendar.py setup work`. O setup instala dependências e descarrega o modelo Whisper small via npm — se o Hugging Face estiver bloqueado no ambiente, NÃO tentar downloads de modelos do HF.
3. **Trazer os vídeos**: `device_stage_files` em lotes (máx. 50 ficheiros e ~500MB por chamada; vídeos de alto bitrate são grandes, por isso usar lotes pequenos; se falhar por tempo, um ficheiro grande de cada vez). Ficheiros acima de ~150MB dão timeout: no computador do utilizador, `mkdir -p _berto_tmp && split -b 45m <video> _berto_tmp/<nome>.part.`, fazer stage das partes (2 por chamada), juntar com `cat` e confirmar o sha256 com o original (`shasum -a 256` no Mac). No fim, avisar que a pasta `_berto_tmp` pode ser apagada. Criar symlinks dos ficheiros staged numa pasta `in/`; usar glob (`ls -d /mnt/user-data/uploads/*`) para a pasta, porque nomes com acentos podem vir noutra normalização Unicode.
4. **Transcrever + plano automático**: `python3 legendar.py plan work in` → `work/plan.json` (incremental).
5. **Rever o plano (passo do Berto — obrigatório)**. Editar `work/plan.json` com um script Python (não reescrever à mão):
   - Corrigir o texto para português de Portugal: "criámos", "connosco", "em baixo", concordância verbal ("Carregue… e preencha"), nomes de marcas e cidades. Manter o que foi dito; corrigir apenas ortografia e erros evidentes do modelo.
   - **Singular/plural e pessoa do verbo**: o modelo acrescenta muitas vezes o "-m" do plural ("Se vivem em Braga e querem engravidar") quando a pessoa disse singular ("Se vive em Braga e quer engravidar"), sobretudo em hooks dirigidos ao espectador ("para si", "você"). Ouvir o áudio nessas palavras e manter a concordância com quem está a ser tratado; se houver dúvida, vai para a janela no passo 6.
   - **HOOKS — escolher os destaques**: em cada bloco, a linha `"style": "M"` é a expressão mais forte e curta (1–2 palavras): números/idades ("35 Anos", "6 Meses"), cidade/público ("de Braga"), o problema ou o produto ("Medicação", "Engravidar"). Primeira letra maiúscula. Linhas `W` curtas (≤ 20 caracteres), minúsculas, sem ponto final. Um bloco = uma ideia, com **2 ou 3 linhas, nunca 4** (se a ideia não couber, partir em dois blocos). Num bloco de 3 linhas, nenhuma linha branca pode ter só uma palavra pequena ("com", "tem", "chega", "estão"): juntá-la à linha vizinha. Só o destaque (M) pode ficar sozinho numa linha. Otimizar sempre: preferir blocos de 2 linhas equilibradas.
   - **Aspas**: usar sempre aspas retas "assim". Nunca « », “ ” ou „ (o render converte, mas o plano deve vir certo).
   - `start` de cada linha = tempo da primeira palavra dessa linha; `end` do bloco = início do bloco seguinte (o último = duração do vídeo).
   - **MEAT/CTA**: os `chunks` devem cortar em pontuação/frases naturais e não deixar uma palavra sozinha.
   - Exemplo de hook bem feito:
     ```json
     [{"end":0.98,"lines":[{"text":"Para mulheres","style":"W","start":0.0},{"text":"de Braga","style":"M","start":0.5}]},
      {"end":2.92,"lines":[{"text":"que estão a fazer","style":"W","start":0.98},{"text":"Medicação","style":"M","start":1.52},{"text":"para emagrecer","style":"W","start":2.08}]}]
     ```
   - Não mostrar o plano inteiro. Anotar as dúvidas reais (palavras que o modelo pode ter ouvido mal, nomes próprios, género de palavras como "sozinho/sozinha", singular/plural como "vive/vivem" e "quer/querem") para as perguntar na janela no passo 6.
6. **Verificar as legendas (obrigatório, antes do render)**: `python3 legendar.py lint work`. Deteta blocos com 4+ linhas, palavras pequenas sozinhas, destaques sem maiúscula, linhas brancas longas ou com ponto final, aspas erradas, espaços a mais, tempos fora de ordem ou sobrepostos, palavras sozinhas no meat/CTA e brasileirismos (bebê, conosco, embaixo, equipe…). Corrigir o plano até dar **0 erros**. Depois, reler o texto completo de cada vídeo (bloco a bloco) à procura de erros de ortografia, concordância e palavras mal ouvidas. O render recusa avançar com erros de lint (só com `--force`, se o utilizador pedir). A seguir, perguntar TODAS as dúvidas de transcrição na janela (regra acima), aplicar as respostas ao plano e correr o lint outra vez. Só se faz o render com 0 erros e 0 dúvidas por responder.
7. **Render**: `python3 legendar.py render work out --font "<fonte>" --color "<hex>"` (paralelo; `--jobs N`; `--only A.mp4,B.mp4` para refazer só alguns).
8. **Verificar o resultado**: extrair 2–3 frames de uma amostra de hooks e de um meat (`ffmpeg -vf select=...,tile`) e olhar para eles antes de entregar (legibilidade da cor sobre o fundo, texto a caber no ecrã, linhas sem se cortarem, pontos de luz só dentro das letras). Confirmar com ffprobe que o bitrate de saída é semelhante ao da fonte. Só entregar quando lint e frames estiverem bem.
9. **Entregar**: copiar cada resultado para `/mnt/user-data/outputs/`, depois `device_commit_files` com stagedPath para `<pasta>/legendados/<nome>_LEG.mp4` (lotes de ~5 ficheiros, UMA chamada de cada vez, nunca em paralelo). Se algum vier rejeitado com "HTTP 400 fetching session file", fazer `SendUserFile` desses ficheiros (display attach) e repetir o commit com o `fileUuid`. Se a escrita falhar, dizer quais ficheiros ficaram só no chat e porquê. Resposta final curta: quantos vídeos, em que pasta e fonte/cor usadas. As dúvidas já foram resolvidas na janela; se surgir alguma nova, pergunta-se na janela, nunca só no texto final.

## Notas

- Vídeos que já tenham legendas gravadas ficam com legendas duplicadas — avisar.
- O modelo (Whisper small) erra sobretudo terminações verbais (singular/plural) e nomes próprios; rever sempre esses pontos. Transcrever outra vez com `word_timestamps=True` e um `initial_prompt` com a frase certa ajuda a confirmar o que foi dito e os tempos de cada palavra.
- **Perguntas escritas no ecrã** (ex.: a pergunta da entrevista "O que pesquisou antes de vir à clínica?" antes de uma resposta): não usar o estilo do hook, que faz parecer um título. Usar um cartão próprio que se leia como pergunta: cartão branco com cantos arredondados, etiqueta "PERGUNTA" pequena na cor de destaque, texto escuro no peso médio da fonte, no terço de cima, com fade curto de entrada e saída. Mostrar uma amostra e perguntar na janela antes de aplicar a todos.
- Se a resolução não for 1080x1920, o script escala tudo proporcionalmente.
- Fontes com métricas grandes (ex.: Montserrat) aparecem um pouco mais pequenas no meat/CTA; se o utilizador pedir maior, ajustar `BODY_SIZE`.
- Sem compressão nenhuma não é possível: gravar legendas obriga a recodificar. Se o utilizador pedir ficheiros para edição posterior, a alternativa é ProRes 422 HQ (.mov, ficheiros ~4x maiores).

## Script `legendar.py`

```python
#!/usr/bin/env python3
"""Legendagem em massa de HOOKs, MEATs e CTAs (Berto).

Uso:
  python3 legendar.py setup   WORK
  python3 legendar.py plan    WORK  VIDEO_DIR      -> WORK/plan.json (transcreve + plano automático)
  python3 legendar.py lint    WORK                 -> verifica erros nas legendas (tem de dar 0)
  python3 legendar.py render  WORK  OUT_DIR [--color "#F01EF0"] [--font "Roboto"] [--only nome1,nome2] [--jobs N] [--force]

Tipo de vídeo pelo nome do ficheiro: começa por HOOK -> hook; MEAT/BODY -> meat; CTA -> cta.
"""
import json, os, re, subprocess, sys, glob, shutil, math
from concurrent.futures import ProcessPoolExecutor

# ---------------------------------------------------------------- estilo
REF_W = 1080
HOOK_CY = 1240            # centro vertical do bloco do hook (em 1080x1920)
BODY_Y = 1290             # posição das legendas do meat/cta
BODY_SIZE = 90
W_SIZE, M_SIZE = 90, 156  # linhas brancas / destaque do hook
MAX_LINE_W = 960
STOP = set("a o as os e é de da do das dos que em no na nos nas um uma para por com se ou ao à há mais já não eu tu ele ela nós vós eles elas me te lhe isto isso este esta esse essa como mas quando está estão ser ter tem".split())
FIX = {"criamos": "criámos", "embaixo": "em baixo", "conosco": "connosco"}

def run(cmd, **kw):
    return subprocess.run(cmd, check=True, **kw)

# ---------------------------------------------------------------- cor / fonte
def hexrgb(h):
    h = h.strip().lstrip("#"); return tuple(int(h[i:i + 2], 16) for i in (0, 2, 4))
def mix(c, d, k): return tuple(int(c[i] + (d[i] - c[i]) * k) for i in range(3))

def hook_fonts(work, family):
    """Descarrega a Google Font (via npm @fontsource) e gera W.ttf (2.º peso mais pesado), M.ttf (mais pesado) e B.ttf (~600, para meat/cta)."""
    from fontTools.ttLib import TTFont
    slug = re.sub(r"[^a-z0-9]+", "-", family.lower()).strip("-")
    d = os.path.join(work, "fonts", slug); os.makedirs(d, exist_ok=True)
    done = [f for f in os.listdir(d) if f.endswith(".ttf")]
    if len(done) < 3:
        tmp = os.path.join(work, "_dl_" + slug); os.makedirs(tmp, exist_ok=True)
        r = subprocess.run(["npm", "pack", "--silent", f"@fontsource/{slug}"], cwd=tmp, capture_output=True, text=True)
        if r.returncode != 0: raise SystemExit(f"Fonte '{family}' não encontrada no Google Fonts (@fontsource/{slug}).")
        tgz = r.stdout.strip().splitlines()[-1]; run(["tar", "xzf", tgz], cwd=tmp)
        files = os.path.join(tmp, "package", "files")
        ws = sorted({int(m.group(1)) for f in os.listdir(files) if (m := re.match(rf"{slug}-latin-(\d+)-normal\.woff2$", f))})
        if not ws: raise SystemExit(f"Fonte '{family}' sem ficheiros latin normais.")
        heavy = ws[-1]; second = ([w for w in ws if w < heavy and w >= 600] or [heavy])[-1]
        body = min(ws, key=lambda w: (abs(w - 600), -w))
        for w, name in [(second, "W.ttf"), (heavy, "M.ttf"), (body, "B.ttf")]:
            f = TTFont(os.path.join(files, f"{slug}-latin-{w}-normal.woff2")); f.flavor = None
            if name == "B.ttf":   # nome único para o libass encontrar sem ambiguidade
                for rec in f["name"].names:
                    if rec.nameID in (1, 4, 6, 16): rec.string = "BertoBody" + slug.replace("-", "")
                    elif rec.nameID in (2, 17): rec.string = "Regular"
            f.save(os.path.join(d, name))
        shutil.rmtree(tmp, ignore_errors=True)
    return os.path.join(d, "W.ttf"), os.path.join(d, "M.ttf")

def body_font(work, family):
    hook_fonts(work, family)
    slug = re.sub(r"[^a-z0-9]+", "-", family.lower()).strip("-")
    return os.path.join(work, "fonts", slug), "BertoBody" + slug.replace("-", ""), os.path.join(work, "fonts", slug, "B.ttf")

# ---------------------------------------------------------------- setup
def setup(work):
    os.makedirs(work, exist_ok=True)
    run([sys.executable, "-m", "pip", "install", "-q", "--break-system-packages",
         "faster-whisper", "fonttools", "brotli", "pillow", "numpy"])
    tmp = os.path.join(work, "_dl"); os.makedirs(tmp, exist_ok=True)
    def npm(pkg):
        out = subprocess.run(["npm", "pack", "--silent", pkg], cwd=tmp, check=True,
                             capture_output=True, text=True).stdout.strip().splitlines()[-1]
        d = os.path.join(tmp, out[:-4]); os.makedirs(d, exist_ok=True)
        run(["tar", "xzf", out, "-C", d], cwd=tmp); return os.path.join(d, "package")
    model = os.path.join(work, "model")
    if not os.path.exists(os.path.join(model, "model.bin")):
        os.makedirs(model, exist_ok=True)
        parts = [npm(f"@skillninja/shortmovie-telop-model-p{i}") for i in range(4)]
        for f in ["config.json", "preprocessor_config.json", "tokenizer.json", "vocabulary.json"]:
            shutil.copy(os.path.join(parts[0], f), model)
        with open(os.path.join(model, "model.bin"), "wb") as out:
            for i, p in enumerate(parts):
                with open(os.path.join(p, f"model.bin.part{i}"), "rb") as fh: shutil.copyfileobj(fh, out)
        want = open(os.path.join(parts[0], "model.sha256")).read().split()[0]
        got = subprocess.run(["sha256sum", os.path.join(model, "model.bin")], capture_output=True, text=True).stdout.split()[0]
        assert want == got, "checksum do modelo não confere"
    shutil.rmtree(tmp, ignore_errors=True)
    print("setup ok")

# ---------------------------------------------------------------- utils
def kind_of(name):
    n = os.path.basename(name).upper()
    if n.startswith("HOOK"): return "hook"
    if n.startswith("CTA"): return "cta"
    return "meat"

def probe(path):
    j = json.loads(subprocess.run(["ffprobe", "-v", "error", "-select_streams", "v:0", "-show_entries",
        "stream=width,height,r_frame_rate:format=duration", "-of", "json", path], capture_output=True, text=True).stdout)
    s = j["streams"][0]; a, b = s["r_frame_rate"].split("/")
    return dict(w=int(s["width"]), h=int(s["height"]), fps=float(a) / float(b), dur=float(j["format"]["duration"]))

def src_bitrate(path):
    r = subprocess.run(["ffprobe", "-v", "error", "-select_streams", "v:0", "-show_entries", "stream=bit_rate:format=bit_rate",
                        "-of", "json", path], capture_output=True, text=True).stdout
    j = json.loads(r); b = j.get("streams", [{}])[0].get("bit_rate") or j.get("format", {}).get("bit_rate")
    return max(8_000_000, int(b)) if b and b != "N/A" else 40_000_000

def loudness(path):
    e = subprocess.run(["ffmpeg", "-hide_banner", "-i", path, "-af", "ebur128", "-f", "null", "-"],
                       capture_output=True, text=True).stderr
    m = re.findall(r"I:\s+(-?[\d.]+) LUFS", e)
    return float(m[-1]) if m else -14.0

def audio16k(path):
    import numpy as np
    raw = subprocess.run(["ffmpeg", "-v", "error", "-i", path, "-f", "s16le", "-ac", "1", "-ar", "16000", "-"],
                         capture_output=True, check=True).stdout
    return np.frombuffer(raw, np.int16).astype(np.float32) / 32768

# ---------------------------------------------------------------- plano automático
def body_chunks(words, maxw=4, maxc=24):
    out, cur = [], []
    for s, e, w in words:
        cur.append((s, e, w))
        txt = " ".join(x[2] for x in cur)
        if re.search(r"[.,?!:;]$", w) or len(cur) >= maxw or len(txt) >= maxc:
            out.append(cur); cur = []
    if cur: out.append(cur)
    merged = []
    for c in out:
        if merged and len(c) == 1 and not re.search(r"[.,?!:;]$", merged[-1][-1][2]) and \
           len(" ".join(x[2] for x in merged[-1] + c)) <= maxc + 6:
            merged[-1] += c
        else: merged.append(c)
    return merged

def keyword_index(ws):
    best, score = 0, -1
    for i, (_, _, w) in enumerate(ws):
        t = re.sub(r"[^\wÀ-ÿ]", "", w.lower())
        sc = 100 if re.match(r"^\d", t) else (0 if t in STOP else len(t))
        if sc > score: best, score = i, sc
    return best

def hook_blocks(words, dur):
    blocks, cur = [], []
    for wd in words:
        cur.append(wd)
        if re.search(r"[.,?!:;]$", wd[2]) or len(cur) >= 6:
            blocks.append(cur); cur = []
    if cur:
        if blocks and len(cur) <= 2: blocks[-1] += cur
        else: blocks.append(cur)
    plan = []
    for bi, b in enumerate(blocks):
        k = keyword_index(b)
        if re.match(r"^\d", b[k][2]) and k + 1 < len(b): kw = b[k:k + 2]; after = b[k + 2:]
        else: kw = [b[k]]; after = b[k + 1:]
        groups = [("W", b[:k]), ("M", kw), ("W", after)]
        lines = []
        for sty, g in groups:
            if not g: continue
            txt = " ".join(x[2] for x in g).rstrip(".,;:")
            if sty == "M": txt = txt[0].upper() + txt[1:]
            lines.append({"text": txt, "style": sty, "start": round(g[0][0], 2)})
        end = blocks[bi + 1][0][0] if bi + 1 < len(blocks) else dur
        plan.append({"end": round(end, 2), "lines": lines})
    return plan

def make_plan(work, vdir):
    from faster_whisper import WhisperModel
    m = WhisperModel(os.path.join(work, "model"), device="cpu", compute_type="int8")
    files = sorted(f for f in glob.glob(os.path.join(vdir, "*")) if f.lower().endswith((".mp4", ".mov", ".m4v")))
    plan_path = os.path.join(work, "plan.json")
    plan = json.load(open(plan_path)) if os.path.exists(plan_path) else {}
    for f in files:
        name = os.path.basename(f)
        if name in plan: continue
        segs, _ = m.transcribe(audio16k(f), language="pt", beam_size=5, word_timestamps=True)
        words = [(x.start, x.end, FIX.get(x.word.strip(), x.word.strip())) for s in segs for x in s.words]
        info = probe(f); k = kind_of(name)
        entry = {"path": os.path.abspath(f), "type": k, "dur": round(info["dur"], 2),
                 "text": " ".join(w[2] for w in words)}
        if k == "hook": entry["blocks"] = hook_blocks(words, info["dur"])
        else:
            cs = body_chunks(words); chunks = []
            for i, c in enumerate(cs):
                st, en = c[0][0], c[-1][1]
                if i + 1 < len(cs) and cs[i + 1][0][0] - en < 0.7: en = cs[i + 1][0][0]
                else: en = min(en + 0.3, info["dur"])
                chunks.append({"text": " ".join(x[2] for x in c), "start": round(st, 2), "end": round(en, 2)})
            entry["chunks"] = chunks
        plan[name] = entry
        json.dump(plan, open(plan_path, "w"), ensure_ascii=False, indent=1)
        print(f"[{k}] {name}: {entry['text']}", flush=True)
    print(f"plano: {plan_path} ({len(plan)} vídeos)")

# ---------------------------------------------------------------- verificação das legendas (lint)
QUOTES = re.compile(r"[«»“”„‟″]")
def Q(t): return QUOTES.sub('"', t)
PTBR = {"bebê": "bebé", "conosco": "connosco", "embaixo": "em baixo", "registro": "registo", "equipe": "equipa",
        "ônibus": "autocarro", "celular": "telemóvel", "academia": "ginásio", "tela": "ecrã", "criamos": "criámos (se for passado)"}

def lint(work):
    plan = json.load(open(os.path.join(work, "plan.json"))); errs = []
    for n, e in plan.items():
        texts = []
        if e["type"] == "hook":
            prev = -1.0; bstart = 0.0
            for bi, b in enumerate(e["blocks"]):
                L = b["lines"]; tag = f"{n} bloco {bi + 1}"
                if len(L) > 3: errs.append(f"{tag}: {len(L)} linhas de uma vez (máx. 3)")
                if len(L) == 3:
                    for l in L:
                        if l["style"] == "W" and len(l["text"].split()) == 1 and len(l["text"]) <= 6:
                            errs.append(f"{tag}: linha só com '{l['text']}' num bloco de 3 linhas")
                if not any(l["style"] == "M" for l in L): errs.append(f"{tag}: sem destaque (M)")
                for l in L:
                    t = l["text"]; texts.append(t)
                    if l["start"] < prev - 1e-6: errs.append(f"{tag}: '{t}' começa antes da linha anterior")
                    if l["start"] >= b["end"]: errs.append(f"{tag}: '{t}' começa depois do fim do bloco")
                    prev = l["start"]
                    if l["style"] == "W":
                        if len(t) > 20: errs.append(f"{tag}: linha branca longa ({len(t)} car.): '{t}'")
                        if t.rstrip().endswith("."): errs.append(f"{tag}: ponto final em '{t}'")
                    elif not re.search(r"(^|\s)[A-ZÀ-Ý0-9]", t): errs.append(f"{tag}: destaque sem maiúscula: '{t}'")
                if b["end"] <= bstart: errs.append(f"{tag}: fim do bloco antes do início")
                bstart = b["end"]
            if abs(e["blocks"][-1]["end"] - e["dur"]) > 0.3: errs.append(f"{n}: último bloco não acaba na duração do vídeo")
        else:
            pe = 0.0
            for i, c in enumerate(e["chunks"]):
                t = c["text"]; texts.append(t)
                if len(t.split()) == 1 and len(t.strip(".,!?:;")) <= 4: errs.append(f"{n} legenda {i + 1}: palavra sozinha '{t}'")
                if c["end"] <= c["start"]: errs.append(f"{n} legenda {i + 1}: fim antes do início")
                if c["start"] < pe - 0.01: errs.append(f"{n} legenda {i + 1}: sobrepõe-se à anterior")
                pe = c["end"]
        for t in texts:
            if QUOTES.search(t): errs.append(f"{n}: aspas erradas em '{t}' (usar aspas retas)")
            if "  " in t or t != t.strip(): errs.append(f"{n}: espaços a mais em '{t}'")
            for w in re.findall(r"[\wÀ-ÿ]+", t.lower()):
                if w in PTBR: errs.append(f"{n}: '{w}' -> {PTBR[w]} (português de Portugal)")
    for x in errs: print("ERRO", x)
    print(f"lint: {len(errs)} erro(s) em {len(plan)} vídeos")
    return errs

# ---------------------------------------------------------------- render hook (PIL)
def hook_frames(entry, work, info, style):
    from PIL import Image, ImageDraw, ImageFont, ImageFilter, ImageChops
    sc = info["w"] / REF_W
    PAD = int(60 * sc)
    fw, fm = hook_fonts(work, style["font"]); base = hexrgb(style["color"])
    C_TOP, C_BOT, C_GLOW, C_EXT = mix(base, (255, 255, 255), .3), mix(base, (0, 0, 0), .08), mix(base, (255, 255, 255), .15), mix(base, (0, 0, 0), .7)
    def font(sty, size): return ImageFont.truetype(fw if sty == "W" else fm, int(size * sc))
    def mask(txt, f):
        l, t, r, b = f.getbbox(txt); m = Image.new("L", (r - l + 2 * PAD, b - t + 2 * PAD), 0)
        ImageDraw.Draw(m).text((PAD - l, PAD - t), txt, font=f, fill=255); return m
    def grad(size, top, bot, y0, y1):
        w, h = size; g = Image.new("RGB", size); d = ImageDraw.Draw(g)
        for y in range(h):
            k = min(1, max(0, (y - y0) / max(1, y1 - y0)))
            d.line([(0, y), (w, y)], fill=tuple(int(top[i] + (bot[i] - top[i]) * k) for i in range(3)))
        return g
    def layer(col, m, a=1.0):
        im = Image.new("RGBA", m.size, col); im.putalpha(m.point(lambda v: int(v * a))); return im
    def shift(m, dx, dy):
        o = Image.new("L", m.size, 0); o.paste(m, (dx, dy)); return o
    def sprite(txt, sty):
        size = W_SIZE if sty == "W" else M_SIZE
        f = font(sty, size)
        while f.getlength(txt) > MAX_LINE_W * sc and size > 40: size -= 4; f = font(sty, size)
        m = mask(txt, f); bb = m.getbbox(); out = Image.new("RGBA", m.size, (0, 0, 0, 0))
        if sty == "W":
            out = Image.alpha_composite(out, layer((0, 0, 0, 255), shift(m, 0, int(5 * sc)).filter(ImageFilter.GaussianBlur(5 * sc)), 0.55))
            g = grad(m.size, (255, 255, 255), (150, 150, 150), bb[1] + int(10 * sc), bb[3]).convert("RGBA")
        else:
            # glow largo + glow interior (mais luz, menos sombra preta)
            out = Image.alpha_composite(out, layer(C_GLOW + (255,), m.filter(ImageFilter.MaxFilter(11)).filter(ImageFilter.GaussianBlur(26 * sc)), 0.85))
            out = Image.alpha_composite(out, layer(mix(base, (0, 0, 0), .2) + (255,), m.filter(ImageFilter.MaxFilter(5)).filter(ImageFilter.GaussianBlur(5 * sc)), 0.55))
            ext = Image.new("L", m.size, 0)
            for i in range(1, int(4 * sc)): ext = ImageChops.lighter(ext, shift(m, i, i + 1))
            out = Image.alpha_composite(out, layer(mix(base, (0, 0, 0), .35) + (255,), ext.filter(ImageFilter.GaussianBlur(1.2)), 0.6))
            g = grad(m.size, C_TOP, C_BOT, bb[1], bb[3]).convert("RGBA")
            g.putalpha(m); out = Image.alpha_composite(out, g)
            # reflexo de luz no topo das letras
            h = bb[3] - bb[1]; hl = Image.new("L", m.size, 0); dh = ImageDraw.Draw(hl)
            for y in range(bb[1], bb[1] + int(h * .45)):
                dh.line([(0, y), (m.width, y)], fill=int(120 * (1 - (y - bb[1]) / (h * .45)) ** 1.5))
            out = Image.alpha_composite(out, layer((255, 255, 255, 255), ImageChops.multiply(hl, m.filter(ImageFilter.MinFilter(3)))))
            # pontos de luz DENTRO das letras: no centro do primeiro traço a partir de cima, recortados pela letra
            sp = Image.new("RGBA", m.size, (0, 0, 0, 0)); px = m.load()
            for fx in ([.18, .62, .9] if len(txt) > 5 else [.3, .8]):
                x = int(bb[0] + (bb[2] - bb[0]) * fx); y0 = None
                for y in range(bb[1], bb[1] + int(h * .6)):
                    if px[x, y] > 200 and y0 is None: y0 = y
                    elif px[x, y] <= 200 and y0 is not None: break
                if y0 is None: continue
                sp = Image.alpha_composite(sp, spot(m.size, x, (y0 + y) // 2, sc * (1.15 if fx > .5 else .9)))
            clip = m.filter(ImageFilter.MinFilter(3)).filter(ImageFilter.GaussianBlur(1))
            sp.putalpha(ImageChops.multiply(sp.getchannel("A"), clip))
            return out, sp
        g.putalpha(m); return Image.alpha_composite(out, g), None
    def spot(size, cx, cy, s):   # ponto de luz suave e difuso (sem raios)
        L = Image.new("L", size, 0); d = ImageDraw.Draw(L); r = int(13 * s)
        d.ellipse([cx - r, cy - r, cx + r, cy + r], fill=255)
        core = L.filter(ImageFilter.GaussianBlur(10 * s)); halo = L.filter(ImageFilter.MaxFilter(9)).filter(ImageFilter.GaussianBlur(32 * s))
        return Image.alpha_composite(layer(C_GLOW + (255,), halo, 0.85), layer((255, 255, 255, 255), core, 0.7))
    STEP = {("W", "W"): 80, ("W", "M"): 108, ("M", "W"): 78, ("M", "M"): 120}   # W->M 108: com 72 a linha branca encostava ao destaque
    blocks = []; prev_end = 0.0
    for b in entry["blocks"]:
        ys = [0]
        for a, c in zip(b["lines"], b["lines"][1:]): ys.append(ys[-1] + STEP[(a["style"], c["style"])])
        off = HOOK_CY - (ys[0] + ys[-1]) / 2
        start = b["lines"][0]["start"] if not blocks else prev_end
        blocks.append((start, b["end"], [(l["start"], sprite(Q(l["text"]), l["style"]), int((y + off) * sc)) for l, y in zip(b["lines"], ys)]))
        prev_end = b["end"]
    W, H, fps = info["w"], info["h"], info["fps"]
    n = int(math.ceil(info["dur"] * fps)); D = 0.32
    for i in range(n):
        t = i / fps; c = Image.new("RGBA", (W, H), (0, 0, 0, 0))
        for bs, be, ls in blocks:
            if not (bs <= t < be): continue
            for st, (sp, spk), y in ls:
                if t < st: continue
                e = 1 - (1 - min(1, (t - st) / D)) ** 3
                s = sp.filter(ImageFilter.GaussianBlur(18 * sc * (1 - e))) if e < 0.98 else sp
                if e < 1:
                    s = s.copy(); s.putalpha(s.getchannel("A").point(lambda v: int(v * min(1, e * 1.3))))
                pos = (W // 2 - s.width // 2, y + int(70 * sc * (1 - e)) - s.height // 2)
                c.alpha_composite(s, pos)
                if spk is not None and t - st > D * 0.6:   # pontos de luz acendem logo a seguir à entrada da linha
                    k = min(1, (t - st - D * .6) / 0.15)
                    a = spk.copy(); a.putalpha(a.getchannel("A").point(lambda v: int(v * k)))
                    c.alpha_composite(a, pos)
        yield c.tobytes()

# ---------------------------------------------------------------- render meat/cta (libass)
def ts(t): t = max(0, t); return f"{int(t // 3600)}:{int(t % 3600 // 60):02}:{t % 60:05.2f}"

def body_ass(entry, info, path, style, work):
    from PIL import ImageFont
    sc = info["w"] / REF_W
    fdir, fname, fttf = body_font(work, style["font"])
    size = int(BODY_SIZE * sc)
    hdr = f"""[Script Info]
ScriptType: v4.00+
PlayResX: {info['w']}
PlayResY: {info['h']}
WrapStyle: 0

[V4+ Styles]
Format: Name, Fontname, Fontsize, PrimaryColour, SecondaryColour, OutlineColour, BackColour, Bold, Italic, Underline, StrikeOut, ScaleX, ScaleY, Spacing, Angle, BorderStyle, Outline, Shadow, Alignment, MarginL, MarginR, MarginV, Encoding
Style: Body,{fname},{size},&H00FFFFFF,&H00FFFFFF,&H90000000,&H80000000,0,0,0,0,100,100,0,0,1,{3 * sc:.1f},{3.5 * sc:.1f},5,50,50,0,1

[Events]
Format: Layer, Start, End, Style, Name, MarginL, MarginR, MarginV, Effect, Text
"""
    y = int(BODY_Y * info["h"] / 1920)
    ev = []
    for c in entry["chunks"]:
        fs = size; f = ImageFont.truetype(fttf, fs)
        while f.getlength(c["text"]) > MAX_LINE_W * sc and fs > 40: fs -= 2; f = ImageFont.truetype(fttf, fs)
        tag = f"\\pos({info['w'] // 2},{y})" + (f"\\fs{fs}" if fs != size else "")
        ev.append(f"Dialogue: 0,{ts(c['start'])},{ts(c['end'])},Body,,0,0,0,,{{{tag}}}{Q(c['text'])}")
    open(path, "w").write(hdr + "\n".join(ev) + "\n")

def render_one(args):
    name, entry, work, outdir, style = args
    src = entry["path"]; info = probe(src)
    out = os.path.join(outdir, os.path.splitext(name)[0] + "_LEG.mp4")
    gain = max(-12, min(12, -14 - loudness(src)))
    afil = f"volume={gain:.1f}dB,alimiter=limit=0.95"
    vbr = src_bitrate(src)   # mesma qualidade do original: igualar o bitrate de vídeo da fonte
    enc = ["-c:v", "libx264", "-preset", "medium", "-b:v", str(vbr), "-maxrate", str(int(vbr * 1.3)),
           "-bufsize", str(vbr * 2), "-pix_fmt", "yuv420p",
           "-c:a", "aac", "-b:a", "320k", "-movflags", "+faststart", out]
    if entry["type"] == "hook":
        cmd = ["ffmpeg", "-y", "-v", "error", "-i", src, "-f", "rawvideo", "-pix_fmt", "rgba",
               "-s", f"{info['w']}x{info['h']}", "-r", str(info["fps"]), "-i", "-",
               "-filter_complex", f"[0:v][1:v]overlay=0:0:eof_action=pass[v];[0:a]{afil}[a]",
               "-map", "[v]", "-map", "[a]"] + enc
        p = subprocess.Popen(cmd, stdin=subprocess.PIPE)
        for fr in hook_frames(entry, work, info, style): p.stdin.write(fr)
        p.stdin.close(); assert p.wait() == 0, name
    else:
        ass = os.path.join(work, "ass", os.path.splitext(name)[0] + ".ass")
        os.makedirs(os.path.dirname(ass), exist_ok=True); body_ass(entry, info, ass, style, work)
        run(["ffmpeg", "-y", "-v", "error", "-i", src, "-vf",
             f"ass={ass}:fontsdir={body_font(work, style['font'])[0]}", "-af", afil] + enc)
    return out

def render(work, outdir, only=None, jobs=None, color="#F01EF0", font="Roboto", force=False):
    if lint(work) and not force: raise SystemExit("Há erros nas legendas: corrigir o plan.json (ou usar --force).")
    plan = json.load(open(os.path.join(work, "plan.json")))
    os.makedirs(outdir, exist_ok=True)
    style = {"color": color, "font": font}; hexrgb(color); hook_fonts(work, font)  # valida e descarrega antes de paralelizar
    items = [(n, e, work, outdir, style) for n, e in plan.items() if not only or n in only]
    jobs = jobs or max(1, (os.cpu_count() or 2) // 2)
    with ProcessPoolExecutor(jobs) as ex:
        for out in ex.map(render_one, items): print("ok", out, flush=True)

if __name__ == "__main__":
    a = sys.argv[1:]
    if len(a) > 1: a[1] = os.path.abspath(a[1])
    if len(a) > 2: a[2] = os.path.abspath(a[2])
    if not a: print(__doc__); sys.exit(1)
    if a[0] == "setup": setup(a[1])
    elif a[0] == "plan": make_plan(a[1], a[2])
    elif a[0] == "lint": sys.exit(1 if lint(a[1]) else 0)
    elif a[0] == "render":
        only = None; jobs = None
        if "--only" in a: only = set(a[a.index("--only") + 1].split(","))
        if "--jobs" in a: jobs = int(a[a.index("--jobs") + 1])
        color = a[a.index("--color") + 1] if "--color" in a else "#F01EF0"
        font = a[a.index("--font") + 1] if "--font" in a else "Roboto"
        render(a[1], a[2], only, jobs, color, font, "--force" in a)
```