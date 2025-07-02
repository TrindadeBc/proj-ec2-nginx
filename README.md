# Projeto DevSecOps – Infraestrutura Manual AWS


Este projeto demonstra a construção de uma infraestrutura básica na AWS **sem uso de IaC**, feita manualmente pelo console. O foco é o aprendizado de redes, segurança, web server e monitoramento com boas práticas de documentação e versionamento.

---

## Etapa 1: Criação da VPC

A VPC foi criada manualmente no console AWS utilizando a opção **"VPC only"** para garantir controle total e documentar cada etapa da arquitetura em nuvem. Essa escolha nos permite adicionar manualmente todos os componentes como sub-redes, roteadores e gateways – prática ideal em ambientes educacionais, versionados e com ou sem foco em IaC (*Infrastructure as Code*).

- **CIDR IPv4:** `10.0.0.0/24`
- **IPv6:** Não utilizado
- **Tenancy:** Default
- **Tag:** `Name = devsecops-vpc`

📸 **Imagem de referência:**  
![Tela de criação da VPC](images/capturas/1.png)

---

## Etapa 2: Criação das Sub-redes

### 🔹 Sub-rede Pública A

A primeira sub-rede pública foi criada manualmente no console AWS:

- **Nome:** `public-subnet-a`
- **Zona de disponibilidade:** `us-east-2a`
- **Bloco CIDR IPv4:** `10.0.0.0/26` (64 IPs)
- **Tag:** `Name = public-subnet-a`

📸 **Imagem de referência:**  
![Criação da sub-rede pública A](images/capturas/2.png)

---

### 🔹 Demais sub-redes

Usamos **Add new subnet** para criar todas em sequência:

- **public-subnet-b** (us-east-2b | 10.0.0.64/26)  
- **private-subnet-a** (us-east-2a | 10.0.0.128/26)  
- **private-subnet-b** (us-east-2b | 10.0.0.192/26)  

📸 **Visão geral das 4 sub-redes:**  
![Visão geral das sub-redes](images/capturas/3.png)

---

## Etapa 3: Internet Gateway & Route Tables

### 🔸 3.1 Criar e anexar Internet Gateway

1. **Create Internet Gateway**  
   - **Name tag:** `devsecops-igw`  
   📸 **Imagem de referência:**  
   ![Create Internet Gateway](images/capturas/4.png)

2. **Attach to VPC**  
   - Selecione **devsecops-igw** → **Actions** → **Attach to VPC** → **devsecops-vpc**  
   📸 **Imagem de referência:**  
   ![Attach IGW à VPC](images/capturas/5.png)

---

### 🔸 3.2 Criar Route Table pública

1. **Create Route Table**  
   - **Name tag:** `public-route-table`  
   - **VPC:** `devsecops-vpc`  
   📸 **Imagem de referência:**  
   ![Create Route Table](images/capturas/6.png)

2. **Adicionar rota 0.0.0.0/0 → IGW**  
   - **Edit routes** → **Add route**  
     - **Destination:** `0.0.0.0/0`  
     - **Target:** `devsecops-igw`  
   📸 **Imagem de referência:**  
   ![Adicionar rota à RT](images/capturas/7.png)

3. **Associar sub-redes públicas**  
   - **Edit subnet associations** → marque **public-subnet-a** e **public-subnet-b** → **Save associations**  
   📸 **Imagem de referência:**  
   ![Associar sub-redes à RT](images/capturas/8.png)

---

## Etapa 4: Lançar EC2 e Configurar Nginx

### 🔹 4.1 Name and Tags

Ao lançar a instância, adicionamos três tags essenciais para rastreamento de custo e organização:

- **Name:** `PB - JUN - 2025`
- **CostCenter:** `C092000024`
- **Project:** `PB - JUN - 2025`

📸 **Imagem de referência:**  
![Name and Tags](images/capturas/16.png)

---

### 🔹 4.2 Escolher AMI

Selecionamos a **Amazon Linux 2023** pela estabilidade, performance e suporte de longo prazo:

- **AMI ID:** `ami-0c803b171269e2d72`
- **Architecture:** 64-bit (x86)
- **Boot mode:** uefi-preferred
- **Username padrão:** `ec2-user`

📸 **Imagem de referência:**  
![Application and OS Images](images/capturas/17.png)

---

### 🔹 4.3 Instance Type & Key Pair

- **Instance type:** `t2.micro` (free-tier, 1 vCPU / 1 GiB RAM)  
- **Key pair:** Criamos uma nova, **DevSecOps-web-key** (`.pem`) para acesso SSH seguro.

📸 **Imagem de referência (create key pair):**  
![Create key pair](images/capturas/10.png)

📸 **Imagem de referência (instance type & selected key):**  
![Instance type e Key pair](images/capturas/11.png)

---

### 🔹 4.4 Network settings

- **VPC:** `devsecops-vpc`  
- **Subnet:** `public-subnet-a` (acesso público)  
- **Auto-assign public IP:** Enabled  
- **Security Group (create):** `devsecops-web-SG`  
  - **Regra SSH:** TCP 22 de **My IP**  
  - **Regra HTTP:** TCP 80 de **Anywhere (0.0.0.0/0)**

📸 **Imagem de referência:**  
![Network settings](images/capturas/12.png)

---

### 🔹 4.5 Storage

Mantemos o root EBS padrão:

- **Volume 1:** 8 GiB, gp3 (SSD)  
- **Encrypted:** Não  
- **IOPS:** 3 000

📸 **Imagem de referência:**  
![Configure storage](images/capturas/13.png)

---

### 🔹 4.6 Advanced details

#### Metadata

- **Metadata accessible:** Enabled  
- **Metadata version:** V2 only (token required)  
- **Hop limit:** 2  

📸 **Imagem de referência:**  
![Metadata settings](images/capturas/14.png)

#### User data (Cloud-Init)

Injetamos um script para instalar e iniciar o Nginx automaticamente, e já servir uma página HTML customizada:

```bash
#!/bin/bash
yum update -y
amazon-linux-extras enable nginx1
yum install -y nginx
systemctl enable nginx
systemctl start nginx

echo "<h1>Hello from DevSecOps EC2 with NGINX</h1>" > /usr/share/nginx/html/index.html

