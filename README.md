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
	* Ubuntu 14.04
	* [Item-Catalog][2] python3 web application.
	* `Python 3.X` along with `pip3`.
	* `PostgreSQL 9.0`.
	* `Psycopg` adapter.

# Instructions

## 1. Amazon EC2 Instance creation
This `README` does not cover the creation of Virtual machines but you can follow the steps to create one in: [EC2 Launch instance][3]

The following steps are based in a `Ubuntu 14.04` Virtual Machine and you must have the `SSH` key to access it.

The SSH access key usually ends with `.pem` or `.rsa`, in this case we will call it `root_key.rsa` and for convenience store it inside `~/.ssh/` folder with `600` file permissions.

## 2. Create ssh pair key
We need a ssh pair key to use in the next steps for `grader` user, so we need to generate one pair with the following instructions: 

* On your local machine (Unix based) execute the command:

```
ssh-keygen
```

* It will prompt for a path to store the key, in this case we can use: 

```
~/.ssh/grader_key.rsa
```

* It generates two files `grader_key.rsa` and `grader_key.rsa.pub`

Note that these files are different to the one you already have to login in your virtual machine.



[1]: http://52.36.132.142/
[2]: https://github.com/aristoteles-nunez/Item-Catalog/
[3]: http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance_linux