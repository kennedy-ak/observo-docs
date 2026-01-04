# API Reference

## ObservoHandler

### Constructor
```python
ObservoHandler(
    project_id: str,
    api_key: str,
    observo_url: str,
    batch_size: int = 10,
    flush_interval: int = 5,
    level = logging.NOTSET
)
```

**Parameters:**
- `project_id` (str, required): Observo project ID
- `api_key` (str, required): Observo API key
- `observo_url` (str, required): Observo API endpoint URL
- `batch_size` (int, optional): Number of logs to batch before sending (default: 10)
- `flush_interval` (int, optional): Seconds between automatic flushes (default: 5)
- `level` (int, optional): Minimum log level to capture (default: NOTSET)

### Methods

#### flush()
Manually flush remaining logs in batch.
```python
handler.flush()
```

#### close()
Close handler and flush remaining logs.
```python
handler.close()
```

## RequestIDMiddleware

Django middleware that generates unique request IDs.

### Usage
```python
MIDDLEWARE = [
    'observo_handler.middleware.RequestIDMiddleware',
    # ...
]
```

### Attributes Added to Request

- `request.request_id` (str): Unique UUID for the request

### Response Headers

Adds `X-Request-ID` header to all responses.

## REST API

### Ingest Endpoint

**POST** `/api/v1/ingest/`

Ingest log entries.

**Headers:**
- `X-Project-ID`: Project ID
- `Authorization`: `Bearer <api_key>`
- `Content-Type`: `application/json`

**Request Body (Single Log):**
```json
{
  "level": "INFO",
  "message": "User logged in",
  "timestamp": "2026-01-04T12:00:00",
  "request_id": "abc-123",
  "user_id": "42",
  "metadata": {"ip": "192.168.1.1"}
}
```

**Request Body (Bulk):**
```json
{
  "logs": [
    {"level": "INFO", "message": "Log 1"},
    {"level": "ERROR", "message": "Log 2"}
  ]
}
```

**Response (Success):**
```json
{
  "status": "success",
  "message": "Logs ingested successfully"
}
```

**Status Codes:**
- `201`: Created
- `400`: Bad Request
- `401`: Unauthorized
- `500`: Server Error

### Health Check

**GET** `/api/v1/health/`

Check API health.

**Response:**
```json
{
  "status": "healthy",
  "service": "Observo"
}
```