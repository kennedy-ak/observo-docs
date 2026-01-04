# Examples

## Basic Logging
```python
import logging

logger = logging.getLogger(__name__)

logger.debug('Debug message')
logger.info('Info message')
logger.warning('Warning message')
logger.error('Error message')
logger.critical('Critical message')
```

## Exception Logging
```python
try:
    result = 1 / 0
except Exception as e:
    logger.exception('Division by zero error')
    # Stack trace automatically included
```

## Custom Metadata
```python
logger.info(
    'User checkout',
    extra={
        'extra_data': {
            'user_id': 123,
            'cart_total': 99.99,
            'items_count': 3
        }
    }
)
```

## Request Tracing
```python
def my_view(request):
    logger.info('Request started')
    
    process_step_1()
    logger.debug('Step 1 completed')
    
    process_step_2()
    logger.debug('Step 2 completed')
    
    logger.info('Request completed')
    # All logs share the same request_id
```

## Structured Logging
```python
import json

logger.info(
    'API call',
    extra={
        'extra_data': {
            'endpoint': '/api/users',
            'method': 'GET',
            'response_time_ms': 245,
            'status_code': 200
        }
    }
)
```

## Django Signals
```python
from django.db.models.signals import post_save
from django.dispatch import receiver
import logging

logger = logging.getLogger(__name__)

@receiver(post_save, sender=User)
def log_user_created(sender, instance, created, **kwargs):
    if created:
        logger.info(
            'New user registered',
            extra={
                'extra_data': {
                    'user_id': instance.id,
                    'username': instance.username,
                    'email': instance.email
                }
            }
        )
```

## Celery Tasks
```python
from celery import shared_task
import logging

logger = logging.getLogger(__name__)

@shared_task
def process_order(order_id):
    logger.info(f'Processing order: {order_id}')
    
    try:
        # Process order
        result = do_processing()
        logger.info(f'Order {order_id} processed successfully')
    except Exception as e:
        logger.error(f'Order {order_id} processing failed', exc_info=True)
        raise
```

## Testing
```python
import logging
from django.test import TestCase

class MyTestCase(TestCase):
    def test_logging(self):
        logger = logging.getLogger(__name__)
        
        with self.assertLogs(logger, level='INFO') as cm:
            logger.info('Test log')
        
        self.assertIn('Test log', cm.output[0])
```