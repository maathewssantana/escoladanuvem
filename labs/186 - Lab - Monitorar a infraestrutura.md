# Monitorar a infraestrutura 🖥️

## Laborátório 🥼

## Objetivo

- Usar o AWS Systems Manager Run Command para instalar o agente do CloudWatch nas instâncias do Amazon Elastic Compute Cloud (Amazon EC2)

- Monitorar logs de aplicações usando o CloudWatch Agent e o CloudWatch Logs

- Monitorar métricas do sistema usando o agente do CloudWatch e as métricas do CloudWatch

- Criar notificações em tempo real usando o CloudWatch Events

- Acompanhar a conformidade da infraestrutura usando o AWS Config

### Diagrama do fluxo ✅

![install-agent](https://github.com/user-attachments/assets/ce967b14-9be7-4a22-9014-e2af7166cd73)


# Execução 🚀


### Tutorial Simplificado: Instalação e Configuração do CloudWatch Agent

#### Passo 1: Instalar o Agente do CloudWatch

1. No Console de Gerenciamento da AWS, vá para **Services** e selecione **Systems Manager**.
2. No painel de navegação à esquerda, selecione **Run Command**. Caso não veja o painel, clique no ícone  no canto superior esquerdo.
3. Selecione **Run a command**.
4. Escolha **AWS-ConfigureAWSPackage**.
5. Em **Command Parameters**, configure:
   - **Action**: Instalar
   - **Name**: AmazonCloudWatchAgent
   - **Version**: latest
6. Em **Targets**, selecione **Choose instances manually** e marque a caixa ao lado de **Web Server**.
7. Na parte inferior, clique em **Run**.
8. Aguarde até que o **Overall status** mude para **Success**. Atualize a página se necessário.
9. Para verificar a saída, clique ao lado da instância e selecione **View output**. Você deve ver a mensagem indicando que o AmazonCloudWatchAgent foi instalado com êxito.

#### Passo 2: Configurar o Agente do CloudWatch

1. No painel de navegação à esquerda, selecione **Parameter Store**.
2. Selecione **Create parameter** e configure:
   - **Name**: Monitor-Web-Server
   - **Description**: Collect web logs and system metrics
   - **Value**: Copie e cole a seguinte configuração:
     ```json
     {
       "logs": {
         "logs_collected": {
           "files": {
             "collect_list": [
               {
                 "log_group_name": "HttpAccessLog",
                 "file_path": "/var/log/httpd/access_log",
                 "log_stream_name": "{instance_id}",
                 "timestamp_format": "%b %d %H:%M:%S"
               },
               {
                 "log_group_name": "HttpErrorLog",
                 "file_path": "/var/log/httpd/error_log",
                 "log_stream_name": "{instance_id}",
                 "timestamp_format": "%b %d %H:%M:%S"
               }
             ]
           }
         }
       },
       "metrics": {
         "metrics_collected": {
           "cpu": {
             "measurement": [
               "cpu_usage_idle",
               "cpu_usage_iowait",
               "cpu_usage_user",
               "cpu_usage_system"
             ],
             "metrics_collection_interval": 10,
             "totalcpu": false
           },
           "disk": {
             "measurement": [
               "used_percent",
               "inodes_free"
             ],
             "metrics_collection_interval": 10,
             "resources": ["*"]
           },
           "diskio": {
             "measurement": ["io_time"],
             "metrics_collection_interval": 10,
             "resources": ["*"]
           },
           "mem": {
             "measurement": ["mem_used_percent"],
             "metrics_collection_interval": 10
           },
           "swap": {
             "measurement": ["swap_used_percent"],
             "metrics_collection_interval": 10
           }
         }
       }
     }
     ```
3. Clique em **Create parameter**.

#### Passo 3: Iniciar o Agente do CloudWatch

1. No painel de navegação à esquerda, selecione **Run Command** novamente.
2. Clique em **Run a command**.
3. Filtro por **Document name prefix** e selecione **AmazonCloudWatch-ManageAgent**.
4. Verifique se o filtro está configurado corretamente e pressione **Enter**.
5. Escolha **AmazonCloudWatch-ManageAgent** (confirme o nome).
6. Na seção **Command parameters**, configure:
   - **Action**: configure
   - **Mode**: ec2
   - **Optional configuration source**: ssm
   - **Optional configuration location**: Monitor-Web-Server
   - **Optional restart**: yes
7. Em **Targets**, selecione **Choose instances manually** e marque a caixa ao lado de **Web Server**.
8. Clique em **Run**.
9. Aguarde até que o **Overall status** mude para **Success**. Atualize a página se necessário.

O agente do CloudWatch agora está em execução na instância, coletando dados de log e métricas para o CloudWatch.

Claro, posso ajudar a simplificar essa tarefa para você. Vamos lá!

### Tarefa 2: Monitorar Logs da Aplicação usando o CloudWatch Logs

#### Passo 1: Acessar o Servidor Web e Gerar Dados de Log

1. No Console de Gerenciamento da AWS, acesse a seção **Systems Manager**.
2. Copie o valor **WebServerIP**.
3. Abra uma nova guia do navegador, cole o **WebServerIP** e pressione **Enter** para acessar a página de teste do servidor web.
4. Insira **/start** no URL do navegador e pressione **Enter** para gerar um erro 404 nos logs de acesso.

#### Passo 2: Visualizar Logs no CloudWatch

1. No Console de Gerenciamento da AWS, selecione **CloudWatch** no menu de serviços.
2. No painel de navegação à esquerda, selecione **Log groups**.
3. Verifique se os logs **HttpAccessLog** e **HttpErrorLog** estão listados. Se não, espere um minuto e selecione **Refresh**.
4. Selecione **HttpAccessLog** e depois o fluxo de logs correspondente à sua instância EC2.

#### Passo 3: Criar um Filtro de Métrica para Erros 404

1. No painel de navegação à esquerda, selecione **Log groups**.
2. Marque a caixa ao lado de **HttpAccessLog**.
3. No menu suspenso **Actions**, selecione **Create metric filter**.
4. Cole a seguinte linha na caixa **Filter pattern**:
    ```plaintext
    [ip, id, user, timestamp, request, status_code=404, size]
    ```
5. Em **Test pattern**, selecione o ID da instância EC2 no menu suspenso e clique em **Test pattern**.
6. Verifique se há resultados com `$status_code` 404 e selecione **Next**.
7. Em **Filter name**, insira **404Errors**.
8. Em **Metric details**, configure:
   - **Metric Namespace**: LogMetrics
   - **Metric Name**: 404Errors
   - **Metric Value**: 1
9. Selecione **Next** e depois **Create metric filter**.

Claro, vamos simplificar isso para você!

### Criar um Alarme Usando o Filtro no CloudWatch Logs

#### Passo 1: Configurar o Alarme

1. No Console de Gerenciamento da AWS, acesse **CloudWatch** e selecione **Log groups**.
2. Marque a caixa de seleção ao lado de **HttpAccessLog**.
3. No menu suspenso **Actions**, selecione **Create metric filter**.
4. Defina o padrão de filtro como:
    ```plaintext
    [ip, id, user, timestamp, request, status_code=404, size]
    ```
5. Teste o padrão e verifique os resultados com **$status_code** 404.
6. Prossiga para criar o filtro e configure:
   - **Filter name**: 404Errors
   - **Metric details**: 
     - **Namespace**: LogMetrics
     - **Name**: 404Errors
     - **Value**: 1
7. Após criar o filtro, selecione **Create alarm**.

#### Passo 2: Definir as Configurações do Alarme

1. Na seção **Metrics**, defina **Period** para 1 minuto.
2. Na seção **Conditions**, configure:
   - **Whenever 404Errors is**: Greater than or equal to
   - **Threshold**: 5
3. Prossiga para a seção **Notification** e configure:
   - **Select an SNS topic**: Create new topic
   - **Email endpoints**: Insira um endereço de e-mail acessível.
4. Crie o tópico e avance para a próxima seção.
5. Em **Name and description**, defina:
   - **Alarm name**: 404 Errors
   - **Alarm description**: Alert when too many 404s detected on an instance
6. Prossiga e finalize criando o alarme.

#### Passo 3: Confirmar e Testar o Alarme

1. Verifique seu e-mail e confirme a assinatura clicando no link de confirmação.
2. Volte ao Console de Gerenciamento da AWS e vá para **CloudWatch**.
3. O alarme deve aparecer em laranja com status **Insufficient data**.
4. Gere dados de log acessando páginas inexistentes no servidor web (repita pelo menos cinco vezes com URLs como `http://<WebServerIP>/start2`).
5. Aguarde de 1 a 2 minutos para que o alarme seja acionado. O status mudará para **Alarm**.
6. Verifique seu e-mail para a notificação do alarme.

Agora, você configurou um alarme que notificará quando muitos erros 404 forem registrados.

### Tarefa 3: Monitorar Métricas de Instância Usando o CloudWatch

#### Passo 1: Examinar Métricas Básicas

1. No Console de Gerenciamento da AWS, vá para **Elastic Compute Cloud**.
2. No painel de navegação à esquerda, selecione **Instances**.
3. Marque a caixa de seleção ao lado de **Web Server**.
4. Na guia **Monitoring**, examine as métricas de CPU, disco e rede.

#### Passo 2: Examinar Métricas Coletadas pelo CloudWatch Agent

1. No Console de Gerenciamento da AWS, vá para **CloudWatch**.
2. No painel de navegação à esquerda, selecione **Metrics** e depois **All Metrics**.
3. Selecione **CWAgent** e explore as diferentes métricas (por exemplo, **device**, **fstype**, **host**, **path**).
4. Examine métricas de espaço em disco e memória do sistema.

Você agora tem uma visão completa das métricas coletadas pelo CloudWatch, tanto externas quanto internas da instância.

Claro, vamos simplificar essa tarefa para você também!

### Tarefa 4: Criar Notificações em Tempo Real com CloudWatch Events

#### Passo 1: Configurar a Regra de Eventos

1. No Console de Gerenciamento da AWS, expanda **Events** no painel de navegação à esquerda e selecione **Rules**.
2. Clique em **Create rule**.
3. Na seção **Event Source**, configure:
   - **Service Name**: Elastic Compute Cloud
   - **Event Type**: EC2 Instance State-change Notification
   - Marque **Specific state(s)** e selecione **stopped** e **terminated**.
4. Na seção **Targets**, clique em **Add target**.
   - **Target type**: SNS topic
   - **Topic**: padrão_CloudWatch_Alarmes_Tópico
5. Na parte inferior, clique em **Configure details**.

#### Passo 2: Definir os Detalhes da Regra

1. Na seção **Rule definition**, configure:
   - **Name**: Instance_Stopped_Terminated
2. Clique em **Create rule**.

#### Passo 3: Configurar Notificação por E-mail

1. No menu **Services**, selecione **Simple Notification Service**.
2. No painel de navegação à esquerda, selecione **Topics**.
3. Clique no link na coluna **Name** para o tópico criado na Tarefa 2.
4. Verifique se há uma assinatura associada ao seu e-mail.

#### Passo 4: Testar a Notificação

1. No menu **Services**, selecione **Elastic Compute Cloud**.
2. No painel de navegação à esquerda, selecione **Instances**.
3. Marque a caixa de seleção ao lado de **Web Server**.
4. Selecione **Instance State**, depois **Stop Instance** e confirme.
5. Aguarde até que a instância esteja no estado **Stopped**.
6. Verifique seu e-mail para a notificação.

### Tarefa 5: Monitorar a Conformidade da Infraestrutura com AWS Config

#### Passo 1: Configurar AWS Config

1. No menu **Services**, selecione **Config**.
2. Se aparecer o botão **Get Started**, clique nele e prossiga com as configurações padrão até confirmar.

#### Passo 2: Adicionar Regras de Conformidade

1. No painel de navegação à esquerda, selecione **Rules** e clique em **Add rule**.
2. Pesquise por **required-tags** e selecione a regra.
3. Na configuração da regra, em **tag1Key**, insira `project`.
4. Clique em **Next** e depois **Add rule**.

#### Passo 3: Adicionar Regra para Volumes EBS

1. Clique novamente em **Add rule**.
2. Pesquise por **ec2-volume-inuse-check** e selecione a regra.
3. Prossiga com as configurações padrão e clique em **Next**, depois **Add rule**.

#### Passo 4: Verificar a Conformidade

1. Aguarde alguns minutos para que as regras sejam avaliadas.
2. Atualize a página do navegador se necessário.
3. Selecione cada regra para visualizar os resultados de conformidade.

