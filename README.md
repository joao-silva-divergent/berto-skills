# Berto — editor de vídeo da Divergent

Skills para o Claude que transformam uma pasta de vídeos brutos em anúncios prontos: legendas, tag do médico, combinações HOOK x MEAT x CTA com logo e música masterizada para redes sociais.

| Skill | O que faz |
|---|---|
| `/berto-captions` | Legenda em massa HOOKs, MEATs e CTAs. |
| `/berto-tags` | Põe a tag com o nome do médico e o n.º da ordem (OM/OMD) nos clips em que ele fala. |
| `/berto-combos` | Gera todas as combinações HOOK x MEAT x CTA e junta o logo da marca no fim. |
| `/berto-musica` | Põe música nas combinações, nivela as vozes e masteriza (−14 LUFS). |
| `/berto-master` | Faz tudo de uma vez, com as perguntas todas no início. |

O Berto pergunta sempre as pastas, fontes, cores, logo, músicas e médicos; não assume nada de nenhuma marca.

## Instalar

**Claude Code (terminal ou app):**

```
/plugin marketplace add joao-silva-divergent/berto-skills
/plugin install berto@berto
```

**Claude (app ou claude.ai):** em Definições → Capacidades → Skills, carrega cada pasta de `skills/` como `.zip`.

**À mão:** copia as pastas de `skills/` para `~/.claude/skills/`.

## Requisitos

`ffmpeg`, Python 3 e Node/npm (para descarregar o modelo de transcrição na primeira utilização).
