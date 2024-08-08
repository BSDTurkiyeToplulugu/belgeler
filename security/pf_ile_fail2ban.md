# FreeBSD üzerinde PF ile fail2ban kullanımı

Merhaba,

Bu yazıda sizlerle fail2ban uygulamasını FreeBSD bir sistem üzerinde PF (Packet Filter) kullanarak nasıl çalıştırabileceğinizi anlatmak istiyorum. fail2ban'a neden ihtiyaç duydum öncelikle bundan bahsederek konuya giriş yapalım. Bir kaç gün önce nginx web sunucuma çok anlamsız ve çok sayıda istekler geldiğini ve bu istekler yüzünden php-fpm sürecinin yoğun çalışarak sistemi zorladığını ve zaman zaman da durduğunu fark ettim. Gelen bu istekleri analiz edip nasıl engellerim diye araştırırken fail2ban uygulamasını fark ettim. Peki nedir bu fail2ban? Kısaca izah etmeye çalışayım;

fail2ban, sunucuları kötü niyetli giriş denemelerine karşı korumak için kullanılan açık kaynaklı bir yazılımdır. Bu yazılım, özellikle SSH, FTP, HTTP, SMTP ve diğer hizmetlere yönelik brute-force saldırılarını tespit etmek ve engellemek için yaygın olarak kullanılır. Fail2ban, sistem loglarını izleyerek belirli bir süre içinde belirli bir sayının üzerinde başarısız oturum açma girişimi veya diğer kötü niyetli faaliyetleri tespit eder. Tespit edilen bu IP adresleri, belirli bir süre boyunca engellenir. Engelleme, genellikle bir firewall kuralı ekleyerek veya mevcut olanı değiştirerek yapılır. Biz de firewall olarak FreeBSD ile birlikte gelen PF firewall yazılımını kullanacağız. O zaman kurulumla işe başlayalım. fail2ban kurmak aslında gayet kolay, vermemiz gereken tek komut;

```
# pkg install py311-fail2ban
```

paket ismi önündeki py311 ifadesi sisteminizdeki Python versiyonuna göre değişebilir. Bunu kontrol etmek için pkg veritabanını arama yaparak tarayabilirsiniz. Yukarıdaki komut fail2ban yazılımını sisteme kuracaktır. Yazılımın ayarlarını yapmak için ```/usr/local/etc/fail2ban``` dizinine gidelim. Bu dizinde öncelikle bir dosyayı farklı bir isimle kopyalamamız lazım;

```
# cp jail.conf jail.local
```

Bu işlemi neden yaptık? Güncellemeler sırasında jail.conf üzerine yazılabileceği ve bizim yaptığımız düzenlemeler kaybolabileceği için bu kopyalama işlemini yaptık. fail2ban'ın çalışma mantığından bahsedelim biraz da. fail2ban, "jails" adı verilen bölümler kullanarak çalışır. Her jail, belirli bir hizmeti izler ve belirli bir filtre kullanır. Filtreler, genellikle regex (düzenli ifade) kullanarak log dosyalarını tarar ve kötü niyetli faaliyetleri tespit eder. Filtre kurallarını filter.d dizini altına ```filtre_adi.conf``` formatında dosyalar oluşturup bu dosyalara yazıyoruz. Hemen nginx için bir filtre oluşturalım, favori metin editörünüz ile ```filter.d dizini``` altında örneğin ```ddos.conf``` adında bir dosya oluşturalım ve bu dosyaya şunu ekleyelim;

```
[Definition]
failregex = <HOST> -.*"(GET|POST).*" 200
ignoreregex =
```

Bu filtre nginx ```access.log``` dosyasında belirli bir ifadeyi arıyor gördüğünüz gibi. Filtre kuralını ekledik, şimdi bu filtreyi kullanabilmek için jail.local dosyasını açalım ve en sona şu ifadeyi ekleyelim;

```
[ddos]
enabled = true
port = http,https
filter = ddos
logpath = /var/log/nginx/access.log
maxretry = 100
bantime = 3600
findtime = 60
```

Bu yapılandırmada:

```
enabled = true ifadesi, bu jail'i etkinleştirir.
port = http,https ifadesi, HTTP ve HTTPS portlarına gelen trafiği denetler.
filter = ddos ifadesi, ddos adında bir filtreyi kullanır.
logpath = /var/log/nginx/access.log ifadesi, Nginx log dosyasının yolunu belirtir.
maxretry = 100 ifadesi, bir IP'nin kaç kez başarısız deneme yapabileceğini belirtir.
bantime = 3600 ifadesi, IP'nin ban süresini (saniye olarak) belirtir.
findtime = 60 Bu süre zarfında belirtilen maxretry sayısının aşılması durumunda IP adresi engellenir. Bu örnekte 1 dakika (60 saniye) olarak ayarlanmıştır.
```

