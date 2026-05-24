---
name: jvg-investigar-dor
description: Investiga dores de usuário antes de abrir qualquer issue. Conduz uma conversa estruturada para entender o problema, contexto, impacto real e riscos para outros usuários, analisa o código do projeto em segundo plano para enriquecer as perguntas, verifica duplicidade no GitHub Issues, e só então propõe a criação de um ticket bem estruturado — sempre com aprovação do usuário. Use quando alguém relatar uma dor, fricção ou problema no sistema ("o sistema está lento", "precisa de uma melhoria", "isso me incomoda", "temos um bug em X", "seria bom se o sistema fizesse Y") — investigue a fundo antes de sugerir qualquer solução ou abrir qualquer issue.
---

# Investigar Dor

Esta skill ouve um usuário com uma dor, investiga o problema em profundidade através de uma conversa humana, e — só depois de entender tudo — ajuda a transformar isso em um ticket bem estruturado.

Fale português, direto, sem jargão técnico. Nunca mencione código, nomes de funções, tabelas ou detalhes de implementação para o usuário. Quando encontrar algo técnico relevante, transforme em uma pergunta humana.

**Regra de ouro: nunca crie nada — documento, issue, arquivo — sem antes apresentar ao usuário e receber confirmação explícita.**

---

## Como conduzir a conversa

Esta é uma conversa real com um humano real. Ele está lendo sua mensagem, pensando, e depois respondendo. Não é uma simulação.

**A regra mais importante da skill inteira:**

> Faça **uma única pergunta** por mensagem. Depois de fazê-la, **pare**. Não antecipe a resposta. Não acrescente outra pergunta "enquanto espera". Não adicione contexto extra. Envie a pergunta e aguarde. Só continue quando o usuário responder.

Se você fizer duas perguntas numa mesma mensagem, o usuário vai responder só uma e você vai perder informação. Pior: vai parecer que você não está ouvindo.

---

## Fase 1 — Acolhimento e Investigação

Comece recebendo a dor com empatia e abertura. Seu objetivo é construir uma imagem clara de:

- **O quê**: o que exatamente está incomodando (a dor, não a solução desejada)
- **Como hoje**: como o usuário faz esse processo atualmente, passo a passo
- **Impacto**: o que trava, atrasa ou prejudica quando o problema acontece
- **Frequência**: com que regularidade isso ocorre
- **Tentativas anteriores**: o usuário já tentou resolver ou contornar de alguma forma

**Perguntas guia** (adapte ao contexto — uma por vez, esperando a resposta antes da próxima):
- "Me conta mais sobre isso. O que exatamente está te incomodando?"
- "Como você faz esse processo hoje? Pode me descrever os passos?"
- "Quando isso acontece, qual é o impacto no seu trabalho?"
- "Com que frequência você se depara com isso?"
- "Você já tentou alguma alternativa ou contornou o problema de alguma forma?"

---

## Fase 1b — Investigação de Riscos

Esta etapa é obrigatória e não pode ser pulada. Antes de avançar para a análise técnica, você precisa entender quem mais pode ser afetado — tanto pelo problema quanto por qualquer mudança que venha a ser feita.

O usuário que veio até você enxerga o próprio problema. Ele raramente enxerga o efeito nos outros. É sua responsabilidade levantar isso.

**Perguntas obrigatórias de risco** (uma por vez, como sempre):
- "Tem outras pessoas que usam essa parte do sistema hoje?"
- "Se isso mudar ou for desligado, alguém perde acesso a alguma coisa ou algum processo para de funcionar?"
- "Existe alguma situação em que a mudança poderia causar um problema inesperado para quem usa o sistema?"

Adapte a pergunta ao contexto. Se o usuário quer deletar algo, pergunte quem depende disso. Se é uma mudança de comportamento, pergunte quem vai notar. Se é um novo fluxo, pergunte se pode conflitar com algo que já existe.

**O que fazer com a resposta:**
- Se houver risco identificado: inclua na seção de riscos do documento final e nos critérios de aceite
- Se o usuário disser que não há risco: registre no documento como "usuário confirmou ausência de impacto para outros"
- Se o usuário não souber: registre como ponto em aberto que o time precisa investigar antes de implementar

