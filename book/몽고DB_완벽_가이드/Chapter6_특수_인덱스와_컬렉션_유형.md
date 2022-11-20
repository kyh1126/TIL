# Chapter 6. 특수 인덱스와 컬렉션 유형

## 1. 공간 정보 인덱스

---

- 몽고DB는 2dsphere와 2d라는 공간 정보 인덱스를 가진다.
    - WGS84 좌표계를 기반으로 지표면을 모델링하는 구면 기하학으로 작동한다.
- 공간 정보 인덱스: `createIndex`와 함께 "2dsphere"를 사용
    
    ```bash
    > db.openStreetMap.createIndex({"loc" : "2dsphere"})
    ```
    
    - 인덱싱할 도형이 포함된 필드(여기서는 "loc")를 지정하는 도큐먼트를 `createIndex`에 전달하고 "2dsphere"를 값으로 지정한다.
- 2dsphere 인덱스를 사용하면 지구의 형태를 고려하므로 2d 인덱스를 사용할 때보다 더 정확한 거리 계산을 할 수 있다.

### 1-1. 공간 정보 쿼리 유형

---

- 공간 정보 쿼리는 교차, 포함, 근접이라는 세 가지 유형이 있다.
    - 찾을 항목을`{"$geometry" : geoJsonDesc}`와 같은 GeoJSON 객체로 지정한다.
- `$geoIntersects`: 쿼리 위치와 교차하는 도큐먼트를 찾을 수 있다.
    
    ```bash
    > var eastVillage = {
      "type": "Polygon",
      "coordinates": [
        [
          [ -73.9732566, 40.7187272 ], [ -73.9724573, 40.7217745 ], [ -73.9717144, 40.7250025 ], [ -73.9714435, 40.7266002 ], [ -73.975735, 40.7284702 ], [ -73.9803565, 40.7304255 ], [ -73.9825505, 40.7313605 ], [ -73.9887732, 40.7339641 ], [ -73.9907554, 40.7348137 ], [ -73.9914581, 40.7317345 ], [ -73.9919248, 40.7311674 ], [ -73.9904979, 40.7305556 ], [ -73.9907017, 40.7298849 ], [ -73.9908171, 40.7297751 ], [ -73.9911416, 40.7286592 ], [ -73.9911943, 40.728492 ], [ -73.9914313, 40.7277405 ], [ -73.9914635, 40.7275759 ], [ -73.9916003, 40.7271124 ], [ -73.9915386, 40.727088 ], [ -73.991788, 40.7263908 ], [ -73.9920616, 40.7256489 ], [ -73.9923298, 40.7248907 ], [ -73.9925954, 40.7241427 ], [ -73.9863029, 40.7222237 ], [ -73.9787659, 40.719947 ], [ -73.9772317, 40.7193229 ], [ -73.9750886, 40.7188838 ], [ -73.9732566, 40.7187272 ]
        ]
      ]
    }
    # 뉴욕의 이스트빌리지 내 한 점을 갖는 점, 선, 다각형이 포함된 도큐먼트를 모두 찾는다.
    > db.openStreetMap.find({"loc" : {"$geoIntersects" : {"$geometry" : eastVillage}}})
    ```
    
- `$geoWithin`: 특정 지역에 완전히 포함된 항목을 쿼리할 수 있다.
    
    ```bash
    # 이스트빌리지를 가로지르거나 이스트빌리지와 부분적으로 겹치는 다각형은 반환하지 않는다.
    > db.openStreetMap.find({"loc" : {"$geoWithin" : {"$geometry" : eastVillage}}})
    ```
    
- `$near`: 주변 위치에 쿼리할 수 있다. 정렬을 포함하는 유일한 공간 정보 연산자다. 결과는 항상 거리가 가장 가까운 곳부터 가장 먼 곳 순으로 반환된다.
    
    ```bash
    db.openStreetMap.find({"loc" : {"$near" : {"$geometry" : eastVillage}}})
    ```
    

### 1-2. 공간 정보 인덱스 사용

---

- 몽고DB의 공간 정보 인덱스를 사용하면 특정 지역과 관련된 모양과 점이 포함된 컬렉션에서 공간 쿼리를 효율적으로 실행할 수 있다.

