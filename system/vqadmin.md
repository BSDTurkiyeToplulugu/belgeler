# Vqadmin "invalid language file" hatası

Herkese Merhaba,

Vqadmin bildiğiniz gibi vpopmail komutlarını web arayüzü ile çalıştırmak için bir cgi betiği. Bu betiği kullanmak istediğimde firefox tarayıcısı sürekli olarak "invalid language file" hatası veriyordu. Hatayı aşmanın yolunu Chrome üzerinden girip öncelikli dil ayarını İngilizce yapmak olarak bulmuştum. Bugün bu hata neden olur nasıl gideririm diye derinliğine bir araştırma yaptım. Sorunu çözdüm ve FreeBSD üzerinde nasıl çözüleceğini sizlerle detaylı şekilde paylaşacağım.

Burada anlatacaklarım FreeBSD port ağacında işlemin nasıl uygulandığını anlatıyor ancak siz kendiniz derlerken de aynı mantığı kullanabilirsiniz.

Öncelikle soruna yol açan şeyin ne olduğunu bulmaya çalıştım. Bulduğum şey şu oldu; Vqadmin uygulamayı ziyaret ettiğimiz tarayıcıdan gelen başlıklardan bazılarını kontrol ediyor. Bu başlıklardan bir tanesi HTTP_ACCEPT_LANGUAGE. Bu başlık içinde güvenlik gerekçesi ile . ve / karakterleri olup olmadığını kontrol eden bir fonksiyon buldum. Sorun da aslında buradan kaynaklanıyor. Vqadmin kullanıcıdan HTTP_ACCEPT_LANGUAGE olarak birbirinden virgüllerle ayrılmış dil kodlarını beklerken modern Firefox tarayıcım şöyle bir HTTP_ACCEPT_LANGUAGE başlığı gönderiyor "tr-TR,tr;q=0.8,en-US;q=0.5,en;q=0.3" Bu başlıktaki q parametresi hangi dilin öncelikli olduğunu sunucu tarafına söyleyen bir parametre. Dikkat etti iseniz bu başlıkta . karakterleri bulunuyor. Bu . karakterleri Vqadmin'in kontrolüne takılıp global_error yükselmesine neden oluyor.

Bunu engellemek için HTTP_ACCEPT_LANGUAGE değerinden noktaların temizlenmesinin yeterli olacağını düşündüm. Bu düşüncemi uygulamaya koydum. Sizlere tek tek ne yaptığımı anlatacağım.

Öncelikle Vqadmin'in port ağacındaki dizinine geçiyoruz,
```
# cd /usr/ports/mail/vqadmin
```
Bu dizinde peş peşe şu komutları veriyoruz,
```
# make distclean
```
```
# make deinstall
```
Bu komutlar daha önceden derlediğimiz Vqadmin dosyalarını temizliyor ve hali hazırdaki kurulumumuzu sistemden kaldırıyor. Şimdi Vqadmin kaynak kodunu indirip work dizinine açalım,
```
# make extract
```
Bu komut kaynak kodu indirip bulunduğumuz dizindeki work dizinine açacaktır. İşlemi tamamladıktan sonra kaynak kodun bulunduğu dizine geçelim,
```
# cd work/vqadmin-2.3.6
```
Bu dizin altındaki lang.c dosyası ilgilendiğimiz dosya. Bu dosyaya bir fonksiyon ve bir ufak ekleme yapacağız. Dosyayı açıyoruz,
```
# ee lang.c
```
Ekleyeceğimiz fonksiyon bir değişken içindeki . karakterlerini temizleyen bir fonksiyon olacak. Aşağıda bulunan bu fonksiyona ait kodları dosyanın 36. satırından itibaren ekleyelim (36. satır önemli olabilir, kodun bütünlüğünü bozmayan bir yer olduğu için özellikle belirtmek istedim bu satırı).
```
char* remove_dots(char *str) {
int i, j;
for (i = 0, j = 0; str[i] != '\0'; i++) {
if (str[i] != '.') {
str[j++] = str[i];
}
}
str[j] = '\0';

return str;
}
```
Bu eklemeden sonra şu satırı bulun,
```
tmpstr = getenv("HTTP_ACCEPT_LANGUAGE");
```
Ve bu satırı şu şekilde değiştirin,
```
tmpstr = remove_dots(getenv("HTTP_ACCEPT_LANGUAGE"));
```
dosyayı kaydedip çıkın. Ve iki dizin üste dönün,
```
# cd ../..
```
Şimdi kodu derleyip yükleyebiliriz,
```
# make install
```
Evet artık "invalid language file" hatasını almıyor olmamız lazım..

Saygılarımla..

# Lisans

Bu makale Bedreddin Şahbaz tarafından yazılmıştır. BSD-3-Clause ile ruhsatlanmıştır.
