# GraphQL Query
GraphQL 의 query 사용에 대해 알아보았다.  
query 작업을 통해 API에 데이터를 요청하며, query 내에는 GraohQL에서 받을 데이터를 기입한다.
  
http://snowtooth.moonhighway.com/ 에서 리조트 데이터를 가지고 테스트해볼 수 있다.

## 1. 단일 쿼리

요청할 데이터를 '필드'로 기입한다.  
다음 쿼리를 실행하여 모든 lift 의 이름과 상태를 확인해볼 수 있다.  
#### [쿼리]
````
query {
  allLifts {
    name
    status
  }
}
````
  
쿼리 구조와 동일한 JSON 결과값을 받아오는 것을 볼 수 있다. (결과 값이 길어 일부 생략)
#### [결과]
````
{
  "data": {
    "allLifts": [
      {
        "name": "Astra Express",
        "status": "OPEN"
      },
      {
        "name": "Jazz Cat",
        "status": "OPEN"
      }
    ]
  }
}
````

## 2. 복수 쿼리
두 개의 결과값의 받아오기 위해 다음과 같이 쿼리를 두개 작성하고 실행해본다.  
결과로 에러가 발생하는 것을 확인할 수 있다.  
````
query {
  allLifts {
    name
    status
  }
}
query {
	allTrails{
    name
    status
  }
}
````
다수의 쿼리를 실행하기 위해서는 하나의 쿼리에 모든 필드를 기입하면 된다.

#### [쿼리]
````
query {
  allLifts {
    name
    status
  }
	allTrails{
    name
    status
  }
}
````
  
다음과 같이 lift 정보와 trail 정보를 받아오는 것을 확인할 수 있다. (결과 값이 길어 일부 생략)

#### [결과]
````
{
  "data": {
    "allLifts": [
      {
        "name": "Astra Express",
        "status": "CLOSED"
      },
      {
        "name": "Jazz Cat",
        "status": "OPEN"
      }
    ],
    "allTrails": [
      {
        "name": "Blue Bird",
        "status": "OPEN"
      },
      {
        "name": "Blackhawk",
        "status": "OPEN"
      }
    ]
  }
}
````

## 3. 별칭 부여

필드명을 다른 이름으로 받아보고 싶을 때는 쿼리 내에서 지정하면 된다. (결과 값이 길어 일부 생략)
#### [쿼리]
````
query {
  lift : allLifts {
    liftName : name
    status
  }
}
````
지정한 "lift" 와 "liftName" 으로 필드명을 반환하는 것을 볼 수 있다.
#### [결과]
````
{
  "data": {
    "lift": [
      {
        "liftName": "Astra Express",
        "status": "CLOSED"
      },
      {
        "liftName": "Jazz Cat",
        "status": "OPEN"
      }
    ]
  }
}
````
