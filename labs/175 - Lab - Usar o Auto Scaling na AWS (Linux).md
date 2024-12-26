# Usar o Auto Scaling na AWS (Linux) 🖥️

## Laborátório 🥼

## Objetivo

- Criar uma instância do EC2 usando um comando da AWS CLI.

- Criar uma AMI usando a AWS CLI.

- Criar um modelo de execução do Amazon EC2.

- Criar uma configuração de execução do Amazon EC2 Auto Scaling.

- Configurar as políticas de scaling e criar um grupo do Auto Scaling para reduzir ou aumentar a quantidade de servidores com base em uma carga variável.



### Diagrama do fluxo ✅

Fluxo inicial:

<img width="598" alt="antes" src="https://github.com/user-attachments/assets/1e790460-6711-412f-a003-444b66d354cb" />



Fluxo Final

<img width="852" alt="depois" src="https://github.com/user-attachments/assets/1a331c6c-f945-433a-b633-408171652ab6" />




# Execução 🚀
### Criar uma AMI para o Amazon EC2 Auto Scaling

#### **Tarefa 1: Criar uma AMI para o Amazon EC2 Auto Scaling**
Nesta tarefa, você aprenderá a criar uma AMI (Amazon Machine Image) usando uma instância EC2 em execução. Você executará todas as operações usando a AWS CLI na instância Command Host.

#### **Tarefa 1.1: Conectar à Instância Command Host**
1. **Acessar o Console EC2**:
   - No Console de Gerenciamento da AWS, pesquise por **EC2** e abra o Console de Gerenciamento do EC2.
2. **Selecionar Instância**:
   - No painel de navegação, selecione **Instâncias**.
   - Encontre e selecione a instância **Command Host**.
3. **Conectar-se à Instância**:
   - Selecione **Conectar-se**.
   - Na guia **EC2 Instance Connect**, selecione **Conectar-se**.

**Observação**: Se preferir usar um cliente SSH, consulte as orientações específicas para conectar-se à instância do Linux.

#### **Tarefa 1.2: Configurar a AWS CLI**
1. **Confirmar a Região**:
   - Execute o comando para confirmar a região da instância:
     ```bash
     curl http://169.254.169.254/latest/dynamic/instance-identity/document | grep region
     ```
2. **Configurar a AWS CLI**:
   - Atualize as credenciais da AWS CLI executando:
     ```bash
     aws configure
     ```
   - Nos prompts, insira as seguintes informações:
     - **ID de chave de acesso da AWS**: Pressione Enter.
     - **AWS Secret Access Key**: Pressione Enter.
     - **Default region name**: Insira a região confirmada anteriormente (por exemplo, us-west-2).
     - **Default output format**: Insira `json`.
    
  Ele não fornece diretamente as credenciais no show, abaixo deixo o local onde pode pegar:
  

3. **Navegar para o Diretório**:
   - Acesse o diretório com os scripts necessários:
     ```bash
     cd /home/ec2-user/
     ```

#### **Tarefa 1.3: Criar uma Instância do EC2**
Nesta tarefa, você usará a AWS CLI para criar uma instância EC2 que hospede um servidor da web.

1. **Inspecionar o Script UserData**:
   - Execute o seguinte comando para visualizar o script `UserData.txt`:
     ```bash
     more UserData.txt
     ```
   - O script realiza várias tarefas de inicialização, incluindo a atualização de software e a instalação de um aplicativo web PHP.

2. **Coletar Valores Necessários**:
   - Na parte superior da página, selecione **Detalhes** e clique em **Mostrar**.
   - Copie os valores `KEYNAME`, `AMIID`, `HTTPACCESS` e `SUBNETID` em um editor de texto e feche o painel Credenciais.

3. **Modificar e Executar o Script**:
   - Substitua os valores no seguinte script pelos valores coletados:
     ```bash
     aws ec2 run-instances --key-name KEYNAME --instance-type t3.micro --image-id AMIID --user-data file:///home/ec2-user/UserData.txt --security-group-ids HTTPACCESS --subnet-id SUBNETID --associate-public-ip-address --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=WebServer}]' --output text --query 'Instances[*].InstanceId'
     ```
   - Execute o script modificado na janela do terminal.
   - Copie e guarde o valor `InstanceId` fornecido na saída do comando.

