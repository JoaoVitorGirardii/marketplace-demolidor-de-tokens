---
name: jvg-gerar-context
description: >
  Analisa o código-fonte do projeto atual e produz um documento de contexto
  (CONTEXT.md) otimizado para ser lido pelo Claude Code em sessões futuras —
  não documentação para humanos, mas um conjunto de regras, contratos e padrões
  que guiam o modelo a gerar código consistente com o projeto. Use esta skill
  sempre que o usuário pedir para documentar o projeto, criar um contexto para a
  IA, entender padrões antes de adicionar features, ou quando disser "quero que
  você entenda meu projeto". Ative também quando o usuário perguntar "como
  adiciono X?", "qual o padrão para Y?", "pode fazer Z?" ou "cria um CONTEXT.md".
---

# Gerar CONTEXT.md

Seu objetivo é produzir um `CONTEXT.md` que, quando carregado como contexto numa
sessão futura do Claude Code, permita ao modelo tomar decisões corretas sobre o
projeto **sem precisar reler todos os arquivos**. O documento é para a IA, não para
o humano — escreva em estilo imperativo, seja específico, cite arquivos e linhas,
e torne as regras inequívocas.

---

## Como executar a análise

### Passo 1 — Orientação inicial
Leia nesta ordem:
1. `CLAUDE.md` e `README.md` (se existirem) — contexto e intenção do projeto
2. `find . -type f | grep -v '\.git'` — mapa completo de arquivos
3. Arquivos de configuração raiz (`.json`, `.toml`, `.yaml`, `.sh`)

### Passo 2 — Leia cada módulo
Para cada arquivo de código relevante, identifique:
- O que ele faz e o que **só ele** faz
- Entradas e saídas (stdin/stdout, args, env vars, sockets, arquivos)
- Dependências (o que importa, o que chama)
- Invariantes: coisas que, se violadas, quebram o sistema

### Passo 3 — Trace o fluxo de dados
Siga o dado de ponta a ponta:
- Onde entra no sistema
- Quais transformações ocorrem (e em qual módulo)
- Onde é persistido
- Como chega ao consumidor final

### Passo 4 — Extraia padrões e restrições
Procure por:
- Padrões que se repetem em ≥2 lugares (são a convenção)
- Código defensivo incomum (costuma ter razão histórica importante)
- Comentários explicando "por quê" — são os invariantes mais valiosos
- O que está ausente propositalmente (sem deps externas, sem async, etc.)

---

## Estrutura do CONTEXT.md gerado

O documento deve ter exatamente estas seções, nesta ordem:

### `## Arquitetura`
Mapa de módulos num formato que o Claude possa referenciar rapidamente:

```
componente/          função em uma linha
  └── subcomponente  função + contratos de I/O
```

Inclua o fluxo de dados como diagrama ASCII se o sistema tiver múltiplas camadas.

### `## Contratos e formatos de dados`
Schemas explícitos dos payloads, eventos ou estruturas que transitam entre módulos.
Use exemplos reais do código, não abstrações. Ex.:

```json
// Payload padrão de evento (todos os hooks produzem isso)
{
  "id": "1716000000000",
  "timestamp": "2026-05-17T10:00:00.000",
  "event_type": "PostToolUse",
  "session_id": "abc123",
  "tool_name": "Read",
  "data": { ... }
}
```

Se um campo é opcional, diga quando ele aparece. Se tem variantes, mostre todas.

### `## Padrões obrigatórios`
Lista de regras que o modelo **deve** seguir ao modificar ou adicionar código.
Formato:

```
PADRÃO: <nome curto>
ONDE: <arquivo(s) ou contexto>
REGRA: <o que fazer>
MOTIVO: <por que — consequência de violar>
EXEMPLO: <trecho de código real do projeto>
```

Inclua pelo menos:
- Como tratar erros neste projeto
- Como nomear variáveis/funções/eventos
- Como lidar com dependências externas
- Padrões de entrada/saída de cada tipo de módulo

### `## Proibido — nunca faça`
Lista curta e direta do que **não pode ser feito**, com a consequência:

```
NÃO: <ação proibida>
POR QUÊ: <o que quebra>
```

Estas são as armadilhas que um desenvolvedor novo (ou um modelo) cairia sem saber.

### `## Como adicionar novas funcionalidades`
Passo a passo para os casos de extensão mais comuns do projeto. Ex.: "como adicionar
um novo tipo de evento", "como adicionar uma nova rota REST", "como adicionar um novo
hook". Para cada caso:
1. Quais arquivos tocar (e quais **não** tocar)
2. Qual contrato de dados deve ser respeitado
3. Onde registrar/instalar a nova peça
4. O que validar antes de considerar pronto

### `## Decisões de design que não são óbvias`
Escolhas que parecem estranhas mas têm razão — sem essa seção o modelo vai
"corrigir" coisas que não devem ser corrigidas. Ex.:
- "Não usamos biblioteca X porque Y"
- "O hook Z é síncrono por necessidade, não por descuido"
- "O fallback de polling existe para cobrir o caso W"

---

## Diretrizes de escrita para o CONTEXT.md

- **Imperativo e sem ambiguidade**: "Sempre use `sys.exit(0)`", não "normalmente usa-se".
- **Cite arquivos e linhas**: `hooks/capture_event.py:95` é melhor que "no hook de captura".
- **Curto por padrão**: se uma seção passar de ~30 linhas, considere separar em arquivo próprio e referenciar.
- **Sem redundância com o código**: o que já está claro lendo o arquivo não precisa ser repetido — documente o *porquê*, não o *o quê*.
- **Versione junto com o código**: ao final do documento, inclua uma linha `_Gerado em: YYYY-MM-DD — revisar ao alterar módulos principais._`

---

## Auto-revisão antes de entregar

Antes de mostrar ao usuário, releia o `CONTEXT.md` que você gerou e verifique:

- [ ] Cada padrão em `## Padrões obrigatórios` cita um `arquivo:linha` real do projeto — se não consegue citar, foi inventado, remova
- [ ] Cada regra em `## Proibido` tem a consequência declarada ("por quê") — sem isso vira lista de manias
- [ ] Nenhuma seção repete informação que está óbvia no código (a regra é documentar o "porquê", não o "o quê")
- [ ] Toda menção a um arquivo usa o caminho real verificável, não nome aproximado
- [ ] Se uma seção passou de ~30 linhas, foi quebrada em arquivo próprio referenciado

Se algum item falhou, corrija antes de mostrar.

---

## Ao finalizar

1. Mostre o `CONTEXT.md` ao usuário para revisão.
2. Pergunte se há invariantes ou decisões que ele sabe que não ficaram óbvias no código.
3. Pergunte se quer salvar como `CONTEXT.md` na raiz do projeto (ou se prefere outro nome/local).
4. Se o projeto já tiver um `CLAUDE.md`, sugira adicionar uma linha `@CONTEXT.md` nele para que o Claude Code carregue automaticamente.

---

## Quando NÃO usar esta skill

- **Já existe um `CONTEXT.md` recente** — use `/jvg-refinar-context` para atualizar pontualmente em vez de regenerar do zero
- **O projeto ainda não tem código suficiente** (greenfield, só configs iniciais) — gerar contexto agora é especular, não documentar
- **O usuário quer documentação para humanos** (README, guia de onboarding) — esta skill produz contexto para IA, escrita imperativa e densa, não voltada à leitura humana
- **O escopo é só um módulo** específico — esta skill mapeia o sistema inteiro; para um trecho isolado, vale escrever inline em vez de gerar arquivo
