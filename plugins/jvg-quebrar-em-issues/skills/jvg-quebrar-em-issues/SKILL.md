---
name: jvg-quebrar-em-issues
description: Recebe uma análise técnica pronta (tipicamente gerada pela skill jvg-investigar-problema) e a transforma num plano de execução publicado como issues vertical-slice no GitHub. Conduz conversa guiada — uma pergunta por vez — para decidir qual abordagem priorizar entre as opções da análise, definir contratos, migrações, estratégia de rollout, observabilidade e testes. Quebra o trabalho em fatias verticais independentes (cada issue é implementável e deployável do começo ao fim sem depender de outras) e só publica no GitHub após aprovação explícita. Use sempre que houver uma análise técnica pronta e o próximo passo for sair dela com tickets acionáveis: "vamos transformar essa análise em issues", "preciso decidir qual abordagem implementar", "como quebra isso em tarefas", "preciso de um plano de execução", "ok, e agora como a gente executa isso", "monta as issues no GitHub a partir desse documento", "preciso planejar o rollout disso", ou sempre que o usuário trouxer um documento técnico e quiser sair com tickets prontos.
disable-model-invocation: true
---

# Quebrar em Issues

Esta skill recebe uma análise técnica pronta (tipicamente gerada pela skill `jvg-investigar-problema`) e conduz o usuário até um plano de execução concreto, publicado como issues no GitHub que podem ser pegadas e implementadas independentemente. A função não é investigar — isso já foi feito.

A skill decide *o que* será feito, *com quais contratos*, *com qual estratégia de rollout*, e quebra isso em fatias deployáveis — mas não implementa.

**Regra de ouro: nunca publique uma issue sem antes apresentar o conteúdo completo ao usuário e receber aprovação explícita.**

---

## Como conduzir a conversa

Esta é uma decisão real sobre execução real. O usuário sabe muito sobre o problema (acabou de fazer a análise técnica), mas pode não ter pensado em todos os eixos: rollout, observabilidade, contratos com outros serviços, ordem do trabalho.

**A regra mais importante da skill:**

> Faça **uma única pergunta** por mensagem. Depois de fazê-la, **pare e aguarde**. Várias perguntas empilhadas geram respostas parciais e você perde sinal — pior, parece que você não está ouvindo.

Tom: técnico, direto, sem rodeios. Quando algo for ambíguo, levante. Quando algo for arriscado, diga. Quando algo já estiver claro na análise, **não pergunte de novo** — referencie o que está lá e siga.

---

## Fase 1 — Recepção da Análise Técnica

A skill aceita a análise técnica em duas formas — detecte automaticamente:

- **Caminho de arquivo**: o usuário passa algo curto que termina em `.md` ou parece um path (`docs/...`, `./...`, `/...`). Use o `Read` para carregar.
- **Conteúdo colado**: o usuário cola o markdown diretamente. Geralmente tem várias quebras de linha, começa com `# ` ou `---`, contém seções como `## Problema` ou `## Opções de Solução`.

Se nenhum dos dois aconteceu na conversa, pergunte:

> "Antes de começarmos, me passa a análise técnica. Você pode colar o conteúdo aqui ou me dar o caminho do arquivo (algo como `docs/analise/...md`)."

Se a entrada for ambígua (uma única linha sem `.md` mas que também não parece markdown), pergunte qual dos dois é.

### Validação inicial do que chegou

Leia a análise inteira com atenção. Ela tipicamente tem as seções: Problema, Arquitetura Afetada, Dependências, Achados Técnicos, Débito Técnico, Riscos, Opções de Solução, Recomendação, Estimativa de Impacto, Próximos Passos.

**Se faltar algo crítico para tomar decisão** (ex: não tem opções de solução, ou os riscos não foram mapeados), pare e diga ao usuário:

> "A análise que você passou não tem [X]. Sem isso eu não tenho base para [Y]. Você quer voltar e completar antes (provavelmente rodando a skill `jvg-investigar-problema`), ou preferia que a gente decida [X] aqui mesmo na conversa?"

Quando a análise estiver utilizável, confirme entendimento em 3-5 linhas:

> "Entendi o seguinte: [problema central], [as opções A/B/C resumidas em uma linha cada], a recomendação foi [X] porque [resumo da justificativa], e os principais riscos são [lista curta]. Antes de seguirmos, isso está correto ou tem algo que eu interpretei mal?"

