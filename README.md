
---

# Hashicorp Nomad, Consul, and Vault

### Por que executar containers com código da Hashicorp?

Nomadic é um framework Terraform para implantar clusters mínimos e seguros de containers usando Hashicorp.
A Hashicorp garante que seus containers Docker estejam em execução (com fail-over para tolerância a falhas) e funcionando de forma segura (rotação de segredos e segurança AWS). A Internet é um lugar perigoso. **Warship Not Whale**.

### Framework Nomadic

1. Provisionamento IaC com Terraform para recursos do cluster.
2. Criação opcional de VPC e recursos de suporte.
3. Bootstrap do servidor via Instance UserData.
4. Consul fornece registro de containers da aplicação.
5. Vault fornece segredos para containers da aplicação.
6. Nomad fornece o agendamento dos containers da aplicação.

O código Terraform do Nomadic implanta um cluster de (3) instâncias EC2 para implementar o menor footprint possível para executar containers com código da Hashicorp.
Os seguintes serviços executam em cada nó do cluster Nomadic:

1. Servidor Vault (uma instância ativa, duas em standby).

   1. API Vault tcp/8200

2. Servidor Consul.

   1. DNS: O servidor DNS (TCP e UDP) 8600
   2. HTTP: A API HTTP (somente TCP) 8500
   3. HTTPS: A API HTTPS (desabilitada) 8501
   4. gRPC: A API gRPC (desabilitada) 8502
   5. LAN Serf: Porta Serf LAN (TCP e UDP) 8301
   6. WAN Serf: Porta Serf WAN (TCP e UDP) 8302
   7. Servidor: Endereço RPC do servidor (somente TCP) 8300

3. Servidor Nomad.

   1. API HTTP tcp/4646
   2. RPC tcp/4647
   3. Serf WAN tcp/4648

Nomadic implanta todos esses serviços em cada uma das três instâncias EC2 do cluster.

* Nomad e Consul usam o protocolo Raft e autodiscovery da EC2 para convergência do cluster.
* Consul lê um token de criptografia comum do SSM Parameter.
* Vault usa auto-unsealing via KMS.
* Vault usa design de uma instância ativa e duas em standby.
* Consul e Nomad usam design de três nós ativos (estado compartilhado do cluster).

### Implantação Nomadic

Para implantar o cluster:

1. defina as variáveis em `terraform.tfvars`
2. `terraform init`
3. `terraform plan`
4. `terraform apply`

### Warships Nomadic e Warship Pipelines

Warships são coleções de recursos que compõem uma aplicação.
Isso inclui o pipeline necessário para atualizações e gerenciamento de ciclo de vida da aplicação.

CodePipeline executa um pipeline único para cada aplicação.
Pipelines Warship podem rodar quantas vezes forem necessárias, atualizando a aplicação apenas quando necessário.
Testes de unidade e integração no pipeline Warship detectam falhas antes do código ir para produção, garantindo alta confiança nas mudanças.
Aplicações Warship podem ser atualizadas dezenas, centenas ou milhares de vezes por dia sem esforço.

**Cada Pipeline de Aplicação é um Warship único**, navegando no perigoso oceano aberto da internet, cada um com seu perfil de segurança e ciclo de vida próprios.

Cada Aplicação Warship contém os seguintes recursos:

1. Volume EFS criado e configurado, incluindo entrada DNS no Route53.
2. ELB frontend criado e configurado, incluindo entrada DNS no Route53.
3. Pipeline CodePipeline:

   1. Estágios de teste de unidade e integração pré-implantação
   2. Implantação e/ou atualização de containers via API do Nomad
   3. Estágios de teste de segurança e aceitação pós-implantação

### Implantação de Warship Pipeline

Para implantar os pipelines Nomadic Warship, use o diretório `warships` como template para o diretório da sua aplicação Warship.
Cada aplicação deve ter um diretório exclusivo.

Depois de configurar o template conforme sua aplicação, execute `terraform init` e `terraform apply` nesse diretório.
Isso criará o pipeline CodePipeline e os recursos EFS e ELB necessários para sua aplicação.
O pipeline implantará e/ou atualizará a aplicação Warship via API do Nomad.

Como cada Warship é apenas uma coleção de recursos Terraform, usuários avançados podem usar Terraform Remote State para gerenciar Warships Nomadic, ao invés de manter um diretório de implantação por aplicação.

### Configuração pré-implantação

As seguintes chaves do AWS SSM Parameter Store devem ser definidas antes de executar a implantação Nomadic:

1. `consul_encryption_key` — Lida por todos os nós Consul e necessária para quorum do cluster Consul.
2. `nomadic_ssh_key` — Usada para comunicação entre clusters e também para pipelines Warship realizarem mudanças na aplicação.

Além disso, é necessário garantir que as variáveis em `terraform.tfvars` estejam configuradas corretamente para o ambiente do usuário.
Por padrão, cada nó do cluster Nomadic será implantado em uma subnet e zona de disponibilidade separadas.
Usuários podem implantar em VPCs/Subnets existentes informando os valores correspondentes.
VPC e subnets são criadas automaticamente por padrão.

### Suporte Nomadic

Suporte Nomadic está disponível em: `nomadic at hex7 dot com`.

### Warship Not Whale.

![alt text](https://github.com/nand0p/nomadic/blob/master/images/nomadic.png?raw=true)

---

