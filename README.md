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

* 타입확인하기

> $ TYPE [key]

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

### List 타입

* 일반적은 프로그래밍 언어에서 데이터를 처리할 때 사용되는 배열(array) 변수와 유사한 데이터 구조
* String 타입의 경우 배열에 저장할 수 있는 데이터 크기는 512M
* List타입은 하나의 Key에 여러 개의 value를 저장할 수 있음.
* 명령어 : lpush, lrange, rpush, rpop, llen, lindex

```bash
127.0.0.1:5000> lpush order_detail:20231001 "<item_id>1</item_id><product_name>bunny boots</product_name><item_price>135</item_price><qty>500</qty><price>67000</price>" "<item_id>2</item_id><product_name>Pro Ski Boots</product_name><item_price>380</item_price><qty>400</qty><price>152000</price>"
2

127.0.0.1:5000> lrange order_detail:20231001 0 10 # lrange key start end
<item_id>2</item_id><product_name>Pro Ski Boots</product_name><item_price>380</item_price><qty>400</qty><price>152000</price>
<item_id>1</item_id><product_name>bunny boots</product_name><item_price>135</item_price><qty>500</qty><price>67000</price>

127.0.0.1:5000> rpush order_detail:20231001 "<item_id>3</item_id><product_name>Bunny Boots</product_name><item_price>135</item_price><qty>10</qty><price>13500</price>"
3

127.0.0.1:5000> lrange order_detail:20231001 0 3  # lrange key start end
<item_id>2</item_id><product_name>Pro Ski Boots</product_name><item_price>380</item_price><qty>400</qty><price>152000</price>
<item_id>1</item_id><product_name>bunny boots</product_name><item_price>135</item_price><qty>500</qty><price>67000</price>
<item_id>3</item_id><product_name>Bunny Boots</product_name><item_price>135</item_price><qty>10</qty><price>13500</price>

127.0.0.1:5000> rpop order_detail:20231001 1 # 가장 오른쪽의 value 값 제거(rpop key 갯수)
<item_id>3</item_id><product_name>Bunny Boots</product_name><item_price>135</item_price><qty>10</qty><price>13500</price>

127.0.0.1:5000> lrange order_detail:20231001 0 3  # 내용 확인
<item_id>2</item_id><product_name>Pro Ski Boots</product_name><item_price>380</item_price><qty>400</qty><price>152000</price>
<item_id>1</item_id><product_name>bunny boots</product_name><item_price>135</item_price><qty>500</qty><price>67000</price>

127.0.0.1:5000> llen order_detail:20231001 # 키에 저장된 갯수 확인
2

127.0.0.1:5000> lindex order_detail:20231001 0 # 해당 키의 0번째 인덱스 값 가져오기
<item_id>2</item_id><product_name>Pro Ski Boots</product_name><item_price>380</item_price><qty>400</qty><price>152000</price>

127.0.0.1:5000> lset order_detail:20231001 0 "<item_id>2</item_id><product_name>Pro Ski Boots</product_name><item_price>380</item_price><qty>200</qty><price>152000</price>" # 첫번째 인덱스 변경 (qty=400 → qty=200)
OK

127.0.0.1:5000> lindex order_detail:20231001 0 # 변경된 값 확인
<item_id>2</item_id><product_name>Pro Ski Boots</product_name><item_price>380</item_price><qty>200</qty><price>152000</price>

127.0.0.1:5000> lpushx order_detail:20231001 "<item_id>4</item_id><product_name>Bunny Boots</product_name><item_price>135</item_price><qty>500</qty><price>67000</price>" # 가장 왼쪽에 새로운 값을 저장
3

127.0.0.1:5000> lrange order_detail:20231001 0 10 # 저장된 값 확인
<item_id>4</item_id><product_name>Bunny Boots</product_name><item_price>135</item_price><qty>500</qty><price>67000</price>
<item_id>2</item_id><product_name>Pro Ski Boots</product_name><item_price>380</item_price><qty>200</qty><price>152000</price>
<item_id>1</item_id><product_name>bunny boots</product_name><item_price>135</item_price><qty>500</qty><price>67000</price>

127.0.0.1:5000> linsert order_detail:20231001 before "<item_id>2</item_id><product_name>Pro Ski Boots</product_name><item_price>380</item_price><qty>200</qty><price>152000</price>" "<item_id>3</item_id><product_name>Bunny Boots</product_name><item_price>135</item_price><qty>500</qty><price>13500</price>" # item2 앞에 item3 추가
4

127.0.0.1:5000> lrange order_detail:20231001 0 10  # item2앞에 item3 추가 확인
<item_id>4</item_id><product_name>Bunny Boots</product_name><item_price>135</item_price><qty>500</qty><price>67000</price>
<item_id>3</item_id><product_name>Bunny Boots</product_name><item_price>135</item_price><qty>500</qty><price>13500</price>
<item_id>2</item_id><product_name>Pro Ski Boots</product_name><item_price>380</item_price><qty>200</qty><price>152000</price>
<item_id>1</item_id><product_name>bunny boots</product_name><item_price>135</item_price><qty>500</qty><price>67000</price>

127.0.0.1:5000> lpush order_detail:20231002 "{item:id:1, product_name:Bunny Boots, item_price:135, qty:500 price:67000}" "{item_id:2, product_name:Pro Ski Boots, item_price:380, qty:400, price:152000}" # Data Format을 JSON 타입으로 저장
2

127.0.0.1:5000> lrange order_detail:20231002 0 1 # 저장된 값 확인
{item_id:2, product_name:Pro Ski Boots, item_price:380, qty:400, price:152000}
{item:id:1, product_name:Bunny Boots, item_price:135, qty:500 price:67000}
```

