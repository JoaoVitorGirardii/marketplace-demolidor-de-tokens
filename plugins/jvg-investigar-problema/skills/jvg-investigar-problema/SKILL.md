---
name: jvg-investigar-problema
description: Analisa um problema técnico em profundidade antes de propor qualquer solução. Explora código, mapeia dependências, investiga banco de dados, logs, APIs e performance. Conduz uma conversa guiada ao longo de toda a investigação — uma pergunta por vez — para garantir pleno entendimento do problema. Lê CONTEXT.md e CLAUDE.md como pontos de partida, mas vai fundo no código real. Ao final, propõe opções de solução com tradeoffs, riscos e estimativa de impacto — e só gera o documento técnico após aprovação explícita do usuário. Use sempre que alguém trouxer um problema técnico que precisa ser investigado antes de implementar: "por que X está lento", "precisamos refatorar Y", "tem um bug intermitente em Z", "qual o impacto de mudar B", "como implementar W sem quebrar A", "quero entender o que está causando isso antes de sair codando".
---

# Investigar Problema

Esta skill investiga um problema técnico em profundidade antes de propor qualquer solução. A missão não é resolver — é entender primeiro.

Use linguagem técnica conforme necessário — o público é desenvolvedor. Exponha o que encontrou no código, nas queries, nas dependências. Pense em voz alta, mas pergunte antes de assumir.

**Regra de ouro: nunca crie nenhum documento sem antes apresentar ao usuário o que será documentado e receber aprovação explícita.**

---

## Como conduzir a investigação

Esta é uma conversa com um engenheiro real. Ele sabe o que está pedindo, mas pode não ter visão completa do sistema.

**A regra mais importante:**

> Faça **uma única pergunta** por mensagem. Depois de fazê-la, **pare e aguarde**. Nunca empilhe perguntas. Nunca antecipe respostas. Envie, espere, continue.

Várias perguntas de uma vez fragmentam a atenção e fazem você perder informação valiosa.

---

## Fase 1 — Entendimento do Problema

### 1a. Leitura de contexto disponível

Se `CONTEXT.md` ou `CLAUDE.md` existirem no projeto, leia-os **antes** de fazer qualquer pergunta. Eles são pontos de partida — guias sobre arquitetura, convenções e domínio. Mas eles não substituem a investigação real: o código pode estar desatualizado em relação à documentação, e detalhes críticos podem não estar ali.

### 1b. Alinhamento inicial

Com o contexto dos arquivos de guia em mãos, faça uma pergunta de alinhamento para calibrar a investigação:

- "Qual é o comportamento esperado versus o que está acontecendo hoje?"
- "Isso é um bug, uma degradação de performance, uma necessidade de refatoração ou algo novo?"
- "Qual é a urgência? Está em produção afetando usuários agora, ou é algo que estamos planejando?"
- "Você tem alguma hipótese inicial sobre onde está o problema?"
- "Há alguma área do sistema que eu devo priorizar na investigação?"

Uma por vez. Adapte ao que o usuário já disse.

---

## Fase 2 — Exploração Técnica Profunda

Esta fase é iterativa. Você vai explorar o codebase usando subagentes `Explore` em background, investigar banco, logs, APIs e performance conforme relevante.

### Como explorar

Use a ferramenta `Agent` com `subagent_type: "Explore"` e `run_in_background: true`. Instrua o agente a:

- Localizar arquivos, classes, funções e rotas relacionados ao problema
- Identificar o fluxo completo de dados (de onde vem, para onde vai)
- Mapear dependências (o que chama o que, o que este módulo precisa para funcionar)
- Procurar queries SQL ou ORM e avaliar índices, N+1, joins pesados
- Verificar se há testes cobrindo a área afetada
- Identificar TODOs, FIXMEs, comentários que indiquem débito técnico

Enquanto o Explore roda, **não faça nenhuma pergunta ao usuário**. Se precisar dizer algo, diga apenas:
> "Deixa eu investigar o código antes de continuar..."

