# demolidor-de-tokens

Skills para o Claude Code desenvolvidas e refinadas por mim ao longo do tempo. Cada skill é um plugin independente que pode ser instalado individualmente no seu ambiente.

---

## Visão geral — ordem de uso

```
INÍCIO DE PROJETO
      │
      ▼
  1. /jvg-gerar-context          ← entenda o projeto antes de qualquer coisa
      │
      │   (quando algo mudar)
      ▼
  2. /jvg-refinar-context        ← mantenha o contexto atualizado
      │
      └─────────────────────────────────────────────┐
                                                     │
    TENHO UMA DOR DE USUÁRIO          TENHO UM PROBLEMA TÉCNICO    TENHO UMA IDEIA DE FEATURE
             │                                  │                            │
             ▼                                  ▼                            ▼
  3. /jvg-investigar-dor         4. /jvg-investigar-problema    5. /jvg-planejar-feature
     (gera ticket direto)            (análise técnica profunda)     (valida o plano antes)
             │                                  │                            │
             │   (se for complexo)              └────────────────────────────┘
             │        │                                      │
             └────────┘                                      ▼
                                               6. /jvg-quebrar-em-issues
                                                  (issues vertical-slice no GitHub)
                                                             │
                                                             ▼
                                                        EXECUÇÃO

  A qualquer momento, antes de abrir PR:
      7. /jvg-merge-from-master  ← sincroniza sua branch com main/master
```

---

## Skills — do primeiro ao último passo

### 1. `jvg-gerar-context`

**Quando usar:** primeira vez que o Claude vai trabalhar sério no projeto, ou quando o contexto está desatualizado por mudanças grandes.

**O que faz:** analisa o código-fonte e produz um `CONTEXT.md` otimizado para ser lido pelo Claude Code — não documentação para humanos, mas regras, contratos e padrões que guiam o modelo a tomar decisões corretas sem reler todos os arquivos a cada sessão.

```
/jvg-gerar-context
```

---

### 2. `jvg-refinar-context`

**Quando usar:** algo mudou no projeto (novo módulo, padrão revisado, decisão arquitetural) e o `CONTEXT.md` está desatualizado, ou quando o Claude está tomando decisões erradas.

**O que faz:** lê o `CONTEXT.md` existente, faz perguntas específicas sobre o que precisa mudar, e aplica as alterações de forma cirúrgica — sem tocar no que não foi pedido.

```
/jvg-refinar-context
```
```
/jvg-refinar-context o módulo de pagamentos agora usa Stripe em vez de PagSeguro
```

---

### 3. `jvg-investigar-dor`

**Quando usar:** alguém (você, o PO, o suporte) relata uma dor ou fricção no sistema — algo lento, confuso ou que incomoda — e você quer transformar isso em um ticket antes de sair codando.

**O que faz:** conduz uma conversa estruturada para entender o problema do ponto de vista do usuário, analisa o código em background para enriquecer as perguntas, verifica duplicidade no GitHub Issues, e gera um ticket com User Story + Critérios de Aceite — só com aprovação explícita.

```
/jvg-investigar-dor
```
```
/jvg-investigar-dor o fluxo de checkout está abandonando muita gente na etapa de endereço
```

---

### 4. `jvg-investigar-problema`

**Quando usar:** há um problema técnico real (bug, lentidão, comportamento inesperado, dívida técnica) e você quer entender a causa antes de propor qualquer solução.

**O que faz:** explora código, mapeia dependências, investiga queries, logs e APIs, conduz uma conversa guiada ao longo da investigação, e ao final propõe opções de solução com tradeoffs, riscos e estimativa de impacto. O documento técnico só é gerado após aprovação explícita.

```
/jvg-investigar-problema
```
```
/jvg-investigar-problema as queries de relatório estão levando 30s e o timeout começa em 25s
```

---

### 5. `jvg-planejar-feature`

**Quando usar:** antes de qualquer implementação relevante — nova feature, refatoração de arquitetura, integração com serviço externo, mudança de schema. A ideia é desafiar o plano antes de codar, não depois.

