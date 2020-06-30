# GraphQL 시작하기
Appsync 사용을 위해 GraphQL 에 대해 알아보고자 한다.  

## 1. 개요
GraphQL은 페이스북에서 만든 API를 위한 쿼리 언어이다.  
  
기존 REST API는 각 기능마다 각각의 엔드포인트를 있으며, 새로운 기능이 추가될 때 마다 새로운 API를 작성하여 엔드포인트가 추가되었다.  
GraapQL은 하나의 엔드포인트를 사용하며, 사용하는 쿼리에 따라 각기 다른 응답을 받는 것이 가능하다.  

## 2. 간단 테스트
다음 사이트에서 간단하게 쿼리를 실행해볼 수 있다. (
* https://graphql-tryout.herokuapp.com/graphql
  
우측 상단의 "Docs" 를 클릭하면 스키마 구조를 확인해볼 수 있다.  
다음과 같은 쿼리를 입력하고 실행해본다.  
````
query{
  account(id: "1"){
    username
    email
  }
}
````
"account" 객체에서 "id" 필드의 인자값을 1로 전달하여, "username"과 "email" 필드의 값을 요청하였다.  
  
쿼리 결과는 다음과 같다. Query 형식과 유사한 구조의 JSON 형식을 받아오는 것을 볼 수 있다.
````
{
  "data": {
    "account": {
      "username": "velopert",
      "email": "public.velopert@gmail.com"
    }
  }
}
````
  
개발자는 무슨 데이터가 필요한지에 대한 요구사항만 작성하면 되며, 어떻게 가져올지에 대해서는 신경쓰지 않아도 된다. 