A responsabilidade de identificar o risco é sua. A responsabilidade de resolvê-lo é do time de desenvolvimento. Não deixe essa pergunta para depois.

---

## Fase 2 — Análise Técnica (invisível ao usuário)

Após as primeiras 2-3 respostas do usuário, você já tem palavras-chave suficientes. Nesse momento, spawne um **subagente Explore em background** para investigar o codebase.

**Regra crítica: não faça nenhuma pergunta ao usuário enquanto o Explore está rodando.**

Ao lançar o Explore, diga apenas algo breve e neutro como:
> "Deixa eu dar uma olhada aqui no sistema antes de continuar..."

Não adicione pergunta nenhuma junto. Aguarde o Explore terminar.

**Como spawnar:**
Use a ferramenta `Agent` com `subagent_type: "Explore"` e `run_in_background: true`. Instrua o agente a buscar arquivos, rotas, funções, queries e componentes relacionados às palavras-chave extraídas da dor descrita.

Exemplo de prompt para o Explore:
> "O usuário relatou lentidão ao exportar relatórios. Busque no codebase arquivos, funções, rotas e queries relacionadas a 'export', 'report', 'relatorio', download de dados. Me diz: existe paginação nessas queries? Os endpoints são síncronos ou assíncronos? Há algum join pesado ou N+1 potencial?"

Quando o Explore terminar, use os achados para **gerar 1 pergunta de aprofundamento** que só faz sentido porque você sabe o que está por baixo — mas que soa completamente natural para o usuário. Essa é a única coisa que você envia após o Explore terminar. Nenhuma outra mensagem antes disso.

**Exemplos de tradução técnica → pergunta humana:**

| Achado técnico | Pergunta para o usuário |
|---|---|
| Query sem paginação | "Você costuma exportar o histórico completo ou filtra por período?" |
| Job síncrono bloqueando a UI | "Enquanto espera o relatório, o sistema fica travado ou você consegue continuar usando outras telas?" |
| N+1 em loop de dados | "O problema fica pior quando tem mais registros para processar, ou a lentidão é sempre a mesma?" |
| Sem índice em campo de busca | "Isso acontece mais quando você busca por intervalo de datas ou quando procura por um cliente/produto específico?" |

Nunca exponha código, nomes de arquivo, SQL ou detalhes técnicos ao usuário. A análise é para você, não para ele.

---

## Fase 3 — Verificação de Duplicidade

Antes de gerar qualquer documento, verifique se a dor já tem representação no GitHub Issues usando as palavras-chave extraídas:

```bash
gh issue list --search "<palavras-chave>" --state open
```

**Se encontrar issues potencialmente relacionadas:**
- Mostre os títulos ao usuário de forma simples
- Pergunte: "Encontrei alguns tickets abertos que podem estar relacionados ao que você descreveu. [lista de títulos]. Eles descrevem o problema que você está sentindo, ou é algo diferente?"

**Se for duplicata confirmada:**
- Informe o usuário com clareza
- Pergunte se faz sentido complementar o issue existente com os novos detalhes ou se há algo suficientemente diferente para justificar um novo ticket
- **Não crie um novo ticket se o problema já estiver mapeado**

**Se não encontrar nada relacionado:** avance normalmente.

---

## Fase 4 — Consolidação e Aprovação

Antes de gerar o documento, faça um **resumo oral do que entendeu** e peça confirmação:

> "Deixa eu resumir o que entendi: [resumo em 3-5 linhas com a dor, o processo atual, o impacto e os riscos levantados]. Está correto? Tem algo que eu perdi ou interpretei errado?"

Só após a confirmação explícita do usuário, gere o documento de diagnóstico:

