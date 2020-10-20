# Working with PostgreSQL

You can use [pgAdmin](https://www.pgadmin.org/) to manage PostrgeSQL.

Navigate to the pgAdmin Welcome page: http://localhost:5050

Login using the PGADMIN_DEFAULT_EMAIL (admin@pgadmin.org) and PGADMIN_DEFAULT_PASSWORD (secret) credentials:

<p align="center">
  <img src="https://github.com/Robinyo/js-docker/blob/master/docs/screen-shots/pgamin-login.png">
</p>

Create Connection Wizard - General Tab:

<p align="center">
  <img src="https://github.com/Robinyo/serendipity-api/blob/master/projects/spring-boot/docs/screen-shots/pgamin-server-general-tab.png">
</p>

Create Connection Wizard - Connection Tab:

<p align="center">
  <img src="https://github.com/Robinyo/serendipity-api/blob/master/projects/spring-boot/docs/screen-shots/pgamin-server-connection-tab.png">
</p>

**Note:** The 'Host name / address' field must match the value (postgres) specified in the project's 
[docker-compose.yml](https://github.com/Robinyo/js-docker/blob/master/docker-compose.yml).
