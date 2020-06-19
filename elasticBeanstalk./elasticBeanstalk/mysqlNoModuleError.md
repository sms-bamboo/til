# ElasticBeanstalk 배포 시 'no module named 'MySQLdb' 에러

## 0. 작업
ElasticBeanstalk 에 MySQL을 사용하는 Django 어플리케이션 배포 중 이슈를 정리하였다.  
  
## 1. 에러
Django 어플리케이션에서 MySQL 을 연결하기 위해 mysqlclient 모듈을 사용한다.
````
...
mysqlclient==1.3.10
````

requirement.txt에 mysqlclient 모듈이 기재되어 있음에도 환경에 어플리케이션 배포가 실패하는 일이 발생한다.  
상세 로그를 보면 다음 사유로 인해 실패했음을 확인할 수 있었다.  
````
no module named 'MySQLdb'
````
  
## 2. 확인
신규 환경을 배포해봐도 동일한 에러가 발생하여 서버에서 직접 접속하여 확인해보기로 했다.  
pip3 사용을 위해 가상 환경에 접속한다. Amazon linux2, python3.7 환경 기준 가상 환경 접속 방법은 다음과 같다.  
````
source /var/app/venv/staging-LQM1lest/bin/activate
````
  
이후 pip install 을 통해 mysqlclient가 정상 설치되는지 확인한다.  
````
(staging) [ec2-user@ip-172-31-3-241 staging-LQM1lest]$ sudo pip3 install mysqlclient
Collecting mysqlclient
  Downloading mysqlclient-1.4.6.tar.gz (85 kB)
     |████████████████████████████████| 85 kB 1.1 MB/s
    ERROR: Command errored out with exit status 1:
     command: /var/app/venv/staging-LQM1lest/bin/python -c 'import sys, setuptools, tokenize; sys.argv[0] = '"'"'/tmp/pip-install-7o4z0xb3/mysqlclient/setup.py'"'"'; __file__='"'"'/tmp/pip-install-7o4z0xb3/mysqlclient/setup.py'"'"';f=getattr(tokenize, '"'"'open'"'"', open)(__file__);code=f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' egg_info --egg-base /tmp/pip-pip-egg-info-t26fayqs
         cwd: /tmp/pip-install-7o4z0xb3/mysqlclient/
    Complete output (12 lines):
    /bin/sh: mysql_config: command not found
    /bin/sh: mariadb_config: command not found
    /bin/sh: mysql_config: command not found
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "/tmp/pip-install-7o4z0xb3/mysqlclient/setup.py", line 16, in <module>
        metadata, options = get_config()
      File "/tmp/pip-install-7o4z0xb3/mysqlclient/setup_posix.py", line 61, in get_config
        libs = mysql_config("libs")
      File "/tmp/pip-install-7o4z0xb3/mysqlclient/setup_posix.py", line 29, in mysql_config
        raise EnvironmentError("%s not found" % (_mysql_config_path,))
    OSError: mysql_config not found
    ----------------------------------------
ERROR: Command errored out with exit status 1: python setup.py egg_info Check the logs for full command output.
````
  
위와 같이 에러가 발생하는 것을 확인할 수 있다.  
mysqlclient 모듈을 설치하기 위해서는 mysql-dev 패키지가 필요하며, 다음 명령어를 통해 설치한다.  
````
sudo yum install mysql-devel -y
````
  
이후 다시 mysqlclient 모듈을 설치해본다.  
````
(staging) [ec2-user@ip-172-31-3-241 staging-LQM1lest]$ sudo pip3 install mysqlclient
Collecting mysqlclient
  Downloading mysqlclient-1.4.6.tar.gz (85 kB)
     |████████████████████████████████| 85 kB 1.0 MB/s
Using legacy setup.py install for mysqlclient, since package 'wheel' is not installed.
Installing collected packages: mysqlclient
    Running setup.py install for mysqlclient ... done
Successfully installed mysqlclient-1.4.6
````

정상 설치됨을 확인할 수 있다.  
이후 프로젝트를 재배포하여 gunicorn 서버를 재기동시킨다.

## 3. config 파일
서버가 새로 기동될 때 mysql-dev 모듈을 자동으로 설치할 수 있도록 다음 config 파일을 만들어 저장한다.  
  
* package.config
````
packages: 
  yum:
    python3-devel: []
    mariadb-devel: []
````
  
이후 신규 서버 배포 시에는 정상 기동 될 것이다.
