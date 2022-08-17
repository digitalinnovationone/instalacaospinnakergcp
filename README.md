# Instalação E Configuração Spinnaker

O Spinnaker no Goole Cloud utiliza GKE, Memorystore, buckets do Cloud Storage e contas de serviço.

## Passo 1 - Clone do Repositório https://github.com/GoogleCloudPlatform/spinnaker-for-gcp.git


```shell
mkdir ~/cloudshell_open && cd ~/cloudshell_open
```

```shell
git clone https://github.com/GoogleCloudPlatform/spinnaker-for-gcp.git
```

```shell
cd spinnaker-for-gcp
```

## Passo 2 - Criação das variáveis para instalação do Spinnaker

```shell
./scripts/install/setup_properties.sh
```

## Passo 3 - Confierir as variáveis geradas.

```shell
cat ./scripts/install/properties
```

## Passo 4 - Aplicar as variáveis no sistema de variáveis do Clou Shell.

```shell
source ./scripts/install/properties
```


## Passo 5 - Configurações de git user.email e user.name

```shell
git config --global user.email "USER@empresa.com"
git config --global user.name "USER"
```

## Passo 6 - Iniciar a instalação

```shell
./scripts/install/setup.sh
```

A instalação pode levar uma média de 15 minutos.


## Passo 7 - Conectando na Interface do Spinnaker

```shell
./scripts/manage/connect_unsecured.sh
```

