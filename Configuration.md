## Entrar a la instancia
```bash
cd ~/
ssh -i <llave.pem> ubuntu@<IP-instancia>
```

## Verificar los contenedores corriendo en la instancia
```bash
sudo docker ps
# Guardamos en CONTAINER ID: b19d27dcb521
```
### Ingresamos al contenedor
```bash
sudo docker exec -it b19d27dcb521 bash
```
Una vez ingresado al contenedor, buscamos la admin key de jenkins
```bash
cat /var/jenkins_home/secrets/initialAdminPassword
```
Copiamos y pegamos el resultado en la dirección: <IP-instancia>:8080

## Primer Admin User
De manera común al entrar por primera vez a Jenkins, nos hará crear el primer usuario administrador, por lo que debemos de llenar los campos y darle a "Save and Continue" luego solo damos "Save and Finish"

## Configuración de Jenkins con Slack
Primero debemos de tener el archivo [Jenkinsfile](./Jenkinsfile) subido a un [REPOSITORIO](https://github.com/Chriss5-2/Jenkins-with-Slack).
Además, en nuestra cuenta de slack, seleccionamos un espacio de trabajo, en el channel **Aplicaciones** le damos click derecho y seleccionamos **Explorar aplicaciones**; buscamos **Jenkins CI** y le damos a instalar, en la nueva ventana, clickeamos **Añadir a Slack** y seleccionamos el canal sobre el que se publicará (de manera general, si no tienes creada, crea un channel llamado **genera** y ubicalo ahí para su integración con Jenkins CI), al integrar, en la nueva sección que nos aparece, buscamos y guardamos los valores de **subdominio de equipo** y **ID de credencial de token de integración**.

## Configurar Jenkins en ec2
Ingresamos al Jenkins de la instancia, vamos a **Settings** > **Plugins**, en ´Available plugins´ buscamos **Slack Notification** y lo instalamos.

Luego vamos a **Settings** > **Credentials** > **System** > **Global** -> **+ Add Credentials**, seleccionamos de tipo **Secret text** y lo rellenamos de la siguiente forma
- Secret: <ID de credencial de token de integración>
- ID: KEY-SLACK (puede ser otra ID pero no se podrá modificar luego)

Luego vamos a **System** bajamos hasta la sección de **Slack**
- Workspace: <subdominio_equipo>
- Credentials: KEY-SLACK
- Default channel: general

Luego clickeamos el botón **Test Connection** y en el equipo de slack, en el channel #general, nos aparecerá un mensaje de conexión
```bash
Slack/Jenkins plugin: you're all set on http://localhost:8080/
```

Además debemos de crear los canales **#deployments** and **#incidents**

## Creación de Pipeline
Creamos un nuevo job en Jenkins de tipo pipeline, en este caso con el nombre **Alertas-Slack** (puede ser cualquier nombre), en Pipeline configuramos de la siguiente manera.
- Definition: Pipeline script from SCM
- SCM: Git
    - Repository URL: <url-donde-está-Jenkinsfile>
    - Branch Specifier: */main (o la rama sobre la que se encuentre el archivo)
- Script Path: Jenkinsfile (si el archivo a ejecutar tiene otro nombre, se cambia)

## Probar conexión
Le damos a **Construit ahora** (en el channel #deployments aparecerá un aviso de que requiere aprobación) vamos a la sección **Stages** del mismo Job, ingresamos a la ejecución, y notamos que hay una pausa, donde tendremos que elegir entre **Proceed** o **Abort** donde la respuesta a esa selección se va a los channels deployments e incidents respectivamente.

## Generar conexión con el push de GitHub
En la sección de **Configurar** del pipeline, en la sección **Triggers** marcamos la opción **GitHub hook trigger for GITScm polling**, aplicamos y guardamos los cambios.
Luego en el repositorio de GitHub vamos a **Settings** > **Webhooks** -> **Add Webhook**:
- Payload URL: http://<IP-Servidor>:8080/github-webhook/
- Content type: application/json