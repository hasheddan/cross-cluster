FROM postgres

ADD . .

CMD ["psql", "-a", "-f", "./init.sql"]