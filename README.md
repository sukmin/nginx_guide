# nginx_guide

## nginx다운로드 주소
http://nginx.org/en/download.html

## 설치 line by line
wget http://nginx.org/download/nginx-1.9.4.tar.gz
tar -xvzf nginx-1.9.4.tar.gz
cd nginx-1.9.4
./configure --prefix=/home/계정명/설치경로/nginx-1.9.4 --with-http_ssl_module --with-http_realip_module --user=계정명 --group=그룹명 