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

### 2.1. Crie uma conta no Docker hub
O Docker Hub será o Container Registry, o local onde as imagens da aplicação serão hospedadas. Caso ainda não tenha, crie uma conta gratuita. Após o login, anote seu nome de usuário, pois o mesmo será importante nos próximos passos.

### 2.2. Geração do Token de Acesso
Por questões de segurança, nunca devemos expor nossa senha principal. Em vez disso, vamos gerar um Token de Acesso (Access Token) que dará permissão ao GitHub Actions para se conectar à nossa conta. O pequeno tutorial abaixo mostrará como deve ser feito.

<img width="409" height="354" alt="Captura de imagem_20250925_151707" src="https://github.com/user-attachments/assets/ab019263-5e0f-4f6e-9e5f-c655a01ec3e5" />

- Clique em "Account settings"

<img width="1904" height="945" alt="Captura de imagem_20250925_151928" src="https://github.com/user-attachments/assets/4897611e-1aca-4834-ac82-b47119bfa5e8" />

- Quando aparecer a tela de configurações, clique em  "Personal access token" e depois clique em "New access token".

- Defina um nome para seu token, assim como um data de expiração e o mais importante, der as permissões de acesso de Read, Write e delete.

- Logo após, clique em "Generate".

- **IMPORTANTE:** Copie o token gerado e guarde-o em um local seguro. O Docker Hub só mostrará este token uma vez. Você precisará dele para configurar os segredos no GitHub.


### 2.3. Configuração dos Segredos no GitHub
Para que o GitHub Actions possa se conectar ao Docker Hub de forma segura, sem expor nossas credenciais diretamente no código, utilizamos os "Secrets" do repositório. Eles funcionam como um cofre de senhas para esta automação. Siga as seguintes etapas:

- Navegue até o seu repositório principal da aplicação (hello-app).

- Vá para a aba **Settings > Secrets and variables > Actions.**

- Clique no botão **"New repository secret"** para adicionar os seguintes segredos:

<img width="1920" height="547" alt="Captura de imagem_20250925_160030" src="https://github.com/user-attachments/assets/5c32ab8a-6700-4329-81f9-89cb1d055fcc" />

**Segredo do Usuário Docker:**
- Name: DOCKER_USERNAME
- Secret: Cole aqui o seu nome de usuário do Docker Hub (que você anotou na etapa 2.1).

**Segredo do Token do Docker:**
- Name: DOCKER_PASSWORD
- Secret: Cole aqui o Token de Acesso que você gerou no Docker Hub na etapa 2.2.

### 2.4. Acesso entre Repositórios com Chave SSH
Para que o workflow no hello-app tenha permissão de escrever (git push) no repositório hello-manifests, é preciso configurar um par de chaves SSH. Use esse comando abaixo para gerar 

```
ssh-keygen -t rsa -b 4096 -f github_deploy_key -C "seu_email@example.com"
```
Este comando cria dois arquivos: github_deploy_key (a chave privada) e github_deploy_key.pub (a chave pública).

<img width="246" height="94" alt="Captura de imagem_20250925_162753" src="https://github.com/user-attachments/assets/9a1e976e-7f3d-4212-be72-63232be6aa82" />

e para ver os conteúdos dessa chave use o comando:

```
cat nome_da_chave
```

#### 2.4.1 Configuração da Chave Pública (A Fechadura)
A chave pública é adicionada ao repositório que receberá o acesso, que no caso é o hello-manifests.

- Navegue até o repositório hello-manifests.
- Vá em **Settings > Deploy keys** e clique em **Add deploy key**.
- Prencha os seguintes campos:
    - Title: Dê um nome, como hello-app-workflow.
    - Key: Cole o conteúdo completo da sua chave pública (github_deploy_key.pub).
    - Marque a caixa "Allow write access" para permitir que a chave faça pushes.
 
#### 2.4.2 Configuração da chave privada (A chave)
A chave privada, que é o segredo, é adicionada ao repositório que realizará a actions, o hello-app.

- Adicione um novo segredo no repositório hello-app:
    - Name: SSH_PRIVATE_KEY
    - Secret: Cole o conteúdo completo da sua chave privada (github_deploy_key).

















  
