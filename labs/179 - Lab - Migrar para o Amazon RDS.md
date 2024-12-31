# Migrar para o Amazon RDS 🖥️

## Laborátório 🥼

## Objetivo

- Criar uma instância do MariaDB do Amazon RDS usando a AWS CLI.

- Migrar dados de um banco de dados do MariaDB em uma instância do EC2 para uma instância do MariaDB do Amazon RDS.

- Monitorar a instância do Amazon RDS usando as métricas do Amazon CloudWatch.

### Diagrama do fluxo ✅

Arquitetura inicial

O diagrama a seguir ilustra a topologia do ambiente de runtime do aplicativo web da cafeteria antes da migração. O banco de dados da aplicação é executado em uma instância do Linux do Amazon Elastic Compute Cloud (Amazon EC2), do Apache, do MySQL e do PHP (LAMP) com o código da aplicação. A instância é do tipo T3 small e é executada em uma sub-rede pública para que os clientes da internet possam acessar o site. Uma instância CLI Host reside na mesma sub-rede para viabilizar a administração da instância usando a AWS Command Line Interface (AWS CLI).

<img width="408" alt="StartingArchitecture" src="https://github.com/user-attachments/assets/f367a8d8-0438-4055-adb6-ff4c8aac42c4" />

Arquitetura final

O diagrama a seguir ilustra a topologia do ambiente de runtime do aplicativo web da cafeteria depois da migração.
Você migrará o banco de dados local da cafeteria para um banco de dados do Amazon RDS que reside fora da instância. O banco de dados do Amazon RDS será implantado na mesma nuvem privada virtual (VPC) da instância.

<img width="758" alt="FinalArchitecture" src="https://github.com/user-attachments/assets/66946a03-a9ec-4db2-8613-35710febf28c" />


# Execução 🚀


### Tarefa 1: Gerar dados de pedidos no site da cafeteria

1. Acesse o site da cafeteria com o URL `CafeInstanceURL`.
2. No site, selecione Menu (Cardápio), adicione itens ao pedido e envie.
3. Registre o número de pedidos na página Histórico de pedidos.

### Tarefa 2: Criar uma instância do Amazon RDS usando a AWS CLI

#### Tarefa 2.1: Conectar-se à instância CLI Host

1. No Console de Gerenciamento da AWS, vá para EC2 e selecione Instâncias.
2. Selecione a instância CLI Host e clique em Conectar-se.

#### Tarefa 2.2: Configurar a AWS CLI

1. Conecte-se à instância CLI Host usando EC2 Instance Connect.
2. Configure a AWS CLI com as credenciais fornecidas:
    ```sh
    aws configure
    ```

3. Insira as seguintes informações:
    - **AWS Access Key ID**
    - **AWS Secret Access Key**
    - **Default region name**
    - **Default output format** (json)

### Componentes a serem criados:

- Um grupo de segurança para a instância do Amazon RDS.
- Duas sub-redes privadas e um grupo de sub-redes de banco de dados.
- Uma instância do MariaDB do Amazon RDS.

### Passos para Criar Componentes Obrigatórios

#### 1. Criar o grupo de segurança CafeDatabaseSG:

```sh
aws ec2 create-security-group \
--group-name CafeDatabaseSG \
--description "Security group for Cafe database" \
--vpc-id <CafeInstance VPC ID>
```

Guarde o `GroupId` gerado para uso posterior.

#### 2. Criar a regra de entrada para o grupo de segurança:

```sh
aws ec2 authorize-security-group-ingress \
--group-id <CafeDatabaseSG Group ID> \
--protocol tcp --port 3306 \
--source-group <CafeSecurityGroup Group ID>
```

#### 3. Verificar se a regra de entrada foi aplicada:

```sh
aws ec2 describe-security-groups \
--query "SecurityGroups[*].[GroupName,GroupId,IpPermissions]" \
--filters "Name=group-name,Values='CafeDatabaseSG'"
```

#### 4. Criar a sub-rede privada 1 CafeDB:

```sh
aws ec2 create-subnet \
--vpc-id <CafeInstance VPC ID> \
--cidr-block 10.200.2.0/23 \
--availability-zone <CafeInstance Availability Zone>
```

Guarde o `SubnetId` gerado para uso posterior.

#### 5. Criar a sub-rede privada 2 CafeDB:

```sh
aws ec2 create-subnet \
--vpc-id <CafeInstance VPC ID> \
--cidr-block 10.200.10.0/23 \
--availability-zone <availability-zone>
```

Guarde o `SubnetId` gerado para uso posterior.

#### 6. Criar o grupo de sub-redes CafeDB:

```sh
aws rds create-db-subnet-group \
--db-subnet-group-name "CafeDB Subnet Group" \
--db-subnet-group-description "DB subnet group for Cafe" \
--subnet-ids <Cafe Private Subnet 1 ID> <Cafe Private Subnet 2 ID> \
--tags "Key=Name,Value=CafeDatabaseSubnetGroup"
```

### Criar a instância do MariaDB do Amazon RDS

1. **Comando para criar a instância**:

   ```sh
   aws rds create-db-instance \
   --db-instance-identifier CafeDBInstance \
   --engine mariadb \
   --engine-version 10.5.13 \
   --db-instance-class db.t3.micro \
   --allocated-storage 20 \
   --availability-zone <CafeInstance Availability Zone> \
   --db-subnet-group-name "CafeDB Subnet Group" \
   --vpc-security-group-ids <CafeDatabaseSG Group ID> \
   --no-publicly-accessible \
   --master-username root --master-user-password 'Re:Start!9'
   ```

   Substitua `<CafeInstance Availability Zone>` pelo valor registrado anteriormente e `<CafeDatabaseSG Group ID>` pelo valor do ID do grupo de segurança criado.

