# Invisibility Cloak
Práctica adicional para la electiva de Ubicuos/IoT de la Universidad del Cauca

La guía de la práctica se puede encontrar en el siguiente enlace: 
https://docs.google.com/document/d/1sLcOA4Ayq_GtO8Cb1-MAYYJ0V_fV1t3uTy9flLXzwFw/edit?usp=sharing


# Simulación de capa de invisibilidad con Raspberry y OpenCV

En esta guía presentaremos cómo simular, mediante una cámara y una Raspberry, un capa de invisibilidad haciendo uso de una librería de python para trabajar con “computer vision” llamada [OpenCV](https://opencv.org/).

La visión por computador o visión artificial es el **conjunto de herramientas y métodos que permiten obtener, procesar y analizar imágenes del mundo real** con la finalidad de que puedan ser tratadas por un ordenador. Esto permite automatizar una amplia gama de tareas al aportar a las máquinas la información que necesitan para la toma de decisiones correctas en cada una de las tareas en las que han sido asignadas.

## Materiales

- **Software**
    - Raspbian Buster (Versión usada: 4.19)
    - Python 3 (Versión usada: 3.7)
    - Librería de python OpenCV (Versión usada: 4.0)
- **Hardware**
    - Placa Raspberry 3 B
    - Unitec JNP-W119 (Cámara web baja definición)
    - Logitech Carl Zeiss Tessar HD 1080p (Cámara web alta definición)

## Instalaciones requeridas

En este tutorial, vamos a suponer que ya tienes una Raspberry Pi 3 B con Raspbian instalado, si no lo tienes puedes instalarlo desde el siguiente [link](https://www.raspberrypi.org/downloads/raspbian/).

Suponiendo que tu sistema operativo está actualizado, necesitaremos uno de los siguientes accesos para el resto de la guia:

  - *Acceso físico* a su Raspberry Pi 3 para que pueda abrir un terminal y ejecutar comandos
  - *Acceso remoto* a través de SSH o VNC.

La guía se realizará mediante VNC a través del programa “VNC Viewer” en una computadora portátil con windows 10. Si deseas usar este metodo mira el siguiente [tutorial](https://www.genbeta.com/paso-a-paso/como-manejar-tu-raspberry-pi-desde-cualquier-pc-o-movil-con-vnc).

A continuación se tomarán algunos pasos del siguiente tutorial “[Install OpenCV 4 on your Raspberry Pi](https://www.pyimagesearch.com/2018/09/26/install-opencv-4-on-your-raspberry-pi/)", se optará por usar los pasos más importantes y los que se usaron para el correcto funcionamiento de la práctica. Cualquier duda o inquietud referenciarse al tutorial tiene un apartado de preguntas y respuestas.

### Paso 1: Expandir el sistema de archivos en la Raspberry Pi

Lo primero que se debe hacer es ingresar a la interfaz de configuración de Raspberry mediante terminal.

    ~ sudo raspi-config

![alt text](https://github.com/LuisFelipeL/InvisibilityCloak/blob/master/images/1.jpg)

Seguido vamos a la opción “Advanced Options”. 

![alt text](https://github.com/LuisFelipeL/InvisibilityCloak/blob/master/images/2.jpg)

Se debe seleccionar la primera opción, “A1 Expand Filesystem", luego con las flechas escoge la opción ”<Finish>“, y reinicie su Raspberry Pi.
  
### Paso 2: Instalar las dependencias de OpenCV en su Raspberry Pi

Actualizamos nuestro sistema

    ~ sudo apt-get update && sudo apt-get upgrade
    
Instalamos paquete de herramientas de desarrollo

    ~ sudo apt-get install build-essential cmake unzip pkg-config
    
Instalamos paquetes de imágenes y videos, que son fundamentales para trabajar con archivos de imágenes y videos.

    ~ sudo apt-get install libjpeg-dev libpng-dev libtiff-dev
    ~ sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev
    ~ sudo apt-get install libxvidcore-dev libx264-dev
    
Instalamos el paquete GTK. Nuestro backend GUI.

    ~ sudo apt-get install libgtk-3-dev
    ~ sudo apt-get install libcanberra-gtk*
    
Instalamos paquetes de optimización numérica que será usada por OpenCV.

    ~ sudo apt-get install libatlas-base-dev gfortran
    
Y por último, instalaremos el paquete de desarrollo de python 3.

    ~ sudo apt-get install python3-dev
    
### Paso 3: Descargar OpenCV en tu Raspberry Pi

El siguiente paso es descargar OpenCV. descargaremos dos repositorios “opencv” y “opencv_contrib”, que contiene módulos y funciones adicionales.

    ~ cd ~
    ~ wget -O opencv.zip https://github.com/opencv/opencv/archive/4.0.0.zip
    ~ wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.0.0.zip
    
Una vez descargados, descomprimimos los archivos “.zip”.

    ~ unzip opencv.zip
    ~ unzip opencv_contrib.zip
    
Cambiar los nombre de los archivos a:

    ~ mv opencv-4.0.0 opencv
    ~ mv opencv_contrib-4.0.0 opencv_contrib

### Paso 4: Configurar un entorno virtual en python 3 para OpenCV

Descargamos e instalamos pip (“Python Package Manager”), el gestor de archivos de python.

    ~ wget https://bootstrap.pypa.io/get-pip.py
    ~ sudo python3 get-pip.py

Instalemos los paquetes para entornos virtuales de python mediante pip.

    ~ sudo pip install virtualenv virtualenvwrapper
    ~ sudo rm -rf ~/get-pip.py ~/.cache/pip

Con algún editor de texto (Ej: nano o vim) abrir el archivo “.profile” en la carpeta de usuario.

    ~ nano ~/.profile
    
Cuando se abra el archivo ve hasta el final y pega las siguientes líneas de código.

    # virtualenv and virtualenvwrapper
    export WORKON_HOME=$HOME/.virtualenvs
    export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
    source /usr/local/bin/virtualenvwrapper.sh

Ejecutamos el archivo mediante el comando source en el terminal.

    ~ source ~/.profile
    
Creamos el entorno virtual donde instalaremos OpenCV.

    ~ mkvirtualenv cv -p python3
    
Después de haber creado el entorno virtual, podemos entrar a él con las siguientes líneas de código.

    ~ source ~/.profile
    ~ workon cv
    (cv) ~
    
Por último, dentro del entorno virtual, instalaremos la librería de python para computación científica “NumPy” mediante pip

    (cv) ~ pip install numpy
    
### Paso 5: Compilar OpenCv en tu Raspberry Pi

En este paso vamos a compilar y configurar OpenCV en nuestra Raspberry Pi.

**NOTA:** Es es el paso más lento ya que las Raspberry debe trabajar con el procesador al maximo. Recomendación, mantenerla en un lugar ventilado ya que esta se calentará demasiado.

Volvamos a la carpeta “opencv” que descargamos y cree una carpeta con el nombre “build”.

    (cv) ~ cd ~/opencv
    (cv) ~/opencv$ mkdir build
    (cv) ~/opencv$ cd build
    (cv) ~/opencv/build$
    
Ahora ejecutemos Cmake con el siguiente contenido para la compilación de OpenCV.

    (cv) ~/opencv/build$ cmake -D CMAKE_BUILD_TYPE=RELEASE \
                            -D CMAKE_INSTALL_PREFIX=/usr/local \
                            -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules \
                            -D ENABLE_NEON=ON \
                            -D ENABLE_VFPV3=ON \
                            -D BUILD_TESTS=OFF \
                            -D OPENCV_ENABLE_NONFREE=ON \
                            -D INSTALL_PYTHON_EXAMPLES=OFF \
                            -D BUILD_EXAMPLES=OFF ..

Antes de comenzar la compilación, vamos a aumentar el espacio de intercambio de la Raspberry. Esto permitirá compilar OpenCV con los cuatro núcleos de Raspberry Pi sin que la compilación se cuelgue debido a que la memoria se agote.

Abre el archivo en la ubicación */etc/dphys-swapfile*.

    (cv) ~/opencv/build$ sudo nano /etc/dphys-swapfile
    
Y edita la variable “*CONF_SWAPSIZE*”, cambiar si valor predeterminado a 100 MB a 2048 MB.

    # set size to absolute value, leaving empty (default) then uses computed value
    #   you most likely don't want this, unless you have an special disk situation
    # CONF_SWAPSIZE=100
    CONF_SWAPSIZE = 2048

Luego de haber editado el archivo, guarde y reinicie el servicio de intercambio, parandolo y volviendolo a correr.

    (cv) ~/opencv/build$ sudo /etc/init.d/dphys-swapfile stop
    (cv) ~/opencv/build$ sudo /etc/init.d/dphys-swapfile start
    
Ahora ya estamos listo para compilar OpenCV.

**NOTA:** En algunos casos la raspberry se va a quedar pegada en unos porcentajes porque es donde va realizar más trabajo, dejarla que hasta que complete el 100% de la compilación. Tener paciencia.

    (cv) ~/opencv/build$ cd ~/opencv
    (cv) ~/opencv$ make -j4
    
Y finalmente instalaremos OpenCV.

    (cv) ~/opencv$ sudo make install 
    (cv) ~/opencv$ sudo ldconfig
    
NOTA: Vuelve abrir el archivo dphys-swapfile y cambia la variable “CONF_SWAPSIZE” otra vez a valor predeterminado (100 MB) y reinicia el servicio.

### Paso 6: Enlazar OpenCV con el entorno virtual

Creamos un enlace entre ficheros, para poder importar OpenCV. 

**NOTA:** Paso crítico, si estás usando otra versión de python cambiar las líneas de código a tu versión actual para evitar errores. Consejo, usar completado automático.

    (cv) ~/opencv$ cd ~/.virtualenvs/cv/lib/python3.7/site-packages/
    (cv) ~/.virtualenvs/cv/lib/python3.7/site-packages$ ln -s /usr/local/python/cv2/python-3.7/cv2.cpython-37m-arm-linux-gnueabihf.so cv2.so
    (cv) ~/.virtualenvs/cv/lib/python3.7/site-packages$ cd ~
    (cv) ~
    
### Paso 7: Comprobar la instalación de OpenCV en tu Raspberry

Para comprobar que OpenCV quedó instalado correctamente abra un terminal y realice lo siguiente

    (cv) ~ python
    Python 3.7 (default, Nov 27 2018, 23:36:35)
    [GCC 7.3.0]
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import cv2
    >>> cv2.__version__
    '4.0.0'
    >>> exit()
    (cv) ~

## Explicación del código

El código está compuesto por 8 partes, y estas son:
 
   - Importación de librerías

    import cv2
    import numpy as np
    import time

   - VideoCapture da la orden a OpenCV de trabajar con la cámara conectada a la raspberry, dejamos una tiempo para la inicialización de la misma y generamos una background o fondo de una captura del entorno para hacer uso de ella más adelante en el código.
   
    cap = cv2.VideoCapture(0)
    time.sleep(3)
    background=0
    for i in range(60):
        ret,background = cap.read()

   - Implementamos un ciclo while y usamos la función isOpen(), para mantener la grabación en curso mientras que este la cámara funcionando. Capturamos un frame y verificamos que se haya tomado.
   
    while(cap.isOpened()):
      ret, img = cap.read()
      if not ret:
        break
        
   - Le damos la orden a OpenCV de trabajar en la escala HSV (Hue, Saturation, Value) [6], que es otra escala parecida al RGB, esto se hace ya que los colores en RGB son muy sensibles a la luz.
   
    hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
    
   - Generamos máscaras en dos rangos de la escala HSV, entre 0 - 8° y 170° - 180°, con sus respectiva saturación y brillo en un arreglo numpy, que representa el color rojo para este caso. Luego son sumadas y se genera origina una máscara que detecta el color rojo.
   
    lower_red = np.array([0,150,40], np.uint8)
    upper_red = np.array([8,255,255], np.uint8)
    mask1 = cv2.inRange(hsv,lower_red,upper_red)
 
    lower_red = np.array([170,150,40], np.uint8)
    upper_red = np.array([180,255,255], np.uint8)
    mask2 = cv2.inRange(hsv,lower_red,upper_red)
 
    mask = mask1+mask2


   - Se aplica un filtro de suavizado gaussiano mediante la función medianBlur() a la máscara para mejorar la detección de los bordes del objeto. Se pasa la máscara por una función de dilatación para mejorar la detección. Si quieres saber más acerca de transformaciones morfológicas revisa el siguiente [link](https://unipython.com/transformaciones-morfologicas/). Por último, la función bitwise_not() invierte la máscara para detectar el fondo, lo opuesto a el objeto detectado.
   
    mask_medianBlur = cv2.medianBlur(mask, 13)
    mask_dilate = cv2.dilate(mask_medianBlur,np.ones((3,3),np.uint8),iterations = 2)
    mask_invert = cv2.bitwise_not(mask_dilate)

   - En este punto, ocurre la magia, la funciona bitwise_and() realiza una operación lógica AND entre la máscara y el frame que se captura en cada iteración del ciclo, como vemos en la primera línea de código, realiza la operación con la máscara invertida, por ello solo deja pasar el fondo y “elimina” el objeto del frame. En la segunda línea de código, toma el background o fondo, capturado al inicio del código, y realiza la operación con la máscara normal, lo cual, solo deja pasar el fondo con la forma del objeto en la máscara.
   
    delete_object = cv2.bitwise_and(img,img,mask=mask_invert)
    background_object = cv2.bitwise_and(background,background,mask=mask_dilate)

   - Finalmente, se realiza una suma matricial entre los dos frames, delete_object y background_object, y genera el frame final simulando que el objeto de color rojo desaparece, colocando un bac ground en reemplazo de este.
   
    final_output = cv2.addWeighted(delete_object,1,background_object,1,0)
    cv2.imshow('Original',img)
    cv2.imshow('Final',final_output)
