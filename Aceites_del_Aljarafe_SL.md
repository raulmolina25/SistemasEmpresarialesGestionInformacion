## Bloque A: Análisis de Mercado y Selección (CE a, c)

Debéis elegir entre **Odoo (SaaS o Community)**, **SAP S/4HANA** o **Zoho One**.

1. Justifica la elección basándote en el perfil de la empresa (25 empleados, presupuesto ajustado, necesidad de personalización en el etiquetado).

Tanto Odoo como Zoho One serían buenas opciones para empresas de 25 empleados con presupuesto ajustado, pero al necesitar personalización en el etiquetado nos hemos decantado por Zoho One, ya que permite la personalizar y adaptación dependiendo de las necesidades del negocio. Odoo es buena para personalizar gracias a la creación de módulos pero esto también puede hacer que el precio se dispare y aumente el tiempo de desarrollo de código de los programadores. 
![logo Zoho One](Logo_Zoho_One.png) 

2. **Cálculo de TCO:** Realiza una estimación a 3 años. No olvidéis incluir:  
   * Coste de licencias/suscripción.

| Plan | Precio mensual | Precio anual (por mes) | Condiciones |
| :---: | :---: | :---: | :---: |
| **Precio de usuario flexible** | 105€ por usuario | 90€ por usuario | Licencias individuales, sin necesidad de cubrir a todos los empleados. |
| **Precio para todos los empleados** | 45€ por empleado | 37€ por empleado | Una única licencia para cada empleado de la empresa. |

A no ser que el cliente concrete que va a querer que todos los empleados tengan licencia, se escogería por lo general el precio de usuario flexible, que al estar haciendo una estimación de 3 años escogeríamos el plan anual, por lo que el precio estimado total sería de **1080€ por empleado** al año, por lo que tres años sería **3240€ por empleado**.

Si el cliente prefiere que todos los empleados tengan licencia se escogería el precio para todos los empleados escogeríamos el precio anual, por lo que serían **444€ al año por empleado**, por lo que en tres años serían 1332**€ por empleado**

* Coste de implantación (vuestras horas de desarrollo: estima 100h a 40€/h).

**40€** puesto que al ofrecer varias aplicaciones generales no se necesitaría tanto tiempo de desarrollo, por lo que vamos considerar 100 horas para la personalizar las aplicaciones ajustándolo a las necesidades concretas de la empresa y Zoho One permite formar flujos de trabajo automatizaso.

* Coste operativo (Hosting en Google Cloud, AWS, Huawei Cloud o similar).

Al utilizar Zoho One utiliza el modelo SaaS por lo que se trabaja directamente a internet

## Bloque B: Diseño de Seguridad RBAC (CE f)

Diseña la matriz de permisos para los siguientes roles, asegurando el **Principio de Mínimo Privilegio**:

* **Administrador:** Acceso total.  
* **Comercial:** Solo ve sus clientes y presupuestos (Record Rules).  
* **Operario de Almacén:** Solo ve stock y albaranes de entrada/salida.  
* **Contable:** Puede mirar facturas pero no puede modificar el stock.

|  | Administrador | Comercial | Operario de Almacén | Contable |
| :---- | :---: | :---: | :---: | :---: |
| Leer clientes | ✓ | ✓ | X | X |
| Modificar clientes | ✓ | X | X | X |
| Leer presupuestos | ✓ | ✓ | X | X |
| Modificar presupuestos | ✓ | X | X | X |
| Leer stock | ✓ | X | ✓ | X |
| Modificar stock | ✓ | X | X | X |
| Leer albaranes de entrada | ✓ | X | ✓ | X |
| Modificar albaranes de entrada | ✓ | X | X | X |
| Leer albaranes de salida | ✓ | X | ✓ | X |
| Modificar albaranes de salida | ✓ | X | X | X |
| Leer facturas | ✓ | X | X | ✓ |
| Modificar facturas | ✓ | X | X | X |

## Bloque C: Documentación de Explotación (CE i)

Siguiendo la norma **ISO/IEC 26514**, redacta un breve **Manual de Despliegue** para que el responsable de IT de la empresa pueda levantar el sistema en caso de caída. Debe incluir:

1. El fragmento de *docker-compose.yml* necesario.  
2. El comando para realizar un backup de la base de datos PostgreSQL.

El sistema está construido en **Docker Compose**. Archivo *docker-compose.yml:*

version: '3.8'

