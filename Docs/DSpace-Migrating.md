# OLD SERVER
## Stop Tomcat service
service tomcat stop
## Update sequences in PostgreSQL
<!-- Default command:
psql -U [database-user] -f [dspace]/etc/postgres/update-sequences.sql [database-name]
-->
psql -U dspace -f [dspace]/etc/postgres/update-sequences.sql dspace
## Make DB dump
pg_dump -U [db_username] [db_name] > [output_file.sql]

# NEW SERVER
## ON HOST Edit local.cfg for DB remove
echo "db.cleanDisabled=false" >> local.cfg
## Copy DB backup into coantiner
docker cp ./backup-files/backup-test.sql dspacedb:/pgdata/backup.sql
## Create empty DB in PostgreSQL container
docker exec -i dspacedb createdb -U [db_username] dspace-old
## Restore backup into created DB
cat ./backup-files/backup-test.sql | docker exec -i 477aea9d3a49 psql -U dspace -d dspace-old

