FROM python:3.11-slim

WORKDIR /app

# Installation des dependances
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

# Copie du code source
COPY . .

# Initialisation de la base de donnees
WORKDIR /app/bad
RUN python db_init.py || true

EXPOSE 5000

CMD ["python", "vulpy.py"]