### Set 타입

* 배열구조가 아닌 여러 개의 Element로 데이터 값을 표현하는 구조
* 명령어 : sadd, smembers, scard, sdiff, sunion

```bash
127.0.0.1:5000> sadd product "id:11, product_name:Sky Pole, item_price:55, qty:100, price:5500" "id:22, product_name:Bunny Boots, item_price:135, qty:500, price:67000"
2

127.0.0.1:5000> sadd product "id:33, product_name:Pants, item_price:10, qty:200, price:2000"
1

127.0.0.1:5000> smembers product  # 해당 키의 전체 데이터 확인
id:22, product_name:Bunny Boots, item_price:135, qty:500, price:67000
id:33, product_name:Pants, item_price:10, qty:200, price:2000
id:11, product_name:Sky Pole, item_price:55, qty:100, price:5500

127.0.0.1:5000> scard product   # 해당 키의 전체 데이터 건수
3

127.0.0.1:5000> sadd product_old "id:91, product_name:Old Sky Pole"  # 새로운 키로 값 추가
1

127.0.0.1:5000> keys * # Key값 확인
product_old
product

127.0.0.1:5000> smembers product_old  # 추가된 키로 내용 확인
id:91, product_name:Old Sky Pole

127.0.0.1:5000> sdiff product product_old  # product와 product_old 중에 product에 만 있는 value 출력
id:22, product_name:Bunny Boots, item_price:135, qty:500, price:67000
id:11, product_name:Sky Pole, item_price:55, qty:100, price:5500
id:33, product_name:Pants, item_price:10, qty:200, price:2000

127.0.0.1:5000> sdiff product_old product  # product와 product_old 중에 product_old에 만 있는 value 출력
id:91, product_name:Old Sky Pole

127.0.0.1:5000> sdiffstore product_diff product product_old  # product와 product_old 중에 product에만 있는 value sets를 product_diff에 저장
3

127.0.0.1:5000> smembers product_diff
id:22, product_name:Bunny Boots, item_price:135, qty:500, price:67000
id:11, product_name:Sky Pole, item_price:55, qty:100, price:5500
id:33, product_name:Pants, item_price:10, qty:200, price:2000

127.0.0.1:5000> sunion product product_old  # product와 product_old를 union 한 결과
id:22, product_name:Bunny Boots, item_price:135, qty:500, price:67000
id:91, product_name:Old Sky Pole
id:11, product_name:Sky Pole, item_price:55, qty:100, price:5500
id:33, product_name:Pants, item_price:10, qty:200, price:2000

127.0.0.1:5000> sunionstore product_new product product_old # product와 product_old의 union한 value sets를 product_new에 저장
4

127.0.0.1:5000> smembers product_new # 저장된 product_new를 확인함.
id:22, product_name:Bunny Boots, item_price:135, qty:500, price:67000
id:91, product_name:Old Sky Pole
id:11, product_name:Sky Pole, item_price:55, qty:100, price:5500
id:33, product_name:Pants, item_price:10, qty:200, price:2000

127.0.0.1:5000> srem product_new "id:11, product_name:Sky Pole, item_price:55, qty:100, price:5500" # 지정해서 제거
1

127.0.0.1:5000> smembers product_new # 제거된 값 확인
id:22, product_name:Bunny Boots, item_price:135, qty:500, price:67000
id:91, product_name:Old Sky Pole
id:33, product_name:Pants, item_price:10, qty:200, price:2000

127.0.0.1:5000> spop product_new  # 랜덤 제거
id:33, product_name:Pants, item_price:10, qty:200, price:2000

127.0.0.1:5000> smembers product_new  # 제거된 값 확인
id:22, product_name:Bunny Boots, item_price:135, qty:500, price:67000
id:91, product_name:Old Sky Pole

127.0.0.1:5000> sadd product.id.index 1 13
2
127.0.0.1:5000> sadd product.id.index 2 12
2
127.0.0.1:5000> sadd product.id.index 3 11
2
127.0.0.1:5000> smembers product.id.index
1
2
3
11
12
13
```

