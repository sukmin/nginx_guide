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
* (없음) : 지정된 패턴으로 시작하면 동작 정규표현식 사용불가.(단순문자열만 가능)
* ~ : 대소문자를 구분하며 정규표현식과 일치해야함.
* ~* : 대소문자를 구분하지 않으며 정규표현식과 일치해야함.
* ^~ : ??? 아직 잘 이해가 안됨 테스트해보고...
* @ : 클라이언트가 접근할 수 없고, 내부요청만 접근할 수 있는 location을 만듬.

### 설정 전체 예제
https://www.nginx.com/resources/wiki/start/topics/examples/full/

### 로그설정 및 정적자원 nginx로 돌리기
```
http {
    ...
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    ...
    server {
        ...
        access_log logs/서비스명/access_로그이름.log main; #서비스명에 해당하는 디렉토리는 nginx/logs디렉토리 밑에 미리 만들어주어야 함
        error_log logs/서비스명/error_로그이름.log;

        location ~* \.(js|jpg|png|css|html|woff2)$ {
            root /home/서비스하는_루트경로;
            expires 1d;
            access_log off;
        }
    }
}
```

### 로그로테이트 설정
```
#!/bin/sh

# File date format
DATE=`/bin/date +%y%m%d`

# Archive period
DAYS=30

NGINX_LOG_DIR=/home/사용자계정/로그위치

function delete_nginx_log {
        find $NGINX_LOG_DIR/상세디렉토리 -mtime +$DAYS -name "access*" -exec rm {} \;
        find $NGINX_LOG_DIR/상세디렉토리 -mtime +$DAYS -name "error*" -exec rm {} \;
}

function rotate_nginx_log() {
        DATE=`/bin/date +%Y%m%d --date '1days ago'`

        mv $NGINX_LOG_DIR/상세디렉토리/access_액세스로그파일명 $NGINX_LOG_DIR/상세디렉토리/access_액세스로그파일명_$DATE
        mv $NGINX_LOG_DIR/상세디렉토리/error_에러로그파일명 $NGINX_LOG_DIR/상세디렉토리/error_에러로그파일명_$DATE

        kill -USR1 `cat $NGINX_LOG_DIR/nginx.pid`
}

delete_nginx_log
rotate_nginx_log
```
쉘스크립트이므로 권한을 주어야 한다.
```
chmod 755 rotate.sh
```
크론에 등록하여 매일 정각에 동작하도록 한다.
```
59 23 * * * /home/사용자계정/스크립트파일경로 > /dev/null 2>&1
```

### TOMCAT 연결
```
server {
    ...
    location / {
            proxy_pass http://127.0.0.1:8080; #로컬에 톰캣

            proxy_redirect off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr; #톰캣에 사용자IP 전달을 위해 추가
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; #톰캣에 사용자IP 전달을 위해 추가

            proxy_intercept_errors on; #톰캣이 죽었을 때 에러페이지를 엔진엑스로 보여주기 위해 설정
            error_page   500 502 503 504 404 @error_page; #톰캣이 죽으면 일반적으로 502에러 발생
        }

        location @error_page{
            root /home/톰캣이_죽었을경우_엔진엑스가_바라볼_경로;
            internal;
            rewrite ^ /error.html;
            break;
        }
}
```
