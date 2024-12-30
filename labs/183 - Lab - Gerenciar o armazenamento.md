# Gerenciar o armazenamento 💻

## Laborátório 🥼

## Objetivo

Gerenciar o armazenamento

- Criar e manter snapshots para instâncias do Amazon EC2.

- Usar a sincronização do Amazon S3 para copiar arquivos de um volume do EBS para um bucket do S3.

- Usar o versionamento do Amazon S3 para recuperar arquivos excluídos.


### Diagrama do fluxo ✅


<img width="769" alt="architecture (1)" src="https://github.com/user-attachments/assets/06263107-7683-4ffe-a70f-71c70cb20305" />


# Execução 🚀

### Tarefa 1: Criar e configurar recursos
Nesta tarefa, você criará um bucket do Amazon S3 e configurará a instância do EC2 chamada “Command Host” para ter acesso seguro a outros recursos da AWS.

#### Tarefa 1.1: Criar um bucket do S3
- Crie um bucket no Amazon S3 usando o Console de Gerenciamento da AWS.
- Nomeie o bucket como “s3-bucket-name”.
- Use a região padrão e conclua a criação do bucket.

#### Tarefa 1.2: Anexar perfil de instância a Processor
- Anexe um perfil do IAM à instância do EC2 “Processor” para conceder permissões.
- Selecione o perfil `S3BucketAccess` no Console de Gerenciamento do EC2.

### Tarefa 2: Gerar snapshots da instância
Usando a AWS CLI para gerenciar snapshots da instância.

#### Tarefa 2.1: Conectar-se à instância do EC2 chamada Command Host
- Use o EC2 Instance Connect para se conectar à instância “Command Host”.
- Conecte-se através do Console de Gerenciamento da AWS ou use um cliente SSH seguindo as instruções fornecidas.

### Tarefa 2.2: Gerar um snapshot inicial

Nesta tarefa, você identificará o volume do EBS que está anexado à instância “Processor” e gerará um snapshot inicial. Aqui estão as etapas detalhadas:

1. **Obter o ID do volume EBS**
   Execute o comando para obter o VolumeId:
   ```sh
   aws ec2 describe-instances --filter 'Name=tag:Name,Values=Processor' --query 'Reservations[0].Instances[0].BlockDeviceMappings[0].Ebs.{VolumeId:VolumeId}'
   ```
   Este comando retorna algo como:
   ```json
   {
     "VolumeId": "vol-1234abcd"
   }
   ```
   Use `VolumeId` nas etapas seguintes.

2. **Obter o ID da instância “Processor”**
   Execute o comando para obter o InstanceId:
   ```sh
   aws ec2 describe-instances --filters 'Name=tag:Name,Values=Processor' --query 'Reservations[0].Instances[0].InstanceId'
   ```
   Este comando retorna algo como:
   ```json
   {
     "InstanceId": "i-0b06965263c7ac08f"
   }
   ```

3. **Encerrar a instância “Processor”**
   Execute o comando para parar a instância:
   ```sh
   aws ec2 stop-instances --instance-ids INSTANCE-ID
   ```
   Substitua `INSTANCE-ID` pelo ID da instância obtido anteriormente.

4. **Verificar se a instância parou**
   Execute o comando para esperar até que a instância pare:
   ```sh
   aws ec2 wait instance-stopped --instance-id INSTANCE-ID
   ```

5. **Criar o snapshot do volume**
   Execute o comando para criar o snapshot:
   ```sh
   aws ec2 create-snapshot --volume-id VOLUME-ID
   ```
   Substitua `VOLUME-ID` pelo ID do volume obtido anteriormente. O comando retorna um SnapshotId como:
   ```json
   {
     "SnapshotId": "snap-0643809e73e6cce13"
   }
   ```

6. **Verificar o status do snapshot**
   Execute o comando para verificar o status do snapshot:
   ```sh
   aws ec2 wait snapshot-completed --snapshot-id SNAPSHOT-ID
   ```
   Substitua `SNAPSHOT-ID` pelo ID do snapshot obtido anteriormente.

