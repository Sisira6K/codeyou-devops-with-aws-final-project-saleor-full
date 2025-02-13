#BASE IMAGE
FROM python:3.12
# INSTALL SYSTEM DEPENDENCIES 
RUN apt-get update && apt-get install -y \
    build-essential \
    curl \
    libpq-dev \
    #FOR TRANSLATION
    gettext \ 
    #LIBRALIES FOR PILLOW
    libffi8 \
    libgdk-pixbuf2.0-0 \
    liblcms2-2 \
    libopenjp2-7 \
    libssl3 \
    libtiff6 \
    libwebp7 \
    shared-mime-info \
    mime-support \
    #LIBRALIES FOR POSTGRESQL
    libpq5 \
    #CLEAN UP
    && apt-get clean \ 
    && rm -rf /var/lib/apt/lists/*
    
# SET PYTHON DEPENDENCIES
WORKDIR /app
RUN pip install --no-cache-dir poetry==1.8.4 && \
    poetry config virtualenvs.create false
# COPY PROJECT FILES
COPY pyproject.toml poetry.lock ./
# INSTALL DEPENDENCIES
RUN poetry install --no-root --no-interaction

# CREATE A NON-ROOT USER
RUN groupadd --system saleor && \
useradd -r -g saleor saleor && chown -R saleor:saleor /app/

USER saleor
# COPY SOURCE CODE
COPY . .

# CONFIGURE STATIC FILES
ENV ALLOWED_HOSTS="*"
ENV DATABASE_URL="postgres://user:password@db:5432/saleor"
ARG STATIC_URL
ENV STATIC_URL=${STATIC_URL:-/static/}
RUN SECRET_KEY=dummy STATIC_URL=${STATIC_URL} python3 manage.py collectstatic --no-input
# EXPOSE PORT
EXPOSE 8000
# ADD START COMMAND OR ENTRYPOINT
CMD ["gunicorn", "--bind", ":8000", "--workers", "4", "--worker-class", "saleor.asgi.gunicorn_worker.UvicornWorker", "saleor.asgi:application"]
