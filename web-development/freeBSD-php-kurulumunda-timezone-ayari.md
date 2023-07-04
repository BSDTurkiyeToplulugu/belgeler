# FreeBSD/PHP Kurulumunda Timezone Ayarı

Merhaba,

FreeBSD üzerinde kurulu PHP+Nginx web sunucumda tarih ile alakalı bazı sorunlar olduğunu farkettim. Şöyle ki sunucumda yapılan işlemlerde UTC+3 değil UTC+0 zaman dilimi kullanılıyor ve tüm işlemler 3 saat geriden yapılıyordu.

Durumdan emin olmak için bir test yapmak istedim. Saati veren bir PHP Scripti sunucumda çalıştırdım ve gerçekten de PHP işlemlerinde saat bilgisinin hatalı olduğunu farkettim. Kullandığım script ve verdiği çıktıyı aşağıya ekliyorum.

Kullandığım script;

```
<?php echo “Saat ” . date(“h:i:sa”); ?> 
```

Bu scriptin verdiği saat bilgisi şöyle;


```
Saat 06:38:19am
```

Oysa sistem saati o anda;

```
# date
Thu Sep  1 09:38:20 +03 2022
```

Sistem saati UTC+3 olduğu halde PHP Scriptleri UTC+0  olarak sonuç veriyor. Bu durumu düzeltmek için php.ini dosyasında ufak bir değişiklik yapmamız gerekiyor. FreeBSD üzerinde standart PHP kurulumunda php.ini dosyasının yolu /usr/local/etc/php.ini olarak belirleniyor. Bu dosyayı açıp içerisindeki şu satırı bulunuz;

```
;date.timezone = 
```

Bu satırın başındaki ; işaretini kaldırıp şu değeri verelim;

```
date.timezone = Europe/Istanbul
```

Burada ters slash işaretine dikkat edelim, normal slash konulursa ayar doğru çalışmıyor. Dosyayı kaydedip çıkarak kuruluma göre web sunucumuzu veya PHP-FPM’i yeniden başlatıp sonucu kontrol edelim. Yukarıdaki scripti çağırdığımızda saat doğru geliyor ise PHP için time-zone ayarımızı başarı ile yapmışız demektir.

# Lisans

Bu makale Bedreddin Şahbaz tarafından yazılmıştır. BSD-3-Clause ile ruhsatlanmıştır.