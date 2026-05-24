# demolidor-de-tokens

Skills para o Claude Code desenvolvidas e refinadas por mim ao longo do tempo. Cada skill é um plugin independente que pode ser instalado individualmente no seu ambiente.

## Skills disponíveis

| Plugin | Descrição |
|--------|-----------|
| `jvg-gerar-context` | Analisa o código-fonte e gera um `CONTEXT.md` otimizado para o Claude Code — regras, contratos e padrões que guiam o modelo em sessões futuras |
| `jvg-refinar-context` | Atualiza cirurgicamente um `CONTEXT.md` existente: faz perguntas antes de escrever, nunca presume intenção |
| `jvg-merge-from-master` | Traz as alterações do `master`/`main` para a branch atual via merge; aborta automaticamente se houver conflitos |
| `jvg-planejar-feature` | Conduz entrevista técnica iterativa para validar planos de implementação em NestJS, TypeScript, Vue.js, TypeORM, Sequelize e microsserviços — desafia planos, encontra riscos ocultos e produz documento de implementação estruturado |
| `jvg-investigar-dor` | Investiga dores de usuário antes de abrir qualquer issue — conduz conversa para entender o problema e impacto real, analisa o código em background, verifica duplicidade no GitHub Issues e gera ticket estruturado com User Story + Critérios de Aceite, sempre com aprovação do usuário |
| `jvg-investigar-problema` | Analisa um problema técnico em profundidade antes de propor qualquer solução — explora código, dependências, banco e performance, propõe opções com tradeoffs e só gera documento técnico após aprovação explícita |
| `jvg-quebrar-em-issues` | Recebe uma análise técnica pronta e a transforma em issues vertical-slice publicadas no GitHub — decide contratos, migrações, rollout e observabilidade, e publica somente após aprovação explícita |

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
claude plugin install ./plugins/jvg-gerar-context
claude plugin install ./plugins/jvg-refinar-context
claude plugin install ./plugins/jvg-merge-from-master
claude plugin install ./plugins/jvg-planejar-feature
claude plugin install ./plugins/jvg-investigar-dor
claude plugin install ./plugins/jvg-investigar-problema
claude plugin install ./plugins/jvg-quebrar-em-issues
```

Ou instale todos de uma vez:

```bash
for plugin in plugins/*/; do claude plugin install "$plugin"; done
```

### 3. Use as skills no Claude Code

Após instalar, as skills ficam disponíveis como slash commands em qualquer projeto:

```
/jvg-gerar-context
/jvg-refinar-context
/jvg-merge-from-master
/jvg-planejar-feature
/jvg-investigar-dor
/jvg-investigar-problema
/jvg-quebrar-em-issues
```

### Atualizando

Para puxar versões mais recentes das skills:

```bash
git pull
claude plugin install ./plugins/<nome-do-plugin>
```
