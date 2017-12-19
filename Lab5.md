Issue#26

Seguimos el tutorial de esta página:
https://simpleisbetterthancomplex.com/tutorial/2017/08/01/how-to-setup-amazon-s3-in-a-django-project.html
Nos saltamos el setup de S3 porque ya está listo
Instalar boto3 y django-storages con pip (osea, lo agregamos al requirements.txt)
Agregamos 'storages' a INSTALLED_APPS en mysite/settings.py
Copiamos los comandos que definen el AWS_S3 y lo modificamos cin nuestras credenciales (el acesss_key_id y la secret_access_key) no lo anotamos en el repositorio, pero sí en las máquinas
Agregamos la línea "AWS_S3_SECURE_URLS = False" para que s3 no use tls
Guardamos y salimos
Modificamos ranger para mostrar un html con una imágen, y así probar que s3 funciona
git commit y git push
git pull en las máquinas
En las máquinas agregamos el acesss_key_id y la secret_access_key
Creamos nuevamente la imagen con docker compose up --build -d
Al crearse el container, se ejecuta "python manage.py collectstatic --noinput" que sincroniza los statics con S3

Issue#27
Issue#28
