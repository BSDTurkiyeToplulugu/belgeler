# freebsd-update install işleminin aşırı uzun sürmesi

Herkese Merhaba,

Bugün 13.2 RELEASE olan sürümümü 14.0 RELEASE sürümüne yükseltmek için freebsd-update aracını kullanmak istediğimde reboot sonrası verilen ikinci "freebsd-update install" komutunun aşırı derecede uzun olduğunu gördüm. Konu ile ilgili biraz araştırma yaptım ve şu link sorunuma çözüm oldu https://forums.freebsd.org/threads/freebsd-13-2-release-14-0-release-upgrade-stuck.91152


Sorun ZFS ile ilgili gibi görünüyor. Geçici bir çözüm olarak işleme başlamadan önce şu komutu çalıştırırsanız bu yavaşlık sorununu çözebiliyorsunuz;

# sysctl vfs.zfs.dmu_offset_next_sync=0

Bu komutu çalıştırdıktan sonra "freebsd-update install" komutunu verin ve işlem normal süresinde tamamlansın.

Ayrıca; FreeBSD işletim sisteminizi freebsd-update ile nasıl güncelleyeceğinizi bilmiyorsanız lütfen FreeBSD Sistem Güncelleme yazımızı inceleyiniz.

Saygılarımla..

# Lisans

Bu makale Bedreddin Şahbaz tarafından yazılmıştır. BSD-3-Clause ile ruhsatlanmıştır.
