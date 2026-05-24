# demolidor-de-tokens

Skills para o Claude Code desenvolvidas e refinadas por mim ao longo do tempo. Cada skill é um plugin independente que pode ser instalado individualmente no seu ambiente.

---

## Skills disponíveis

| Plugin | Categoria | Descrição curta |
|--------|-----------|-----------------|
| `jvg-gerar-context` | documentação | Gera um `CONTEXT.md` otimizado para o Claude Code entender seu projeto |
| `jvg-refinar-context` | documentação | Atualiza cirurgicamente um `CONTEXT.md` existente |
| `jvg-merge-from-master` | git | Sincroniza a branch atual com `master`/`main` |
| `jvg-planejar-feature` | planejamento | Entrevista técnica iterativa para validar uma feature antes de codar |
| `jvg-investigar-dor` | planejamento | Investiga dores de usuário e gera tickets bem estruturados |
| `jvg-investigar-problema` | planejamento | Investiga um problema técnico em profundidade antes de propor solução |
| `jvg-quebrar-em-issues` | planejamento | Transforma uma análise técnica em issues vertical-slice no GitHub |

---

## Como cada skill funciona

### `jvg-gerar-context`

**Quando usar:** primeira vez que o Claude vai trabalhar sério no seu projeto, ou quando o contexto está desatualizado por conta de mudanças grandes.

**O que faz:** analisa o código-fonte e produz um `CONTEXT.md` otimizado para ser lido pelo Claude Code — não documentação para humanos, mas regras, contratos e padrões que guiam o modelo a tomar decisões corretas sem reler todos os arquivos a cada sessão.

**Como usar:**
```
/jvg-gerar-context
```
Sem argumentos. A skill explora o repositório sozinha, faz perguntas para entender padrões que não ficam evidentes no código, e gera o documento ao final.

---

### `jvg-refinar-context`

**Quando usar:** algo mudou no projeto (novo módulo, padrão revisado, decisão arquitetural) e o `CONTEXT.md` está desatualizado, ou quando o Claude está tomando decisões erradas porque o contexto não reflete mais a realidade.

**O que faz:** lê o `CONTEXT.md` existente, faz perguntas específicas sobre o que precisa mudar, e aplica as alterações de forma cirúrgica — sem tocar no que não foi pedido.

**Como usar:**
```
/jvg-refinar-context
```
Ou com contexto direto:
```
/jvg-refinar-context o módulo de pagamentos foi refatorado e agora usa Stripe em vez de PagSeguro
```

---

### `jvg-merge-from-master`

**Quando usar:** sua branch está desatualizada em relação ao `main`/`master` e você quer trazer as mudanças antes de continuar.

**O que faz:** detecta automaticamente se a branch principal é `master` ou `main`, faz fetch, tenta o merge — e se houver conflitos, aborta e lista os arquivos conflitantes sem bagunçar nada.

**Como usar:**
```
/jvg-merge-from-master
```

---

### `jvg-planejar-feature`

**Quando usar:** antes de qualquer implementação relevante — nova feature, refatoração de arquitetura, integração com serviço externo, mudança de schema. A ideia é desafiar o plano antes de codar, não depois.

**O que faz:** conduz uma entrevista técnica iterativa (uma pergunta por vez) para encontrar pontos cegos, riscos ocultos, acoplamentos perigosos e complexidade desnecessária. Ao final, produz um documento de implementação com decisões, riscos e ordem de passos.

**Como usar:**
```
/jvg-planejar-feature
```
Ou já descrevendo o que quer fazer:
```
/jvg-planejar-feature quero adicionar rate limiting por usuário no gateway
```

---

### `jvg-investigar-dor`

**Quando usar:** alguém (você, o PO, o suporte) relata uma dor ou fricção no sistema — algo lento, algo confuso, algo que incomoda — e você quer transformar isso em um ticket antes de sair codando.

**O que faz:** conduz uma conversa estruturada para entender o problema do ponto de vista do usuário (sem expor detalhes técnicos), analisa o código em background para enriquecer as perguntas, verifica duplicidade no GitHub Issues, e gera um ticket com User Story + Critérios de Aceite — só com aprovação explícita.

