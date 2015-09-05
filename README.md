# nginx_guide

## nginx다운로드 주소
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
chmod u+s nginx

#사용자 계정으로 접근
cd /home/계정명/설치경로
ln -s /home/계정명/설치경로/nginx-1.9.4 nginx
```

## 옵션에서 추가한 모듈
* --with-http_ssl_module : HTTPS를 사용해 페이지를 서비스하는 SSL 모듈을 포함
* --with-http_realip_module : 요청 헤더 데이터로부터 실제 IP 주소를 읽어내는 Real IP 모듈을 포함
* 이외에도 매우 많은 옵션들이 있는데 http://www.yes24.com/24/goods/5721174 이 책에도 나와있고 웹에도 자료가 많습니다.

## 설치시 chmod u+s nginx 한것의 의미
리눅스에 내정된 포트 80 등등은 루트로만 띄을수 있다. 루트외에 사용자계정으로 띄우고 싶은 경우 u+s 퍼미션을 주어야 한다.
