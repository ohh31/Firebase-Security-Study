

# Firestore Security Rules 작성요령



> **Firestore 보안규칙의 이점**
>
> 보안규칙 구현이 간단하며 보안규칙을 위해 인프라를 관리하거나 복잡한 서버측 인증 및 인증 코드를 작성할 필요 없다.
>
> 
>
> **Firestore 보안규칙의 필요성**
>
>  Firebase는 디폴트로 사용자데이터에 대한 보안을 제공하지 않는다. 따라서 보안규칙을 통해 인증되지 않은 사용자의 데이터베이스 접근을 막아 데이터가 노출되거나 손상되는 것을 막는다.
>
> <u>관련기사</u>
>
> https://www.hauri.co.kr/security/boannews_view.html?intSeq=12099&page=1&article_num=12044





### 1. 보안규칙 버전 

```firestore
rules_version = '2'; 
```







### 2. 서비스 및 데이터 베이스 선언

```firestore
service cloud.firestore {
match /databases/{database}/documents
```

- cloud.firestore 에 대해 규칙을 적용한다고 선언한다.

- 구체화시킬 path의 pattern을 정의한다. 해당 코드를 사용하도록 한다.







### 3. 보안규칙 작성 



###### 기본구조

    service cloud.firestore {
    
      match /databases/{database}/documents {
          match /stories/{storyid} {
          allow write: if request.auth.uid == resource.data.author;
      	  }
    	}
    } 

  






#### 1) 규칙을 적용할 문서를 지정한다.



- cities collection 내의 모든 문서에 대해 보안규칙 적용 (하위 컬렉션은 적용되지 않는다.)

```firestore
match /cities/{city} {
```



- cities collection 내 seoul이라는 특정 문서에 대해 보안규칙 적용

```firestore		
match /cities/ Seoul {
```



- cities collection 내 모든 하위 범주에 대해 보안규칙 적용

```firestore
match /cities/{document=**}
```



-{ } 내의 이름은 컬렉션 이름과 다르게 임의로 설정하면 된다. 





#### 2) 해당 문서에 대한 보안 규칙을 정의한다.

​	

#### 	a. 보안 요청에 대한 행동(action) 정의 

######  	

##### 읽기

```firestore
allow read: if <condition>;
allow get: if <condition>;
allow list: if <condition>;
```

-  read : 문서를 읽거나 가져오는 행동 ( get + list )

-  get : 단일문서를 읽거나 가져오는 행동 

```dart
Firestore.instance
        	 .collection(‘users’)
       		 .document(uid)
       		 .get()
```

- list : 쿼리 문서 혹은 collection내 모든 문서를 읽거나 가져오는 행동

```dart
Firestore.instance
        .collection(‘users’)
		.get()
```



##### 쓰기

```firestore
allow write: if <condition>;
allow create: if <condition>;
allow update: if <condition>;
allow delete: if <condition>;
```

- write : 문서를 쓰는 행동 ( create + update + delete )

- create : 문서를 생성하는 행동

- update : 문서를 수정하는 행동

- delete : 문서를 삭제하는 행동





#### 	b. 보안 조건 정의



##### ==[ 데이터에 접근할 사용자에 대한 보안조건 ]==



- **모든사람들이 읽고 쓰기 가능**

```firestore
allow read, write: if true
```



- **콘텐츠 소유자만 읽고 쓰기 가능**

```firestore
allow read, write: if request.auth.uid == request.resource.data.author_uid
```

 request.auth.uid와 firestore에 추가되고자 하는 문서의 author_uid 필드의 값이 같을 때 읽고 쓰기를 허용



- **모든 접근을 차단 **

```firestore
allow read, write: if false
```



- **그 외 request.auth 활용** 

<u>Firebase Auth를 사용하여 인증되었다고 가정하는 사용자 정보</u>가 들어 있다.

```firestore
request.auth.uid  

request.auth.token.email
request.auth.token.email_verified // 해당 전자메일 주소가 확인 되었는지 확인한다. 
request.auth.token.email.matches('.*google[.]com$') 
// Google.com을 사용한 사람들만 읽도록 하게 하고 싶다. .matches('.*@domain[.]com')

request.auth.token.phone_number
request.auth.token.firebase.sign_in_provider == "phone"
//(custom, password, phone, anonymous, google.com, facebook.com, github.com, twitter.com)
```

이메일 인증 예시)

```firestore
allow read : if request.auth.token.email != null && request.auth.token.email_verified 
```

>  https://firebase.google.com/docs/reference/rules/rules.firestore.Request





- **사용자 인증 클레임 생성**

서버 측 라이브러리 또는 cloud function을 사용하여 특정 사용자에 대해 설정하는 사용자 지정 변수이다.

