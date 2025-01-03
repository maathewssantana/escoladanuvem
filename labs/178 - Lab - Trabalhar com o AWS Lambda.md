# Trabalhar com o AWS Lambda 🖥️

## Laborátório 🥼

## Objetivo

- Reconhecer as permissões necessárias da política do AWS Identity and Access Management (IAM) para viabilizar uma função do Lambda para outros recursos da Amazon Web Services (AWS).

- Criar uma camada do Lambda para satisfazer uma dependência de biblioteca externa.

- Criar funções do Lambda que extraiam dados do banco de dados e enviar relatórios ao usuário.

- Implantar e testar uma função do Lambda que é iniciada com base em uma programação e que invoca outra função.

- Usar o CloudWatch Logs para solucionar problemas ao executar uma função do Lambda.

### Diagrama do fluxo ✅

<img width="794" alt="lambda-activity-architecture" src="https://github.com/user-attachments/assets/fdd429be-e1f6-46e7-a326-6417d01c6cdf" />



O diagrama inclui as seguintes etapas de função:

Etapa	Detalhes
- 1	Um evento do Amazon CloudWatch Events chama a função do Lambda salesAnalysisReport às 20h todos os dias, de segunda a sábado.
- 2	A função do Lambda salesAnalysisReport invoca outra função do Lambda, salesAnalysisReportDataExtractor, para recuperar os dados do relatório.
- 3	A função salesAnalysisReportDataExtractor executa uma consulta analítica no banco de dados da cafeteria (cafe_db).
- 4	O resultado da consulta retorna para a função salesAnalysisReport.
- 5	A função salesAnalysisReport formata o relatório em uma mensagem e o publica no tópico do Amazon Simple Notification Service (Amazon SNS) salesAnalysisReportTopic.
- 6	O tópico do SNS salesAnalysisReportTopic envia a mensagem por e-mail ao administrador.

# Execução 🚀

### Tarefa 1: Observar as configurações do perfil do IAM

**Tarefa 1.1: Observar as configurações do perfil do IAM salesAnalysisReport**

1. No Console de Gerenciamento da AWS, selecione Serviços > Segurança, identidade e conformidade > IAM.
2. No painel de navegação, selecione Perfis.
3. Na caixa de pesquisa, insira `sales`.
4. Nos resultados filtrados, selecione o hiperlink `salesAnalysisReportRole`.
5. Na guia **Relações de confiança**, observe que `lambda.amazonaws.com` está listada como uma entidade confiável.
6. Na guia **Permissões**, observe as quatro políticas atribuídas a este perfil:
   - **AmazonSNSFullAccess**: Concede acesso total aos recursos do Amazon SNS.
   - **AmazonSSMReadOnlyAccess**: Concede acesso somente leitura aos recursos do Systems Manager.
   - **AWSLambdaBasicRunRole**: Concede permissões de gravação para o CloudWatch Logs.
   - **AWSLambdaRole**: Permite que uma função do Lambda invoque outra função do Lambda.

**Tarefa 1.2: Observar as configurações do perfil do IAM salesAnalysisReportDERole**

1. Selecione Perfis novamente.
2. Na caixa de pesquisa, insira `sales`.
3. Nos resultados filtrados, selecione o hiperlink `salesAnalysisReportDERole`.
4. Na guia **Relações de confiança**, observe que `lambda.amazonaws.com` está listada como uma entidade confiável.
5. Na guia **Permissões**, observe as permissões concedidas:
   - **AWSLambdaBasicRunRole**: Concede permissões de gravação para o CloudWatch Logs.
   - **AWSLambdaVPCAccessRunRole**: Concede permissões para gerenciar interfaces de rede elástica para conectar uma função a uma VPC.

### Tarefa 2: Criar uma camada do Lambda e uma função extratora de dados do Lambda

**Tarefa 2.1: Criar uma camada do Lambda**

1. Abra o Console de Gerenciamento da AWS e selecione Serviços > Computação > Lambda.
2. Selecione **Camadas**.
3. Selecione **Criar camada**.
4. Configure as seguintes definições:
   - Em **Nome**, insira `pymysqlLibrary`.
   - Em **Descrição**, insira `PyMySQL library modules`.
   - Selecione **Fazer upload de um arquivo.zip** e faça o upload do arquivo `pymysql-v3.zip`.
   - Em **Runtimes compatíveis**, selecione `Python 3.9`.
5. Selecione **Criar**.

A mensagem “Camada pymysqlLibrary versão 1 criada com êxito” será exibida.

