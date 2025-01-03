# Troubleshooting a VPC 🌐

## Laborátório 🥼

## Objetivo 

Troubleshooting a VPC

- Criar logs de fluxo da VPC.

- Solucionar problemas de configuração da VPC.

- Analisar logs de fluxo.

### Diagrama do fluxo ✅


<img width="729" alt="Architecture" src="https://github.com/user-attachments/assets/9b5d46a9-a925-4ecc-86f3-5353aba43e34" />


# Execução 🚀

### Conectar-se à instância CLI Host
1. No Console de Gerenciamento da AWS, pesquise e abra o **EC2**.
2. Selecione **Instâncias** e escolha a instância CLI Host.
3. Clique em **Conectar-se** e, em seguida, na guia **EC2 Instance Connect**, clique em **Conectar-se** para abrir o terminal.

### Configurar a AWS CLI na instância CLI Host
1. No terminal do EC2 Instance Connect, configure o perfil da AWS CLI com o comando:
   ```bash
   aws configure
   ```
2. Insira as credenciais conforme solicitado:
   - **AWS Access Key ID**: Insira o valor de `AccessKey`.
   - **AWS Secret Access Key**: Insira o valor de `SecretKey`.
   - **Default region name**: Insira `us-west-2`.
   - **Default output format**: Insira `json`.

### Criar logs de fluxo da VPC
1. **Criar um bucket do S3**:
   ```bash
   aws s3api create-bucket --bucket flowlog###### --region 'us-west-2' --create-bucket-configuration LocationConstraint='us-west-2'
   ```
   Substitua `######` por seis números aleatórios.

2. **Obter o ID de VPC1**:
   ```bash
   aws ec2 describe-vpcs --query 'Vpcs[*].[VpcId,Tags[?Key==`Name`].Value,CidrBlock]' --filters "Name=tag:Name,Values='VPC1'"
   ```

3. **Criar logs de fluxo da VPC**:
   ```bash
   aws ec2 create-flow-logs --resource-type VPC --resource-ids <vpc-id> --traffic-type ALL --log-destination-type s3 --log-destination arn:aws:s3:::<flowlog######>
   ```
   Substitua `<flowlog######>` pelo nome do bucket e `<vpc-id>` pelo ID da VPC.

4. **Confirmar a criação do log de fluxo**:
   ```bash
   aws ec2 describe-flow-logs
   ```

### Solucionar problemas de configuração da VPC
1. **Verificar o acesso à instância do servidor web**:
   - Copie o endereço IP de WebServerIP e cole-o em uma nova guia do navegador.

2. **Descrever a instância do servidor web**:
   ```bash
   aws ec2 describe-instances --filter "Name=ip-address,Values='<WebServerIP>'"
   ```

3. **Filtrar detalhes relevantes**:
   ```bash
   aws ec2 describe-instances --filter "Name=ip-address,Values='<WebServerIP>'" --query 'Reservations[*].Instances[*].[State,PrivateIpAddress,InstanceId,SecurityGroups,SubnetId,KeyName]'
   ```

4. **Tentar conexão SSH com a instância do servidor web**:
   - No Console de Gerenciamento da AWS, selecione **Instâncias**.
   - Selecione a instância Café Web Server.
   - Clique em **Conectar-se** na guia **EC2 Instance Connect**.

### Desafio n.º 1 da solução de problemas

Para resolver o problema de carregamento da página web da instância do servidor web usando apenas a AWS CLI:

1. **Instale o utilitário Nmap na instância CLI Host**:
   ```bash
   sudo yum install -y nmap
   ```

2. **Verifique quais portas estão abertas na instância do servidor web**:
   ```bash
   nmap <WebServerIP>
   ```
   Substitua `<WebServerIP>` pelo endereço IP público real.

3. **Verifique os detalhes do grupo de segurança**:
   ```bash
   aws ec2 describe-security-groups --group-ids <WebServerSgId>
   ```

4. **Verifique as configurações da tabela de rota associada à sub-rede**:
   ```bash
   aws ec2 describe-route-tables --filter "Name=association.subnet-id,Values='<VPC1PubSubnetID>'"
   ```

   Verifique as rotas para garantir que há uma rota para o gateway da internet (Internet Gateway).

