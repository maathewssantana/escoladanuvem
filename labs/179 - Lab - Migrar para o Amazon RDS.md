# Migrar para o Amazon RDS 🖥️

## Laborátório 🥼

## Objetivo

- Criar uma instância do MariaDB do Amazon RDS usando a AWS CLI.

- Migrar dados de um banco de dados do MariaDB em uma instância do EC2 para uma instância do MariaDB do Amazon RDS.

- Monitorar a instância do Amazon RDS usando as métricas do Amazon CloudWatch.

### Diagrama do fluxo ✅

Arquitetura inicial

O diagrama a seguir ilustra a topologia do ambiente de runtime do aplicativo web da cafeteria antes da migração. O banco de dados da aplicação é executado em uma instância do Linux do Amazon Elastic Compute Cloud (Amazon EC2), do Apache, do MySQL e do PHP (LAMP) com o código da aplicação. A instância é do tipo T3 small e é executada em uma sub-rede pública para que os clientes da internet possam acessar o site. Uma instância CLI Host reside na mesma sub-rede para viabilizar a administração da instância usando a AWS Command Line Interface (AWS CLI).

<img width="408" alt="StartingArchitecture" src="https://github.com/user-attachments/assets/f367a8d8-0438-4055-adb6-ff4c8aac42c4" />

Arquitetura final

O diagrama a seguir ilustra a topologia do ambiente de runtime do aplicativo web da cafeteria depois da migração.
Você migrará o banco de dados local da cafeteria para um banco de dados do Amazon RDS que reside fora da instância. O banco de dados do Amazon RDS será implantado na mesma nuvem privada virtual (VPC) da instância.

<img width="758" alt="FinalArchitecture" src="https://github.com/user-attachments/assets/66946a03-a9ec-4db2-8613-35710febf28c" />


# Execução 🚀

