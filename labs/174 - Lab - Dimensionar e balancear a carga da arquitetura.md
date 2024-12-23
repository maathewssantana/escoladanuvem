# Dimensionar e balancear a carga da arquitetura 🖥️

## Laborátório 🥼

## Objetivo

Balanceador de carga

- Criar um balanceador de carga.

- Criar um modelo de execução e um grupo do Auto Scaling.

- Configurar um grupo do Auto Scaling para dimensionar novas instâncias em sub-redes privadas.

- Criar alarmes do Amazon CloudWatch para monitorar o desempenho da infraestrutura.

### Diagrama do fluxo ✅

<img width="1267" alt="FinalArchitecture" src="https://github.com/user-attachments/assets/9ab7c680-fb9c-4869-acf9-be4bab6b6855" />


# Execução 🚀

### Tutorial: Criar uma AMI e um Balanceador de Carga na AWS

#### **Tarefa 1: Criar uma AMI para o Auto Scaling**
1. **Acessar Console de EC2**:
   - No Console de Gerenciamento da AWS, pesquise por EC2 e abra o Console de Gerenciamento do Amazon EC2.
2. **Selecionar Instância**:
   - No painel de navegação à esquerda, selecione Instâncias.
   - Encontre e selecione a instância **Web Server 1**.
3. **Criar Imagem**:
   - Na lista suspensa **Ações**, selecione **Imagem e modelos > Criar imagem**.
   - Configure as seguintes opções:
     - **Nome da imagem**: Web Server AMI.
     - **Descrição da imagem (opcional)**: Lab AMI for Web Server.
   - Selecione **Criar imagem**.
   - Anote o ID da nova AMI para uso posterior.

#### **Tarefa 2: Criar um Balanceador de Carga**
1. **Acessar Balanceamento de Carga**:
   - No painel de navegação à esquerda, selecione **Balanceadores de carga**.
   - Selecione **Criar balanceador de carga**.
2. **Configurar Balanceador de Carga**:
   - Para **Application Load Balancer**, selecione a opção **Criar**.
   - Na seção **Configuração básica**, configure:
     - **Nome do balanceador de carga**: LabELB.
   - Na seção **Mapeamento de rede**, configure:
     - **VPC**: Selecione VPC do laboratório.
     - **Mapeamentos**: Selecione as duas Zonas de Disponibilidade listadas.
       - **Sub-rede pública 1** para a primeira Zona de Disponibilidade.
       - **Sub-rede pública 2** para a segunda Zona de Disponibilidade.
   - Na seção **Grupos de segurança**:
     - Remova o grupo de segurança padrão.
     - Selecione **Grupo de segurança da web**.

3. **Configurar Grupo de Destino**:
   - Na seção **Listeners e roteamento**, selecione **Criar grupo de destino**.
   - Configure o grupo de destino na nova guia:
     - **Tipo de destino**: Instâncias.
     - **Nome do grupo de destino**: lab-target-group.
     - Selecione **Próximo** e depois **Criar grupo de destino**.
   - Volte para a guia do balanceador de carga e selecione **Atualizar** na lista suspensa **Encaminhar para**.
   - Selecione **lab-target-group** como o grupo de destino.
   - Selecione **Criar balanceador de carga**.

4. **Salvar Informações do Balanceador de Carga**:
   - Anote o Nome do DNS do balanceador de carga **LabELB**.

#### **Tarefa 3: Criar um Modelo de Execução**
Nesta tarefa, você vai criar um modelo de execução para um grupo do Auto Scaling. O modelo especifica informações essenciais para iniciar instâncias EC2, como AMI, tipo de instância, par de chaves, grupo de segurança e discos.

1. **Acessar Console de EC2**:
   - Na barra de pesquisa do Console de Gerenciamento da AWS, insira e escolha **EC2**.
   - No painel de navegação à esquerda, selecione **Modelos de execução**.

2. **Criar Modelo de Execução**:
   - Selecione **Criar modelo de execução**.
   - Na página **Criar modelo de execução**, configure as seguintes opções:

     - **Nome do modelo de execução**: Insira `lab-app-launch-template`.
     - **Descrição da versão do modelo**: Insira `A web server for the load test app`.
     - **Orientação sobre o Auto Scaling**: Selecione a opção para receber orientação.

3. **Configurar Imagens e Tipo de Instância**:
   - Na seção **Imagens da aplicação e do sistema operacional (imagem de máquina da Amazon)**, selecione a guia **Minhas AMIs** e escolha `WebServerAMI`.
   - Na seção **Tipo de instância**, selecione `t3.micro`.

4. **Configurar Par de Chaves**:
   - Na seção **Par de chaves (login)**, confirme que está definido como **Não incluir no modelo de execução**.

5. **Configurar Grupo de Segurança**:
   - Na seção **Configurações de rede**, selecione o **Grupo de segurança da web**.

6. **Criar Modelo de Execução**:
   - Selecione **Criar modelo de execução**.
   - Você deve receber uma mensagem de sucesso semelhante a **lab-app-launch-template criado com êxito**.
   - Selecione **Visualizar modelos de execução**.

#### **Tarefa 4: Criar um Grupo do Auto Scaling**
Nesta tarefa, você vai usar o modelo de execução para criar um grupo do Auto Scaling, permitindo que novas instâncias EC2 sejam criadas e gerenciadas automaticamente conforme necessário.

