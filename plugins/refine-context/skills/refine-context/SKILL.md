---
name: refine-context
description: >
  Atualiza e aprimora cirurgicamente um CONTEXT.md existente (gerado pela skill
  analyze-codebase). Faz perguntas precisas e pertinentes ao usuário para entender
  exatamente o que deve mudar ANTES de alterar qualquer coisa — nunca presume
  intenção, nunca toca no que não foi pedido. Use esta skill quando o usuário
  pedir para melhorar, corrigir, atualizar ou refinar o CONTEXT.md; quando algo
  no contexto estiver desatualizado após uma mudança no código; quando o usuário
  disser "o contexto está errado", "preciso adicionar X ao contexto", "algo mudou",
  "atualize o CONTEXT.md" ou "o Claude não está entendendo Y corretamente".
---

# Refine Context

Sua missão é atualizar o `CONTEXT.md` do projeto de forma cirúrgica — mudando
apenas o que foi pedido, preservando tudo o mais. O maior risco desta skill é
entender errado e sobrescrever algo correto. Por isso, a regra de ouro é:
**perguntar antes de escrever**.

---

## Fase 1 — Orientação (nunca pule)

Antes de qualquer pergunta, faça estas leituras em paralelo:

1. Leia o `CONTEXT.md` existente na raiz do projeto (ou onde o usuário indicar).
2. Leia o `CLAUDE.md` se existir — para entender o contexto maior do projeto.
3. Identifique a solicitação do usuário: o que ele quer mudar/adicionar/corrigir?

Se não houver `CONTEXT.md`, informe o usuário e sugira rodar `/analyze-codebase`
primeiro. Não crie um CONTEXT.md do zero com esta skill.

---

## Fase 2 — Perguntas obrigatórias

Com base no que você leu, formule perguntas **específicas ao conteúdo atual** do
CONTEXT.md e ao que o usuário pediu. Não faça perguntas genéricas — cada pergunta
deve referenciar algo concreto do documento ou da solicitação.

### Perguntas sempre relevantes (adapte ao contexto):

**Sobre o escopo da mudança:**
- "Você quer alterar apenas a seção [X] ou há outras partes afetadas?"
- "A mudança é resultado de uma alteração no código (qual?) ou de uma percepção
  de que o documento estava errado desde o início?"
- "Existe alguma seção que você quer preservar exatamente como está, mesmo que
  eu identifique inconsistências nela?"

**Sobre o que está errado/incompleto:**
- "O que exatamente está errado em [trecho específico do CONTEXT.md]? É uma
  informação factualmente incorreta, desatualizada, ou mal formulada para a IA?"
- "O padrão [X citado no CONTEXT.md] ainda é válido? Ele foi substituído ou
  continua sendo a convenção?"

**Sobre o que deve ser adicionado:**
- "Quando você diz que falta [Y], isso é algo que já existe no código mas não
  está documentado, ou é uma decisão de design que você quer registrar?"
- "Qual o nível de detalhe esperado? Um parágrafo, um exemplo de código, ou
  uma nova seção completa?"

**Sobre restrições ao que não pode mudar:**
- "Tem alguma seção que foi escrita de propósito de um jeito específico e não
  deve ser reescrita mesmo que pareça melhorar?"
- "As seções 'Proibido' e 'Padrões obrigatórios' refletem o estado atual, ou
  alguma restrição foi removida/adicionada recentemente?"

### Regras para as perguntas:
- Faça entre 3 e 6 perguntas por rodada — não mais, não menos.
- Agrupe perguntas relacionadas.
- Se a resposta a uma pergunta tornar outra irrelevante, descarte a segunda.
- Após as respostas, se ainda houver ambiguidade sobre *o que exatamente mudar*,
  faça uma segunda rodada de perguntas mais focadas. Nunca avance para a edição
  com dúvidas sobre o escopo.

---

## Fase 3 — Proposta de mudanças (antes de editar)

Depois de ter clareza total sobre o escopo, apresente uma proposta explícita:

```
PROPOSTA DE ALTERAÇÕES NO CONTEXT.md

Seções que serão modificadas:
  - [Nome da seção]: [o que muda e por quê]

Seções que serão adicionadas:
  - [Nome da nova seção]: [o que vai conter]

Seções que serão removidas:
  - [Nome da seção]: [por que está sendo removida]

Seções que NÃO serão tocadas:
  - [lista das demais seções]

Confirma? (responda "sim" ou aponte o que ajustar)
```

Só avance para a edição após confirmação explícita do usuário.

---

## Fase 4 — Edição cirúrgica

Com a proposta confirmada:

1. Faça apenas as alterações aprovadas — nem mais, nem menos.
2. Preserve a voz e o estilo do documento original (se era imperativo, mantenha
   imperativo; se usava exemplos de código, mantenha o padrão).
3. Atualize a linha de data no rodapé: `_Atualizado em: YYYY-MM-DD — [motivo breve]._`
4. Se uma seção crescer muito com as adições, considere sugerir ao usuário criar
   um arquivo separado referenciado pelo CONTEXT.md (ex.: `CONTEXT-hooks.md`).

---

## Fase 5 — Revisão pós-edição

Após salvar:

1. Mostre um resumo das alterações feitas (não o arquivo inteiro — só o delta).
2. Pergunte: "Há mais algum ponto que ficou fora do escopo desta atualização mas
   que você gostaria de registrar enquanto está com o documento aberto?"
3. Se o usuário mencionar algo adicional, volte à Fase 2 com esse novo escopo.

---

## Princípios que guiam esta skill

**Precisão sobre velocidade.** É melhor gastar 2 minutos a mais perguntando do
que sobrescrever uma decisão de design importante por engano.

**O documento é fonte de verdade para a IA.** Cada alteração afeta como o Claude
Code vai se comportar nas próximas sessões. Uma regra removida por descuido pode
levar o modelo a cometer erros sistemáticos no projeto.

**Contexto mínimo suficiente.** Resistir à tentação de "melhorar" o documento
além do pedido — acumulação de informação sem curadoria degrada a qualidade do
contexto tanto quanto informação faltante.
