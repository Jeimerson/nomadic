---

# Warship Shared Resources

### Cada aplicacao, ou Warship, tem uma jornada e um ciclo de vida unicos.

* Os recursos Terraform neste diretório são compartilhados entre todos os pipelines de aplicações Warship.
* Diretório Docker `nomadic_application_example`. Este diretório contém o Dockerfile da aplicação e arquivos de suporte.
* Diretório template Terraform `skeleton`. Este diretório de exemplo é usado para implantar Pipelines Warship (CodePipelines).

### Para criar uma Warship Application (imagem de container, pipeline e implantação):

1. Implante os recursos compartilhados (S3 e IAM).

   1. Atualize `terraform.tfvars` neste diretório conforme necessário.
   2. Execute `terraform init` e `terraform apply` neste diretório existente.
   3. Os recursos compartilhados são publicados no SSM ParameterStore para uso dos pipelines.

2. Implante as warships individuais (pipelines da aplicação).

   1. Copie o diretório `skeleton` para um diretório único da aplicação.
   2. Atualize `terraform.tfvars` neste diretório conforme necessário.
   3. Execute `terraform init` e `terraform apply` neste novo diretório.
   4. O pipeline CodePipeline da aplicação Warship é criado e executado.

      1. A aplicação agora deve estar ativa no cluster Nomadic.
      2. O container deve estar em execução no Nomad.
      3. O registro do container deve estar visível no Consul.

3. O Application Load Balancer e o DNS configurados e adicionados ao Route53. (Opcional)

4. O volume EFS é criado e disponibilizado aos containers em execução para estado compartilhado de filesystem. (Opcional)

---
