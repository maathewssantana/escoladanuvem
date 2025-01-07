# Trabalhar com o Amazon S3 🖥️

## Laborátório 🥼

## Objetivo

- Usar os comandos s3api e s3 da AWS CLI para criar e configurar um bucket do S3.

- Verificar as permissões de gravação para um usuário em um bucket do S3.

- Configurar a notificação de evento em um bucket do S3. 

### Diagrama do fluxo ✅

<img width="534" alt="Architecture (1)" src="https://github.com/user-attachments/assets/57474a95-3de5-40db-9b18-b5fb73a444ed" />


# Execução 🚀

### Tarefa 1.1: Conectar-se à Instância do EC2 CLI Host
1. No Console de Gerenciamento da AWS, pesquise e selecione **EC2**.
2. No painel de navegação, selecione **Instâncias**.
3. Na lista de instâncias, selecione a instância **CLI Host**.
4. Selecione **Conectar-se**.
5. Na guia **EC2 Instance Connect**, selecione **Conectar-se**. Uma nova guia do navegador com a janela do terminal do EC2 Instance Connect será aberta.
6. Use essa janela do terminal para concluir as tarefas do laboratório. Se o terminal deixar de responder, atualize o navegador ou reconecte-se.

### Tarefa 1.2: Configurar a AWS CLI na Instância CLI Host
1. No terminal do EC2 Instance Connect, execute: `aws configure`
2. Nos prompts, insira as seguintes informações:
   - **AWS Access Key ID**: valor de `AccessKey`
   - **AWS Secret Access Key**: valor de `SecretKey`
   - **Default region name**: `us-west-2`
   - **Default output format**: `json`

### Tarefa 2: Criar e Inicializar o Bucket de Compartilhamento do S3
1. Para criar um bucket do S3, execute: 
   ```bash
   aws s3 mb s3://<cafe-xxxnnn> --region 'us-west-2'
   ```
   - Substitua `<cafe-xxxnnn>` pelo nome do bucket desejado.
2. Para carregar imagens no bucket S3 sob o prefixo `/images`, execute:
   ```bash
   aws s3 sync ~/initial-images/ s3://<cafe-xxxnnn>/images
   ```
   - Substitua `<cafe-xxxnnn>` pelo nome do bucket.
3. Para verificar se os arquivos foram sincronizados com o bucket S3, execute:
   ```bash
   aws s3 ls s3://<cafe-xxxnnn>/images/ --human-readable --summarize
   ```

### Tarefa 3: Revisar as Permissões do Usuário e do Grupo de Usuários do IAM

### Tarefa 3.1: Analisar o Grupo do IAM Mediaco
1. No Console de Gerenciamento da AWS, pesquise e selecione **IAM**.
2. No painel de navegação à esquerda, selecione **Grupos de usuários**.
3. Na lista, selecione **mediaco**.
4. Na página **Resumo do grupo mediaco**, selecione a guia **Permissões**.
5. Expanda a política **IAMUserChangePassword** e revise suas permissões.
6. Expanda a política **mediaCoPolicy** e observe as seguintes declarações:
   - **AllowGroupToSeeBucketListInTheConsole**: Permite visualizar a lista de buckets do S3.
   - **AllowRootLevelListingOfTheBucket**: Permite visualizar a lista de objetos de primeiro nível no bucket.
   - **AllowUserSpecificActionsOnlyInTheSpecificPrefix**: Permite ações específicas (GetObject, PutObject, DeleteObject) em objetos da pasta `cafe-*/images/*`.

### Tarefa 3.2: Analisar o Usuário MediacoUser do IAM
1. No painel de navegação do IAM, selecione **Usuários**.
2. Na lista, selecione **mediacouser**.
3. Na guia **Permissões**, revise as políticas **IAMUserChangePassword** e **mediaCoPolicy**.
4. Na guia **Grupos**, verifique a associação ao grupo **mediaco**.
5. Na guia **Credenciais de segurança**, crie uma nova chave de acesso:
   - Selecione **Criar chave de acesso**.
   - Escolha **Command Line Interface (CLI)** e confirme.
   - Baixe o arquivo `.csv`.
6. Copie o **Link de login do console**.

### Tarefa 3.3: Testar as Permissões do MediacoUser
1. Faça login no Console da AWS como **mediacouser** utilizando o link de login e as credenciais fornecidas.
2. No Console da AWS, pesquise e selecione **S3**.
3. Na lista de buckets, selecione o bucket criado.
4. Para **visualizar** uma imagem, selecione `Donuts.jpg` e escolha **Abrir**.
5. Para **fazer upload** de uma imagem, selecione **Fazer upload**, adicione um arquivo e confirme.
6. Para **excluir** uma imagem, selecione `Cup-of-Hot-Chocolate.jpg`, escolha **Excluir** e confirme.
7. Para testar o caso de uso não autorizado, tente alterar as permissões do bucket na guia **Permissões**. A mensagem “Permissões insuficientes” deve aparecer.


### Tarefa 4: Configurar Notificações de Eventos no Bucket de Compartilhamento do S3

