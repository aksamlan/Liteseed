# Liteseed

> AO ve Arweave ekosisteminden elde ettiğimiz faydalar net bir şekilde ortada - Liteseed kurulumu kolay eksik kalmasın.

> Donanım yazmıyorum, herhangi bir sunucunuzun yanına kurabilir veya minimal bir cihaz tercih edilebilir.

> Teşvikli mi? Evet ama detay yok kesin demiyorum discord duyuruda yazıyor - taktir sizindir.

> Kolay kurulum için mümkün mertebe kısa ve özet yazıyorum..

## Kurulum:
```console
# güncelleme
sudo apt update -y && sudo apt upgrade -y
sudo apt install docker.io -y

git clone https://github.com/liteseed/edge

cd ./edge
docker build -t edge .

sudo docker volume create liteseed

# bir key oluşturacağız burada daha sonra kaydetmeyi göstereceğim.
docker run -v liteseed:/data edge generate
docker run -v liteseed:/data edge migrate

screen -S liteseed
docker run -v liteseed:/data edge start
# akabinde ctrl a d ile çıkalım.

# bu komutla cüzdan adresimizi öğrenip test tokeni isteyelim
docker run -v liteseed:/data edge balance
# buradaki cüzdana hello@liteseed.xyz mail adresıne göndererek token isteylim.
```

> Buradan sonra token geldikten sonra devam edebilirsiniz.


```console
# Burada tırnakların içersinde bir github veya twitter bağlantını koy.
docker run -v liteseed:/data edge stake -u "https://ruesandora.com"

# bu seger bu komutu girdiğinizde token elinizde kalmayacak ve stake kısmı Yes olacaktır.
docker run -v liteseed:/data edge balance

# screen içersine girip durdurup tekrar başlatalım.
screen -r liteseed
docker run -v liteseed:/data edge start
```

<img width="832" alt="Ekran Resmi 2024-05-16 15 49 59" src="https://github.com/ruesandora/Liteseed/assets/101149671/f684c11b-6c78-46c9-927e-40904e9eedf4">

> Böyle bir görsel node'unuzun doğru çalıştığını temsil eder.

> Hayırlı olsun, proje adına büyük konuşmak için çok erken - beklentisiz ilerleyiniz.

```console
cd /var/lib/docker/volumes/liteseed/_data
cat signer.json
# bu komut ile private keyinizi yedekleyebilirsiniz.
```

## Buradan sonra website işlemlerine geçiyoruz.

##### certbot yüklüyoruz

```bash
sudo apt update
sudo apt install -y certbot nginx
```

##### Ayarlar

```bash
sudo certbot certonly --manual --preferred-challenges dns \
--email <your-email-address> \
-d <your-domain.abc>
```

#Bu kodu giriyoruz ve aşağıdaki kodları içerisine yapıştırıyoruz. Aşağıdaki kodda yazan your-domain kısımlarını düzenliyoruz. <> bu kısımları da silmeniz gerek. `sudo nano /etc/nginx/sites-available/default`

```conf
# Force redirects from HTTP to HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name <your-domain.abc>;

    location / {
        return 301 https://$host$request_uri;
    }
}

# Forward traffic to your node and provide SSL certificates
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name <your-domain.abc>;

    ssl_certificate /etc/letsencrypt/live/<your-domain.abc>/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/<your-domain.abc>/privkey.pem;

    location / {
        proxy_pass http://localhost:8080; # or your port you changed at 6.
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1;
    }
}
```

Kayıt ediyoruz ve çıkıyoruz.

Onaylayıp yeniden balatırıyoruz Nginx'i.

```bash
sudo nginx -t
```

And restart nginx

```bash
sudo service nginx restart
```


#### 7.2 Cloudflare ayarlama

Cloudflare DNS service bölümüne giriyoruz, add A record your domain to IP **WITH proxy**.

![proxy on](./images//proxy-on.png)

şimdi **Rules** > **Origin Rules** bölümüne gidiyoruz.

 `+ Create rule` tıklıyoruz, `Rule name (required)` bir isim belirliyoruz ve field'e `Hostname` seçiyoruz yanına da domaininizi ekliyoruz.

ve `Rewrite to...` port numaranızı yazıyoruz. (8080)

<img src="./images//proxy-port.png" width="480" alt="proxy port" />


`deploy` diyoruz.


### 8. Sonra stake ettiğiniz tokenleri web sitenize girdiğinizde aşağıdaki şekilde görüyorsunuz.


```json
{
  "Address": "<address>",
  "Gateway": {
    "URL": "https://arweave.net"
  },
  "Name": "Edge",
  "Version": "0.0.4"
}
```

Stake işlemi aşağıdaki şekilde olmalıdır.

```
sudo docker run -v liteseed:/data edge stake -u $DOMAIN
```

## Etiket

`ar`, `liteseed`, `docker`, `go`, `domain`