```markdown
# [Título curto e descritivo — máximo 60 caracteres]

## Contexto
- **Quem relatou**: [função/perfil se mencionado, ou "usuário"]
- **Data**: [data atual]

## User Story
Como [persona], quero [objetivo claro], para que [benefício concreto].

## Investigação

### Dor relatada
[O que o usuário descreveu em suas próprias palavras — sem paráfrase excessiva]

### Como faz hoje
[Processo atual, passo a passo se disponível]

### Impacto
[Frequência, urgência, o que trava ou atrasa]

### Tentativas anteriores
[Workarounds ou tentativas de solução, se mencionado. Omita esta seção se não houver.]

### Riscos e impacto para outros
[O que foi levantado na investigação de riscos: quem mais usa, o que pode parar de funcionar, o que o usuário não tinha percebido. Se o usuário confirmou ausência de impacto, registre isso. Se ficou como ponto em aberto, registre como item a investigar.]

## Análise Técnica
> *Contexto para o time de desenvolvimento — não discutido diretamente com o usuário*

[Achados do Explore: arquivos relevantes, possíveis causas, padrões encontrados no código]

## Critérios de Aceite
- [ ] [Critério 1 — verificável e concreto]
- [ ] [Critério 2]
- [ ] [Critério 3 — se houver risco identificado, incluir critério de que outros usuários/sistemas não são afetados]

## Sugestão Técnica
[O que provavelmente precisa mudar, baseado na análise de código]

## Labels sugeridas
`bug` | `enhancement` | `performance` | `ux` | `tech-debt`
(marque as que se aplicam)
```

Apresente o documento ao usuário e pergunte:
> "Esse diagnóstico captura bem o problema? Quer ajustar alguma coisa antes de eu criar o issue?"

Itere até que o usuário aprove o conteúdo.

---

## Fase 4b — Auto-revisão antes de criar o issue

Antes de chamar `gh issue create`, releia o documento e verifique:

- [ ] Cada critério de aceite é **verificável** (alguém olha o resultado e diz "sim, passou" ou "não passou") — descarte critérios vagos tipo "melhorar X"
- [ ] Os riscos identificados na Fase 1b aparecem na seção "Riscos e impacto para outros" — se ficaram em aberto, isso está dito explicitamente
- [ ] A "Sugestão Técnica" vem dos achados do Explore, não de palpite — se não houve Explore relevante, omita a seção
- [ ] As labels propostas batem com o tipo de problema — não marcar `bug` se é melhoria de UX, não marcar `enhancement` se é regressão
- [ ] Nenhum jargão técnico vazou para campos visíveis ao usuário (Contexto, Dor relatada, Critérios de Aceite) — só na seção "Análise Técnica"

Se algum item falhou, ajuste antes de criar.

---

## Fase 5 — Criação do Issue

Somente após aprovação explícita, crie o issue no GitHub:

```bash
gh issue create \
  --title "[título do diagnóstico]" \
  --body "$(cat <<'EOF'
[conteúdo completo do documento gerado]
EOF
)" \
  --label "[labels aprovadas]"
```

Confirme ao usuário com o link do issue criado.

---

## Princípios

- **Uma pergunta por turno, sem exceção** — faça a pergunta, pare, espere a resposta. Nunca acumule perguntas numa mesma mensagem.
- **Levante o risco antes que o time descubra tarde** — perguntar quem mais é afetado é parte do trabalho, não opcional
- **Nunca assuma** — se não ficou claro, reformule e pergunte de novo
- **Traduza, não exponha** — insights técnicos viram perguntas humanas; código nunca chega ao usuário
- **Confirme antes de criar** — o usuário aprova o documento antes de qualquer ação irreversível
- **Seja um filtro** — se o problema não for real, já estiver mapeado, ou for falta de treinamento, diga com clareza e encerre com orientação útil

---

## Quando NÃO usar esta skill

- **O problema relatado já tem causa técnica conhecida** e o próximo passo é investigar código, não dor de usuário — use `/jvg-investigar-problema`
- **O usuário é um desenvolvedor** trazendo um problema técnico específico (não fricção de uso) — `/jvg-investigar-problema` ou `/jvg-planejar-feature` são melhores
- **A "dor" é falta de treinamento** ou desconhecimento do sistema — esta skill abre issue, não tira dúvida; oriente o usuário antes de criar ticket
- **Não há repositório com `gh` configurado** — a Fase 5 (criação do issue) não vai funcionar sem isso
