# Solucionar problemas com a criação de uma instância do EC2 🖥️

## Laborátório 🥼

## Objetivo

- Iniciar uma instância do EC2 usando a AWS CLI.

- Solucionar problemas em comandos da AWS CLI e nas configurações do serviço Amazon EC2 usando dicas básicas de solução de problemas e o utilitário nmap de código aberto.

### Diagrama do fluxo ✅

<img width="1568" alt="Architecture" src="https://github.com/user-attachments/assets/5148461b-9792-4fc9-9ea5-381ba381602e" />


# Execução 🚀


### Tutorial: Conexão e Configuração da AWS CLI

#### **Tarefa 1: Conectar-se à Instância CLI Host**
1. **Acessar Console de EC2**:
   - No Console de Gerenciamento da AWS, pesquise por EC2 e abra o Console de Gerenciamento do Amazon EC2.
2. **Selecionar Instância**:
   - No painel de navegação, selecione Instâncias.
   - Na lista de instâncias, selecione a instância CLI Host.
3. **Conectar-se à Instância**:
   - Acima da lista de instâncias, selecione Conectar-se.
   - Na guia EC2 Instance Connect, selecione Conectar-se.

**Observação**: Se preferir usar um cliente SSH para se conectar, consulte as orientações em "Conecte-se à sua instância do Linux".

#### **Tarefa 2: Configurar a AWS CLI**
1. **Configurar a AWS CLI**:
   - Para configurar o perfil da AWS CLI, execute o comando: `aws configure`.
2. **Inserir Credenciais**:
   - **AWS Access Key ID**: Insira o ID da chave de acesso.
   - **AWS Secret Access Key**: Insira a chave de acesso secreta.
   - **Default Region Name**: Insira o nome da região padrão.
   - **Default Output Format**: Insira `json`.

#### **Tarefa 3: Criar uma Instância do EC2**
Nesta tarefa, você irá criar uma instância LAMP (Linux, Apache, MySQL, PHP) do EC2 usando um script de shell que contém problemas intencionais para que você possa identificar e corrigir. 

#### **Tarefa 3.1: Observar os Detalhes do Script**
1. **Criar um Backup do Script**:
   - Mude para o diretório onde o arquivo de script está localizado e crie um backup:
     ```bash
     cd ~/sysops-activity-files/starters
     cp create-lamp-instance-v2.sh create-lamp-instance.backup
     ```

2. **Abrir o Script em um Editor de Texto**:
   - Use o editor VI para abrir o arquivo de script:
     ```bash
     view create-lamp-instance-v2.sh
     ```

3. **Analisar o Conteúdo do Script**:
   - O script começa com `#!/bin/bash` indicando que é um arquivo bash.
   - Define o tamanho da instância como `t3.small`.
   - Invoca `describe-regions` para listar todas as regiões AWS e consulta uma VPC existente chamada "VPC da cafeteria".
   - Usa comandos da AWS CLI para obter IDs da sub-rede, nome do par de chaves e ID da AMI.
   - Limpa recursos AWS preexistentes, como instâncias `cafeserver` e grupos de segurança `cafeSG`.
   - Cria um grupo de segurança com portas 22 e 80 abertas.
   - Cria a instância EC2 usando valores coletados anteriormente.
   - Analisa e reflete detalhes da instância EC2 no terminal.

4. **Visualizar o Script de Dados do Usuário**:
   - Exibir o conteúdo do script de dados do usuário que instala um servidor web, PHP e um servidor de banco de dados:
     ```bash
     cat create-lamp-instance-userdata-v2.txt
     ```

#### **Tarefa 3.2: Tentar Executar o Script**
1. **Executar o Script**:
   - Tente executar o script de shell:
     ```bash
     ./create-lamp-instance-v2.sh
     ```
   - O script falha e fecha sem ser concluído. Isso está dentro do esperado.

#### **Tarefa 3.3: Solução de Problemas**

**Problema #1**:
- **Erro**: `InvalidAMIID.NotFound: o ID da imagem ‘[ami-xxxxxxxxxx”]’ não existe`.
- **Dicas**:
  - Localize a linha no script que gerou o erro.
  - Verifique se o valor de região usado para o comando `run-instances` está correto.
  - Corrija o problema no script e execute-o novamente.
- **Resultado Esperado**: O comando `run-instances` deve ser executado com êxito e um endereço IPv4 público deve ser atribuído à nova instância.

**Problema #2**:
- **Erro**: Página da web de teste não carrega.
- **Dicas**:
  - Verifique se a porta TCP 80 está aberta.
  - Verifique se o serviço do servidor web (httpd) está funcionando.
  - Instale `nmap` para verificar as portas:
    ```bash
    sudo yum install -y nmap
    nmap -Pn <public-ip>
    ```
  - Teste se o script de dados do usuário foi executado.
- **Resultado Esperado**: A mensagem “Saudações do seu servidor web” deve ser exibida no navegador.

- **Verificar Script de Dados do Usuário**:
  - Visualizar o arquivo de log:
    ```bash
    sudo tail -f /var/log/cloud-init-output.log
    ```
  - Certifique-se de que não há mensagens de erro e que os arquivos do aplicativo web foram baixados e extraídos corretamente.

#### **Tarefa 4: Verificar a Funcionalidade do Site**
1. **Verificar Implantação do Site**:
   - Acesse `http://<public-ip>/cafe` no navegador.
   - A página inicial do site da cafeteria deve ser exibida.

2. **Testar Funcionalidade do Site**:
   - Clique no link Menu e selecione sobremesas para pedir.
   - Verifique a Confirmação do Pedido e o Histórico de Pedidos.

Se precisar de mais detalhes ou tiver outras perguntas, estou aqui para ajudar! 😊
