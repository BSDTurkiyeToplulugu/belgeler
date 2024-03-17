# Port Ağacında Kullanılabilecek make Komutları

Herkese Yeniden Merhaba,

FreeBSD işletim sistemini ilginç kılan özelliklerinden birisi bildiğiniz gibi "Port Ağacı". Bu yazıda port ağacının detaylarına girmeden port ağacında kullanabileceğiniz make komutları hakkında yazmak istedim. İşte make komutlarının en çok kullanılanları ve ne işe yaradıkları;

**make config:** Portun konfigürasyon ekranını gösterir

**make fetch:** Portun kaynak kodlarını indirir, bunun dışında hiç bir işlem yapmaz

**make:** Kaynak kodu indirir ve derler ama kurmaz

**make build:** Portu derler

**make install:** Kaynak kodu indirir, derler ve kurar

**make clean:** Derlenmiş kodları (portun içindeki work dizinindeki) siler

**make install clean:** Kaynak kodu indirir, derler, kurar sonrasında derlenmiş kodları (portun içindeki work dizinindeki) siler

**make distclean:** Derlenmiş kodları (portun içindeki work dizinindeki) siler ayrıca kaynak kod dosyasını da siler

**make deinstall:** Portu kaldırır

**make reinstall:** Kurulu bir portu yeniden kurar ve yükler.

**make package:** Portu paketler

**make check:** Portun testlerini çalıştırır

Faydalı olması ümidi ile..

# Lisans

Bu makale Bedreddin Şahbaz tarafından yazılmıştır. BSD-3-Clause ile ruhsatlanmıştır.
