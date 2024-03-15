inside terminal:
    docker run -p 6379:6379 --name some-redis -d redis
    (downloads the official Redis Docker image from Docker Hub and runs it on port 6379 in the background)

    docker exec -it some-redis redis-cli ping
    (test if redis is properly installed or not. it displays 'pong' if redis properly installed)


create django project and app
inside project create celery.py

celery.py:
    import os

    from celery import Celery

    from django.conf import settings

    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'project.settings')

    app = Celery("project")

    app.config_from_object('django.conf:settings', namespace='CELERY')

    app.autodiscover_tasks(lambda: settings.INSTALLED_APPS)

    @app.task
    def divide(x, y):
        import time
        time.sleep(5)
        return x / y

 __init__.py:
    from .celery import app as celery_app

    __all__ = ('celery_app',)


settings.py:

    CELERY_BROKER_URL = "redis://127.0.0.1:6379/0"
    CELERY_RESULT_BACKEND = "redis://127.0.0.1:6379/0"

run celery in terminal:
    celery -A project worker --loglevel=info

terminal:
    python manage.py shell
    from project.celery import divide
    task = divide.delay(10,2)
    print(type(task))
    print(task.state, task.result)

check the task state using flower:
    celery -A project flower --port=5555

in browser:
    localhost:5555