# Build stage
FROM python:3.11-alpine AS builder

WORKDIR /app

# Upgrade pip and setuptools FIRST to avoid CVEs
RUN pip install --no-cache-dir --upgrade pip==24.3.1 setuptools==80.10.2 wheel==0.46.3

# Install build dependencies
RUN apk add --no-cache \
    gcc \
    libffi-dev \
    musl-dev \
    postgresql-dev

COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# Runtime stage
FROM python:3.11-alpine

WORKDIR /app

# Install runtime deps
RUN apk add --no-cache curl libpq

# Copy installed packages
COPY --from=builder /install /usr/local

RUN mkdir -p /app/app/{./model_path, ./static/gallery}

# Copy application code
COPY . .

# Create non-root user and set permissions
RUN adduser -D app && chown -R app:app /app
USER app

EXPOSE 5000

CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "2", "--timeout", "120", "run:app"]