services:

  db\_zoho\_backup:  
    image: postgres:15  
    container\_name: lab\_zoho\_db  
    environment:  
      \- PUID=1000  
      \- PGID=1000  
      \- TZ=Europe/Madrid  
      \- POSTGRES\_DB=zoho\_local  
      \- POSTGRES\_USER=admin\_zoho  
      \- POSTGRES\_PASSWORD=sistemas\_informaticos  
    ports:  
      \- "5432:5432"  
    restart: unless-stopped

  zoho\_agent:  
    image: linuxserver/webtop:ubuntu-xfce  
    container\_name: lab\_zoho\_conector  
    environment:  
      \- PUID=1000  
      \- PGID=1000  
      \- TZ=Europe/Madrid  
    ports:  
      \- "3000:3000"   
    shm\_size: "1gb"  
    restart: unless-stopped

Para hacer un **backup**:
** Como la base de datos esta en Docker tendremos que añadir docker exec previamente al comando. **

**Asegurarse como se ven los datos:** 

SELECT \* FROM customers;

**Si desea realizar copias de seguridad y restaurar la base de datos rápidamente entre varios servidores, ingrese lo siguiente:**

pg\_dump \-h 78.43.11.2 db\_name | psql \-h 72.43.11.2 db\_name

La primera parte permitió conectarse al servidor con db y utilizar la salida de redirección de tubería al servidor de destino con un comando después de un signo separado.

**Y usando el comando scp podemos transferir db:**

scp ./backup.sql root@94.141.98.9:/

**Deberíamos crear una base de datos y restaurar el contenido con el siguiente comando:**

CREATE DATABASE db12;

**Luego restaure con el comando psql:**

psql \-U postgres \-d db12 \< backup.sql

Si la base de datos tiene **muchos datos** y un **gran tamaño**, utilice **split** y **utilidad zip** para transferir piezas de datos:

pg\_dump name\_db | gzip \> name\_archive.gz

gunzip \-c name\_archive.gz | psql name\_db

Y dividir con la siguiente sintaxis:

pg\_dump name\_db | split \-b 4G \- name\_file

cat name\_file | psql name\_db

Eso ayuda a que la transferencia de datos sea más efectiva en el proceso de copia de seguridad.

Referencias:  
[https://www.cosmikal.es/principio-de-minimo-privilegio-least-privilege-enforcement-fundamento-tecnico-para-una-gestion-de-accesos-segura-y-sostenible/](https://www.cosmikal.es/principio-de-minimo-privilegio-least-privilege-enforcement-fundamento-tecnico-para-una-gestion-de-accesos-segura-y-sostenible/)  
[https://www.ibm.com/docs/es/aix/7.3.0?topic=privileges-least-privilege-principle](https://www.ibm.com/docs/es/aix/7.3.0?topic=privileges-least-privilege-principle)  
R. S. Sandhu, E. J. Coyne, H. L. Feinstein, y C. E. Youman, "Role-based access control models," IEEE Computer, vol. 29, no. 2, pp. 38-47, 1996\. \[Online\]. Disponible en:  
[https://csrc.nist.gov/csrc/media/projects/role-based-access-control/documents/sandhu96.pdf](https://csrc.nist.gov/csrc/media/projects/role-based-access-control/documents/sandhu96.pdf)  
[https://docs.google.com/document/d/16yClUUrFCrQCH9nQBI9hp701W2qQyvvTErRwTeUxk7c/edit?usp=sharing](https://docs.google.com/document/d/16yClUUrFCrQCH9nQBI9hp701W2qQyvvTErRwTeUxk7c/edit?usp=sharing)  
[https://www.appvizer.com/magazine/operations/erp/zoho-vs-odoo](https://www.appvizer.com/magazine/operations/erp/zoho-vs-odoo)   
[https://www.zoho.com/es-xl/one/pricing/](https://www.zoho.com/es-xl/one/pricing/)   
[https://www.agenciareinicia.com/blog/zoho-one-caracteristicas-precio/\#que-incluye](https://www.agenciareinicia.com/blog/zoho-one-caracteristicas-precio/#que-incluye)   
[https://serverspace.io/es/support/help/how-to-backup-and-restore-in-a-postgresql/](https://serverspace.io/es/support/help/how-to-backup-and-restore-in-a-postgresql/)

Poner imagenes en el md:  
\!\[Descripción de la imagen\](/images/picture.jpg) , referencia [https://denshub.com/es/hugo-post-insert-image/](https://denshub.com/es/hugo-post-insert-image/)  
