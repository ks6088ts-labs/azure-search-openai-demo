# https://github.com/microsoft/sample-app-aoai-chatGPT/blob/main/WebApp.Dockerfile

FROM node:20-alpine AS frontend
RUN mkdir -p /home/node/app/frontend/node_modules && chown -R node:node /home/node/app/frontend
RUN mkdir -p /home/node/app/backend && chown -R node:node /home/node/app/backend

WORKDIR /home/node/app/frontend
COPY ./frontend/ ./
USER node
RUN npm ci
RUN npm run build

FROM python:3.9.7-alpine3.14

WORKDIR /usr/src/app/backend

RUN apk add --no-cache --virtual .build-deps \
    build-base \
    libffi-dev \
    openssl-dev \
    curl \
    && apk add --no-cache \
    libpq \
    && pip install --no-cache-dir uwsgi

COPY ./backend/requirements.txt /usr/src/app/backend
RUN pip install --no-cache-dir -r /usr/src/app/backend/requirements.txt \
    && rm -rf /root/.cache

COPY ./backend /usr/src/app/backend
COPY --from=frontend /home/node/app/backend/static  /usr/src/app/backend/static/
EXPOSE 80
CMD ["uwsgi", "--http", ":80", "--wsgi-file", "app.py", "--callable", "app", "-b","32768"]
