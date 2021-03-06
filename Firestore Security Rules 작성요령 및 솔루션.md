

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



```
service firebase.storage {  
match /b/{bucket}/o 
```

- cloud.storage 에 대해 규칙을 적용한다고 선언한다.



### 3. 보안규칙 작성 



###### 기본구조



- cloud.firestore 

    service cloud.firestore {
    
      match /databases/{database}/documents {
          match /stories/{storyid} {
          allow read, write: if request.auth != null;    
      	  }
    	}
    } 

- cloud.storage 

```
service firebase.storage {  
	match /b/{bucket}/o {    
		match /images/{imageId} {      
		allow read, write: if request.auth != null;    
		}  
	}
}
```







#### 1) 규칙을 적용할 문서를 지정한다.



[cloud firestore]

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



[cloud storage]

- 모든 폴더에 대한 보안규칙 적용 

```firestore
match /{allPaths=**} {
```



- profile_imgs 폴더 내 모든 하위 범주에 대해 보안규칙 적용

```firestore
match /profile_imgs/{allPaths=**} {
```



- profile_imgs의 모든 폴더들에 대한 보안규칙 적용 

```firestore
match /profile_imgs/{profile_img}{
```



- profile_imgs의 모든 폴더의 하위 파일들에 대한 보안규칙 적용 

```firestore
match /profile_imgs/{profile_img}/{filename}{
```



- profile_imgs/2s6xdCuygpMv0ralWTnxbRZcYo73 폴더에 대한 보안규칙 적용

```firestore
match /profile_imgs/2s6xdCuygpMv0ralWTnxbRZcYo73 {
```



- profile_imgs/2s6xdCuygpMv0ralWTnxbRZcYo73/3409.jpg 폴더에 대한 보안규칙 적용

```firestore
match /profile_imgs/2s6xdCuygpMv0ralWTnxbRZcYo73/3409.jpg{
```





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



**cloud firestore**

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



> **request.resource.data란?**
>
> 사용자가 write하기를 시도하는 document의 모든 필드를 표현한다.

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



> **resource.data란? **
>
> 이미  데이터베이스에 있는 문서를 나타내는 것 



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

  
  
  - exist  : 다른 문서에 데이터 존재 여부를 알기 위함
  - get  : 지정 문서 외에 다른 문서를 가져와서 비교   



*exist는 다른 문서에 특정 문서가 존재하는지를 판별한다. 

특성 필드 값이 존재하는지를 확인하기 위해서는 get을 사용한다.



<u>exist 예시</u>  


```firestore
//users collection내에 특정 uid의 문서가 존재하는가  

if exists(/databases/$(database)/documents/users/$(request.auth.uid))
if exists(/databases/$(database)/documents/Users/$('2s6xdCuygpMv0ralWTnxbRZcYo73'));

//users subcollection내에 request.auth.uid인 문서가 존재하는가  

if exists(/databases/$(database)/documents/leagues/$(league)/users/$(request.auth.uid))} 
```

    //Users 컬렉션 내 특정문서의 'name' 필드를 write하려고 한다. 
    //새로운 name의 필드값이  Kits의 document id에 존재할 때만 허용한다.
    
     match /Users/{user} {
    		allow write :if exists(/databases/$(database)/documents/Kits/$(request.resource.data.name));
    }
```firestore
//Users 컬렉션 내 특정 문서의 name 필드를 read 혹은 write하려고 한다. 
//write하고자 하는 문서의 birth 필드값이 Kits의 document id로 존재할 때 허용한다. 

match /Users/{user} {
			allow read, write :if exists(/databases/$(database)/documents/Kits/$(resource.data.birth));
    }
```

*request.resource.data는 사용자가 데이터베이스에 write하기를 시도하는 데이터임으로 read에는 적용할 수 없다. 반면 resource.data는 현재 데이터베이스에 존재하는 데이터임으로 read에 적용할 수 있다.



<u>get 예시</u> 

```firestore
//request.auth.uid인 문서의 admin 필드가 true인가    
if get(/databases/$(database)/documents/users/$(request.auth.uid)).data.admin == true
```

```firestore
//map 형태의 필드값을 가져올 때
// (roles / request.auth.uid / role / isadmin : true) 일때 승인

match /posts/{docId} {    
    allow read, write: if hasRole("isAdmin")  
}      
function hasRole(userRole) {    
  return get(/databases/$(database)/documents/roles/$(request.auth.uid)).data.role[userRole] 
}
```

```firestore
//map 형태의 필드값을 가져올 때
//(restaurants/restaurantID/ private_data (sub collection) / private / roles / request.auth.uid == editor or owner ) 일 때 승인

if get(/databases/$(database)/documents/restaurants/$(restaurantID)/private_data/private).data.roles[request.auth.uid] == ["editor" ,"owner"]
```

> $(database) :  Cloud Firestore Security Rules 와일드 카드로 사용할 수 있는 데이터베이스 인스턴스를 정의하는 매개변수 
>
> *match /databases/**{database}**/documents*







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



**cloud storage**

