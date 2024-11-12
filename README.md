# tutorial flask

## ¿Qué es Flask?
Flask Python es un sencillo framework web, idóneo tanto para principiantes como para profesionales. Flask incluye únicamente lo esencial, pero los usuarios pueden implementar bibliotecas externas para ampliar sus funcionalidades.

## 1.- Iniciar un proyecto y su configuración

* Crea la carpeta del proyecto, esto lo puedes hacer desde el navegador de archivos de tu computadora o desde la línea de comando.
* Abre la carpeta con VS Code
* Abre la terminal en VS Code
* Instala e inicializa el entorno virtual
```
virtualenv venv
```
```
source venv/bin/activate
```
* Ahora vamos a instalar los paquetes necesarios:
  * Instalamos Flask
  * Instalamos psycopg2-binary que nos permitirá conectarnos con el servidor de la base de datos
  * Instalamos Flask-SQLAlchemy que es el ORM con el que vamos a interactuar con la base de datos
  * Instalamos Flask-Mail que nos permitirá enviar correos
  * Instalamos flask_cors que nos permitirá trabajar con los HEADERS y CORS
  * Instalamos PyJWT que nos permitirá trabajar con tokes y autenticación
  * Instalamos dotenv que nos permitirá trabajar con variables de entorno
```
pip install Flask psycopg2-binary Flask-SQLAlchemy Flask-Mail flask_cors PyJWT
```

## 2.- Estructura del proyecto
Organiza tu proyecto con esta estructura básica

```
my_flask_app/
├── app.py           # Archivo principal de la aplicación
├── config.py        # Configuración de la app (base de datos, correo)
├── models.py        # Definición de los modelos de la base de datos
├── .env
└── requirements.txt # Dependencias del proyecto
```

## 3.- Configuración de la aplicación y base de datos
Para trabajar con la base de datos necesitas crear una cuenta en [neon.tech](https://neon.tech) y ahí crear una base de datos gratuita, te dará el comando completo que deberás copiar y agregar en tu archivo ```.env```

```
# config.py
from dotenv import load_dotenv #Importamos el paquete dotenv
import os
load_dotenv() # Cargamos el archivo con las variables de entorno y vamos a usar os.getenv('nombre_variable') para acceder al dato

class Config:
    SECRET_KEY = os.getenv('SECRET_KEY') # os.environ.get('SECRET_KEY') or 'una_llave_secreta'
    SQLALCHEMY_DATABASE_URI = os.getenv('BBDD_URL') # os.environ.get('BBDD_URL')
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    JWT_SECRET_KEY = os.getenv('JWT_SECRET_KEY') #or 'una_llave_secreta_para_jwt'  # Nueva clave para JWT

    # Configuración para enviar correo
    MAIL_SERVER = 'smtp.dreamhost.com'
    MAIL_PORT = 587
    MAIL_USE_TLS = True
    MAIL_USERNAME =  os.getenv('EMAIL_USER')  # Añade tu email #'fernando@hackademy.mx' 
    MAIL_PASSWORD =  os.getenv('EMAIL_PASS')  # Añade tu password #'2yZtrUQK'
```
En nuestro archivo ```.env``` vamos a guardar los datos de nuestras variables:
```
EMAIL_USER=dirección de correo de la que mandaremos mails
EMAIL_PASS=contraseña del correo
JWT_SECRET_KEY=en gogle hay generadores de secrets para JWT
BBDD_URL='el comando que te da neon'
```

## 4.- Nuestro primer modelo en ```models.py```
En el archivo ```models.py``` importamos SQLAlquemy que será nuestro ORM, con esto podemos trabajar con la base de datos a traves de clases.
Nuestro primer modelo será el de usuarios.
```
# models.py
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

class User(db.Model):
    __tablename__ = 'users'
    
    id = db.Column(db.Integer, primary_key=True)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password = db.Column(db.String(60), nullable=False)
    otp = db.Column(db.String(6), nullable=True)
    is_verified = db.Column(db.Boolean, default=False)
```

## 5.- Crear aplicación
Vamos a crear nuestra aplicación, para esto deberemos de modificar el archivo ```app.py```

```
# app.py
from flask import Flask
from config import Config
from models import db
import random
from flask import request, jsonify
from models import User
from flask_mail import Mail, Message

app = Flask(__name__)
app.config.from_object(Config)
db.init_app(app)

with app.app_context():
    db.create_all()
```
## 6.- Mi primer EndPoint
Ahora vamos a desarrollar la primer funcionalidad de nuestro proyecto, un EP que nos permita registrar usuarias en nuestra base de datos, para esto vamos a agregar código despues del método ```db.create_all()``. Recuerda que para python lo importante es la identación así que nuestro código, del EP, debe de empezar al inicio de la línea.

```
mail = Mail(app)

def generate_otp():
    return str(random.randint(100000, 999999))

@app.route('/register', methods=['POST'])
def register():
    data = request.get_json()
    email = data.get('email')
    password = data.get('password')

    # Verifica si el usuario ya existe
    if User.query.filter_by(email=email).first():
        return jsonify({'message': 'El usuario ya existe.'}), 400

    # Crea el usuario con OTP
    otp = generate_otp()
    new_user = User(email=email, password=password, otp=otp)
    db.session.add(new_user)
    db.session.commit()

    # Envía el correo electrónico con OTP
    msg = Message('Verificación de tu cuenta', sender=app.config['MAIL_USERNAME'], recipients=[email])
    msg.body = f'Tu OTP es: {otp}'
    mail.send(msg)

    return jsonify({'message': 'Usuario registrado. Revisa tu correo para la verificación.'}), 201
```
Este endpoint:

* Recibe el correo electrónico y la contraseña del usuario.
* Verifica si el correo ya está registrado.
* Genera un OTP y lo envía por correo.
* Devuelve un mensaje de confirmación.

Con esto ya tienes una estructura básica para el registro. En el próximo paso, agregaremos la lógica para que el usuario valide su OTP y active su cuenta.

## 7.- Ejecutar mi proyecto
Para correr tu proyecto y hacer pruebas necesitas ejecutar, en la terminal de VS Code, el siguiente comando ```python -m flask run ``` con esto tu proyecto estará corriendo.
La terminal te dará una url tipo ```127.0.0.1:5000``` que es donde está corriendo tu proyecto, deberás abrir Postman y configurar un EP POST para pasarle una URL tipo ```127.0.0.1:5000/register```.

En Postman debes de configurar para el el body sea ```raw``` y ```json```, para pasar un json con esta estructura:
```
{
    "email":"",
    "password":"",
    "date_birth":""
}
```

## Y listo, tendrás tu primer proyecto.
