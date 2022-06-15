# Proyecto - SoundStream

## Descripción del proyecto
SoundStream es una plataforma de streaming de música, intentando promover el arte Guatemalteco prometiendo una experiencia robusta, confiable y amigable para sus clientes. Es una plataforma completamente en la nube, diseñada para ser utilizada en cualquier navegador Web. En esta plataforma, los clientes podrán crear una cuenta y decidir suscribirse a un plan gratis o pagado. La diferencia entre ambos planes es únicamente la cantidad de canciones que un cliente puede escuchar seguido antes de recibir un anuncio, y la capacidad de retroceder canciones o reproducir playlists personales. La descripción de las funcionalidades que tendrá SoundStream será definida en el enunciado a continuación. 

# Fase 2

Esta fase consistirá en agregar nuevas funcionalidades a la aplicación web, además se realizará la migración a la nube para permitir a los usuarios realizar una prueba beta al sistema.

## Nuevas funcionalidades
### Pantalla de bienvenida
Esta debe ser una vista de bienvenida al usuario, para ideas de como diseñarla y qué datos agregar en ella pueden ver los siguientes ejemplos:

https://www.spotify.com/gt/
https://www.deezer.com/mx/
https://soundcloud.com/ 
https://www.apple.com/la/apple-music/

### Pantalla de suscripciones
En esta pantalla se debe mostrar las dos suscripciones, comparándo los beneficios de cada una de ellas y el precio de las mismas. Si el usuario selecciona cualquiera de estas, deberá de ser redireccionado a la página de registro, automáticamente seleccionando el tipo de suscripción que eligió. 

https://www.spotify.com/gt/premium/
https://search.muz.li/inspiration/pricing-and-plan-page-inspiration/

### Registro
En el registro del usuario se deben requerir únicamente un correo electrónico, una contraseña con confirmación, y el tipo de suscripción que el usuario desea. Si el tipo de suscripción es premium, el usuario también deberá ingresar su número de tarjeta, código CVV y fecha de expiración

### Login
La página de Login permite al usuario ingresar al sistema, se necesita únicamente un correo electrónico y una contraseña para esto. Además debe tener un botón para realizar el proceso de reinicio de contraseña que se detalla a continuación.

### Reiniciar contraseña
Se desea permitir a los usuarios enviar un correo de recuperación de contraseña. La lógica es la siguiente: 

1. El usuario selecciona recuperar contraseña.
2. El usuario ingresa su correo electrónico
3. Si no hay ningún usuario con el correo, deberá mostrar un mensaje de error.
4. Si el correo está vinculado a un usuario, deberá enviar un número de 6 dígitos que será su código de verificación.
5. En la pantalla se requerirá el código de verificación
6. Si el código es incorrecto, se mostrará un error.
7. Si el código es correcto, se permitirá al usuario cambiar su contraseña.
8. Finalmente, se redireccionará al usuario nuevamente a la página de Login.

https://rhyce.dev/2021/08/07/how-to-send-an-email-from-sendinblue-with-node-js/
https://levelup.gitconnected.com/how-to-send-emails-from-node-js-with-sendinblue-c4caacb68f31

### Perfil de usuario
El usuiario podrá acceder a una página para ver sus datos, cancelar o cambiar su suscripción, y verificar el historial de canciones que ha reproducido. 

### Historial de reproducción
En el perfil del usuario se mostrará el detalle de cada una de las canciones que el usuario ha reproducido. Esta lista servirá como una nueva lista de reproducción y deberá mostrarse tanto en el perfil del usuario como en una página por si sola. Cada canción debe aparecer solamente una vez en esta lista.

## Migración a la nube

En esta fase se realizará la migración de los servicios a la nube que provee GCP.  El diagrama general de la nueva arquitectura en la nube se detalla en este apartado.

> Nota: la arquitectura del ambiente de desarrollo sigue siendo válida, sin embargo esta les será útil únicamente para probar su funcionalidad en su ambiente local de desarrollo. La migración a la nube tiene como objetivo tener un ambiente de producción diferente al de su máquina local.

### Diagrama general de la arquitectura en el ambiente de producción

El objetivo de migrar a la nube es implementar un ambiente de producción, donde los usuarios finales podrán acceder y utilizar el sistema por completo. 

![Diagrama general](./img/ayd2_diagrama-Diagrama%20Fase%202.drawio.png)

### Explicación del diagrama

#### APP ENGINE

Se almacenará la interfaz del usuario en App Engine, un servicio que provee Google para desplegar aplicaciones web. 

* https://javascript.plainenglish.io/quickly-deploy-your-react-app-on-googles-app-engine-6bb97480cc9c
* https://www.codingdeft.com/posts/react-deploy-google-cloud-app-engine/
* https://www.doit-intl.com/deploying-a-react-app-to-googles-app-engine/

#### GOOGLE KUBERNETES ENGINE

El backend completo de la aplicación será desplegado en un clúster de Kubernetes, utilizando GKE. 

