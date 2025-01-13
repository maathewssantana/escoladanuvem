# Otimizar a utilização 🖥️

## Laborátório 🥼

## Objetivo

Otimizar uma instância do Amazon Elastic Compute Cloud (Amazon EC2) para reduzir os custos.

Usar a Calculadora de Preços da AWS para estimar os custos dos serviços da AWS.

### Diagrama do fluxo ✅

Antes e depois da otimização

![optimize-resources-architecture](https://github.com/user-attachments/assets/39ec74b0-9112-4559-bd20-54959993bd14)


# Execução 🚀

**Tarefa 1: Otimizar o site**
1. **Remover banco de dados local**: Remova o banco de dados local da instância EC2, pois ele foi migrado para o Amazon RDS.
2. **Alterar tipo de instância**: Alterne o tipo de instância de t3.small para t3.micro para reduzir custos.

**Tarefa 1.1: Conectar-se à instância usando SSH**
- **Windows**:
  - Baixe o PuTTY e o arquivo labsuser.ppk.
  - Use o PuTTY para conectar-se à instância EC2 utilizando o endereço IP público.
- **macOS/Linux**:
  - Baixe o arquivo labsuser.pem.
  - Abra o terminal e altere o diretório para o local do arquivo.
  - Altere as permissões do arquivo para somente leitura.
  - Conecte-se à instância EC2 utilizando o comando SSH com o endereço IP público.

**Tarefa 1.3: Desinstalar MariaDB e redimensionar a instância**
1. **Desinstalar MariaDB**:
   - Pare e desinstale o MariaDB na instância EC2.
2. **Alterar tipo de instância**:
   - Pare a instância, altere o tipo para t3.micro, e reinicie a instância.

**Tarefa 2: Usar a Calculadora de Preços da AWS**
1. **Estimar custos**: Utilize a Calculadora de Preços da AWS para estimar os custos antes e depois da otimização da instância EC2.

### Tarefa 2.1: Calcular os custos antes da otimização

1. **Instância Amazon EC2**
   - **Região**: Região em que a instância CafeInstance do EC2 está sendo executada
   - **Tipo de instância**: t3.small
   - **Classe da instância**: Sob demanda
   - **Utilização**: 100% ao mês
   - **Sistema operacional**: Linux
   - **Volume do Amazon EBS**: SSD de uso geral (gp2), 40 GB (incluindo 20 GB ocupados pelo banco de dados local)

2. **Instância Amazon RDS**
   - **Classe da instância**: db.t3.micro
   - **Mecanismo**: MariaDB
   - **Armazenamento alocado**: 20 GB

3. **Criar estimativa na Calculadora de Preços da AWS**
   - Acesse [calculator.aws](https://calculator.aws).
   - Clique em "Create estimate".
   - Configure o serviço Amazon EC2 e Amazon RDS conforme os detalhes acima.

### Tarefa 2.2: Calcular os custos após a otimização

1. **Instância Amazon EC2**
   - **Tipo de instância**: t3.micro (reduzido de t3.small)
   - **Amazon Elastic Block Store (EBS)**: SSD de uso geral (gp2), 20 GB (reduzido de 40 GB porque o banco de dados local foi removido)

2. **Modificar a estimativa na Calculadora de Preços da AWS**
   - Acesse a estimativa existente em [calculator.aws](https://calculator.aws/#/estimate).
   - Edite o registro do Amazon EC2 para refletir as mudanças no tipo de instância e no armazenamento.

### Tarefa 2.3: Estimar a economia de custos projetada

1. **Antes da otimização**
   - **Amazon RDS service**: $14.71
   - **Amazon EC2 service**: $20.89
   - **Total**: $35.60

2. **Após a otimização**
   - **Amazon EC2 service**: $10.47
   - **Amazon RDS service**: $14.71
   - **Total**: $25.18

3. **Economia de custos mensal**
   - **Overall monthly cost savings**: $10.42

### Conclusão
Parabéns pela otimização! Com a remoção do banco de dados local e a redução do tamanho da instância EC2, você economizará cerca de $10 por mês.
