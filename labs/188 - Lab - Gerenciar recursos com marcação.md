# Otimizar a utilização 🖥️

## Laborátório 🥼

## Objetivo

Aplicar tags a recursos existentes da AWS.

Encontrar recursos com base em tags.

Usar a AWS CLI ou o AWS SDK para PHP para interromper e terminar as instâncias do Amazon EC2 com base em determinados atributos do recurso.

### Diagrama do fluxo ✅


![lab-7-less-instances](https://github.com/user-attachments/assets/d9c16848-565f-441f-a4f2-5f725b67188b)


# Execução 🚀

**Tarefa 1: Usar tags para gerenciar recursos**

1. **Conectar-se à Command Host**:
   - **Windows**:
     - Baixe e configure o PuTTY.
     - Use o arquivo labsuser.ppk e o IP público para conectar-se via SSH.
   - **Mac/Linux**:
     - Baixe e configure o arquivo labsuser.pem.
     - Use o comando SSH com o IP público para se conectar.

2. **Encontrar instâncias de desenvolvimento para o projeto**:
   - Use a AWS CLI para encontrar instâncias marcadas com a tag `Project=ERPSystem`.
   - Utilize o parâmetro `--query` para limitar a saída a propriedades específicas, como `InstanceId`, `AvailabilityZone`, `Project`, `Environment` e `Version`.

   Exemplos de comandos:

   ```bash
   aws ec2 describe-instances --filter "Name=tag:Project,Values=ERPSystem" --query 'Reservations[*].Instances[*].{ID:InstanceId,AZ:Placement.AvailabilityZone}'
   ```

   ```bash
   aws ec2 describe-instances --filter "Name=tag:Project,Values=ERPSystem" --query 'Reservations[*].Instances[*].{ID:InstanceId,AZ:Placement.AvailabilityZone,Project:Tags[?Key==`Project`] | [0].Value}'
   ```

   ```bash
   aws ec2 describe-instances --filter "Name=tag:Project,Values=ERPSystem" "Name=tag:Environment,Values=development" --query 'Reservations[*].Instances[*].{ID:InstanceId,AZ:Placement.AvailabilityZone,Project:Tags[?Key==`Project`] | [0].Value,Environment:Tags[?Key==`Environment`] | [0].Value,Version:Tags[?Key==`Version`] | [0].Value}'
   ```

**Tarefa 1: Alterar a tag Versão para o processo de desenvolvimento**

1. **Conectar-se à Command Host**:
   - **Windows**:
     - Baixe e configure o PuTTY.
     - Use o arquivo `labsuser.ppk` e o IP público para conectar-se via SSH.
   - **Mac/Linux**:
     - Baixe e configure o arquivo `labsuser.pem`.
     - Use o comando SSH com o IP público para se conectar.

2. **Criar e executar script para alterar a tag Versão**:
   - Crie o arquivo `/home/ec2-user/change-resource-tags.sh`:
     ```bash
     nano change-resource-tags.sh
     ```
   - Adicione o seguinte conteúdo ao script:
     ```bash
     #!/bin/bash

     ids=$(aws ec2 describe-instances --filter "Name=tag:Project,Values=ERPSystem" "Name=tag:Environment,Values=development" --query 'Reservations[*].Instances[*].InstanceId' --output text)

     aws ec2 create-tags --resources $ids --tags 'Key=Version,Value=1.1'
     ```
   - Execute o script:
     ```bash
     ./change-resource-tags.sh
     ```
   - Verifique se a alteração foi aplicada:
     ```bash
     aws ec2 describe-instances --filter "Name=tag:Project,Values=ERPSystem" --query 'Reservations[*].Instances[*].{ID:InstanceId, AZ:Placement.AvailabilityZone, Project:Tags[?Key==`Project`] | [0].Value,Environment:Tags[?Key==`Environment`] | [0].Value,Version:Tags[?Key==`Version`] | [0].Value}'
     ```

**Tarefa 2: Interromper e iniciar recursos por uma tag**

1. **Examinar o script Stopinator**:
   - Abra o arquivo `stopinator.php` e examine seu conteúdo:
     ```bash
     nano stopinator.php
     ```

2. **Interromper e reiniciar instâncias**:
   - Use o script `stopinator.php` para desativar e ativar instâncias de desenvolvimento:
     ```bash
     ./stopinator.php -t"Project=ERPSystem;Environment=development"
     ```
     ```bash
     ./stopinator.php -t"Project=ERPSystem;Environment=development" -s
     ```

**Tarefa 3: Desafio: Terminar instâncias não conformes**

1. **Revisar o script Tag-Or-Terminate**:
   - Abra o arquivo `terminate-instances.php` e examine seu conteúdo:
     ```bash
     nano terminate-instances.php
     ```
   - Identifique todas as instâncias que têm a tag `Environment` definida.
   - Compare com todas as instâncias disponíveis e registre as IDs das instâncias não marcadas.
   - Termine as instâncias não marcadas.
  
**Tarefa 3.1: Revisar o script de Tag-Or-Terminate**

1. **Abrir o arquivo `terminate-instances.php`**:
   ```bash
   nano terminate-instances.php
   ```
   - Examine o bloco `params`: verifique os argumentos `region` e `subnetid`.
   - **Obter lista de instâncias com a tag Environment**:
     - O script usa `describeInstances` para localizar instâncias com a tag Environment.
   - **Comparar instâncias e adicionar IDs não conformes**:
     - Instâncias sem a tag são adicionadas a uma lista de terminação.
   - **Terminar instâncias não conformes**:
     - Usar a lista de IDs para o método `terminateInstances`.

**Configurar o ambiente para testar o script**

1. **Remover a tag Environment**:
   - No Console EC2, selecione instâncias em sua sub-rede privada.
   - Na guia Tags, clique em Adicionar/editar tags.
   - Remova a tag Environment e clique em Salvar.

**Executar o script**

1. **Obter Região e ID da sub-rede**:
   - No Console EC2, na guia Descrição da instância, copie a região (sem a última letra) e o ID da sub-rede.

2. **Executar o script**:
   - Na sessão SSH, execute o script com os argumentos da região e da sub-rede:
     ```bash
     ./terminate-instances.php -region <region> -subnetid <subnet-id>
     ```
   - Resultados esperados:
     ```plaintext
     Verificando i-dd3a90d1 
     Verificando i-a4248ea8 
     ...
     Encerrando instâncias... 
     Instâncias encerradas.
     ```

