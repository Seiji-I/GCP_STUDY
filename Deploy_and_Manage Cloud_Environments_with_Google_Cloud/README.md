# Migrate a stand-alone PostgreSQL database to a Cloud SQL for PostgreSQL instance
Enable these services
```
gcloud services enable datamigration.googleapis.com
gcloud services enable servicenetworking.googleapis.com
```
Download and apply some additions to the PostgreSQL configuration files (to enable pglogical extension) and restart the postgresql service
```
sudo apt install postgresql-13-pglogical

sudo su - postgres -c "gsutil cp gs://cloud-training/gsp918/pg_hba_append.conf ."
sudo su - postgres -c "gsutil cp gs://cloud-training/gsp918/postgresql_append.conf ."
sudo su - postgres -c "cat pg_hba_append.conf >> /etc/postgresql/13/main/pg_hba.conf"
sudo su - postgres -c "cat postgresql_append.conf >> /etc/postgresql/13/main/postgresql.conf"

sudo systemctl restart postgresql@13-main
```
Launch the psql tool
```
sudo su - postgres
psql
```
Add the pglogical database extension to the postgres
```
\c postgres;
CREATE EXTENSION pglogical;
\c orders;
CREATE EXTENSION pglogical;
```
