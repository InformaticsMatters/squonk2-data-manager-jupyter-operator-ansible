# Ansible playbooks for the Squonk2 Jupyter Operator

[![lint](https://github.com/informaticsmatters/squonk2-data-manager-jupyter-operator-ansible/actions/workflows/lint.yaml/badge.svg)](https://github.com/informaticsmatters/squonk2-data-manager-jupyter-operator-ansible/actions/workflows/lint.yaml)

![GitHub](https://img.shields.io/github/license/informaticsmatters/squonk2-data-manager-jupyter-operator-ansible)

![GitHub tag (latest SemVer pre-release)](https://img.shields.io/github/v/tag/informaticsmatters/squonk2-data-manager-jupyter-operator-ansible)?include_prereleases)

[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow.svg)](https://conventionalcommits.org)
[![pre-commit](https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit&logoColor=white)](https://github.com/pre-commit/pre-commit)

This repo contains Ansible playbooks for the Squonk2 Jupyter operator.

## Contributing
The project uses: -

- [pre-commit] to enforce linting of files prior to committing them to the
  upstream repository
- [Commitizen] to enforce a [Conventional Commit] commit message format

You **MUST** comply with these choices in order to  contribute to the project.

To get started review the pre-commit utility and the conventional commit style
and then set-up your local clone by following the **Installation** and
**Quick Start** sections: -

    pip install pre-commit
    pre-commit install -t commit-msg -t pre-commit

Now the project's rules will run on every commit, and you can check the
current health of your clone with: -

    pre-commit run --all-files

## Deploying into the Data Manager API
We use [Ansible] 3 and community modules in [Ansible Galaxy] as the deployment
mechanism, using the `operator` Ansible role in this repository and a
Kubernetes config (KUBECONFIG). All of this is done via a suitable Python
environment using the requirements in the root of the project...

    python -m venv venv
    source venv/bin/activate
    pip install --upgrade pip
    pip install -r requirements.txt

Set your KUBECONFIG for the cluster and verify its right: -

    export KUBECONFIG=~/k8s-config/local-config
    kubectl get no
    [...]

Now, create a parameter file (i.e. `parameters.yaml`) based on the project's
`example-parameters.yaml`, setting values for the operator that match your
needs. Then deploy, using Ansible, from the root of the project: -

    export PARAMS=parameters
    ansible-playbook -e @${PARAMS}.yaml site.yaml

That deploys the operator and its CRD to your chosen operator namespace.
To deploy the Data Manager RBAC and Jupyter notebook configuration objects
you need to run the `site_dm.yaml` playbook: -

    ansible-playbook -e @${PARAMS}.yaml site_dm.yaml

>   If deploying to multiple Data Managers you should just need one operator
    and then deploy RBACs to each DM namespace. Remember to also adjust the
    annotations of for CRD so each DM namespace recognises it as a valid
    application.

To remove the operator (assuming there are no operator-derived instances)...

    ansible-playbook -e @${PARAMS}.yaml -e jo_state=absent site.yaml

>   The current Data Manager API assumes that once an Application (operator)
    has been installed it is not removed. So, removing the operator here
    is described simply to illustrate a 'clean-up' - you would not
    normally remove an Application operator in a production environment.

### Deploying to the official cluster
The parameters used to deploy the operator to our 'official' cluster
are held in this repository.

To deploy the operator itself run the main 'site' playbook with
a suitable set of parameters: -

    export KUBECONFIG=~/k8s-config/config-aws-im-main-eks
    export PARAMS=staging
    ansible-playbook -e @${PARAMS}-parameters.yaml site.yaml

Then, you must run the `site_dm` playbook to for each Data Manager
you wish to configure: -

    ansible-playbook -e xchem-dev-integration-parameters.yaml site_dm.yaml

    ansible-playbook -e xchem-dev-test-parameters.yaml site_dm.yaml

This will install the RBAC and configuration objects for Jupyter
to the corresponding DM namespaces.

---

[ansible]: https://www.ansible.com
[certificate manager]: https://cert-manager.io/docs/installation/kubernetes/
[commitizen]: https://commitizen-tools.github.io/commitizen/
[conventional commit]: https://www.conventionalcommits.org/en/v1.0.0/
[pre-commit]: https://pre-commit.com
