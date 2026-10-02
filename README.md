# ☁️ AWS Storage Management & Backup Automation (EBS & S3)

![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Python](https://img.shields.io/badge/Python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)

Projeto prático focado no gerenciamento, automação de backups e ciclo de vida de armazenamento na **Amazon Web Services (AWS)** utilizando **EBS Snapshots**, **Amazon S3 Sync**, **S3 Versioning** e automação com **Python (Boto3)**.

---

## 🎯 Objetivos do Projeto

- Gerenciar backups de volumes **Amazon EBS** via AWS CLI.
- Automatizar a criação recorrente de snapshots utilizando **Cron Jobs** no Linux.
- Implementar política de retenção automatizada de snapshots com **Python (Boto3)**, mantendo apenas as versões mais recentes para redução de custos.
- Sincronizar diretórios locais com **Amazon S3** (`aws s3 sync`) com remoção espelhada (`--delete`).
- Garantir resiliência contra exclusão acidental através do **S3 Versioning** e recuperação de arquivos via AWS CLI.

---

## 🛠️ Tecnologias e Ferramentas

- **Amazon EC2**: Instâncias Linux (`Command Host` e `Processor`).
- **Amazon EBS**: Gerenciamento e captura de snapshots de volumes de blocos.
- **Amazon S3**: Armazenamento de objetos, sincronização e versionamento.
- **AWS IAM**: Configuração de Roles e Instance Profiles com princípio de menor privilégio.
- **AWS CLI**: Interface de linha de comando para automação AWS.
- **Python 3.8 & Boto3**: Scripting para gerenciamento de recursos AWS.
- **Linux Crontab**: Agendamento automático de tarefas de backup.

---

## 📐 Arquitetura da Solução

                +------------------------------------+
                |        Amazon S3 Bucket            |
                |   (Versioning & Remote Backup)     |
                +-----------------+------------------+
                                  ^
                                  | (S3 Sync & Version Recovery)
                                  v
+-------------------+       +---------+----------+
|   Command Host    | ----> | Processor Instance |
|  (Admin & Cron)   |       |   (EBS Volume)     |
+-------------------+       +--------------------+
|                            |
+--> Snapshots (Boto3) <-----+

---

## 🚀 Passos Executados

### 1. Configuração de Segurança e Recursos Inicial
- Criação de um Bucket S3 globalmente exclusivo (`lab-storage-thais-farias-99`).
- Associação de IAM Role (`S3BucketAccess`) à instância EC2 `Processor` para permitir acesso seguro aos serviços S3 e EBS via CLI sem hardcode de credenciais.

### 2. Automação de Snapshots EBS e Política de Retenção
- Interrupção consistente da instância `Processor` para criação do snapshot base limpo do volume EBS.
- Configuração de **Cron Job** para disparar a criação de snapshots a cada minuto:
  ```bash
  * * * * * aws ec2 create-snapshot --volume-id vol-0962bdb149f28fad6 2>&1 >> /tmp/cronlog

- Execução do script Python (snapshotter_v2.py) utilizando a biblioteca Boto3 para ordenar os snapshots por data de criação e remover os excedentes, retendo rigidamente apenas os 2 snapshots mais recentes.

### 3. Sincronização S3 e Recuperação com Versionamento
- Ativação do versionamento no bucket S3:

  aws s3api put-bucket-versioning --bucket lab-storage-thais-farias-99 --versioning-configuration Status=Enabled

  - Sincronização da pasta local de arquivos para o S3:
 
  aws s3 sync files s3://lab-storage-thais-farias-99/files/
  
  - Simulação de exclusão acidental local e sincronização da remoção no S3 com a flag --delete.
  