<img width="255" alt="layer-structure" src="https://github.com/user-attachments/assets/392f0c94-29e1-498e-9e2e-175b3cd44610" />

### Tarefa 2.2: Criar a função extratora de dados do Lambda

1. **Criar a função**:
   - No painel de navegação, selecione **Funções** para abrir a página de Funções.
   - Selecione **Criar função** e configure as seguintes opções:
     - Em **Criar do zero**.
     - Em **Nome da função**, insira `salesAnalysisReportDataExtractor`.
     - Em **Tempo de execução**, selecione `Python 3.9`.
     - Expanda **Alterar a função de execução padrão**:
       - Em **Papel de execução**, selecione **Usar uma função existente**.
       - Em **Função existente**, selecione `salesAnalysisReportDERole`.
   - Selecione **Criar função**.
   - Uma nova página é aberta com a mensagem: A função `salesAnalysisReportDataExtractor` foi criada com êxito.

### Tarefa 2.3: Adicionar a camada do Lambda à função

1. No painel **Visão geral da função**, selecione **Camadas**.
2. Na parte inferior da página, no painel **Camadas**, selecione **Adicionar uma camada**.
3. Na página **Adicionar camada**, configure as seguintes opções:
   - Em **Escolha uma camada**, selecione **Camadas personalizadas**.
   - Em **Camadas personalizadas**, selecione `pymysqlLibrary`.
   - Em **Versão**, selecione `1`.
4. Selecione **Adicionar**.
5. O painel **Visão geral da função** mostra uma contagem de (1) no nó de Camadas para a função.

### Tarefa 2.4: Importar o código da função extratora de dados do Lambda

1. Acesse a página **Lambda > Funções > salesAnalysisReportDataExtractor**.
2. No painel **Configurações de runtime**, selecione **Editar**.
3. Em **Manipulador**, insira `salesAnalysisReportDataExtractor.lambda_handler`.
4. Selecione **Salvar**.
5. No painel **Origem do código**, selecione **Fazer upload de**.
6. Selecione **Arquivo .zip**.
7. Selecione **Upload** e, depois, navegue e selecione o arquivo `salesAnalysisReportDataExtractor-v3.zip` que você baixou anteriormente.
8. Selecione **Salvar**.
9. O código da função do Lambda é importado e exibido no painel **Origem do código**. Se necessário, no painel de navegação **Ambiente**, clique duas vezes em `salesAnalysisReportDataExtractor.py` para exibir o código.
10. Revise o código Python que implementa a função.

### Tarefa 2.5: Definir configurações de rede para a função

1. Selecione a guia **Configuração** e escolha **VPC**.
2. Selecione **Editar** e configure as seguintes opções:
   - Em **VPC**, selecione a opção com a VPC da cafeteria como o Nome.
   - Em **Sub-redes**, selecione a opção com Sub-rede pública 1 da cafeteria como o Nome.
   - Dica: é possível ignorar o aviso (se houver) que recomenda a escolha de pelo menos duas sub-redes para execução no modo de alta disponibilidade.
   - Em **Grupos de segurança**, selecione a opção com `CafeSecurityGroup` como o Nome.
3. Selecione **Salvar**.

### Tarefa 3: Testar a função extratora de dados do Lambda

**Tarefa 3.1: Iniciar um teste da função do Lambda**

1. Abra o **Console de Gerenciamento da AWS** em uma nova guia do navegador e selecione **Serviços > Gerenciamento e governança > Systems Manager**.
2. No painel de navegação, selecione **Armazenamento de parâmetros**.
3. Selecione cada um dos nomes dos seguintes parâmetros e copie e cole o Valor de cada um em um documento no editor de texto:
   - `/cafe/dbUrl`
   - `/cafe/dbName`
   - `/cafe/dbUser`
   - `/cafe/dbPassword`
4. Retorne à guia do navegador do Console de Gerenciamento do Lambda. Na página da função `salesAnalysisReportDataExtractor`, selecione a guia **Testar**.
5. Configure o painel **Eventos de teste** da seguinte forma:
   - Em **Ação de evento de teste**, selecione **Criar novo evento**.
   - Em **Nome do evento**, insira `SARDETestEvent`.
   - Em **Modelo**, selecione `hello-world`.
