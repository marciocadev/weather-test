FROM python:3.11-rc-alpine

RUN mkdir /src/
WORKDIR /src
# Install gcc for python dependencies
RUN apk add --update --no-cache --virtual .tmp_build_deps \
    gcc

# Copy and install python dependencies
COPY ./requirements-base.txt /src/requirements.txt
RUN pip install -r requirements.txt
# Remove gcc from image
RUN apk del .tmp_build_deps

COPY ./src/ /src/
ENV FLASK_APP=app.py

############# DEBUGGER #############
FROM base AS debug
COPY ./requirements-debug.txt /src/requirements.txt
RUN pip install -r requirements.txt
# Keeps Python from generating .pyc files in the container
ENV PYTHONDONTWRITEBYTECODE 1 
# Turns off buffering for easier container logging
ENV PYTHONUNBUFFERED 1

ARG author
RUN echo "Author: $author"

WORKDIR /src/
CMD python -m ptvsd --host 0.0.0.0 --port 5678 --wait --multiprocess -m flask run -h 0.0.0.0 -p 5000

############# PRODUCTION #############
FROM base AS prod

# Copy and install python dependencies
COPY ./requirements-prod.txt /src/requirements.txt
RUN pip install -r requirements.txt

# Switching to a non-root user, please refer to https://aka.ms/vscode-docker-python-user-rights
RUN adduser --disabled-password appuser && chown -R appuser /src
USER appuser

CMD gunicorn --bind 0.0.0.0:5000 wsgi:app 
#flask run -h 0.0.0.0 -p 5000