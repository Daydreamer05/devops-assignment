FROM python:3.12-slim

COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/uv
WORKDIR /app

COPY pyproject.toml uv.lock /app/

RUN uv export --frozen --no-dev --format=requirements-txt -o /tmp/requirements.txt && \
    uv pip install --system -r /tmp/requirements.txt

COPY . /app/

EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
