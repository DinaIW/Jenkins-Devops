# Utilisation de l'image Python 3.8 slim comme base
FROM python:3.8-slim

# Définition du répertoire de travail dans l'image
WORKDIR /app

# Copie du requirements.txt spécifique à movie-service dans le répertoire de travail
COPY ./movie-service/requirements.txt .

# Installation des dépendances Python spécifiées dans requirements.txt
RUN apt-get update \
    && apt-get install gcc -y \
    && apt-get clean

RUN pip install -r requirements.txt \
    && rm -rf /root/.cache/pip

# Copie du reste des fichiers de l'application
COPY ./movie-service .

# Commande par défaut pour démarrer l'application
CMD [ "python", "app.py" ]
