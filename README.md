# Proposta de Arquitetura de Software para o Sistema de Telemedicina da Health&Med

## Visão Geral

A Health&Med está desenvolvendo um sistema proprietário para telemedicina, substituindo soluções terceirizadas, visando maior qualidade, segurança dos dados dos pacientes e redução de custos. Este documento detalha a arquitetura do software com foco em arquitetos de software e desenvolvedores, fornecendo uma visão técnica abrangente para facilitar a implementação pelo time de desenvolvimento. O backend será implementado em .NET devido à familiaridade da equipe de desenvolvimento com essa tecnologia.

## Requisitos Funcionais

1. **Autenticação do Usuário (Médico)**
   - Login usando o número de CRM e senha.

2. **Cadastro/Edição de Horários Disponíveis (Médico)**
   - Cadastro e edição de horários disponíveis para consultas.

3. **Aceite ou Recusa de Consultas Médicas (Médico)**
   - Aceitação ou recusa de consultas agendadas.

4. **Autenticação do Usuário (Paciente)**
   - Login usando e-mail, CPF e senha.

5. **Busca por Médicos (Paciente)**
   - Visualização de médicos disponíveis com filtros por especialidade, distância e avaliação.

6. **Agendamento de Consultas (Paciente)**
   - Visualização da agenda do médico e agendamento de consultas.
   - Cancelamento de consultas com justificativa.

7. **Teleconsulta**
   - Criação de link de reunião online para a consulta agendada.

8. **Prontuário Eletrônico**
   - Acesso e upload de documentos médicos pelo paciente.
   - Compartilhamento de prontuário com médicos, controlando acesso e duração.

## Requisitos Não Funcionais

1. **Alta Disponibilidade**
   - Sistema disponível 24/7.

2. **Escalabilidade**
   - Suporte para até 20.000 usuários simultâneos.

3. **Segurança**
   - Alta segurança para prontuário eletrônico.
   - Proteção de dados sensíveis conforme melhores práticas.

## Arquitetura da Solução

### Diagrama da Arquitetura

```plaintext
                +---------------------+
                |      Frontend       |
                +---------------------+
                          |
                          v
                +---------------------+
                |    API Gateway      |
                +---------------------+
                          |
+-----------+---------+---------+-----------+
|           |         |         |           |
v           v         v         v           v
+-----------+---------+---------+-----------+
|  Auth     |  Medico |  Paciente |  Consulta  |  Prontuario  |
| Service   | Service | Service |  Service  |  Service  |
+-----------+---------+---------+-----------+
      |           |         |         |           |
      v           v         v         v           v
+-----------+---------+---------+-----------+
|          Database Services                |
|   (RDS, DynamoDB, S3)                     |
+-------------------------------------------+
                          |
                          v
                +---------------------+
                |       S3 Bucket     |
                +---------------------+
                          |
                          v
                +---------------------+
                |     CloudFront      |
                +---------------------+
```

### Descrição dos Componentes

#### Frontend

- **Tecnologia**: React.js
- **Funcionalidades**:
  - Interface responsiva para médicos e pacientes.
  - Integração com API Gateway para comunicação com os serviços backend.
- **Justificativa**:
  - React.js é uma biblioteca de frontend amplamente utilizada que permite a construção de interfaces de usuário interativas e dinâmicas. Sua popularidade e comunidade ativa garantem uma vasta gama de recursos e suporte.

#### API Gateway

- **Tecnologia**: AWS API Gateway
- **Funcionalidades**:
  - Roteamento de requisições para os microsserviços.
  - Autenticação, autorização e monitoramento.
  - Rate limiting para controle de tráfego.
- **Justificativa**:
  - O AWS API Gateway facilita a criação, publicação, manutenção, monitoramento e proteção de APIs em qualquer escala. Ele também oferece integrações nativas com outros serviços da AWS, o que simplifica a arquitetura.

### Microsserviços

Os microsserviços serão implementados em .NET, cada um responsável por uma funcionalidade específica. Cada serviço será containerizado usando Docker e orquestrado pelo Amazon ECS com AWS Fargate. A comunicação entre microsserviços será feita via HTTP/HTTPS através do API Gateway ou diretamente utilizando o AWS App Mesh para permitir a comunicação de serviço a serviço, com controle de tráfego, resiliência e segurança.

#### Auth Service

- **Tecnologia**: .NET, AWS Cognito
- **Funcionalidades**:
  - Gerenciamento de usuários (login, logout, recuperação de senha).
  - Emissão e validação de tokens JWT.
- **Endpoints**:
  - `POST /auth/login`
  - `POST /auth/logout`
  - `POST /auth/recover`