5. **Crie uma nova rota, se necessário**:
   ```bash
   aws ec2 create-route --route-table-id <route-table-id> --destination-cidr-block 0.0.0.0/0 --gateway-id <gateway-id>
   ```

6. **Recarregue a página da web no navegador**:
   A página deve exibir a mensagem “Hello From Your Web Server!”.

### Desafio n.º 2 da solução de problemas

Para resolver o problema de conexão SSH à instância do servidor web:

1. **Verifique as configurações da lista de controle de acesso de rede (ACL de rede)**:
   ```bash
   aws ec2 describe-network-acls --filter "Name=association.subnet-id,Values='<VPC1PublicSubnetID>'" --query 'NetworkAcls[*].[NetworkAclId,Entries]'
   ```

2. **Excluir entradas problemáticas da ACL de rede**:
   ```bash
   aws ec2 delete-network-acl-entry --network-acl-id <network-acl-id> --rule-number <rule-number>
   ```
   Substitua `<network-acl-id>` e `<rule-number>` pelos valores recuperados anteriormente.

3. **Tente se conectar novamente à instância do servidor web usando o EC2 Instance Connect**:
   - **Verifique a conexão SSH**:
     ```bash
     ssh ec2-user@<WebServerIP>
     ```
     Após conectar-se, execute o comando `hostname` para confirmar que você se conectou à instância correta.

### Tarefa 4: Analisar logs de fluxo

#### Tarefa 4.1: Baixar e extrair os logs de fluxo

1. **Crie um diretório local no terminal de CLI Host**:
   ```bash
   mkdir flowlogs
   ```
   
2. **Navegue para o novo diretório**:
   ```bash
   cd flowlogs
   ```
   
3. **Liste os buckets do S3 para encontrar o nome do bucket**:
   ```bash
   aws s3 ls
   ```
   
4. **Baixe os logs de fluxo do bucket do S3**:
   ```bash
   aws s3 cp s3://<flowlog######>/ . --recursive
   ```
   Substitua `<flowlog######>` pelo nome do bucket.

5. **Navegue para o subdiretório onde os logs foram baixados**:
   ```bash
   cd <AWSLogs/AccountID/vpcflowlogs/us-west-2/yyyy/mm/dd/>
   ```
   
6. **Extraia os arquivos de log**:
   ```bash
   gunzip *.gz
   ```

#### Tarefa 4.2: Analisar os logs

1. **Examine a estrutura dos logs**:
   ```bash
   head <file name>
   ```
   Substitua `<file name>` pelo nome de um arquivo retornado pelo comando `ls`.

2. **Procure eventos REJECT nos logs**:
   ```bash
   grep -rn REJECT .
   ```
   
3. **Conte o número de eventos REJECT**:
   ```bash
   grep -rn REJECT . | wc -l
   ```
   
4. **Refine a pesquisa para a porta 22 (SSH)**:
   ```bash
   grep -rn 22 . | grep REJECT
   ```

5. **Determine o endereço IP da sua máquina local**:
   - No Console de Gerenciamento da AWS, acesse os **Grupos de segurança**.
   - Edite as **Regras de entrada** do grupo **WebSecurityGroup** e adicione uma regra com a origem **Meu IP** para capturar o endereço IP.
   - Copie o endereço IP preenchido automaticamente, sem o sufixo `/32`.

6. **Refine a pesquisa nos logs para o seu endereço IP**:
   ```bash
   grep -rn 22 . | grep REJECT | grep <ip-address>
   ```
   Substitua `<ip-address>` pelo endereço IP capturado.

7. **Confirme o ID da interface de rede**:
   ```bash
   aws ec2 describe-network-interfaces --filters "Name=association.public-ip,Values='<WebServerIP>'" --query 'NetworkInterfaces[*].[NetworkInterfaceId,Association.PublicIp]'
   ```
   Substitua `<WebServerIP>` pelo endereço IP da instância do servidor web.

8. **Converta carimbos de data/hora Unix em formato legível**:
   ```bash
   date -d @<timestamp>
   ```
