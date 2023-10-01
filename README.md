# redis

* redis파일 다운 로드
  [redis 파일 다운로드](https://download.redis.io/redis-stable.tar.gz)
```bash
$ sudo useradd redis -m 

$ passwd redis #패스워드 입력함 

$ su redis 
암호 : redis123

$ wget https://download.redis.io/redis-stable.tar.gz 

$ tar xvfz redis-4.0.2.tar.gz 

$ mv redis-stable/* /home/redis 

$ make 

$ cp redis.conf redis_5000.conf 

$ vi redis_5000.conf 
  ##############수정 내용################# 
  daemonize no 
  port 5000 
  logflie "/home/redis/redis_5000.log" 

##########서버 실행############### 
$ src/redis-server /home/redis/redis_5000.conf 

##########클라이언트 실행########## 
$ src/redis-cli -p 5000
$ src/redis-cli -p 5000 --raw   # 한글 처리를 위해 필요 (--raw)
```


* 서버를 실행했을 때 워닝 발생함(log파일 참조)
  * 메모리 부족으로 문제가 발생할 수 있으니 메모리 확장이 가능한 환경으로 변경하라는 요청이 있음
> sudo sysctl vm.overcommit_memory=1

---

## 용어 설명
1. Table : 하나의 DB에서 데이터를 저장하는 논리적 구조 
2. Data Sets : 테이블을 구성하는 논리적 단위. \
하나의 데이타 넷은 하나의 key와 한 개 이상의 Field/Element로 구성
3. Key : 하나 이상의 조합된 값으로 표현 가능
4. Values : 해당 Key에 대한 구체적인 데이터 값을 표현

## 데이터 입력 / 수정 / 삭제 / 조회
|종류|내용|
|-|-|
| set | 데이터를 저장할 때 (key, value)|
| get | 저장된 데이터를 검색할 때 |
| rename | 저장된 데이터 값을 변경할 때 |
| randomkey | 저장된 key 중에 하나의 key를 랜덤하게 검색할 때 |
| keys | 저장된 모든 key를 검색할 때 |
| exists | 검색 대상 key가 존재하는지 여부를 확인할 때 |
| mset / mget | 여러 개의 key와 value를 한 번에 저장하고 검색할 때 |
| strlen | 저장된 데이터의 문자열 길이 |
| flushall | 현재 저장되어 있는 모든 key 삭제 |
| setex | ```ex```는 expire(일정 시간 동안만 저장)<br>ex) setex 1111 30 "JM JOO"<br># 30초 후에 삭제 |
| ttl | 현재 남은 시간 확인<br> ex) ttl 1111
| 연속번호 발행 | mset seq_no 202310010001<br>```incr``` seq_no<br>```decr``` seq_no<br>```incrby``` deq_no 2<br>```decrby``` seq_no 10 |
| append | 현재 value에 다른 값을 추가 |
| save | 현재 입력한 key/value 값을 파일로 저장 (dump.rdb)


## 데이터 타입
|종류|내용|
|-|-|
|strings|문자(text), Binary 유형 데이터를 저장|
|List|하나의 Key에 여러 개의 배열 값을 저장|
|Hash|하나의 Key에 여러 개의 Fields와 Value로 구성된 테이블을 저장|
|Set<br>Sorted set|정렬되지 않은 String 타입<br>Set과 Hash를 결합한 타입|
|Bitmaps|0 & 1로 표현하는 데이터 타입|
|HyperLogLogs|Element 중에서 Unique 한 개수의 Element 만 계산|
|Geospatial|좌표 데이터를 저장 및 관리하는 데이터 타입|

### Hash 타입
* 기존 관계형 DB에서 Primary Key와 하나 이상의 컬럼으로 구성된 테이블 구조와 유사함.
* Key는 [오브젝트명]:[하나 이상의 필드값]으로 표현
* 필드 갯수 제한 없음
* 명령어 : hmset, hget, hgetall, hkey, hlen, hexists, hkeys, hvals

```bash
127.0.0.1:5000> hmset order:20231001 customer_name "Wonman & Sports" emp_name "Magee" total 601100 payment_type "Credit" order_filled "Y" ship_date 20180925

127.0.0.1:5000> hget order:20231001 customer_name

127.0.0.1:5000> hget order:20231001 ship_date

127.0.0.1:5000> hgetall order:20231001
 1) "customer_name"   # Field1
 2) "Wonman & Sports" # Value1
 3) "emp_name"        # Field2
 4) "Magee"           # Value2
 5) "total"
 6) "601100"
 7) "payment_type"
 8) "Credit"
 9) "order_filled"
10) "Y"
11) "ship_date"
12) "20180925"

127.0.0.1:5000> hdel order:20231001 ship_date   # 특정 필드 제거

127.0.0.1:5000> hgetall order:20231001
 1) "customer_name"
 2) "Wonman & Sports"
 3) "emp_name"
 4) "Magee"
 5) "total"
 6) "601100"
 7) "payment_type"
 8) "Credit"
 9) "order_filled"
10) "Y"

127.0.0.1:5000> hmset order:20231001 ship_date 20180925   # 필드 다시 추가
OK

127.0.0.1:5000> hgetall order:20231001    # 추가된 필드 확인
 1) "customer_name"
 2) "Wonman & Sports"
 3) "emp_name"
 4) "Magee"
 5) "total"
 6) "601100"
 7) "payment_type"
 8) "Credit"
 9) "order_filled"
10) "Y"
11) "ship_date"     # 추가된 필드
12) "20180925"

127.0.0.1:5000> hkeys order:20231001
customer_name
emp_name
total
payment_type
order_filled
ship_date

127.0.0.1:5000> hvals order:20231001
Wonman & Sports
한글입력잘되나?
601100
Credit
Y
20180925

127.0.0.1:5000> hlen order:20231001     # 해당 키에 대한 모든 필드 갯수
6
```
