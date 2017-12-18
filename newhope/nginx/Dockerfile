FROM nginx:alpine

ADD sites-available/django.conf /etc/nginx/sites-available/django.conf
RUN ln -s /etc/nginx/sites-available/django.conf /etc/nginx/sites-enabled/django.conf
