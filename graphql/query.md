# GraphQL Query
GraphQL 의 query 사용에 대해 알아보았다.  
query 작업을 통해 API에 데이터를 요청하며, query 내에는 GraohQL에서 받을 데이터를 기입한다.
  
http://snowtooth.moonhighway.com/ 에서 리프트 데이터를 가지고 테스트해볼 수 있다.

## 1. 단일 쿼리
  
다음 쿼리를 실행하여 모든 리프트의 이름과 상태를 확인해본다.  
#### [쿼리]
````
query {
  allLifts {
    name
    status
  }
}
````

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