6. No painel **JSON do evento**, substitua o objeto JSON pelo seguinte objeto JSON:
   ```json
   {
     "dbUrl": "<value of /cafe/dbUrl parameter>",
     "dbName": "<value of /cafe/dbName parameter>",
     "dbUser": "<value of /cafe/dbUser parameter>",
     "dbPassword": "<value of /cafe/dbPassword parameter>"
   }
   ```
   Substitua o valor de cada parâmetro pelos valores colados em um editor de texto nas etapas anteriores. Insira esses valores entre aspas.
7. Selecione **Salvar**.
8. Selecione **Testar**.
9. Após alguns momentos, a página mostra a mensagem “Resultado da execução: com falha”.

### Tarefa 3.2: Solucionar problemas da função extratora de dados do Lambda

O erro "Task timed out after 3.00 seconds" indica que a função Lambda expirou após três segundos, o que geralmente ocorre quando a função não consegue concluir sua execução no tempo permitido. Aqui estão os passos para solucionar esse problema:

1. **Analise as Regras de Entrada do Grupo de Segurança**:
   - No Console de Gerenciamento da AWS, selecione **VPC** e observe as Regras de Entrada do grupo de segurança usado pela instância do EC2 que executa o banco de dados.
   - Certifique-se de que a porta 3306 (usada pelo MySQL) está listada nas Regras de Entrada. Se não estiver, você pode adicionar essa regra.

2. **Editar o Grupo de Segurança (se necessário)**:
   - Se a porta 3306 não estiver listada, selecione o link do grupo de segurança para editar e adicionar uma regra de entrada que permita o tráfego na porta 3306.

Após realizar essas verificações e possíveis correções, volte à guia do navegador com a página da função `salesAnalysisReportDataExtractor` e execute um novo teste.

### Tarefa 3.3: Analisar e Corrigir a Função do Lambda

1. **Configuração do Grupo de Segurança**:
   - Verifique se o grupo de segurança da instância EC2 permite tráfego na porta 3306 para o banco de dados MySQL.

2. **Reexecutar o Teste**:
   - Retorne à guia do navegador com a página da função `salesAnalysisReportDataExtractor`.
   - Selecione a guia **Testar** e clique em **Testar** novamente.

   Se tudo estiver configurado corretamente, a execução da função deve ser bem-sucedida e você verá uma mensagem “Resultado da execução: com êxito (logs)”.

### Tarefa 3.4: Fazer um Pedido e Testar Novamente

1. **Abrir o Site da Cafeteria**:
   - **Opção 1**:
     - No Console de Gerenciamento da AWS, vá para **Serviços > Computação > Elastic Compute Cloud**.
     - No painel de navegação, selecione **Instâncias**.
     - Selecione `CafeInstance` e copie o Endereço IPv4 público.
     - Em uma guia do navegador, insira `http://publicIP/cafe`, substituindo `publicIP` pelo endereço IPv4 público copiado, e pressione Enter.

   - **Opção 2**:
     - Na parte superior das instruções, selecione **Detalhes** e escolha **Mostrar**.
     - Na janela **Credenciais**, copie o `CafePublicIP`.
     - Em uma guia do navegador, insira `http://publicIP/cafe`, substituindo `publicIP` pelo endereço IPv4 público copiado, e pressione Enter.

2. **Fazer Pedidos no Site da Cafeteria**:
   - No site, selecione **Menu** e faça alguns pedidos para preencher o banco de dados com dados.

3. **Testar a Função Novamente**:
   - Retorne à guia do navegador com a página da função `salesAnalysisReportDataExtractor`.
   - Selecione a guia **Testar** e clique em **Testar**.

   Agora, o objeto JSON exibido deve conter informações sobre os pedidos feitos, semelhantes ao exemplo fornecido.

### Tarefa 4: Configurar Notificações

**Tarefa 4.1: Criar um Tópico do SNS**

1. No Console de Gerenciamento da AWS, selecione **Serviços > Integração de aplicações > Simple Notification Service**.
2. No painel de navegação, selecione **Tópicos** e clique em **Criar tópico**.
   - **Observação**: Se o link **Tópicos** não estiver visível, clique no ícone de três linhas horizontais e selecione **Tópicos**.
3. Configure as seguintes opções na página **Criar tópico**:
   - **Tipo**: Selecione **Padrão**.
   - **Nome**: Insira `salesAnalysisReportTopic`.
   - **Nome de exibição**: Insira `SARTopic`.
4. Clique em **Criar tópico**.
5. Copie e cole o valor ARN em um documento do editor de texto. Você precisará especificar esse ARN ao configurar a próxima função do Lambda.