4. **Monitorar o Status da Instância**:
   - Use o comando `aws ec2 wait instance-running` para monitorar o status da instância:
     ```bash
     aws ec2 wait instance-running --instance-ids NEW-INSTANCE-ID
     ```
   - Aguarde o comando retornar para um prompt antes de prosseguir.

5. **Testar o Servidor Web**:
   - Obtenha o nome do DNS público da instância:
     ```bash
     aws ec2 describe-instances --instance-id NEW-INSTANCE-ID --query 'Reservations[0].Instances[0].NetworkInterfaces[0].Association.PublicDnsName'
     ```
   - Copie a saída sem as aspas. Este será o `PUBLIC-DNS-ADDRESS`.

6. **Verificar a Instalação do Servidor Web**:
   - Abra uma nova guia do navegador e insira o `PUBLIC-DNS-ADDRESS` copiado.
   - Aguarde alguns minutos para que o servidor da web seja instalado.
   - Teste a URL:
     ```bash
     http://PUBLIC-DNS-ADDRESS/index.php
     ```
   - Se o servidor da web não estiver em execução, verifique com o instrutor.

#### **Tarefa 1.4: Criar uma AMI Personalizada**
1. **Criar AMI**:
   - Utilize o comando `aws ec2 create-image` para criar uma AMI baseada na instância recém-criada:
     ```bash
     aws ec2 create-image --name WebServerAMI --instance-id NEW-INSTANCE-ID
     ```
   - Este comando reiniciará a instância para garantir a integridade da imagem no sistema de arquivos.

#### **Tarefa 2: Criar um Ambiente de Auto Scaling**
Nesta seção, você configurará um balanceador de carga e um grupo de Auto Scaling para gerenciar dinamicamente instâncias EC2 com base na carga.

#### **Tarefa 2.1: Criar um Application Load Balancer**
1. **Acessar Balanceadores de Carga**:
   - No Console de Gerenciamento do EC2, selecione **Balanceadores de carga**.
   - Selecione **Criar balanceador de carga**.
   - Escolha **Application Load Balancer** e selecione **Criar**.

2. **Configurar Balanceador de Carga**:
   - **Nome do balanceador de carga**: Insira `WebServerELB`.
   - **VPC**: Selecione `VPC do laboratório`.
   - **Mapeamentos**: Selecione as duas Zonas de Disponibilidade (Sub-rede pública 1 e Sub-rede pública 2).
   - **Grupos de segurança**: Remova o grupo de segurança padrão e selecione `HTTPAccess`.

3. **Configurar Grupo de Destino**:
   - Na seção **Listeners e roteamento**, selecione **Criar grupo de destino**.
   - Configure o grupo de destino:
     - **Tipo de destino**: Instâncias.
     - **Nome do grupo de destino**: `webserver-app`.
     - **Tipo de verificação de integridade**: Selecione `/index.php`.
   - Selecione **Próximo** e depois **Criar grupo de destino**.
   - Volte para a guia do balanceador de carga e selecione **Atualizar** em **Encaminhar para**, escolhendo `webserver-app`.

4. **Criar Balanceador de Carga**:
   - Selecione **Criar balanceador de carga**.
   - Após a criação, copie o Nome do DNS do balanceador de carga `WebServerELB` e guarde-o para uso posterior.

#### **Tarefa 2.2: Criar um Modelo de Execução**
1. **Acessar Modelos de Execução**:
   - No Console de Gerenciamento do EC2, selecione **Modelos de execução** no painel de navegação à esquerda.
2. **Criar Modelo de Execução**:
   - Selecione **Criar modelo de execução**.
   - Configure as seguintes opções:
     - **Nome do modelo de execução**: `web-app-launch-template`
     - **Descrição da versão do modelo**: `A web server for the load test app`
     - **Orientação sobre o Auto Scaling**: Selecione para fornecer orientação.
   - Na seção **Imagens da aplicação e do sistema operacional**, selecione `WebServerAMI` na guia Minhas AMIs.
   - Na seção **Tipo de instância**, selecione `t3.micro`.
   - Na seção **Par de chaves (login)**, confirme que está definido como **Não incluir no modelo de execução**.
   - Na seção **Configurações de rede**, selecione o grupo de segurança `HTTPAccess`.
   - Selecione **Criar modelo de execução**.

