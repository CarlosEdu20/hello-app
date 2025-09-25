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
  