**Tarefa 4.2: Assinar o Tópico do SNS**

1. Selecione **Criar assinatura** e configure as seguintes opções:
   - **Protocolo**: Selecione **E-mail**.
   - **Endpoint**: Insira um endereço de e-mail que você possa acessar.
2. Clique em **Criar assinatura**.
3. A assinatura será criada com o status **Confirmação pendente**.
4. Verifique a caixa de entrada do endereço de e-mail fornecido. Você deverá ver um e-mail do SARTopic com o assunto “Notificação da AWS – Confirmação de assinatura”.
5. Abra o e-mail e selecione **Confirmar assinatura**. Uma nova guia do navegador será aberta com a mensagem “Assinatura confirmada!”.

### Tarefa 5: Criar a Função do Lambda `salesAnalysisReport`

**Função do Lambda `salesAnalysisReport`**:
- **Recupera** as informações de conexão do banco de dados no armazenamento de parâmetros.
- **Invoca** a função do Lambda `salesAnalysisReportDataExtractor`, que recupera os dados do relatório do banco de dados.
- **Formata e publica** uma mensagem contendo os dados do relatório no tópico do SNS.

**Tarefa 5.1: Conectar-se à Instância CLI Host**

1. No Console de Gerenciamento do EC2, no painel de navegação, selecione **Instâncias**.
2. Na lista de instâncias do EC2, marque a caixa de seleção para a instância **CLI Host**.
3. Clique em **Conectar-se**.
4. Na guia **EC2 Instance Connect**, selecione **Conectar-se** para se conectar à CLI Host.

**Tarefa 5.2: Configurar a AWS CLI**

1. Na janela do terminal EC2 Instance Connect, execute o comando para atualizar o software AWS CLI com as credenciais:
   ```bash
   aws configure
   ```
2. Nos prompts, insira as seguintes informações:
   - **AWS Access Key ID**: Acima destas instruções, selecione o menu suspenso **Detalhes** e escolha **Mostrar**. Na janela **Credenciais**, copie o valor **AccessKey**, cole-o na janela do terminal e pressione Enter.
   - **AWS Secret Access Key**: Na mesma janela **Credenciais**, copie o valor **SecretKey**, cole-o na janela do terminal e pressione Enter.
   - **Default region name**: Insira `us-west-2` (ou a região onde você criou a função anterior do Lambda).
   - **Default output format**: Insira `json` e pressione Enter.

Para criar a função do Lambda `salesAnalysisReport` usando a AWS CLI, siga os passos abaixo:

### Tarefa 5.3: Criar a função do Lambda `salesAnalysisReport` usando a AWS CLI

1. **Verificar se o arquivo `salesAnalysisReport-v2.zip` está na CLI Host**:
   No terminal da instância CLI Host, execute os seguintes comandos:
   ```bash
   cd activity-files
   ls
   ```

2. **Recuperar o ARN do perfil do IAM `salesAnalysisReportRole`**:
   - No Console de Gerenciamento do IAM, selecione **Perfis**.
   - Na caixa de pesquisa, insira `salesAnalysisReportRole` e selecione o nome do perfil.
   - Copie o ARN do perfil e cole-o em um documento do editor de texto.

3. **Criar a função do Lambda usando o comando `create-function`**:
   No terminal da instância CLI Host, execute o seguinte comando, substituindo `<salesAnalysisReportRoleARN>` pelo ARN do perfil que você copiou e `<region>` pelo código da Região `us-west-2`:
   ```bash
   aws lambda create-function \
   --function-name salesAnalysisReport \
   --runtime python3.9 \
   --zip-file fileb://salesAnalysisReport-v2.zip \
   --handler salesAnalysisReport.lambda_handler \
   --region <region> \
   --role <salesAnalysisReportRoleARN>
   ```

   Após a execução bem-sucedida do comando, ele exibirá um objeto JSON descrevendo os atributos da função.

### Tarefa 5.4: Configurar a função do Lambda `salesAnalysisReport`

1. **Abrir o console de gerenciamento do Lambda**:
   - Selecione **Funções** e escolha `salesAnalysisReport`.
   - A página **Detalhes da função** será aberta.

2. **Revisar os detalhes da função**:
   - Leia o código da função e use os comentários incorporados para entender a lógica.

