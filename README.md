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

### 2.5. Token de acesso (PAT) para API do Github
Enquanto a chave SSH serve para operações Git, a criação do Pull Request é uma operação da API do GitHub. Para isso, precisamos de um Personal Access Token (PAT). Para gerar o PAT, siga estes seguintes passos:

- No seu perfil do GitHub, vá em **Settings > Developer settings > Personal access tokens > Tokens (classic)**.
- Clique em **Generate new Token**

Agora faça as seguintes configurações no token:

- Note: Dê um nome descritivo, como actions-cross-repo-pr.
- Expiration: Defina uma data de validade.
- Select scopes: Marque a caixa de seleção principal repo.

Agora configure os segredos no repositório do hello-app.
- Name: GH_PAT
- Secret: Cole o Personal Access Token que você acabou de gerar.


### 2.6. O Workflow de Automação
Com todas as permissões e segredos devidamente configurados, A etapa final é criar o arquivo de workflow que orquestra toda a automação. Este arquivo é o cérebro da pipeline de CI/CD. Crie a estrutura de pastas como `.github/workflows/` no seu repositório **hello-app** e, dentro dela, crie o arquivo `ci.yml` com o conteúdo disponibilizado neste repositório. Com este arquivo, a Etapa 2 está finalizado. Agora você tem uma pipeline de CI totalmente funcional que é acionada a cada alteração no código da sua aplicação.


Se tudo ocorrer certo, a pipeline gerada será essa:
<img width="1914" height="822" alt="Captura de imagem_20250926_102251" src="https://github.com/user-attachments/assets/4545c812-7ebc-4644-b4a8-4a002aa801b9" />

A imagem no Docker Hub será essa:

<img width="1910" height="450" alt="Captura de imagem_20250926_102512" src="https://github.com/user-attachments/assets/ba6940e4-fba2-47f9-93d1-8b59f5bcb859" />

Aqui mostra a evidência do pull request para o hello-manifests: 

<img width="1889" height="841" alt="Captura de imagem_20250926_103748" src="https://github.com/user-attachments/assets/edfab371-6be5-4d0b-b15a-5b7d1d4a00d7" />


## Etapa 3: Criação dos Manifestos Kubernetes
Com a pipeline de CI funcionando devidamente, agora devemos definir como a aplicação será executada dentro do cluster Kubernetes. Para isso, deve-se criar os arquivos de manifesto no repositório hello-manifests. Estes arquivos vão ser a "fonte da verdade" que o ArgoCD utilizará para gerenciar a aplicação.

### 3.1. Clone do Repositório de Manifestos
Primeiramente, clonamos o repositório localmente para poder criar e editar os arquivos.

```
git clone https://github.com/SEU_USUARIO/hello-manifests.git
cd hello-manifests
```

### 3.2. Manifesto de Deployment 
O Deployment é um objeto do Kubernetes que gerencia a implantação e a escalabilidade da aplicação, garantindo que um número específico de réplicas (pods) esteja sempre em execução. 

Criaremos o arquivo deployment.yaml com seguinte conteúdo:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-app
  template:
    metadata:
      labels:
        app: hello-app
    spec:
      containers:
      - name: hello-app
        # IMPORTANTE: A tag inicial pode ser 'latest' ou um commit inicial.
        # A pipeline de CI/CD irá atualizar esta linha automaticamente.
        image: SEU_DOCKER_USERNAME/hello-app:latest
        ports:
        - containerPort: 80
```
Lembre-se de substituir SEU_DOCKER_USERNAME pelo seu nome de usuário do Docker Hub.


### 3.3. Manifesto de Service 
O service expõe nossa aplicação como um serviço de rede, criando um ponto de acesso estável para os pods gerenciados pelo Deployment.

Criaremos o seguinte arquivo service.yaml:

```
apiVersion: v1
kind: Service
metadata:
  name: hello-app-service
spec:
  # O seletor conecta este serviço aos pods com a label 'app: hello-app'
  selector:
    app: hello-app
  ports:
    - protocol: TCP
      # A porta 8080 do cluster irá redirecionar para a porta 80 do nosso contêiner
      port: 8080
      targetPort: 80
