# 200617 Django 배포
Django 를 ElasticBeanstalk 에 배포하는데 일부 문제 사항이 있었다.  
관련하여 알아낸 사항과 문제 사항 등을 기록하였다.  

## 1. static 파일
개발환경에서는 runserver를 통해 로컬에서 프로젝트를 기동시켜볼 수 있다.  
```
python manage.py runserver 0:8000
```

막상 테스트가 완료되어 운영환경(nginx, wsgi)에 업로드하면 어플리케이션에서 static 파일을 찾지 못해 페이지가 정상동작하지 않는 경우가 있다. 이는 실제로 static 파일이 분산되어 있기 때문이다.  
  
runserver 는 다양한 기능들이 있으며, 그 중 프로젝트 내 static 파일들을 알아서 모아주는 기능이 있다.  
때문에 runserver 실행 시에는 정상 동작하게 되는 것이며, 실제로는 분산된 static 파일을 collectstatic 을 통해 한 곳으로 파일을 모아주는 과정이 필요하다.  
    
먼저 settings.py에 다음 변수를 추가한다.

* settings.py
````
...
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static')

````

"STATIC_URL" 은 static 파일이 위치하는 경로이다.  
"STATIC_ROOT" 는 collectstatic 시 static 파일이 실제 모아지는 위치가 된다.  
  
설정이 완료되었으면 collectstatic 을 실행하여 파일을 모아준다.  
````
python manage.py collectstatic
````

## 2. ElasticBeanstalk 에서의 static 파일
ElaistcBeanstalk 에서는 static 파일을 nginx 에서 제공하기 때문에, 다음과 같이 config 파일을 구성하여 nginx 에 static 파일의 위치를 알릴 필요가 있다.  

````
option_settings:
  aws:elasticbeanstalk:environment:proxy:staticfiles:
    /static: static
````
  
## 3. DB 마이그레이션
일반적으로 django 로 개발되었다면 SQLite 에 DB 구조 및 데이터가 저장되어 있다.  
어플리케이션에 병합되어 있는 형태이기 때문에, 운영 환경에서는 RDB로 분리시켜줄 필요가 있다.  
  
RDS mysql 을 생성하여, 다음과 같은 방안으로 적용하였다.  
  
다음 명령어로 DB 데이터를 export 한다.
````
python manage.py dumpdata > db.json
````

settings.py 에서 다음과 같이 변경하여 RDS DB에 연결한다.  
````
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': '데이터베이스명',
        'USER': '사용자명',
        'PASSWORD': '패스워드',
        'HOST': 'RDS-엔드포인트',
        'PORT': '3306',
    }
}
````
DB 구조를 마이그레이션 한다.
````
python manage.py migrate
````
다음 과정을 진행한다. (DB에서 "ContentType" 을 제거하는 과정으로 보인다. 추가 확인이 필요할 것 같다.)
````
python manage.py shell
[1] : from django.contrib.contenttypes.models import ContentType
[2] : ContentType.objects.all().delete()
[3] : exit()
````
export 했던 데이터를 다시 DB에 import 한다.
````
python manage.py loaddata db.json
````
  
## 4. 문제사항
3번 항목을 진행하였으나.. 기존 테스트 환경에서는 없었던 에러가 발생하였다.
따로 기록을 못해서 유사 사례에서 에러를 발췌하였는데, pk로 인한 에러로 보인다.
````
django.db.utils.IntegrityError: Problem installing fixture 'C:..\lit\backups\dbbackup_20190915_145546.json': 
Could not load contenttypes.ContentType(pk=17): duplicate key value violates unique constraint 
"django_content_type_a pp_label_model_76bd3d3b_uniq" 
````
dump 시 다음과 같이 실행하여 필요없는 데이터를 제외해야 하는 것으로 보인다.  
다만... 작업 대상자(K 부업) 가 작업 중단을 요청하여 직접 적용해보지 못해 안타깝다.. 아마 가능하지 않을까 싶다...
````
python manage.py dumpdata --natural-foreign --natural-primary --exclude contenttypes --exclude auth.permission --exclude admin.logentry --exclude sessions.session --indent 4 > db.json
````
