# Guia Definitivo: Limpando Manifestos Kubernetes com Krew e Neat

Este repositório é um guia prático e direto ao ponto para instalar o **Krew** (gerenciador de plugins do kubectl) e o plugin **Neat** (`kubectl-neat`). 

Se você trabalha com Kubernetes, sabe que rodar um `kubectl get pod <nome> -o yaml` retorna um manifesto gigante, cheio de metadados gerados automaticamente pelo cluster (como `managedFields`, `creationTimestamp`, `status`, etc.). O `neat` limpa todo esse "ruído", devolvendo apenas o que realmente importa.

---

## Índice

- [Pré-requisitos](#-pré-requisitos)
- [Passo 1: Instalando o Krew](#-passo-1-instalando-o-krew)
- [Passo 2: Configurando o PATH](#-passo-2-configurando-o-path)
- [Passo 3: Instalando o Neat](#-passo-3-instalando-o-neat)
- [Como usar o Neat na prática](#-como-usar-o-neat-na-prática)
- [Troubleshooting](#-troubleshooting)
- [Contribuindo](#-contribuindo)

---

## Pré-requisitos

Antes de começar, certifique-se de ter instalado em sua máquina:
* `kubectl` configurado e funcional.
* `git`
* `curl`

---

## Passo 1: Instalando o Krew

O Krew funciona como o `apt` ou `brew`, mas é exclusivo para plugins do `kubectl`. Para instalá-lo no Linux ou macOS, abra seu terminal e execute o bloco abaixo:

```bash
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "[https://github.com/kubernetes-sigs/krew/releases/latest/download/$](https://github.com/kubernetes-sigs/krew/releases/latest/download/$){KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)
```
## Passo 2: Configurando o PATH
Para que o seu terminal reconheça os comandos do Krew, é obrigatório adicionar o diretório dele ao seu PATH.

Se você usa Bash (padrão na maioria do Linux):

```Bash
echo 'export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Se você usa Zsh (padrão no macOS):

```Bash
echo 'export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

Verifique a instalação:

```Bash
kubectl krew version
```

## Passo 3: Instalando o plugin Neat
Com o Krew configurado, instalar plugins fica extremamente simples. Execute:

```Bash
kubectl krew install neat
```

Você pode verificar se ele foi instalado corretamente listando seus plugins:

```Bash
kubectl krew list
```

## Como usar o Neat na prática
A forma mais comum de utilizar o Neat é através de pipes (|) junto com seus comandos tradicionais do kubectl.

Exemplo 1: Visualização limpa
Imagine que você quer extrair o YAML de um deployment para entender como ele foi feito. Em vez de trazer dezenas de linhas de metadados inúteis, use:

```Bash
kubectl get deployment shrek-deployment -n pantano -o yaml | kubectl neat
```

Exemplo 2: Exportando para arquivo
Se quiser salvar um ConfigMap limpo, pronto para ser commitado ou aplicado em outro cluster:

```Bash
kubectl get deployment shrek-deployment -n pantano -o yaml | kubectl neat > shrek-deployment-limpo.yaml
```
O resultado será um YAML enxuto, sem managedFields, uid, resourceVersion ou status.

## Veja o resultado nos arquivos do repositório
O arquivo shrek-deployment-without-neat.yaml é a extração de um YAML sem o uso do Kubectl neat. Com ele vem diversas sujeiras que não serão utilizados.

O arquivo shrek-deployment-with-neat.yaml é a extração de um YAML com o uso do Kubectl neat. Com ele vem somente o necessário para criar um novo deployment ou qualquer objeto no Kubernetes.

## Troubleshooting
Erro: unknown command "krew" for "kubectl"
Isso significa que o seu terminal não encontrou o Krew. Volte ao Passo 2, certifique-se de que exportou o PATH corretamente e recarregou o terminal (source).