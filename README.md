# ArzStack Hub — Skill

Ensina o seu agente de IA (Claude Code, Codex, Gemini, e afins) a **descobrir e
instalar itens do catálogo do seu ArzStack Hub** — skills, MCP servers, knowledge
e automations — usando o CLI `@arzstack/hub`, em vez de você ficar copiando
comando da web.

Com ela instalada, é só pedir em linguagem natural ("instala a skill de resumo do
nosso hub", "quais MCPs a gente tem?", "me configura o catálogo da org") que o
agente faz o login, navega o catálogo e instala o item certo pra você.

É uma **Agent Skill** padrão (uma pasta com um `SKILL.md`) — instala igual a
qualquer outra skill.

---

## Instalação

### Opção A — pelo próprio Hub (recomendado)

Se você já usa o `@arzstack/hub`, instale esta skill como qualquer item do
catálogo (ela é global/padrão da plataforma):

```bash
npx @arzstack/hub install      # escolha "ArzStack Hub" na lista → global
```

### Opção B — manual (clone na pasta de skills do seu agente)

A skill é só esta pasta com o `SKILL.md`. Clone ela no diretório de skills do seu
agente e reinicie/recarregue o agente:

| Agente | Pasta de skills | Comando |
|---|---|---|
| **Claude Code** (global) | `~/.claude/skills/` | `git clone https://github.com/ArzStack/arzstack-hub-skill ~/.claude/skills/arzstack-hub` |
| **Claude Code** (projeto) | `.claude/skills/` | `git clone https://github.com/ArzStack/arzstack-hub-skill .claude/skills/arzstack-hub` |
| **Codex / Gemini** (convenção `~/.agents/skills`) | `~/.agents/skills/` | `git clone https://github.com/ArzStack/arzstack-hub-skill ~/.agents/skills/arzstack-hub` |
| **Outro agente** | pasta de skills do agente | clone esta pasta lá dentro |

> Não tem `git`? Baixe o ZIP (botão **Code → Download ZIP**) e extraia a pasta no
> diretório de skills correspondente.

Depois de clonar, **reinicie o agente** pra ele carregar a skill.

---

## Usando

Não tem comando especial — depois de instalada, é conversa normal. A skill dispara
quando você pede algo relacionado ao catálogo, por exemplo:

- "instala a skill de resumo de chamados do nosso hub"
- "quais MCP servers a minha org tem disponível?"
- "me configura aqui o que tem no ArzStack pra esse projeto"

> Por baixo, o agente usa o **modo interativo** do CLI (`npx @arzstack/hub install`
> sem id, ou `browse`): lista os itens da sua org com contagem por tipo, navega
> paginado, e instala em **projeto** ou **global** — sem você copiar id nem comando.

Na primeira vez, o agente vai pedir pra você **logar**: crie um token em
**Perfil → Tokens de CLI** no portal do Hub e rode `npx @arzstack/hub login`.

---

## Atualizando

```bash
git -C <pasta-da-skill> pull        # ex.: ~/.claude/skills/arzstack-hub
```

Cada release é marcada com tag semver (`v1.0.0`, `v1.1.0`, …).

## Licença

[MIT](./LICENSE).
