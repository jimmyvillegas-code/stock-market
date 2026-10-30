# Stock Market – Despliegue de Servicios de Red

Taller 1 de **Computación Distribuida y Frameworks para Desarrollo de Aplicaciones Inteligentes** – Universidad del Valle.

Web service en **Python + Flask** que consulta el histórico de precios de una acción (por defecto `GOOGL`) usando [yfinance](https://pypi.org/project/yfinance/), empaquetado en un contenedor **Docker** y desplegado en una instancia **AWS EC2**.

## Archivos

| Archivo | Descripción |
|---|---|
| `read_value_yf.py` | Script que descarga los datos del stock desde Yahoo Finance |
| `app.py` | Web service Flask que expone los datos como tabla HTML en `/` |
| `requirements.txt` | Dependencias (`yfinance`, `flask`) |
| `Dockerfile` | Imagen del web service (versión 2.0) |
| `Dockerfile.cli` | Imagen del script por línea de comandos (versión 1.0) |

## Versión 1.0 – Línea de comandos

```bash
docker image build -f Dockerfile.cli -t stock-market:1.0 .
docker container run --rm stock-market:1.0                 # GOOGL
docker container run --rm -e STOCK=AMZN stock-market:1.0   # otro stock
```

## Versión 2.0 – Web service

```bash
docker image build -t stock-market:2.0 .
docker container run --rm --name stock-market -p 5000:5000 stock-market:2.0
```

Abrir http://localhost:5000

> En macOS el puerto 5000 lo usa AirPlay; en ese caso usar `-p 5001:5000` y abrir http://localhost:5001

## Publicación en Docker Hub

Imagen multi-arquitectura (`amd64` para EC2 y `arm64` para Mac con chip Apple):

```bash
docker buildx build --platform linux/amd64,linux/arm64 -t jialviga00/stock-market:2.0 --push .
```

## Despliegue en AWS EC2

En una instancia Ubuntu con tráfico HTTP (puerto 80) habilitado:

```bash
sudo apt-get update && sudo apt-get install -y docker.io
sudo usermod -aG docker ubuntu   # cerrar y volver a abrir la sesión SSH
docker container run -d --rm --name stock-market -p 80:5000 jialviga00/stock-market:2.0
```

El servicio queda disponible en `http://<DNS-público-de-la-instancia>`.
