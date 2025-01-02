---
author: 김규형
date: 2023-09-12 14:10:00 +0800
categories: [NHN Academy, CS]
tags: [CS, NHN Acedemy, Java]
render_with_liquid: true
---

# JSON in Java(23. 10. 27)

- *Maven* 에서 JSON in Java 설치
- `lib` 폴더에 넣어서 사용한다.
- json 모듈의 계층 구조
    
    ```java
    classDiagram
      Object <|-- CDL
      Object <|-- Cookie
      Object <|-- CookieList
      Object <|-- HTTP
      Object <|-- JSONArray
      Object <|-- JSONML
      Object <|-- JSONObject
      Object <|-- JSONPointer
      Object <|-- JSONString
      Object <|-- JSONTokener
      Object <|-- JSONWriter
    	JSONObject
    	JSONString
    	JSONTokener <|-- HTTPTokener
    	JSONTokener <|-- XMLTokener
    	JSONWriter <|-- JSONStringer
    	Throwable <|-- Exception
    	Exception <|-- RuntimeException
    	RuntimeException <|-- JSONException
    	JSONException <|-- JSONPointerException
    ```
    

### Constructor

- `JSONObject` 를 사용해서 json 객체를 생성
    
    ```java
    public static void main(String args[]) {
            JSONObject jsonObject = new JSONObject();
            jsonObject.put("name", "nhn");
            System.out.println(jsonObject.toString());
        }
    ```
    
- 다른 json 객체를 value로 넣은 예제
    
    ```java
    public static void main(String args[]) {
            JSONObject jsonObject = new JSONObject();
            JSONObject addressJsonObject = new JSONObject();
            addressJsonObject.put("code", 13487);
            addressJsonObject.put("city", "Seongnam");
            jsonObject.put("address", addressJsonObject);
            jsonObject.put("name", "nhn");
            System.out.println(jsonObject.toString());
        }
    ```
    

### JSON 문자열

- Json을 이종 시스템간 주고 받을 경우 문자열 형태로 많이 사용된다.
- 문자열로 표현된 JSON Object
    
    `"{\"name\" : \"nhn\"}”`
    
- 이러한 문자열을 이종 시스템에서 받아 object를 생성하는 생성자도 존재
    
    `JSONObject(String source)`
    
    ```
    public static void main(String args[]) {
            String jsonText = "{\"name\" : \"nhn\"}";
    
            try {
            JSONObject object = new JSONObject(jsonText);
    
            System.out.println(object.toString());
            } catch (JSONException e) {
                e.printStackTrace();
            }
    ```
    
- 문자열 뿐만 아니라 객체로도 json 객체 생성 가능
    
    ```java
    public class Person {
        String name;
    
        public Person(String name) {
            this.name = name;
        }
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    }
    
    Person person = new Person("nhn");
            try {
            JSONObject object = new JSONObject(person);
    
            System.out.println(object);
            } catch (JSONException e) {
                e.printStackTrace();
            }
    ```
    
- 값을 변경 할때에도 `put` 을 활용한다
    - 이미 있는 name이라면 값을 변경
    
    ```java
    JSONObject object = new JSONObject(person);
    
    object.put("name", "academy");
            
    System.out.println(object + " | " + person.getName());
    
    {"name":"academy"} | nhn
    ```
    
    - 객체의 값은 변하지 않고 json 값만 변하였다.
- 값을 가져올 때에는 `get` 을 활용한다.
    
    ```java
    JSONObject customer = new JSONObject();
            customer.put("name", "nhn");
            customer.put("age", 20);
            System.out.println(customer);
    
            Object name = customer.get("name");
            System.out.println(name.getClass().getTypeName());
            System.out.println(name);
    
            Object age = customer.getInt("age");
            System.out.println(age.getClass().getTypeName());
            System.out.println(age);
    ```
    
    - `getInt` 를 활용하여 받아올 data type을 지정 가능하다.
- `option` 을 활용해서 초기 타입을 지정 가능
    
    ```java
    try {
              String jsonString = "{\"model\":\"K2\",\"power\":10}";
              JSONObject jsonObject = new JSONObject(jsonString);
              System.out.println(jsonString.toString());
              System.out.println(jsonObject.optBoolean("flyable", false));
          } catch (JSONException e) {
              System.err.println(e);
          }
    ```
    
    - `optBoolean` 을 참조

### Json Array

- Json array를 활용해 국내 주요 도시를 나열
    
    ```java
    public static void main(String args[]) {
    
            String[] cities = new String[] { "서울", "부산", "광주", "대구", "대전", "인천", "울산" };
    
            JSONArray jsonArray = new JSONArray();
            for (String city : cities) {
                jsonArray.put(city);
            }
    
            JSONObject object = new JSONObject();
            object.put("도시", jsonArray);
    
            System.out.println(object.toString());
        }
    ```
    

**Q1. JSON으로 동물 분류 및 나열**

- 동물
    - 조류
        - 갈매기
        - 참새
        - 펭귄
    - 포유류
        - 사자
        - 호랑이
        - 말
- JSON 문자열을 사용하지 말고, JSONObject default constructor로 object를 생성 후 put method를 이용해 추가하라.

---

