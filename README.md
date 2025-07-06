# Projetos do curso Aula Devops Fundamentals

# 🚀 Infraestrutura como Código com Terraform no Azure DevOps

Este guia resume os passos para configurar a infraestrutura no Microsoft Azure usando Terraform via Azure DevOps.

## ✅ Pré-requisitos

- Conta no Azure e permissões de Contributor no Subscription
- Projeto no Azure DevOps
- Repositório com os arquivos `.tf` já versionados (infraestrutura)

---

## 1️⃣ Criar o Service Connection no Azure

### Etapas:

1. Acesse seu projeto no [Azure DevOps](https://dev.azure.com/)
2. Vá em: **Project Settings** > **Service connections**
3. Clique em **New service connection**
4. Escolha a opção **Azure Resource Manager**
5. Selecione o método **Service principal (automatic)** e clique em **Next**
6. Preencha:
   - **Subscription**: selecione a sua
   - **Scope level**: Subscription ou Management Group
   - **Resource Group** (opcional)
   - **Service connection name**: ex. `sc-terraform-infra`
7. Marque “Grant access permission to all pipelines”
8. Clique em **Save**

---

## 2️⃣ Criar o Pipeline no Azure DevOps com Terraform

### Etapas:

1. No menu lateral, clique em **Pipelines** > **New pipeline**
2. Escolha **Azure Repos Git**
3. Selecione o repositório com os arquivos Terraform
4. Escolha **Starter pipeline** ou use `existing YAML`
5. Substitua o conteúdo pelo exemplo abaixo:

```yaml
trigger:
  branches:
    include:
      - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  serviceConnection: 'sc-terraform-infra'
  azureSubscription: 'Nome-da-sua-subscription'
  resourceGroup: 'rg-devops-infra'
  location: 'eastus'

stages:
- stage: Terraform
  jobs:
  - job: PlanAndApply
    steps:
    - task: TerraformInstaller@0
      inputs:
        terraformVersion: '1.5.7'

    - task: TerraformTaskV4@4
      displayName: 'Terraform Init'
      inputs:
        provider: 'azurerm'
        command: 'init'
        backendServiceArm: '$(serviceConnection)'
        backendAzureRmResourceGroupName: '$(resourceGroup)'
        backendAzureRmStorageAccountName: 'terraformstatesa'
        backendAzureRmContainerName: 'tfstate'
        backendAzureRmKey: 'infra.tfstate'

    - task: TerraformTaskV4@4
      displayName: 'Terraform Plan'
      inputs:
        provider: 'azurerm'
        command: 'plan'
        environmentServiceNameAzureRM: '$(serviceConnection)'

    - task: TerraformTaskV4@4
      displayName: 'Terraform Apply'
      inputs:
        provider: 'azurerm'
        command: 'apply'
        environmentServiceNameAzureRM: '$(serviceConnection)'
        args: '-auto-approve'


3️⃣ Planejamento e Aplicação da Infraestrutura (resumo)

terraform init: inicializa o diretório e configura o backend

terraform plan: mostra as ações que o Terraform realizará

terraform apply: aplica as mudanças e cria os recursos

Esses comandos são executados automaticamente pelo pipeline, conforme configurado acima.


4️⃣ Exemplo de criação de Resource Group com Tags

Exemplo de código Terraform (main.tf):

provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "rg" {
  name     = "rg-devops-infra"
  location = "East US"

  tags = {
    Environment = "Dev"
    Owner       = "Equipe-Infrastructure"
    Projeto     = "TerraformPipeline"
  }
}


✅ Conclusão

Com isso, temos:

Um pipeline automatizado no Azure DevOps

Conexão segura com o Azure por Service Principal

Aplicação da infraestrutura com versionamento

Resource Group com tags para rastreabilidade

Recomenda-se usar um Storage Account e container para armazenar o estado remoto (tfstate) e garantir segurança e rastreabilidade da infraestrutura.