# redis

* redis파일 다운 로드
  [redis 파일 다운로드](https://download.redis.io/redis-stable.tar.gz)

> $ sudo useradd redis -m \
> $ passwd redis #패스워드 입력함 \
> $ su redis \
> 암호 : redis123 \
> $ wget https://download.redis.io/redis-stable.tar.gz \
> $ tar xvfz redis-4.0.2.tar.gz \
> $ mv redis-stable/* /home/redis \
> $ make \
> $ cp redis.conf redis_5000.conf \
> $ vi redis_5000.conf \
>   ##############수정 내용################# \
>   daemonize no \
>   port 5000 \
>   logflie "/home/redis/redis_5000.log" \
> ##########서버 실행############### \
> $ src/redis-server /home/redis/redis_5000.conf \
> ##########클라이언트 실행########## \
> $ src/redis-cli -p 5000

* 서버를 실행했을 때 워닝 발생함(log파일 참조)
  * 메모리 부족으로 문제가 발생할 수 있으니 메모리 확장이 가능한 환경으로 변경하라는 요청이 있음
> sudo sysctl vm.overcommit_memory=1

