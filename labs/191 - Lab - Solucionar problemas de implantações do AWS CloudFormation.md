# solucionar problemas do CloudFormation 🖥️

## Laborátório 🥼

## Objetivo

Praticar usando o JMESPath para consultar documentos em formato JSON.

Solucionar problemas de implantação de uma pilha do AWS CloudFormation usando a AWS CLI.

Analisar arquivos de log em uma instância do EC2 do Linux para determinar a causa de uma falha create-stack.

Solucionar problemas de uma ação delete-stack com falha.

### Diagrama do fluxo ✅

![architecture](https://github.com/user-attachments/assets/ffc59107-0b74-404d-834f-26027b5dc2d8)


# Execução 🚀

### Tutorial Resumido

### Relevância do Caso de Negócio
Sofia está discutindo com Olivia a implantação da AWS para a cafeteria Café. Martha e Frank gostariam que Sofia e Nikhil tivessem habilidades para desenvolver infraestrutura como código (IaC). Olivia sugere que Sofia trabalhe em uma prova de conceito (POC).

### Objetivo
Você, no papel de Sofia, vai criar e executar a implantação de um servidor web dentro de uma VPC personalizada usando um modelo do AWS CloudFormation. Você também aprenderá a solucionar problemas de implantações.

### Etapas da Atividade

#### Como Iniciar o Ambiente de Atividade
1. Clique em **Start Lab**.
2. Aguarde até que o status do laboratório seja "ready" e feche o painel.

#### Tarefa 1: Pratique a Consulta de Dados no Formato JSON usando o JMESPath
1. Acesse [jmespath.org](http://jmespath.org).
2. Substitua o documento JSON pela estrutura abaixo:
   ```json
   {
     "desserts": [
       {"name": "Chocolate cake", "price": "20.00"},
       {"name": "Ice cream", "price": "15.00"},
       {"name": "Carrot cake", "price": "22.00"}
     ]
   }
   ```
3. Na caixa de pesquisa, insira `desserts`.
4. Consulte o segundo elemento: `desserts[1]`.
5. Recupere o nome do primeiro elemento: `desserts[0].name`.
6. Recupere nome e preço do primeiro elemento: `desserts[0].[name,price]`.
7. Retorne nomes de todos os elementos: `desserts[].name`.
8. Utilize filtro para retornar atributos de "Carrot cake": `desserts[?name=='Carrot cake']`.

#### Tarefa 2: Solucionar Problemas e Trabalhar com Pilhas do AWS CloudFormation
1. Estabeleça uma conexão SSH com a instância EC2 "CLI Host" na sub-rede pública de "VPC2".

#### Tarefa 2.1 para Windows: SSH para Instância CLI Host
1. Baixe o arquivo `labsuser.ppk` do painel de detalhes.
2. Anote o endereço `PublicIP`.
3. Baixe e abra o PuTTY.
4. Configure a sessão seguindo as instruções para conectar à instância Linux usando PuTTY.

   ### Tarefa 2.1 para macOS/Linux: SSH para instância de host da CLI
Estas instruções são somente para usuários do Mac/Linux. Se você for usuário do Windows, avance para a próxima tarefa.

1. **Abrir Credenciais**:
   - Clique no menu suspenso **Detalhes** acima das instruções e selecione **Mostrar**.
   - Clique no botão **Download PEM** e salve o arquivo `labsuser.pem`.
   - Clique no **X** para fechar o painel de Credenciais.

2. **Configurar Terminal**:
   - Abra uma janela do terminal.
   - Navegue até o diretório onde o arquivo `labsuser.pem` foi salvo:
     ```sh
     cd ~/Downloads
     ```

3. **Alterar Permissões**:
   - Ajuste as permissões do arquivo `labsuser.pem` para leitura apenas:
     ```sh
     chmod 400 labsuser.pem
     ```

4. **Estabelecer Conexão SSH**:
   - No painel **Detalhes**, copie o valor do `CliHostIP`.
   - Execute o comando SSH, substituindo `<public-ip>` pelo endereço IP real:
     ```sh
     ssh -i labsuser.pem ec2-user@<public-ip>
     ```
   - Digite `yes` quando solicitado para permitir a primeira conexão ao servidor SSH remoto.

### Tarefa 2.2: Configurar a AWS CLI
1. **Descobrir Região**:
   - Execute o comando para descobrir a região da instância:
     ```sh
     curl http://169.254.169.254/latest/dynamic/instance-identity/document | grep region
     ```

2. **Configurar AWS CLI**:
   - Atualize a configuração da AWS CLI com suas credenciais:
     ```sh
     aws configure
     ```
   - Nos prompts, insira as seguintes informações:
     - **ID da chave de acesso da AWS**: Copie o valor de `AccessKey` do painel de Credenciais.
     - **Chave de acesso secreta da AWS**: Copie o valor de `SecretKey` do painel de Credenciais.
     - **Nome da região padrão**: Insira o nome da região obtida no passo anterior (ex.: `us-east-1`).
     - **Formato de saída padrão**: Digite `json`.

### Tarefa 2.3: Tentar Criar uma Pilha do AWS CloudFormation
1. **Visualizar Modelo**:
   - Observe o conteúdo do modelo `template1.yaml`:
     ```sh
     less template1.yaml
     ```
   - Pressione `q` para sair do utilitário `less`.

2. **Criar a Pilha**:
   - Execute o comando para criar a pilha:
     ```sh
     aws cloudformation create-stack \
     --stack-name myStack \
     --template-body file://template1.yaml \
     --capabilities CAPABILITY_NAMED_IAM \
     --parameters ParameterKey=KeyName,ParameterValue=vockey
     ```

3. **Monitorar Criação**:
   - Verifique o status de cada recurso criado pela pilha:
     ```sh
     watch -n 5 -d \
     aws cloudformation describe-stack-resources \
     --stack-name myStack \
     --query 'StackResources[*].[ResourceType,ResourceStatus]' \
     --output table
     ```

4. **Solucionar Problemas**:
   - Se os recursos começarem a ser excluídos, pressione `CTRL+C` para sair do utilitário de observação.
   - Execute o comando para ver o status e detalhes da pilha:
     ```sh
     watch -n 5 -d \
     aws cloudformation describe-stacks \
     --stack-name myStack \
     --output table
     ```

5. **Analisar Problemas**:
   - Verifique os eventos de falha de criação:
     ```sh
     aws cloudformation describe-stack-events \
     --stack-name myStack \
     --query "StackEvents[?ResourceStatus == 'CREATE_FAILED']"
     ```

6. **Excluir Pilha**:
   - Exclua a pilha com o comando:
     ```sh
     aws cloudformation delete-stack --stack-name myStack
     ```

### Tarefa 2.4: Evitar a Reversão em uma Pilha do AWS CloudFormation

1. **Criar a Pilha sem Reversão**:
   - Execute o comando `create-stack` novamente, especificando `--on-failure DO_NOTHING`:
     ```sh
     aws cloudformation create-stack \
     --stack-name myStack \
     --template-body file://template1.yaml \
     --capabilities CAPABILITY_NAMED_IAM \
     --on-failure DO_NOTHING \
     --parameters ParameterKey=KeyName,ParameterValue=vockey
     ```

2. **Monitorar a Criação**:
   - Execute o comando `describe-stack-resources` para monitorar os recursos:
     ```sh
     watch -n 5 -d \
     aws cloudformation describe-stack-resources \
     --stack-name myStack \
     --query 'StackResources[*].[ResourceType,ResourceStatus]' \
     --output table
     ```

3. **Verificar o Status da Pilha**:
   - Após a falha da `WaitCondition`, outros recursos devem manter o status `CREATE_COMPLETE`. Pressione `CTRL+C` para sair do utilitário `watch`.
   - Execute o comando `describe-stacks`:
     ```sh
     aws cloudformation describe-stacks \
     --stack-name myStack \
     --output table
     ```

4. **Analisar Detalhes do Evento de Falha**:
   - Verifique os eventos de falha de criação:
     ```sh
     aws cloudformation describe-stack-events \
     --stack-name myStack \
     --query "StackEvents[?ResourceStatus == 'CREATE_FAILED']"
     ```

5. **Conectar à Instância do EC2**:
   - Obtenha o endereço IP público da instância do Web Server:
     ```sh
     aws ec2 describe-instances \
     --filters "Name=tag:Name,Values='Web Server'" \
     --query 'Reservations[].Instances[].[State.Name,PublicIpAddress]'
     ```
   - Conecte-se à instância do Web Server via SSH usando o endereço IP obtido.

6. **Analisar Logs do Usuário**:
   - Visualize as últimas 50 linhas do log `cloud-init`:
     ```sh
     tail -50 /var/log/cloud-init-output.log
     ```
   - Observe os erros, como "No package http available".

7. **Revisar e Corrigir o Script UserData**:
   - Visualize o script `part-001`:
     ```sh
     sudo cat /var/lib/cloud/instance/scripts/part-001
     ```
   - Identifique que a linha `yum install -y http` deve ser alterada para `httpd`.

8. **Corrigir o Modelo**:
   - No CLI Host, atualize o modelo para corrigir o erro:
     ```sh
     vim template1.yaml
     ```
   - Substitua `http` por `httpd` na linha correspondente.
   - Confirme a alteração:
     ```sh
     cat template1.yaml | grep httpd
     ```

9. **Excluir a Pilha com Falha**:
   - Exclua a pilha:
     ```sh
     aws cloudformation delete-stack --stack-name myStack
     ```
   - Use `describe-stacks` para verificar o status da exclusão:
     ```sh
     watch -n 5 -d \
     aws cloudformation describe-stacks \
     --stack-name myStack \
     --output table
     ```

### Tarefa 2.5: Corrigir e Criar com Êxito a Pilha do AWS CloudFormation

1. **Recriar a Pilha**:
   - Execute o comando `create-stack` novamente:
     ```sh
     aws cloudformation create-stack \
     --stack-name myStack \
     --template-body file://template1.yaml \
     --capabilities CAPABILITY_NAMED_IAM \
     --on-failure DO_NOTHING \
     --parameters ParameterKey=KeyName,ParameterValue=vockey
     ```

2. **Monitorar a Criação**:
   - Use `describe-stack-resources` para verificar a criação:
     ```sh
     watch -n 5 -d \
     aws cloudformation describe-stack-resources \
     --stack-name myStack \
     --query 'StackResources[*].[ResourceType,ResourceStatus]' \
     --output table
     ```

3. **Verificar o Status da Pilha**:
   - A pilha deve ter o status `CREATE_COMPLETE`.
   - Verifique a saída para o endereço IP do servidor web e o nome do bucket do S3:
     ```sh
     aws cloudformation describe-stacks \
     --stack-name myStack \
     --output table
     ```

4. **Testar o Servidor Web**:
   - Copie o endereço IP público do comando anterior.
   - Abra uma guia do navegador e insira o endereço IP.
   - A mensagem "Hello from your web server!" deve ser exibida.
### Tarefa 3: Fazer Modificações Manuais e Detectar Desvios

#### Tarefa 3.1: Fazer Modificações Manuais nos Grupos de Segurança

1. **Acessar o Console de Gerenciamento da AWS**:
   - Clique em **AWS** no topo das instruções para abrir o console em uma nova guia.

2. **Modificar Regras de Segurança**:
   - No menu **Serviços**, selecione **EC2**.
   - Clique em **Instâncias** e selecione **Web Server**.
   - Vá para a guia **Segurança** e selecione o grupo de segurança **WebServerSG**.
   - Na guia **Regras de entrada**, clique em **Editar regras de entrada**.
   - Modifique a regra de entrada SSH (porta 22) para usar **Meu IP** na coluna **Origem**.
   - Clique em **Salvar regras**.

#### Tarefa 3.2: Adicionar um Objeto ao Bucket do S3

1. **Obter Nome do Bucket**:
   - No terminal conectado ao CLI Host, obtenha o nome do bucket e salve em uma variável:
     ```sh
     bucketName=$(aws cloudformation describe-stacks --stack-name myStack --query "Stacks[*].Outputs[?OutputKey == 'BucketName'].[OutputValue]" --output text)
     echo "bucketName = "$bucketName
     ```

2. **Adicionar Arquivo ao Bucket**:
   - Crie um arquivo vazio:
     ```sh
     touch myfile
     ```
   - Copie o arquivo para o bucket:
     ```sh
     aws s3 cp myfile s3://$bucketName/
     ```

3. **Verificar Arquivo no Bucket**:
   - Verifique se o arquivo foi adicionado:
     ```sh
     aws s3 ls $bucketName/
     ```

#### Tarefa 3.3: Detectar Desvios

1. **Iniciar Detecção de Desvio**:
   - Execute o comando para detectar desvios:
     ```sh
     aws cloudformation detect-stack-drift --stack-name myStack
     ```
   - O comando deve retornar um `StackDriftDetectionId`.

2. **Monitorar Status de Desvio**:
   - Substitua `<driftId>` pelo valor de `StackDriftDetectionId`:
     ```sh
     aws cloudformation describe-stack-drift-detection-status --stack-drift-detection-id <driftId>
     ```

3. **Descrever Recursos Desviados**:
   - Execute o comando para descrever os recursos desviados:
     ```sh
     aws cloudformation describe-stack-resource-drifts --stack-name myStack
     ```

4. **Resumir Status dos Recursos**:
   - Execute o comando para visualizar os recursos em uma tabela:
     ```sh
     aws cloudformation describe-stack-resources --stack-name myStack --query 'StackResources[*].[ResourceType,ResourceStatus,DriftInformation.StackResourceDriftStatus]' --output table
     ```

5. **Detalhes dos Recursos Desviados**:
   - Obtenha os detalhes específicos dos recursos com status `MODIFIED`:
     ```sh
     aws cloudformation describe-stack-resource-drifts --stack-name myStack --stack-resource-drift-status-filters MODIFIED
     ```

6. **Tentar Atualizar a Pilha**:
   - Execute o comando para tentar atualizar a pilha:
     ```sh
     aws cloudformation update-stack --stack-name myStack --template-body file://template1.yaml --parameters ParameterKey=KeyName,ParameterValue=vockey
     ```
   - A saída deve indicar um erro, pois o desvio deve ser resolvido manualmente.

#### Tarefa 4: Tentar Excluir a Pilha

1. **Excluir a Pilha**:
   - Execute o comando para excluir a pilha:
     ```sh
     aws cloudformation delete-stack --stack-name myStack
     ```

2. **Monitorar a Exclusão**:
   - Execute o comando para observar o status dos recursos:
     ```sh
     watch -n 5 -d aws cloudformation describe-stack-resources --stack-name myStack --query 'StackResources[*].[ResourceType,ResourceStatus]' --output table
     ```

3. **Verificar Falha na Exclusão**:
   - Execute o comando para verificar o status da pilha:
     ```sh
     aws cloudformation describe-stacks --stack-name myStack --output table
     ```
   - O status deve indicar `DELETE_FAILED` devido ao bucket S3 contendo objetos.

#### Desafio: Manter o Arquivo e Excluir a Pilha

1. **Resolver o Desafio**:
   - Descubra como manter o bucket e o arquivo, e ainda assim excluir a pilha com sucesso:
     ```sh
     aws cloudformation delete-stack --stack-name myStack --retain-resources <LogicalResourceId>
     ```

Lembre-se, a chave é usar a AWS CLI para evitar perder dados no bucket do S3 enquanto ainda exclui a pilha. 🚀

**Atualização da cafeteria Café**:
No dia seguinte, Sofia explicou à equipe do Café como a infraestrutura como código com CloudFormation ajudou a melhorar a gestão de recursos AWS, possibilitando a criação de ambientes de desenvolvimento e produção separados, mas correspondentes. Essa abordagem trouxe benefícios como detecção de desvios e reconstrução rápida de infraestruturas complexas na nuvem.