Aguarde confirmação ou correção antes de avançar.

---

## Fase 2 — Decisão da Abordagem

A análise traz opções (A, B, C) e uma recomendação. **A recomendação da análise não é a decisão final** — a decisão é do usuário. Sua função é facilitar essa escolha, não passar batido na recomendação.

Pergunte:

> "A análise recomenda a Opção [X] porque [resumo da justificativa]. Você quer seguir com ela, ou prefere outra (ou uma variação)?"

Se o usuário escolher diferente da recomendação, pergunte por quê — pode haver restrição que você precisa saber para planejar (prazo, custo, time disponível, dependência externa, política interna). Não é contestação, é coleta de informação.

Se vier uma variação ("Opção B mas sem a parte do cache", "uma mistura de A e C"), reescreva a abordagem final em 3-4 linhas e confirme:

> "Então a abordagem é: [descrição]. Fechado assim?"

Só avance quando a abordagem estiver inequívoca. Plano construído sobre abordagem ambígua produz issues ambíguas.

---

## Fase 3 — Decisões de Execução

Agora preencha os eixos que vão alimentar o conteúdo de cada issue. **Nem todo problema usa todos os eixos** — se a abordagem não toca em banco, pule migrações; se não tem rollout faseado, registre "big-bang" e segue. Não invente decisão para encher linguiça.

Para cada eixo relevante: **primeiro proponha** uma decisão concreta baseada na análise e na abordagem escolhida, **depois pergunte** se está de acordo. Proposta concreta é mais fácil de reagir do que pergunta aberta.

Vá um eixo por vez. Dentro de cada eixo, uma pergunta por turno.

### 3.1 Contratos

O que muda na interface entre módulos, serviços, ou sistemas? Pense em:

- APIs HTTP: endpoints novos ou modificados, mudança de schema de request/response
- Eventos / mensageria: novos eventos publicados ou consumidos, mudança de payload
- Bibliotecas internas e packages: novas funções exportadas, mudança de assinatura
- Interfaces de banco que outros sistemas consultam diretamente

**Pergunta-modelo**: "Pela abordagem, vai surgir um novo endpoint `POST /api/...` com payload `{...}`. Algum consumidor já está esperando esse contrato, ou temos liberdade pra desenhar?"

Se não houver contrato externo afetado, registre internamente e siga.

### 3.2 Migrações

O que muda no schema do banco ou em dados existentes? Pense em:

- Migration adiciona/altera colunas, índices, tabelas
- Precisa fazer backfill ou normalização de dados antigos
- Tem dependência de ordem com o deploy do código (migration antes? depois?)
- Tem caminho de rollback?

**Pergunta-modelo**: "A análise aponta uma nova coluna `Y` em `Z`. Essa tabela tem que volume de dados (centenas, milhares, milhões de rows)? Isso afeta se a gente vai com `NOT NULL` + default direto ou nullable com backfill em background."

Se não tiver mudança de banco, declare isso e siga.

### 3.3 Rollout

Como essa mudança chega em produção sem virar incidente? Pense em:

- Feature flag (nome, escopo, quem controla, como liga/desliga)
- Canary / rollout faseado (percentual inicial, critério de avanço)
- Big-bang (deploy direto, sem flag — quando o risco é baixo)
- Coexistência com comportamento antigo (qual é a transição)

**Pergunta-modelo**: "Pelo impacto apontado na análise, sugiro feature flag `feature_x_enabled` começando em 10% dos [usuários/tenants/etc.]. Faz sentido, ou tem algo operacional que muda isso (ex: vocês não usam flags, ou querem dual-write)?"

### 3.4 Observabilidade

Como detectar regressão ou sucesso em produção? Pense em:

- Logs estruturados (o que logar e em qual nível)
- Métricas (contador, latência, taxa de erro)
- Traces (spans relevantes)
- Alertas (que condição dispara, para quem, em qual canal)

**Pergunta-modelo**: "Pra detectar regressão, sugiro instrumentar métrica `feature_x_request_duration_ms` (p50/p95/p99) e alerta se p95 passar de [N]ms. Vocês têm algum stack/canal padrão (Datadog, Grafana, PagerDuty, Slack)?"

### 3.5 Testes

