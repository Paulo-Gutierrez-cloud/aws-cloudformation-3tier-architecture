🌐 VPC Networking Stack
Stack de red que crea la infraestructura base de networking para la aplicación.
📋 Descripción
Este template de CloudFormation crea una VPC completa con subnets públicas y privadas distribuidas en múltiples zonas de disponibilidad para alta disponibilidad.
🏗️ Recursos Creados
1. VPC

CIDR Block: 10.0.0.0/16
DNS Hostnames: Habilitado
DNS Support: Habilitado

2. Subnets Públicas (2)

Public Subnet 1: 10.0.1.0/24 (AZ-a)
Public Subnet 2: 10.0.2.0/24 (AZ-b)
Auto-assign Public IP: Habilitado

3. Subnets Privadas (2)

Private Subnet 1: 10.0.11.0/24 (AZ-a)
Private Subnet 2: 10.0.12.0/24 (AZ-b)
Auto-assign Public IP: Deshabilitado

4. Internet Gateway

Conectado a la VPC
Permite acceso a internet para subnets públicas

5. Route Tables

Public Route Table: Ruta 0.0.0.0/0 → Internet Gateway
Private Route Table: Solo rutas locales (sin internet)

📊 Diagrama de Arquitectura
VPC (10.0.0.0/16)
│
├── AZ us-east-1a
│   ├── Public Subnet (10.0.1.0/24)
│   │   └── Para: EC2, Load Balancers
│   └── Private Subnet (10.0.11.0/24)
│       └── Para: RDS, Backend services
│
├── AZ us-east-1b
│   ├── Public Subnet (10.0.2.0/24)
│   │   └── Para: EC2, Load Balancers
│   └── Private Subnet (10.0.12.0/24)
│       └── Para: RDS, Backend services
│
└── Internet Gateway
    └── Conectado a Public Subnets
🚀 Despliegue
Usando AWS Console
bash1. Ve a CloudFormation → Create stack
2. Upload template: vpc-template.yaml
3. Parámetros:
   - Stack name: Lab-VPC
   - EnvironmentName: Lab-VPC
4. Next → Next → Create stack
5. Espera 3-5 minutos
Usando AWS CLI
bashaws cloudformation create-stack \
  --stack-name Lab-VPC \
  --template-body file://vpc-template.yaml \
  --parameters ParameterKey=EnvironmentName,ParameterValue=Lab-VPC
📤 Outputs (Exports)
Este stack exporta los siguientes valores para uso de otros stacks:
Export NameDescripciónValorLab-VPC-VPC-IDID de la VPCvpc-xxxxxLab-VPC-Public-Subnet-1ID de Public Subnet 1subnet-xxxxxLab-VPC-Public-Subnet-2ID de Public Subnet 2subnet-xxxxxLab-VPC-Private-Subnet-1ID de Private Subnet 1subnet-xxxxxLab-VPC-Private-Subnet-2ID de Private Subnet 2subnet-xxxxx
🔍 Validación
Verificar recursos creados
bash# Listar recursos del stack
aws cloudformation list-stack-resources \
  --stack-name Lab-VPC

# Ver VPC creada
aws ec2 describe-vpcs \
  --filters "Name=tag:Name,Values=Lab-VPC"

# Ver subnets
aws ec2 describe-subnets \
  --filters "Name=vpc-id,Values=<VPC-ID>"
Tests de conectividad
bash# Verificar que subnets públicas tienen ruta a IGW
# Verificar que subnets privadas NO tienen ruta a IGW
# Confirmar que están en diferentes AZs
🎯 Mejores Prácticas Implementadas
✅ Multi-AZ: Subnets en diferentes zonas de disponibilidad
✅ Segregación: Subnets públicas y privadas separadas
✅ DNS: DNS hostnames habilitado para resolución de nombres
✅ Tags: Todos los recursos etiquetados correctamente
✅ Exports: Valores exportados para reutilización
💰 Costos
VPC y componentes de red: $0.00
Los siguientes recursos NO tienen costo:

VPC
Subnets
Internet Gateway
Route Tables
Security Groups

🗑️ Eliminación
⚠️ ADVERTENCIA: Elimina primero todos los stacks que dependen de esta VPC.
bash# Orden correcto de eliminación:
1. Eliminar Lab-RDS (si existe)
2. Eliminar Lab-EC2 (si existe)
3. Eliminar Lab-VPC (este stack)

# Comando CLI:
aws cloudformation delete-stack --stack-name Lab-VPC
📚 Referencias

AWS VPC Documentation
VPC Best Practices
CIDR Calculator

🔧 Modificaciones Futuras
Posibles mejoras para este template:

 Agregar NAT Gateway para subnets privadas
 Implementar VPC Flow Logs
 Agregar Network ACLs personalizadas
 Configurar VPC Endpoints para S3
 Implementar IPv6 support