* https://kubernetes.io/docs/tasks/configure-pod-container/translate-compose-kubernetes/
* https://loft.sh/blog/docker-compose-to-kubernetes-step-by-step-migration/
* https://itoutposts.com/blog/advantages-and-best-practices-for-docker-to-kubernetes-migration/

En este únicamente se incluirán los siguientes servicios:
1. El servicio de auth
2. El servicio de account
3. El servicio de streaming
4. El reverse proxy de nginx

> **No se deben incluir la base de datos SQL ni la base de datos cache.**

#### COMPUTE ENGINE
Se creará una pequeña máquina virtual que tendrá instalado Redis para ser utilizado como base de datos de cache. Esta base de datos será únicamente accesible por la API de Kubernetes.

#### CLOUD STORAGE Y CLOUD SQL
Ambos servicios seguirán funcionando de igual forma que en la fase #1. Se debe restringir el acceso a Cloud SQL para que únicamente GKE pueda acceder a ella. 

#### CONTAINER REGISTRY
Este servicio permite subir imágenes de contenedores, el cual utilizarán cada vez que realicen un cambio a **develop**. Pueden utilizar el Container Registry de Gitlab o de GCP. 

* https://docs.gitlab.com/ee/user/packages/container_registry/
* https://cloud.google.com/container-registry/docs/pushing-and-pulling

#### CLOUD MONITORING
Este servicio permite crear Dashboards para la visibilidad de la utilización de recursos tanto de GKE como de GCE. Es necesario realizar dos dashboards, uno para el clúster de Kubernetes que contiene la API, y el otro para la máquina virtual de Compute Engine que contiene la base de datos cache. La información que contendrá estos dashboards queda a discreción del grupo.

* https://cloud.google.com/stackdriver/docs/solutions/gke/observing?hl=es-419
* https://medium.com/google-cloud/gke-monitoring-84170ea44833
* https://cloud.google.com/monitoring?hl=es

## Testing
Uno de los objetivos más importantes de esta fase es realizar diversas pruebas a las funcionalidades que agregamos. El coverage para esta fase es únicamente del 70%, sin embargo la ponderación de la misma será incrementada. 

### Unit testing
Toda la funcionalidad anterior y nueva debe contar con pruebas unitarias. *Se dará una mayor ponderación a las pruebas unitarias del cliente que a las del backend.*

> Para esta fase, aparte de usar Mocks, deberán utilizar Spy **para al menos 3 tests.**

### Integration testing - client
Deberán realizar pruebas de integración a su aplicación. Cada vista debe tener al menos una prueba de integración. Las vistas a considerar son:

1. Login
2. Vista de Suscripción
3. Vista del radio

https://www.youtube.com/watch?v=4UQ1beAJL7o
https://stackshare.io/stackups/enzyme-vs-jasmine
https://revelry.co/insights/development/react-testing-with-jasmine/
https://styleguide.pivotal.io/guides/unit-testing-with-jasmine/
https://jasmine.github.io/
https://dev.to/alextomas80/hacer-test-de-componentes-en-react-con-enzyme-7l1
https://enzymejs.github.io/enzyme/

### Integration testing - API
Deberán realizar pruebas de integración a sus endpoints. Deben tener al menos 3 pruebas de integración para diferentes endpoints de la API. Al menos una de las pruebas debe ser realizada utilizando un POST. 

https://www.toptal.com/nodejs/nodejs-guide-integration-tests
https://www.freecodecamp.org/news/end-point-testing/

### Configuración de SonarQube

Al menos una persona del grupo debe de instalar SonarQube en su máquina de desarrollo, y configurarlo para realizar un análisis del código de las APIs. Las métricas a tomar en cuenta son:

1. Code Complexity
2. Duplications
3. Maintainability (Code-smells, technical debts)
4. Reliability (Bugs, Reliability Rating)
5. Security (Vulnerabilities, Security Rating)
6. Code Size, % comments, functions, statements

https://medium.com/swlh/nodejs-code-evaluation-using-jest-sonarqube-and-docker-f6b41b2c319d
https://morioh.com/p/578ae2a50596
https://www.youtube.com/watch?v=ZJyvk2HMFF4&ab_channel=JoaoVictor
https://docs.sonarqube.org/latest/user-guide/metric-definitions/

## Consideraciones

- Utilizar los lenguajes, frameworks y herramientas detalladas en el enunciado.
- Se trabajará en equipos no mayores a 5 personas.
- Copias totales o parciales tendrán una nota de 0 puntos y será reportado a las autoridades correspondientes.
- Deben realizar al menos 2 sprints en cada fase. 
- Se revisará que hayan utilizado Slack para la comunicación de su equipo.
- Se revisará que hayan creado correctamente las historias y tickets en Jira. 
- Se revisará que hayan trabajado correctamente con Git-Flow.
- Todas las dudas se resolverán en el foro correspondiente en UEDi o en el periodo de clase. 

## Fechas de entrega

- PRIMERA FASE: **13 de junio de 2022**
- SEGUNDA FASE: **21 de junio de 2022**
- TERCERA FASE: **30 de junio de 2022**