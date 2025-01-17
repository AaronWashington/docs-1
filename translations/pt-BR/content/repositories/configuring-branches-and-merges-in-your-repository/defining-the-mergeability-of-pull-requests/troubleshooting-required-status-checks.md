---
title: Solução de problemas de verificações de status necessárias
intro: Você pode verificar erros comuns e resolver problemas com as verificações de status necessárias.
product: '{% data reusables.gated-features.protected-branches %}'
versions:
  fpt: '*'
  ghes: '*'
  ghae: '*'
  ghec: '*'
topics:
  - Repositories
redirect_from:
  - /github/administering-a-repository/troubleshooting-required-status-checks
  - /github/administering-a-repository/defining-the-mergeability-of-pull-requests/troubleshooting-required-status-checks
shortTitle: Verificações de status necessárias
---

Se você tiver uma verificação e um status com o mesmo nome e selecionar esse nome como uma verificação de status obrigatória, a verificação e o status serão obrigatórios. Para obter mais informações, consulte "[Verificações](/rest/reference/checks)".

Depois que você habilitar as verificações de status solicitadas, seu branch pode precisar estar atualizado com o branch de base antes da ação de merge. Isso garante que o branch foi testado com o código mais recente do branch base. Se o branch estiver desatualizado, você precisará fazer merge do branch base no seu branch. Para obter mais informações, consulte "[Sobre branches protegidos](/github/administering-a-repository/about-protected-branches#require-status-checks-before-merging)".

{% note %}

**Observação:** também é possível atualizar o seu branch com o branch base usando o rebase do Git. Para obter mais informações, consulte "[Rebase no Git](/github/getting-started-with-github/about-git-rebase)".

{% endnote %}

Não será possível fazer push de alterações locais em um branch protegido enquanto todas as verificações de status obrigatórias não forem aprovadas. Sendo assim, você receberá uma mensagem de erro semelhante a esta.

```shell
remote: error: GH006: Protected branch update failed for refs/heads/main.
remote: error: Required status check "ci-build" is failing
```
{% note %}

**Observação:** as pull requests que são atualizadas e passam nas verificações de status obrigatórias podem sofrer merge localmente e enviadas por push para o branch protegido. Isso pode ser feito sem verificações de status em execução no próprio commit de merge.

{% endnote %}

{% ifversion fpt or ghae or ghes or ghec %}

## Conflitos entre o título do commit e o commit de merge do teste

Por vezes, os resultados das verificações de status para o commit de mescla teste e o commit principal entrarão em conflito. Se o commit de merge de testes tem status, o commit de merge de testes deve passar. Caso contrário, o status do commit principal deve passar antes de você poder mesclar o branch. Para obter mais informações sobre commits de merge de teste, consulte "[Pulls](/rest/reference/pulls#get-a-pull-request)".

![Branch com commits de mescla conflitantes](/assets/images/help/repository/req-status-check-conflicting-merge-commits.png)
{% endif %}

## Manipulação ignorada, mas verificações necessárias

{% note %}

**Note:** If a workflow is skipped due to [path filtering](/actions/using-workflows/workflow-syntax-for-github-actions#onpushpull_requestpull_request_targetpathspaths-ignore), [branch filtering](/actions/using-workflows/workflow-syntax-for-github-actions#onpull_requestpull_request_targetbranchesbranches-ignore) or a [commit message](/actions/managing-workflow-runs/skipping-workflow-runs), then checks associated with that workflow will remain in a "Pending" state. A pull request that requires those checks to be successful will be blocked from merging.

If a job in a workflow is skipped due to a conditional, it will report its status as "Success". For more information see [Skipping workflow runs](/actions/managing-workflow-runs/skipping-workflow-runs) and [Using conditions to control job execution](/actions/using-jobs/using-conditions-to-control-job-execution).

{% endnote %}

### Exemplo

The following example shows a workflow that requires a "Successful" completion status for the `build` job, but the workflow will be skipped if the pull request does not change any files in the `scripts` directory.

```yaml
name: ci
on:
  pull_request:
    paths:
      - 'scripts/**'
jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [12.x, 14.x, 16.x]
    steps:
    - uses: {% data reusables.actions.action-checkout %}
    - name: Use Node.js {% raw %}${{ matrix.node-version }}{% endraw %}
      uses: {% data reusables.actions.action-setup-node %}
      with:
        node-version: {% raw %}${{ matrix.node-version }}{% endraw %}
        cache: 'npm'
    - run: npm ci
    - run: npm run build --if-present
    - run: npm test
```

Due to [path filtering](/actions/using-workflows/workflow-syntax-for-github-actions#onpushpull_requestpull_request_targetpathspaths-ignore), a pull request that only changes a file in the root of the repository will not trigger this workflow and is blocked from merging. Você verá o seguinte status no pull request:

![Verificação obrigatória ignorada mas mostrada como pendente](/assets/images/help/repository/PR-required-check-skipped.png)

Você pode corrigir isso criando um fluxo de trabalho genérico, com o mesmo nome, que retornará verdadeiro em qualquer caso semelhante ao fluxo de trabalho abaixo:

```yaml
name: ci
on:
  pull_request:
    paths-ignore:
      - 'scripts/**'
      - 'middleware/**'
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: 'echo "No build required" '
```
Agora, as verificações sempre passarão sempre que alguém enviar uma solicitação pull que não altere os arquivos listados em `caminhos` no primeiro fluxo de trabalho.

![Verificação ignorada mas passa graças a um fluxo de trabalho genérico](/assets/images/help/repository/PR-required-check-passed-using-generic.png)

{% note %}

**Notas:**
* Certifique-se de que a chave `nome` e o nome do trabalho necessário sejam o mesmo em ambos os arquivos do fluxo de trabalho. Para obter mais informações, consulte "[Sintaxe do fluxo de trabalho para {% data variables.product.prodname_actions %}](/actions/reference/workflow-syntax-for-github-actions)".
* O exemplo acima usa {% data variables.product.prodname_actions %}, mas esta solução alternativa também se aplica a outros provedores de CI/CD que se integram a {% data variables.product.company_short %}.

{% endnote %}

{% ifversion fpt or ghes > 3.3 or ghae-issue-5379 or ghec %}Também é possível que um branch protegido exiga uma verificação de status de um {% data variables.product.prodname_github_app %} específico. Se você vir uma mensagem parecida com a seguinte, você deverá verificar se a verificação listada na caixa de merge foi definida pelo aplicativo esperado.

```
A "criação" da verificação de status necessária não foi definida pelo {% data variables.product.prodname_github_app %} esperado.
```
{% endif %}
