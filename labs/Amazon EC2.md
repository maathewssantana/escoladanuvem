# Criar uma EC2 🖥️

## Laborátório 🥼

## Objetivo

- Executar uma instância do EC2 usando o Console de Gerenciamento da AWS.

- Conectar-se à instância do EC2 usando o EC2 Instance Connect.

- Iniciar uma instância do EC2 usando a AWS CLI.

### Diagrama do fluxo ✅

![EDN-EC2-CLI drawio](https://github.com/user-attachments/assets/d5f44b3f-e111-4773-a76c-2ab03deadd50)


# Execução 🚀


**Etapa 1: Escolher um Nome e Tags**
1. Use tags para categorizar recursos da AWS, como por finalidade ou ambiente.
2. Nomeie sua instância; a AWS criará um par de chave-valor com a chave "Name".
3. Insira "Bastion host" na seção Nome e tags.

**Etapa 2: Escolher uma AMI**
1. Selecione uma Imagem de Máquina da Amazon (AMI).
2. Use "Amazon Linux 2" na seção Application and OS Images (Amazon Machine Image).

**Etapa 3: Escolher um Tipo de Instância**
1. Escolha o tipo de instância com base em recursos necessários.
2. Selecione "t3.micro" na lista suspensa Tipo de instância.

**Etapa 4: Configurar um Par de Chaves**
1. EC2 usa criptografia de chave pública para login.
2. Para este laboratório, use EC2 Instance Connect e selecione "Prosseguir sem um par de chaves".

**Etapa 5: Definir as Configurações de Rede**
1. Selecione a VPC e sub-rede apropriadas (use "Lab VPC" e "Sub-rede pública").
2. Configure o grupo de segurança com nome "Bastion security group" e descrição "Permit SSH connections".

**Etapa 6: Adicionar Armazenamento**
1. Utilize o volume de disco padrão de 8 GiB.
2. Mantenha a configuração padrão no painel Configurar armazenamento.

**Etapa 7: Configurar Detalhes Avançados**
1. Expanda o painel Detalhes avançados.
2. Selecione "Bastion-Role" na lista suspensa Perfil de instância do IAM.
3. Mantenha as configurações padrão para os demais valores.

**Etapa 8: Iniciar uma Instância do EC2**
1. Revise os detalhes de configuração.
2. Escolha "Executar instância" e visualize todas as instâncias.

---

### Tarefa 2: Fazer Login no Host Bastion
1. Use o EC2 Instance Connect no Console de Gerenciamento do EC2.
2. Selecione a instância do host bastion e conecte-se a ela.

---

### Tarefa 3: Iniciar uma Instância do EC2 usando a AWS CLI

**Etapa 1: Recuperar a AMI**
1. Use AWS Systems Manager Parameter Store para obter o ID da AMI mais recente.
2. Execute o script para recuperar a AMI:
   ```bash
   AZ=`curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone`
   export AWS_DEFAULT_REGION=${AZ::-1}
   AMI=$(aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --query 'Parameters[0].[Value]' --output text)
   echo $AMI
   ```

**Etapa 2: Recuperar a Sub-rede**
1. Execute o comando para obter o ID da sub-rede pública:
   ```bash
   SUBNET=$(aws ec2 describe-subnets --filters 'Name=tag:Name,Values=Public Subnet' --query Subnets[].SubnetId --output text)
   echo $SUBNET
   ```

---

### Tutorial Resumido

**Etapa 3: Recuperar o Grupo de Segurança**
1. Execute o seguinte comando para recuperar o ID do grupo de segurança da web:
   ```bash
   SG=$(aws ec2 describe-security-groups --filters Name=group-name,Values=WebSecurityGroup --query SecurityGroups[].GroupId --output text)
   echo $SG
   ```

**Etapa 4: Baixar um Script de Dados do Usuário**
1. Baixe o script de dados do usuário:
   ```bash
   wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-RSJAWS-1-23732/171-lab-JAWS-create-ec2/s3/UserData.txt
   ```
2. Visualize o conteúdo do script:
   ```bash
   cat UserData.txt
   ```
   O script faz o seguinte:
   - Instala um servidor web.
   - Baixa um arquivo .zip que contém o aplicativo web.
   - Instala o aplicativo web.

**Etapa 5: Iniciar a Instância**
1. Execute o seguinte comando para iniciar a instância:
   ```bash
   INSTANCE=$(\
   aws ec2 run-instances \
   --image-id $AMI \
   --subnet-id $SUBNET \
   --security-group-ids $SG \
   --user-data file:///home/ec2-user/UserData.txt \
   --instance-type t3.micro \
   --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Web Server}]' \
   --query 'Instances[*].InstanceId' \
   --output text \
   )
   echo $INSTANCE
   ```

**Etapa 6: Aguardar até que a Instância Esteja Pronta**
1. Monitore o status da instância usando a AWS CLI:
   ```bash
   aws ec2 describe-instances --instance-ids $INSTANCE
   ```
2. Para verificar o estado da instância, execute o seguinte comando:
   ```bash
   aws ec2 describe-instances --instance-ids $INSTANCE --query 'Reservations[].Instances[].State.Name' --output text
   ```
   Repita este comando até que ele retorne um status "running".

**Etapa 7: Testar o Servidor Web**
1. Recupere o URL público da instância:
   ```bash
   aws ec2 describe-instances --instance-ids $INSTANCE --query Reservations[].Instances[].PublicDnsName --output text
   ```
2. Copie o nome do DNS exibido e cole em uma nova guia do navegador da web. Pressione Enter para verificar se o servidor web foi iniciado com sucesso.

Espero que isso ajude! Se precisar de mais alguma coisa, estou à disposição. 😊