O que precisa ser coberto antes de considerar pronto? Pense em:

- Unitários (lógica nova, casos de borda)
- Integração (contratos entre módulos, banco real, fila real)
- E2E (fluxo crítico do usuário ponta a ponta)
- Manual (cenários que não vale automatizar)

**Pergunta-modelo**: "Os critérios mínimos de teste que sugiro: [3-5 cenários concretos]. Tem algum cenário específico que você quer garantir, ou algum tipo de teste obrigatório no projeto (ex: cobertura mínima, teste e2e obrigatório em fluxo de pagamento)?"

---

## Fase 4 — Quebra em Vertical Slices

Agora consolide tudo em **fatias verticais independentes**. Cada fatia vira uma issue no GitHub.

### O que é uma vertical slice (regra dura)

Cada slice precisa ser:

- **Implementável do começo ao fim sem depender de outra issue do plano.** Atravessa todas as camadas relevantes (schema → service → API → UI → testes → observabilidade) em uma única passagem.
- **Deployável de forma isolada.** Quando essa issue entra em produção sozinha, agrega valor ou ao menos não quebra nada.
- **Demoável.** Alguém olhando o resultado consegue dizer "isso funciona".
- **Pequena o suficiente** para uma pessoa pegar e concluir em alguns dias.

**Por que isso importa**: dependências entre issues criam fila de bloqueio, dificultam priorização, e quando uma issue trava todo o resto trava junto. Vertical slice é o oposto disso — múltiplas pessoas podem pegar issues diferentes em paralelo e cada uma chega em produção quando estiver pronta.

**Anti-padrões a evitar:**

- ❌ **Slice horizontal**: "criar a migration", "criar o service", "criar o controller". Isso é camada, não fatia.
- ❌ **Slice que depende de outra para ser deployável**. Se a issue 2 só funciona depois da issue 1, junte as duas numa única slice maior.
- ❌ **Slice gigante** cobrindo o problema inteiro. Quebre em fatias menores que entreguem valor incremental.

**Quando o trabalho realmente exige sequência** (ex: migration precisa rodar antes do código que a usa), prefira **agrupar tudo em uma única issue** ao invés de criar dependências entre issues. A migration faz parte da slice que a usa.

### Apresentação da quebra

Apresente como lista numerada. Para cada slice:

```
Slice N — [Título curto]
- O que entrega: [comportamento end-to-end que essa slice traz]
- Camadas tocadas: [schema, service, API, UI, etc.]
- Critério de "pronto": [como saber que está completa, alto nível]
- Inclui dos eixos da Fase 3: [ex: "feature flag feature_x_enabled", "migration add_y_to_z", "métrica feature_x_request_duration_ms"]
- Tamanho estimado: P / M / G
```

Depois pergunte:

> "Essas são as [N] fatias que enxergo. Antes de eu detalhar e publicar:
> 1. A granularidade está boa? Alguma é grande demais (precisa quebrar) ou pequena demais (precisa juntar com outra)?
> 2. Falta alguma fatia, ou sobra alguma?"

Itere até o usuário aprovar a lista. Não avance com lista provisória.

---

## Fase 5 — Detalhamento de Cada Issue

Para cada slice aprovada, monte o corpo da issue antes de publicar. Use este template:

```markdown
## O que entrega

[Descrição do comportamento end-to-end que essa issue entrega — pense no que o sistema *passa a fazer*, não em "criar a função X". Linguagem voltada para quem vai implementar.]

## Decisões já tomadas

- **Abordagem**: [resumo da abordagem escolhida na Fase 2]
- **Contrato**: [se aplicável — endpoint, evento, schema. Omita se não houver]
- **Migração**: [se aplicável — qual mudança de schema, com ou sem backfill. Omita se não houver]
- **Rollout**: [feature flag, percentual, big-bang. Omita se não for relevante]
- **Observabilidade**: [logs / métricas / alertas que esta issue deve instrumentar]

## Critérios de aceite

- [ ] [Critério funcional verificável]
- [ ] [Critério funcional verificável]
- [ ] [Critério de teste — ex: testes unitários cobrindo X; teste e2e do fluxo Y]
- [ ] [Critério de observabilidade — ex: métrica Z aparece no dashboard]
- [ ] [Critério de rollout — ex: feature flag criada e desligada por padrão]

## Contexto

[Referência à análise técnica que originou o plano (link, path, ou cole o resumo). Notas relevantes que ajudam quem vai implementar a entender por que estamos fazendo isso desta forma.]

## Riscos desta fatia

[Riscos específicos desta fatia. Riscos transversais ficam mencionados como contexto, não duplicados em cada issue.]
```

