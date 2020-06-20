# Управление пакетами. Дистрибьюция софта 
```
vagrant up
vagrant ssh
yum repolist
yum provides nginx
```
____
1. Установим следующие утилиты: 
``` 
yum install -y redhat-lsb-core wget rpmdevtools rpm-build createrepo yum-utils gcc
```
2. Скачаем rpm-пакет nginx и установим его:
```
wget https://nginx.org/packages/centos/7/SRPMS/nginx-1.18.0-1.el7.ngx.src.rpm
rpm -i nginx-1.18.0-1.el7.ngx.src.rpm
```
3. Установим необходимые зависимости для сборки rpm-пакета:
```
yum-builddep /root/rpmbuild/SPECS/nginx.spec -y
```
4. Скачаем и распакуем архив с исходниками OpenSSL:
```
wget https://www.openssl.org/source/latest.tar.gz
tar -xvf latest.tar.gz --directory /usr/lib
```
5. Добавим в spec-файл nginx-a опцию, указывающую расположение библиотек OpenSSL
```
sed -i 's|--with-debug|--with-openssl=/usr/lib/openssl-1.1.1g|' /root/rpmbuild/SPECS/nginx.spec
```
6. Соберем rpm-пакет с обновленным spec-ом (-bb - сборка бинарного пакета):
```
rpmbuild --bb /root/rpmbuild/SPECS/nginx.spec
```
7. Установим nginx из только что собранного rpm-пакета:
```
yum localinstall -y /root/rpmbuild/RPMS/x86_64/nginx-1.18.0-1.el7.ngx.x86_64.rpm 
sed -i '/index  index.html index.htm;/a autoindex on;' /etc/nginx/conf.d/default.conf
systemctl enable --now nginx
```
8. Создадим rpm-репозиторий:
```
mkdir /usr/share/nginx/html/repo
cp /root/rpmbuild/RPMS/x86_64/nginx-1.18.0-1.el7.ngx.x86_64.rpm /usr/share/nginx/html/repo/
createrepo /usr/share/nginx/html/repo/
```
9. Добавим ранее созданный локальный репозиторий в список доступных:
```
cat >> /etc/yum.repos.d/custom.repo << EOF
[custom]
name=custom-repo
baseurl=http://localhost/repo
gpgcheck=0
enabled=1
EOF
```
