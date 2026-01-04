# Observo - Centralized Logging Platform

Modern, self-hosted centralized logging and monitoring platform for Django applications.

🔗 **Platform:** https://observo.digitalrepublic.space
📦 **Package:** `pip install observo-handler`
📚 **Documentation:** https://github.com/kennedy-ak/observo-docs

## Features

- 📊 **Project-based Organization** - Manage multiple applications
- 🔍 **Powerful Search & Filtering** - Find logs instantly
- 🚀 **Real-time Logging** - Sub-second log ingestion
- 📝 **Request Tracing** - Track logs across requests
- 🔐 **Secure API Authentication** - Per-project API keys
- 🎨 **Clean Dashboard** - Beautiful Bootstrap UI

## Quick Start

### 1. Create Project in Observo

1. Visit https://observo.digitalrepublic.space
2. Login or request access
3. Create a new project
4. Copy your **Project ID** and **API Key**

### 2. Install Handler
```bash
pip install observo-handler
```

### 3. Configure Django

Add to `settings.py`:
```python
# Add middleware
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'observo_handler.middleware.RequestIDMiddleware',  # Add this
    # ... rest of middleware
]

# Configure logging
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'observo': {
            'class': 'observo_handler.ObservoHandler',
            'level': 'INFO',
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

### 4. Start Logging
```python
import logging

logger = logging.getLogger(__name__)
logger.info('Hello Observo!')
```

### 5. View Logs

Visit https://observo.digitalrepublic.space and see your logs in real-time!

## Documentation

- [Getting Started Guide](docs/getting-started/README.md)
- [Django Integration](docs/integration/django.md)
- [Flask Integration](docs/integration/flask.md)
- [API Reference](docs/api-reference/README.md)
- [Examples](docs/examples/README.md)

## Support

- 📧 Email: akogokennedy@gmail.com
- 🐛 Issues: https://github.com/kennedy-ak/observo-handler/issues
- 💬 Discussions: https://github.com/kennedy-ak/observo-handler/discussions

## License

MIT License - See [LICENSE](LICENSE)