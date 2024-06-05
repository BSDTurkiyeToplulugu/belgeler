# ZFS dosya sisteminde diskin boş alanının sürekli azalması

Merhaba,

Bugün sizlere ZFS dosya sisteminin yeni fark ettiğim bir özelliğinden bahsetmek isterim. Anlamam tesadüfen oldu, muhtemelen ZFS uzmanları anlatacağın şey bu mu diyebilir lakin bulduğum ana kadar durum çok gizemli idi :) Sanallaştırılmış FreeBSD sunucularımda boş disk alanının sürekli azalması dikkatimi çekmiş idi. Bu durumu ilk önce freebsd-update ile sistemi güncelleyeceğim zaman fark etmiştim. Diskte yeterli alan olmaması nedeni ile işlem tamamlanmayınca gerekmeyen dosyaları silerek yer açmaya çalıştım. İlk etapta bunda başarılı da oldum. Gereksiz log dosyaları, pkg dosyaları vb. silerek belli bir alan kazanabildim. Ancak ilerleyen zamanda boş disk alanı yine azaldı ve daha önce kullandığım yöntemler bu sefer işe yaramadı. Duruma anlam veremediğim için çareyi sanal disk boyutunu büyütmekte buldum. Ancak boyutu büyütülmüş sanal disk bile bir süre sonra dolmaya başladı. Diski dolduran şeyin ne olduğunu tesadüf eseri öğrendim. ZFS dosya sisteminin belli aralıklarla aldığı "snapshot"lar diskin dolma nedeni.

ZFS snapshotlar nedir? ZFS snapshotlar, dosya sistemince oluşturulan ve dosya sisteminin belli bir andaki anlık görünümü olarak özetlenebilir. Veri kurtarma veya sistemin eski bir andaki durumuna geri dönme gibi senaryolarda yararı olan bir özellik. Ancak bu yararlı özellik siz fark etmeden diskinizi doldurmaya devam ediyor. İşe yaramayan snapshotları temizlemek diskte alan açmanızı sağlayabilir. Öncelikle sistemimizdeki snapshotları listeleyelim, bunun için;

```
# zfs list -t snapshot

Bu komutun şunun gibi bir çıktısı olacaktır;

NAME USED AVAIL REFER MOUNTPOINT
zroot/ROOT/default@2022-11-12-11:57:51-0 478M - 4.43G -
zroot/ROOT/default@2022-11-18-12:00:56-0 145M - 4.41G -
zroot/ROOT/default@2022-12-01-10:20:54-0 241M - 4.41G -
zroot/ROOT/default@2023-02-20-12:02:29-0 930M - 4.64G -
zroot/ROOT/default@2023-08-16-15:59:50-0 186M - 2.71G -
zroot/ROOT/default@2023-09-02-19:32:52-0 7.60M - 4.08G -
zroot/ROOT/default@2023-09-02-19:34:51-0 4.49M - 4.16G -
zroot/ROOT/default@2023-10-05-13:48:35-0 172M - 4.32G -
zroot/ROOT/default@2023-11-20-10:22:20-0 142M - 4.33G -
zroot/ROOT/default@2023-12-12-15:57:14-0 16.7M - 4.41G -
zroot/ROOT/default@2023-12-14-19:06:07-0 9.52M - 4.47G -
zroot/ROOT/default@2024-01-02-10:23:20-0 290M - 4.50G -
zroot/ROOT/default@2024-02-20-15:32:53-0 785M - 4.86G -
zroot/ROOT/default@2024-04-02-10:10:08-0 537M - 2.83G -
zroot/ROOT/default@2024-05-16-10:03:23-0 7.12M - 4.79G -
zroot/ROOT/default@2024-05-16-10:06:06-0 0B - 4.87G -
zroot/ROOT/default@2024-05-16-10:06:19-0 0B - 4.87G -
zroot/ROOT/default@2024-05-16-12:04:25-0 307M - 5.05G -
```

Gördüğünüz gibi ZFS tarafından oluşturulmuş pek çok snapshot dosyası var, dosyaların disk üzerinde kapladığı alan da muhtelif. Şimdi bu snapshotların tümünü veya istediklerinizi nasıl temizleyeceğinizden bahsedelim. Kullanılacak komuttan bahsetmeden önce bir uyarı! Az sonra yapacağımız snapshot silme işlemi geri dönüşü olmayan bir işlem, lütfen ihtiyacınız olan bir snapshot üzerinde silme yapmadığınıza emin olun.. Tüm snapshotları temizlemek isterseniz şu komutu verin;

```
# zfs destroy -r zroot/ROOT/default
```

Ben listeden sadece istediğim snapshotları sileceğim derseniz kullanmanız gereken komut ise şu;

```
# zfs destroy -R zroot/ROOT/default@2022-11-12-11:57:51-0
```

```"zfs destroy -R"``` ifadesinden sonra silmek istediğimiz snapshotın adını yazıyoruz. ```-R``` parametresi recursive olarak bu snapshot ile ilgili klonlar varsa onları da siliyor. Şimdi temizlik öncesi ve sonrası zpool list komutu çıktılarına bakalım. Snapshotları silmeden önce diskin durumu şöyle idi;

```
# zpool list
NAME SIZE ALLOC FREE CKPOINT EXPANDSZ FRAG CAP DEDUP HEALTH ALTROOT
zroot 22.5G 20.5G 2.03G - - 79% 90% 1.00x ONLINE -
```

Temizlik sonrasında ise;

```
# zpool list
NAME SIZE ALLOC FREE CKPOINT EXPANDSZ FRAG CAP DEDUP HEALTH ALTROOT
zroot 22.5G 9.06G 13.4G - - 49% 40% 1.00x ONLINE -
```

Gördüğünüz gibi boş disk alanında bir hayli artış var.. Siz de FreeBSD sisteminizde anlam veremediğiniz disk alanı azalması yaşıyorsanız ZFS dosya sisteminin snapshotlarını bir kontrol edin.

Saygılarımla..

# Lisans

Bu makale Bedreddin Şahbaz tarafından yazılmıştır. BSD-3-Clause ile ruhsatlanmıştır.


