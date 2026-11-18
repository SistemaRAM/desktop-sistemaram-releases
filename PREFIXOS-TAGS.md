# Prefixos de tag aceitos pelo workflow `Release Manifest`

O workflow `.github/workflows/release-manifest.yml` lê o nome da tag e, pelo **prefixo**, escolhe em qual arquivo JSON a release será registrada.

## Regra de detecção

```bash
^(manager|comandas|instaladores|drivers|add-?ons)\.(.+)$
```

- Formato: `<prefixo>.<versão>` (o ponto após o prefixo é obrigatório e precisa ter algo depois).
- Diferencia maiúsculas de minúsculas (use tudo minúsculo).
- Tag que não casa com nenhum prefixo cai no manifesto padrão `releases.json`.

## Prefixos

| Prefixo | Arquivo gerado | Exemplo de tag | Uso |
|---|---|---|---|
| *(nenhum)* | `releases.json` | `18.0.3.11` | Sistema Desktop (RAMSolution), versão `YEAR.MAJOR.MINOR.PATCH` |
| `manager.` | `releases-manager.json` | `manager.v1.2.3` | RAManager |
| `comandas.` | `releases-comandas.json` | `comandas.1.0.4` | App/módulo de Comandas |
| `instaladores.` | `releases-instaladores.json` | `instaladores.2.0.0` | Instaladores |
| `drivers.` | `releases-drivers.json` | `drivers.1.1.0` | Drivers |
| `add-ons.` ou `addons.` | `releases-addons.json` | `add-ons.1.0.0` | Add-ons (o RAManager os localiza por esse padrão) |

> Os usos de cada produto são inferidos pelo nome; ajuste a descrição se algum estiver diferente.

## O que acontece quando a tag é criada

1. Gatilhos: `create` (tag nova), `release` (`released`/`edited`) ou `workflow_dispatch` (informando a tag manualmente).
2. O prefixo define o arquivo-alvo.
3. O job espera a release existir (10 tentativas, 15 s) e coleta nome, link, descrição e anexos. Se não achar, grava com valores padrão e `anexos: []`.
4. Grava/atualiza a entrada `.[<tag>]` no JSON e faz commit `chore: add <tag> to <arquivo>` na branch padrão.

## Atenções

- **Tag com `v` na frente** (`v18.0.3.1`, como sugere o README) **não** é prefixo de produto: vai para `releases.json` com a chave `v18.0.3.1`. As tags reais do sistema desktop usam o número puro (`18.0.3.11`).
- Prefixo errado ou com maiúscula (`Manager.1.0.0`, `driver.1.0.0`) vai silenciosamente para `releases.json`.
- Para reprocessar uma tag: Actions → *Release Manifest* → *Run workflow* → informar a tag completa, com prefixo.
- Para criar um novo produto, é preciso editar a regex e o `case` do workflow.
