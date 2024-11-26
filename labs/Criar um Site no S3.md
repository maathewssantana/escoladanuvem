# Criar um site no S3 🪣

## Laborátório 🥼

## Objetivo

- Executar comandos da AWS CLI que usam os serviços do IAM e do Amazon S3.
- Implantar um site estático em um bucket do S3.
- Criar um script que use a AWS CLI para copiar arquivos de um diretório local para o Amazon S3.

### Diagrama do fluxo ✅

![Criar um Site S3 drawio](https://github.com/user-attachments/assets/aca04ebe-4e6d-4aef-a28e-e1561630f15a)


# Site Original 🌐
![site-café](https://github.com/user-attachments/assets/7379c8fe-0547-49f1-a753-cd9a08454520)

# Execução 🚀

### Tutorial de Uso

#### 1. Conectar-se a uma Instância do Amazon Linux EC2 usando SSM
- Conecte-se à instância do Amazon EC2 através de um URL usando o Session Manager do AWS Systems Manager.

#### 2. Configurar a AWS CLI
- Após a conexão na instância, configure a AWS CLI com o comando `aws configure`.
- Utilize a região `us-west-2` e o formato de saída `json`.

#### 3. Criar um Bucket do S3
- Crie um bucket no S3 usando o comando:
  ```bash
  aws s3api create-bucket --bucket <mybucket> --region us-west-2 --create-bucket-configuration LocationConstraint=us-west-2
  ```

#### 4. Criar um Usuário do IAM com Acesso Total ao Bucket S3
- Crie um usuário do IAM:
  ```bash
  aws iam create-user --user-name awsS3user
  ```
- Configure um perfil de login para o novo usuário:
  ```bash
  aws iam create-login-profile --user-name awsS3user --password Training123!
  ```
- Liste as políticas de IAM:
  ```bash
  aws iam list-policies --query "Policies[?contains(PolicyName,'S3')]"
  ```
  (Utilize a política `AmazonS3FullAccess`)
- Conceda acesso total ao usuário criado:
  ```bash
  aws iam attach-user-policy --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess --user-name awsS3user
  ```

#### 5. Extrair os Arquivos do Site Estático
- Volte para a instância local e extraia os arquivos do site estático:
  ```bash
  cd ~/sysops-activity-files
  tar xvzf static-website-v2.tar.gz
  cd static-website
  ```

#### 6. Fazer Upload dos Arquivos para o Bucket S3
- Configure o bucket para hospedar um site:
  ```bash
  aws s3 website s3://<my-bucket>/ --index-document index.html
  ```
- Realize o upload dos arquivos para o bucket:
  ```bash
  aws s3 cp /home/ec2-user/sysops-activity-files/static-website/ s3://<my-bucket>/ --recursive --acl public-read
  ```

#### 7. Listar os Arquivos no Bucket
- Liste os arquivos presentes no bucket:
  ```bash
  aws s3 ls <my-bucket>
  ```

![arquivosbucket](https://github.com/user-attachments/assets/98e209be-c7a8-4931-876a-8b131d505c55)

#### 8. Criar um Arquivo em Lote para Atualização do Site
- Digite `history` no terminal para visualizar o histórico de comandos.
- Crie um arquivo em bash para atualização do site:
  ```bash
  cd ~
  touch update-website.sh
  vi update-website.sh
  ```
  Pressione `i` para entrar no modo de edição no VI e cole o código abaixo. Pressione `Esc` e digite `:wq` para salvar.
  ```bash
  #!/bin/bash
  aws s3 cp /home/ec2-user/sysops-activity-files/static-website/ s3://<my-bucket>/ --recursive --acl public-read
  ```
- Transforme o arquivo em um script executável:
  ```bash
  chmod +x update-website.sh
  ```
- Execute o script sempre que fizer uma alteração no site estático para atualizá-lo:
  ```bash
  ./update-website.sh
  ```

Espero que isso ajude! Se precisar de mais alguma coisa, estou à disposição. 😊







