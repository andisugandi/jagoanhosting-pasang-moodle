# Cara Konvensional v.s Kontainer dalam Memasang Moodle

DSDP 3 - Cara Instalasi Moodle di VPS/Server Melalui Cara Manual

Berkas presentasinya dapat diperoleh di: [jagoanhosting-pasang-moodle.pdf](https://github.com/andisugandi/jagoanhosting-pasang-moodle/blob/master/jagoanhosting-pasang-moodle.pdf).

## Manakah yang Terbaik?

1. Memasang Moodle Cara Konvensional, atau
2. Memasang Moodle Berbasis Kontainer?

## Memasang Moodle Cara Konvensional

[![jagoanhosting-pasang-moodle](https://img.youtube.com/vi/G2qoDaS4g0g/0.jpg)](https://youtu.be/G2qoDaS4g0g)

**1. Sediakan Server (VPS).**

- Pasang VPS sesuai keinginan.
- Dalam tutorial ini, penulis menggunakan Linux CentOS 7.
- Sebagai awal, kita pastikan *firewall* sudah dipasang dan diatur dengan tepat:
  ~~~bash
  $ sudo yum install firewalld
  $ sudo systemctl enable --now firewalld
  $ sudo firewall-cmd --permanent --add-interface=eth0 --zone=public
  $ sudo firewall-cmd --reload
  $ sudo firewall-cmd --list-all --zone=public
  ~~~

**2. Pasang Paket.**

- **Web Server**

  ~~~bash
  $ sudo yum update
  $ sudo yum install httpd httpd-tools
  $ sudo systemctl enable --now httpd
  $ sudo systemctl status httpd
  ~~~

  ~~~bash
  $ sudo firewall-cmd --permanent --add-service={http,https} --zone=public
  $ sudo firewall-cmd --reload
  $ sudo firewall-cmd --list-all --zone=public
  ~~~
- **Modul PHP**

  ~~~bash
  $ sudo rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
  $ sudo rpm -Uvh http://rpms.remirepo.net/enterprise/remi-release-7.rpm
  ~~~

  ~~~bash
  $ sudo yum-config-manager --enable remi-php74
  $ sudo yum install php php-fpm php-cli php-pspell php-curl php-gd php-intl php-mysqlnd php-xml php-xmlrpc php-ldap php-zip php-json php-soap php-opcache php-readline php-mbstring php-sodium
  ~~~

  ~~~bash
  $ sudo echo '<?php phpinfo(); ?>' > /var/www/html/info.php
  ~~~

  ~~~bash
  $ sudo systemctl restart httpd
  ~~~
- **Basis Data**

  ~~~bash
  $ sudo vim /etc/yum.repos.d/MariaDB.repo
  ~~~

  Isinya:

  ~~~bash
  [mariadb]
  name = MariaDB
  baseurl = http://yum.mariadb.org/10.5/centos7-amd64
  gpgkey = https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
  gpgcheck=1
  ~~~

  ~~~bash
  $ sudo yum install MariaDB-server MariaDB-client
  ~~~

**3. Konfigurasi.**

- **Web Server**

  ~~~bash
  $ sudo vim /etc/httpd/conf.d/web1.namasekolah.com.conf
  ~~~

  isinya:

  ~~~bash
  <VirtualHost *:80>
   ServerName web1.namasekolah.com
   DocumentRoot /var/www/html/moodle
   DirectoryIndex index.html index.htm index.php index.php4 index.php5
   <Directory /var/www/html/moodle>
       Options -Indexes +IncludesNOEXEC +SymLinksIfOwnerMatch
       Allow from all
       AllowOverride All   
   </Directory>
   ErrorLog /var/log/httpd/web1.namasekolah.com_error_log
   CustomLog /var/log/httpd/web1.namasekolah.com_access_log combined
  </VirtualHost>
  ~~~

  ~~~bash
  $ sudo systemctl reload httpd
  ~~~
- **Modul PHP**

  ~~~bash
  $ sudo vim /etc/php.ini
  ~~~

  Ubah nilai baris `max_input_vars` sehingga berisi nilai: `10000`.

  ~~~bash
  max_input_vars = 10000
  ~~~

  ~~~bash
  $ sudo systemctl restart httpd
  ~~~
- **Basis Data**

  ~~~bash
  $ sudo vim /etc/my.cnf.d/server.cnf
  ~~~

  Sesuaikan isi `[mariadb]` menjadi seperti berikut::

  ~~~bash
  [mariadb]
  name = MariaDB
  baseurl = http://yum.mariadb.org/10.5/centos7-amd64
  gpgkey = https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
  gpgcheck=1
  ~~~

  ~~~bash
  $ sudo systemctl enable --now mariadb
  ~~~

  ~~~bash
  $ sudo mysql_secure_installation
  ~~~

  ~~~bash
  $ mysql -u root -p
  ~~~

  ~~~mysql
  > CREATE DATABASE moodledb DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
  > CREATE USER 'moodleuser'@'localhost' IDENTIFIED BY 't4ny4adm1n';
  > GRANT ALL ON moodledb.* TO 'moodleuser'@'localhost';
  ~~~

**4. Unduh Kode Moodle.**

Duplikasi kode Moodle menggunakan Git:

~~~bash
$ sudo yum install git
$ cd /var/www/html
$ sudo git clone https://github.com/moodle/moodle.git
~~~

Memilih Versi Moodle:

~~~bash
$ cd moodle
$ sudo git branch --track MOODLE_311_STABLE origin/MOODLE_311_STABLE
$ sudo git checkout MOODLE_311_STABLE
$ sudo mkdir /var/moodledata
~~~

Pastikan kepemilikian Kode Web Moodle oleh akun web werver (`apache`):

~~~bash
$ sudo chown apache:apache -R /var/www/html/moodle /var/moodledata
$ sudo chmod 777 /var/moodledata
~~~

Untuk distro Linux turunan Red Hat, terutama yang mengaktifkan fitur `SELinux`, hak Akses baca-tulis ke direktori Moodle oleh web server harus didefinisikan dengan tepat:

~~~bash
$ sudo semanage fcontext -at httpd_sys_rw_content_t '/var/www/moodle(/.*)?'
$ sudo restorecon -Rv '/var/www/moodle/'
$ sudo semanage fcontext -at httpd_sys_rw_content_t '/var/moodledata(/.*)?'
$ sudo restorecon -Rv '/var/moodledata/'
~~~

**5. Pasang Sertifikat SSL/TLS.**

~~~bash
$ sudo yum install certbot mod_ssl python2-certbot-apache
$ sudo certbot --apache
~~~

6. Pasang (*Deploy*) Moodle di VPS *via* Web-Installer.
7. Optimasi (jika perlu).

---

Jika memang pilihan untuk memasang (*deploy*) Moodle adalah dengan menggunakan fasilitas Kontainer (Docker), maka layanan web server dan basis datanya harus dimatikan terlebih dahulu dari VPS yang pernah digunakan sebelumnya, lalu paket keduanya dihapus.

Pada layanan web berbasis kontainer, *listen* port nomor: `80` dan `443` akan dilakukan oleh *Load Balancer* dari Traefik.

---

## Memasang Moodle Berbasis Kontainer

**1. Sediakan Server (VPS).**

- Pasang VPS sesuai keinginan.
- Pastikan tidak ada layanan web dan basis data yang sedang berjalan:

  ~~~bash
  $ sudo systemctl stop httpd
  $ sudo systemctl stop mariadb
  ~~~
- Jangan lupa, juga pastikan tidak ada paket web server (misal: `httpd`) dan basis data yang terpasang:

  ~~~bash
  $ sudo yum remove httpd MariaDB-server MariaDB-client
  ~~~
- Selanjutnya (jika belum), pasang dan atur *firewall* dengan tepat:

  ~~~bash
  $ sudo yum install firewalld
  $ sudo systemctl enable --now firewalld
  $ sudo firewall-cmd --permanent --add-interface=eth0 --zone=public
  $ sudo firewall-cmd --reload
  $ sudo firewall-cmd --list-all --zone=public
  ~~~

**2. Pasang Mesin Kontainer (Docker) + Docker Compose.**

- Pastikan paket Docker versi bawaan dihapus terlebih dahulu:

  ~~~bash
  $ sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
  ~~~

- Tambahkan repositori resmi Docker, lalu pasang paketnya:

  ~~~bash
  $ sudo yum install yum-utils
  $ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
  $ sudo yum install docker-ce docker-ce-cli containerd.io
  ~~~

- Jalankan layanan Docker, lalu tambahkan akun yang kita gunakan ke group `docker`:

  ~~~bash
  $ sudo systemctl enable --now docker
  $ sudo usermod -aG docker $USER
  ~~~

Lakukan *logout*, lalu *login* kembali.

- Selanjutnya pasang Docker Compose:

  ~~~bash
  $ sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  $ sudo chmod +x /usr/local/bin/docker-compose
  $ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
  $ sudo systemctl restart docker
  ~~~

- Akhirnya periksa apakah binari Docker dan Docker Compose sudah bisa dijalankan:

  ~~~bash
  $ docker --version
  $ docker-compose --version
  ~~~

**3. Konfigurasi Load Balancer & Sertifikat SSL/TLS via Traefik.**

- Persiapkan susunan berkas dan direktori yang diperlukan:

  ~~~bash
  $ mkdir ~/traefik && cd ~/traefik
  $ mkdir -p data/configuration
  $ touch docker-compose.yml
  $ touch data/acme.json
  $ touch data/configuration/dynamic.yml
  $ touch data/traefik.yml
  $ chmod 600 data/acme.json
  ~~~

  ~~~bash
  traefik/
  ├── data
  │   ├── acme.json
  │   ├── configurations
  │   │   └── dynamic.yml
  │   └── traefik.yml
  └── docker-compose.yml

  2 directories, 4 files
  ~~~

- Buatkan kata kunci untuk login akun `admin` Traefik agar dapat mengakses Dashboard Traefik:

  ~~~bash
  $ htpasswd -nb admin passwordUntukAkunAdminTrafik
  ~~~

  Hasilnya keluarannya misal seperti ini:

  ~~~bash
  admin:$apr1$karakteracakpunyaandamestinyaberbedadanunik
  ~~~

Simpan untuk nanti digunakan di berkas `data/configuration/dynamic.yml`.

- Isi berkas `docker-compose.yml`:

  ~~~yml
  version: '3.9'

  services:
    traefik:
      image: traefik:v2.4
      container_name: traefik
      restart: always
      security_opt:
        - no-new-privileges:true
      ports:
        - 80:80
        - 443:443
      volumes:
        - /etc/localtime:/etc/localtime:ro
        - /var/run/docker.sock:/var/run/docker.sock:ro
        - ./data/traefik.yml:/traefik.yml:ro
        - ./data/acme.json:/acme.json
        # Add folder with dynamic configuration yml
        - ./data/configurations:/configurations
      networks:
        - proxy
      labels:
        - "traefik.enable=true"
        - "traefik.docker.network=proxy"
        - "traefik.http.routers.traefik-secure.entrypoints=websecure"
        - "traefik.http.routers.traefik-secure.rule=Host(`web1.namasekolah.com`)"
        - "traefik.http.routers.traefik-secure.middlewares=user-auth@file"
        - "traefik.http.routers.traefik-secure.service=api@internal"

  networks:
    proxy:
      external: true
  ~~~

- Isi berkas `data/traefik.yml`:

  ~~~yml
  api:
  dashboard: true

  entryPoints:
    web:
      address: :80
      http:
        redirections:
          entryPoint:
            to: websecure

    websecure:
      address: :443
      http:
        middlewares:
          - secureHeaders@file
          - nofloc@file
        tls:
          certResolver: letsencrypt

  pilot:
    dashboard: false

  providers:
    docker:
      endpoint: "unix:///var/run/docker.sock"
      exposedByDefault: false
    file:
      filename: /configurations/dynamic.yml

  certificatesResolvers:
    letsencrypt:
      acme:
        email: andi@sibermu.ac.id
        storage: acme.json
        keyType: EC384
        httpChallenge:
          entryPoint: web

    buypass:
      acme:
        email: andi@sibermu.ac.id
        storage: acme.json
        caServer: https://api.buypass.com/acme/directory
        keyType: EC256
        httpChallenge:
          entryPoint: web
  ~~~

- Isi berkas `data/configurations/dynamic.yml`:

  ~~~yml
  # Dynamic configuration
  http:
    middlewares:
      nofloc:
        headers:
          customResponseHeaders:
            Permissions-Policy: "interest-cohort=()"
      secureHeaders:
        headers:
          sslRedirect: true
          forceSTSHeader: true
          stsIncludeSubdomains: true
          stsPreload: true
          stsSeconds: 31536000   

      # UserName : admin
      # Password : passwordUntukAkunAdminTrefik
      user-auth:
        basicAuth:
          users:
            - "admin:$apr1$ntilzY14$fDIINe9XxjPnVV6kt1V.1/"

  tls:
    options:
      default:
        cipherSuites:
          - TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
          - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
          - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
          - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
          - TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305
          - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
        minVersion: VersionTLS12
  ~~~

- Jangan lupa untuk membuatkan Docker Network: `proxy` terlebih dahulu:

  ~~~bash
  $ docker network create proxy
  ~~~

**4. Konfigurasi *Image* Moodle (Persistent Volume Moodle, moodledata, MariaDB).**
- Siapkan direktori Moodle & buat berkas `docker-compose.yml` di dalamnya:

  ~~~bash
  $ mkdir ~/moodle && cd ~/moodle
  $ vim docker-compose.yml
  ~~~

  Berikut isi berkas `docker-compose.yml`:

    ~~~yml
    version: '3.7'

    services:
      mariadb:
        image: 'docker.io/bitnami/mariadb:10.5-debian-10'
        container_name: mariadb
        environment:
          # Bagian ini HARUS DIGANTI dengan mengikuti panduan:
          # “Manage sensitive data with Docker secrets”
          # https://docs.docker.com/engine/swarm/secrets
          - MARIADB_ROOT_PASSWORD=jagoanhosting
          - MARIADB_USER=jagoanhostinguser
          - MARIADB_DATABASE=jagoanhostingdb
          - MARIADB_PASSWORD=jagoanhosting
          # ---Agar informasi kredensial dapat aman
          - MARIADB_CHARACTER_SET=utf8mb4
          - MARIADB_COLLATE=utf8mb4_unicode_ci
        networks:
          - moodlenetwork
        volumes:
          - 'moodle-mariadb:/bitnami/mariadb'
      moodle:
        image: 'docker.io/bitnami/moodle:3-debian-10'
        container_name: moodle
        environment:
          - MOODLE_DATABASE_HOST=mariadb
          - MOODLE_DATABASE_PORT_NUMBER=3306
          # Bagian ini HARUS DIGANTI dengan mengikuti panduan:
          # “Manage sensitive data with Docker secrets”
          # https://docs.docker.com/engine/swarm/secrets
          - MOODLE_DATABASE_USER=jagoanhostinguser
          - MOODLE_DATABASE_NAME=jagoanhostingdb
          - MOODLE_PASSWORD=jagoanhosting
          - MOODLE_EMAIL=andi@sibermu.ac.id
          - MOODLE_DATABASE_PASSWORD=jagoanhosting
          # ---Agar informasi kredensial dapat aman
          - PHP_MEMORY_LIMIT=1024M
          - PHP_POST_MAX_SIZE=1024M
          - PHP_UPLOAD_MAX_FILESIZE=1024M
          - PHP_MAX_EXECUTION_TIME=46800
        labels:
          - "traefik.enable=true"
          - "traefik.docker.network=proxy"
          - "traefik.http.routers.moodle-secure.entrypoints=websecure"
          - "traefik.http.routers.moodle-secure.rule=Host(`web2.namasekolah.com`)"
        volumes:
          - 'moodle_data:/bitnami/moodle'
          - 'moodledata_data:/bitnami/moodledata'
        networks:
          - moodlenetwork
          - proxy
        depends_on:
          - mariadb

    volumes:
      moodle-mariadb:
        driver: local
      moodle_data:
        driver: local
      moodledata_data:
        driver: local

    networks:
      moodlenetwork:
      proxy:
        external: true
    ~~~

**5. Jalankan Kontainer Traefik.**

  - Pastikan berada di dalam direktori `traefik`, lalu jalankan `docker-compose`:
    ~~~bash
    $ cd ~/traefik
    $ docker-compose up -d
    ~~~

**6. Jalankan Kontainer Moodle.**

- Pastikan berada di dalam direktori `moodle`, lalu jalankan `docker-compose`:

    ~~~bash
    $ cd ~/moodle
    $ docker-compose up -d
    ~~~

**7. Optimasi (jika perlu).**
