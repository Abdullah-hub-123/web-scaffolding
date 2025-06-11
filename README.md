# web-scaffolding (Flask)

# app_generica.py

from flask import Flask, render_template, request, url_for, session
import random

from db_handler import setup_database, save_record, get_records_by_user
app = Flask(__name__)
app.secret_key = "a_generic_secret_key"

# Prepara la base de datos al iniciar la aplicación
setup_database()

# Ruta principal para el login de usuario
    @app.route("/", methods=['GET', 'POST'])
    def login():
        if request.method == 'POST':
            # Recoge las credenciales del formulario
            username = request.form.get('username')
            password = request.form.get('password')
    
            # REEMPLAZAR: Lógica de autenticación real aquí
            if password == '1234':
                session['user_id'] = username
                return render_template('dashboard.html')
            else:
                # Muestra un mensaje de error si la autenticación falla
                error_msg = "Acceso denegado. Por favor, inténtelo de nuevo."
                return render_template('login.html', error_message=error_msg)
        
        # Muestra el formulario de login en la primera visita (GET)
        return render_template('login.html')

# Ruta para el panel principal o menú de opciones
    @app.route("/dashboard", methods=['GET'])
    def dashboard():
        if 'user_id' in session:
            return render_template('dashboard.html', username=session['user_id'])
        else:
            # Si no hay sesión, redirige al login
            return render_template('login.html')

# Ruta para una funcionalidad genérica de la aplicación
    @app.route("/feature_one", methods=['GET'])
    def feature_one():
        if 'user_id' in session:
            return render_template('feature_one.html')
        else:
            return render_template('login.html')

# Ruta para configurar e iniciar una tarea interactiva
    @app.route("/interactive_task", methods=['GET', 'POST'])
    def interactive_task():
        if 'user_id' not in session:
            return render_template('login.html')
    
        if request.method == 'POST':
            # El usuario ha enviado la configuración de la tarea
            mode = request.form.get('mode')
            
            # Genera datos basados en el modo seleccionado
            if mode == 'easy':
                value1 = random.randint(0, 9)
                value2 = random.randint(0, 9)
            else:
                value1 = random.randint(0, 99)
                value2 = random.randint(0, 99)
            
            # Guarda los datos de la tarea en la sesión
            session['value1'] = value1
            session['value2'] = value2
            session['mode'] = mode
            
            # Muestra la interfaz de la tarea
            return render_template('task_interface.html', username=session['user_id'], v1=value1, v2=value2, current_mode=mode)
    
        # Si es GET, muestra la página para configurar la tarea
        return render_template('task_setup.html', username=session['user_id'])

# Ruta para procesar la respuesta enviada por el usuario
    @app.route("/submit_answer", methods=['POST'])
    def process_submission():
        if 'user_id' not in session:
            return render_template('login.html')
    
        # Obtiene la respuesta del usuario y los datos guardados en sesión
        user_input = int(request.form.get('user_answer'))
        value1 = session.get('value1')
        value2 = session.get('value2')
        mode = session.get('mode')
        
        # Compara la respuesta y establece un flag de resultado
        # REEMPLAZAR: Con tu propia lógica de verificación
        if user_input == (value1 + value2):
            result_flag = 1 # 1 para correcto
        else:
            result_flag = 0 # 0 para incorrecto
    
        # Guarda el resultado en la base de datos
        save_record(session['user_id'], mode, result_flag)
    
        # Obtiene el historial de registros para el usuario
        history = get_records_by_user(session['user_id'])
        
        # Muestra la página de resultados
        return render_template('result_page.html', username=session['user_id'], result=result_flag, records=history)

# Punto de entrada para ejecutar la aplicación
    if __name__ == "__main__":
        app.run(debug=True)


_____________________________________________________________________________________________________________________________________________________________________________

from flask import Flask, render_template, request, url_for
# REEMPLAZAR: con el nombre de tu módulo para la base de datos
from db_utils import db_manager 

# --- Configuración de la Aplicación ---
    app = Flask(__name__)
    app.secret_key = 'a_very_secret_key'

# REEMPLAZAR: Credenciales de administrador o lógica de autenticación
    ADMIN_USER = "admin"
    ADMIN_PASS = "admin"

# --- Definición de Rutas ---

# Ruta para el login inicial
    @app.route("/", methods=["GET", "POST"])
    def main_route():
        if request.method == "POST":
            # Recoge los datos del formulario de login
            input_user = request.form.get('username')
            input_pass = request.form.get('password')
    
            # Comprueba las credenciales
            if input_user == ADMIN_USER and input_pass == ADMIN_PASS:
                # Si son correctas, muestra el menú principal
                return render_template('main_menu.html')
            else:
                # Si no, muestra un mensaje de error en la página de login
                error_message = 'Credenciales incorrectas. Inténtelo de nuevo.'
                return render_template("login.html", status_message=error_message)
        
        # Muestra la página de login en la primera visita (GET)
        return render_template("login.html")

# Ruta para añadir un nuevo registro a la base de datos
    @app.route("/add", methods=["GET", "POST"])
    def add_entry():
        if request.method == "POST":
            # Recoge los datos del formulario de creación
            record_id = request.form["record_id"]
            value = float(request.form["value"])
            
            # Llama a la función del gestor de BD para insertar el dato
            is_successful = db_manager.create_entry(record_id, value)
            
            # REEMPLAZAR: con tu lógica para mostrar un resultado
            # Ejemplo: mostrar una imagen o mensaje diferente según el valor
            status_flag = value >= 5 
            image_file = "success.jpg" if status_flag else "failure.jpg"
            
            return render_template("result.html", success=status_flag, image=image_file, op_success=is_successful)
        
        # Muestra el formulario para añadir un nuevo registro
        return render_template("add_form.html")

# Ruta para modificar un registro existente
    @app.route("/update", methods=["GET", "POST"])
    def update_entry():
        if request.method == "POST":
            # Recoge los datos del formulario de actualización
            record_id = request.form["record_id"]
            new_value = float(request.form["value"])
            
            # Llama a la función del gestor de BD para actualizar
            db_manager.modify_entry(record_id, new_value)
            
            # Muestra una página de confirmación
            success_message = f"Registro actualizado para el ID: {record_id}"
            return render_template("result.html", status_message=success_message, updated=True)
        
        # Muestra el formulario para actualizar un registro
        return render_template("update_form.html")

# Ruta para mostrar todos los registros de la base de datos
    @app.route("/view_all")
    def display_all_entries():
        # Obtiene todos los registros desde el gestor de BD
        all_records = db_manager.get_all_entries()
        # Pasa los registros a una plantilla para mostrarlos
        return render_template("view_data.html", records=all_records)

# --- Punto de Entrada ---
    if __name__ == "__main__":
        # Inicializa la base de datos antes de arrancar la app
        db_manager.initialize_db()
        app.run(debug=True)

__________________________________________________________________________________________________________________________________________________________________________________________________
