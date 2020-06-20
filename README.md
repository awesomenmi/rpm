# Управление пакетами. Дистрибьюция софта 
## Выполненные шаги
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
## Схема работы стенда

1. Запуск стенда:
```
vagrant up
vagrant ssh
```
2. Вывод списка подключенных репозиториев:
```
yum repolist
```

```
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.reconn.ru
 * extras: mirror.reconn.ru
 * updates: mirror.reconn.ru
base                                                                     | 3.6 kB  00:00:00     
custom                                                                   | 2.9 kB  00:00:00     
extras                                                                   | 2.9 kB  00:00:00     
updates                                                                  | 2.9 kB  00:00:00     
(1/2): custom/primary_db                                                 | 2.8 kB  00:00:00     
(2/2): updates/7/x86_64/primary_db                                       | 2.1 MB  00:00:00     
repo id                                     repo name                                     status
base/7/x86_64                               CentOS-7 - Base                               10,070
custom                                      custom-repo                                        1
extras/7/x86_64                             CentOS-7 - Extras                                397
updates/7/x86_64                            CentOS-7 - Updates                               760
repolist: 11,228
```
3. Определим, к какому пакету относится nginx
```
yum provides nginx
```
```
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.reconn.ru
 * extras: mirror.reconn.ru
 * updates: mirror.reconn.ru
1:nginx-1.18.0-1.el7.ngx.x86_64 : High performance web server
Repo        : custom



1:nginx-1.18.0-1.el7.ngx.x86_64 : High performance web server
Repo        : installed




```
