# Trabalhar com o Amazon EBS 💻

## Laborátório 🥼

## Objetivo

Criar um volume do EBS.

- Anexar e montar o volume do EBS em uma instância do EC2.

- Criar um snapshot de um volume do EBS.

- Criar um volume do EBS com base em um snapshot.

### Diagrama do fluxo ✅


<img width="813" alt="lab-scenario" src="https://github.com/user-attachments/assets/24d6800c-bb3a-4161-94e0-72c51f4db4f5" />


# Execução 🚀

### Criar e Anexar um Volume do EBS

#### **Tarefa 1: Criar um Volume do EBS**
1. **Criar Volume**:
   - No Console de Gerenciamento do EC2, selecione **Elastic Block Store** > **Volumes**.
   - Escolha **Criar volume** e configure:
     - **Tipo de volume**: General Purpose SSD (gp2)
     - **Tamanho**: 1 GiB
     - **Zona de Disponibilidade**: Mesma que a instância EC2 (us-west-2a)
     - **Tags**: Adicione a tag `Name: My Volume`
   - Selecione **Criar volume**.

#### **Tarefa 2: Anexar o Volume a uma Instância do EC2**
1. **Anexar Volume**:
   - Selecione `My Volume`.
   - No menu **Ações**, escolha **Anexar volume**.
   - Na lista **Instância**, escolha a instância `Lab`.
   - O campo **Nome do dispositivo** é `/dev/sdf`.
   - Selecione **Associar volume**.

#### **Tarefa 3: Conectar-se à Instância do EC2**
1. **Conectar-se à Instância**:
   - No Console de Gerenciamento do EC2, selecione a instância `Lab`.
   - Escolha **Conectar-se** > **EC2 Instance Connect**.
   - Selecione **Conectar-se** para abrir a janela do terminal do EC2 Instance Connect.

#### **Tarefa 4: Criar e Configurar o Sistema de Arquivos**
1. **Visualizar Armazenamento Disponível**:
   - Execute o comando:
     ```bash
     df -h
     ```

2. **Criar Sistema de Arquivos ext3**:
   - Formate o novo volume com o sistema de arquivos ext3:
     ```bash
     sudo mkfs -t ext3 /dev/sdf
     ```

3. **Criar Diretório de Montagem**:
   - Crie o diretório onde o novo volume será montado:
     ```bash
     sudo mkdir /mnt/data-store
     ```

4. **Montar o Novo Volume**:
   - Monte o volume recém-criado no diretório:
     ```bash
     sudo mount /dev/sdf /mnt/data-store
     ```
   - Adicione a configuração no arquivo `/etc/fstab` para montagem automática:
     ```bash
     echo "/dev/sdf   /mnt/data-store ext3 defaults,noatime 1 2" | sudo tee -a /etc/fstab
     ```

5. **Verificar Configuração de Montagem**:
   - Verifique o conteúdo do arquivo de configuração:
     ```bash
     cat /etc/fstab
     ```
   - Exiba novamente o armazenamento disponível:
     ```bash
     df -h
     ```

6. **Criar e Verificar Arquivo no Volume Montado**:
   - Crie um arquivo e adicione texto no volume montado:
     ```bash
     sudo sh -c "echo some text has been written > /mnt/data-store/file.txt"
     ```
   - Verifique o conteúdo do arquivo:
     ```bash
     cat /mnt/data-store/file.txt
     ```

#### **Tarefa 5: Criar um Snapshot do Amazon EBS**
1. **Criar Snapshot**:
   - No Console de Gerenciamento do EC2, selecione **Volumes** e escolha `My Volume`.
   - No menu **Ações**, selecione **Criar snapshot**.
   - Adicione uma tag com **Chave**: `Name` e **Valor**: `My Snapshot`.
   - Selecione **Criar snapshot**.
2. **Verificar Status do Snapshot**:
   - No painel de navegação à esquerda, selecione **Snapshots**.
   - O status mudará de `Pendente` para `Concluído` após a criação.
3. **Excluir Arquivo do Volume**:
   - No terminal do EC2 Instance Connect, execute:
     ```bash
     sudo rm /mnt/data-store/file.txt
     ```
   - Verifique se o arquivo foi excluído:
     ```bash
     ls /mnt/data-store/file.txt
     ```

#### **Tarefa 6: Restaurar o Snapshot do Amazon EBS**
1. **Criar Volume Usando o Snapshot**:
   - No Console de Gerenciamento do EC2, selecione `My Snapshot`.
   - No menu **Ações**, selecione **Criar volume com o snapshot**.
   - Selecione a mesma **Zona de disponibilidade** utilizada anteriormente.
   - Adicione uma tag com **Chave**: `Name` e **Valor**: `Restored Volume`.
   - Selecione **Criar volume**.
2. **Verificar Status do Novo Volume**:
   - No painel de navegação à esquerda, selecione **Volumes**.
   - O status do novo volume será `Disponível`.

#### **Tarefa 6.2: Anexar o Volume Restaurado à Instância do EC2**
1. **Anexar Volume**:
   - Selecione `Restored Volume`.
   - No menu **Ações**, escolha **Anexar volume**.
   - Na lista **Instância**, escolha a instância `Lab`.
   - O campo **Nome do dispositivo** está definido como `/dev/sdg`.
   - Selecione **Associar volume**.
   - O status do volume mudará para `Em uso`.

#### **Tarefa 6.3: Montar o Volume Restaurado**
1. **Criar Diretório de Montagem**:
   - No terminal do EC2 Instance Connect, execute:
     ```bash
     sudo mkdir /mnt/data-store2
     ```
2. **Montar o Volume**:
   - Monte o volume restaurado:
     ```bash
     sudo mount /dev/sdg /mnt/data-store2
     ```
3. **Verificar Arquivo**:
   - Verifique se o arquivo `file.txt` está presente:
     ```bash
     ls /mnt/data-store2/file.txt
     ```
   - Você deve ver o arquivo `file.txt`.

