# Linux-Server-Configuration-4-Python-App
Configuration steps for production deploy of python3 web application


# Finished Application

The application can be seen in:
[http://52.36.132.142/][1]

# Project Goals

The main goal for this project is deploy a web application made with `Flask` using an `Apache` web server.

* **User access**.
	* Create user with `sudo` privilegies. In this case the user will be `grader`.
	* Use a key based `SSH` authentication with user `grader`.
	* Create user with database limited acces. This user will be `catalog`.
* **Firewall**.
	* The `SSH` service will be attending connections on port: `2200`
	* Only allow incoming connections for: 
		* `SSH`: `2200` 
		* `HTTP`: `80` 
		* `NTP`: `123` 
* **Software**.
	* [Item-Catalog][2] python3 web application.
	* `Python 3.X` along with `pip3`.
	* `PostgreSQL 9.0`.
	* `Psycopg` adapter.




[1]: http://52.36.132.142/
[2]: https://github.com/aristoteles-nunez/Item-Catalog/
