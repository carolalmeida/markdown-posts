<div style="text-align:center"><img src="logo_post.png" align="middle" width="250"></div>

# Jenkins X: Nova solução de CI/CD para Kubernetes - Parte 1

Motivados pelo uso crescente de conteineres (ex: Docker), de distribuição de software, de microserviços e do kubernetes como plataforma de gerenciamento, uma equipe da empresa CloudBees disponibilizou recentemente à comunidade uma nova solução de CI/CD, chamada de "Jenkins X".

A plataforma utiliza o "Core" do Jenkins (nosso velho conhecido de guerra), com algumas features adicionais que têm o objetivo de "resolver os problemas de deploy na nuvem usando o kubernetes". Parece promissor, não?

Vamos conferir!

## Features

### Command-line tool (denominado "JX")

Isso mesmo, desenvolveram uma "*command-line tool*" para gerenciar o Jenkins X e os recursos do Kubernetes. A feature é semelhante ao **KUBECTL** (Kubernetes) e ao **OC** (Openshift):

http://jenkins-x.io/getting-started/install/

É possível criar o Jenkins X em algum cluster Kubernetes existente ou até mesmo criar um cluster + Jenkins na nuvem:

* Amazon Web Services (necessário ter o KOPS - https://kubernetes.io/docs/getting-started-guides/kops/)
* Microsoft Azure (Azure Kubernetes Service)
* Google Cloud (Google Kubernetes Engine)

Ainda é possível usar uma instalação local com minikube (https://kubernetes.io/docs/getting-started-guides/minikube/)

Por fim, é necessário configurar uma conta **GitHub** durante a instalação, pois o Jenkins X vai criar alguns repositórios automaticamente para o **GitOps**. Calma, já já a gente explica.

Ou seja, prepare-se para usar muito o comando JX! Veja alguns exemplos:

* Instalando o Jenkins

```

$ jx install

```

* Criando um cluster na AWS com Jenkins X

```

$ jx create cluster aws

```

### CI/CD com GitOps

Você já ouviu falar em GitOps? Não?

É uma prática atual para desenvolvimento de aplicativos de alta velocidade, que usa ferramentas de cloud nativas. Na prática, é o "empoderamento do desenvolvedor na operação de ambientes".

Pois bem, neste processo utilizamos ferramentas de desenvolvimento (GIT) como forma de conduzir operações nos ambientes (Ex: Kubernetes). Quando passamos a utilizar o GIT como "fonte confiável", conseguimos ter ao mesmo tempo rastreabilidade, confiabilidade (pois as operações são realizadas via PULL REQUESTS) e adotar boas práticas utilizadas em processos de CI/CD. O GIT é o "carro-chefe" na condução de operações no Jenkins X, desde a criação de ambientes até o processo de promoção. Para tudo isso são criados PULL REQUESTS que, combinados com as automações pré-definidas na plataforma, realizam deploys sem a necessidade de uso de um CLI (kubectl) conduzindo tal operação.

### Criação simplificada de pipelines

A princípio você não precisa ser um expert em "Jenkinsfile". Existem dois métodos para criar um projeto de forma rápida no Jenkins X usando o JX:

1) Importando um repositório GIT:

```

$ cd git-repo/
$ jx import

```

Após a importação do projeto, é criado um **DRAFT_PACK** no repositório de acordo com a linguagem, que pode ser (atualmente):

* C#
* Golang
* Graddle
* JavaScript
* Maven
* PHP
* Python
* Ruby
* Rust
* Swift

Alguns recursos deste **DRAFT_PACK** precisam ser versionados no repositório GIT:

* Jenkinsfile - instruções de Deploy e steps de build e versionamento;
* Dockerfile - instruções de build de acordo com a linguagem;
* HelmChart - configurações da aplicação pré-definida (inclusive com porta default da linguagem mapeada Ex: 8080->8080) para o deploy no kubernetes.

Se o usuário do GIT Jenkins X tiver permissão (commit) no repositório, um *WebHook* será configurado automaticamente para a pipeline recém-criada (sempre serão criadas pipelines do tipo  multibranch).

> Legal, não? Até aqui já economizamos um bom tempo que perderíamos criando e testando estas configurações, e ainda é possível customizar estes arquivos depois, uma vez que eles estarão em seu repositório.

2) A outra forma é usar um QuickStart template de acordo com a linguagem desejada (http://jenkins-x.io/developing/create-quickstart/) e depois ir adicionando código. Essa forma é recomendada principalmente para início de projetos.

```

$ jx create quickstart -l node-http

```

### Ambientes

Como o GitOps é o "leme" do Jenkins X, podemos criar os ambientes (Ex: Staging, Production e etc) usando a command-line tool (JX). Para isso, precisamos seguir os seguintes passos:

* Um repositório no "GitHub" com as configurações do ambiente será criado automaticamente. Como está sendo adotado o GitOps, o fluxo de promoção/deploy será via "Pull Request";
* Criar um namespace no Kubernetes;
* Na criação é possível passar algumas opções de política de ambiente, como se a promoção é automática ou manual, por exemplo, ou a url do GIT caso já exista o repo com as configurações para o Jenkins X (caso tenha deletado e deseja recuperar o ambiente).

```

$ jx create env -n myapp-production

```

Depois, gerencie o ambiente via JX:

1) Relação dos ambientes