### Tarefa 4.1: Criar e Configurar o Tópico do SNS `s3NotificationTopic`
1. No Console de Gerenciamento da AWS, pesquise e selecione **SNS** para abrir o console do Simple Notification Service.
2. No painel de navegação, selecione **Tópicos**.
3. Selecione **Criar tópico** e escolha **Padrão**.
4. Em **Nome**, insira `s3NotificationTopic` e selecione **Criar tópico**.
5. Na página `s3NotificationTopic`, copie o valor **ARN** para um editor de texto.
6. Para configurar a política de acesso do tópico, selecione **Editar**.
7. Expanda a seção **Política de acesso – opcional** e substitua o conteúdo pelo seguinte JSON (substitua `<ARN do s3NotificationTopic>` e `<cafe-xxxnnn>` pelos valores corretos):
   ```json
   {
     "Version": "2008-10-17",
     "Id": "S3PublishPolicy",
     "Statement": [
       {
         "Sid": "AllowPublishFromS3",
         "Effect": "Allow",
         "Principal": {
           "Service": "s3.amazonaws.com"
         },
         "Action": "SNS:Publish",
         "Resource": "<ARN do s3NotificationTopic>",
         "Condition": {
           "ArnLike": {
             "aws:SourceArn": "arn:aws:s3:*:*:<cafe-xxxnnn>"
           }
         }
       }
     ]
   }
   ```
8. Selecione **Salvar alterações**.
9. Para se inscrever no tópico, vá para a guia **Assinaturas**, selecione **Criar assinatura**, escolha **E-mail** como protocolo e insira seu endereço de e-mail.
10. Confirme a assinatura através do e-mail de confirmação recebido.

### Tarefa 4.2: Adicionar uma Configuração de Notificação de Eventos ao Bucket do S3
1. Na janela do terminal da instância CLI Host, edite um novo arquivo chamado `s3EventNotification.json`:
   ```bash
   vi s3EventNotification.json
   ```
2. No editor, insira o seguinte JSON (substitua `<ARN de s3NotificationTopic>` pelo valor correto):
   ```json
   {
     "TopicConfigurations": [
       {
         "TopicArn": "<ARN de s3NotificationTopic>",
         "Events": ["s3:ObjectCreated:*","s3:ObjectRemoved:*"],
         "Filter": {
           "Key": {
             "FilterRules": [
               {
                 "Name": "prefix",
                 "Value": "images/"
               }
             ]
           }
         }
       }
     ]
   }
   ```
3. Salve o arquivo e saia do editor (`:wq`).
4. Para associar o arquivo de configuração ao bucket do S3, execute:
   ```bash
   aws s3api put-bucket-notification-configuration --bucket <cafe-xxxnnn> --notification-configuration file://s3EventNotification.json
   ```
   - Substitua `<cafe-xxxnnn>` pelo nome do bucket.
5. Verifique a caixa de entrada do e-mail para ver a notificação de teste do Amazon S3.

### Tarefa 5: Testar as Notificações de Eventos do Bucket de Compartilhamento do S3

1. **Configurar as Credenciais do MediacoUser na Instância CLI Host**
   - No terminal SSH da instância CLI Host, execute: 
     ```bash
     aws configure
     ```
   - Nos prompts, insira os valores das credenciais do mediacouser (do arquivo baixado na Tarefa 3):
     - **AWS Access Key ID**: `<valor do ID da chave de acesso>`
     - **AWS Secret Access Key**: `<valor da Chave de acesso secreta>`
     - **Default region name**: pressione `Enter` para manter `us-west-2`.
     - **Default output format**: insira `json`.

2. **Testar o Caso de Uso "Put" (Fazer Upload)**
   - Execute o comando para fazer upload de uma imagem:
     ```bash
     aws s3api put-object --bucket <cafe-xxxnnn> --key images/Caramel-Delight.jpg --body ~/new-images/Caramel-Delight.jpg
     ```
   - Verifique a caixa de entrada do e-mail para uma notificação com o assunto "Notificação do Amazon S3".
   - A mensagem deve indicar `ObjectCreated:Put` para `images/Caramel-Delight.jpg`.

3. **Testar o Caso de Uso "Get" (Obter Objeto)**
   - Execute o comando para obter um objeto:
     ```bash
     aws s3api get-object --bucket <cafe-xxxnnn> --key images/Donuts.jpg Donuts.jpg
     ```
   - Observe que não é gerada uma notificação por e-mail para essa operação.

4. **Testar o Caso de Uso "Delete" (Excluir Objeto)**
   - Execute o comando para excluir um objeto:
     ```bash
     aws s3api delete-object --bucket <cafe-xxxnnn> --key images/Strawberry-Tarts.jpg
     ```
   - Verifique a caixa de entrada do e-mail para uma notificação com o assunto "Notificação do Amazon S3".
   - A mensagem deve indicar `ObjectRemoved:Delete` para `images/Strawberry-Tarts.jpg`.

5. **Testar um Caso de Uso Não Autorizado**
   - Tente alterar a permissão de um objeto:
     ```bash
     aws s3api put-object-acl --bucket <cafe-xxxnnn> --key images/Donuts.jpg --acl public-read
     ```
   - O comando deve falhar com a mensagem de erro: “Ocorreu um erro (AccessDenied) ao chamar a operação PutObjectAcl: acesso negado”.
