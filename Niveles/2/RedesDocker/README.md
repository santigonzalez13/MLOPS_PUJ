# Redes en Docker

Explicación de cómo funcionan las redes en Docker, cómo se configuran en `docker-compose.yml` y cómo interactúan entre servicios dentro de un entorno de contenedores. Siga los siguientes pasos si desea crear/entender el funcionamiento basico de una red de docker.

---

## **1. Conceptos Fundamentales**

Las redes en Docker permiten la comunicación entre contenedores** sin exponer servicios al exterior

### **1.1 Redes en Docker**

Docker proporciona un sistema de redes internas que permite que los contenedores se comuniquen entre sí sin exponer los servicios al host (máquina anfitriona).

Tipos de redes en Docker:

1. **Bridge (Predeterminada)**: Permite la comunicación entre contenedores en la misma red interna de Docker.
2. **Host**: Usa directamente la red del host (sin aislamiento).
3. **None**: Sin conectividad de red.
4. **Overlay**: Para comunicación entre múltiples hosts con Docker Swarm.
5. **Macvlan**: Permite asignar direcciones IP específicas a los contenedores.

En `docker-compose`, normalmente usamos la red `bridge` para conectar servicios entre sí.

---

## **2. Comunicación entre Contenedores en una Red Docker**

Creamos un escenario donde un servicio de Python actúa como cliente HTTP y otro como servidor API con **FastAPI**.

### **Estructura del Ejemplo**

```text

docker_network_example/
│── docker-compose.yml
│── app_server/
│   ├── server.py
│   ├── Dockerfile
│── app_client/
│   ├── client.py
│   ├── Dockerfile
```

---

### **2.1 Servidor API con FastAPI**

El servidor expone un endpoint en `http://server:8000/hello`.

#### **Archivo: `app_server/server.py`**

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/hello")
def hello():
    return {"message": "Hola desde el servidor!"}
```

#### **Dockerfile del Servidor (`app_server/Dockerfile`)**

```Dockerfile
FROM python:3.9
WORKDIR /app
COPY server.py .
RUN pip install fastapi uvicorn
CMD ["uvicorn", "server:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

### **2.2 Cliente que Consume el API**

El cliente intenta hacer una solicitud HTTP al servidor.

#### **Archivo: `app_client/client.py`**

```python
import requests
import time

server_url = "http://server:8000/hello"

time.sleep(5)  # Espera a que el servidor se inicie

try:
    response = requests.get(server_url)
    print("Respuesta del servidor:", response.json())
except Exception as e:
    print("Error:", e)
```

#### **Dockerfile del Cliente (`app_client/Dockerfile`)**

```Dockerfile
FROM python:3.9
WORKDIR /app
COPY client.py .
RUN pip install requests
CMD ["python", "client.py"]
```

---

### **2.3 Configuración de Redes en `docker-compose.yml`**

Ahora conectamos ambos servicios en la misma red.

#### **Archivo: `docker-compose.yml`**

```yaml
version: '3.8'

services:
  server:
    build: ./app_server
    container_name: server
    networks:
      - my_network

  client:
    build: ./app_client
    container_name: client
    depends_on:
      - server
    networks:
      - my_network

networks:
  my_network:
    driver: bridge
```

---

### **2.4 Prueba**

Ejecutar el siguiente comando:

```bash
docker-compose up --build
```

Salida esperada del cliente:

```json
Respuesta del servidor: {'message': 'Hola desde el servidor!'}
```

Se creó una red llamada `my_network`. Ambos servicios (`server` y `client`) están conectados a esa red. El cliente puede referenciar al servidor con `http://server:8000/hello` sin necesidad de conocer su IP.
