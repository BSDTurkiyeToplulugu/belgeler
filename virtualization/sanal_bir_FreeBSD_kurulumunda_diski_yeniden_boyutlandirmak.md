# Sanal Bir FreeBSD Kurulumunda Diski Yeniden boyutlandırmak

Merhaba,

Bugün sizlere sanallaştırma ortamında kurduğumuz FreeBSD sistemimizin HDD boyutunu nasıl arttıracağımızdan bahsetmek istiyorum. İlk kurulum esnasında sisteme tanımladığımız disk boyutu bazı nedenlerle bazen yetersiz kalabilmekte, diski yeniden nasıl boyutlandıracağımızı ZFS kullanılmış bir kurulum için nasıl yapmanız gerektiğini anlatmaya çalışacağım.

ÖNEMLİ NOT! Burada sizlere anlatacağım işlemler diskin kullanılmaz hale gelmesine yol açabileceği için lütfen tüm kritik verilerinizi doğru şekilde yedeklediğinizden emin olmadan uygulamaya geçmeyiniz.

## **Adım 1:**

Öncelikle kullandığımız sanallaştırma aracında sanal makinemiz için tanımladığımız disk alanını büyütmekle işe başlıyoruz. Bu işlem sanallaştırma ortamınıza göre farklılık gösterebileceği için detaya girmiyorum. Ben işlemi Proxmox VE üzerinde yaptım. Bu işlemi kolayca sanal makinenin içerisine girip Donanım seçeneğine giderek Disk Boyutlandır seçeneği ile yaptım ve sanal makineme 4 GB ek disk alanı eklenmesini sağladım.

## **Adım 2:**

Sanal makinemizde konsole gidip şu komutu çalıştırıyoruz;

```
root@web:/home/bedo # gpart show
=> 40 33554352 da0 GPT (20G) [CORRUPT]
40 1024 1 freebsd-boot (512K)
1064 984 - free - (492K)
2048 4194304 2 freebsd-swap (2.0G)
4196352 29358040 3 freebsd-zfs (14G)
```

gördüğünüz gibi en üstte CORRUPT olduğu görülüyor diskin. Öncelikle bu durumu düzeltelim. Bunun için çalıştırmamız gereken komut;

```
root@web:/home/bedo # gpart recover /dev/da0
da0 recovered
```

da0 recover edildi. Durumu kontrol edelim;

```
root@web:/home/bedo # gpart show
=> 40 41942960 da0 GPT (20G)
40 1024 1 freebsd-boot (512K)
1064 984 - free - (492K)
2048 4194304 2 freebsd-swap (2.0G)
4196352 29358040 3 freebsd-zfs (14G)
33554392 8388608 - free - (4.0G)
```

evet eklediğimiz 4 GB ilave disk alanı free olarak görünüyor artık.


## **Adım 3:**

4 GB olarak görünen free alanı freebsd-zfs'e eklememiz gerekiyor. Vermemiz gereken komut;

```
root@web:/home/bedo # gpart resize -i 3 da0
da0p3 resized
```

buradaki ```"-i 3"``` parametresi ```gpart show``` komutu çıktısında yer alan freebsd-zfs'in numarası fark ettiğiniz gibi. Son drumu kontrol edelim;

```
root@web:/home/bedo # gpart show
=> 40 41942960 da0 GPT (20G)
40 1024 1 freebsd-boot (512K)
1064 984 - free - (492K)
2048 4194304 2 freebsd-swap (2.0G)
4196352 37746648 3 freebsd-zfs (18G)
```

evet ilave 4 GB alan artık freebsd-zfs'e eklenmiş halde.

## **Adım 4:**

Yapmamız gereken son bir işlem var;

```
root@web:/home/bedo # zpool online -e zroot da0p3
```

komutunu verip işlemi tamamlıyoruz. ```zpool list``` komutunun ilave alan eklemeden önceki ve sonraki çıktılarını paylaşıyorum sizlerle;

Öncesi;

```
root@web:/home/bedo # zpool list
NAME SIZE ALLOC FREE CKPOINT EXPANDSZ FRAG CAP DEDUP HEALTH ALTROOT
zroot  13.5G 9.72G 3.78G - - 59% 72% 1.00x ONLINE -
```

Sonrası;

```
root@web:/home/bedo # zpool list
NAME SIZE ALLOC FREE CKPOINT EXPANDSZ FRAG CAP DEDUP HEALTH ALTROOT
zroot 17.5G 9.72G 7.78G - - 46% 55% 1.00x ONLINE -
```

SIZE 13.5 G değerinden 17.5 G değerine yükselmiş durumda. Yeni alanı kullanmaya başlayabilirsiniz.

Saygılarımla..

# Lisans

Bu makale Bedreddin Şahbaz tarafından yazılmıştır. BSD-3-Clause ile ruhsatlanmıştır.