### Sorted Set 타입

* Set 타입과 동일한 데이터 구조
* 저장된 값이 분류(Sorting)된 상태
* 명령어 : zadd, zrange, zcard, zcount, zrank zrevrank

```bash
127.0.0.1:5000> zadd order_detail:20231001 1 "{product_name:Bunny Boots, item_price:135, qty:500, price:67000}" 2 "{product_name:Pants, item_price:10, qty:200, price:2000}" 3 "{product_name:Sky Pole, item_price:55, qty:100, price:5500}" # 지정한 값 순서대로 저장됨
3

127.0.0.1:5000> zrange order_detail:20231001 0 -1 # 전체 내용 표시
{product_name:Bunny Boots, item_price:135, qty:500, price:67000}
{product_name:Pants, item_price:10, qty:200, price:2000}
{product_name:Sky Pole, item_price:55, qty:100, price:5500}

127.0.0.1:5000> zadd order_detail:20231001 3 "{product_name:Ski Pants, item_price:100, qty:10, price:1000}" # 4번째로 값을 추가하기
1

127.0.0.1:5000> zrange order_detail:20231001 0 -1 # 전체 데이터 확인
{product_name:Bunny Boots, item_price:135, qty:500, price:67000}
{product_name:Pants, item_price:10, qty:200, price:2000}
{product_name:Ski Pants, item_price:100, qty:10, price:1000}
{product_name:Sky Pole, item_price:55, qty:100, price:5500}

127.0.0.1:5000> zcard order_detail:20231001   # 해당 Key로 저장되어 있는 list value 갯수
4

127.0.0.1:5000> zcount order_detail:20231001 1 3
4

127.0.0.1:5000> zrem order_detail:20231001 "{product_name:Pants, item_price:10, qty:200, price:2000}" # 지정한 데이터 삭제
1

127.0.0.1:5000> zrange order_detail:20231001 0 -1 # 삭제된 데이터 확인
{product_name:Bunny Boots, item_price:135, qty:500, price:67000}
{product_name:Ski Pants, item_price:100, qty:10, price:1000}
{product_name:Sky Pole, item_price:55, qty:100, price:5500}

127.0.0.1:5000> zrank order_detail:20231001 "{product_name:Ski Pants, item_price:100, qty:10, price:1000}"
1
127.0.0.1:5000> zrank order_detail:20231001 "{product_name:Bunny Boots, item_price:135, qty:500, price:67000}"
0
127.0.0.1:5000> zrank order_detail:20231001 "{product_name:Sky Pole, item_price:55, qty:100, price:5500}"
2

127.0.0.1:5000> zrevrank order_detail:20231001 "{product_name:Sky Pole, item_price:55, qty:100, price:5500}" # 저장된 rank의 reverse rank
0
127.0.0.1:5000> zrevrank order_detail:20231001 "{product_name:Ski Pants, item_price:100, qty:10, price:1000}"
1
127.0.0.1:5000> zrevrank order_detail:20231001 "{product_name:Bunny Boots, item_price:135, qty:500, price:67000}"
2

127.0.0.1:5000> zscore order_detail:20231001 "{product_name:Sky Pole, item_price:55, qty:100, price:5500}" #데이터가 저장된 시점의 value의 Score(position) rank와 score는 다른 의미임.
3
127.0.0.1:5000> zscore order_detail:20231001 "{product_name:Ski Pants, item_price:100, qty:10, price:1000}"
3
127.0.0.1:5000> zscore order_detail:20231001 "{product_name:Bunny Boots, item_price:135, qty:500, price:67000}"
1

```

### Bit 타입

* 0과 1을 저장
* 명령어 : setbit, getbit, bitcount

### Geo 타입

* 지리정보를 저장
* geoadd, geopos, geodist, georadius, geohash

### HyperLogLogs 타입

* 관계형 DB의 테이블 구조에서 Check제약조건과 유사한 개념
* 해당 컬럼에 반드시 저장되어야 하는 값 만을 저장할 때 사용
* 필요에 따라 연결(Link)하여 사용할 수 있는 데이터 타입
* 명령어 : pfadd, pfcount, pfmerge

```bash
127.0.0.1:5000> pfadd domestic_city "Seoul" "Busan" "Daejeon" "Gwangju" "Daegu" "Ulsan"
1
127.0.0.1:5000> pfadd foreign_city "Los Angeles" "San Diego" "New York" "Washington"
1
127.0.0.1:5000> pfadd credit_type "cach" "credit card" "point" "check" "cash"
1

127.0.0.1:5000> pfcount credit_type
5
127.0.0.1:5000> pfcount domestic_city
6
127.0.0.1:5000> pfcount foreign_city
4

127.0.0.1:5000> pfmerge international_city domestic_city foreign_city
OK
127.0.0.1:5000> pfcount international_city
10
```
