# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## O que é este repositório

Marketplace pessoal de skills para o Claude Code. Cada skill é um plugin independente com prefixo `jvg-` para identificação rápida em projetos com múltiplos marketplaces instalados.

## Estrutura de um plugin

```
plugins/jvg-<nome>/
├── .claude-plugin/
│   └── plugin.json        ← manifesto local (name + description)
└── skills/
    └── jvg-<nome>/
        └── SKILL.md       ← prompt da skill (frontmatter name: + corpo)
```

O índice do marketplace está em `.claude-plugin/marketplace.json` na raiz.

## Regras ao adicionar ou renomear uma skill

Toda skill deve ter prefixo `jvg-` em **quatro lugares**:
1. Nome do diretório do plugin: `plugins/jvg-<nome>/`
2. Nome do diretório da skill: `skills/jvg-<nome>/`
3. Campo `name` em `.claude-plugin/plugin.json`
4. Campo `name` no frontmatter do `SKILL.md`

E também em `.claude-plugin/marketplace.json` nos campos `name` e `source` da entrada correspondente.

## Fluxo para adicionar uma skill nova

1. Criar `plugins/jvg-<nome>/.claude-plugin/plugin.json` com `name` e `description`
2. Criar `plugins/jvg-<nome>/skills/jvg-<nome>/SKILL.md` com frontmatter `name:` e `description:`
3. Adicionar entrada em `.claude-plugin/marketplace.json`
4. `git commit && git push`
5. `/plugin marketplace update demolidor-de-tokens` para aplicar na máquina

## Instalar o marketplace localmente

```bash
claude plugin marketplace add joao-vitor-girardi/marketplace-demolidor-de-tokens --scope user
```

## Skills disponíveis

| Skill | Categoria |
|---|---|
| `jvg-analyze-codebase` | documentação |
| `jvg-refine-context` | documentação |
| `jvg-merge-from-master` | git |
| `jvg-to-issues` | planejamento |
| `jvg-engenheiro-paranoico` | arquitetura |
| `jvg-ze-diagnostico` | planejamento |