Bu jail kuralı ```access.log``` dosyasında 60 saniye içinde bir ip numarasından 100 adet istek tespit ederse o ip adresine 1 saat boyunca ban koyacaktır.

Bu noktaya kadar işin fail2ban kısmında çalıştık. Şimdi PF için bir kaç ayar yapmamız lazım. fail2ban bir aksiyonu nasıl yapacağına ```action.d dizinindeki``` dosyalara bakarak karar veriyor. PF ile ilgili bir ayar dosyası var ancak ben bunu kaldırıp yerine daha sade bir dosya koydum.

```
# cp pf.conf pf.conf.ori
```

```pf.conf dosyasını``` açarak içerine şunları ekleyelim;

```
[Definition]
actionstart =
actionstop =
actioncheck =
actionban = /sbin/pfctl -t <tablename> -T add <ip>/32
actionunban = /sbin/pfctl -t <tablename> -T delete <ip>/32
[Init]
tablename = fail2ban
```

```jail.local dosyasında``` yapmamız gereken bir şey kaldı. Dosyayı açarak şu parametreleri bulun ve aşağıdaki gibi değiştirin;

```
banaction = pf[actiontype=<allports>]
banaction_allports = pf[actiontype=<allports>]
```

Hemen hemen hazırız. PF için bir kural eklememiz gerekiyor, ```/etc/pf.conf``` dosyasını açalım/yoksa oluşturalım ve içerisine uygun yerde olacak şekilde şu satırları ekleyelim;

```
table <fail2ban> persist
```

```
block in quick log on vmx0 from <fail2ban> to any
```

Lütfen vmx0 yazan yere kendi ağ arayüzünüzü yazın. Bu bilgiye ifconfig komutunu kullanarak erişebilirsiniz. Yapmamız gereken son şey ```/etc/rc.conf dosyasına``` gerekli satırları eklemek ve uygulamaları başlatmak. ```rc.conf``` dosyamızı açalım ve şu satırları ekleyelim;

```
fail2ban_enable="YES"
pf_enable="YES"
pf_rules="/etc/pf.conf"
pf_flags=""
pflog_enable="YES"
pflog_logfile="/var/log/pflog"
```

Uygulamaları başlatalım;

```
# /usr/local/etc/rc.d/fail2ban start
# service pf enable
```

Bir sorun olmadı ise ```fail2ban``` ```nginx access.log dosyamızı``` izlemeye başlamış olmalı ve verdiğimiz kurala uygun ip numaralarına PF kullarak ban koymalıdır.  Sistemin çalışıp çalışmadığını anlamanın pratik bir yolu ```fail2ban-client``` ve ```pfctl``` komutlarını kullanmak. Aşağıdaki komut bize ban konuşmuş bir ip olup olmadığını verecektir;

```
# fail2ban-client status ddos
```

Bu komutun çıktısı şuna benzemeli;

```
Status for the jail: ddos
|- Filter
| |- Currently failed: 2
| |- Total failed: 107
| `- File list: /var/log/nginx/access.log
`- Actions
|- Currently banned: 0
|- Total banned: 0
`- Banned IP list:
```

Gördüğünüz gibi şu anda sistem henüz hiç bir ip ye ban koymamış. Bu listede "Banned IP list" alanında gördüğünüz ip numaralarını şu komutu verdiğinizde de görüyorsanız her şey yolunda demektir;

```
# pfctl -t fail2ban -T show
```

Bu komut da o anda PF tarafından ```fail2ban``` tablosunda engellenmiş olan ip numaralarının listesini verecektir. Önceki komuttaki ip ler ile bu listedeki ip ler bire bir birbirini tutuyorsa sistemi düzgün şekilde konfigüre etmişsiniz demektir.

Sonuç olarak; ```fail2ban``` pek çok yerleşik filtre ile birlikte gelen sizin de yeni filtreler oluşturabileceğiniz, hayal gücünüz ile genişleme kapasitesine sahip güzel bir güvenlik aracı. Umarım makale faydalı olmuştur.

Saygılarımla..


# Lisans

Bu makale Bedreddin Şahbaz tarafından yazılmıştır. BSD-3-Clause ile ruhsatlanmıştır.