1. **Selecionar Modelo de Execução**:
   - Na lista de modelos de execução, selecione **lab-app-launch-template**.
   - Na lista suspensa **Ações**, escolha **Criar grupo do Auto Scaling**.

2. **Configurar Grupo do Auto Scaling**:
   - **Nome do grupo**: Insira `Lab Auto Scaling Group`.
   - Selecione **Próximo**.

3. **Configurar Rede**:
   - **VPC**: Selecione **VPC do laboratório**.
   - **Zonas de disponibilidade e sub-redes**: Selecione **Sub-rede privada 1 (10.0.1.0/24)** e **Sub-rede privada 2 (10.0.3.0/24)**.
   - Selecione **Próximo**.

4. **Configurar Opções Avançadas**:
   - **Balanceamento de carga**: Selecione **Anexar a um balanceador de carga existente**.
   - **Grupos de destino de balanceador de carga existentes**: Selecione `lab-target-group | HTTP`.
   - **Tipo de verificação de integridade**: Selecione **ELB**.
   - Selecione **Próximo**.

5. **Configurar Políticas de Scaling e Tamanho do Grupo**:
   - **Capacidade desejada**: 2.
   - **Capacidade mínima**: 2.
   - **Capacidade máxima**: 4.
   - **Política de escalabilidade de rastreamento de destino**:
     - **Tipo de métrica**: Média de utilização da CPU.
     - **Valor de destino**: 50.
   - Selecione **Próximo**.

6. **Adicionar Notificações (Opcional)**:
   - Selecione **Próximo**.

7. **Adicionar Tags (Opcional)**:
   - Selecione **Adicionar tag**.
   - **Chave**: Insira `Name`.
   - **Valor**: Insira `Lab Instance`.
   - Selecione **Próximo**.

8. **Criar Grupo do Auto Scaling**:
   - Selecione **Criar grupo do Auto Scaling**.

Essas opções irão iniciar instâncias EC2 em sub-redes privadas nas duas Zonas de Disponibilidade, com uma contagem inicial de instâncias igual a zero, mas novas instâncias serão iniciadas para atingir a contagem desejada de duas instâncias.

**Observação**: Se houver um erro relacionado ao tipo de instância `t3.micro` não estar disponível, execute novamente essa tarefa selecionando o tipo de instância `t2.micro`.

#### **Tarefa 5: Verificar se o Balanceamento de Carga está Funcionando**

1. **Verificar Instâncias**:
   - No painel de navegação à esquerda, selecione **Instâncias**.
   - Confirme que há duas novas instâncias chamadas **Instância do laboratório** iniciadas pelo Auto Scaling. Atualize se necessário.

2. **Verificar Health Check**:
   - Na seção **Balanceamento de carga**, selecione **Grupos de destino**.
   - Selecione **lab-target-group**.
   - Na seção **Destinos registrados**, verifique se as duas instâncias estão listadas.
   - Aguarde até que o **Status de integridade** das instâncias mude para **íntegro**.

3. **Acessar Aplicação**:
   - Abra uma nova guia do navegador, cole o **nome do DNS** do balanceador de carga e pressione Enter.
   - A aplicação **Teste de carga** deve aparecer no navegador, indicando que o balanceador de carga está funcionando corretamente.

#### **Tarefa 6: Testar o Auto Scaling**

1. **Verificar Alarmes no CloudWatch**:
   - No Console de Gerenciamento da AWS, na barra de pesquisa, insira e escolha **CloudWatch**.
   - Na seção **Alarmes**, selecione **Todos os alarmes**.
   - Verifique que os alarmes criados automaticamente pelo grupo do Auto Scaling estão presentes.
   - Selecione o alarme que tem **AlarmHigh** no nome e confirme que o estado é **OK**.

2. **Gerar Carga na Aplicação**:
   - Volte para a guia do navegador com a aplicação **Teste de carga**.
   - Selecione **Teste de carga** ao lado do logotipo da AWS para gerar cargas elevadas.

3. **Monitorar Alarmes e Auto Scaling**:
   - Volte para o Console do CloudWatch e atualize a cada sessenta segundos.
   - O alarme **AlarmHigh** deve mudar para **Em alarme** quando a utilização da CPU ultrapassar 50%.
   - Aguarde até que novas instâncias sejam adicionadas pelo Auto Scaling.

4. **Verificar Novas Instâncias**:
   - No Console de Gerenciamento da AWS, na barra de pesquisa, insira e escolha **EC2**.
   - Na seção **Instâncias**, verifique se há mais de duas instâncias denominadas **Instância do laboratório** em execução.

#### **Tarefa 7: Encerrar a Instância Web Server 1**
Nesta tarefa, você vai encerrar a instância Web Server 1 que foi utilizada para criar a AMI usada pelo grupo do Auto Scaling. 

1. **Selecionar a Instância**:
   - Na lista de instâncias, selecione **Web Server 1**.
   - Verifique se apenas a instância **Web Server 1** está selecionada.

2. **Encerrar a Instância**:
   - No menu suspenso **Estado da instância**, selecione **Encerrar instância**.
   - Confirme a ação selecionando **Encerrar**.