- 사용자 인증은 cloud firestore와 같다. 

- 문서 엑세스에 관한 추가 정보는 문서 하단의 공식문서를 참조하자 

```
//파일의 사이즈 

if request.resource.size < 5 * 1024 * 1024
```

```
//파일의 타입 

if request.resource.contentType.matches('image/.*') 
	&& request.resource.contentType == resource.contentType;
```

```
 //file name에 대한 보안규칙
  
 match /{imageId} {       
 allow write: imageId == "profilePhoto.png" && imageId.size() < 32     
 }   
```



**과금 정책 **

- 규칙 평가는 요청별로 한 번만 요금이 부과되기 때문에 문서를 한 번에 하나씩 읽는 것보다 한 번에 여러 문서를 읽는 편이 비용이 적게 든다.
- 요청한 규칙이 문서를 여러 번 참조하더라도 읽기 요금은 종속 문서당 1회만 청구된다.
- 하단의 공식문서를 참조





### 4. 프로젝트 별 보안규칙 적용  

> 보안규칙이 필요한 항목과 적용한 보안규칙을 적어주시면 감사하겠습니다 :) 



#### 농플

- 보안규칙이 필요한 항목 

  - 데이터 접근할 사용자에 대한 보안규칙

  - 문서 엑세스를 위한 보안규칙 


< User collection >

_mapLoggedInToState()

_mapEmailDuplicationChangedToState() 이메일 중복확인 



[Read] [create] [update]

- 로그인 인증 전에 User collection 데이터가 필요하다면 모두 접근 

- 그렇지 않다면 로그인 인증이 된 사용자만 가능 (uid 존재)

[delete] 

- 불가능 (탈퇴가능 여부에 따름)



< facility collection>

-홈화면에서 mapPageLoadedToState()

-facility setting bloc에서 _mapPageLoadedToState()

-account  삭제 시 정보 불러옴

-facility 정보 추가

-실시간 'skyNow'    'tempNow'    'humidity'  업데이트 



[Read]

- 로그인 인증이 된 사용자만 가능 (uid 존재)

- 현재 사용자의 uid가 facility의 uid 필드에 존재할 때

[Write]

- 로그인 인증이 된 사용자만 가능 (uid 존재)

- 현재 사용자의 uid가 요청하는 facility의 uid로 존재할 때 



< Account collection> 

그래프 그릴 때 (fid가 같고 주어진 기간 내에 있는 정보 쿼리)

account bloc의 _mapPageLoadedToState (주어진 fid가 같고 count와 date에 따라 정렬 count가 0보다 크고 3개씩)

getDailyAccount (주어진 날짜와 fid가 같은 데이터)

_mapAllDailyAcntLoadedToState (fid가 같고 count가 0보다 클 때 )

account 삭제 (주어진 fid가 같은 정보)



[Read]

- 로그인 인증이 된 사용자만 가능 (uid 존재)

- 나의 uid가 기존 문서의 field 에 존재할 때 
- 기존문서의 fid가 facility의 document id로 존재할 때

- 문서의 count필드가 0보다 크거나 같을 때 

[Write] 

- 로그인 인증이 된 사용자만 가능 (uid 존재)

- 요청하는문서의  uid가 User의 document id로 존재할 때  
- 요청하는 문서의 uid가 나의 uid와 같을 때 

- 요청하는 문서의 fid가 facility의 document id로 존재할 때



< Categories >
작물 정보 추가할 때 식물 이름 검색 

[Read] 

- 로그인 인증이 된 사용자만 가능 (uid 존재)

- 해당 유저의 uid가 User collection에 존재할 때

[write] 

- 불가능



< journal collection >
[Read] 

- 로그인 인증이 된 사용자만 가능 (uid 존재)

- fid가 facility의 document id로 존재할 때 

[Write]

- 로그인 인증이 된 사용자만 가능 (uid 존재)

- 요청하는 문서의 fid가 facility에 존재할 때 





 < keyword collection>

[Read]

- 로그인 인증이 된 사용자만 가능 (uid 존재)

- fid가 facility의 document id로 존재할 때 

[Write]

- 로그인 인증이 된 사용자만 가능 (uid 존재)

- 요청하는 문서의 fid가 facility에 존재할 때 

 

< MonthlyAccount collection>

< WeeklyAccount collection>

< YearlyAccount collection >

[Read]

- 로그인 인증이 된 사용자만 가능 (uid 존재)

- fid가 facility의 document id로 존재할 때 

- 기존 문서의 uid들이 Users collection의 document id 로 존재할 때

[Write]

- 로그인 인증이 된 사용자만 가능 (uid 존재)
- fid가 facility의 document id로 존재할 때 

- 요청하는 문서필드의 uid가 Users collection의 document id 로 존재할 때
- 요청하는 문서필드의 uid가 나의 uid와 같을 때  



< Report collection>

< ReportFavorite collection>

[Read]

- 로그인 인증이 된 사용자만 가능 (uid 존재)

- fid가 facility의 document id로 존재할 때 
- content가 null 아닐 때 
- writer가 SYN 데이터팀인 정보만 불러오기 

