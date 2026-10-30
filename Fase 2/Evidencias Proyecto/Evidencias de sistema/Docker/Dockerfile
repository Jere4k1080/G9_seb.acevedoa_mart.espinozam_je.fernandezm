FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PORT=8080

WORKDIR /srv
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app/ app/
RUN useradd --system --no-create-home api
USER api

# Cloud Run inyecta PORT; 8080 si no viene. Su proxy es la única entrada, así que se confía en
# X-Forwarded-Proto: sin eso el 307 por barra final apunta a http://.
CMD ["sh", "-c", "exec uvicorn app.main:app --host 0.0.0.0 --port ${PORT:-8080} --proxy-headers --forwarded-allow-ips '*'"]
