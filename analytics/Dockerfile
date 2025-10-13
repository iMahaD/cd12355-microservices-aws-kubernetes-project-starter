# syntax=docker/dockerfile:1
FROM python:3.11-alpine

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

# Install build dependencies for psycopg2 and others
RUN apk add --no-cache gcc musl-dev python3-dev libffi-dev openssl-dev cargo postgresql-dev

# Copy requirements and install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy app files
COPY . .

EXPOSE 5153

CMD ["python", "app.py"]
