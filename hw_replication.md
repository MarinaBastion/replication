# create two images in docker, the docker-compose file:
version: '3.8'

services:

  postgresql_01:
    image: postgres:14.4
    container_name: postgresql_01
    restart: always
    volumes:
      - ./postgres1/:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: postgres024

  postgresql_02:
    image: postgres:14.4
    container_name: postgresql_02
    restart: always
    volumes:
      - ./postgres2/:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: postgres024

# docker-compose up -d , docker ps:
CONTAINER ID   IMAGE           COMMAND                  CREATED        STATUS          PORTS      NAMES
da1c7110b063   postgres:14.4   "docker-entrypoint.s…"   20 hours ago   Up 33 minutes   5432/tcp   postgresql_02
00fdb12411d3   postgres:14.4   "docker-entrypoint.s…"   20 hours ago   Up 18 hours     5432/tcp   postgresql_01
# settings on master:
1. docker exec -it postgresql_01 bash
2. su - postgres
3. createuser --replication -P repluser -> exit

# settings in postgresql.conf: 
    wal_level = replica
    max_wal_senders = 2
    max_replication_slots = 2
    hot_standby = on
    hot_standby_feedback = on

# add to pg_hba.conf:  
    host    replication     all             0.0.0.0/0           md5
# docker restart postgresql_01

# clear data catalog on slave :
 rm -r ./postgres2/* 

# do replica:
1. docker exec -it postgresql_02 bash
2. su - postgres -c "pg_basebackup --host=postgresql_01 --username=repluser --pgdata=/var/lib/postgresql/data --wal-method=stream --write-recovery-conf"




