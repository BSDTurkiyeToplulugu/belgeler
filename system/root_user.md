# Güncelleme sonrası root kullanıcısı için şifre istemeden oturum açılması sorunu

Merhaba,

FreeBSD 13.2-RELEASE sistemimi 14.0-RELEASE'e freebsd-update aracı ile güncelledim ve sonrasında ssh üzerinden sisteme bağlandığımda su komutu ile root olmak istediğimde şifre sorulmadığını fark ettim. su komutu verdiğim anda root olabiliyordum. Sorunun neden kaynaklandığını tespit edemesem de basit bir çözümü var. passwd komutu ile root şifrenizi yeniden set ederseniz problem çözülüyor.

İpucu için caylak caylakpenguen'e teşekkürler..

# Lisans

Bu makale Bedreddin Şahbaz tarafından yazılmıştır. BSD-3-Clause ile ruhsatlanmıştır.
