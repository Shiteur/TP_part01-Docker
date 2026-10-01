#image
FROM postgres:17.2-alpine

# environnement avec le nom de la base de donnée le pseudo et le mdp
ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd


COPY CreateSheme.sql /docker-entrypoint-initdb.d
COPY InsertData.sql /docker-entrypoint-initdb.d