Apresente o **primeiro issue completo** ao usuário e pergunte:

> "Esse é o primeiro issue. Tá no formato que você quer? Algum ajuste no template ou no conteúdo antes de eu montar os outros?"

Quando o formato estiver aprovado, monte os demais. Apresente todos em sequência (ou em lote, se o usuário preferir) e aguarde aprovação final antes de publicar.

### Auto-revisão antes de publicar

Antes de chamar `gh issue create`, releia **cada issue** e verifique:

- [ ] Esta issue, deployada **sozinha** em produção, agrega valor ou ao menos não quebra nada — se a resposta é "depende da issue X", junte as duas
- [ ] Tem ao menos um critério de aceite **verificável** (não "feature funciona", mas "endpoint X retorna 200 com payload Y")
- [ ] As "Decisões já tomadas" referenciam apenas decisões confirmadas na Fase 3 — nada inventado para parecer completo
- [ ] Se a issue tem migration, ela inclui também o código que consome a migration (sem dependência cruzada com outra issue)
- [ ] Riscos transversais aparecem como contexto na issue, não duplicados em cada uma

Se algum item falhou, ajuste antes de publicar.

---

## Fase 6 — Publicação no GitHub

### Descobrir o repositório

Tente detectar automaticamente:

```bash
gh repo view --json nameWithOwner -q .nameWithOwner
```

Se não funcionar (sem `gh` configurado, sem repo remoto, etc.), pergunte:

> "Não consegui detectar o repositório destino. Qual é? (formato `dono/repo`)"

### Confirmar labels

Antes de publicar, pergunte sobre labels:

> "Quer aplicar alguma label nessas issues? (ex: `feature`, `tech-debt`, `priority/high`, ou o vocabulário que vocês usam). Se quiser, eu rodo `gh label list` pra te mostrar as labels existentes."

### Confirmação final

> "Vou publicar [N] issues no repositório `dono/repo` agora, com as labels `[lista]`. Confirma?"

### Publicar

Para cada issue, use o `gh` CLI passando o body via HEREDOC para preservar formatação:

```bash
gh issue create \
  --repo "<dono/repo>" \
  --title "<título da slice>" \
  --label "<label1,label2>" \
  --body "$(cat <<'EOF'
<corpo da issue gerado na Fase 5>
EOF
)"
```

Publique em ordem (1 → N). Capture a URL retornada de cada uma.

### Fechar com URLs

Apresente a lista publicada:

```
Publicadas:
1. #123 — [título] — https://github.com/...
2. #124 — [título] — https://github.com/...
...
```

Se alguma falhar, informe qual e o erro. Pergunte se quer tentar de novo ou ajustar.

---

## O que esta skill NÃO faz

- **Não investiga código.** Isso é da `jvg-investigar-problema`. Se chegar aqui sem análise técnica, pare e oriente o usuário a investigar antes.
- **Não implementa.** Termina com issues publicadas. A implementação vem depois.
- **Não publica nada sem aprovação.** Nenhum "deixa, publica logo" sem confirmação explícita.
- **Não inventa contratos, migrações ou rollouts.** Toda decisão de execução passa pelo usuário antes de virar issue.
- **Não cria dependência entre issues.** Se duas issues precisam ser deployadas juntas, são a mesma issue.

---

## Princípios

- **Uma pergunta por turno** — faça, pare, espere. Nunca acumule perguntas.
- **A decisão é do usuário.** Você propõe, ele decide. A recomendação da análise é informação, não veredito.
- **Vertical slice é regra dura.** Se uma issue não roda sozinha em produção, é a mesma issue de outra.
- **Confirme antes de publicar.** Issue no GitHub é ação visível para o time inteiro — trate como tal.
- **Se não está 100% claro, pergunte.** Uma pergunta a mais agora vale mais que cinco issues ambíguas depois.
- **Pule eixos irrelevantes.** Se a abordagem não mexe em banco, não invente migração. Se não tem rollout faseado, registre "big-bang" e siga.