**O que faz:** conduz uma entrevista técnica iterativa (uma pergunta por vez) para encontrar pontos cegos, riscos ocultos, acoplamentos perigosos e complexidade desnecessária. Ao final, produz um documento de implementação com decisões, riscos e ordem de passos.

```
/jvg-planejar-feature
```
```
/jvg-planejar-feature quero adicionar rate limiting por usuário no gateway
```

---

### 6. `jvg-quebrar-em-issues`

**Quando usar:** você tem uma análise técnica ou plano de feature prontos e quer transformá-los em tickets acionáveis no GitHub.

**O que faz:** conduz uma conversa para decidir qual abordagem priorizar, definir contratos entre serviços, estratégia de rollout, observabilidade e testes. Quebra o trabalho em fatias verticais independentes — cada issue é implementável e deployável do começo ao fim sem depender de outras. Só publica no GitHub após aprovação explícita.

```
/jvg-quebrar-em-issues
```

Funciona melhor com o documento de análise ou plano aberto na conversa.

---

### 7. `jvg-merge-from-master`

**Quando usar:** sua branch está desatualizada em relação ao `main`/`master` e você quer trazer as mudanças antes de continuar ou abrir PR.

**O que faz:** detecta automaticamente se a branch principal é `master` ou `main`, faz fetch, tenta o merge — e se houver conflitos, aborta e lista os arquivos conflitantes sem bagunçar nada.

```
/jvg-merge-from-master
```

---

## Fluxos de trabalho completos

### Projeto novo ou retorno após longa ausência

```
1. /jvg-gerar-context
      └─→ CONTEXT.md gerado na raiz do projeto
```

Nas sessões seguintes o Claude lê o `CONTEXT.md` automaticamente. Quando o projeto evoluir:

```
2. /jvg-refinar-context
      └─→ CONTEXT.md atualizado cirurgicamente
```

---

### Bug, lentidão ou comportamento inesperado

```
1. /jvg-investigar-problema
      └─→ análise técnica com causa raiz + opções de solução

2. /jvg-quebrar-em-issues
      └─→ issues vertical-slice publicadas no GitHub
```

---

### Feature nova

```
1. /jvg-planejar-feature
      └─→ plano validado com riscos e decisões documentadas

2. /jvg-quebrar-em-issues
      └─→ issues vertical-slice publicadas no GitHub
```

---

### Dor relatada pelo usuário, PO ou suporte

```
1. /jvg-investigar-dor
      └─→ ticket com User Story + Critérios de Aceite

   (se o ticket revelar problema técnico complexo)

2. /jvg-investigar-problema
      └─→ análise técnica da causa raiz

3. /jvg-quebrar-em-issues
      └─→ issues para execução
```

---

### Antes de abrir PR

```
/jvg-merge-from-master
      └─→ branch sincronizada; conflitos listados se houver
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

### 2. Instale os plugins

Instale um plugin específico:

```bash
claude plugin install ./plugins/jvg-gerar-context
claude plugin install ./plugins/jvg-refinar-context
claude plugin install ./plugins/jvg-investigar-dor
claude plugin install ./plugins/jvg-investigar-problema
claude plugin install ./plugins/jvg-planejar-feature
claude plugin install ./plugins/jvg-quebrar-em-issues
claude plugin install ./plugins/jvg-merge-from-master
```

Ou instale todos de uma vez:

```bash
for plugin in plugins/*/; do claude plugin install "$plugin"; done
```

### 3. Use as skills

Após instalar, as skills ficam disponíveis como slash commands em qualquer projeto:

```
/jvg-gerar-context
/jvg-refinar-context
/jvg-investigar-dor
/jvg-investigar-problema
/jvg-planejar-feature
/jvg-quebrar-em-issues
/jvg-merge-from-master
```

### Atualizando

```bash
git pull
for plugin in plugins/*/; do claude plugin install "$plugin"; done
```
