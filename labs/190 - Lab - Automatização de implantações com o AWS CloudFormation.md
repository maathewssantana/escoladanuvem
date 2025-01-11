#A utomação com o CloudFormation🖥️

## Laborátório 🥼

## Objetivo

O laboratório demonstrará como:

Implantar uma pilha do AWS CloudFormation com uma nuvem privada virtual (VPC) definida e um grupo de segurança.

Configurar uma pilha do AWS CloudFormation com recursos, como um bucket do Amazon Simple Storage Solution (S3) e o Amazon Elastic Compute Cloud (EC2).

Terminar uma pilha do AWS CloudFormation e seus respectivos recursos.

### Diagrama do fluxo ✅

![initial-template](https://github.com/user-attachments/assets/df64e85f-c8f7-435a-a089-7dcde8294c67)


# Execução 🚀

#### Tarefa 1: Implantar uma Pilha do CloudFormation
1. **Baixar e abrir o modelo**:
   - Baixe o arquivo `task1.yaml`.
   - Abra o arquivo em um editor de texto.

2. **Analisar o modelo**:
   - A seção `Parameters` solicita entradas para a VPC.
   - A seção `Resources` define a infraestrutura (VPC e grupo de segurança).
   - A seção `Outputs` fornece informações sobre os recursos criados.

3. **Criar a pilha no CloudFormation**:
   - No console AWS, navegue até CloudFormation e clique em "Criar pilha".
   - Faça o upload do arquivo `task1.yaml` e prossiga com os passos:
     - Defina o nome da pilha como "Lab".
     - Use os valores padrão para os parâmetros.
   - Revise e crie a pilha.

4. **Monitorar o processo**:
   - Acompanhe as guias "Eventos" e "Recursos" para ver as atividades e recursos sendo criados.
   - Aguarde até que o status mude para `CREATE_COMPLETE`.

#### Tarefa 2: Adicionar um Bucket do Amazon S3 à Pilha
1. **Editar o modelo**:
   - Abra o arquivo `task1.yaml` e adicione a definição do bucket S3 na seção `Resources`:
     ```yaml
     MyBucket:
       Type: AWS::S3::Bucket
     ```

2. **Atualizar a pilha**:
   - No console CloudFormation, selecione a pilha "Lab" e clique em "Atualizar".
   - Substitua o modelo atual pelo arquivo atualizado.
   - Prossiga com os passos para revisar e atualizar a pilha.

3. **Verificar o novo recurso**:
   - Acompanhe o status da pilha até ver `UPDATE_COMPLETE`.
   - Verifique a criação do bucket na guia "Recursos" e opcionalmente no console do S3.

#### Tarefa 3: Adicionar uma Instância do Amazon EC2 à Pilha
1. **Adicionar o parâmetro AMI**:
   - Adicione estas linhas na seção `Parameters` do modelo:
     ```yaml
     AmazonLinuxAMIID:
       Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
       Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2
     ```

2. **Definir a instância EC2**:
   - Adicione a definição na seção `Resources`:
     ```yaml
     AppInstance:
       Type: AWS::EC2::Instance
       Properties:
         ImageId: !Ref AmazonLinuxAMIID
         InstanceType: t3.micro
         SecurityGroupIds:
           - !Ref AppSecurityGroup
         SubnetId: !Ref PublicSubnet
         Tags:
           - Key: Name
             Value: App Server
     ```

3. **Atualizar a pilha**:
   - Salve o arquivo atualizado e use o console CloudFormation para atualizar a pilha.
   - Monitorar até o status `UPDATE_COMPLETE`.

4. **Verificar a nova instância**:
   - Verifique a criação da instância na guia "Recursos" e opcionalmente no console do EC2.

Essas etapas ajudam a criar e gerenciar a infraestrutura na AWS de forma eficiente utilizando o CloudFormation! 🚀

Código gerado:

AWSTemplateFormatVersion: 2010-09-09
Description: Lab template

# Lab VPC with public subnet and Internet Gateway

Parameters:

  LabVpcCidr:
    Type: String
    Default: 10.0.0.0/20

  PublicSubnetCidr:
    Type: String
    Default: 10.0.0.0/24

  AmazonLinuxAMIID:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2

Resources:

###########
# Mybucket
###########

  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: meu-bucket-simples-515582

###########
# VPC with Internet Gateway
###########

  LabVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref LabVpcCidr
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: Lab VPC

  IGW:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: Lab IGW

  VPCtoIGWConnection:
    Type: AWS::EC2::VPCGatewayAttachment
    DependsOn:
      - IGW
      - LabVPC
    Properties:
      InternetGatewayId: !Ref IGW
      VpcId: !Ref LabVPC

###########
# Public Route Table
###########

  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    DependsOn: LabVPC
    Properties:
      VpcId: !Ref LabVPC
      Tags:
        - Key: Name
          Value: Public Route Table

  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn:
      - PublicRouteTable
      - IGW
    Properties:
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref IGW
      RouteTableId: !Ref PublicRouteTable

###########
# Public Subnet
###########

  PublicSubnet:
    Type: AWS::EC2::Subnet
    DependsOn: LabVPC
    Properties:
      VpcId: !Ref LabVPC
      MapPublicIpOnLaunch: true
      CidrBlock: !Ref PublicSubnetCidr
      AvailabilityZone: !Select 
        - 0
        - !GetAZs 
          Ref: AWS::Region
      Tags:
        - Key: Name
          Value: Public Subnet

  PublicRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    DependsOn:
      - PublicRouteTable
      - PublicSubnet
    Properties:
      RouteTableId: !Ref PublicRouteTable
      SubnetId: !Ref PublicSubnet

###########
# App Security Group
###########

  AppSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    DependsOn: LabVPC
    Properties:
      GroupName: App
      GroupDescription: Enable access to App
      VpcId: !Ref LabVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: App

###########
# EC2 Instance
###########

  AppInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref AmazonLinuxAMIID
      InstanceType: t3.micro
      SecurityGroupIds:
        - !Ref AppSecurityGroup
      SubnetId: !Ref PublicSubnet
      Tags:
        - Key: Name
          Value: App Server

###########
# Outputs
###########

Outputs:

  LabVPCDefaultSecurityGroup:
    Value: !Sub ${LabVPC.DefaultSecurityGroup}