**Como usar:**
```
/jvg-investigar-dor
```
Ou relatando a dor diretamente:
```
/jvg-investigar-dor o fluxo de checkout está abandonando muita gente na etapa de endereço
```

---

### `jvg-investigar-problema`

**Quando usar:** há um problema técnico real (bug, lentidão, comportamento inesperado, dívida técnica) e você quer entender a causa antes de sair propondo solução.

**O que faz:** explora código, mapeia dependências, investiga queries, logs e APIs, conduz uma conversa guiada com o desenvolvedor ao longo da investigação, e ao final propõe opções de solução com tradeoffs, riscos e estimativa de impacto. O documento técnico só é gerado após aprovação explícita.

**Como usar:**
```
/jvg-investigar-problema
```
Ou descrevendo o problema:
```
/jvg-investigar-problema as queries de relatório estão levando 30s e o timeout começa em 25s
```

---

### `jvg-quebrar-em-issues`

**Quando usar:** você tem uma análise técnica pronta (geralmente vinda do `jvg-investigar-problema`) e o próximo passo é transformá-la em tickets acionáveis no GitHub.

**O que faz:** conduz uma conversa para decidir qual abordagem priorizar, definir contratos entre serviços, estratégia de rollout, observabilidade e testes. Quebra o trabalho em fatias verticais independentes — cada issue é implementável e deployável do começo ao fim sem depender de outras. Só publica no GitHub após aprovação explícita.

**Como usar:**
```
/jvg-quebrar-em-issues
```
Funciona melhor se o documento de análise técnica estiver aberto ou colado na conversa.

---

## Fluxos de trabalho

### Entender o projeto antes de começar

Ideal quando você entra num projeto novo ou volta depois de um tempo.

```
/jvg-gerar-context
```

Após isso, o `CONTEXT.md` fica na raiz do projeto. Nas sessões seguintes, o Claude lê esse arquivo automaticamente se estiver no `.claude/commands` ou referenciado no `CLAUDE.md`.

Quando o projeto evoluir:

```
/jvg-refinar-context
```

---

### Investigar e resolver um problema técnico

Quando há um bug, lentidão ou comportamento inesperado que precisa ser investigado antes de qualquer code.

```
1. /jvg-investigar-problema   ← entende a causa raiz, propõe opções
2. /jvg-quebrar-em-issues     ← transforma a análise em issues no GitHub
```

As issues geradas pelo `jvg-quebrar-em-issues` são fatias verticais — cada uma pode ser pega por um desenvolvedor diferente e implementada independentemente.

---

### Planejar e executar uma feature nova

Quando você tem uma ideia de feature e quer validá-la antes de codar.

```
1. /jvg-planejar-feature      ← desafia o plano, encontra riscos, produz documento
2. /jvg-quebrar-em-issues     ← transforma o documento em issues no GitHub
```

---

### Transformar dor de usuário em ticket

Quando alguém relata um problema do ponto de vista do usuário (não técnico) e você quer criar um ticket bem estruturado.

```
/jvg-investigar-dor           ← investiga a dor, gera ticket com User Story
```

Opcionalmente, se o ticket gerado revelar um problema técnico complexo:

```
/jvg-investigar-problema      ← investiga a causa técnica por trás da dor
/jvg-quebrar-em-issues        ← quebra em issues para execução
```

---

### Sincronizar branch antes de abrir PR

```
/jvg-merge-from-master
```

---

## Como instalar

### Pré-requisito

Você precisa do [Claude Code](https://claude.ai/code) instalado:

```bash
npm i -g @anthropic-ai/claude-code
```

### 1. Clone o repositório

```bash
git clone https://github.com/JoaoVitorGirardii/marketplace-demolidor-de-tokens.git
cd marketplace-demolidor-de-tokens
```

### 2. Instale os plugins que quiser

Instale um plugin específico:

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

```bash
git pull
for plugin in plugins/*/; do claude plugin install "$plugin"; done
```