- 쿼리에서의 2D vs. 구면 기하학
    - 공간 정보 쿼리는 쿼리와 사용 중인 인덱스 유형에 따라 구면 또는 2D(평면) 구조를 사용한다.
    - 몽고DB의 쿼리 유형 및 도형 유형
        
        
        | 쿼리 유형 | 기하 구조 유형 |
        | --- | --- |
        | $near (GeoJSON point, 2dsphere 인덱스) | 구면 |
        | $near (레거시 좌표, 2d 인덱스) | 평면 |
        | $geoNear (GeoJSON point, 2dsphere 인덱스) | 구면 |
        | $geoNear (레거시 좌표, 2d 인덱스) | 평면 |
        | $nearSphere (GeoJSON point, 2dsphere 인덱스) | 구면 |
        | $nearSphere (레거시 좌표, 2d 인덱스) | 구면 |
        | $geoWithin : { $geometry: …} | 구면 |
        | $geoWithin : { $box: …} | 평면 |
        | $geoWithin : { $polygon: …} | 평면 |
        | $geoWithin : { $center: …} | 평면 |
        | $geoWithin : { $centerSphere: …} | 구면 |
        | $geoIntersects | 구면 |
    - 2d 인덱스는 구에서 평면 기하학과 거리 계산을 모두 지원한다(즉 `$nearSphere`를 사용한다)는 점에 유의하자.
    - `$geoNear` 연산자는 집계 연산자라는 점에 유의하자.
        - geoNear 명령과 `$geoNear` 집계 연산자를 사용하려면 컬렉션에 2dsphere 인덱스와 2d 인덱스가 최대 1개만 있어야 한다.
        - `$near`나 `$geoWithin`과 같은 공간 정보 쿼리 연산자는 컬렉션이 여러 개의 공간 정보 인덱스를 갖도록 허용한다.

- 왜곡
    - 구면 기하는 지도에 시각화하면 왜곡이 있는데, 지구와 같은 3차원 구를 평면에 투사하는 특성 때문이다.

