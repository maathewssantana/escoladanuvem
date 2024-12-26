# Roteamento de failover do Amazon Route 53 🖥️

## Laborátório 🥼

## Objetivo

- Configurar uma health check do Route 53 que envie e-mails quando a integridade de um endpoint HTTP deixar de ser íntegro.

- Configurar o roteamento de failover no Route 53.



### Diagrama do fluxo ✅

![failover-arch](https://github.com/user-attachments/assets/be4e0d93-bef6-42d9-b5e1-384a1a2697d1)



# Execução 🚀

### Confirmar os Sites da Cafeteria

#### **Tarefa 1: Confirmar os Sites da Cafeteria**
Nesta tarefa, você analisará os recursos criados automaticamente pelo AWS CloudFormation para a aplicação da cafeteria.

1. **Obter Detalhes de Credenciais**:
   - Na parte superior da página, selecione **Detalhes** e clique em **Mostrar** em AWS.
   - Copie os valores dos parâmetros a seguir para um editor de texto:
     - `CafeInstance1IPAddress`
     - `PrimaryWebSiteURL`
     - `SecondaryWebsiteURL`
     - `CafeInstance2IPAddress`
   - Feche o painel Credenciais.

2. **Acessar o Console de Gerenciamento do EC2**:
   - Na barra de pesquisa, insira e escolha **EC2**.
   - No painel de navegação à esquerda, na seção **Instâncias**, selecione **Instâncias**.
   - Duas instâncias do EC2 já foram criadas: `CafeInstance1` (sub-rede pública 1) e `CafeInstance2` (sub-rede pública 2).

3. **Verificar Aplicações nas Instâncias**:
   - Abra uma nova guia do navegador e cole o valor de `PrimaryWebSiteURL`. A página da aplicação da cafeteria será aberta.
   - Observe as informações do servidor e a Zona de Disponibilidade.
   - Abra uma nova guia do navegador e cole o valor de `SecondaryWebsiteURL`. Confirme que a segunda instância possui configurações semelhantes.

4. **Testar Funcionalidade da Aplicação**:
   - Em um dos sites, selecione **Menu**.
   - Selecione um item do menu e escolha **Enviar pedido**. A página de Confirmação do Pedido mostrará o horário do pedido.
   - Confirme que ambas as instâncias estão executando a aplicação da cafeteria em diferentes Zonas de Disponibilidade para oferecer alta disponibilidade.

#### **Tarefa 2: Configurar uma Health Check do Route 53**
Nesta tarefa, você configurará uma health check no Route 53 para monitorar o site primário, essencial para o failover.

1. **Acessar o Console do Route 53**:
   - No Console de Gerenciamento da AWS, vá para o menu **Serviços** e selecione **Route 53**.

2. **Criar Verificação de Integridade**:
   - No painel de navegação à esquerda, selecione **Verificações de integridade**.
   - Selecione **Criar verificação de integridade** e configure:
     - **Nome**: `Primary-Website-Health`
     - **O que monitorar**: Endpoint
     - **Especifique o endpoint por**: Endereço IP
     - **Endereço IP**: Cole o endereço IPv4 público de `CafeInstance1`
     - **Caminho**: `cafe`
   - Expanda **Configuração avançada** e configure:
     - **Intervalo de solicitações**: Rápido (10 segundos)
     - **Limite de falha**: 2
   - Selecione **Próximo**.

3. **Configurar Notificações**:
   - Em **Ser notificado se uma verificação de integridade falhar**, configure:
     - **Criar alarme**: Sim
     - **Enviar notificação para**: Novo tópico do SNS
     - **Nome do tópico**: `Primary-Website-Health`
     - **Endereço de e-mail do destinatário**: Insira um e-mail acessível
   - Selecione **Criar verificação de integridade**.

4. **Verificar o Status da Health Check**:
   - O Route 53 irá monitorar periodicamente o site.
   - Pode levar até um minuto para que a health check mostre o status **Íntegro**. Atualize a visualização conforme necessário.
   - Selecione **Integridade do site principal** e a guia **Monitoramento** para uma visualização detalhada.
   - Confira seu e-mail e selecione o link **Confirmar assinatura** para concluir a configuração do alerta.

#### **Tarefa 3: Configurar Registros do Route 53**
Você aprenderá a configurar registros do Route 53 para a zona hospedada, incluindo o roteamento de failover.

#### **Tarefa 3.1: Criar um Registro A para o Site Principal**
1. **Acessar Zonas Hospedadas**:
   - No console do Route 53, no painel de navegação à esquerda, selecione **Zonas hospedadas**.
   - Selecione o domínio `XXXXXX_XXXXXXXXXX.vocareum.training` criado para você.

2. **Entender os Registros Existentes**:
   - Dois registros já existem: NS (servidores de nomes) e SOA (início do registro de autoridade).
   - Não adicione, altere ou exclua servidores de nomes desses registros.

3. **Criar um Novo Registro A**:
   - Selecione **Criar registro** e configure:
     - **Nome do registro**: `www`
     - **Tipo de registro**: `A: Rotear o tráfego para um endereço IPv4 e alguns recursos da AWS`
     - **Valor**: Insira o endereço IP para `CafeInstance1IPAddress`
     - **TTL (segundos)**: 15
     - **Política de roteamento**: `Failover`
     - **Tipo de registro de failover**: `Primário`
     - **ID da verificação de integridade**: `Integridade do site principal`
     - **ID do registro**: `FailoverPrimary`
   - Selecione **Criar registros**.

4. **Verificar Novo Registro**:
   - O novo registro do tipo A deve aparecer como o terceiro registro na página **Zonas hospedadas**.

#### **Tarefa 3.2: Criar um Registro A para o Site Secundário**
1. **Criar Registro A**:
   - No console do Route 53, selecione **Criar registro**.
   - Configure as seguintes opções:
     - **Nome do registro**: `www`
     - **Tipo de registro**: `A: Rotear o tráfego para um endereço IPv4 e alguns recursos da AWS`
     - **Valor**: Insira o endereço IP de `CafeInstance2IPAddress`
     - **TTL (segundos)**: 15
     - **Política de roteamento**: `Failover`
     - **Tipo de registro de failover**: `Secundário`
     - **ID da verificação de integridade**: Deixe em branco
     - **ID do registro**: `FailoverSecondary`
   - Selecione **Criar registros**.
2. **Verificar Novo Registro**:
   - Outro registro do tipo A aparecerá na página **Zonas hospedadas**.

#### **Tarefa 4: Verificar a Resolução de DNS**
1. **Verificar Registro A**:
   - Marque a caixa de seleção de um dos registros A.
   - No painel **Detalhes do registro**, copie o valor **Nome do registro**.
2. **Testar Resolução de DNS**:
   - Abra uma nova guia do navegador, cole o nome do registro A e insira `/cafe` no final do URL.
   - Carregue a página para verificar se o site principal da cafeteria é exibido.

   Dica: O URL deverá ser algo como `http://www.XXXXXX_XXXXXXXXXX.vocareum.training/cafe/`, onde os Xs são dígitos exclusivos.

#### **Tarefa 5: Verificar a Funcionalidade do Failover**
Nesta tarefa, você verificará se o Route 53 falha corretamente para o servidor secundário em caso de falha no servidor primário. 

1. **Simular Falha no Servidor Primário**:
   - No Console de Gerenciamento da AWS, vá para o menu **Serviços**, selecione **EC2** e escolha **Instâncias**.
   - Selecione `CafeInstance1`.
   - No menu **Estado da instância**, escolha **Interromper instância**.
   - Na janela **Interromper instância?**, selecione **Interromper**.

2. **Verificar Health Check no Route 53**:
   - No menu **Serviços**, selecione **Route 53**.
   - No painel de navegação à esquerda, selecione **Verificações de integridade**.
   - Selecione **Integridade do site principal** e, no painel inferior, escolha **Monitoramento**.
   - Aguarde até que o status de integridade do site principal mude para **Não íntegro**. Atualize a página conforme necessário.

3. **Testar Failover no Navegador**:
   - Volte para a guia do navegador onde o site `vocareum_XXXXXX_XXXXXXXXXX.training/cafe` está aberto e atualize a página.
   - Observe que a **Região/Zona de disponibilidade** agora exibe uma Zona de Disponibilidade diferente (por exemplo, `us-west-2b` em vez de `us-west-2a`), indicando que o site está sendo servido pela instância `CafeInstance2`.

4. **Confirmar Notificação por E-mail**:
   - Verifique o e-mail para uma notificação da AWS intitulada “ALARM: Primary-Website-Health-awsroute53-...”.
   - Confirme os detalhes sobre o que acionou o alarme.

### Conclusão
Este tutorial ensina a verificar a funcionalidade de failover configurando uma interrupção manual da instância primária e observando se o Route 53 redireciona corretamente o tráfego para a instância secundária. Seguindo estas etapas, você assegura que sua aplicação web possui um failover robusto e eficiente.

Se precisar de mais informações ou tiver outras perguntas, estou aqui para ajudar! 😊