```java
public static void main(String args[]) {

        String[] Birds = new String[] {"갈매기", "참새","팽귄"};
        String[] mammalias =  new String[] {"사자", "호랑이", "말"};

        JSONArray birdsArray = new JSONArray();
        JSONArray mammaliasArray = new JSONArray();

        for (String Bird : Birds) {
            birdsArray.put(Bird);
        }

        for (String mamalia : mammalias) {
            mammaliasArray.put(mamalia);
        }

        JSONObject jsonmammalias = new JSONObject();
        JSONObject jsonbirds = new JSONObject();

        jsonbirds.put("조류", birdsArray );
        jsonmammalias.put("포유류", mammaliasArray);

        JSONArray animalsArray = new JSONArray();
        animalsArray.put(jsonbirds);
        animalsArray.put(jsonmammalias);

        JSONObject object = new JSONObject();

        object.put("동물", animalsArray);

        System.out.println(object.toString());
    }

{"동물":[{"조류":["갈매기","참새","팽귄"]},{"포유류":["사자","호랑이","말"]}]}
```

**Q2. 숫자 1의 인덱스를 출력 {”true”, “1”, “true}**

```java
public static void main(String[] args) {
        try {
            JSONArray jsonArray = new JSONArray();

            jsonArray.put("true");
            jsonArray.put(1);
            jsonArray.put(true);
            int index = 0;
            for (Object entity : jsonArray) {
                if (entity instanceof Integer && (int) entity == 1) {
                    break;
                } index++;
            }

            if (index < 0) {
                System.out.println("1을 찾을 수 없습니다.");
            } else {
                System.out.println("1은 " + (index + 1) + "번째에 위치합니다.");
            }
        } catch (JSONException e) {
            System.err.println(e);
        }
    }

1은 2번째에 위치합니다.
```

**Q3. JSON을 이용해 사용자/아이템/전적 관리를 위한 파일 기반의 데이터베이스를 만들어라.**

*관리 정보*

- 사용자
    - 아이디
    - 닉네임
- 아이템
    - 아이디
    - 모델
    - 체력(0 ~ 10000)
    - 공격력(0 ~ 100)
    - 방어력(0 ~ 50)
    - 이동속도(0 ~ 100)
    - 공격속도(0 ~ 100)
- 전적
    - 대전 횟수
    - 승리 횟수
- 변경 이력

### 요구 사항

1. 구성 요소별로 class를 정의하라.
2. 생성된 데이터는 JSON 문자열로 파일에 저장한다.
3. 파일에서 JSON 문자열을 읽어 들여서 각 object를 생성한다.
4. 데이터를 저장할 때마다 변경 이력을 추가하라
    ◦ 변경 이력에는 시간과 변경 내용이 포함된다.
  ```java
public class Json extends Object{ 

    private static String user1 = "{\"userid\" : \"unggu\" , \"username\" : \"unggu \"}";
    private static String item1 =  "{\"itemname\" : \"ak\", \"model\" :  \"ak-47\", \"hp\" : 13 , \"pow\" : 26 , \"def\" : 23 , \"spd\" : 123 , \"dps\" : 11}";
    private static String record1 =  "{\"recordcount\" : 12 , \"wincount\" : 8}";

    private static String user2 = "{\"userid\" : \"adfs123\" , \"username\" : \"junggu \"}";
    private static String item2 =  "{\"itemname\" : \"m\", \"model\" : \"m16\", \"hp\" : 21 , \"pow\" : 23 , \"def\" : 53 , \"spd\" : 313 , \"dps\" :52}";
    private static String record2 =  "{\"recordcount\" : 21 , \"wincount\" : 6}";

    public static void main(String[] args) {
        try {
     
                JSONObject userObject = new JSONObject(user1);
                JSONObject itemObject = new JSONObject(item1);
                JSONObject recordObject = new JSONObject(record1);
                User user1 = new User(userObject.getString("userid"),userObject.getString("username"), LocalTime.now());
                Item item1 = new Item(itemObject.getString("itemname"), itemObject.getString("model"), itemObject.getInt("hp"), itemObject.getInt("pow"), itemObject.getInt("def"), itemObject.getInt("spd"), itemObject.getInt("dps"), LocalTime.now());
                Record record1 = new Record(recordObject.getInt("recordcount"), recordObject.getInt("wincount"), LocalTime.now());

                userObject = new JSONObject(user2);
                itemObject = new JSONObject(item2);
                recordObject = new JSONObject(record2);
                User user2 = new User(userObject.getString("userid"),userObject.getString("username"), LocalTime.now());
                Item item2 = new Item(itemObject.getString("itemname"), itemObject.getString("model"), itemObject.getInt("hp"), itemObject.getInt("pow"), itemObject.getInt("def"), itemObject.getInt("spd"), itemObject.getInt("dps"), LocalTime.now());
                Record record2 = new Record(recordObject.getInt("recordcount"), recordObject.getInt("wincount"), LocalTime.now());
            } catch (JSONException e) {
            System.err.println(e);
        }
    }
}
```

  ```java
    import java.sql.Time;
import java.time.LocalTime;

public class Record {

    private int battlecont;
    private int wincount;
    private LocalTime time;

    Record(int b, int w, LocalTime t) {
        this.battlecont = b;
        this.wincount = w;
        time = t;
    }
}
  ```


  ```java
    import java.time.LocalTime;

public class User {
    private String userid;
    private String username;
    private LocalTime time;

    private String olduserid;
    private String oldusername;
    private LocalTime oldtime;

    public User(String u, String u2, LocalTime t) {
        this.userid = u;
        this.username = u2;
        this.time = t;
    }

    @Override
    public String toString() {
        return userid + " " + username + " " + time;
    }
}
```

  ```java
      import java.time.LocalTime;

public class Item {
    private String itemname;
    private String model;
    private int hp;
    private int pow;
    private int def;
    private int spd;
    private int dps;
    private LocalTime time; 

    Item(String itemname, String model, int h, int p, int d, int s, int dps, LocalTime t) {
        itemname = itemname;
        model = model;
        hp = h;
        pow = p;
        def = d;
        spd = s;
        dps = dps;
        this.time = t;
    }
}
```

