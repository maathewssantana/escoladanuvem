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
