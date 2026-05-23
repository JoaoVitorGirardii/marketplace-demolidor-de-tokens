---
name: merge-from-master
description: Traz todas as alterações da branch principal (master ou main) para a branch atual via merge. Use quando quiser sincronizar sua branch com o master/main. Se houver conflitos, apenas informa quais arquivos conflitaram sem tentar resolver — o merge só é finalizado se não houver conflitos.
---

## Fluxo

1. Verificar a branch atual
2. Detectar qual é a branch principal: testar `origin/master` e `origin/main` (usar a que existir)
3. Fazer fetch da origin para atualizar refs remotas
4. Tentar merge da branch principal na branch atual
5. **Se conflitos:** abortar o merge (`git merge --abort`) e listar os arquivos conflitantes para o usuário
6. **Se sem conflitos:** merge finalizado com sucesso

## Execução

```bash
# Salvar branch atual
CURRENT=$(git branch --show-current)

# Detectar branch principal
if git ls-remote --exit-code origin master &>/dev/null; then
  BASE_BRANCH="master"
elif git ls-remote --exit-code origin main &>/dev/null; then
  BASE_BRANCH="main"
else
  echo "Nenhuma branch principal encontrada (master ou main) no remote origin."
  exit 1
fi

# Fetch para atualizar remote
git fetch origin $BASE_BRANCH

# Tentar merge sem commit automático para inspecionar resultado
git merge origin/$BASE_BRANCH --no-edit
```

Se o comando retornar código de saída diferente de 0 ou listar conflitos em `git status`:

```bash
# Listar arquivos em conflito
git diff --name-only --diff-filter=U

# Abortar — nunca resolver conflitos automaticamente
git merge --abort
```

Informe ao usuário: "Conflitos detectados nos seguintes arquivos: [lista]. Merge abortado. Resolva manualmente e faça o merge novamente."

Se sem conflitos, informe: "Merge concluído sem conflitos. Branch `[CURRENT]` atualizada com `[BASE_BRANCH]`."

## Regras

- **Nunca** resolver conflitos automaticamente
- **Nunca** fazer `git push` após o merge
- Detectar automaticamente se a branch principal é `master` ou `main` no remote
- Se não existir no remote, tentar localmente (`master` ou `main`)
- Sempre executar `git fetch` antes do merge para garantir que o remote está atualizado