```firestore
match / {everythingInMyDatabase == **}{
allow read, write : if request.auth.token.super_admin == true;
}

match /reviews / {reviewID}{
allow read, write : if request.auth.token.role == "moderator";
}
```





##### ==[ 문서의 엑세스에 대한 보안조건 ]==

불필요한 내용들의 포함을 막기 위해 사용한다.

 

> request data : the data requests that's coming in
>
> target documents : you're looking to either read or update ( resource.data )
>
> some other document : some other data located in some other part



- **사용자가 작성한 문서를 데이터베이스에 쓸 경우 (request.resource.data) **

request.resource.data :  사용자가 write하기를 시도했던 document의 모든 필드를 표현한다. 

```firestore
//작성하고자 하는 문서의 score 필드값이 50일 때 쓰기 허용
allow write : if request.resource.data.score == 50
allow write : if request.resource.data["score"] == 50
```

```firestore
//score 필드의 값이 숫자임을 확신할 수 있으며 null value를 허용하지 않는다
allow write : if request.resource.data.score is number 
```

```firestore
//score 필드 값이 1보다 크고 5보다 작은 문서만 쓸 수 있도록 한다.  
allow write : if request.resource.data.score >= 1 
allow write : if request.resource.data.score <= 5
```

```firestore
//작성하고자하는 문서의 category의 내용이 [widgets, things]내에 있을 경우만 허용  
allow write : if request.resource.data.category in ['widgets', 'things']
```

```firestore
//headline이 string type이고 200 character 미만일 때만 쓸 수 있도록 한다.
allow write : if request.resource.data.headline is string && request.resource.data.headline.size() <200 
```

```firestore
//firestore에 추가되고자 하는 문서의 reviewID 필드값과 request.auth.uid 같을 때만 쓸 수 있도록 허용한다. 
allow write : if request.resource.data.reviewID == request.auth.uid; 
```



-  **이미 데이터베이스에 존재하는 문서를 읽거나 쓸 경우(resource.data) **

resource.data :  이미  데이터베이스에 있는 문서를 나타내는 것 

```firestore
//지정한 문서의 reviewerID 필드와 request.auth.uid이 같을 때만 문서를 수정할 수 있도록 한다.
allow update: if resource.data.reviewerID == request.auth.uid 
```

```firestore
//score의 필드값을 변화시키지 않기 위함. 
allow update: resource.data.score == resource.data.score;
```

```firestore
 // 지정문서의 x 필드의 값이 5 이상인 데이터만 사용자가 읽도록 하기 위함
 allow read: if resource.data.x > 5; 
```

```firestore
// 지정문서의 published 필드값이 true인 데이터만 사용자가 읽도록 하기 위함 
allow read: if resource.data.published == true ; 
```

```firestore
allow read: if resource.data.name == 'John Doe'
```



- **다른 문서에 대한 엑세스 (지정한 문서 외에 다른 문서를 사용해야할 경우) **

  - get  : 지정 문서 외에 다른 문서를 가져와서 비교 
  - exist  : 다른 문서에 데이터 존재 여부를 알기 위함  


```firestore
if exists(/databases/$(database)/documents/users/$(request.auth.uid))

if get(/databases/$(database)/documents/users/$(request.auth.uid)).data.admin == true
```

> $(database) :  Cloud Firestore Security Rules 와일드 카드로 사용할 수 있는 데이터베이스 인스턴스를 정의하는 매개변수 
>
> *match /databases/**{database}**/documents*

​		

-map 형태의 필드값을 가져올 때

```firestore
// (roles / request.auth.uid / role / isadmin : true) 일때 승인

match /posts/{docId} {    
    allow read, write: if hasRole("isAdmin")  
}      
function hasRole(userRole) {    
  return get(/databases/$(database)/documents/roles/$(request.auth.uid)).data.role[userRole] 
}
```

```firestore
//(restaurants/restaurantID/ private_data (sub collection) / private / roles / request.auth.uid == editor or owner ) 일 때 승인

if get(/databases/$(database)/documents/restaurants/$(restaurantID)/private_data/private).data.roles[request.auth.uid] == ["editor" ,"owner"]
```



- **쿼리한 내용에 대한 데이터를 가져올 경우 ( request.query )**
  - limit - query limit clause.
  - offset - query offset clause.
  - orderBy - query orderBy clause.

```firestore
document fusersQuery = Firestore.instance.collection('users').orderBy('lastLaunch').limit(20)
fusersQuery.getDocuments()
```

```
allow list; if request.query.limit <= 20 && request.query.orderBy.lasetLaunch == "ASC";
```



- **사용자가 요청한 시간에 따라 보안규칙을 적용할 경우 (request.time)  **

```firestore
//요청한 시간이 2019년 3월 24일 이후 일때만 읽기 허용
allow read: if request.time.toMillis() >=
             timestamp.date(2020,1,1).toMillis(); 

```