- **Implementação**:
  - Utilização de AWS Cognito para gerenciamento de usuários.
  - Emissão de tokens JWT para autenticação nas chamadas subsequentes.
- **Justificativa**:
  - AWS Cognito é um serviço de identidade que permite adicionar controle de autenticação, autorização e usuário às suas aplicações com facilidade. Ele se integra bem com outras soluções AWS e oferece suporte para padrões de segurança como OAuth 2.0, SAML e OpenID Connect.

#### Medico Service

- **Tecnologia**: .NET
- **Funcionalidades**:
  - Cadastro e edição de horários disponíveis.
  - Aceitação e recusa de consultas.
- **Endpoints**:
  - `POST /medico/horarios`
  - `PUT /medico/horarios/{id}`
  - `POST /medico/consultas/{id}/aceitar`
  - `POST /medico/consultas/{id}/recusar`
- **Persistência**:
  - Banco de dados RDS para armazenar informações dos médicos.
- **Implementação**:
  - APIs RESTful para interação com o frontend.
  - Persistência de dados em Amazon RDS.
  - Comunicação com o Consulta Service para atualização de status de consultas.
- **Justificativa**:
  - .NET é uma plataforma robusta e madura para o desenvolvimento de aplicações backend. Sua integração com AWS, especialmente com serviços como RDS, facilita a criação de aplicações escaláveis e seguras.

#### Paciente Service

- **Tecnologia**: .NET
- **Funcionalidades**:
  - Cadastro e edição de informações dos pacientes.
  - Busca por médicos e agendamento de consultas.
- **Endpoints**:
  - `POST /paciente`
  - `PUT /paciente/{id}`
  - `GET /medicos`
  - `POST /consultas`
  - `DELETE /consultas/{id}`
- **Persistência**:
  - Banco de dados RDS para informações dos pacientes e consultas.
- **Implementação**:
  - APIs RESTful para interação com o frontend.
  - Integração com serviços de terceiros para validação de CPF.
  - Comunicação com o Medico Service para busca de médicos e consulta de horários.
- **Justificativa**:
  - A escolha pelo .NET permite à equipe aproveitar seu conhecimento existente, reduzindo a curva de aprendizado e acelerando o desenvolvimento.

#### Consulta Service

- **Tecnologia**: .NET
- **Funcionalidades**:
  - Criação de links de reunião online.
  - Gerenciamento de teleconsultas.
- **Endpoints**:
  - `POST /consultas/{id}/link`
- **Implementação**:
  - Integração com AWS Chime para criação de links de reuniões.
  - Armazenamento de links de reuniões em DynamoDB.
  - Comunicação com Medico Service e Paciente Service para atualizações de status de consultas e links de reuniões.
- **Justificativa**:
  - AWS Chime fornece uma solução completa para comunicações, permitindo a integração de funcionalidades de vídeo e voz em aplicações com facilidade.

#### Prontuario Service

- **Tecnologia**: .NET
- **Funcionalidades**:
  - Upload e gerenciamento de documentos médicos.
  - Compartilhamento de documentos com médicos.
- **Endpoints**:
  - `GET /prontuario/{id}`
  - `POST /prontuario/{id}/upload`
  - `POST /prontuario/{id}/compartilhar`
- **Persistência**:
  - Amazon S3 para armazenamento de documentos.
  - Metadados armazenados no DynamoDB.
- **Implementação**:
  - Upload de documentos diretamente para Amazon S3.
  - Armazenamento de metadados de documentos em DynamoDB.
  - Controle de acesso aos documentos via AWS S3 Policies.
  - Comunicação com Paciente Service para gerenciamento de documentos e compartilhamento.
- **Justificativa**:
  - Amazon S3 oferece armazenamento seguro, durável e escalável. Combinado com DynamoDB, que fornece baixa latência para metadados, garante uma solução eficiente para gerenciamento de prontuários eletrônicos.

### Comunicação entre Microsserviços

A comunicação entre os microsserviços será feita através do API Gateway para exposição de APIs públicas e do AWS App Mesh para comunicação direta entre serviços. Abaixo, detalhes de como a comunicação será gerenciada:

1. **API Gateway**:
   - Utilizado para comunicação entre o frontend e os microsserviços.
   - Também utilizado para chamadas públicas entre microsserviços, onde necessário.
- **Justificativa**:
  - API Gateway proporciona um ponto centralizado de gerenciamento e segurança para APIs, simplificando a comunicação

 entre componentes e garantindo uma gestão eficiente de autenticação e autorização.

2. **AWS App Mesh**:
   - Utilizado para comunicação interna entre microsserviços.
   - Proporciona controle de tráfego, resiliência e segurança.
   - Facilita a observabilidade e gerenciamento de rede.
