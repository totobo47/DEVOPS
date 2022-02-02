FROM postgres:11.6-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd

COPY createSchema.sql /docker-entrypoint-initdb.d/
COPY insertData.sql /docker-entrypoint-initdb.d/