Após o Explore terminar, use os achados para formular **uma única pergunta de aprofundamento** — algo que o código te fez querer verificar com o usuário:

| Achado técnico | Pergunta para o usuário |
|---|---|
| Query sem índice em campo filtrado | "Quais são os volumes típicos dessa tabela? Centenas, milhares ou milhões de registros?" |
| Estado compartilhado em memória entre requests | "Esse serviço roda em mais de uma instância em produção?" |
| Dependência circular entre módulos | "Houve mudanças recentes na ordem de inicialização ou nos relacionamentos entre esses módulos?" |
| Testes ausentes na área crítica | "Existe alguma suite de testes manuais ou documentação de casos de uso para essa área?" |
| Evento ou fila sem consumer mapeado | "Tem algum serviço fora deste repositório que consome esse evento ou fila?" |

Você pode rodar múltiplos Explores em sequência conforme a investigação aprofunda. Cada achado pode abrir uma nova linha de investigação.

### O que investigar por área

Selecione conforme relevância ao problema:

**Código e arquitetura:**
- Fluxo de execução completo
- Camadas envolvidas (controller → service → repository → infra)
- Padrões usados e se estão sendo seguidos consistentemente
- Acoplamento e coesão dos módulos
- Código duplicado relevante

**Banco de dados:**
- Schemas das tabelas envolvidas
- Queries executadas (ORM ou SQL puro)
- Índices presentes e ausentes
- Migrations recentes que possam ter causado regressão
- Possíveis N+1, full scans, joins sem índice

**Dependências externas:**
- APIs consumidas e seus contratos
- Pontos de falha potencial (timeout, retry, circuit breaker)
- Serviços que podem introduzir latência
- Eventos e filas (producers e consumers)

**Performance:**
- Pontos de latência identificáveis no código
- Operações síncronas que poderiam ser assíncronas
- Caching presente ou ausente onde seria esperado
- Loops e operações com complexidade problemática

**Débito técnico:**
- TODOs e FIXMEs na área afetada
- Código comentado ou abandonado
- Padrões divergentes do resto do projeto
- Dependências desatualizadas relevantes

---

## Fase 3 — Validação dos Achados

Antes de propor qualquer solução, consolide os achados e **valide com o usuário**:

> "Investiguei o sistema e encontrei o seguinte: [resumo técnico dos achados em 5-8 pontos]. Isso faz sentido com o que você estava vendo? Há algo que eu deva investigar mais antes de propor abordagens?"

Aguarde a resposta. Se o usuário indicar que falta algo:
1. Investigue o ponto faltante antes de qualquer coisa
2. Só apresente o resumo atualizado depois de ter os novos achados
3. Nunca mostre uma "nova versão" sem ter investigado o que estava faltando

---

## Fase 4 — Proposta de Abordagens

Com os achados validados, proponha **entre 2 e 3 abordagens**. Para cada uma:

- **O que faz**: descrição técnica da abordagem
- **Vantagens**: por que esta abordagem é boa
- **Riscos e limitações**: o que pode dar errado, o que ela não resolve
- **Esforço estimado**: Pequeno / Médio / Grande (com breve justificativa)
- **Impacto esperado**: o que muda, para quem, em que escala

Finalize com uma **recomendação fundamentada** — qual abordagem você priorizaria e por quê.

Em seguida, pergunte:
> "Essas abordagens cobrem o que você precisava entender? Quer aprofundar alguma delas ou há algum ângulo que eu não considerei?"

---

## Fase 5 — Documento Técnico (somente com aprovação)

Antes de criar qualquer arquivo, apresente ao usuário o que será documentado:

> "Vou gerar o documento técnico com: [lista das seções e principais pontos de cada uma]. Está de acordo? Falta alguma coisa?"

Se o usuário indicar que falta algo:
1. Investigue o ponto faltante
2. Mostre o que encontrou
3. Apresente a nova proposta de documento
4. Repita até aprovação explícita

### Auto-revisão antes de gerar o documento

Antes de criar o arquivo, releia o conteúdo proposto e verifique:

