# Trabalhar com o AWS CloudTrail 🖥️

## Laborátório 🥼

## Objetivo

Configurar uma trilha do CloudTrail.

Analisar os logs do CloudTrail usando vários métodos para descobrir informações relevantes.

Importar dados de log do CloudTrail para o Athena.

Executar consultas no Athena para filtrar registros de log do CloudTrail.

Resolver problemas de segurança na conta da AWS e em uma instância do EC2 Linux.

### Diagrama do fluxo ✅

![architecturetrail](https://github.com/user-attachments/assets/ab8b2b57-d161-4912-b2ab-dc95b84b3ae1)


# Execução 🚀

#### Relevância do Caso de Negócio
Martha e Frank, da cafeteria Café, perceberam que o site foi invadido e precisam descobrir quem fez isso, além de evitar que aconteça novamente. Frequentemente, eles fazem alterações no site, o que pode causar problemas.

Você assumirá o papel de Sofia para investigar e resolver a situação.

#### Etapas da Atividade

##### Como Iniciar o Ambiente de Atividade
1. Selecione **Start Lab** para iniciar o laboratório.
2. Aguarde até que a mensagem **Lab status: ready** seja exibida.
3. Feche o painel **Start Lab**.
4. Selecione **AWS** para abrir o Console de Gerenciamento da AWS em uma nova guia do navegador.

##### Tarefa 1: Modificar um Grupo de Segurança e Observar o Site
1. No menu **Services**, escolha **EC2**.
2. Selecione **Instances** e encontre **Café Web Server**.
3. Na guia **Security**, selecione o grupo de segurança sg-xxxxxxxxxx.
4. Em **Inbound rules**, adicione a regra:
   - **Tipo**: SSH
   - **Port Range**: 22
   - **Source**: My IP
5. Salve as regras.

##### Verificar o Site da Cafeteria
1. Em **Instances**, selecione **Café Web Server**.
2. Copie o **Public IPv4 address**.
3. Abra uma nova guia do navegador e navegue até `http://<WebServerIP>/cafe/`.

##### Tarefa 2: Criar um Log do CloudTrail e Observar o Site Invadido

###### Tarefa 2.1: Criar um Log do CloudTrail
1. No menu **Services**, selecione **CloudTrail**.
2. No painel de navegação, escolha **Trails**.
3. Selecione **Create trail** e configure:
   - **Trail name**: monitor
   - **Create a new S3 bucket**: monitoring####
   - **AWS KMS alias**: suas iniciais-KMS
4. Conclua a criação da trilha.

###### Tarefa 2.2: Observar o Site Invadido
1. Volte para o navegador com o site da cafeteria e atualize a página.
2. Observe que o site foi invadido.
3. No Console de Gerenciamento da AWS, navegue até **EC2** e examine a instância **Café Web Server**.
4. Na guia **Security**, observe as **Inbound rules**. Verifique a regra que permite SSH de qualquer lugar (0.0.0.0/0).

##### Investigar nos Logs do CloudTrail
1. Use o **CloudTrail** para descobrir quem adicionou a regra de segurança extra e comprometeu o site.

### Tarefa 3: Analisar os Logs do CloudTrail usando `grep`

#### Tarefa 3.1: Conectar-se à Instância do EC2 Café Web Server usando SSH
**Usuários do Windows** devem seguir as instruções para Windows. **Usuários do macOS e Linux** devem seguir as instruções para macOS/Linux.

##### Tarefa 3.2 para Windows: Conectar-se via SSH

1. No menu suspenso **Detalhes** acima destas instruções, selecione **Mostrar**.
2. Na janela de Credenciais, clique em **Download PPK** e salve o arquivo `labsuser.ppk` (normalmente no diretório Downloads).
3. Anote o endereço do **PublicIP**.
4. Saia do painel Detalhes clicando no X.
5. Baixe e instale o PuTTY [aqui](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) se ainda não o tiver.
6. Abra o `putty.exe` e configure sua sessão seguindo as instruções do [link de suporte](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html).

##### Tarefa 3.2 para macOS/Linux: Conectar-se via SSH

1. No menu suspenso **Detalhes** acima destas instruções, selecione **Mostrar**.
2. Na janela de Credenciais, clique em **Download PEM** e salve o arquivo `labsuser.pem`.
3. Saia do painel Detalhes clicando no X.
4. Abra uma janela do terminal e altere para o diretório onde o arquivo `labsuser.pem` foi salvo:
   ```bash
   cd ~/Downloads
   ```
5. Altere as permissões da chave:
   ```bash
   chmod 400 labsuser.pem
   ```
6. No Console de Gerenciamento da AWS, selecione **EC2**, marque a instância **Café Web Server** e copie o **Public IPv4 address**.
7. No terminal, conecte-se à instância EC2:
   ```bash
   ssh -i labsuser.pem ec2-user@<public-ip>
   ```
   Substitua `<public-ip>` pelo endereço IP público real que você copiou. Digite `yes` quando solicitado para permitir a conexão.

#### Tarefa 3.3: Baixar e Extrair os Logs do CloudTrail

1. Verifique se o terminal está conectado via SSH à instância do EC2 Café Web Server.
2. Crie um diretório local no servidor web para baixar os arquivos de log do CloudTrail:
   ```bash
   mkdir ctraillogs
   cd ctraillogs
   ```
3. Liste os buckets S3 e recupere o nome do bucket:
   ```bash
   aws s3 ls
   ```
4. Baixe os logs do CloudTrail. Substitua `<monitoring####>` pelo nome real do bucket:
   ```bash
   aws s3 cp s3://<monitoring####>/ . --recursive
   ```
   Se não houver saída, espere alguns minutos e tente novamente.
5. Navegue para o subdiretório onde os logs foram baixados:
   ```bash
   cd AWSLogs/<account-num>/CloudTrail/<Region>/<yyyy>/<mm>/<dd>
   ```
6. Extraia os logs:
   ```bash
   gunzip *.gz
   ```
7. Liste os arquivos extraídos:
   ```bash
   ls
   ```

### Tarefa 3.4: Analisar os Logs do CloudTrail usando `grep`

#### Passo 1: Analisar a Estrutura dos Logs

1. **Copie um dos nomes de arquivo** retornados pelo comando `ls` que você executou.
2. **Visualize o conteúdo do arquivo**:
   ```bash
   cat <filename.json>
   ```
   Substitua `<filename.json>` pelo nome real do arquivo copiado.

   ![example-log-entry](https://github.com/user-attachments/assets/d20f347e-7234-4b16-8ed8-5a6100196ac9)


4. **Formate a saída para facilitar a leitura**:
   ```bash
   cat <filename.json> | python -m json.tool
   ```

#### Passo 2: Filtrar Resultados

1. **Defina o endereço WebServerIP como uma variável** (substitua `<WebServerIP>` pelo endereço IP real):
   ```bash
   ip=<WebServerIP>
   ```

2. **Filtre logs pelo endereço IP de origem**:
   ```bash
   for i in $(ls); do echo $i && cat $i | python -m json.tool | grep sourceIPAddress ; done
   ```

3. **Filtre logs pelo nome do evento**:
   ```bash
   for i in $(ls); do echo $i && cat $i | python -m json.tool | grep eventName ; done
   ```

#### Tarefa 3.5: Analisar os Logs usando Comandos CloudTrail da AWS CLI

1. **Pesquise eventos de login no console**:
   ```bash
   aws cloudtrail lookup-events --lookup-attributes AttributeKey=EventName,AttributeValue=ConsoleLogin
   ```

2. **Encontre ações nos grupos de segurança**:
   ```bash
   aws cloudtrail lookup-events --lookup-attributes AttributeKey=ResourceType,AttributeValue=AWS::EC2::SecurityGroup --output text
   ```

3. **Obtenha o ID do grupo de segurança usado pela instância Café Web Server**:
   ```bash
   region=$(curl http://169.254.169.254/latest/dynamic/instance-identity/document | grep region | cut -d '"' -f4)
   sgId=$(aws ec2 describe-instances --filters "Name=tag:Name,Values='Cafe Web Server'" --query 'Reservations[*].Instances[*].SecurityGroups[*].[GroupId]' --region $region --output text)
   echo $sgId
   ```

4. **Filtre resultados pelo ID do grupo de segurança**:
   ```bash
   aws cloudtrail lookup-events --lookup-attributes AttributeKey=ResourceType,AttributeValue=AWS::EC2::SecurityGroup --region $region --output text | grep $sgId
   ```
### Tarefa 4: Analisar os Logs do CloudTrail usando Athena

#### Visão Geral
Utilizar o Athena facilita a análise dos logs do CloudTrail, permitindo que você execute consultas SQL diretamente nos dados armazenados no Amazon S3.

#### Tarefa 4.1: Criar a Tabela no Athena

1. **Acessar o Console do CloudTrail:**
   - No Console de Gerenciamento da AWS, vá para **CloudTrail**.
   - No painel de navegação, selecione **Event history**.

2. **Criar a Tabela do Athena:**
   - Na página **Event history**, clique em **Create Athena table**.
   - Selecione o bucket do S3 `monitoring####` que você configurou anteriormente.
   - Analise a declaração `CREATE TABLE` gerada.
   - Confirme e clique em **Create table**.
   - A tabela será criada com um nome padrão que inclui o nome do bucket S3.

3. **Ir para o Athena:**
   - No menu **Services**, selecione **Analytics** e escolha o serviço **Athena**.

#### Tarefa 4.2: Analisar Logs usando o Athena

1. **Configurar o Editor de Consultas do Athena:**
   - Se necessário, selecione **Explore query editor** para abrir o editor.
   - No painel esquerdo, expanda a tabela `cloudtrail_logs_monitoring####` para ver os nomes das colunas.

2. **Configurar a Localização dos Resultados da Consulta:**
   - Na barra de menus no canto superior direito, selecione **Settings** e clique em **Manage**.
   - Defina **Location of query result** para `s3://monitoring####/results/`.
   - Salve as configurações.

3. **Executar uma Consulta Simples:**
   - No painel **Editor**, cole a consulta SQL a seguir, substituindo `####` pelos números da sua tabela real:
     ```sql
     SELECT *
     FROM cloudtrail_logs_monitoring####
     LIMIT 5
     ```
   - Clique em **Run** para executar a consulta e visualizar os resultados.

4. **Refinar a Consulta para Dados Específicos:**
   - Execute uma nova consulta para retornar apenas as colunas relevantes:
     ```sql
     SELECT useridentity.userName, eventtime, eventsource, eventname, requestparameters
     FROM cloudtrail_logs_monitoring####
     LIMIT 30
     ```

### Desafio: Identificar o Hacker

#### Dicas para o Desafio:

1. **Analisar Dados Retornados:**
   - Use os resultados das consultas anteriores para entender melhor os tipos de dados.

2. **Filtrar por Eventos do Amazon EC2:**
   - Use cláusulas `WHERE` para filtrar eventos específicos:
     ```sql
     SELECT *
     FROM cloudtrail_logs_monitoring####
     WHERE eventsource = 'ec2.amazonaws.com'
     ```

3. **Remover a Cláusula `LIMIT`:**
   - Para garantir que você esteja visualizando todos os logs, remova a cláusula `LIMIT`.

4. **Examinar a Coluna `eventname`:**
   - Refine a consulta para procurar eventos que contenham a palavra "Security":
     ```sql
     SELECT *
     FROM cloudtrail_logs_monitoring####
     WHERE eventname LIKE '%Security%'
     ```

5. **Ajustar a Cláusula `WHERE` para um `eventname` Específico:**
   - Caso identifique eventos suspeitos, ajuste a consulta para focar em um nome de evento específico.

6. **Consulta Geral Útil:**
   - Execute uma consulta geral para identificar ações e usuários ativos:
     ```sql
     SELECT DISTINCT useridentity.userName, eventName, eventSource
     FROM cloudtrail_logs_monitoring####
     WHERE from_iso8601_timestamp(eventtime) > date_add('day', -1, now())
     ORDER BY eventSource
     ```

Você terá concluído o desafio com sucesso se identificar:
- O nome de usuário da AWS que criou a brecha de segurança.
- A hora exata da violação.
- O endereço IP usado na violação.
- O método de execução da violação (acesso ao console ou programático).

### Tarefa 5: Analisar e Melhorar a Segurança Após a Invasão

#### Tarefa 5.1: Verificar os Usuários do Sistema Operacional

1. **Descobrir quem fez login recentemente:**
   ```bash
   sudo aureport --auth
   ```
   - Verifique se há usuários desconhecidos, como `chaos-user`.

2. **Verificar quem está conectado no momento:**
   ```bash
   who
   ```

3. **Remover o usuário conectado:**
   - Tente remover o usuário:
     ```bash
     sudo userdel -r chaos-user
     ```
   - Se falhar, interrompa o processo ativo (substitua `ProcNum` pelo número do processo):
     ```bash
     sudo kill -9 ProcNum
     ```

4. **Verifique novamente se o usuário foi removido:**
   ```bash
   who
   sudo userdel -r chaos-user
   ```

5. **Verificar outros usuários que podem fazer login:**
   ```bash
   sudo cat /etc/passwd | grep -v nologin
   ```

#### Tarefa 5.2: Atualizar a Segurança SSH

1. **Verificar e editar as configurações de SSH:**
   ```bash
   sudo ls -l /etc/ssh/sshd_config
   sudo vi /etc/ssh/sshd_config
   ```
   - Comente a linha `PasswordAuthentication yes` e descomente a linha `#PasswordAuthentication no`.

2. **Salvar e reiniciar o serviço SSH:**
   ```bash
   sudo service sshd restart
   ```

3. **Atualizar as regras de segurança no console EC2:**
   - Excluir a regra que permite acesso à porta 22 de `0.0.0.0/0`.

#### Tarefa 5.3: Corrigir o Site

1. **Navegar até o diretório de imagens do site:**
   ```bash
   cd /var/www/html/cafe/images/
   ls -l
   ```

2. **Restaurar o arquivo de imagem original:**
   ```bash
   sudo mv Coffee-and-Pastries.backup Coffee-and-Pastries.jpg
   ```

3. **Recarregar o site no navegador:**
   - Acesse `http://<WebServerIP>/cafe` e pressione Shift + Atualizar.

#### Tarefa 5.4: Excluir o Usuário Hacker da AWS

1. **Acessar o IAM no Console da AWS:**
   - Selecione o menu **Services** e escolha **IAM**.

2. **Excluir o usuário `chaos`:**
   - Marque a caixa ao lado do usuário `chaos` e selecione **Excluir**.

### Conclusão

Todos na cafeteria Café estão aliviados por Sofia ter descoberto a identidade do hacker e removido o acesso ao servidor web e à conta da AWS. A equipe agora entende a importância de manter o site seguro e continuará a usar o CloudTrail como ferramenta chave para auditoria de atividades na conta da AWS.
