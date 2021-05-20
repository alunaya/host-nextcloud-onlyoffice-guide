# Hướng dẫn cài đặt thử nghiệm Nextcloud và OnlyOffice trên một Server

## Tạo các container
Đầu tiên ta cần cài đặt [Docker](https://docs.docker.com/engine/install/ubuntu/) và [Docker-compose](https://docs.docker.com/compose/install/)

Tạo các thư mục:
- /mnt/docker-mount/nextcloud-app
- /mnt/docker-mount/nextcloud-config
- /mnt/docker-mount/nextcloud-data

Clone git repo này về, sau đó trong thư mục repo này ta chạy
```sh
docker-compose up -d
```

Câu lệnh trên sẽ tạo 4 container:
![Docker containers](/screenshot/docker-ps.PNG)

## Kết nối onlyoffice và nextcloud

### Config Onlyoffice
Ta cần terminal vào trong docker container của OnlyOffice
```sh
docker exec -it <onlyoffice-container-id> /bin/bash
```

chỉnh sửa file */etc/onlyoffice/documentserver/local.json* như sau:
- Sửa các khóa *services.CoAuthoring.token.enable.browser*, *services.CoAuthoring.token.enable.request.inbox*, *services.CoAuthoring.token.enable.request.outbox* thành **true**
- Sửa các khóa *services.CoAuthoring.secret.inbox.string*, *services.CoAuthoring.secret.inbox.string*, *services.CoAuthoring.secret.inbox.string* thầnh **<secret-key-bạn-định-dùng>**

![onlyoffice config](/screenshot/onlyoffice-config.PNG)

Sau đó ta cần chạy câu lệnh dưới để lưu thay đổi
```sh
supervisorctl restart all
```

### Config Nextcloud
Trong file config.php tại thư mục **/mnt/docker-mount/nextcloud-config** ta thêm *1=>'nextcloud'* vào **trusted_domains** array

![nextcloud config](/screenshot/config-php.PNG)

Ta cần cài đặt Onlyoffice apps cho Nextcloud, ta có 2 các cài đặt
- Dùng app store của nextcloud (nếu có internet)
- Tải về và cài đặt ở local ([Xem hướng dẫn cài đặt](#Huong-dan-cai-dat-o-local))

Vào */settings/apps/disabled*, enable OnlyOffice app

Tiếp đó ta vào */settings/admin/onlyoffice* config các trường
- Document Editing Service address: public address của OnlyOffice Server (vd: http://192.168.56.3:8080/) 
- Document Editing Service address for internal requests from the server (trong advanced setting): http://docserver/ (theo alias docker) (Optional)
- Server address for internal requests from the Document Editing Service (trong advanced setting): http://nextcloud/ (theo alias docker) (Optional)
- Đánh sercret key đã config trên OnlyOffice vào trường secretkey

![nextcloud onlyoffice app](/screenshot/nextcloud-onlyoffice-app.PNG)

#### Hướng dẫn cài đặt ở local
Tải release của onlyoffice app (ở đây ta dùng link bản v7.0.2, có thể dùng bản khác)
```sh
wget --no-check-certificate --content-disposition https://github.com/ONLYOFFICE/onlyoffice-nextcloud/releases/download/v7.0.2/onlyoffice.tar.gz
```

Sau đó ta giải nén ra thư mục chứa app của nextcloud
```sh
tar -xf onlyoffice.tar.gz -C /mnt/docker-mount/nextcloud-config
```

