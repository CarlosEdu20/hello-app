# CI/CD com o Github Actions

## Objetivo:
Nos tempos atuais, empresas de todos os portes estão adotando práticas de automação no ciclo de desenvolvimento, conhecidas como CI/CD (Integração Contínua e Entrega Contínua). O objetivo é simples: entregar código com rapidez, segurança e consistência. 

Neste cenário, ferramentas como **GitHub Actions** e **ArgoCD** ganham destaque:
- O **GitHub Actions** permite automatizar o build, os testes e a publicação de
imagens Docker diretamente a partir dos commits.

- O **ArgoCD**, por sua vez, implementa o conceito de **GitOps**, onde o próprio Git é a
“fonte de verdade” da infraestrutura e dos deploys em Kubernetes.

Com isso, este projeto busca automatizar o ciclo completo de desenvolvimento, build, deploy e execução de uma aplicação FastAPI simples, usando GitHub Actions para CI/CD, Docker Hub como registry, e ArgoCD para entrega contínua em Kubernetes local com Rancher Desktop.

## Pré-requisitos:
Estes são os pré-requistos para fazer este projeto funcionar:

- Conta no GitHub (repo público)
- Conta no Docker Hub com token de acesso
- Rancher Desktop com Kubernetes habilitado
- kubectl configurado corretamente (kubectl get nodes)
- ArgoCD instalado no cluster local
- Git instalado
- Python 3 e Docker instalados

## Tecnologias usadas:
As tecnologias que fazem este projeto funcionar de maneira adequada são essas:

- Docker
- Python 3
- FastAPI
- Uvicorn
- Kubernetes
- K3D (um tipo de distribuição kubernetes)
- Github e Git
- Rancher Desktop
- ArgoCD

## Etapa 1: Estrutura do projeto e criação dos arquivos
Nesta primeira etapa, é onde será preparada todos os arquivos que serão a fundação deste projeto. Adotando a prática GitOps, a estrutura foi dividida em dois repositórios Git distintos com responsabilidades diferentes; o primeiro contém o código-fonte da aplicação e a pipeline de CI/CD, enquanto o segundo armazena os manifestos Kubernetes que definem o estado desejado da aplicação para o ArgoCD.

### 1.1. Criação dos Repositórios no GitHub
A metodologia GitOps separa o código da aplicação da configuração do ambiente. Para isso, é uma boa prática criar dois repositórios:

- hello-app: Repositório para o código-fonte da aplicação, Dockerfile e o workflow de CI/CD.
- hello-manifests: Repositório para os manifestos Kubernetes que descrevem o estado desejado da aplicação no cluster.

Como os repositórios estão separados, acesse esse link https://github.com/CarlosEdu20/hello-manifests.

### 1.2. Desenvolvimento da Aplicação
Após criar os repositórios, clonamos o `hello-app` localmente para adicionar os arquivos da aplicação. Para isso, usaremos o seguinte comando.

```
git clone https://github.com/SEU_USUARIO/hello-app.git
cd hello-app
```

### 1.3. Código da Aplicação
Crie um arquivo chamado `main.py` com um endpoint simples que retorna uma mensagem "Hello World". O código do arquivo está disponível neste repositório.

### 1.4. Dependências
Crie um arquivo chamado `requirements.txt`, nele ficarão as bibliotecas Python necessárias para a aplicação rodar. As bibleotecas estão presente nestes reposítório.

### 1.5. Containerização 
Crie um arquivo Dockerfile para definir a receita de construção da imagem Docker. Este arquivo instrui o Docker a copiar o código, instalar as dependências e definir o comando para executar a aplicação. O script do Dockerfile está logo acima

### 1.6. Enviando tudo para o Github
Com todos os arquivos já criados, envie para o repositório remoto no Github usando os seguintes comandos:

```
git add .
git commit -m "feat: initial application setup"
git push origin main
```

Portanto, a configuração inicial do projeto está concluída.

## Etapa 2: Automação com GitHub Actions (CI/CD)
Com a base da aplicação criada, o próximo passo é automatizar o processo de build e a proposta de deploy. Para isso, o github actions será implementado, o mesmo funcionará como um servidor de Integração Contínua (CI). O objetivo desta etapa é criar um workflow que, a cada push na branch principal, automaticamente constrói uma nova imagem Docker e abre um Pull Request para atualizar a versão no repositório de manifestos.



  
