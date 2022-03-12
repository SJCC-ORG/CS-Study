**TABLE ex)**

![image1](/데이터베이스/Join/image/join1.png)

### LEFT OUTER JOIN

왼쪽에 있는 table과 그 바깥에 있는 테이블을 연결하여 보여줌, 테이블에 둘 중 하나의 데이터만 있어도 연결 가능.

![image2](/데이터베이스/Join/image/join2.png)

ex) SELECT * FROM topic LEFT JOIN author ON topic.author_id = author.aid;

![image3](/데이터베이스/Join/image/join3.png)

두개 뿐만 아니라 세개의 table도 연결 가능

ex) SELECT * FROM topic LEFT JOIN author ON topic.author_id = author.aid LEFT JOIN profile ON author.profile_id = profile.pid;

![image4](/데이터베이스/Join/image/join4.png)

필요한 정보만 불러올 수도 있고 별칭을 붙일 수도 있다.

ex) SELECT tid, topic.title, author_id, name, profile.title AS job_title FROM topic LEFT JOIN author ON topic.author_id = author.aid LEFT JOIN profile ON author.profile_id = profile.pid;

![image5](/데이터베이스/Join/image/join5.png)

ex) SELECT tid, topic.title, author_id, name, profile.title AS job_title FROM topic LEFT JOIN author ON topic.author_id = author.aid LEFT JOIN profile ON author.profile_id = profile.pid WHERE aid = 1;

![image6](/데이터베이스/Join/image/join6.png)

## RIGHT OUTER JOIN

오른쪽에 있는 table과 그 바깥에 있는 테이블을 연결하여 보여줌, 테이블에 둘 중 하나의 데이터만 있어도 연결 가능.

![image7](/데이터베이스/Join/image/join7.png)

나머지는 LEFT OUTER JOIN과 같고 방향만 다름.

## INNER JOIN

왼쪽에 있는 테이블과 오른쪽에 있는 테이블에 데이터가 모두 있어야만 연결한다. 즉 교집합일때만 JOIN 해줌.

![image8](/데이터베이스/Join/image/join8.png)

ex) SELECT * FROM topic INNER JOIN author ON topic.author_id = author.aid;

![image9](/데이터베이스/Join/image/join9.png)

세 개도 연결 가능

ex) SELECT * FROM topic INNER JOIN author ON topic.author_id = author.aid INNER JOIN profile ON profile.pid = author.profile_id;

![image10](/데이터베이스/Join/image/join10.png)

## FULL OUTER JOIN

양 쪽 데이터 모두 연결해줌. (잘 사용하지는 않음)

![image11](/데이터베이스/Join/image/join11.png)

ex) SELECT * FROM topic FULL OUTER JOIN author ON topic.author_id = author.aid;

→ (SELECT * FROM topic LEFT JOIN author ON topic.author_id = author.aid) UNION (SELECT * FROM topic RIGHT JOIN author ON topic.author_id = author.aid); 

![image12](/데이터베이스/Join/image/join12.png)

## EXCLUSIVE LEFT JOIN

교집합을 제외한 왼쪽 데이터만을 나타낼때 사용함.

![image13](/데이터베이스/Join/image/join13.png)

ex) SELECT * FROM topic LEFT JOIN author ON topic.author_id = author.aid WHERE author.aid is NULL;

![image14](/데이터베이스/Join/image/join14.png)

맨 밑 Oracle만 나오게 됨

## EXCLUSIVE RIGHT JOIN

교집합을 제외한 오른쪽 데이터만을 나타낼때 사용함.

![image15](/데이터베이스/Join/image/join15.png)

blackdew만 나오게 됨