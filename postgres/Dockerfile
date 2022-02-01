FROM postgres:11.6-alpine

COPY createSchema.sql /docker-entrypoint-initdb.d/
COPY insertData.sql /docker-entrypoint-initdb.d/