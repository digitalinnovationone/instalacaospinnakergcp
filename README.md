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


# Inclusao segundo cluster GKE

## Passo 1 - Configurar a região do cluster

```shell
APP_REGION=us-east1-b; gcloud config set compute/zone $APP_REGION
```

## Passo 2 - Criação do Cluster
```shell
gcloud container clusters create app-cluster --machine-type=n1-standard-2
```

## Passo 3 - Configurar o Spinnaker Cluster

```shell
./scripts/manage/add_gke_account.sh
```


```shell
kubectl config use-context gke_${PROJECT_ID}_${ZONE}_spinnaker-1
```

```shell
./scripts/manage/push_and_apply.sh
```

# Configurando o Pipeline de Exemplo

## 1. Entre na pasta samples

```shell
cd ./samples/helloworldwebapp/
```

```shell
~/spin app save --application-name helloworldwebapp \
    --cloud-providers kubernetes --owner-email $IAP_USER
```

## 2. Faça deploy do Pipeline de Stagging

```shell
cat templates/pipelines/deploystaging_json.template | envsubst  > templates/pipelines/deploystaging.json
```

```shell
~/spin pi save -f templates/pipelines/deploystaging.json
```
## 3. Salve o Stagging Pipeline ID

```shell
export DEPLOY_STAGING_PIPELINE_ID=$(~/spin pi get -a helloworldwebapp -n 'Deploy to Staging' | jq -r '.id')
```
## 4. Faça deploy do Pipeline de Prod


```shell
cat templates/pipelines/deployprod_json.template | envsubst  > templates/pipelines/deployprod.json
```

```shell
~/spin pi save -f templates/pipelines/deployprod.json
```

# Criando o código fonte da aplicação

## 1. Crie uma pasta com o código fonte


```shell
mkdir -p ~/$PROJECT_ID/spinnaker-for-gcp-helloworldwebapp/
```

## 2. Copie o código fonte da aplicação

```shell
cp -r templates/repo/config ~/$PROJECT_ID/spinnaker-for-gcp-helloworldwebapp/
```

```shell
cp templates/repo/Dockerfile ~/$PROJECT_ID/spinnaker-for-gcp-helloworldwebapp/
```

```shell
cat templates/repo/cloudbuild_yaml.template | envsubst '$BUCKET_NAME' > ~/$PROJECT_ID/spinnaker-for-gcp-helloworldwebapp/cloudbuild.yaml
cat ~/$PROJECT_ID/spinnaker-for-gcp-helloworldwebapp/config/staging/replicaset_yaml.template | envsubst > ~/$PROJECT_ID/spinnaker-for-gcp-helloworldwebapp/config/staging/replicaset.yaml
rm ~/$PROJECT_ID/spinnaker-for-gcp-helloworldwebapp/config/staging/replicaset_yaml.template
cat ~/$PROJECT_ID/spinnaker-for-gcp-helloworldwebapp/config/prod/replicaset_yaml.template | envsubst > ~/$PROJECT_ID/spinnaker-for-gcp-helloworldwebapp/config/prod/replicaset.yaml
rm ~/$PROJECT_ID/spinnaker-for-gcp-helloworldwebapp/config/prod/replicaset_yaml.template
```

# Criando o Repositório Git com Google Cloud Source Repositories

## 1. Entre no diretório da aplicação

```shell
cd ~/$PROJECT_ID/spinnaker-for-gcp-helloworldwebapp
```

## 2. Inicialize o git adicionando os arquivos e fazendo o commit

```shell
git init
git add .
git commit -m "Initial commit"
```

## 3. Crie o repositório


```shell
gcloud source repos create spinnaker-for-gcp-helloworldwebapp
git config credential.helper gcloud.sh
```

## 4. Adicione o repositório remoto

```shell
git remote add origin https://source.developers.google.com/p/$PROJECT_ID/r/spinnaker-for-gcp-helloworldwebapp
```

## 5. Push master


```shell
git push origin master
```


# Configure Cloud Build Triggers

## 1. Revise os passos do Coud Build

```shell
cat ./cloudbuild.yaml
```
## 2. Crie o Cloud Build Trigger com gcloud sdk

```shell
gcloud beta builds triggers create cloud-source-repositories \
    --repo spinnaker-for-gcp-helloworldwebapp \
    --branch-pattern master \
    --build-config cloudbuild.yaml \
    --included-files "src/**,config/**"
```

# Compilando a imagem da aplicação

## 1. Edite a aplicação e faça o commit

```shell
sed -i 's/Hello World/Hello World 1.0/g' ./src/main.go
```

```shell
git commit -a -m "Change to 1.0"
```

## 2. Faça o push para master

```shell
git push origin master
```