- **Justificativa**:
  - AWS App Mesh permite a implementação de uma malha de serviço, proporcionando visibilidade e controle sobre o tráfego entre microsserviços, essencial para garantir a resiliência e a segurança da aplicação.

### Database Services

1. **Amazon RDS**
   - **Configuração**: Multi-AZ para alta disponibilidade.
   - **Uso**: Armazenamento de dados transacionais (médicos, pacientes, consultas).
   - **Configuração Detalhada**:
     - Configuração de snapshots automáticos para backup.
     - Monitoramento de desempenho usando CloudWatch.
     - Failover automático para instâncias em outra zona de disponibilidade.
- **Justificativa**:
  - Amazon RDS facilita a configuração, operação e escalabilidade de bancos de dados relacionais na nuvem. O suporte Multi-AZ garante alta disponibilidade e recuperação de desastres, essencial para uma aplicação crítica no setor de saúde.

2. **Amazon DynamoDB**
   - **Uso**: Armazenamento de dados de acesso rápido e escalável (metadados de documentos).
   - **Configuração Detalhada**:
     - Tabelas configuradas com chaves primárias compostas para acesso eficiente.
     - Capacidade de leitura e escrita provisionada com auto scaling.
     - Implementação de índices secundários globais para queries avançadas.
- **Justificativa**:
  - DynamoDB oferece uma solução de banco de dados NoSQL totalmente gerenciada e escalável, ideal para aplicações que exigem alta performance em leituras e escritas com baixa latência.

3. **Amazon S3**
   - **Uso**: Armazenamento de documentos médicos do prontuário eletrônico.
   - **Configuração Detalhada**:
     - Buckets configurados com políticas de acesso restrito.
     - Criptografia de dados em repouso usando KMS.
     - Integração com CloudFront para distribuição de documentos.
- **Justificativa**:
  - S3 é altamente durável e escalável, proporcionando armazenamento seguro para grandes volumes de dados, como documentos médicos, com políticas de acesso detalhadas e criptografia.

### CloudFront

- **Uso**: Distribuição de conteúdo estático (frontend) e documentos armazenados no S3.
- **Configuração Detalhada**:
  - Configuração de distribuições para baixa latência e alta disponibilidade.
  - Regras de cache para otimização de desempenho.
  - Integração com AWS WAF para proteção contra ataques DDoS.
- **Justificativa**:
  - CloudFront acelera a entrega de conteúdo ao distribuir globalmente e cachear arquivos próximos aos usuários finais, melhorando a performance da aplicação.

### Segurança

1. **Criptografia**:
   - **TLS**: Criptografia de dados em trânsito.
   - **KMS**: Criptografia de dados em repouso usando AWS Key Management Service.
   - **Configuração Detalhada**:
     - Certificados SSL/TLS gerenciados pelo AWS Certificate Manager.
     - Chaves de criptografia gerenciadas pelo AWS KMS com políticas de rotação.
- **Justificativa**:
  - TLS e KMS garantem que os dados estejam protegidos tanto em trânsito quanto em repouso, atendendo aos requisitos de segurança e conformidade.

2. **Gerenciamento de Identidade e Acesso (IAM)**:
   - **Políticas IAM**: Definição de políticas detalhadas para controle de acesso aos recursos AWS.
   - **MFA**: Autenticação multi-fator para usuários administradores.
   - **Configuração Detalhada**:
     - Políticas de IAM baseadas em funções para controle de acesso granular.
     - Implementação de roles específicas para cada serviço e microsserviço.
     - Configuração de MFA obrigatória para acesso administrativo.
- **Justificativa**:
  - IAM permite a definição de políticas de acesso refinadas, garantindo que os usuários tenham apenas as permissões necessárias para desempenhar suas funções, aumentando a segurança da infraestrutura.

3. **Monitoramento e Logs**:
   - **CloudWatch**: Monitoramento de métricas e logs das aplicações e infraestrutura.
   - **CloudTrail**: Auditoria de ações na conta AWS para rastreabilidade.
   - **Configuração Detalhada**:
     - Dashboards customizáveis para monitoramento em tempo real.
     - Alarmes configurados para alertar sobre problemas de desempenho ou disponibilidade.
     - Integração com AWS Lambda para ações automatizadas em resposta a eventos.
- **Justificativa**:
  - CloudWatch e CloudTrail fornecem visibilidade completa sobre a operação e segurança da aplicação, permitindo resposta rápida a incidentes e conformidade com auditorias.

### Escalabilidade e Alta Disponibilidade

1. **Auto Scaling**:
   - Configuração de auto scaling para serviços ECS com AWS Fargate, garantindo escalabilidade conforme a demanda.
   - **Configuração Detalhada**:
     - Regras de auto scaling baseadas em métricas de CPU, memória e requisições.
     - Políticas de escalabilidade para ajustar a capacidade conforme a demanda.