- 레스토랑 검색
    - 뉴욕에 위치한 지역(https://oreil.ly/rpGna)과 레스토랑(https://oreil.ly/JXYd-) 데이터셋을 이용해 작업한다.
    - `mongoimport` 도구를 사용해 데이터셋을 데이터베이스로 가져오자.
        
        ```bash
        # mongoimport ~/neighborhoods.json -c neighborhoods
        2022-11-20T07:48:13.958+0000	connected to: mongodb://localhost/
        2022-11-20T07:48:14.314+0000	195 document(s) imported successfully. 0 document(s) failed to import.
        
        # mongoimport ~/restaurants.json -c restaurants
        2022-11-20T07:48:58.904+0000	connected to: mongodb://localhost/
        2022-11-20T07:48:59.420+0000	25359 document(s) imported successfully. 0 document(s) failed to import.
        ```
        
    - `createIndex` 명령을 사용해 각 컬렉션에서 2dsphere 인덱스를 만든다.
        
        ```bash
        test> db.neighborhoods.createIndex({location:"2dsphere"})
        location_2dsphere
        test> db.restaurants.createIndex({location:"2dsphere"})
        location_2dsphere
        ```
        
- 데이터 탐색
    
    ```bash
    # 뉴욕 내 헬스 키친(클린턴) 지역
    test> db.neighborhoods.find({name: "Clinton"})
    [
      {
        _id: ObjectId("55cb9c666c522cafdb053a71"),
        geometry: {
          coordinates: [
            [
              [ -73.99383108136983, 40.772931787850595 ],
    ...
              [ -73.99416328456306, 40.77248719102126 ],
              [ -73.99383108136983, 40.772931787850595 ]
            ]
          ],
          type: 'Polygon'
        },
        name: 'Clinton'
      }
    ]
    
    # '424 West 43rd Street'에 위치한 'Little Pie Company'
    test> db.restaurants.find({name: "Little Pie Company"})
    [
      {
        _id: ObjectId("55cba2476c522cafdb053dea"),
        location: { coordinates: [ -73.99331699999999, 40.7594404 ], type: 'Point' },
        name: 'Little Pie Company'
      }
    ]
    ```
    
- 현재 지역 찾기
    - `$geoIntersects`: 사용자가 현재 위치한 지역을 쉽게 찾을 수 있다.
        
        ```bash
        # 사용자 위치의 경도 좌표는 -73.93414657, 위도 좌표는 40.82302903이라고 가정
        # GeoJSON 형식의 특수한 $geometry 필드로 점을 지정해 현재 지역(헬스 키친)을 찾는다.
        test> db.neighborhoods.findOne({geometry:{$geoIntersects:{$geometry:{type:"Point", coordinates:[-73.93414657,40.82302903]}}}})
        {
          _id: ObjectId("55cb9c666c522cafdb053a68"),
          geometry: {
            coordinates: [
              [
                [ -73.93383000695911, 40.81949109558767 ],
                [ -73.93411701695138, 40.81955053491088 ],
                ...
              ]
            ],
            type: 'Polygon'
          },
          name: 'Central Harlem North-Polo Grounds'
        }
        ```
        
- 지역 내 모든 레스토랑 찾기
    - 특정 지역 내 레스토랑을 모두 찾는 쿼리: 사용자가 위치한 지역을 찾은 후 해당 지역 내 레스토랑 수를 계산한다.
    - ex> 헬스 키친 내에 있는 레스토랑을 모두 찾아보자
        
        ```bash
        test> var neighborhood = db.neighborhoods.findOne({
          geometry: {
            $geoIntersects: {
              $geometry: {
                type: "Point",
                coordinates: [-73.93414657,40.82302903]
              }
            }
          }
        });
        
        # 해당 지역에 위치한 레스토랑의 이름을 모두 알려준다.
        test> db.restaurants.find({
          location: {
            $geoWithin: {
              // 앞에서 검색한 인접 객체의 기하 구조를 사용한다.
              $geometry: neighborhood.geometry
            }
          }
        },
        // 일치하는 레스토랑의 이름만 표시한다.
        {name: 1, _id: 0});
        [
          { name: 'Perfect Taste' },
          { name: 'Event Productions Catering & Food Services' },
        ...
          { name: 'To Your Health & Happiness' }
        ]
        Type "it" for more
        ```
        
- 범위 내에서 레스토랑 찾기
    - 특정 지점으로부터 지정된 거리 내에 있는 레스토랑을 찾을 수 있다.
    - "`$centerSphere`"와 함께 "`$geoWithin`"을 사용: 정렬되지 않은 순서로 결과를 반환
        - "`$centerSphere`": 중심과 반경을 라디안으로 지정해 원형 영역을 나타내는 몽고DB 전용 구문
        
        ```bash
        # 사용자로부터 5마일 이내에 있는 레스토랑을 모두 찾는다.
        test> db.restaurants.find({
          location: {
            $geoWithin: { $centerSphere: [ [-73.93414657,40.82302903], 5/3963.2 ] }
          }
        })
        [
          {
            _id: ObjectId("55cba2476c522cafdb0552d5"),
            location: { coordinates: [ -73.9911006, 40.76503719999999 ], type: 'Point' },
            name: "Cakes 'N Shapes"
          },
        ...
        ]
        Type "it" for more
        ```
        
    - "`$maxDistance`"와 함께 "`$nearSphere`"를 사용: 거리 순으로 정렬된 결과를 반환
        
        ```bash
        # 사용자로부터 5마일 이내에 있는 모든 레스토랑을 가장 가까운 곳에서 가장 먼 곳 순으로 반환한다.
        test> var METERS_PER_MILE = 1609.34;
        
        test> db.restaurants.find({ location: { $nearSphere: { $geometry: { type: "Point",coordinates: [-73.93414657,40.82302903] }, $maxDistance: 5*METERS_PER_MILE } } });
        [
          {
            _id: ObjectId("55cba2476c522cafdb058c83"),
            location: { coordinates: [ -73.9316894, 40.8231974 ], type: 'Point' },
            name: 'Gotham Stadium Tennis Center Cafe'
          },
        ...
        ]
        Type "it" for more
        ```
        

### 1-3. 복합 공간 정보 인덱스

---

- 공간 정보 인덱스는 다른 인덱스 종류와 마찬가지로 다른 필드와 묶음으로써 더 복잡한 쿼리를 최적화할 수 있다.
    
    ```bash
    > db.openStreetMap.createIndex({"tags" : 1, "location" : "2dsphere"})
    
    # 헬스 키친에 있는 피자 레스토랑을 빠르게 찾을 수 있다.
    > db.openStreetMap.find({"loc" : {"$geoWithin" : {"$geometry" : hellsKitchen.geometry}}, "tags" : "pizza"})
    ```
    

### 1-4. 2d 인덱스

---

- 비구체 지도에는 "2dsphere" 대신 "2d" 인덱스를 사용한다.
    
    ```bash
    > db.hyrule.createIndex({"tile" : "2d"})
    ```
    
    - 도큐먼트는 "2d" 인덱스 필드에 요소가 2개인 배열을 사용하며, 배열 요소는 각각 경도 좌표와 위도 좌표를 나타낸다.

- 기본적으로 2d 인덱스는 값이 -180과 180 사이에 있다고 가정한다. 범위를 넓히거나 좁히려면 `createIndex`를 사용해 최솟값과 최댓값을 지정한다.
    
    ```bash
    > db.hyrule.createIndex({"light-years" : "2d"}, {"min" : -1000, "max" : 1000})
    ```
    

- 2d 인덱스는 "`$geoWithin`", "`$nearSphere`", "`$near`" 쿼리 셀렉터를 지원한다.
    - "`$geoWithin`": 평평한 표면에 정의된 영역 내 점을 쿼리하는데 사용한다. 직사각형, 다각형, 원, 구 안에 있는 모든 점을 쿼리할 수 있으며 "`$geometry`" 연산자를 사용해 GeoJSON 객체를 지정한다.
        
        ```bash
        # 왼쪽 하단 모서리 [10, 10]과 오른쪽 상단 모서리 [100, 100]으로 정의된 사각형 내 도큐먼트에 대한 쿼리
        > db.hyrule.find({
          tile: { $geoWithin: { $box: [[10, 10], [100, 100]] } }
        })
        
        # 중심이 [-17, 20.5]이고 반지름이 25인 원 안에 있는 도큐먼트를 쿼리
        > db.hyrule.find({
          tile: { $geoWithin: { $center: [[-17, 20.5] , 25] } }
        })
        
        # [0, 0], [3, 6], [6, 0]으로 정의된 다각형 내 좌표를 포함하는 도큐먼트를 모두 반환
        > db.hyrule.find({
          tile: { $geoWithin: { $polygon: [[0, 0], [3, 6], [6, 0]] }
        })
        ```
        
        - `$box`: 요소가 2개인 배열을 사용한다. 첫 번째 요소는 왼쪽 하단 모서리 좌표를, 두 번째 요소는 오른쪽 상단 모서리 좌표를 지정한다.

- 몽고DB는 레거시 지원을 위해 2d 인덱스에 대한 기초적인 구형 쿼리도 지원한다.
    
    ```bash
    > db.hyrule.find({
      loc: { $geoWithin: { $centerSphere: [[88, 30], 10/3963.2] } }
    })
    
    # (20, 21)에 가장 가까운 10개의 도큐먼트를 반환
    > db.hyrule.find({"tile" : {"$near" : [20, 21]}}).limit(10)
    ```
    
    - 구 안에 있는 레거시 좌표 쌍을 쿼리: "`$centerSphere`" 연산자와 함께 "`$geoWithin`"을 사용한다.
    - 주변에 있는 점을 쿼리: "`$near`"를 사용한다. 제한이 지정되지 않았으면 기본적으로 100개의 도큐먼트로 제한이 적용된다.

## 2. 전문 검색을 위한 인덱스

---

- 몽고DB의 text 인덱스는 전문 검색의 요구 사항을 지원한다.
    - 애플리케이션 사용자가 제목, 설명 등 컬렉션 내에 있는 필드의 텍스트와 일치시키는 키워드 쿼리를 하게 하려면 text 인덱스를 사용하자.

- 정규 표현식을 이용해 문자열에 쿼리 → 이 방식에는 다소 한계가 있다.
    - 쿼리 속도가 느리며, 문법과 같은 언어 특성을 반영하기도 쉽지 않다.(ex> "entry"는 "entries"와 일치해야 한다)

- text 인덱스는 텍스트를 빠르게 검색하는 기능을 제공하며, 언어에 적합한 토큰화, 정지 단어, 형태속 분석 등 일반적인 검색 엔진 요구 사항을 지원한다.
- text 인덱스를 만들면 시스템 리소스가 많이 소비될 수 있다.
    - 텍스트 검색을 사용하면 문자열이 토큰화되고, 형태소화되며, 인덱스는 잠재적으로 여러 위치에서 갱신된다.
    - text로 인덱싱된 컬렉션에서는 쓰기 성능이 다른 컬렉션에서보다 떨어지는 경향이 있다.
    - 샤딩을 하면 데이터 이동 속도가 느려지며, 모든 텍스트는 새 샤드로 마이그레이션될 때 다시 인덱싱돼야 한다.

### 2-1. 텍스트 인덱스 생성

---

- ex> 위키피디아 문서 컬렉션에 인덱싱한다.
    - 가중치를 지정하면 몽고DB가 각 필드에 지정하는 상대적 중요도를 제어할 수 있다.
    - 인덱스를 생성한 후에는 삭제하고 다시 생성하지 않는 한 필드 가중치를 변경할 수 없다.
    - "`$ **`"에 인덱스를 만들면 도큐먼트의 모든 문자열 필드에 전문 인덱스를 생성할 수 있다. 모든 최상위 문자열 필드를 인덱싱할 뿐 아니라 내장 도큐먼트와 배열에서 문자열 필드를 검색한다.
    
    ```bash
    # createIndex 호출은 "title"과 "body" 필드 내 용어를 기반으로 인덱스를 생성한다.
    > db.articles.createIndex({"title": "text", "body" : "text"})
    
    # "body" 필드와 "title" 필드에 3 : 2 비율의 가중치를 부여한다.
    > db.articles.createIndex({"title": "text", "body": "text"},
      {"weights" : { "title" : 3, "body" : 2}})
    
    # 컬렉션에 따라 도큐먼트에 포함될 필드를 모를 수도 있다.
    > db.articles.createIndex({"$**" : "text"})
    ```
    

### 2-2. 텍스트 검색

---

- "`$text`" 쿼리 연산자를 사용해 text 인덱스가 있는 컬렉션에 텍스트 검색을 수행하자.
    - 공백과 (대부분의) 구두점을 구분 기호로 사용해 검색 문자열을 토큰화하며, 검색 문자열에서 모든 토큰의 논리적 OR를 수행한다.
        
        ```bash
        # "impact"나 "crater" 또는 "lunar"라는 용어가 포함된 기사를 모두 찾는다.
        # 여기서 인덱스는 문서 제목과 본문 내 용어를 기반으로 하므로, 쿼리는 두 필드 중 하나에 해당 용어를 포함하는 도큐먼트와 일치한다.
        > db.articles.find(
          {"$text": {"$search": "impact crater lunar"}}, {title: 1}
        ).limit(10)
        # -> 쿼리 결과들이 그다지 관련성이 없음
        # 1. 쿼리가 매우 광범위하다.
        # 2. 텍스트 검색이 기본적으로 결과를 관련성에 따라 정렬하지 않는다.
        ```
        
    - 텍스트를 큰따옴표로 묶어서 정확히 일치하는 구문을 검색할 수 있다.
        
        ```bash
        # "impact crater" AND "lunar"
        > db.articles.find(
          {$text: {$search: "\"impact crater\" lunar"}}, {title: 1}
        ).limit(10)
        ```
        
    - 쿼리의 개별 용어 사이에 논리적 AND를 실행하려면 각 용어를 큰따옴표로 묶어 구문으로 처리하자.
        
        ```bash
        # "impact crater" AND "lunar" AND "meteor"
        > db.articles.find(
          {$text: {$search: "\"impact crater\" \"lunar\" \"meteor\""}},
          {title: 1}
        ).limit(10)
        ```
        

- 텍스트 쿼리를 사용하면 각 쿼리 결과에 메타데이터가 연결된다.
    - 메타데이터는 `$meta` 연산자를 사용해 명시적으로 투영하지 않는 한 쿼리 결과에 표시되지 않는다.
    - 관련성 스코어는 "textScore"라는 메타 데이터 필드에 저장된다.
    
    ```bash
    # 내림차순으로 정렬된다.
    > db.articles.find(
      {$text: {$search: "\"impact crater\" lunar"}},
      {title: 1, score: {$meta: "textScore"}}
    ).sort({score: {$meta: "textScore"}})
    .limit(10)
    ```
    
- 집계 파이프라인에서도 텍스트 검색을 사용한다.

### 2-3. 전문 검색 최적화

---

- 전문 검색을 최적화하는 방법은 두 가지다.
    - 다른 기준으로 검색 결과를 좁힐 수 있다면 복합 인덱스를 생성할 때 다른 기준을 첫 번째로 두고 전문 필드를 그 다음으로 둔다. (파티셔닝)
        
        ```bash
        # 전문 인덱스를 "date"에 따라 몇 개의 작은 트리 구조로 쪼갬 -> 파티셔닝
        > db.blog.createIndex({"date" : 1, "post" : "text"})
        ```
        
    - 다른 기준을 뒤쪽에 두어 사용할 수도 있다.
        
        ```bash
        # "author"와 "post" 필드만 반환한다면 두 필드에 대해 복합 인덱스를 생성할 수 있다.
        > db.blog.createIndex({"post" : "text", "author" : 1})
        
        # 접두사 방식과 접미사 방식을 합칠 수도 있다.
        > db.blog.createIndex({"date" : 1, "post" : "text", "author" : 1})
        ```
        

### 2-4. 다른 언어로 검색하기

---

- 언어에 따라 형태소 분석 방법이 다르므로 인덱스나 도큐먼트가 어떤 언어로 쓰였는지 명시해야 한다.

- text 인덱스에는 "`default_language`" 옵션을 지정할 수 있고 기본값은 "english"로 설정되지만 다양한 언어로 설정할 수 있다.
    
    ```bash
    # 프랑스어 인덱스
    # 별도로 언어를 지정하지 않는 한 프랑스어가 형태소 분석에 사용된다.
    > db.users.createIndex(
      {"profil" : "text", "intérêts" : "text"},
      {"default_language" : "french"}
    )
    ```
    
- 도큐먼트의 언어를 "language" 필드에 명시해 도큐먼트별로 형태소 분석 언어를 다르게 지정할 수 있다.
    
    ```bash
    db.users.insert({"username" : "swedishChef", "profile" : "Bork de bork", language : "swedish"})
    ```
    

## 3. 제한 컬렉션

---

- 몽고DB의 '일반적인' 컬렉션은 동적으로 생성되고 추가적인 데이터에 맞춰 크기가 자동으로 늘어난다.
- 몽고DB는 제한 컬렉션이라는 다른 형태의 컬렉션을 지원하는데, 이는 미리 생성돼 크기가 고정된다.
    - 환형 큐처럼 동작한다. 빈 공간이 없으면 가장 오래된 도큐먼트가 지워지고 새로운 도큐먼트가 그 자리를 차지한다.
- 일반적으로 몽고DB TTL 인덱스는 와이어드타이거 스토리지 엔진에서 더 나은 성능을 발휘하므로 제한 컬렉션보다 권장된다.
    - TTL 인덱스는 날짜 유형 필드 값과 인덱스 TTL 값을 기반으로 일반 컬렉션에서 데이터가 만료되고 제거된다.
- 제한 컬렉션은 유연성이 부족하지만 로깅에는 나름 유용하다.

### 3-1. 제한 컬렉션 생성

---

- 일반 컬렉션과 달리 제한 컬렉션은 사용되기 전에 명시적으로 생성돼야 한다.
    - 제한 컬렉션을 생성하려면 `create` 명령어를 사용한다. 셸에서는 `createCollection`을 사용해 생성할 수 있다. 도큐먼트 수에 제한을 지정할 수 있다.
    
    ```bash
    # 10만 바이트 고정 크기로 제한 컬렉션 my_collection을 만든다.
    > db.createCollection("my_collection", {"capped": true, "size": 100000});
    
    # 도큐먼트를 100개로 제한
    > db.createCollection("my_collection2",
      {"capped": true, "size": 100000, "max": 100});
    ```
    

- 제한 컬렉션은 일단 생성되면 변경할 수 없다.(속성을 변경하려면 삭제 후 재생성해야 한다)
    - 따라서 크기가 큰 컬렉션은 생성하기 전에 신중히 검토하자.

- 기존의 일반 컬렉션을 제한 컬렉션으로 변환할 수 있다.
    - `convertToCapped` 명령어를 사용해 실행할 수 있다.
    
    ```bash
    > db.runCommand({"convertToCapped" : "test", "size" : 10000});
    ```
    

### 3-2. 꼬리를 무는 커서

---

- 꼬리를 무는 커서: 결과를 모두 꺼낸 후에도 종료되지 않는 특수한 형태의 커서
    - `tail -f`와 비슷하게 결과를 지속적으로 꺼내는 기능을 수행한다.
- 일반 컬렉션에서는 입력 순서가 추적되지 않기 때문에 꼬리를 무는 커서는 제한 컬렉션에만 사용한다.
    - 꼬리를 무는 커서는 아무런 결과가 없으면 10분 후에 종료되므로 이후에 컬렉션에 다시 쿼리하는 로직을 포함시켜야 한다.
    
    ```php
    # PHP
    # 커서는 시간이 초과되거나 누군가가 쿼리 작업을 중지할 때까지는 결과를 처리하거나 다른 결과가 더 도착하기를 기다린다.
    $cursor = $collection->find([], [
      'cursorType' => MongoDB\Operation\Find::TAILABLE_AWAIT,
      'maxAwaitTimeMS' => 100,
    ]);
    
    while (true) {
      if ($iterator->valid()) {
        $document = $iterator->current();
        printf("Consumed document created at: %s\n", $document->createdAt);
      }
    
      $iterator->next();
    }
    ```
    

## 4. TTL 인덱스

---

- TTL 인덱스를 이용해서 각 도큐먼트에 유효 시간을 설정할 수 있다. 도큐먼트는 미리 설정한 시간에 도달하면 지워진다.
    - 세션 스토리지와 같은 문제를 캐싱하는데 유용하다.
    - "`expireAfterSeconds`" 옵션을 명시해 TTL 인덱스를 생성한다.
    - 서버 시간이 도큐먼트 시간의 "`expireAfterSeconds`"초를 지나면 도큐먼트가 삭제된다.
    
    ```bash
    // 24시간 뒤 타임아웃
    > db.sessions.createIndex({"lastUpdated":1, {"expireAfterSeconds":60*60*24})
    ```
    

- 몽고DB는 TTL 인덱스를 매분마다 청소하므로 초 단위로 신경 쓸 필요 없다.
    - `collMod` 명령어를 이용해  "`expireAfterSeconds`"를 변경할 수 있다.
    
    ```bash
    > db.runCommand({
      "collMod" : "someapp.cache",
      "index" : {
        "keyPattern": {"lastUpdated" : 1},
        "expireAfterSeconds" : 3600
      }
    });
    ```
    
- 하나의 컬렉션에 TTL 인덱스를 여러 개 가질 수 있다.

## 5. GridFS로 파일 저장하기

---

- GridFS는 몽고DB에 대용량 이진 파일을 저장하는 메커니즘이다.
- 파일을 저장할 때 GridFS를 고려하는 데는 몇 가지 이유가 있다.
    - GridFS를 이용하면 아키텍처 스택을 단순화할 수 있다.
        - 이미 몽고DB를 사용 중이라면 파일 스토리지를 위한 별도의 도구 대신 GridFS를 사용하면 된다.
    - GridFS는 몽고DB를 위해 설정한 기존의 복제나 자동 샤딩을 이용할 수 있어, 파일 스토리지를 위한 장애 조치와 분산 확장이 더욱 쉽다.
    - GridFS는 사용자가 올린 파일을 저장할 때 특정 파일시스템이 갖는 문제를 피할 수 있다.
        - ex> GridFS는 같은 디렉터리에 대량의 파일을 저장해도 문제가 없다.
- 몇가지 단점도 있다.
    - 성능이 느리다. 몽고DB에서 파일에 접근하면 파일시스템에서 직접 접근할 때만큼 빠르지 않다.
    - 도큐먼트를 수정하려면 도큐먼트 전체를 삭제하고 다시 저장하는 방법밖에 없다.
        - 몽고DB는 파일을 여러 개의 도큐먼트로 저장하므로 한 파일의 모든 청크에 동시에 락을 걸 수 없다.

<aside>
💡 GridFS는 큰 변화가 없고 순차적인 방식으로 접근하려는 대용량 파일을 저장할 때 일반적으로 최선의 메커니즘이다.

</aside>

### 5-1. GridFS 시작하기: mongofiles

---

- `mongofiles`는 GridFS에서 파일을 올리고, 받고, 목록을 출력하고, 검색하고, 삭제할 때 등에 사용한다.
    
    ```bash
    # echo "Hello, world" > foo.tx
    # mongofiles put foo.tx
    2022-11-20T12:07:53.437+0000	connected to: mongodb://localhost/
    2022-11-20T12:07:53.438+0000	adding gridFile: foo.tx
    
    2022-11-20T12:07:53.526+0000	added gridFile: foo.tx
    
    # mongofiles list
    2022-11-20T12:08:21.871+0000	connected to: mongodb://localhost/
    foo.tx	13
    
    # rm foo.tx
    # mongofiles get foo.tx
    2022-11-20T12:08:48.249+0000	connected to: mongodb://localhost/
    2022-11-20T12:08:48.252+0000	finished writing to foo.tx
    
    # cat foo.tx
    Hello, world
    ```
    
    - `put`: 파일시스템으로부터 파일을 받아 GridFS에 추가한다.
    - `list`: GridFS에 올린 파일 목록을 보여줌
    - `get`: `put`의 반대 연산으로 GridFS에서 파일을 받아 파일시스템에 저장한다.
    - `search`: 파일명으로 GridFS에 저장된 파일을 검색
    - `delete`: GridFS에 저장된 파일을 삭제

### 5-2. 몽고DB 드라이버로 GridFS 작업하기

---

- 모든 클라이언트 라이브러리는 GridFS API를 가진다. ex> 파이몽고

### 5-3. 내부 살펴보기

---

- GridFS는 파일 저장을 위한 간단한 명세이며 일반 몽고DB 도큐먼트를 기반으로 만들어졌다.
    - GridFS 기본 개념: 대용량 파일을 청크로 나눈 후 각 청크를 도큐먼트로 저장할 수 있다.
- GridFS의 청크는 자체 컬렉션에 저장된다.
    - 청크는 기본적으로 `fs.chunks` 컬렉션에 저장되지만 바꿀 수 있다.
    - 청크 컬랙션 내 각 도큐먼트 구조
        
        ```bash
        {
          "_id" : ObjectId("..."),
          "n" : 0,
          "data" : BinData("..."),
          "files_id" : ObjectId("...")
        }
        ```
        
        - 다른 몽고DB 도큐먼트와 마찬가지로 청크도 고유한 "`_id`"를 가진다.
        - "`files_id`": 청크에 대한 메타데이터를 포함하는 파일 도큐먼트의 "`_id`"
        - "`n`": 다른 청크를 기준으로 하는 파일 내 청크의 위치
        - "`data`": 파일 내 청크의 크기(바이트 단위)
    
    - 각 파일의 메타데이터는 기본적으로 별도의 컬렉션인 `fs.files`에 저장된다.
        - `files` 컬렉션 내 각 도큐먼트는 GridFS에서 하나의 파일을 나타내며, 해당 파일과 관련된 어떤 메타데이터든 포함할 수 있다.
        - 사용자 정의 키 외에도 GridFS 명세에서 강제하는 키가 몇 개 더 있다
            - "`_id`": 파일의 고유ID, 각 청크에서 file_id 키의 값으로 저장
            - "`length`": 파일 내용의 총 바이트 수
            - "`chunkSize`": 파일을 구성하는 각 청크의 크기 (바이트)
            - "`uploadDate`": GridFS에 파일이 저장된 시간
            - "`md5`": 서버에서 생성된 파일 내용의 MD5 체크썸
    
    - `distinct` 명령어를 사용해 GridFS에 저장된 고유한 파일명 목록을 얻을 수 있다.
        
        ```bash
        > ds.fs.files.distinct("filename")
        [ "foo.txt" , "bar.txt" , "baz.txt" ]
        ```
        
    
    - 참고
        - [https://github.com/mongodb-the-definitive-guide-3e/mongodb-the-definitive-guide-3e/tree/master/chapter6](https://github.com/mongodb-the-definitive-guide-3e/mongodb-the-definitive-guide-3e/tree/master/chapter6)
        - [https://jackjeong.tistory.com/m/176](https://jackjeong.tistory.com/m/176)