
# 🛠️ Projeto DevSecOps – Infraestrutura Manual AWS

Este projeto demonstra a construção de uma infraestrutura básica na AWS **sem uso de IaC**, feita manualmente pelo console. O foco é o aprendizado de redes, segurança, web server e monitoramento com boas práticas de documentação e versionamento.

---

## Etapa 1: Criação da VPC

A VPC foi criada manualmente no console AWS utilizando a opção **"VPC only"** para garantir controle total e documentar cada etapa da arquitetura em nuvem. Essa escolha nos permite adicionar manualmente todos os componentes como sub-redes, roteadores e gateways – prática ideal em ambientes educacionais, versionados e com ou sem foco em IaC (*Infrastructure as Code*).

- **CIDR IPv4:** `10.0.0.0/24`  
- **IPv6:** Não utilizado  
- **Tenancy:** Default  
- **Tag:** `Name = devsecops-vpc`  

📸 **Imagem de referência:**  
`/images/capturas/1.png` – Tela de criação da VPC no console AWS.

---

## Etapa 2: Criação das Sub-redes

### 🔹 Sub-rede Pública A

A primeira sub-rede pública foi criada manualmente no console AWS:

- **Nome:** `public-subnet-a`  
- **Zona de disponibilidade:** `us-east-2a`  
- **Bloco CIDR IPv4:** `10.0.0.0/26` (64 IPs disponíveis)  
- **Tag:** `Name = public-subnet-a`  

📸 **Imagem de referência:**  
`/images/capturas/2.png` – Tela de criação da sub-rede pública A.

> Será usada futuramente para hospedar a EC2 acessível.

---

### 🔹 Demais sub-redes

Usamos **“Add new subnet”** para criar em sequência:

- **public-subnet-b** (us-east-2b | 10.0.0.64/26)  
- **private-subnet-a** (us-east-2a | 10.0.0.128/26)  
- **private-subnet-b** (us-east-2b | 10.0.0.192/26)  

📸 **Imagem de referência:**  
`/images/capturas/3.png` – Visão geral das 4 sub-redes criadas.

---

## Etapa 3: Internet Gateway & Route Tables

### 🔸 3.1 Criar e anexar Internet Gateway

1. **Create Internet Gateway**  
   - Nome: `devsecops-igw`  
   - Print: `/images/capturas/4.png`  

2. **Attach to VPC**  
   - Selecione `devsecops-igw` → **Actions** → **Attach to VPC** → `devsecops-vpc`  
   - Print: `/images/capturas/5.png`

---

### 🔸 3.2 Criar Route Table pública

1. **Create Route Table**  
   - Nome: `public-route-table`  
   - VPC: `devsecops-vpc`  
   - Print: `/images/capturas/6.png`

2. **Adicionar rota 0.0.0.0/0 → IGW**  
   - **Edit routes** → **Add route**  
     - Destination: `0.0.0.0/0`  
     - Target: `devsecops-igw`  
   - Print: `/images/capturas/7.png`

3. **Associar sub-redes públicas**  
   - **Edit subnet associations** → marque `public-subnet-a` e `public-subnet-b`  
   - Print (desassociando primeiro as privadas): `/images/capturas/8.png`  
   - Print (associando as públicas): `/images/capturas/9.png`

---

## Etapa 4: Provisionamento da Instância EC2 com NGINX

### 🔹 4.1 Name and Tags

- **Tags obrigatórias**:

  | Key         | Value           | Resource Types        |
  |-------------|------------------|-----------------------|
  | Name        | PB - JUN - 2025 | Instances, Volumes    |
  | CostCenter  | C092000024      | Instances, Volumes    |
  | Project     | PB - JUN - 2025 | Instances, Volumes    |

📸 `/images/capturas/10.png`

---

### 🔹 4.2 Seleção da AMI

- **Amazon Linux 2023** (kernel-6.1, HVM, 64-bit)  
- **AMI ID:** `ami-0c803b171269e2d72`  
- Print: `/images/capturas/11.png`

---

### 🔹 4.3 Tipo de Instância & Key Pair

- **Instance type:** `t2.micro` (Free Tier)  
- **Key pair:** `DevSecOps-web-key` (RSA/.pem)  
- Print: `/images/capturas/12.png`

---

### 🔹 4.4 Network Settings & Security Group

- **VPC:** `devsecops-vpc`  
- **Subnet:** `public-subnet-a`  
- **Auto-assign public IP:** **Enable**  
- **Security group:** Criado novo `devsecops-web-SG`  
  - **Inbound:**
    - SSH (22) → **My IP**  
    - HTTP (80) → **Anywhere (0.0.0.0/0)**  
- Print: `/images/capturas/13.png`

---

### 🔹 4.5 Configure Storage

- **Root volume (gp3):** 8 GiB, 3 000 IOPS, Delete on Termination ✔️  
- Print: `/images/capturas/14.png`

---

### 🔹 4.6 Advanced Details

#### Metadata
- **Metadata version:** V2 only (token required)  
- **Metadata accessible:** Enabled  
- Print: `/images/capturas/15.png`

#### User Data
```bash
#!/bin/bash
yum update -y
amazon-linux-extras enable nginx1
yum install -y nginx
systemctl enable nginx
systemctl start nginx
echo "<h1>Hello from DevSecOps EC2 with NGINX</h1>" \
  > /usr/share/nginx/html/index.html