- **Justificativa**:
  - Auto Scaling assegura que a aplicação pode lidar com variações na carga de trabalho, ajustando automaticamente a capacidade para manter a performance e a disponibilidade.

2. **RDS Multi-AZ**:
   - Banco de dados relacional configurado em Multi-AZ para alta disponibilidade e failover automático.
   - **Configuração Detalhada**:
     - Configuração de réplicas de leitura para distribuição de carga.
     - Implementação de políticas de backup e recuperação.
- **Justificativa**:
  - Multi-AZ proporciona alta disponibilidade e resiliência para o banco de dados, essencial para uma aplicação crítica no setor de saúde.

3. **S3 e CloudFront**:
   - Uso de S3 para armazenamento redundante e CloudFront para entrega rápida de conteúdo.
   - **Configuração Detalhada**:
     - Configuração de políticas de versão para S3.
     - Distribuição global com CloudFront para baixa latência.
- **Justificativa**:
  - A combinação de S3 e CloudFront garante que os dados estejam sempre disponíveis e possam ser acessados rapidamente por usuários em qualquer lugar do mundo.

### Infraestrutura como Código (IaC) com Terraform

#### Provisionamento de Recursos

- **VPC**: Configuração de VPC com subnets públicas e privadas, roteadores e gateways NAT.
- **Security Groups**: Configuração de grupos de segurança para controlar o tráfego de rede.
- **RDS**: Provisionamento de instância RDS Multi-AZ.
- **DynamoDB**: Criação de tabelas DynamoDB.
- **S3**: Criação de buckets S3 com políticas de acesso e criptografia.
- **ECS Cluster**: Configuração de cluster ECS com tarefas e serviços definidos.
- **API Gateway**: Configuração de API Gateway com rotas para os microsserviços.
- **Lambda**: Criação de funções Lambda para tarefas event-driven.
- **Justificativa**:
  - Terraform permite o gerenciamento de infraestrutura como código, facilitando a automação, a consistência e a reprodutibilidade do ambiente de TI.

#### Automação

- **Pipeline de CI/CD**:
  - **CodePipeline**: Orquestração do fluxo de CI/CD.
  - **CodeBuild**: Compilação e testes automatizados dos microsserviços.
  - **CodeDeploy**: Deploy automatizado nos serviços ECS.
- **Justificativa**:
  - Pipelines de CI/CD garantem um fluxo de desenvolvimento eficiente e seguro, reduzindo o tempo de deploy e minimizando erros humanos.

### Demonstração da Infraestrutura na Cloud

A infraestrutura será demonstrada com a aplicação funcionando em um ambiente AWS, incluindo exemplos reais de chamadas de API.

### Demonstração do MVP

A aplicação MVP incluirá as seguintes funcionalidades:

1. **Autenticação do Usuário (Médico)**
   - Login utilizando AWS Cognito.

2. **Cadastro/Edição de Horários Disponíveis (Médico)**
   - Endpoint para cadastro e edição de horários.

3. **Aceite ou Recusa de Consultas Médicas (Médico)**
   - Endpoint para aceite/recusa de consultas.

4. **Autenticação do Usuário (Paciente)**
   - Login utilizando AWS Cognito.

5. **Busca por Médicos (Paciente)**
   - Endpoint para busca de médicos com filtros.

6. **Agendamento de Consultas (Paciente)**
   - Endpoint para agendamento de consultas com visualização de agenda.

## Avaliação

1. **Melhores Práticas de Qualidade e Arquitetura de Software**
   - Princípios SOLID e Clean Code.
   - Arquitetura de microsserviços para alta modularidade e manutenção.

2. **Práticas de Desenvolvimento Seguro**
   - AWS Cognito para autenticação segura.
   - Criptografia de dados sensíveis em trânsito e em repouso.

3. **Documentação Abrangente**
   - Documentação detalhada de todos os componentes, endpoints de API, fluxos de CI/CD e infraestrutura na nuvem.

4. **Automatização da Infraestrutura**
   - Uso de Terraform para provisionamento automatizado de infraestrutura.
   - Pipelines de CI/CD para deploy automatizado e contínuo.

## Conclusão

A arquitetura proposta para o sistema de tele

medicina da Health&Med é robusta, escalável e segura, utilizando as melhores práticas de desenvolvimento de software e infraestrutura. Com uma abordagem de microsserviços, AWS como provedor de infraestrutura e automação completa via Terraform e CI/CD, a Health&Med estará preparada para oferecer um serviço de alta qualidade e confiabilidade para seus usuários.
