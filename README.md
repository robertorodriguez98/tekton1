# Ciclo CI/CD con Tekton y ArgoCD

## Requisitos

**Local**

* Clúster de kubernetes (en mi caso, minikube)
* Kubectl
* Bash

**Infraestructura**

* Cuenta de dockerhub
* Permisos sobre el repositorio [https://github.com/robertorodriguez98/argocd1](https://github.com/robertorodriguez98/argocd1)
* Token de github con los siguientes permisos: repo, admin:org y admin:repo_hook.

## Propósito del repositorio

Este repositorio contiene los archivos necesarios para crear un pipeline de tekton que se encargará de crear una imagen docker, subirla a dockerhub y actualizar el repositorio de argocd con los cambios realizados.

Este repositorio contiene 

1. Desarrollador modifica el index.html.
2. Al hace un push, salta un webhook que conecta con un EventListener de tekton, que ejecuta el pipeline que realiza lo siguiente:
    * Clona el repositorio.
    * Se construye la imagen docker y se sube a dockerhub.
    * Se actualiza el repositorio de argocd con la nueva versión de la imagen.
3. ArgoCD detecta los cambios en el repositorio y despliega la nueva versión de la aplicación.

## Pasos a seguir

1. Clonar en este repositorio en nuestra máquina, en caso de que tengamos git instalado en nuestro pc usareamos el siguiente comando:

```bash
git clone https://github.com/robertorodriguez98/tekton1.git
```

En el caso de no tener instalado git podremos descargarnos el repositorio comprimido. En la página principal del repositorio pulsaremos el botón verde, **Code** , y eligiremos la opción **Download ZIP**.

2. Creamos un token de github, siguiendo los siguientes pasos:
    1. Hacemos click en el logo de nuestro usuario arriba a la derecha.
    2. Seleccionamos **Settings**.
    3. Seleccionamos **Developer settings**.
    4. Seleccionamos **Personal access tokens**.
    5. Generamos un nuevo token, con los sigientes permisos:
        * repo
        * admin:org
        * admin:repo_hook
3. Instalamos minikube; vamos a usar como driver docker, por lo que lo instalamos también:

```bash
sudo apt install docker.io
sudo usermod -aG docker usuario
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube start --driver=docker
```

4. Instalamos kubectl, siguiendo los siguientes pasos:

```bash
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install kubectl /usr/local/bin/kubectl
```

5. Ahora creamos los secret que usaremos en el pipeline:
    * **dockerhub**:
        1. iniciamos sesión con la línea de comandos:
        ```bash
         docker login index.docker.io 
        ```
        2. copiamos el contenido del archivo **config.json** que se ha creado en la carpeta **.docker** de nuestro usuario, y lo codificamos en base64:
        ```bash
        cat ~/.docker/config.json | base64 -w0 
        ```
        3. Creamos el fichero docker-credentials.yaml con el siguiente contenido:
        ```yaml
        apiVersion: v1
        kind: Secret
        metadata:
        name: docker-credentials
        data:
        config.json: <contenido codificado en base64>
        ```
        4. finalmente creamos el secret:
        ```bash
        kubectl apply -f docker-credentials.yaml
        ```
    * **github**:
    

