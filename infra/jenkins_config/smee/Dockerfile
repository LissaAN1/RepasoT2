# Contenedor liviano que ejecuta smee-client: reenvía los eventos del
# webhook de GitHub (recibidos en un canal público de smee.io) hacia el
# endpoint /github-webhook/ de Jenkins, sin necesidad de exponer Jenkins
# a Internet.
FROM node:20-alpine

RUN npm install -g smee-client

# TODO
# Destino por defecto (se puede sobreescribir desde docker-compose.yml).
# SMEE_URL llega desde el .env (canal propio de smee.io).
ENV SMEE_TARGET=http://jenkins:8080/github-webhook/

ENTRYPOINT ["sh", "-c", "smee -u \"$SMEE_URL\" -t \"$SMEE_TARGET\""]
