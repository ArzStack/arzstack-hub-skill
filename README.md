# ArzStack Hub — skill de onboarding do CLI

Skill (formato [Agent Skills](https://docs.claude.com/en/docs/agents-and-tools/agent-skills))
que ensina um agente de IA (Claude Code, Codex, Gemini) a usar o CLI
**`@arzstack/hub`** pra descobrir e instalar itens do catálogo da organização —
skills, MCP servers, knowledge e automations — em nome do usuário.

É a skill **padrão/global** do ArzStack Hub: distribuída pra todas as orgs, ela faz
o agente já "saber" navegar e instalar o resto do catálogo.

## Estrutura

```
.
├── SKILL.md      # a skill em si (frontmatter + instruções) — é o que o agente lê
└── README.md     # este arquivo
```

Ao instalar pelo Hub, o repositório é clonado pra
`~/.agents/skills/<slug>/` (global) ou `./.agents/skills/<slug>/` (projeto), e o
agente passa a enxergar o `SKILL.md` na raiz.

## Versionamento

- A branch `main` é a versão publicada.
- Releases marcam com **tags semver**: `v1.0.0`, `v1.1.0`, …
- O item no catálogo do Hub guarda a versão como metadado. O `git clone` do CLI
  hoje pega o HEAD da branch default; pra fixar uma tag específica é só evoluir o
  install pra `git clone --branch <tag>` (melhoria futura do CLI).

## Como publicar/atualizar

1. Edite o `SKILL.md`.
2. Commit + tag: `git tag v1.x.0 && git push --tags`.
3. No Hub (Admin Plataforma), o item global aponta pra este repo (`resource_url`)
   e é distribuído às orgs.

## Licença

MIT.
