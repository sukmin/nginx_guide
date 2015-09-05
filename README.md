# nginx_guide

## nginx 다운로드 주소
http://nginx.org/en/download.html

## 설치 line by line

```
#ROOT 계정에서 수행
wget http://nginx.org/download/nginx-1.9.4.tar.gz
tar -xvzf nginx-1.9.4.tar.gz
cd nginx-1.9.4
./configure --prefix=/home/계정명/설치경로/nginx-1.9.4 --with-http_ssl_module --with-http_realip_module --user=계정명 --group=그룹명
make
make install
cd /home/계정명/설치경로
chown -R 계정명.그룹명 nginx-1.9.4
cd nginx-1.9.4/sbin/
chown root.그룹명 nginx
chmod u+s nginx

#사용자 계정으로 접근
cd /home/계정명/설치경로
ln -s /home/계정명/설치경로/nginx-1.9.4 nginx
```

## 옵션에서 추가한 모듈
* --with-http_ssl_module : HTTPS를 사용해 페이지를 서비스하는 SSL 모듈을 포함
* --with-http_realip_module : 요청 헤더 데이터로부터 실제 IP 주소를 읽어내는 Real IP 모듈을 포함
* 이외에도 매우 많은 옵션들이 있는데 http://www.yes24.com/24/goods/5721174 이 책에도 나와있고 웹에도 자료가 많습니다.

## 설치시 chown root.그룹명 nginx , chmod u+s nginx 한것의 의미
리눅스에 내정된 포트 80 등등은 루트로만 띄을수 있다. 루트외에 사용자계정으로 띄우고 싶은 경우 u+s 퍼미션을 주어야 한다.

## 엔진엑스 기본 명령
```
cd /home/계정명/설치경로/nginx/sbin
# 여기서 명령을 실행
```
* ./nginx : 실행
* ./nginx -s stop : 데몬을 즉각적으로 종료
* ./nginx -s quit : 데몬을 정상적으로 종료
* ./nginx -s reopen : 로그파일을 재오픈
* ./nginx -s reload : 환경설정파일을 다시 로드(아파치의 그레이스풀과 동일한듯?)
* ./nginx -V : 버전 및 설치옵션 확인

## 설정이 제대로 되어있는지 확인
```
cd /home/계정명/설치경로/nginx/sbin

./nginx -t

#다른경로에 있는 설정파일의 유효성 확인
nginx -t -c 다른경로
```

## 설정파일의 위치
```
/home/계정명/설치경로/nginx/conf/nginx.conf
```

## Hello Nginx 띄우기
아래와 같이 설정한다.
```
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;

    server {
        listen       80;
        server_name  100.10.10.10; #서버IP 또는 DNS설정 하셨다면 도메인을 쓰세요.

        location / {
            root   /home/계정명/html파일이있는주소;
            index  index.html index.htm;
        }

        error_page   500 502 503 504 404 /error.html;
        
    }
    
}
```

아래의 두 html 파일을 적절한 위치에 넣는다.
```
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Hello Nginx</title>
</head>
<body>
Hello Nginx!!!
</body>
</html>
```
```
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Error Nginx</title>
</head>
<body>
This is Error!
</body>
</html>
```
## 로케이션 변경자(location modifier)
```
location [=|~|~*|^~|@] pattern { ... }
```
* = : 패턴과 정확히 일치해야 동작. 정규표현식 사용불가.(단순문자열만 가능)
(없음) : ㅋㅋ
~ : ㅎㅎ
~* : ㅎㅎ
^~ : ㅆ
@ : ㅠㅠ
