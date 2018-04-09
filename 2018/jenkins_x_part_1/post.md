<div style="text-align:center"><img src="logo_post.png" align="middle" width="250"></div>

# Jenkins X: Nova solução de CI/CD para Kubernetes - Parte 1

Recentemente foi disponibilizado á comunidade, uma nova solução de CI/CD denominada "Jenkins X", desenvolvido pela equipe da empresa CloudBees, que foram motivados pelo uso crescente de containeres (Ex: Docker) como distribuição de software, adoção de microserviços e o kubernetes como plataforma de gerenciamento.

trata-se de uma plataforma que utiliza o "Core" do Jenkins (nosso velho conhecido de guerra),com algumas features adicionais que tem o objetivo de "resolver os problemas de deploy na nuvem, usando o kubernetes", parece promissor não ?

Vamos conferir !

## Features

### Command-line tool (denominado "JX")

Isso mesmo, foi desenvolvido uma "*command-line tool*" para gerenciamento do Jenkins X e recursos do Kubernetes, é semelhante ao **KUBECTL** (Kubernetes) e **OC** (Openshift):

http://jenkins-x.io/getting-started/install/

É possível criar o Jenkins X em cluster Kubernetes existente, ou até mesmo criar um cluster + Jenkins na nuvem:

* Amazon Web Services (necessário ter o KOPS - https://kubernetes.io/docs/getting-started-guides/kops/)
* Microsoft Azure (Azure Kubernetes Service)
* Google Cloud (Google Kubernetes Engine)

Ou usando uma instalação local com minikube (https://kubernetes.io/docs/getting-started-guides/minikube/)

Além disso também é necessário configurar uma conta **GitHub** (durante a instalação, pois o Jenkins X irá criar alguns repositórios automaticamente para o **GitOps**, iremos explicar ao longo do post.

Ou seja prepare-se para usar muito o comando JX, segue alguns exemplos:

* Instalando o Jenkins

```

$ jx install

```

* Criando um cluster na AWS com Jenkins X

```

$ jx create cluster aws

```

### CI/CD com GitOps

Já ouviu falar em GitOps ? Não !

É uma prática atual para desenvolvimento de aplicativos de alta velocidade, usando ferramentas de cloud nativas.
Na prática é o "empoderamento do desenvolvedor na operação de ambientes".
Pois bem neste processo utilizamos ferramentas de desenvolvimento (GIT) como forma de conduzir operações nos ambientes (Ex: Kubernetes), quando passamos á utilizar o GIT como "fonte confiável", conseguimos ter ao mesmo tempo rastreabilidade, confiabilidade (pois as operações são realizadas por PULL REQUEST) e adotar boas práticas utilizadas em processos de CI/CD, é o "carro-chefe" na condução de operações no Jenkins X, desde a criação de ambientes ao processo de promoção, é criado um PULL REQUEST que combinado com as automações pré-definidas na plataforma realiza deploys sem a necessidade de uso de um CLI (kubectl) conduzindo tal operação.

### Criação simplificada de pipelines

Inicialmente não é necessário um conhecimento avançado em "Jenkinsfile", há 2 métodos para criar um projeto de forma rápida no Jenkins X usando o JX:

1) Importando um repositório GIT:

```

$ cd git-repo/
$ jx import

```

Após a importação do projeto, é criado um **DRAFT_PACK** no repositório de acordo com a linguagem.

atualmente há suporte para estas linguagens:

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

Neste **DRAFT_PACK** contém alguns recursos que será necessário versioná-los no repositório GIT:

* Jenkinsfile - Instruções de Deploy e steps de build e versionamento
* Dockerfile - Instruções de build de acordo com a linguagem
* HelmChart - Configurações da aplicação pré-definida (inclusive com porta default da linguagem mapeada Ex: 8080->8080) para o deploy no kubernetes

Se o usuário do GIT Jenkins X tiver permissão (commit) no repositório, automaticamente será configurado um *WebHook* para a pipeline recém-criada (Sempre será criado pipelines do tipo  multibranch)

> Legal não ? Até aqui já economizamos um bom tempo que perderíamos criando e testando estas configurações, e depois é possível customizar estes arquivos, uma vez que estará em seu repositório.

2) Ou usando um QuickStart template de acordo com a linguagem desejada (http://jenkins-x.io/developing/create-quickstart/) e posteriormente ir adicionando o código (recomendado para inicio de projeto)

```

$ jx create quickstart -l node-http

```

### Ambientes

Como o GitOps é o "leme" do Jenkins X, podemos criar os ambientes (Ex: Staging, Production e etc) usando a command-line tool (JX), será executada a sequência de ações:

* É criado automaticamente um repositório no "GitHub" com as configurações do ambiente, como está sendo adotado o GitOps, o fluxo de promoção/deploy será por "Pull Request".
* É criado um namespace no Kubernetes.
* No ato da criação é possível passar algumas opções de política de ambiente, por exemplo, se a promoção é automática ou manual, ou a url do GIT caso já exista o repo com as configurações para o Jenkins X (caso tenha deletado e deseja recuperar o ambiente)

```

$ jx create env -n myapp-production

```

E depois gerencie o ambiente via JX:

1) Relação dos ambientes

```

$ jx get envs

NAME              LABEL             KIND              PROMOTE  NAMESPACE            ORDER CLUSTER SOURCE  REF PR
myapp-production  myapp-production  myapp-production  Auto     jx-myapp-production  0   

```

### "Pull Request Preview Environment"

É aqui que o DevOps passar a fazer café (trocadilhos á parte), quando criamos um projeto no Jenkins X é gerado o "DRAFT_PACK" no qual contém algumas instruções como citado anteriormente, pois bem, detalhando:

* Quando há um "Pull Request" pendente, automaticamente é gerado um BUILD e DEPLOY em um ambiente temporário, denominado como "**PREVIEW**".
* É criado uma namespace específica do Pull Request no Kubernetes, isolado dos outros ambientes produtivos ou não, que pode ser descartado posteriormente

```

$ cd git-repo/
$ jx preview myapp

```

> Ou seja, o DEV não precisa mais demandar a criação de um ambiente temporário para validar o PULL REQUEST, o grupo consegue validar a feature, divulgar a URL para o Product Owner validar uma feature por exemplo, sem fazer o uso de "na minha máquina funciona (mesmo em tempos de Docker isso ainda existe)", se antecipando a quebras, mesmo que em Development, que possa interromper o CI.


### Feedback em "Issues" e "Pull request"

Também é possível usar o JX para feedback:

```

$ jx create issue -t "vamos deixar as coisas mais incríveis"

```

Ou até mesmo configura um tracker do JIRA no repositório:

```

$ jx create tracker server jira https://mycompany.atlassian.net/

```

### Addons

Ainda não é muitos, mas é um recurso interessante para criar recursos próprios para gerenciamento pelo JX, e não diretamente pelo cluster Kubernetes, atualmente é possível adicionar o Grafana, Gitea

```

$ jx create addon grafana

```

## Comparativo

### Prós

* O GitOps é o de fato o "Modus operandi" no Jenkins X, ajuda muito á converter o conceito em prática.
* O uso do Helm (gerenciador de pacotes do Kubernetes) como mecanismo para manter a integridade da aplicação, bem como as configurações do próprio ambiente (Ex: Environments) é positivo.
* O desenvolvimento de uma command-line tool (JX) foi essencial para este projeto, espero que tenha mais features com o tempo.

### Contras

* A command-line tool "JX" atende bem, mas ainda falta alguns recursos, como a ausência de um comando para obter a relação de aplicativos ativos no "preview", forçando a usar o "kubectl" para obter isso direto do cluster.
* O processo de instalação requer conhecimentos com Kubernetes, pois em algum momento será necessário para um possível "Troubleshooting", para casos onde o Jenkins X está sendo instalado em um cluster já existente.

## Considerações

Acredito que o objetivo de o desenvolvedor ter mais autonomia está sendo alcançado, com gerenciamento simplificado (kubernetes), e um processo de deploy moderno (GitOps).

Claro ainda há pontos de melhoria como citado acima, mas ao sair da zona de conforto (CloudBees) para tentar resolver problemas de CI/CD na nuvem em favor da comunidade e se adaptar as mudanças é louvável, vale a pena testar para ter uma alternativa futuramente, visto que pipelines com "Jenkinsfile" é largamente utilizada, minimizando transtornos de adaptação futuramente além de adotar as boas práticas que estão sendo propostas pela plataforma.

## Observações

Se ainda tem dúvida com o Kubernetes, recomendo a leitura destes posts:

* [As diferenças entre Docker, Kubernetes e Openshift](https://www.concrete.com.br/2017/06/26/docker-kubernetes-e-openshift/)
* [Tudo o que você precisa saber sobre Kubernetes - Parte 1](https://www.concrete.com.br/2018/02/22/tudo-o-que-voce-precisa-saber-sobre-kubernetes/)
* [Tudo o que você precisa saber sobre Kubernetes - Parte 2](https://www.concrete.com.br/2018/02/23/tudo-o-que-voce-precisa-saber-sobre-kubernetes-parte-2/)

Aqui também tem as refências para este POST:

* [Introducing Jenkins X: a CI/CD solution for modern cloud applications on Kubernetes](https://jenkins.io/blog/2018/03/19/introducing-jenkins-x/)
* [GitHub Jenkins X](https://github.com/jenkins-x)
* [Jenkins X is a CI/CD solution for modern cloud applications on Kubernetes](http://jenkins-x.io/)

No próximo post vamos explicar passo-a-passo o processo de instalação e uso de uma aplicação Demo.

Fica ligado e até a próxima !