7. **Reiniciar a instância “Processor”**
   Execute o comando para iniciar a instância:
   ```sh
   aws ec2 start-instances --instance-ids INSTANCE-ID
   ```
### Tarefa 2.3: Programar a criação de snapshots subsequentes

Vamos configurar um processo de snapshot recorrente usando cron no Linux.

1. **Criar e programar um registro do cron**
   Execute o comando para criar um trabalho cron que gera um snapshot a cada minuto:
   ```sh
   echo "* * * * *  aws ec2 create-snapshot --volume-id VOLUME-ID 2>&1 >> /tmp/cronlog" > cronjob
   crontab cronjob
   ```
   Substitua `VOLUME-ID` pelo ID do volume EBS que você recuperou anteriormente.

2. **Verificar a criação de snapshots subsequentes**
   Para verificar se os snapshots estão sendo criados, execute:
   ```sh
   aws ec2 describe-snapshots --filters "Name=volume-id,Values=VOLUME-ID"
   ```
   Aguarde alguns minutos e execute novamente o comando para ver mais snapshots.

### Tarefa 2.4: Reter os últimos dois snapshots

Agora, vamos executar um script Python para manter apenas os dois últimos snapshots.

1. **Interromper o trabalho cron**
   Execute o comando para parar o trabalho cron:
   ```sh
   crontab -r
   ```

2. **Examinar o script Python `snapshotter_v2.py`**
   Visualize o conteúdo do script:
   ```sh
   more /home/ec2-user/snapshotter_v2.py
   ```

   O script gera snapshots de todos os volumes EBS associados à sua conta e remove todos, exceto os dois snapshots mais recentes.

3. **Verificar os snapshots existentes**
   Execute o comando para listar os snapshots do volume:
   ```sh
   aws ec2 describe-snapshots --filters "Name=volume-id,Values=VOLUME-ID" --query 'Snapshots[*].SnapshotId'
   ```

4. **Executar o script `snapshotter_v2.py`**
   Execute o script Python:
   ```sh
   python3 snapshotter_v2.py
   ```

   O script retornará uma lista dos snapshots que foram excluídos.

5. **Verificar novamente os snapshots**
   Execute o comando novamente para verificar os snapshots restantes:
   ```sh
   aws ec2 describe-snapshots --filters "Name=volume-id,Values=VOLUME-ID" --query 'Snapshots[*].SnapshotId'
   ```
### Tarefa 3: Desafio: Sincronizar arquivos com o Amazon S3

Para este desafio, você precisará sincronizar o conteúdo de um diretório com o bucket do Amazon S3 que você criou anteriormente. Aqui está um resumo das etapas para alcançar isso:

### Descrição do Desafio

1. **Baixar os arquivos de exemplo**
   ```sh
   wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-RSJAWS-1-23732/183-lab-JAWS-managing-storage/s3/files.zip
   ```

2. **Descompactar os arquivos**
   ```sh
   unzip files.zip
   ```

3. **Ativar o versionamento para o bucket do Amazon S3**
   ```sh
   aws s3api put-bucket-versioning --bucket S3-BUCKET-NAME --versioning-configuration Status=Enabled
   ```

4. **Sincronizar o conteúdo da pasta descompactada com o bucket do Amazon S3**
   ```sh
   aws s3 sync files s3://S3-BUCKET-NAME/files/
   ```

5. **Modificar o comando para excluir um arquivo do Amazon S3 quando o arquivo correspondente for excluído localmente**
   ```sh
   rm files/file1.txt
   aws s3 sync files s3://S3-BUCKET-NAME/files/ --delete
   ```

6. **Recuperar o arquivo excluído do Amazon S3 usando versionamento**
   - Listar versões do objeto:
     ```sh
     aws s3api list-object-versions --bucket S3-BUCKET-NAME --prefix files/file1.txt
     ```

   - Recuperar a versão anterior:
     ```sh
     aws s3api get-object --bucket S3-BUCKET-NAME --key files/file1.txt --version-id VERSION-ID files/file1.txt
     ```

   - Sincronizar novamente:
     ```sh
     aws s3 sync files s3://S3-BUCKET-NAME/files/
     ```