```

### 3.4. Envio dos Manifestos ao GitHub
Após criar os dois arquivos, nós os enviamos para a branch `main` do repositório hello-manifests.

```
git add .
git commit -m "feat: add initial deployment and service manifests"
git push origin main
```

Com esta etapa concluída, o repositório de manifestos está pronto para ser monitorado pelo ArgoCD.


## Etapa 4: Criação da Aplicação no ArgoCD
Com a pipeline de CI (Etapa 2) e os manifestos (Etapa 3) prontos, esta é a etapa que implementa a Entrega Contínua (CD). Configuraremos o ArgoCD para monitorar o repositório hello-manifests e garantir que o estado do nosso cluster Kubernetes seja sempre um reflexo fiel do que está definido nos arquivos de manifesto.


### 4.1. Acesso à Interface Web do ArgoCD
Para acessar a interface do ArgoCD, que normalmente não é exposta publicamente, usaremos o encaminhamento de portas do kubectl. Com isso, Digite o seguinte comando:

```
kubectl port-forward svc/argocd-server -n argocd 8888:443
```
Mantenha este terminal em execução enquanto usa a interface.

Abra seu navegador e acesse https://localhost:8888/.

### 4.2. Login no ArgoCD
Use as seguintes credenciais para fazer o login:

- **Username:** admin
- **Password:** Para obter a senha inicial, execute o seguinte comando no terminal:

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

<img width="1902" height="946" alt="Captura de imagem_20250926_120804" src="https://github.com/user-attachments/assets/af027e93-bf94-4806-8b2a-14b4ded00b79" />

### 4.3. Configurando a aplicação
Dentro do painel do ArgoCD, vamos criar a aplicação que fará o vínculo com nosso repositório Git.

- Clique no botão **"+ NEW APP"** no canto superior esquerdo.
- Preencha o formulário com as informações a seguir:
  
    **General:**
  - **Application Name:** hello-app
  - **Project Name:** default
  - **Sync Policy**: Automatic

   **Source:**
  - **Repository URL:** A URL do seu repositório de manifestos (ex: https://github.com/SEU_USUARIO/hello-manifests.git).
  - **Revision:** Sua branch principal.
  - **Path:**.

   **Destination:**
  - **Cluster URL:** https://kubernetes.default.svc
  - **Namespace:** Coloque o namespace que você criou

Após preencher, clique em **"CREATE"** no topo da página.


### 4.4. Sincronização e validação
Após a criação da aplicação, o ArgoCD exibirá um "card" representando o hello-app. Como a política de sincronização é automática, o ArgoCD irá imediatamente:

- Clonar o repositório hello-manifests.

- Comparar os arquivos (deployment.yaml, service.yaml) com o que está rodando no cluster.

- Aplicar os manifestos para criar os recursos da aplicação no Kubernetes.


 Em alguns instantes, o status da aplicação deve mudar para Synced (Sincronizado) e Healthy (Saudável), indicando que a implantação foi bem-sucedida.

 <img width="1834" height="936" alt="Captura de imagem_20250926_143230" src="https://github.com/user-attachments/assets/c604607a-932c-4044-bd68-e87b5c897bdb" />


 ## Etapa 5: Acessar e testar a aplicação localmente
 Com a aplicação sincronizada pelo ArgoCD, a etapa final do projeto é verificar se ela está rodando corretamente no cluster e se conseguimos acessá-la.

 ### 5.1. Verificando o status do pod
 O primeiro passo é usar o kubectl para listar os pods no cluster e confirmar se o mesmo foram criados e estão funcionando na aplicação. Para esse propósito, digite o seguinte comando

```
kubectl get pods -n (nome_da_sua_namespace)
```

A saída esperada deve ser essa:

<img width="1897" height="102" alt="Captura de imagem_20250926_144910" src="https://github.com/user-attachments/assets/1320d26d-ad1a-4365-ac74-3671439a0871" />

### 5.2. Acessando a Aplicação Localmente
Agora que sabemos que o pod está rodando sem problemas, vamos acessar a aplicação para garantir que ela está respondendo. Para isso, usamos o **port-forward** para criar um túnel de rede seguro do seu computador até o serviço no cluster. Mantenha o terminal anterior aberto e, em um novo terminal, execute o comando:

```
kubectl port-forward service/hello-app-service -n <nome_da_sua_namespace> 8081:8080

```

Logo após, irá mostrar a aplicação funcionando:

<img width="1916" height="936" alt="Captura de imagem_20250926_150938" src="https://github.com/user-attachments/assets/81255c3a-cfa7-41d2-889b-361778e1265b" />


A resposta da aplicação via `curl`:

<img width="1910" height="98" alt="Captura de imagem_20250926_151215" src="https://github.com/user-attachments/assets/b98fd9b7-9791-40d3-ab30-4865e7271ecc" />


### 5.3. Testando a alteração no repositório
Para ter certeza que o ArgoCD está sincronizando corretamente, vamos alterar a mensagem do arquivo `main.py`.

Vamos colocar essa mensagem "Alteração feita com sucesso" e logo seguido vamos dar uma git push do repositório. Percebe-se que vai ser tem um novo pull request no repositório hello-manifests, aprove o mesmo.

Com isso, o ArgoCD verá que teve uma mudança na fonte e aplicará as alterações feitas. Como mostra nessa imagem


<img width="1903" height="932" alt="Captura de imagem_20250926_152501" src="https://github.com/user-attachments/assets/bd7d4eb9-9fd0-4355-b319-72631d771373" />


Veja que foi criado um novo pod da aplicação. A imagem logo abaixo mostra a mensagem nova:

<img width="1904" height="963" alt="Captura de imagem_20250926_152913" src="https://github.com/user-attachments/assets/88b15b0b-2efd-4624-836e-8e45bbdc8bd5" />






 


 




































  
