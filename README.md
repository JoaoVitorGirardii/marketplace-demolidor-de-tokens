# demolidor-de-tokens

Skills para o Claude Code desenvolvidas e refinadas por mim ao longo do tempo. Cada skill é um plugin independente que pode ser instalado individualmente no seu ambiente.

## Skills disponíveis

| Plugin | Descrição |
|--------|-----------|
| `analyze-codebase` | Analisa o código-fonte e gera um `CONTEXT.md` otimizado para o Claude Code — regras, contratos e padrões que guiam o modelo em sessões futuras |
| `refine-context` | Atualiza cirurgicamente um `CONTEXT.md` existente: faz perguntas antes de escrever, nunca presume intenção |
| `merge-from-master` | Traz as alterações do `master`/`main` para a branch atual via merge; aborta automaticamente se houver conflitos |
| `conversa-com-engenheiro` | Conduz entrevista técnica iterativa como Staff Engineer especializado em NestJS, TypeScript, Vue.js, TypeORM, Sequelize e microsserviços — desafia planos, encontra riscos ocultos e produz documento de implementação estruturado |

## Como instalar

### Pré-requisito

Você precisa do [Claude Code](https://claude.ai/code) instalado (`npm i -g @anthropic-ai/claude-code`).

### 1. Clone o repositório

```bash
git clone https://github.com/JoaoVitorGirardii/marketplace-demolidor-de-tokens.git
cd marketplace-demolidor-de-tokens
```

### 2. Instale os plugins que quiser

Instale um plugin específico apontando para o diretório dele:

```bash
claude plugin install ./plugins/analyze-codebase
claude plugin install ./plugins/refine-context
claude plugin install ./plugins/merge-from-master
claude plugin install ./plugins/conversa-com-engenheiro
```

Ou instale todos de uma vez:

```bash
for plugin in plugins/*/; do claude plugin install "$plugin"; done
```

### 3. Use as skills no Claude Code

Após instalar, as skills ficam disponíveis como slash commands em qualquer projeto:

```
/analyze-codebase
/refine-context
/merge-from-master
/conversa-com-engenheiro
```

### Atualizando

Para puxar versões mais recentes das skills:

```bash
git pull
claude plugin install ./plugins/<nome-do-plugin>
```
