

# Lab Vulnerable utilizando AWS



Durante nuestro viaje de aprendizaje en el área de ciberseguridad siempre se ha dificultado al inicio encontrar entornos de prueba vulnerables, no obstante gracias al esfuerzo de muchas personas podemos decir que ese punto ha ido mejorando, dentro de los lugares preferidos de aprendizaje para está área podemos mencionar https://tryhackme.com/, https://hackthebox.com/, https://rangeforce.com/, https://www.vulnhub.com/, etc. 

Aunque nada va reemplazar el hecho que uno despliegue su infraestructura en un hypervisor como Vmware, Virtualbox, hyperV o que repliquemos un ejercicio de cyber range con un Active Directory con workstation, sin embargo rápidamente se nos presenta la limitante los recursos de memoria, disco duro, procesador por ejemplo los recursos mínimos para un ejercicio de cyber range serían los siguiente:

- Memoria Ram 16GB disponible (4GB para DC, 4GB para dos workstations y el resto para SO)
- 60GB de almacenamiento como mino disponible

De igual manera podríamos implementar nuestro CTF con contenedores vulnerables, mi amigo y colega Elzer Pineda escribió un [articulo](https://backtrackacademy.com/articulo/como-crear-un-ctf-con-owasp-juice-shop-y-traefik) donde explica como hacerlo con docker y traefik.  

El objetivo de este articulo es demostrar la implementación de un hacking lab en AWS bastante similar a un home lab y utilizando una máquina virtual de [vulnhub](vulnhub.com). 

### Requisitos

1. Cuenta de AWS (no gratuita)
2. Una vm vulnerable de vulnhub usaremos la siguiente https://download.vulnhub.com/breach/Breach-1.0.zip
3. De igual manera debemos verificar que la Vm este dentro de los kernels permitidos https://docs.aws.amazon.com/vm-import/latest/userguide/prerequisites.html



1. #### Creación VPC

Vamos a proceder a crear una red virtual en donde incluiremos la máquina vulnerable y un Kali linux, buscamos en la barrita de búsqueda **vpc** y seleccionamos la opción de VPC señalada en la siguiente imagen

![1](img/1.png)

Ahora vamos a proceder a crear nuestra VPC, haciendo click en el botón naranja **Crear VPC**

![2](img/2.png)

Debemos realizar las siguiente modificaciones, en el nombre del proyecto y la red debe ser **192.168.110.0/24**, para este lab es asi dado que es la red usada en la vm de vulnhub.

![3](img/3.png)

En la zonas de disponibilidad solamente marcamos que sea una 

![4](img/4.png)

Procedemos a darle click a crear VPC

![5](img/5.png)

Mientras se crea debemos observar que todo lo que se está generando

![6](img/6.png)

Cunando este listo podemos ir a **ver VPC**

![7](img/7.png)

Esta lista nuestra vpc

![8](img/8.png)

2. #### Creación de Instancia EC2 con Kali 

Escribimos en nuestra barra de búsqueda Ec2

![9](img/9.png)

En el panel de Ec2 vamos a instancias y luego Lanzar instancias

![10](img/10.png)

Procedemos  a ponerle un nombre a la instancias y debemos ir a buscar la imagen de Kali en el martketplace

![11](img/11.png)

Buscamos la imagen correcta en este caso escribimos **Kali** en el buscador y vamos a seleccionar la imagen

 ![12](img/12.png)

Procedemos a darle click  a **continuar**

![13](img/13.png)

Si todo sale bien regresamos a la pantalla anterior y debe salir la imagen a utilizar

![14](img/14.png)

Procedemos a seleccionar el tipo de instancia y generar el par de llaves

![15](img/15.png)

Creamos el par de llaves

<img src="img/16.png" alt="16" style="zoom:50%;" />

Debe quedarnos seleccionada la llave que acabos de crear y debemos guardarla en un ruta donde seamos los propietarios

![17](img/17.png)

En la sección de **Configuraciones de red**, vamos a editar y seleccionar la red que creamos anteriormente

<img src="img/18.png" alt="18" style="zoom:50%;" />

Debe aparecernos la vpc que creamos

![19](img/19.png)

Habilitamos la asignación de IP pública, en esta sección podemos dejar las demás configuraciones como están, recordar revisar que tengamos una regla para acceder por ssh al Kali

<img src="img/20.png" alt="20" style="zoom:50%;" />

Por último le damos 25GB de almacenamiento y ya podemos **Lanzar Instancia**

![21](img/21.png)

Probamos el acceso a la instancia, le damos permisos correspondientes a la llave y accedemos 

```shell
chmod 400 kalidemo
ssh -i kalidemo.pem kali@54.236.37.6
```

<img src="img/22.png" alt="22" style="zoom: 67%;" />

Verificación del acceso al Kali

![23](img/23.png)



3. #### Subida archivo OVA a AWS S3

   Primero debemos comprobar la región donde se encuentra la instancia, para esta demo se encuentra en **us-east-1**

   ![24](img/24.png)

   Vamos a crear un bucket de S3, por lo que buscamos en la barra s3 y damos click 

   ![25](img/25.png)

   Creamos un bucket

   

   ![26](img/26.png)

   Colocamos el nombre que deseamos y verificamos la región sea **us-east-1**

   ![27](img/27.png)

   Para temas de este laboratorio vamos a darles permiso público al bucket, desmarcamos la opción **Bloquear todo el acceso público**

   ![28](img/28.png)

   Aceptamos los términos y lo demos lo podemos dejar por defecto

   ![29](img/29.png)

   Clickeamnos **crear bucket**

   ![30](img/30.png)

Damos Click en el nombre del bucket

![31](img/31.png)

Vamos a cargar el ova que bajamos de vulnhub

![](img/32.png)

Es importa antes de iniciar la carga del ova, verificar que nombre del archivo no tenga espacios

![32](img/33.png)

Por último comprobemos si los permisos fueron otorgados correctamente accedemos al bucket desde el navegador 

![34](img/34.png)



Una vez seleccionado el objeto debemos apuntar los **URI** lo usaremos más adelante y copiar la **URL del objeto**

![35](img/35.png)

Si les aparece algo como esto, debemos verificar los permisos

![36](img/36.png)

Vamos a cambiar el método de permisos de los bucket

![37](img/37.png)

Seleccionamos como aparece en la imagen

![38](img/38.png)

Regresamos a los permisos del objeto 

![39](img/39.png)

Vamos a editar la lista de control de acceso

![40](img/40.png)

Habilitamos la lectura pública, aceptamos los términos y guardamos. **Recordemos esto solamente lo hacemos de esta manera dado que es un lab, podemos restringir los permisos cuando ya hemos importado el ova**

<img src="img/41.png" alt="41" style="zoom:50%;" />

Comprobamos el acceso al objeto

<img src="img/42.png" alt="42" style="zoom:50%;" />



4. Subida del OVA

   Los siguientes pasos los vamos a realizar desde el cloud shell de aws, no importa el servicio donde estemos podemos darle click al icono de la terminal

   ![43](img/43.png)



Cerramos el mensaje

![44](img/44.png)

Se nos debe desplegar una terminal interactiva

![45](img/45.png)

Necesitamos crear tres archivos **containers.json** con la ruta del .ova, el archivo **trust-policy.json** para aguar la política de importación y por último el archivo **role-policy.json** con la política donde se encuentra la imagen.

Vamos a crear containers.json

`vi containers.json`

Va contener lo siguiente

```json
 [
{
"Description": "My Server OVA",
"Format": "ova",
"Url": "s3://my-import-bucket/vms/my-server-vm.ova"
}
]
```

Reemplazamos el url por la ruta del bucket de s3

<img src="img/46.png" alt="46" style="zoom:50%;" />



Creamos el archivo trust-policy.json

`vi trust-policy.json`

Con el siguiente contenido

```json
{
"Version": "2012-10-17",
"Statement": [
{
"Effect": "Allow",
"Principal": { "Service": "vmie.amazonaws.com" },
"Action": "sts:AssumeRole",
"Condition": {
"StringEquals":{
"sts:Externalid": "vmimport"
}
}
}
]
}
```

<img src="img/47.png" alt="47" style="zoom:50%;" />



Crearemos un role llamado vmimport y le damos acceso de importación y exportación.

` aws iam create-role --role-name vmimport --assume-role-policy-document "file://~/trust-policy.json"`

Si todo sale bien debemos ver lo siguiente

![48](img/48.png)

Creamos un archivo con la política de role-policy.json

`vi role-policy.json`

Con el siguiente contenido, reemplazamos donde dice **<your arn>** por el correspondiente del bucket

```json
{
"Version":"2012-10-17",
"Statement":[
{
"Effect": "Allow",
"Action": [
"s3:GetBucketLocation",
"s3:GetObject",
"s3:ListBucket"
],
"Resource": [
"<your arn>",
"<your arn>/*"
]
},
{
"Effect": "Allow",
"Action": [
"s3:GetBucketLocation",
"s3:GetObject",
"s3:ListBucket",
"s3:PutObject",
"s3:GetBucketAcl"
],
"Resource": [
"<your arn>",
"your arn/*"
]
},
{
"Effect": "Allow",
"Action": [
"ec2:ModifySnapshotAttribute",
"ec2:CopySnapshot",
"ec2:RegisterImage",
"ec2:Describe*"
],
"Resource": "*"
}
]
}
```

![49](img/49.png)

Utilice el siguiente comando **put-role-policy** para adjuntar la política al rol creado anteriormente. Asegúrese de especificar la ruta completa a la ubicación del archivo role-policy.json.

`aws iam put-role-policy --role-name vmimport --policy-name vmimport --policy-document "file://~/role-policy.json"`

![50](img/50.png)

Importamos la imagen

`aws ec2 import-image --description "Vulnerable Ova" --disk-containers "file://~/containers.json"`

![51](img/51.png)

Podremos verificar el estatus con el siguiente comando

`aws ec2 describe-import-image-tasks --import-task-ids import-ami-0b9aef01fc95f30d2`

Donde el **--import-task-ids** lo obtenemos del mensaje de arriba en este caso **"ImportTaskId": "import-ami-06351f55bca1ab48f"**

<img src="img/52.png" alt="52" style="zoom:50%;" />

El estatus debe pasar a **completed**

<img src="img/53.png" alt="53" style="zoom:50%;" />

5. Creación de la VM

   Vamos instancias EC2  y la sección de AMI y verificamos que en efecto se importo con el id **import-ami-06351f55bca1ab48f** y procedemos a lanzar una instancia

   ![54](img/54.png)

Al seleccionar lanzar la instancia desde la Ami, nos crea la misma a partir del OVA importada

De esta parte podemos dejar los datos por defecto a excepción **Configuraciones de red**, aquí procedemos a seleccionar el vpc que creamos inicialmente y bajamos el firewall de la instancia

![55](img/55.png)

Permitimos todo el tráfico y lanzamos la instancia

![56](img/56.png)

Verificamos las instancias estén en ejecución

![57](img/57.png)

Podemos acceder al ip publico de la instancia vulnerable y comenzar con nuestro pentest

![58](img/58.png)



6. Bibliografía

- *Required permissions for VM Import/Export - VM Import/Export*. (s. f.). https://docs.aws.amazon.com/vm-import/latest/userguide/required-permissions.html

