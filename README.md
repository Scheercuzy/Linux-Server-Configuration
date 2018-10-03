# Linux Server Configuration

This is a Udacity project on Configuring a Linux Server on an instance on the web.

In this project I secured a server from a number of attack vectors, installed and configured a database server, and deployed one of my existing web applications onto it.

## Server location

35.227.116.194

## List of software installed

- python3
- uWSGI
- Nginx
- UFW
- PostgresSQL

## Summary of the Configuration 

- The firewall on this system deny's all incoming connections to any ports except ports `2200`, `80` and `123`
- SSH is configured to work on a custom port of `2200`
- Key-based SSH authentication is enforced
- Root cannot log in remotely
- user `grader` has sudo privilege
- uWSGI was setup to serve the catalog app
- systemd runs the uWSGI app
- Nginx serves the uWSGI app on port 80



#### Configuration locations

SSH configs: `/etc/ssh/sshd_config`

Catalog project location: `/home/grader/FSND-catalog-project`

uWSGI config: ``/etc/uwsgi/apps-available/catalog-app.ini`

SystemD service to run uWSGI app: ``/etc/systemd/system/catalog-app.uwsgi.service`

Nginx config: ``/etc/nginx/sites-available/catalog-app`



## References

[Systemd and uWSGI](https://uwsgi-docs.readthedocs.io/en/latest/Systemd.html)
[Digital Ocean deploying flask App](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-uswgi-and-nginx-on-ubuntu-18-04)
[uWSGI python plugin fix](https://stackoverflow.com/questions/20176959/uwsgi-no-request-plugin-is-loaded-you-will-not-be-able-to-manage-requests)
[Standlone containers](http://flask.pocoo.org/docs/1.0/deploying/wsgi-standalone/)
