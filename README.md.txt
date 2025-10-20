# 🏗️ AWS CloudFormation - Infraestructura como Código (IaC)


> Proyecto de infraestructura modular en AWS utilizando CloudFormation para despliegue automatizado y repetible.

## 📋 Tabla de Contenidos

- [Descripción](#-descripción)
- [Arquitectura](#️-arquitectura)
- [Componentes](#️-componentes)
- [Prerequisitos](#-prerequisitos)
- [Guía de Despliegue](#-guía-de-despliegue)
- [Pruebas y Validación](#-pruebas-y-validación)
- [Costos Estimados](#-costos-estimados)
- [Troubleshooting](#-troubleshooting)
- [Lecciones Aprendidas](#-lecciones-aprendidas)
- [Limpieza de Recursos](#️-limpieza-de-recursos)
- [Autor](#-autor)

---

## 🎯 Descripción

Este proyecto demuestra el uso de **Infrastructure as Code (IaC)** mediante AWS CloudFormation para crear una arquitectura de aplicación web escalable y modular. La infraestructura está dividida en componentes independientes que se pueden desplegar y gestionar de forma separada.

### Objetivos del Proyecto

- ✅ Implementar infraestructura como código usando CloudFormation
- ✅ Crear arquitectura modular y reutilizable
- ✅ Demostrar integración entre servicios AWS (EC2, RDS, VPC)
- ✅ Aplicar mejores prácticas de seguridad (Security Groups, subnets privadas)
- ✅ Documentar proceso completo para portafolio profesional

---

## 🏗️ Arquitectura

```
┌─────────────────────────────────────────────────────────────┐
│                          VPC (10.0.0.0/16)                  │
│                                                             │
│  ┌─────────────────────┐      ┌─────────────────────┐     │
│  │  Public Subnet 1    │      │  Public Subnet 2    │     │
│  │   (10.0.1.0/24)     │      │   (10.0.2.0/24)     │     │
│  │                     │      │                     │     │
│  │  ┌──────────────┐   │      │                     │     │
│  │  │  EC2 Instance│   │      │                     │     │
│  │  │  (Web Server)│   │      │                     │     │
│  │  └──────┬───────┘   │      │                     │     │
│  └─────────┼───────────┘      └─────────────────────┘     │
│            │                                               │
│  ┌─────────┼───────────┐      ┌─────────────────────┐     │
│  │         ↓           │      │                     │     │
│  │  Private Subnet 1   │      │  Private Subnet 2   │     │
│  │   (10.0.11.0/24)    │      │   (10.0.12.0/24)    │     │
│  │                     │      │                     │     │
│  │  ┌──────────────────────────────────────┐       │     │
│  │  │     RDS MySQL Database               │       │     │
│  │  │     (Multi-AZ Subnet Group)          │       │     │
│  │  └──────────────────────────────────────┘       │     │
│  └─────────────────────┘      └─────────────────────┘     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
                            │
                            ↓
                     Internet Gateway
```

### Flujo de Datos

1. Usuario accede a la instancia EC2 vía HTTP (puerto 80)
2. EC2 se conecta a RDS MySQL en subnet privada (puerto 3306)
3. RDS responde con datos solicitados
4. EC2 renderiza y devuelve respuesta al usuario

---

## 🛠️ Componentes

### 1. Networking Stack (VPC)

**Archivo:** `01-networking/vpc-template.yaml`

- **VPC** con CIDR 10.0.0.0/16
- **2 Subnets Públicas** (AZ diferentes para HA)
- **2 Subnets Privadas** (para recursos de base de datos)
- **Internet Gateway** para acceso público
- **Route Tables** configuradas correctamente
- **DNS Hostnames** habilitado

### 2. Compute Stack (EC2)

**Archivo:** `02-compute/ec2-webserver-template.yaml`

- **Instancia EC2** t2.micro (Free Tier eligible)
- **AMI** Amazon Linux 2 (latest)
- **Apache Web Server** instalado automáticamente vía UserData
- **Security Group** con reglas:
  - HTTP (80) abierto a internet
  - SSH (22) para administración
- **Elastic IP** asignada automáticamente
- **IAM Instance Profile** (LabInstanceProfile)

### 3. Database Stack (RDS)

**Archivo:** `03-database/rds-mysql-template.yaml`

- **RDS MySQL 8.0** (versión latest)
- **Clase de instancia** db.t3.micro
- **20 GB** de almacenamiento SSD (gp2)
- **Subnet Group** en subnets privadas
- **Security Group** permite solo tráfico desde VPC
- **Backups automáticos** (7 días de retención)
- **No públicamente accesible** (seguridad)

---

## 📦 Prerequisitos

Antes de comenzar, asegúrate de tener:

- ✅ Conocimientos básicos de:
  - CloudFormation
  - Redes (VPC, Subnets, Security Groups)
  - EC2 y RDS

### Restricciones del Laboratorio

Este proyecto fue desarrollado en un ambiente de laboratorio con las siguientes restricciones:

- Instancias EC2: Solo t2/t3 (nano a medium)
- RDS: Solo db.t3.micro a db.t3.medium
- Máximo 9 instancias simultáneas
- Storage máximo: 100 GB
- Usar LabRole para permisos IAM

---

## 🚀 Guía de Despliegue

### Paso 1: Crear Key Pair

```bash
# Desde AWS Console:
1. Ve a EC2 → Key Pairs
2. Click "Create key pair"
3. Name: lab-key
4. Type: RSA
5. Format: .pem (Linux/Mac) o .ppk (Windows)
6. Click "Create key pair" y descarga
```

### Paso 2: Desplegar VPC Stack

```bash
# En CloudFormation Console:
1. Click "Create stack" → "With new resources"
2. Upload template: 01-networking/vpc-template.yaml
3. Stack name: Lab-VPC
4. Parameters:
   - EnvironmentName: Lab-VPC (default)
5. Next → Next → Create stack
6. Espera 3-5 minutos hasta CREATE_COMPLETE
```

**Recursos creados:**
- 1 VPC
- 4 Subnets (2 públicas, 2 privadas)
- 1 Internet Gateway
- 2 Route Tables
- Route Table Associations

### Paso 3: Desplegar EC2 Stack

```bash
# En CloudFormation Console:
1. Create stack → Upload: 02-compute/ec2-webserver-template.yaml
2. Stack name: Lab-EC2
3. Parameters:
   - KeyName: lab-key (selecciona tu key)
   - VPCStackName: Lab-VPC
   - LatestAmiId: (usa default)
4. Create stack
5. Espera 5-7 minutos hasta CREATE_COMPLETE
```

**Verificación:**
```bash
# Obtén la IP pública desde Outputs tab
# Abre en navegador: http://<PUBLIC-IP>
# Deberías ver: "Servidor Web en CloudFormation"
```

### Paso 4: Desplegar RDS Stack

```bash
# En CloudFormation Console:
1. Create stack → Upload: 03-database/rds-mysql-template.yaml
2. Stack name: Lab-RDS
3. Parameters:
   - VPCStackName: Lab-VPC
   - DBUsername: admin
   - DBPassword: [tu-password-seguro]  ⚠️ GUARDA este password
4. Create stack
5. Espera 10-15 minutos hasta CREATE_COMPLETE (RDS tarda más)
```

**⚠️ IMPORTANTE:** Guarda el password en un lugar seguro, lo necesitarás para conectarte.

### Paso 5: Verificar Integración EC2 → RDS

```bash
# Conéctate a EC2 vía SSH:
ssh -i lab-key.pem ec2-user@<EC2-PUBLIC-IP>

# Instala cliente MySQL:
sudo yum install mysql -y

# Obtén DBEndpoint desde CloudFormation Outputs del stack Lab-RDS
# Ejemplo: lab-mysql-db.abc123xyz.us-east-1.rds.amazonaws.com

# Conéctate a RDS:
mysql -h <DB-ENDPOINT> -u admin -p

# Ingresa password cuando lo solicite
# Si conecta exitosamente, verás: MySQL [(none)]>

# Prueba comandos:
SHOW DATABASES;
USE labdatabase;
CREATE TABLE test (id INT, name VARCHAR(50));
INSERT INTO test VALUES (1, 'Prueba exitosa');
SELECT * FROM test;
EXIT;
```

---

## ✅ Pruebas y Validación

### Tests Realizados

| Test | Componente | Estado |
|------|-----------|--------|-----------|
| VPC creada correctamente | Networking | ✅ | 
| Subnets en diferentes AZs | Networking | ✅ |
| Apache respondiendo HTTP | Compute | ✅ |
| RDS instancia disponible | Database | ✅ |
| Conexión EC2 → RDS | Integration | ✅ | 
| Queries SQL funcionando | Integration | ✅ | 
| Security Groups correctos | Security | ✅ | 

### Comandos de Validación

```bash
# Validar templates antes de desplegar:
aws cloudformation validate-template \
    --template-body file://01-networking/vpc-template.yaml

# Ver outputs de un stack:
aws cloudformation describe-stacks \
    --stack-name Lab-VPC \
    --query 'Stacks[0].Outputs'

# Ver recursos creados:
aws cloudformation list-stack-resources \
    --stack-name Lab-VPC
```

---

## 💰 Costos Estimados

### Cálculo por Hora

| Servicio | Tipo | Precio/Hora | Cantidad | Subtotal |
|----------|------|-------------|----------|----------|
| VPC/Networking | - | $0.00 | - | $0.00 |
| EC2 | t2.micro | $0.0116 | 1 | $0.0116 |
| RDS | db.t3.micro | $0.017 | 1 | $0.017 |
| EBS | gp2 20GB | ~$0.0003 | 2 | $0.0006 |
| **TOTAL** | - | - | - | **~$0.03/hora** |

### Cálculo Mensual (24/7)

```
$0.03/hora × 24 horas × 30 días = ~$21.60/mes
```

### 💡 Ahorro con Free Tier

Si tu cuenta es elegible para Free Tier:
- EC2 t2.micro: 750 horas/mes GRATIS
- RDS db.t3.micro: 750 horas/mes GRATIS
- 20 GB EBS: GRATIS

**Costo real con Free Tier: ~$0.00/mes** (primeros 12 meses)

### ⚠️ Recomendación

Detén o elimina recursos cuando no los uses:
```bash
# Detener EC2 (mantiene configuración):
aws ec2 stop-instances --instance-ids i-xxxxxxxxx

# Eliminar stack completo (recomendado):
# Sigue las instrucciones de "Limpieza de Recursos"
```

---

## 🔧 Troubleshooting

### Problema 1: Error de versión MySQL

**Error:**
```
Cannot find version 8.0.35 for mysql
```

**Causa:** Versión específica no disponible en la región.

**Solución:**
```yaml
# En rds-mysql-template.yaml, cambiar:
EngineVersion: '8.0.35'  # ❌
# Por:
EngineVersion: '8.0'     # ✅ Usa última versión disponible
```

### Problema 2: RDS no encuentra subnets privadas

**Error:**
```
Value (Lab-VPC-Private-Subnet-1) for parameter cannot be found
```

**Causa:** Template VPC original no incluía subnets privadas.

**Solución:**
1. Actualizar vpc-template.yaml con subnets privadas
2. Agregar exports en Outputs section
3. Actualizar o recrear stack VPC

### Problema 3: No puedo conectarme a RDS desde EC2

**Error:**
```
ERROR 2003: Can't connect to MySQL server
```

**Causas posibles:**
- Security Group de RDS no permite conexiones desde VPC
- EC2 en subnet pública, RDS en privada sin ruta
- Endpoint incorrecto

**Solución:**
```bash
# 1. Verifica Security Group de RDS:
#    Debe permitir MySQL (3306) desde CIDR 10.0.0.0/16

# 2. Verifica endpoint:
#    Copia exactamente desde CloudFormation Outputs

# 3. Prueba conectividad:
telnet <RDS-ENDPOINT> 3306
# Si conecta, verás texto aleatorio (está funcionando)
```
---

## 📚 Lecciones Aprendidas

### 1. Modularidad es clave

**Aprendizaje:** Separar infraestructura en stacks independientes facilita:
- Mantenimiento y actualizaciones
- Troubleshooting de componentes específicos
- Reutilización de templates
- Eliminación selectiva de recursos

### 2. Importancia de Exports/Imports

**Aprendizaje:** CloudFormation Exports permiten:
- Compartir recursos entre stacks
- Mantener dependencias claras
- Evitar hard-coding de IDs

**Ejemplo:**
```yaml
# Stack VPC exporta:
Outputs:
  VPCId:
    Export:
      Name: !Sub '${EnvironmentName}-VPC-ID'

# Stack EC2 importa:
VpcId:
  Fn::ImportValue: !Sub '${VPCStackName}-VPC-ID'
```

### 3. Versionado flexible vs específico

**Aprendizaje:** Para MySQL EngineVersion:
- `'8.0.35'` → Versión específica (puede fallar)
- `'8.0'` → Versión flexible (más confiable)

**Recomendación:** Usa versiones flexibles en templates reutilizables.

### 4. Seguridad por capas

**Aprendizaje:** Implementar defensa en profundidad:
- RDS en subnets **privadas** (sin acceso internet)
- Security Groups con **mínimo privilegio**
- EC2 como **bastion** para acceso a RDS
- Passwords vía **parámetros NoEcho**

### 5. Orden de despliegue importa

**Aprendizaje:** Respetar dependencias:
1. Primero: VPC (foundation)
2. Segundo: EC2, RDS (dependen de VPC)
3. Eliminación: Orden inverso

### 6. UserData para automatización

**Aprendizaje:** UserData permite:
- Instalar software al lanzar instancia
- Configurar servicios automáticamente
- Reducir pasos manuales post-despliegue

**Ejemplo útil:**
```yaml
UserData:
  Fn::Base64: !Sub |
    #!/bin/bash
    yum update -y
    yum install -y httpd mysql
    systemctl start httpd
    echo "Server ready!" > /var/www/html/index.html
```

---

## 🗑️ Limpieza de Recursos

**⚠️ IMPORTANTE:** Eliminar en orden inverso al despliegue.

### Paso 1: Eliminar RDS Stack

```bash
# CloudFormation Console:
1. Selecciona stack "Lab-RDS"
2. Click "Delete"
3. Confirm
4. Espera 10-15 minutos (RDS tarda en eliminar)
```

### Paso 2: Eliminar EC2 Stack

```bash
1. Selecciona stack "Lab-EC2"
2. Click "Delete"
3. Confirm
4. Espera 3-5 minutos
```

### Paso 3: Eliminar VPC Stack

```bash
1. Selecciona stack "Lab-VPC"
2. Click "Delete"
3. Confirm
4. Espera 2-3 minutos
```

### Verificación Final

```bash
# Verifica que no queden recursos:
1. EC2 → Instances (debe estar vacío)
2. RDS → Databases (debe estar vacío)
3. VPC → Your VPCs (debe tener solo default VPC)
4. CloudFormation → Stacks (filtrar por "Deleted")
```

---

## 👨‍💻 Autor

**Paulo Gutierrez Carrillo**
- 🎓 AWS Cloud Practitioner (en preparación)
- 💼 LinkedIn: (https://www.linkedin.com/in/paulo-gutierrez-a7a832237/)
- 🐙 GitHub: [Paulo-Gutierrez-cloud](https://github.com/Paulo-Gutierrez-cloud)
- 📧 Email: 3gp200@gmail.com

---

## 📄 Licencia

Este proyecto está bajo la Licencia MIT - ver el archivo [LICENSE](LICENSE) para detalles.

---

## 🙏 Agradecimientos

- AWS Documentation
- CloudFormation Best Practices Guide
- Comunidad de AWS en español

---

## 📌 Notas Adicionales

Este proyecto fue desarrollado como parte de la preparación para la certificación **AWS Certified Cloud Practitioner**. El objetivo es demostrar conocimientos prácticos en:

- ✅ Infrastructure as Code (IaC)
- ✅ Servicios fundamentales de AWS
- ✅ Mejores prácticas de arquitectura
- ✅ Seguridad en la nube
- ✅ Gestión de costos

---

**⭐ Si este proyecto te fue útil, considera darle una estrella en GitHub!**

**🔗 Links Útiles:**
- [AWS CloudFormation Documentation](https://docs.aws.amazon.com/cloudformation/)
- [AWS Free Tier](https://aws.amazon.com/free/)
- [AWS Architecture Center](https://aws.amazon.com/architecture/)