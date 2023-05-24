# Let’s Encrypt ve tinydns Kullanarak Wildcard Sertifika Üretimi

Bu dökümanda; Let’s Encrypt Wilcard Sertifikasını, DNS sunucu olarak tinydns kullanan bir sistem yöneticisinin nasıl kolayca alabileceğini ve akabinde sertifika yenilenmesinde işi nasıl otomatize edebileceğini kendi deneyimim üzerinden aktarmaya çalışacağım.

 Let’s Encrypt nedir, ne işe yarar, nasıl çalışır sorularına bu yazıda çok yer vermek istemiyorum. Eğer Let’s Encrypt nedir, nasıl çalışır konusunda bilginiz yetersiz ise öncelikle o konuda biraz araştırma yapmanız yerinde olacaktır.

 Let’s Encrypt’in sertifika oluşturma için bir kaç “challange” metodu olduğunu biliyorsunuz. Wildcard tipi sertifikalar için bir kısıtlama söz konusu ve bu tip sertifika alabilmek için DNS challange metodunu kullanmak mecburi. Bu metod çok kabaca şöyle çalışıyor; Let’s Encrypt sertifika oluşturma aşamasında, txt tipinde bir kayıt üretip bunu DNS sunucunuzdan yayınlamanızı istiyor. Siz DNS kaydını girip sertifika işlemine devam ettiğinizde Let’s Encrypt size ilettiği txt kaydının alan adınızda olup olmadığını sorguluyor ve sorgu başarılı olur ise wildcard sertifika üretme işlemi başarı ile gerçekleşiyor. Buradaki amaç elbette sertifika üretmek istediğiniz alan adının gerçekten sahibi olup olmadığını otomatik bir şekilde kontrol etmek. Let’s Encrypt süreçleri otomatik hale getirip insan müdahalesini gerektirmeyen bir yapıya sahip.

 Bu işlem için Let’s Encrypt üzerinde kullanılabilecek plug-in yazılımları mevcut. DNS hizmetini aldığınız büyük oyuncular için işlemi otomatikleştiren plug-in bulabiliyorsunuz. Ancak kendi tinydns sunucusunu çalıştıran benim gibiler için bir kaç düzenleme yapmak gerekiyor. Neyse ki tinydns’in esnek ve basit yapısı nedeni ile bu gayet kolay. Nasıl yapıldığına geçelim.

 Öncelikle yaptığım her şey FreeBSD üzerinde bunu belirteyim. GNU/Linux üzerinde de çok yakın bir şekilde aynı yolu izleyebilirsiniz. Bazı dosya yolları ve komutlar farklılık gösterebilir.

 Let’s Encrypt dosyalarının bulunduğu /usr/local/etc/letsencrypt dizinine girelim. Bu dizinde cli.ini adında bir ayar dosyası göreceksiniz. Bu dosyayı aşağıdaki gibi düzenleyelim;

 ```
 manual
server = https://acme-v02.api.letsencrypt.org/directory
rsa-key-size = 4096
preferred-challenges = dns
manual-auth-hook = /usr/local/etc/letsencrypt/certbot-dns-auth.sh
manual-cleanup-hook = /usr/local/etc/letsencrypt/certbot-dns-auth-cleanup.sh
 ```

 certbot yazılımı bize çok rahatlık sağlayan manual-auth-hook ve manual-cleanup-hook tanımlama şansını veriyor. Bu tanımlamalar sayesinde certbot içerisinden shell scriptlerimizi çağırarak sertifika üretimi öncesinde ve sonrasında bazı işlemler yapabiliyoruz. Sertifika oluşturma öncesinde tinydns sunucumuza Let’s Encrypt tarafından gönderilen txt kaydını giren shell scripti görelim,

 ```
 # cat /usr/local/etc/letsencrypt/certbot-dns-auth.sh
#!/usr/local/bin/bash
 ```

 ```
 cd /etc/tinydns/root
cp -f data data-good
echo "'_acme-challenge.$CERTBOT_DOMAIN:$CERTBOT_VALIDATION:120" >> data
/usr/local/etc/letsencrypt/update.sh
# Sleep to make sure the change has time to propagate over to DNS
sleep 25
 ```

 tinydns için benim kurulumunda root dizini /etc/tinydns/root yolunda, bu dizine geçip data dosyasını data-good olarak her ihtimale karşı yedekliyoruz. certbot tarafından gönderilen txt tipi kaydı data dosyasına yazıyoruz ve update.sh shell scriptini çağırıyoruz. Bu scripti de inceleyelim;

 ```
 # cat /usr/local/etc/letsencrypt/update.sh
#!/bin/sh
 ```

 ```
 make
rsync -avz -e ssh /etc/tinydns/root/data.cdb root@ikinci_tinydns_ip:/etc/tinydns/root
rsync -avz -e ssh /etc/tinydns/root/data root@ikinci_tinydns_ip:/etc/tinydns/root
 ```

 make komutu ile data dosyasını cdb formatına çevirip rsync satırları ile varsa ikinci, üçüncü, vb. diğer sunucularımıza da bu değişimi gönderiyoruz. Böylece tüm tinydns sunucularımızda gerekli txt kaydı kaydedilmiş oldu. Bir sonraki aşamada /usr/local/etc/letsencrypt/certbot-dns-auth-cleanup.sh scripti çalışacak ve girilen txt kaydını DNS sunucumuzdan temizleyecek. Bu scriptin içeriği de:

 ```
 # cat /usr/local/etc/letsencrypt/certbot-dns-auth-cleanup.sh
#!/usr/local/bin/bash

cd /etc/tinydns/root
sed '$d' data > tmp; mv -f tmp data
/usr/local/etc/letsencrypt/update.sh
 ```

 Yine tinydns root dizinine geçip biraz önce eklediğimiz kaydı silerek update.sh scriptini yeniden çağırıyoruz.

 Bu scriptlerin tümü başarı ile çalıştığında cerbot şu şekilde çıktı üretecek;

 ```
 # certbot certonly -d '*.domain.com'
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Requesting a certificate for *.domain.com
Hook '--manual-auth-hook' for domain.com ran with output:
 /usr/local/bin/tinydns-data
 sending incremental file list
 data.cdb

 sent 1,970 bytes  received 137 bytes  1,404.67 bytes/sec
 total size is 11,671  speedup is 5.54
 sending incremental file list
 data

  sent 281 bytes  received 101 bytes  764.00 bytes/sec
 total size is 7,206  speedup is 18.86
Hook '--manual-cleanup-hook' for domain.com ran with output:
 /usr/local/bin/tinydns-data
 sending incremental file list
 data.cdb


 sent 1,848 bytes  received 137 bytes  3,970.00 bytes/sec
 total size is 11,551  speedup is 5.82
 sending incremental file list
 data

  sent 218 bytes  received 101 bytes  638.00 bytes/sec
 total size is 7,121  speedup is 22.32

 Successfully received certificate.
Certificate is saved at: /usr/local/etc/letsencrypt/live/domain.com/fullchain.pem
Key is saved at:         /usr/local/etc/letsencrypt/live/domain.com/privkey.pem
This certificate expires on 2022-04-01.
These files will be updated when the certificate renews.


NEXT STEPS:
- The certificate will need to be renewed before it expires. Certbot can automatically renew the certificate in the background, but you may need to take steps to enable that functionality. See https://certbot.org/renewal-setup for instructions.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
 ```

 Sertifikamız başarı ile oluştu. Yapılması gereken tek bir şey var. Let’s Encrypt wildcard sertifikaları 3 ay geçerli oalcak şekilde veriyor. Bu nedenle 3 ayda bir bu sertifikaların yenilenmesi lazım. cli.ini dosyasında yaptığımız düzenleme sertifika yenilenirken de işe yarıyor. Tek yapmamız gereken bir cron işi oluşturmak ve her gün düzenli şekilde certbot’a sistemimizde yüklü sertifikaları denetletmek. Örnek bir cron işi;

 ```
 0 1 * * * root /usr/local/bin/certbot renew >> /var/log/letsencrypt/renew.log --post-hook "service nginx restart"
 ```

 Tebrikler, Let’s Encrypt wildcard sertifikanızı güle güle kullanın..

# Lisans

Bu makale Bedreddin Şahbaz tarafından yazılmıştır. BSD-3-Clause ile ruhsatlanmıştır.
