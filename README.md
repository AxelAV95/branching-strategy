# Escenario Real: Desarrollo de la nueva funcionalidad de recomendaciones de productos

Imaginemos que Ana, una desarrolladora, está trabajando en una nueva funcionalidad para la aplicación web de e-commerce que ofrece "Recomendaciones de Productos" a los usuarios en función de su historial de compras. A continuación, detallamos cómo este cambio se maneja utilizando una estrategia de branching con un flujo de Continuous Delivery (CD), empleando herramientas como Jenkins y Argo CD para los despliegues.

# 1. Desarrollo en la Rama Feature
Desarrollo y Pruebas en Entorno de Desarrollo
Ana comienza a trabajar en la nueva funcionalidad creando una rama feature/recommendations desde la rama main. A medida que realiza cambios en el backend y frontend, realiza múltiples commits. Cada vez que realiza un push al repositorio, un webhook se activa y dispara un pipeline en Jenkins. Este pipeline ejecuta las siguientes acciones:

- Ejecución de pruebas unitarias.
- Construcción de la imagen Docker de la aplicación.
- Despliegue automático en el clúster de desarrollo de Kubernetes correspondiente a la rama feature/recommendations.

En este momento, el pipeline también ejecuta un script automatizado que actualiza el manifiesto solo en el entorno de desarrollo, reflejando los cambios en el clúster de Kubernetes dedicado a este entorno.

# 2. Integración en la Rama Main (Staging)

Fusión con Main y Despliegue en Staging

Una vez que Ana termina de implementar la funcionalidad y pasa las pruebas iniciales en el entorno de desarrollo, crea un pull request para fusionar su rama feature/recommendations con la rama main. Después de que el equipo de revisión aprueba el código, el merge se lleva a cabo. Al hacer este merge, un webhook activa otro pipeline de Jenkins que realiza las siguientes acciones:

- Despliega la nueva versión en el entorno de staging, donde se realizarán pruebas de integración.
- Ejecuta pruebas funcionales y de integración.

Al igual que en el paso anterior, Jenkins ejecuta un script que actualiza solo el manifiesto de Kubernetes correspondiente al entorno staging. Esto asegura que los cambios se apliquen únicamente al entorno de staging sin afectar el entorno de desarrollo.

# 3. Creación de la Rama Release (Preproducción)

Preparación para Producción y Despliegue en UAT
Cuando el equipo considera que la funcionalidad está lista para su implementación final, el DevOps (Carlos) crea una rama release/1.2.0 desde la rama main.

git checkout main
git pull
git checkout -b release/1.2.0

Al crear la rama release, el pipeline correspondiente en Jenkins se activa automáticamente y realiza las siguientes acciones:

Despliega la versión en el entorno UAT (User Acceptance Testing) para las pruebas de aceptación final.
Ejecuta todas las pruebas necesarias en UAT para validar que todo funcione correctamente en un entorno similar a producción.
En este punto, Jenkins actualiza el manifiesto correspondiente al entorno UAT mediante un script automatizado, asegurándose de que la configuración de Kubernetes en ese entorno esté alineada con los cambios más recientes.

# 4. Validación en UAT y Promoción a Producción
Pruebas de Aceptación
En UAT, el equipo de QA y usuarios clave prueban la funcionalidad en condiciones de casi producción. Si todo va bien y no se detectan problemas, el equipo de QA da su aprobación para proceder al despliegue en producción.

Acción del DevOps para Desplegar a Producción
Una vez que se ha validado la funcionalidad en UAT, el DevOps o el equipo de Release realiza los siguientes pasos para promover la nueva versión a producción:

- Revisión y Confirmación de las Pruebas en UAT: El equipo de QA ha validado que la funcionalidad está funcionando correctamente en el entorno de UAT y ha aprobado el cambio.
- Fusión de la Rama Release a Main: El DevOps realiza un merge manual de la rama release/1.2.0 a main (si no se realiza de forma automática). Esto garantiza que el código que se validó en UAT esté actualizado y listo para producción.
git checkout main
git pull
git merge release/1.2.0

Activación del Pipeline de Producción: Una vez que el merge se completa, se activa un webhook que inicia el pipeline de producción en Jenkins. Este pipeline realiza lo siguiente:

- Construye las imágenes Docker correspondientes a la nueva versión de la aplicación.
- Despliega los cambios al entorno de producción, es decir, al clúster de Kubernetes de producción.
- Argo CD sincroniza los cambios del repositorio Git con el clúster de producción, aplicando el manifiesto actualizado de Kubernetes.
  
# 5. Despliegue Automático a Producción
Despliegue a Producción y Verificación Final
Cuando el pipeline de producción se ejecuta, Argo CD se encarga de gestionar la sincronización entre el repositorio Git y el clúster de Kubernetes de producción. Asegura que el manifiesto de producción refleje los cambios y que todo esté configurado correctamente.

En este punto, Argo CD realiza el despliegue de la nueva funcionalidad en el entorno de producción de manera automática, sin intervención manual.

# 6. Uso de Argo CD en Todo el Flujo
GitOps y Despliegue Automático
En todo el proceso, Argo CD asegura que los manifiestos de Kubernetes estén sincronizados con los entornos correspondientes:

- Desarrollo: Argo CD mantiene el clúster de desarrollo actualizado con los cambios de la rama feature.
- Staging: Argo CD maneja el despliegue de la rama main en el entorno de staging.
- UAT: Argo CD maneja el despliegue de la rama release en el entorno de UAT.
- Producción: Finalmente, Argo CD garantiza que el clúster de producción reciba la nueva versión después de que el DevOps haya realizado el merge de release a main y el pipeline de producción haya sido ejecutado.
  
# Resumen del Flujo Completo
- Desarrollo en Feature: Ana crea y desarrolla una nueva funcionalidad en una rama feature y la despliega automáticamente en el clúster de desarrollo.
- Integración en Main: Los cambios de la rama feature se fusionan con main y se despliegan en el entorno de staging para pruebas de integración.
- Preparación para Producción: El DevOps crea una rama release para validar en UAT. Las pruebas de aceptación se realizan, y los cambios se aprueban.
- Promoción a Producción: El DevOps realiza el merge de la rama release a main, lo que activa el pipeline de producción, desencadenando el despliegue en el entorno de producción.
- Despliegue Automático con Argo CD: Argo CD asegura que los manifiestos de Kubernetes estén sincronizados en todos los entornos y gestiona el despliegue automático en producción.