2. **Monitorar o status da instância de banco de dados**:

   ```sh
   aws rds describe-db-instances \
   --db-instance-identifier CafeDBInstance \
   --query "DBInstances[*].[Endpoint.Address,AvailabilityZone,PreferredBackupWindow,BackupRetentionPeriod,DBInstanceStatus]"
   ```

   Esse comando exibe as seguintes informações:

   - Endereço do endpoint
   - Zona de Disponibilidade
   - Janela de backup preferencial
   - Período de retenção de backup
   - Status do banco de dados

3. **Aguarde até o status da instância ser "available"**:
   - O status inicial será `creating`, depois `modifying`, e `backing-up`, até finalmente chegar a `available`.

4. **Registrar o valor do endpoint**:
   - Anote o endereço do endpoint usando o formato: `RDS Instance Database Endpoint Address: cafedbinstance.xxxxxxx.us-west-2.rds.amazonaws.com`

Continue repetindo o comando de monitoramento até o status exibir `available`. Boa sorte com a criação da sua instância do MariaDB! Se precisar de mais alguma coisa, estou por aqui.

### Tarefa 3: Migrar dados da aplicação para a instância do Amazon RDS

**Passos principais**:

#### 1. Conectar-se à CafeInstance
- Use o EC2 Instance Connect para se conectar à instância CLI Host conforme fez anteriormente.

#### 2. Criar um backup do banco de dados local
- Execute o seguinte comando para criar o backup usando `mysqldump`:
   ```sh
   mysqldump --user=root --password='Re:Start!9' \
   --databases cafe_db --add-drop-database > cafedb-backup.sql
   ```

#### 3. Examinar o arquivo de backup (opcional)
- Para visualizar o conteúdo do arquivo de backup, use o comando:
   ```sh
   less cafedb-backup.sql
   ```

#### 4. Restaurar o backup para o banco de dados do Amazon RDS
- Execute o seguinte comando para restaurar o backup no RDS:
   ```sh
   mysql --user=root --password='Re:Start!9' \
   --host=<RDS Instance Database Endpoint Address> \
   < cafedb-backup.sql
   ```

#### 5. Verificar a migração de dados
- Abra uma sessão interativa do MySQL na instância RDS e recupere os dados:
   ```sh
   mysql --user=root --password='Re:Start!9' \
   --host=<RDS Instance Database Endpoint Address> \
   cafe_db
   ```
- Execute a consulta para verificar os dados:
   ```sql
   select * from product;
   ```

#### 6. Sair da sessão do MySQL interativa
- Para sair da sessão, use o comando:
   ```sh
   exit
   ```
### Tarefa 4: Configurar o site para usar a instância do Amazon RDS

**Passos principais**:

1. **Acessar o AWS Systems Manager**:
   - No Console de Gerenciamento da AWS, pesquise e selecione **Systems Manager**.
   - No painel de navegação à esquerda, selecione **Armazenamento de parâmetros**.

2. **Editar o parâmetro dbUrl**:
   - Na lista Meus parâmetros, selecione `/cafe/dbUrl`.
   - Selecione **Editar**.
   - Na página Detalhes do parâmetro, substitua o valor pelo endereço do endpoint do banco de dados da instância do RDS.
   - Selecione **Salvar alterações**.

3. **Testar o site da cafeteria**:
   - Em uma nova janela do navegador, cole o URL `CafeInstanceURL`.
   - Verifique a página inicial do site e a guia **Histórico de pedidos**.
   - Compare o número de pedidos com o número anterior à migração.
   - Opcional: Faça novos pedidos para confirmar o funcionamento.

### Tarefa 5: Monitorar o banco de dados do Amazon RDS

**Passos principais**:

1. **Acessar o Amazon RDS**:
   - No Console de Gerenciamento da AWS, pesquise e selecione **RDS**.
   - No painel de navegação à esquerda, selecione **Bancos de dados**.
   - Selecione `cafedbinstance` na lista.

2. **Monitorar as métricas de desempenho**:
   - Selecione a guia **Monitoramento**.
   - Visualize as métricas como `CPUUtilization`, `DatabaseConnections`, `FreeStorageSpace`, `FreeableMemory`, `WriteIOPS`, `ReadIOPS`.

3. **Criar uma conexão com o banco de dados do RDS**:
   - No terminal da `CafeInstance`, execute:
     ```sh
     mysql --user=root --password='Re:Start!9' \
     --host=<RDS Instance Database Endpoint Address> \
     cafe_db
     ```

4. **Recuperar dados na tabela de produtos**:
   - Execute a declaração SQL:
     ```sql
     select * from product;
     ```

5. **Verificar a métrica `DatabaseConnections`**:
   - No console do Amazon RDS, selecione o grafo **DatabaseConnections**.
   - Aguarde um minuto e selecione **Atualizar** se necessário.

6. **Fechar a sessão do SQL interativa**:
   - No terminal `CafeInstance`, digite:
     ```sh
     exit
     ```
   - Aguarde um minuto e atualize o grafo `DatabaseConnections` no console do RDS.
