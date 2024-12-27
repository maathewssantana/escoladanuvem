#Configurar uma VPC 🛜

## Laborátório 🥼

## Objetivo

- Criar uma VPC com uma sub-rede privada e uma sub-rede pública, um gateway de internet e um gateway NAT.

- Configurar as tabelas de rota associadas às sub-redes para o tráfego local e para o tráfego vinculado à internet usando um gateway de internet e um gateway NAT.

- Iniciar um servidor bastion na sub-rede pública.

- Usar um servidor bastion para fazer login em uma instância em uma sub-rede privada.

- Se tiver tempo, poderá concluir uma seção de desafio opcional na qual criará uma instância do Amazon EC2 em uma sub-rede privada e se conectará a ela por meio do servidor bastion.

### Diagrama do fluxo ✅


![architecture](https://github.com/user-attachments/assets/27c43ba0-404c-4280-82e6-44ec8bc17b69)


# Execução 🚀

###  Criar uma VPC e Sub-redes na AWS

#### **Tarefa 1: Criar uma VPC**
1. **Acessar Console de Gerenciamento de VPC**:
   - No Console de Gerenciamento da AWS, pesquise por **VPC** e selecione a opção.
2. **Criar Nova VPC**:
   - No painel de navegação à esquerda, selecione **Suas VPCs**.
   - Escolha **Criar VPC** e configure as opções:
     - **Recursos a serem criados**: Somente VPC
     - **Tag de nome**: Lab VPC
     - **IPv4 CIDR block**: Entrada manual de CIDR IPv4
     - **IPv4 CIDR**: 10.0.0.0/16
     - **IPv6 CIDR block**: Nenhum bloco CIDR IPv6
     - **Locação**: Padrão
   - Escolha **Criar VPC**.
   - Após criação bem-sucedida, selecione **Ações** e escolha **Editar configurações da VPC**.
   - Na seção **Configurações de DNS**, selecione **Habilitar nomes de host DNS** e clique em **Salvar**.

#### **Tarefa 2: Criar Sub-redes**
Você criará uma sub-rede pública e uma sub-rede privada.

##### **Tarefa 2.1: Criar uma Sub-rede Pública**
1. **Acessar Sub-redes**:
   - No painel de navegação à esquerda, selecione **Sub-redes**.
2. **Criar Sub-rede Pública**:
   - Escolha **Criar sub-rede** e configure as opções:
     - **ID da VPC**: Lab VPC
     - **Nome da sub-rede**: Public Subnet
     - **Zona de disponibilidade**: Escolha a primeira na lista
     - **IPv4 CIDR block**: 10.0.0.0/24
   - Selecione **Criar sub-rede**.
3. **Configurar IP Público Automático**:
   - Selecione a **Sub-rede pública**.
   - Escolha **Ações** e selecione **Editar configurações da sub-rede**.
   - Na seção **Configurações de atribuição automática de IP**, selecione **Enable auto-assign public IPv4 address**.
   - Selecione **Salvar**.
  
### Criar Sub-rede Privada, Gateway de Internet e Configurar Tabelas de Rota

#### **Tarefa 2.2: Criar uma Sub-rede Privada**
1. **Criar Sub-rede Privada**:
   - Repita as etapas da criação da sub-rede pública, mas configure as seguintes opções:
     - **ID da VPC**: Lab VPC
     - **Nome da sub-rede**: Private Subnet
     - **Zona de disponibilidade**: Escolha a primeira na lista
     - **IPv4 CIDR block**: 10.0.2.0/23
   - Selecione **Criar sub-rede**.

#### **Tarefa 3: Criar um Gateway de Internet**
1. **Criar Gateway de Internet**:
   - No painel de navegação à esquerda, selecione **Gateways da internet**.
   - Escolha **Criar gateway de Internet** e insira `Lab IGW` em Tag de nome.
   - Selecione **Criar gateway da Internet**.
   - Selecione **Ações** e escolha **Associar a uma VPC**.

#### **Tarefa 4: Configurar Tabelas de Rota**
1. **Criar Tabela de Rota Privada**:
   - No painel de navegação à esquerda, selecione **Tabelas de rotas**.
   - Escolha a tabela de rota que inclui `Lab VPC` na coluna VPC.
   - Na coluna Nome, selecione o ícone de edição, insira `Private Route Table` e escolha **Salvar**.
   - Selecione a guia **Rotas**. A rota existente mostra tráfego para 10.0.0/16 roteado localmente.

2. **Criar Tabela de Rota Pública**:
   - Selecione **Criar tabela de rotas** e configure as seguintes opções:
     - **Nome (opcional)**: Public Route Table
     - **VPC**: Lab VPC
   - Escolha **Criar tabela de rotas**.
   - Após criar a tabela, na guia **Rotas**, selecione **Editar rotas**.
   - Adicione uma rota para tráfego vinculado à internet:
     - **Destino**: 0.0.0.0/0
     - **Alvo**: Gateway da Internet (escolha `Lab IGW` na lista)
   - Selecione **Salvar alterações**.

3. **Associar Tabela de Rota à Sub-rede Pública**:
   - Selecione a guia **Associações de sub-rede**.
   - Escolha **Editar associações de sub-rede**.
   - Escolha **Sub-rede pública** e selecione **Salvar associações**.

Agora, sua sub-rede pública está configurada para enviar tráfego à internet por meio do gateway de internet.

#### **Tarefa 5: Iniciar um Servidor Bastion na Sub-rede Pública**
1. **Acessar Console EC2**:
   - No Console de Gerenciamento da AWS, pesquise por **EC2** e selecione a opção.
2. **Executar Instância**:
   - No painel de navegação à esquerda, selecione **Instâncias**.
   - Escolha **Executar instâncias** e configure as opções:
     - **Nome e tags**: Bastion Server
     - **Imagens de aplicação e SO (AMI)**: Amazon Linux 2023 AMI
     - **Tipo de instância**: t3.micro
     - **Par de chaves (login)**: Prosseguir sem um par de chaves
     - **Configurações de rede**:
       - **VPC**: Lab VPC
       - **Sub-rede**: Sub-rede pública
       - **Atribuir IP público automaticamente**: Habilitar
       - **Firewall (grupos de segurança)**: Criar grupo de segurança
         - **Nome do grupo de segurança**: Bastion Security Group
         - **Descrição**: Allow SSH
         - **Regras de entrada**:
           - **Tipo**: SSH
           - **Tipo de origem**: Qualquer lugar
   - Selecione **Executar instância** e, em seguida, **Visualizar todas as instâncias**.

#### **Tarefa 6: Criar um Gateway NAT**
1. **Acessar Gateways NAT**:
   - No Console de Gerenciamento da AWS, pesquise por **NAT gateways** e selecione a opção.
2. **Criar Gateway NAT**:
   - Selecione **Criar gateway NAT** e configure:
     - **Nome**: Lab NAT gateway
     - **Sub-rede**: Sub-rede pública
     - **Alocar IP elástico**
   - Escolha **Criar um gateway NAT**.
3. **Configurar Tabela de Rota Privada**:
   - No painel de navegação à esquerda, selecione **Tabelas de rotas** e escolha **Tabela de rotas privadas**.
   - Selecione a guia **Rotas** e escolha **Editar rotas**.
   - Adicione uma rota:
     - **Destino**: 0.0.0.0/0
     - **Alvo**: Gateway NAT (selecione o `nat-` na lista)
   - Selecione **Salvar alterações**.

Agora, os recursos na sub-rede privada podem se comunicar com a internet através do gateway NAT.

### Desafio Opcional: Testar a Sub-rede Privada

Este desafio opcional testa se uma instância EC2 na sub-rede privada pode se comunicar com a internet.

#### **Iniciar uma Instância na Sub-rede Privada**
1. **Configurar a Instância**:
   - No Console de Gerenciamento da AWS, vá para **EC2** e selecione **Executar instâncias**.
   - Configure as opções:
     - **Nome e tags**: Private Instance
     - **Imagens de aplicação e SO (AMI)**: Amazon Linux 2023 AMI
     - **Tipo de instância**: t3.micro
     - **Par de chaves (login)**: Prosseguir sem um par de chaves
     - **Configurações de rede**: Editar
       - **VPC**: Lab VPC
       - **Sub-rede**: Private Subnet
       - **Firewall (grupos de segurança)**: Criar grupo de segurança
         - **Nome**: Private Instance SG
         - **Descrição**: Allow SSH from Bastion
         - **Regras de entrada**:
           - **Tipo**: SSH
           - **Tipo de origem**: Personalizado
           - **Origem**: 10.0.0.0/16
     - **Detalhes avançados**: Cole o script em Dados do usuário:
       ```bash
       #!/bin/bash
       echo 'lab-password' | passwd ec2-user --stdin
       sed -i 's|[#]*PasswordAuthentication no|PasswordAuthentication yes|g' /etc/ssh/sshd_config
       systemctl restart sshd.service
       ```
   - Selecione **Executar instância** e, em seguida, **Visualizar todas as instâncias**.

#### **Fazer Login no Servidor Bastion**
1. **Acessar o Servidor Bastion**:
   - No Console de Gerenciamento da AWS, vá para **EC2** e selecione a instância **Bastion Server**.
   - Selecione **Conectar-se** usando **EC2 Instance Connect**.

#### **Fazer Login na Instância Privada**
1. **Conectar à Instância Privada**:
   - No Console de Gerenciamento do EC2, copie o endereço IPv4 privado da **Instância privada**.
   - No terminal do Servidor Bastion, execute:
     ```bash
     ssh PRIVATE-IP
     ```
     Substitua `PRIVATE-IP` pelo endereço IPv4 privado.
   - Quando solicitado, insira `yes` e depois `lab-password`.

#### **Testar o Gateway NAT**
1. **Verificar Acesso à Internet**:
   - Na instância privada, execute o comando:
     ```bash
     ping -c 3 amazon.com
     ```
   - Verifique a saída, que deve ser similar a:
     ```bash
     PING amazon.com (176.32.98.166) 56(84) bytes of data.
     64 bytes from 176.32.98.166: icmp_seq=1 ttl=222 time=79.2 ms
     64 bytes from 176.32.98.166: icmp_seq=2 ttl=222 time=79.2 ms
     64 bytes from 176.32.98.166: icmp_seq=3 ttl=222 time=79.0 ms
     ```

Essa saída confirma que a instância privada se comunicou com sucesso com a internet através do gateway NAT.

### Conclusão
Este desafio opcional confirma que a configuração da sub-rede privada e do gateway NAT foi bem-sucedida, permitindo que uma instância EC2 na sub-rede privada acesse a internet. 

Se precisar de mais informações ou tiver outras perguntas, estou aqui para ajudar! 😊