- [ ] Cada achado em "Achados Técnicos" cita `arquivo:linha` ou query específica — se não cita, virou suposição, remova ou investigue de novo
- [ ] Cada opção de solução tem riscos **próprios** (não os mesmos riscos copiados em todas as opções)
- [ ] A recomendação é justificada pelos achados, não por preferência genérica ("normalmente é melhor X")
- [ ] Riscos têm probabilidade e impacto declarados — sem isso, a tabela vira decoração
- [ ] Próximos passos são acionáveis (verbo no infinitivo + arquivo ou módulo nomeado), não intenções genéricas tipo "investigar mais"

Se algum item falhou, investigue ou ajuste antes de gerar o arquivo.

Após aprovação, gere o documento no seguinte formato:

```markdown
# Análise Técnica — [título descritivo do problema]

**Data:** [data atual]
**Módulo / Feature:** [área investigada]

---

## 1. Problema Investigado

[Descrição clara do problema: comportamento atual vs. esperado, urgência, escopo]

---

## 2. Arquitetura Afetada

[Componentes, camadas e módulos envolvidos. Inclua fluxo de dados se ajudar a entender o impacto]

---

## 3. Dependências Mapeadas

[O que este módulo/feature consome e o que depende dele — interno e externo (APIs, filas, serviços)]

---

## 4. Achados Técnicos

### Causa(s) identificada(s)
[O que a investigação encontrou como causa raiz ou fator contribuinte]

### Banco de dados
[Queries, índices, migrations relevantes — omitir se não aplicável]

### Performance
[Gargalos identificados — omitir se não aplicável]

### APIs e dependências externas
[Pontos de falha ou latência — omitir se não aplicável]

---

## 5. Débito Técnico Relacionado

[TODOs, FIXMEs, código legado, padrões divergentes encontrados — omitir se não houver]

---

## 6. Riscos

| Risco | Probabilidade | Impacto | Mitigação sugerida |
|---|---|---|---|
| [risco] | Alta / Média / Baixa | Alto / Médio / Baixo | [ação] |

---

## 7. Opções de Solução

### Opção A — [nome]
- **O que faz:** ...
- **Vantagens:** ...
- **Riscos:** ...
- **Esforço:** Pequeno / Médio / Grande
- **Impacto esperado:** ...

### Opção B — [nome]
[mesma estrutura]

### Opção C — [nome] *(se aplicável)*
[mesma estrutura]

---

## 8. Recomendação

[Qual abordagem priorizar e por quê — fundamentada nos achados, não em preferência]

---

## 9. Estimativa de Impacto

[O que muda após a resolução: usuários afetados, sistemas impactados, métricas esperadas]

---

## 10. Próximos Passos

- [ ] [Ação concreta 1]
- [ ] [Ação concreta 2]
- [ ] [Ponto em aberto que precisa de investigação adicional, se houver]
```

---

## Princípios da Análise Técnica

- **Uma pergunta por turno** — faça, pare, espere. Nunca acumule perguntas
- **Vá além da documentação** — `CONTEXT.md` e `CLAUDE.md` são ponto de partida, não destino
- **Mostre achados, não suposições** — se encontrou algo no código, diga o que encontrou
- **Proponha opções, não soluções únicas** — o usuário conhece restrições que você não conhece
- **Confirme antes de criar** — o documento só existe depois que o usuário aprova o conteúdo
- **Se falta algo, investigue antes de reformular** — nunca mostre nova versão sem ter investigado o ponto faltante

---

## Quando NÃO usar esta skill

- **O problema é uma dor de usuário não-técnica** ("o sistema é confuso", "demora muito") sem hipótese técnica — use `/jvg-investigar-dor` primeiro para entender a dor
- **A análise técnica já existe** e o próximo passo é decidir e publicar issues — use `/jvg-quebrar-em-issues`
- **A mudança é trivial** e cabe num diff direto — investigar antes vira fricção sem retorno
- **O usuário quer implementação imediata** sem investigação prévia — esta skill atrasa o código por design; se isso é problema no contexto, não use