[Write]

- 로그인 인증이 된 사용자만 가능 (uid 존재)

- fid가 facility의 document id로 존재할 때 

- content가 null 아닐 때 



< picture collection>

[Read]

- 로그인 인증이 된 사용자만 가능 (uid 존재)

- fid가 facility의 document id로 존재할 때 

[Write]

- 로그인 인증이 된 사용자만 가능 (uid 존재)

- fid가 facility의 document id로 존재할 때 

- 요청하는 문서의 path 필드가 null이 아닐 때 



< tag collection> 

[Read] 

- 로그인 인증이 된 사용자만 가능 (uid 존재)

- tag의 fid가 facility의 uid로 존재할 때
- 현재 요청하는 tag의 tag가 null이 아닐 때      

[Write]

- 로그인 인증이 된 사용자만 가능 (uid 존재)

- 현재 요청하는 tag의 fid가 facility의 uid로 존재할 때

- 현재 요청하는 tag의 tag가 null이 아닐 때      

**중복된 보안규칙 항목들이 많으니 함수사용



- 적용한 보안규칙 

```firestore

```



#### HEM

- **보안규칙이 필요한 항목**  
  
  - **데이터 접근할 사용자에 대한 보안규칙**
  - **문서 엑세스를 위한 보안규칙 **
  
  
  
  < phone collection (추가필요) >
  
  [read] 
  
  로그인 전 휴대폰 번호 읽기 
  
  ​       -아무나 다 가능 
  
  
  
  < Users collection >
  
  [read] 
  
  로그인 후 본인과 본인자녀의 정보만 읽기  
  
  사진 삭제 시 family fid가 같은 데이터 가져오기  
  
    - 현재 사용자의 인증된 번호와 Users collection에 저장된 번호가 같은 데이터만 읽기 가능 
     - 로그인이 되어있을 때 (uid와 phone 정보가 존재할 때 )
  
  [create] 
  
  회원가입 로그인 인증 후 Users collection에 정보 추가할 때 
  
  Users collection에 자녀 정보 추가할 때
  
  - 로그인이 되어있을 때 (uid와 phone 정보가 존재할 때 )
  - 현재 사용자의 인증된 번호와 추가하고자 하는 문서의 phone field 정보가 같을 때 
  
  - Users collection시 field 타입이 맞을 경우만 엑세스 허용 
  
  [update] 
  
  휴대폰 번호 업데이트 
  
  프로필사진 업데이트 
  
   stage flag , fcmtoken 업데이트 (?)
  
  - 로그인이 되어있을 때 (uid와 phone 정보가 존재할 때 )
  
  - phone 필드는 string type이며 10자 혹은 11자만 가능
  - name , fid, regdttm, uid필드 변경되지 않도록 
  
  [delete]
  
  본인 탈퇴
  
  자녀 정보 삭제 
  
  - 로그인이 되어있을 때 (uid와 phone 정보가 존재할 때 )
  - 현재 사용자의 인증된 번호와 Users collection에 저장된 번호가 같은 데이터만 삭제 가능
  
  
  
  
  
  < Kits collection >
  
  [read] 
  
  본인 kit 정보만 읽기 
  
  - 로그인이 되어있을 때 (uid와 phone 정보가 존재할 때 )
  
  - kit의 uid 필드가 users의 document id로 존재할 때  
  
  [create]
  
  키트 추가 
  
  - 로그인이 되어있을 때 (uid와 phone 정보가 존재할 때 )
  - valid kit 컬렉션 문서 id에 요청하는 kit name이 존재할 때 (kit 추가)
  - 요청하는 kit name이 데이터베이스에 저장되어 있는 kit uid에 존재하지 않을 때  
  
  [update]
  
  키트 정보 업데이트 
  
  - 로그인이 되어있을 때 (uid와 phone 정보가 존재할 때 )
  
  
  
  - name , kid , regdttm 변화 되지 않도록   
  
  [delete]
  
  - 로그인이 되어있을 때 (uid와 phone 정보가 존재할 때 )
  
  
  
  < Results collection >
  
  [read] 
  
  본인 result 정보만 읽기 
  
  - 로그인이 되어있을 때 (uid와 phone 정보가 존재할 때 )
  
  - results의 id 정보가 kits의 document id로 존재할 때		
  
  [write]
  
  - 불가능
  
  
  
  < valid kits >
  
  [read]
  
  로그인 되있는 사람
  
  [create, update]
  
  불가능
  
  [delete]
  
  로그인 되있는 사람

​		

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







> **Firestore  API 문서**
>
> https://firebase.google.com/docs/reference/rules/rules.Boolean
>
> **Firestore 공식 보안규칙문서**
>
> https://firebase.google.com/docs/firestore/security/get-started
>
> **Firestore 공식 과금정책 문서  (보안규칙)**
>
> https://firebase.google.com/docs/firestore/pricing
>
> **cloud storage 공식 보안규칙 문서** 
>
> https://firebase.google.com/docs/storage/security/secure-files?hl=ko