```firestore
//데이터 베이스에 저장된 publication timestamp 와 요청된 시간 비교 
// 1 millisecond = 0.001 seconds.

allow read: if request.time.toMillis() >=
            resource.data.publicAt.seconds * 1000;
```



- **time에 대한 조건을 생성할 경우 (duration) **

  - duration.time(hours, mins, secs, nanos) 

  ```firestore
   //마지막으로 전송된 알람시간 이후 1시간이 지난 문서만 read할 수 있다.
   
  allow read: if request.time > (resource.data.lastSendNotification + duration.time(1, 0, 0, 0));
  
  ```
  
  - duration.value(간격, 단위)  
    
    | Unit | Description  |
    | :--- | :----------- |
    | w    | Weeks        |
    | d    | Days         |
    | h    | Hours        |
    | m    | Minutes      |
    | s    | Seconds      |
    | ms   | Milliseconds |
    | ns   | Nanoseconds  |
```
match /notification/{notificationId} {
   allow create: if 
      request.time.toMillis() > getRequiredTimeInMillis();
            
   function getRequiredTimeInMillis() {
     return (getLastSendNotificationTimestamp().seconds +
                   duration.value(2, 'h').seconds()) * 1000;
   }
             
   function getLastSendNotificationTimestamp() {
     return get(/databases/$(database)/documents/
        user/$(request.auth.uid)).data.lastSendNotification;
   }
}

//마지막으로 전송된 알람시간 이후 2시간이 지나야 문서를 create할 수 있다.
```

https://firebase.google.com/docs/reference/rules/rules.duration_



- **함수 사용**

반복적으로 사용하는 규칙의 경우 함수로 만들어놓으면 편리하다 

```
service cloud.firestore {

  match /databases/{database}/documents {
 
 //지정문서범위 바깥 쪽에 있는 함수
 	 function outerauthorOrPublished(storyid) {
        return resource.data.published == true || request.auth.uid == 	resource.data.author;
      }
     match /stories/{storyid} {
    
     //지정문서범위 내에 있는 함수
         function innerauthorOrPublished() {
           return resource.data.published == true || request.auth.uid == 	esource.data.author;
         }

        
         allow list: if request.query.limit <= 10 &&
                         innerauthorOrPublished();
    
         allow get: if outerauthorOrPublished(storyid); 
         allow write: if request.auth.uid == resource.data.author;
        }
      }
    } 
```

```firestore

```



- **반드시 주의해야할 점**

-**보안규칙은 필터가 아니다. **실제 데이터에 접근하는 것이 아니라 코드를 통해 가정된 데이터에 보안규칙을 적용하기 때문에 쿼리한 문서에 대해 보안규칙을 적용할 경우 코드 상에서의 쿼리 내용과 보안규칙에서의 쿼리 내용이 동일해야한다. 코드가 아닌 보안규칙에만 쿼리를 적용하면 조건에 맞지 않는 데이터가 들어올 수 있기 때문에 보안규칙이 해당 데이터를 거부할 수 있다. 

```dart
allow read, write: if request.auth.uid == resource.data.author;
```



올바른 예시)

```dart
FirebaseUser user = await _firebaseAuth.currentUser();
Firestore.instance.collection("stories").where("author", isEqualTo: user.uid)
         .getDocuments()  
```



잘못된 예시) 해당 쿼리를 가져올 수 없다.

```dart
FirebaseUser user = await _firebaseAuth.currentUser();
Firestore.instance.collection("stories").getDocuments()
```



-이메일 인증서비스를 사용하지 않을 때 이메일 관련 조건을 추가하면 오류가 생긴다.

-휴대폰 인증서비스에 대한 보안규칙 사용 시 Authentication에 저장된 형태인  phone_number == '821045241423' 형태만 허용가능하다.  '01045241423' 형태는 같지 않다고 인식하니 유의하자.





### 4. 프로젝트 별 보안규칙 적용  

> 보안규칙이 필요한 항목과 적용한 보안규칙을 적어주시면 감사하겠습니다 :) 



#### 농플

- 보안규칙이 필요한 항목 

  - 데이터 접근할 사용자에 대한 보안규칙

  - 문서 엑세스를 위한 보안규칙 

  

- 적용한 보안규칙 

```firestore

```



#### HEM

- 보안규칙이 필요한 항목 
  - 데이터 접근할 사용자에 대한 보안규칙
  - 문서 엑세스를 위한 보안규칙 
- 적용한 보안규칙 

```firestore

```



#### 허그인

- 보안규칙이 필요한 항목 
  - 데이터 접근할 사용자에 대한 보안규칙
  - 문서 엑세스를 위한 보안규칙 
- 적용한 보안규칙 

```firestore

```