3. **Configurar a variável de ambiente `topicARN`**:
   - Selecione a guia **Configuração** e depois **Variáveis de ambiente**.
   - Selecione **Editar**.
   - Clique em **Adicionar variável de ambiente** e configure as seguintes opções:
     - **Chave**: Insira `topicARN`.
     - **Valor**: Cole o valor do ARN do tópico do SNS `salesAnalysisReportTopic` que você copiou anteriormente.
   - Selecione **Salvar**.
   - A mensagem “A função `salesAnalysisReport` foi atualizada com êxito” será exibida.

### Tarefa 5.5: Testar a função do Lambda `salesAnalysisReport`

1. **Configurar e executar o evento de teste**:
   - Selecione a guia **Testar** e configure o evento de teste da seguinte forma:
     - Em **Ação de evento de teste**, selecione **Criar novo evento**.
     - Em **Nome do evento**, insira `SARTestEvent`.
     - Em **Modelo**, selecione `hello-world`.
   - A função não exige parâmetros de entrada, então deixe o JSON padrão como está.
   - Selecione **Salvar**.
   - Selecione **Testar**.

2. **Resultado esperado**:
   - Uma caixa verde com a mensagem “Resultado da execução: com êxito (logs)” será exibida.
   - Se ocorrer um erro de tempo limite, clique em **Testar** novamente. Se o problema persistir, aumente o tempo limite seguindo estes passos:
     - Selecione a guia **Configuração**.
     - Selecione **Configuração geral**.
     - Selecione **Editar**.
     - Ajuste o **Tempo limite** conforme necessário e selecione **Salvar**.

3. **Verificar o resultado**:
   - A função deve exibir o seguinte objeto JSON:
     ```json
     {
         "statusCode": 200,
         "body": "\"Sale Analysis Report sent.\""
     }

     
     ```
     <img width="385" alt="emailed-report" src="https://github.com/user-attachments/assets/013b2902-4c36-4278-a8d4-6d4df9a2f5fa" />

   - Confira a caixa de entrada de e-mail. Se não houver erros, você receberá um e-mail de Notificações da AWS com o assunto “Relatório diário de análise de vendas”.

Adicionar um gatilho à função do Lambda `salesAnalysisReport` permitirá que o relatório seja gerado automaticamente todos os dias. Aqui estão os passos detalhados para configurar isso:

### Tarefa 5.6: Adicionar um gatilho à função do Lambda `salesAnalysisReport`

1. **Adicionar um Gatilho**:
   - No painel **Visão geral da função**, selecione **Adicionar gatilho**.
   - O painel **Adicionar gatilho** será exibido.

2. **Configurar o Gatilho**:
   - No painel **Configuração do gatilho**, na lista suspensa, selecione **EventBridge (CloudWatch Events)**.
   - Em **Regra**, selecione **Criar uma regra**.
   - Configure as seguintes opções:
     - **Nome da regra**: Insira `salesAnalysisReportDailyTrigger`.
     - **Descrição da regra**: Insira `Initiates report generation on a daily basis`.
     - **Tipo de regra**: Selecione **Expressão de programação**.
     - **Expressão de programação**: Especifique a programação desejada usando uma expressão Cron.

3. **Exemplo de Expressões Cron**:
   - **Se você estiver em Londres (fuso horário UTC) e a hora atual for 11h30**:
     ```cron
     cron(35 11 ? * MON-SAT *)
     ```
   - **Se você estiver em Nova York (fuso horário UTC -5 durante o Horário Padrão do Leste) e a hora atual for 11h30**:
     ```cron
     cron(35 16 ? * MON-SAT *)
     ```

   - Para testar, insira uma expressão que programa o gatilho para cinco minutos a partir da hora atual.

   - **Dica**: Para obter mais informações sobre a sintaxe de expressões de programação para regras, consulte [Schedule Expressions for Rules](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-create-rule-schedule.html).

4. **Adicionar o Gatilho**:
   - Selecione **Adicionar**.
   - O novo gatilho será criado e exibido nos painéis **Visão geral da função** e **Gatilhos**.

### Teste e Verificação

1. **Aguarde cinco minutos** e confira a caixa de entrada de e-mail.
2. **Verifique o e-mail**:
   - Se não houver erros, você verá um novo e-mail de Notificações da AWS com o assunto “Relatório de análise de vendas diárias”.

### Desafio: Ajustar a Expressão Cron para Produção

Para configurar o relatório para ser gerado todos os dias, de segunda a sábado, às 20h no fuso horário UTC, a expressão Cron deverá ser:

```cron
cron(0 20 ? * MON-SAT *)
```

Essa expressão agenda o evento para ser invocado às 20h UTC de segunda a sábado.
