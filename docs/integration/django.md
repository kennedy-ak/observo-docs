# Django Integration Guide

Complete guide to integrating Observo with Django applications.

## Table of Contents

1. [Basic Setup](#basic-setup)
2. [Request Tracing](#request-tracing)
3. [Custom Metadata](#custom-metadata)
4. [Environment-specific Configuration](#environment-specific-configuration)
5. [Production Best Practices](#production-best-practices)

## Basic Setup

### Installation
```bash
pip install observo-handler
```

### Minimal Configuration
```python
# settings.py

LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'observo': {
            'class': 'observo_handler.ObservoHandler',
            'project_id': 'your-project-id',
            'api_key': 'your-api-key',
            'observo_url': 'https://observo.digitalrepublic.space/api/v1/ingest/',
        },
    },
    'root': {
        'handlers': ['observo'],
        'level': 'INFO',
    },
}
```

## Request Tracing

Enable request tracing to track all logs belonging to a single HTTP request:

### 1. Add Middleware
```python
# settings.py

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'observo_handler.middleware.RequestIDMiddleware',  # Add this
    'django.contrib.sessions.middleware.SessionMiddleware',
    # ... rest
]
```

### 2. Use in Views
```python
import logging

logger = logging.getLogger(__name__)

def my_view(request):
    # All logs in this request will have the same request_id
    logger.info('Starting request processing')
    
    result = do_something()
    logger.debug(f'Intermediate result: {result}')
    
    logger.info('Request completed successfully')
    
    # The request_id is automatically added to all logs
    return HttpResponse('Done')
```

### 3. Access Request ID
```python
def my_view(request):
    request_id = request.request_id  # Available on request object
    logger.info(f'Processing request: {request_id}')
```

### 4. View in Observo

In the Observo dashboard:
1. Click on any log entry
2. Note the **request_id**
3. Click "View Related Logs"
4. See all logs from that single request

## Custom Metadata

Add custom data to your logs:
```python
logger.info(
    'User performed action',
    extra={
        'extra_data': {
            'user_email': 'user@example.com',
            'action': 'purchase',
            'amount': 99.99,
            'items': ['item1', 'item2']
        }
    }
)
```

In Observo, this appears in the "metadata" field.

## Environment-specific Configuration

### Development
```python
# settings/development.py

LOGGING['handlers']['observo']['level'] = 'DEBUG'
LOGGING['handlers']['observo']['batch_size'] = 5  # Send more frequently
```

### Production
```python
# settings/production.py

LOGGING['handlers']['observo']['level'] = 'INFO'
LOGGING['handlers']['observo']['batch_size'] = 50  # Batch more for efficiency
```

## Production Best Practices

### 1. Use Environment Variables
```python
import os

LOGGING = {
    'handlers': {
        'observo': {
            'class': 'observo_handler.ObservoHandler',
            'project_id': os.getenv('OBSERVO_PROJECT_ID'),
            'api_key': os.getenv('OBSERVO_API_KEY'),
            'observo_url': os.getenv('OBSERVO_URL'),
        },
    },
}
```

### 2. Add Console Handler for Development
```python
LOGGING = {
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
        },
        'observo': {
            'class': 'observo_handler.ObservoHandler',
            # ...
        },
    },
    'root': {
        'handlers': ['console', 'observo'],  # Both handlers
    },
}
```

### 3. Configure Different Loggers
```python
LOGGING = {
    'loggers': {
        'django': {
            'handlers': ['observo'],
            'level': 'INFO',
        },
        'django.request': {
            'handlers': ['observo'],
            'level': 'ERROR',  # Only errors for requests
        },
        'myapp': {
            'handlers': ['console', 'observo'],
            'level': 'DEBUG',  # Debug for your app
        },
    },
}
```

### 4. Handle Sensitive Data

Never log sensitive information:
```python
# ❌ BAD
logger.info(f'User password: {password}')
logger.info(f'Credit card: {card_number}')

# ✅ GOOD
logger.info('User logged in successfully')
logger.info('Payment processed')
```

## Complete Example
```python
# settings.py

import os

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'observo_handler.middleware.RequestIDMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {process:d} {thread:d} {message}',
            'style': '{',
        },
    },
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'formatter': 'verbose',
        },
        'observo': {
            'class': 'observo_handler.ObservoHandler',
            'level': 'INFO',
            'project_id': os.getenv('OBSERVO_PROJECT_ID'),
            'api_key': os.getenv('OBSERVO_API_KEY'),
            'observo_url': os.getenv(
                'OBSERVO_URL',
                'https://observo.digitalrepublic.space/api/v1/ingest/'
            ),
            'batch_size': 10,
            'flush_interval': 5,
        },
    },
    'root': {
        'handlers': ['console', 'observo'],
        'level': 'INFO',
    },
    'loggers': {
        'django': {
            'handlers': ['console', 'observo'],
            'level': 'INFO',
            'propagate': False,
        },
        'django.request': {
            'handlers': ['console', 'observo'],
            'level': 'WARNING',
            'propagate': False,
        },
        'myapp': {
            'handlers': ['console', 'observo'],
            'level': 'DEBUG',
            'propagate': False,
        },
    },
}
```

## Troubleshooting

### Logs not appearing?

1. Check credentials in settings
2. Test API endpoint:
```bash
curl https://observo.digitalrepublic.space/api/v1/health/
```

3. Check for errors:
```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

### Performance issues?

Increase batch size:
```python
'batch_size': 50,  # Send in larger batches
'flush_interval': 10,  # Wait longer before flushing
```

## Next Steps

- [API Reference](../api-reference/README.md)
- [Examples](../examples/README.md)
- [Flask Integration](flask.md)