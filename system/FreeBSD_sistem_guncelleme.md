# FreeBSD Sistem Güncelleme

Merhaba,

Şunda hem fikirizdir ki, güvenlik yamalarını zamanında uygulamak ve işletim sistemini daha yeni bir sürüme yükseltmek, sistem yönetiminin önemli yönleridir. FreeBSD, her iki görevi de gerçekleştirmek için kullanılabilen, freebsd-update adı verilen bir yardımcı program içerir.


Bu yardımcı program yardımıyla, yamayı veya yeni bir çekirdeği manuel olarak derleyip yüklemeye gerek kalmadan FreeBSD'yi güncellemek mümkündür. Bu da güncelleme işlemini sistem yöneticisi açısından kolaylaştıran bir durum.


freebsd-update programı minör sistem güncellemeleri yapabildiği gibi başka bir sürüm dalına yükseltmeleri de destekler. Örnek vermek gerekirse; mesela 13.1_RELEASE kullanıyorsunuz. Bu sürüm dalında güncelleme yaparak örneğin 13.1-RELEASE-p9 sürümüne güncelleme yapabilirsiniz. Ya da sürüm dalını tamamen güncelleyerek 13.1 sürümünden 13.2 sürümüne geçiş de yapabilirsiniz. Şimdi ilk durumu nasıl yapabileceğimizi inceleyelim, aslında bu çok kolay. Vermemiz gereken sadece iki komut var;

```
# freebsd-update fetch
```
```
# freebsd-update install
```

bu iki komut ve ardından sistemi yeniden başlattığımızda bulunduğumuz sürüm dalı içindeki son gücellemeleri sisteme uygulamış oluyoruz. Bu makalede asıl irdeleyeceğimiz güncelleme başka bir sürüm dalına yapılacak güncelleme olacak. Dilerseniz bu işlemi de madde madde inceleyelim..

## **Adım 1:**

Her zaman olduğu gibi işleme başlamadan önce sistemdeki kritik tüm veriyi yedeklemeniz önemli. Sistemdeki kritik verilerin yedeklendiğine emin olmadan lütfen güncelleme yapmayınız..

## **Adim 2:**

Kurulu olan sürüm dalımızı en son güncellemeye getirelim;

```
# freebsd-update fetch
```
```
# freebsd-update install
```

Aslında bu yukarıda da bahsettiğimiz işlem. Sürüm dalı güncellemeden önce sistemimizi içinde bulunduğumuz dalın en güncel haline çıkarıyoruz. Bu komutlar zaten güncel bir dalda olmamız durumunda şöyle bir çıktı üretecektir;

```
src component not installed, skipped 
Looking up update.FreeBSD.org mirrors... 2 mirrors found. 
Fetching metadata signature for 13.1-RELEASE from update2.freebsd.org... done. 
Fetching metadata index... done. 
Inspecting system... done. 
Preparing to download files... done.
```

No updates needed to update system to 13.1-RELEASE-p9.

Sistemi yeniden başlatalım;

```
# reboot
```

## **Adım 3:**

Sistemde halihazırda bulunan tüm paketleri güncelleyelim.. Ben pkg kullanıyorum bu işlemi gerçekleştirmek için verilmesi gereken komutlar;

```
# pkg update
```
```
# pkg upgrade
```

## **Adım 4:**

FreeBSD 13.1 sürümünden 13.2 sürümüne geçmek için gerekli komutu verelim;

```
# freebsd-update -r 13.2-RELEASE upgrade
```

Bu komutu verdikten sistem kontrol edilecek ve size bulunanların uygun görünüp görünmediği sorulacaktır. Y ile onaylayıp bu aşamayı geçin. İşlemler tamamlandıktan sonra değişiklikleri uygulamak için;

```
# freebsd-update install
```

komutunu verin. Bu aşamada sunucunuz SSH ile uzaktan eriştiğiniz bir sunucu ise minik bir kontrol yapmak hayatınızı kolaylaştırabilir;

```
# /usr/sbin/sshd -t
```

komutu ile SSH servisinde bir sorun olup olmadığını test edelim. Bir sorun yoksa sistemi yeniden başlatalım;

```
# reboot
```

sistem yeniden açıldıktan sonra bir kez daha;

```
# freebsd-update install
```

komutunu veriyoruz.

## **Adım 5:**

Sistemde kurulu paketleri güncellemeye başlıyoruz. Öncelikle pkg'nin kendisini güncelliyoruz;

```
# pkg-static install -f pkg
```

Bu komutun çıktısı şöyle olacaktır;

```
Updating FreeBSD repository catalogue...
FreeBSD repository is up to date.
All repositories are up to date.
The following 1 package(s) will be affected (of 0 checked):

Installed packages to be REINSTALLED:
	pkg-1.19.1_1

Number of packages to be reinstalled: 1

8 MiB to be downloaded.

Proceed with this action? [y/N]: y 
[1/1] Fetching pkg-1.19.1_1.pkg: 100% 8 MiB 2.9MB/s 00:03 
Checking integrity... done (0 conflicting) 
[1/1] Reinstalling pkg-1.19.1_1... 
[1/1] Extracting pkg-1.19.1_1: 100%
```

Şimdi sırası ile şu komutları verelim;

```
# pkg update
```
```
# pkg upgrade
```
```
# freebsd-update fetch
```
```
# freebsd-update install
```

Son bir kez sistemi yeniden başlatıyoruz;

```
# reboot
```

## **Adım 6:**

Sistemimizin gerçekten de 13.2 sürümüne yükselip yükselmediğini kontrol edelim;

```
# freebsd-version
```
```
# uname -r
```

bu komutların çıktıları sıra ile;

```
# 13.2-RELEASE-p2
```
```
# 13.2-RELEASE-p2
```

Tebrikler, FreeBSD işletim sisteminizi başarı ile 13.1 sürümünden 13.2 sürümüne çıkarttınız. Umarım işinize yarayan bir makale olmuştur..

# Lisans

Bu makale Bedreddin Şahbaz tarafından yazılmıştır. BSD-3-Clause ile ruhsatlanmıştır.
