# Bitácora de aprendizaje: Servidores virtuales y contenedores

**Asignatura:** Introducción al Cloud Computing (CDY2201) · **Semana 2**
**Estudiante:** Claudio Fuentes A. · **Carrera:** Analista Programador
**Fecha:** Noviembre 2026

> Actividad sumativa: *Relacionando software y hardware: servidores virtuales y contenedores*.
> Caso: migración parcial de **TechFest Soluciones** a una arquitectura híbrida (estaciones locales + Amazon EC2 con Docker).

---

## Índice
1. [Parte 1: Análisis de caso](#parte-1-análisis-de-caso)
2. [Parte 2: Laboratorio en AWS](#parte-2-laboratorio-en-aws)
3. [Comparación t3.micro vs t3.small](#comparación-t3micro-vs-t3small)
4. [Seguridad: Security Group](#seguridad-security-group)
5. [Aprendizajes y retroalimentación](#aprendizajes-y-retroalimentación)

---

# Parte 1: Análisis de caso

## Caso: Mejorando la infraestructura tecnológica de TechFest Soluciones

### Contexto de la empresa

TechFest Soluciones es una startup emergente que se dedica a la organización de ferias tecnológicas y eventos relacionados con la innovación digital. Actualmente, la empresa ha comenzado a captar la atención de diversas empresas y startups, gracias a su habilidad para conectar a los expositores con un público interesado en tecnología.

El equipo está compuesto por cuatro personas: un director de eventos, dos coordinadores que se encargan de la logística y la comunicación con los expositores, y un técnico responsable de la gestión de la página web y de la atención al cliente a través de chat en vivo y correo electrónico.

La empresa cuenta con cuatro computadoras portátiles de gama media y un router básico. Sin embargo, debido al crecimiento y éxito en la organización de eventos, han decidido invertir en mejorar sus estaciones de trabajo y migrar su plataforma web a la nube utilizando **Amazon EC2 con Docker**, en lugar de comprar un servidor local físico.

El sitio web maneja un tráfico moderado que aumenta considerablemente durante las semanas previas a los eventos. Esta variabilidad hizo que un servidor local fuera poco práctico, ya que estaría sobredimensionado la mayor parte del tiempo y subdimensionado durante los picos. La nube les permite ajustar recursos según la demanda.

Me han contratado como consultor tecnológico para asesorarles en la implementación de esta **arquitectura híbrida** (equipos locales + servidor cloud).

### Objetivos de la implementación

- Mejorar la capacidad de respuesta y el rendimiento del sitio web de la empresa.
- Optimizar la gestión de la información de los expositores y asistentes.
- Incrementar la seguridad de los datos y transacciones.
- Facilitar la colaboración y el trabajo remoto entre los miembros del equipo.

### Componentes de hardware: estaciones de trabajo (on-premise)

| Componente | Detalle |
|---|---|
| Procesador | AMD Ryzen 9 5900X |
| Memoria RAM | 32 GB DDR4 |
| Almacenamiento | Samsung 980 Pro 1 TB NVMe SSD |
| Tarjeta gráfica | Nvidia GeForce RTX 3060 |
| Monitor | HP Z27 27" 4K UHD |
| Teclado y ratón | Logitech MX Keys & MX Master 3 Wireless |
| Conectividad | Intel Wi-Fi 6E AX210, Realtek 2.5GbE NIC |
| Sistema operativo | Windows 11 Pro |

### Componentes de red local

- Router empresarial
- Switch administrable
- NAS (Network Attached Storage) para respaldos internos

### Componentes del servidor cloud (Amazon EC2)

| Componente | Detalle |
|---|---|
| Instancia | EC2 tipo t3.micro (2 vCPU, 1 GiB RAM) |
| AMI | Amazon Linux 2023 |
| Almacenamiento | EBS 20 GiB gp3 |
| Red | VPC con subred pública, Security Group (puertos 22, 80, 443) |
| Software | Docker instalado, contenedor nginx ejecutando la plataforma de gestión de eventos |

### Software implementado en estaciones de trabajo

- Suite de productividad empresarial (Microsoft 365)
- Soluciones de ciberseguridad avanzadas (antivirus, firewall, herramientas de detección de intrusiones)
- Herramientas de colaboración y videoconferencia
- Software de respaldo y recuperación de datos

---

## 1. Equivalencias entre hardware físico y virtual

| Componente físico (estación de trabajo) | Función | Equivalente virtual (EC2) | Función en la instancia |
|---|---|---|---|
| **CPU:** AMD Ryzen 9 5900X | Ejecuta las instrucciones de programas y del sistema operativo. | **vCPU** (t3: 2 vCPU) | Hilo de ejecución asignado a la instancia sobre el hardware del host de AWS. Define cuántas tareas simultáneas procesa. |
| **RAM:** 32 GB DDR4 | Memoria volátil de trabajo: más capacidad permite manejar más procesos a la vez. | **Memoria de la instancia** (t3.micro: 1 GiB · t3.small: 2 GiB) | RAM asignada según el tipo de instancia. Aquí la comparten SO, Docker y los contenedores. |
| **Almacenamiento local:** Samsung 980 Pro 1 TB NVMe | Almacenamiento no volátil de SO, aplicaciones y archivos. | **Volumen EBS** (gp3) | Disco de bloques persistente conectado a la instancia. Sobrevive a reinicios y se puede ampliar. |
| **Tarjeta de red:** Intel Wi-Fi 6E AX210 / Realtek 2.5GbE | Conecta el equipo a la red y a internet. | **ENI** (Elastic Network Interface) | Interfaz de red virtual: tiene IP privada, IP pública asociada y Security Groups. |
| **Sistema operativo:** Windows 11 Pro | Gestiona el hardware y ejecuta el software. | **AMI** (Amazon Linux 2023) | Plantilla que define el SO y el software base con que arranca la instancia. |
| **Red local:** router y switch | Interconectan los equipos y controlan el acceso a la red. | **VPC + subred pública + Security Group** (puertos 22, 80, 443) | Red privada virtual aislada. El Security Group actúa como firewall de la instancia y define qué puertos están abiertos. |
| **GPU:** Nvidia RTX 3060 | Procesa gráficos y video. | *Sin equivalente en t3* | Las instancias t3 no incluyen GPU. El servidor web no la necesita. |
| **Monitor, teclado y ratón** | Entrada y salida para el usuario. | *Sin equivalente físico* | La instancia se administra de forma remota por SSH y por la consola de AWS. |

> **Instancia EC2** es el servidor virtual completo. **t3.micro / t3.small** es el *tipo* de instancia, es decir, la combinación de vCPU y RAM que se contrata.

## 2. Software y aporte a los objetivos de la empresa

**Estaciones de trabajo**

| Software | Propósito | Objetivo que apoya |
|---|---|---|
| Microsoft 365 | Productividad: documentos, correo, trabajo compartido. | Gestión de información de expositores y asistentes. |
| Ciberseguridad (antivirus, firewall, detección de intrusiones) | Proteger equipos y datos locales. | Seguridad de datos y transacciones. |
| Colaboración y videoconferencia | Comunicación entre el equipo y con expositores. | Trabajo remoto y colaboración. |
| Respaldo y recuperación | Copias de seguridad en la NAS. | Continuidad y protección de la información. |

**Servidor EC2**

| Software | Función | Aporte |
|---|---|---|
| **Amazon Linux 2023** | Sistema operativo del servidor, actualizado por consola (`dnf`). | Base estable y segura. |
| **Docker** | Plataforma para **crear y ejecutar contenedores**: empaqueta una aplicación con sus dependencias para que corra igual en cualquier entorno. | Despliegues rápidos, repetibles y fáciles de escalar. |
| **nginx** | **Servidor web**: atiende solicitudes HTTP y entrega el contenido a los usuarios. En este caso corre *dentro* de un contenedor Docker. | Sitio web disponible y con buen rendimiento. |

> **Diferencia clave:** Docker es el *motor* que ejecuta contenedores; nginx es la *aplicación* que corre dentro del contenedor y atiende las peticiones web. Docker no sirve páginas por sí solo y nginx no crea contenedores.

**Otros controles en el servidor (comparados con las estaciones Windows)**
- **Acceso:** en Windows se usan cuentas y contraseñas; en EC2 se usan **llaves SSH**.
- **Actualizaciones:** en Windows con Windows Update; en EC2 por consola (`dnf update`).
- **Auditoría:** en Windows con el visor de eventos; en EC2 con los logs de nginx y Docker.
- **Respaldo:** la NAS cubre el respaldo local. En la nube se apoya en volúmenes EBS (y snapshots).
- **Firewall:** el Security Group controla el tráfico entrante y saliente.

## 3. Tres aspectos en que EC2 influye en la plataforma

1. **Escalabilidad:** se pueden aumentar recursos en los picos previos a los eventos y reducirlos después, sin quedar sobredimensionados.
2. **Costos:** se paga por uso en lugar de una inversión inicial en hardware, energía y mantenimiento.
3. **Seguridad:** AWS protege la infraestructura física y la empresa controla quién accede mediante Security Groups, llaves SSH e IAM.

## 4. Desafíos de la arquitectura híbrida y medidas de mitigación

| Desafío | Riesgo | Medidas concretas |
|---|---|---|
| **Sincronización de datos** | Información distinta entre la NAS y AWS; dudas sobre cuál es la fuente de verdad. | Definir qué sistema es el maestro. Automatizar la sincronización con tareas programadas (`cron` + `rclone` / `aws s3 sync`) o **AWS DataSync**. Verificar con logs y alertas de fallo. |
| **Seguridad** | Ampliar la superficie de ataque, sobre todo desde la red local. | **Cifrar las comunicaciones** (HTTPS/TLS en el 443, VPN o SSH). Restringir el SSH a IPs autorizadas, aplicar mínimo privilegio (IAM) y usar MFA. |
| **Costos** | Mantener dos entornos en paralelo y que el gasto en cloud crezca sin control. | **Monitorear recursos y costos** con CloudWatch, Cost Explorer y **AWS Budgets** con alertas. Apagar instancias que no se usen y dimensionar según métricas reales. |

## 5. Costo-beneficio: EC2 + Docker vs servidor físico

| Factor | Servidor local | Servidor EC2 |
|---|---|---|
| Costo inicial | Alto: compra del equipo que soporta la BD. | Bajo: sin compra de hardware. |
| Mantención | Recurrente: mantenimiento y energía. | Cuota según uso y recursos contratados. |
| Escalabilidad | Requiere comprar más hardware. | Se cambia el tipo de instancia o el volumen desde la plataforma. |

**Ventajas**
1. **Elasticidad:** los recursos se ajustan a la demanda variable (picos antes de los eventos).
2. **Menor inversión inicial:** se elimina la compra de hardware y su mantenimiento físico.

**Desventajas**
1. **Dependencia de conectividad:** si falla internet (en la oficina o en el proveedor), el equipo pierde acceso al servidor y a la administración remota. Se mitiga con conexiones redundantes y alertas de disponibilidad.
2. **Costos operacionales recurrentes:** se paga mientras la instancia exista. Además se suman EBS, IP pública y transferencia de datos. A largo plazo, con una carga estable y constante, puede superar el costo de un servidor propio ya amortizado.

> Una t3.micro (1 GiB) queda corta si Docker y el SO consumen gran parte de la memoria. Una t3.small (mínimo) o t3.medium (recomendada) hace que la opción cloud sea más conveniente.

---

# Parte 2: Laboratorio en AWS

## Tarea 1: Crear la instancia EC2

| Parámetro | Guía | Lo que usé |
|---|---|---|
| Nombre | `servidor-web` | `servidor-web` |
| AMI | Amazon Linux 2023 (64 bits, x86) | Amazon Linux 2023 (64 bits, x86) |
| Tipo | t3.micro | **t3.small** (ver [justificación](#comparación-t3micro-vs-t3small)) |
| Almacenamiento | 10 GiB gp3 | 10 GiB gp3 |
| Red | Subred pública con IP pública | Subred pública con IP pública |
| Security Group | SSH (22) y HTTP (80) | SSH (22) y HTTP (80) |

[Creación de la instancia]
<img width="1032" height="423" alt="imagen" src="https://github.com/user-attachments/assets/7428f172-38f4-45bf-95f2-3096e14f7409" />

<img width="1033" height="657" alt="imagen" src="https://github.com/user-attachments/assets/e4ff21e9-d053-46ee-a3a4-3ea927291cc3" />

<img width="1033" height="629" alt="imagen" src="https://github.com/user-attachments/assets/cdeb5f7a-466b-44fd-9a93-be7a860e39f0" />
Seleccionamos el tipo de instancia , NOTA: Si bien la actividad contempla el uso de un sistema t3.micro de 1gb , se considera que por ámbitos de este laboratorio se incorporara una t3.small 2gb,
producto de que ejecutaremos imágenes y el Docker que ocuparan gran parte de la capacidad de una t3.micro , para trabajar con un poco mas de procesamiento y holgura se considera este cambio .

<img width="1033" height="706" alt="imagen" src="https://github.com/user-attachments/assets/1ac22565-0ab0-4c04-8918-ccdd0489940e" />
La guía nos indica que debemos permitir el trafico http y tener una ssh publica 
Red: Subred pública con IP pública habilitada
Security Group: Permitir SSH (puerto 22), HTTP (puerto 80)
Al igual que se configura el almacenamiento indicado por la guía a 10 gb

<img width="503" height="472" alt="imagen" src="https://github.com/user-attachments/assets/47a2993d-d0e1-42ea-9dc6-8dedde55ac99" />

Al finalizar tenemos en el cuadro derecho , el resumen de detalles de nuestra instancia donde se contemplan las características solicitadas para la actividad 

## Tarea 2: Componentes de hardware virtual

| Componente | Dónde verlo |
|---|---|
| Tipo de instancia (vCPU y RAM) | Detalles de la instancia |
| Volumen EBS | Pestaña *Almacenamiento* |
| ENI (IP privada y pública) | Pestaña *Redes* |
| AMI (sistema operativo) | Detalles de la instancia |

[Componentes virtuales]
<img width="1032" height="428" alt="imagen" src="https://github.com/user-attachments/assets/a56bd836-717e-4045-80be-6d730ee18f6f" />

## Tarea 3: Instalar Docker

Previo a esto nos conectamos al terminal 
Conexión por SSH con **MobaXterm**.
<img width="975" height="616" alt="imagen" src="https://github.com/user-attachments/assets/a317eb82-4dfa-4c90-b1d8-282903b42be1" />

y actualizamos nuestro sistema 

<img width="670" height="163" alt="imagen" src="https://github.com/user-attachments/assets/98ab5295-50b3-43cf-8bb1-c673b4d67fe3" />

Instalamos Docker , pero para ello debe instalar las siguientes dependencias : 
<img width="1033" height="304" alt="imagen" src="https://github.com/user-attachments/assets/ff13778a-ed59-4d91-8691-dad1d57569c2" />

tras instalar saldra el siguiente mensaje 
<img width="1033" height="525" alt="imagen" src="https://github.com/user-attachments/assets/4c7b040b-8ad7-4af1-b3ec-ff497c25e37a" />

Verificamos la vercion instalada , la cual nos indica que la versión instalada es la 25.0.14 la cual es la actual a la fecha 
<img width="1033" height="205" alt="imagen" src="https://github.com/user-attachments/assets/94b598e3-c519-4f00-b210-11ba5867a485" />


```bash
sudo dnf update -y
sudo dnf install -y docker
docker --version
```

### Diagnóstico: el servicio Docker estaba detenido

Al verificar el estado del servicio, Docker aparecía **inactivo**: estaba instalado, pero el *daemon* no estaba corriendo y no podía conectarse a su servicio.

```bash
sudo systemctl status docker        # inactivo
sudo systemctl enable --now docker  # habilita al arranque e inicia ahora
sudo systemctl status docker        # activo (running)
```

[Docker detenido]
<img width="1033" height="164" alt="imagen" src="https://github.com/user-attachments/assets/0387049c-b360-40b8-bb2f-e34e8e7eacad" />

[Docker activo]
<img width="1033" height="349" alt="imagen" src="https://github.com/user-attachments/assets/7f15c182-20c3-4e10-8fdf-9eac0289d3a8" />


> **Aprendizaje:** en infraestructura cloud no basta con instalar una herramienta; hay que comprobar que su servicio esté operativo.

## Tarea 4: Ejecutar contenedor nginx

```bash
sudo docker run -d --name <mi-nombre> -p 80:80 nginx:stable
sudo docker ps
```

La imagen `nginx:stable` se ejecuta en segundo plano (`-d`) y publica el puerto 80 del contenedor en el 80 de la instancia. Luego se abre `http://<IP-pública>` en el navegador y aparece la página de bienvenida de nginx.

[Contenedor en ejecución]
<img width="1033" height="297" alt="imagen" src="https://github.com/user-attachments/assets/aef2a692-d940-416a-87af-bbf9bbea9b24" />

verificamos que al usar la ip que entrega aws que nos llevara a nginx : 
<img width="1032" height="526" alt="imagen" src="https://github.com/user-attachments/assets/75c7d25b-cdb4-498d-a97e-d426d071f8ce" />

[Bienvenida de nginx]
<img width="1033" height="327" alt="imagen" src="https://github.com/user-attachments/assets/f3a234ab-1e3d-4cb2-b53e-9235b21e1b90" />

---

# Comparación t3.micro vs t3.small

Se usó **t3.small** en lugar de la t3.micro de la guía para tener más memoria al trabajar con Docker y las imágenes.

| Criterio | t3.micro | t3.small |
|---|---|---|
| vCPU | 2 | 2 |
| Memoria | 1 GiB | 2 GiB |
| Costo referencial (Linux, us-east-1, on-demand) | ~US$ 0,0104 / hora | ~US$ 0,0208 / hora |
| Rendimiento con Docker | Poca holgura: SO + Docker + contenedor consumen gran parte de la RAM | Más margen para el SO, el daemon y los contenedores |

> Los precios son referenciales y varían según región y fecha. Conviene confirmarlos en la calculadora de AWS.

**Medición de consumo de memoria** (comandos para comparar ambas instancias):

```bash
free -h
sudo docker stats --no-stream
```

| Instancia | Memoria usada (SO + Docker) | Memoria usada con nginx | Memoria libre |
|---|---|---|---|
| t3.small | _completar_ | _completar_ | _completar_ |
| t3.micro | _completar_ | _completar_ | _completar_ |

**Conclusión:** _completar con los resultados de la medición_.

---

# Seguridad: Security Group

En el laboratorio el puerto **22 (SSH)** quedó abierto a cualquier IP (`0.0.0.0/0`). Esto expone el servidor a intentos de acceso no autorizados.

**Mejoras para próximas implementaciones**
- Restringir el puerto 22 solo a las IP autorizadas (por ejemplo `mi.ip.publica/32`).
- Mantener únicamente las reglas necesarias (22 restringido y 80; 443 solo cuando exista HTTPS).
- Usar llaves SSH y revisar periódicamente las reglas del Security Group.

---

# Aprendizajes y retroalimentación

**Qué aprendí**
- Cada componente físico tiene un equivalente virtual con una función específica: CPU↔vCPU, disco local↔EBS, tarjeta de red↔ENI, SO↔AMI.
- Docker y nginx cumplen roles distintos: Docker ejecuta contenedores y nginx sirve contenido web.
- Una arquitectura híbrida exige medidas concretas de sincronización, seguridad y control de costos.
- Es necesario verificar que los servicios estén activos, no solo instalados.

  