```

$ jx get envs

NAME              LABEL             KIND              PROMOTE  NAMESPACE            ORDER CLUSTER SOURCE  REF PR
myapp-production  myapp-production  myapp-production  Auto     jx-myapp-production  0   

```

### "Pull Request Preview Environment"

É aqui que o DevOps passar a fazer café (trocadilhos à parte). Quando criamos um projeto no Jenkins X geramos o "DRAFT_PACK", que contém algumas instruções. Pois bem, detalhando:

* Quando há um "Pull Request" pendente, automaticamente são gerados um BUILD e um DEPLOY em um ambiente temporário, o que chamamos de "**PREVIEW**".
* Também é criada uma namespace específica do Pull Request no Kubernetes, isolada dos outros ambientes, produtivos ou não, e que pode ser descartada posteriormente

```

$ cd git-repo/
$ jx preview myapp

```

> Ou seja, o DEV não precisa mais demandar a criação de um ambiente temporário para validar o PULL REQUEST, o grupo consegue validar a feature e divulgar a URL para o Product Owner sem usar o "na minha máquina funciona". Sério, mesmo em tempos de Docker isso ainda existe. Isso permite que as quebras que possam interromper o CI sejam antecipadas, mesmo em Development.


### Feedback em "Issues" e "Pull request"

Também é possível usar o JX para feedback:

```

$ jx create issue -t "vamos deixar as coisas mais incríveis"

```

Ou até mesmo configurar um tracker do JIRA no repositório:

```

$ jx create tracker server jira https://mycompany.atlassian.net/

```

### Addons

Ainda não são muitos, mas é um recurso interessante para criar recursos próprios para gerenciamento pelo JX, e não diretamente pelo cluster Kubernetes. Atualmente é possível adicionar o Grafana, Gitea:

```

$ jx create addon grafana

```

## Comparativo

### Prós

* O GitOps é o de fato o "Modus operandi" no Jenkins X, ajuda muito a converter o conceito em prática;
* O uso do Helm (gerenciador de pacotes do Kubernetes) como mecanismo para manter a integridade da aplicação, bem como as configurações do próprio ambiente (Ex: Environments), é positivo;
* O desenvolvimento de uma command-line tool (JX) foi essencial para este projeto, espero que surjam mais features com o tempo.

### Contras

* A command-line tool "JX" atende bem, mas ainda falta alguns recursos, como a ausência de um comando para obter a relação de aplicativos ativos no "preview", o que nos força a usar o "kubectl" para obter isso direto do cluster;
* O processo de instalação requer conhecimentos em Kubernetes, que em algum momento serão necessários para um possível "Troubleshooting", casos nos quais o Jenkins X está sendo instalado em um cluster já existente.

## Considerações

O objetivo de o desenvolvedor ter mais autonomia está sendo alcançado, com gerenciamento simplificado (kubernetes) e um processo de deploy moderno (GitOps).

Ainda há pontos de melhoria, mas sair da zona de conforto (CloudBees) para tentar resolver problemas de CI/CD na nuvem em favor da comunidade e se adaptar às mudanças é louvável. Vale a pena testar para ter uma alternativa futuramente, visto que pipelines com "Jenkinsfile" são largamente utilizadas, o que minimiza transtornos de adaptação além de adotar as boas práticas que estão sendo propostas pela plataforma.

## Observações

Se ainda tem dúvida com o Kubernetes, recomendo a leitura destes posts:

* [As diferenças entre Docker, Kubernetes e Openshift](https://www.concrete.com.br/2017/06/26/docker-kubernetes-e-openshift/)
* [Tudo o que você precisa saber sobre Kubernetes - Parte 1](https://www.concrete.com.br/2018/02/22/tudo-o-que-voce-precisa-saber-sobre-kubernetes/)
* [Tudo o que você precisa saber sobre Kubernetes - Parte 2](https://www.concrete.com.br/2018/02/23/tudo-o-que-voce-precisa-saber-sobre-kubernetes-parte-2/)

Aqui também tem as refências para este POST:

* [Introducing Jenkins X: a CI/CD solution for modern cloud applications on Kubernetes](https://jenkins.io/blog/2018/03/19/introducing-jenkins-x/)
* [GitHub Jenkins X](https://github.com/jenkins-x)
* [Jenkins X is a CI/CD solution for modern cloud applications on Kubernetes](http://jenkins-x.io/)

Amanhã a gente volta para explicar passo a passo o processo de instalação e o uso de uma aplicação Demo. Até a próxima!