3. **Verificar Criação do Modelo**:
   - Você deve receber a mensagem `web-app-launch-template criado com êxito`.
   - Selecione **Visualizar modelos de execução**.

#### **Tarefa 2.3: Criar um Grupo do Auto Scaling**
1. **Selecionar Modelo de Execução**:
   - Selecione `web-app-launch-template` e, na lista suspensa **Ações**, escolha **Criar grupo do Auto Scaling**.
2. **Configurar Grupo do Auto Scaling**:
   - **Nome do grupo**: Insira `Web App Auto Scaling Group`.
   - **Rede**:
     - **VPC**: Selecione `VPC do laboratório`.
     - **Zonas de disponibilidade e sub-redes**: Selecione `Sub-rede privada 1 (10.0.2.0/24)` e `Sub-rede privada 2 (10.0.4.0/24)`.
   - **Balanceamento de carga**:
     - Selecione **Anexar a um balanceador de carga existente**.
     - **Grupos de destino de balanceador de carga existentes**: Selecione `webserver-app | HTTP`.
     - **Tipos de verificações de integridade adicionais**: Ative as verificações de integridade do Elastic Load Balancing.
   - **Políticas de Scaling e Tamanho do Grupo**:
     - **Capacidade desejada**: 2
     - **Capacidade mínima**: 2
     - **Capacidade máxima**: 4
     - **Política de escalabilidade de rastreamento de destino**:
       - **Tipo de métrica**: Média de utilização da CPU.
       - **Valor de destino**: 50
   - Selecione **Próximo**.
3. **Adicionar Tags**:
   - Selecione **Adicionar tag**.
   - **Chave**: `Name`
   - **Valor**: `WebApp`
   - Selecione **Próximo**.
4. **Criar Grupo do Auto Scaling**:
   - Selecione **Criar grupo do Auto Scaling**.

Essas opções iniciarão instâncias do EC2 em sub-redes privadas nas duas Zonas de Disponibilidade.

**Observação**: Se houver um erro relacionado ao tipo de instância `t3.micro` não estar disponível, execute novamente essa tarefa selecionando o tipo de instância `t2.micro`.

#### **Tarefa 3: Verificar a Configuração do Auto Scaling**
1. **Verificar Instâncias**:
   - No painel de navegação à esquerda, selecione **Instâncias**.
   - Duas novas instâncias chamadas **WebApp** estão sendo criadas como parte do grupo do Auto Scaling. A verificação de status dessas instâncias estará como **Inicializando**.
   - Aguarde até que o status seja **Aprovado em 2/2 verificações**. Atualize a página conforme necessário.

2. **Verificar Grupos de Destino**:
   - No painel de navegação à esquerda, selecione **Grupos de destino** na seção **Balanceamento de carga**.
   - Selecione o grupo de destino **webserver-app**.
   - Verifique se as duas instâncias estão sendo criadas e se o status de verificação é **íntegro**.

#### **Tarefa 4: Testar a Configuração do Auto Scaling**
1. **Acessar Aplicação Web**:
   - Abra uma nova guia do navegador e cole o **Nome do DNS** do balanceador de carga na barra de endereços e pressione Enter.
   - Na página da web, clique em **Iniciar stress**. Isso aumentará a utilização da CPU para 100% na instância que atendeu à solicitação.

2. **Verificar Atividade do Auto Scaling**:
   - No Console de Gerenciamento do EC2, selecione **Grupos do Auto Scaling** na seção **Auto Scaling**.
   - Selecione o **Grupo do Auto Scaling do aplicativo web**.
   - Na guia **Atividade**, aguarde alguns minutos até que uma nova instância seja adicionada. Isso acontece porque o Amazon CloudWatch detectou que a utilização média da CPU excedeu 50%, invocando a política de expansão.

3. **Verificar Novas Instâncias**:
   - Confirme as novas instâncias sendo iniciadas no painel do EC2.

### Conclusão
Este tutorial ensina como verificar e testar a configuração do Auto Scaling e balanceamento de carga na AWS. Ao seguir essas etapas, você garante que suas instâncias EC2 possam escalar automaticamente conforme a carga aumenta, proporcionando alta disponibilidade e desempenho eficiente.



