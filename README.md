# Laboratório 4 — Exercício 8: conflito de PR resolvido na interface web

Resolver um conflito textual de Pull Request **pelo editor web do GitHub**, sem terminal, e verificar em qual branch a resolução fica registrada.

> Esta é a tentativa que cumpre o enunciado. A primeira, com os desvios explicados, está em [lab4-ex08-pr-interface](https://github.com/Engenharia-de-Software-com-Devops/lab4-ex08-pr-interface).
>
> **O PR #1 continua aberto de propósito.** O exercício pede para resolver o conflito e **não** integrar o PR.

## Conceito-chave

Quando um PR tem conflito, o botão **Resolve conflicts** do GitHub abre um editor com os mesmos marcadores do terminal:

```text
<<<<<<< feature/titulo-web
titulo=Blog da turma
=======
titulo=Blog do curso
>>>>>>> main
```

Ao clicar em **Commit merge**, o GitHub faz um merge **da `main` para dentro da branch do PR** (`Merge branch 'main' into feature/titulo-web`). Esse commit tem **dois pais** e fica na branch de origem. **A `main` não muda nesse momento.** Integrar o PR na `main` é uma ação separada, feita depois.

## Cenário montado (igual nas duas tentativas)

```bash
git init -b main && printf 'titulo=Blog\n' > pagina.txt
git commit -am "base: titulo"                       # base comum
git switch -c feature/titulo-web
printf 'titulo=Blog da turma\n' > pagina.txt
git commit -am "feat: titulo da turma"              # lado do PR
git push -u origin feature/titulo-web               # PR aberto: feature/titulo-web -> main
git switch main
printf 'titulo=Blog do curso\n' > pagina.txt
git commit -am "main: titulo do curso"              # mesma linha, outro texto
git push origin main                                # o PR passa a ter conflito
```

## Verificação local

```bash
git fetch origin
git switch feature/titulo-web && git merge --ff-only origin/feature/titulo-web
cat pagina.txt                           # texto escolhido na resolução
git rev-list --parents -n 1 HEAD         # <merge> <pai 1: feat> <pai 2: main>, dois pais
git show origin/main:pagina.txt          # a main deve continuar com "titulo=Blog do curso"
```

## Resolução feita na interface

1. Abrir o PR e confirmar o aviso *This branch has conflicts that must be resolved* em `pagina.txt`.
2. **Resolve conflicts**: substituir todo o bloco conflitante por `titulo=Blog da turma e do curso`, sem marcadores.
3. **Mark as resolved**, depois **Commit merge**.
4. Voltar ao PR: o conflito sumiu. **Não** clicar em *Merge pull request*.

## Resultado

```text
*   def5254 Merge branch 'main' into feature/titulo-web   <- resolução (branch do PR)
|\
| * 4ab9eb9 main: titulo do curso                         <- main continua aqui
* | 65bb2a1 feat: titulo da turma
|/
* 198baaa base: titulo
```

| Critério de conclusão | Verificação | Resultado |
|---|---|---|
| PR sem conflito de integração | Página do PR | Sem conflito; PR aberto |
| Texto combinado na branch de origem | `git show origin/feature/titulo-web:pagina.txt` | `titulo=Blog da turma e do curso` |
| Commit de resolução com dois pais | `git rev-list --parents -n 1 def5254` | pais `65bb2a1` (feat) e `4ab9eb9` (main) |
| `main` intacta | `git show origin/main:pagina.txt` | `titulo=Blog do curso` |

> Detalhe: na resolução ficou uma linha em branco antes do texto em `pagina.txt`. Isso não afeta os critérios, mas o ideal seria apagar essa linha no editor.

## Pergunta do enunciado

**Clicar em _Commit merge_ conclui o PR na `main`?**
Não. O editor integra a branch de base (`main`) na branch de origem (`feature/titulo-web`) e registra a resolução ali. Integrar o PR na `main` é uma ação posterior e separada.

## Revisão rápida
- [ ] Sei que a resolução pela web cria um commit de merge **na branch do PR**.
- [ ] Sei verificar os dois pais com `git rev-list --parents -n 1`.
- [ ] Sei que *resolver o conflito* e *integrar o PR* são passos diferentes.
- [ ] Sei quando preferir resolver localmente: conflitos grandes, código que precisa de teste, arquivos binários ou renomeados.

## Referências
- [GitHub — Resolving a merge conflict on GitHub](https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/resolving-a-merge-conflict-on-github)
