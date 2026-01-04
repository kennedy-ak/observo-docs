# Getting Started with Observo

## Prerequisites

- Python 3.8+
- Django 3.2+ (or Flask 2.0+)
- Observo account (request access at https://observo.digitalrepublic.space)

## Step 1: Create Observo Project

1. **Login to Observo:**
   - Visit https://observo.digitalrepublic.space
   - Login with your credentials
   
2. **Create Project:**
   - Click "New Project"
   - Enter project name (e.g., "My Django App")
   - Select environment (Production/Staging/Development)
   - Click "Create"

3. **Copy Credentials:**
   - Click on your project
   - Copy **Project ID**
   - Copy **API Key**
   - ⚠️ **Keep these secret!**

## Step 2: Install observo-handler
```bash
pip install observo-handler
```

Or add to `requirements.txt`:
```
observo-handler==1.0.0
```

## Step 3: Configure Your Application

### Django Configuration

Edit `settings.py`:
```python
# 1. Add middleware (for request tracing)
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'observo_handler.middleware.RequestIDMiddleware',  # Add this line
    'django.contrib.sessions.middleware.SessionMiddleware',
    # ... rest of middleware
]

# 2. Configure logging
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {message}',
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
            'project_id': 'YOUR-PROJECT-ID-HERE',
            'api_key': 'YOUR-API-KEY-HERE',
            'observo_url': 'https://observo.digitalrepublic.space/api/v1/ingest/',
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
    },
}
```

### Using Environment Variables (Recommended)
```python
import os

LOGGING = {
    # ...
    'handlers': {
        'observo': {
            'class': 'observo_handler.ObservoHandler',
            'level': 'INFO',
            'project_id': os.getenv('OBSERVO_PROJECT_ID'),
            'api_key': os.getenv('OBSERVO_API_KEY'),
            'observo_url': os.getenv('OBSERVO_URL', 'https://observo.digitalrepublic.space/api/v1/ingest/'),
        },
    },
}
```

Then create `.env`:
```env
OBSERVO_PROJECT_ID=your-project-id
OBSERVO_API_KEY=your-api-key
OBSERVO_URL=https://observo.digitalrepublic.space/api/v1/ingest/
```

## Step 4: Start Logging
```python
import logging

logger = logging.getLogger(__name__)

def my_view(request):
    logger.info('User accessed view')
    logger.debug('Processing request')
    
    try:
        # Your code
        result = process_data()
        logger.info('Operation successful')
    except Exception as e:
        logger.error('Operation failed', exc_info=True)
```

## Step 5: View Logs

1. Visit https://observo.digitalrepublic.space
2. Click on your project
3. Click "View Logs"
4. 🎉 See your logs in real-time!

## Next Steps

- [Advanced Configuration](../integration/advanced.md)
- [Request Tracing](../integration/request-tracing.md)
- [Custom Metadata](../integration/custom-metadata.md)
- [Production Best Practices](../integration/production.md)