The guide covers:

Dependencies: What packages to install.
Core Configuration: How to set up celery.py and __init__.py.
Settings: The essential configuration for your settings.py.
Task Creation: How to define a background task.
Scheduling (Cron): How to use the Django Admin for periodic tasks.
Running Services: How to start the Redis, Worker, and Beat processes.

1. Install Dependencies
You will need a message broker (Redis is recommended) and the following Python packages:

bash
pip install celery redis django-celery-beat django-celery-results
2. Core Project Configuration
celery.py
 (In project root next to 
settings.py
)
Create this file to define the Celery application instance.

python
import os
from celery import Celery
# Set default Django settings module
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'your_project_name.settings')
app = Celery('your_project_name')
# Load config from settings.py using CELERY_ prefix
app.config_from_object('django.conf:settings', namespace='CELERY')
# Automatically discover tasks in all installed apps
app.autodiscover_tasks()
init
.py
 (In project root next to 
settings.py
)
Ensure Celery starts when Django starts.

python
from .celery import app as celery_app
__all__ = ('celery_app',)
3. Django Settings (
settings.py
)
python
INSTALLED_APPS = [
    ...,
    'django_celery_results',
    'django_celery_beat', # Only if you need cron/periodic tasks
]
# Celery Configuration
CELERY_BROKER_URL = 'redis://localhost:6379/0'
CELERY_RESULT_BACKEND = 'django-db'
CELERY_CACHE_BACKEND = 'django-cache'
CELERY_ACCEPT_CONTENT = ['json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'
CELERY_TIMEZONE = 'Asia/Kolkata' # Set to your local timezone
# If using django-celery-beat for database-backed scheduling
CELERY_BEAT_SCHEDULER = 'django_celery_beat.schedulers:DatabaseScheduler'
4. Creating a Task
In any app (e.g., myapp/tasks.py), define your task:

python
from celery import shared_task
@shared_task
def add(x, y):
    return x + y
5. Scheduling a Cron (Periodic Task)
Via Django Admin (Recommended)
Run migrations: python manage.py migrate
Create admin user: python manage.py createsuperuser
Go to /admin -> Periodic Tasks.
Create a Crontab (e.g., every minute: * * * * *).
Create a Periodic Task, select your task, and assign the Crontab.
6. Running Services
You must run three separate processes:

Django Development Server: python manage.py runserver
Celery Worker: celery -A your_project_name worker --loglevel=info
Celery Beat: celery -A your_project_name beat --loglevel=info
IMPORTANT

Windows Users: If the worker isn't processing tasks, try adding --pool=solo: celery -A your_project_name worker --loglevel=info --pool=solo
