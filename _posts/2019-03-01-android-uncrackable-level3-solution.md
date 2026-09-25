---
title: OWASP Uncrackable Android Level 3
date: 2019-03-01 05:07:41 +0300
categories: [mobile]
tags: [android, frida, mobile, owasp uncrackable]
description: Owasp Uncrackable Android Level 3 uygulamasının incelenmesi ve içerdiği güvenlik önlemlerinin atlatılması
media_subpath: /assets/img/posts/android-uncrackable-level3-solution
image: cover.png
---

İlk olarak uygulama “.apk” dosyasının içeriği herhangi bir arşiv açma aracı kullanarak dışarı çıkartılır ve uygulamaya ait “.dex” dosyası elde edilir. Uygulama kodlarını JD-GUI decompiler aracı ile inceleyebilmek için “.dex” dosyalarını “.jar” formatına dönüştürmemiz gerekmektedir. Dönüşüm işlemi “d2j-dex2jar” aracı yardımı ile gerçekleştirilmektedir.

![](01.png)

JD-GUI decompiler aracı ile elde edilmiş olan “classes-dex2jar.jar” dosyası açılır ve uygulama kodları incelenmeye başlanır. “MainActivity” sınıfına ait “onCreate” metodunun, uygulamanın çalışmaya başlaması ile tetiklendiği ve içerisindeki işlemleri gerçekleştirdiği görülmektedir.

“onCreate”
metodu içerisinde;

- Root kontrolü
- Native kütüphanesi bütünlük kontrolü
- Debuggable özelliğinin kontrolü

![](02.png)

Sistem üzerinde root, bütünlük veya debuggable ihlallerinden birisinin tespiti sonucunda ise uygulamanın “showDialog” metodu kullanılarak kapatıldığı görülmektedir. Genel yapısı itibari ile UnCrackable-Level2 uygulamasına benzediği görülmektedir.

![](03.png)

Uncrackable-Level2 uygulamasından farklı olarak Uncrackable-Level3 içerisinde bütünlük kontrolü yöntemi ile binary patching işlemine karşı önlem alındığı görülmektedir. Bu kontrolün atlatılması, “.smali” kodlarının değiştirilmesi ile gerçekleştirilebilmektedir. Peki smali nedir?

Smali/Backsmali, assembler/dissassembler araçlarıdır. Yani? Android uygulamaları içerisinde bulunan “.dex” dosyaları insan gözü ile okunarak anlaşılamayacak kadar karmaşıktır. Backsmali aracı “.dex” dosyaları insanlar tarafından anlaşılabilir bir formatta dönüştürülebilmesini sağlanmaktadır. Bir diğer önemli konu ise, backsmali aracı ile elde edilen anlaşılır kodların değiştirilebilmesi ve smali aracı kullanılarak tekrar “.dex” formatına dönüştürülebilmesidir. Bu işlemin ismi smali injection olarak bilinmektedir.

Smali
injection işlemi için bizler “apktool” aracını kullanacağız. Apktool
smali dönüşümüne ek olarak, “.apk” içerisinde bulunan
“AndroidManifest.xml” vb. dosyaların da okunabilir formata
dönüştürülmesini sağlamaktadır.

**Not:** “.apk” dosya içeriğinin herhangi bir arşiv aracı kullanarak dışarı çıkartılması ile “AndroidManifest.xml” dosyası elde edilebilir. Elde edilen bu dosya herhangi bir text-editör yardımı ile açıldığında içeriği aşağıdaki şekilde görünmektedir.

![](04.png)

“apktool d UnCrackable-Level3.apk” komutu ile uygulama kodlarının decode/disassemble edilmesi gerçekleştirilmektedir (“d” parametresi decode/disassemble işlemi için kullanılmaktadır)

![](05.png)

İşleminin tamamlanmasının ardından, oluşan klasör içerisinde bulunan “smali/sg/vantagepoint/uncrackable3/MainActivity.smali” dosyası herhangi bir text-editör yardımı ile açılmaktadır.

![](06.png)

JD-GUI üzerinde tespit edilen kontrollerin “MainActivity.smali” dosyası üzerinden silinebilmesi için ilk olarak “onCreate” metodunun yeri tespit edilmelidir.

![](07.png)

Smali kodları içerisinde bu kontrolleri gerçekleştirildiği bölüm aşağıdaki şekildedir.

![](08.png)

Görüldüğü üzere “if-nez” şartlanma komutu ile “checkRoot1()Z”, “checkRoot2()Z” ve diğer metotların kontrol edildiği, tespit edilen bir durum sonucunda ise “cond_0” tag değerine sıçrama gerçekleştirilmektedir. Tespit edilen tüm kontrollere ait smali kodlarının silinmesi sonrasında elde edilen görüntü aşağıdaki şekildedir.

![](09.png)

Elde edilen yeni smali kodları, “apktool b UnCrackable-Level3” komutu kullanılarak “.apk” dosyasına dönüştürülür (İlgili “.apk” dosyası “UnCrackable-Level3/dist” dizini altında bulunmaktadır)

![](10.png)

Oluşturulan
“.apk” dosyası imzalanarak sistem üzerine yüklenmelidir. Aksi
taktirde mobil cihaz içerisinde çalıştırılmayacaktır. İmzalama işlemi iki
adımda gerçekleştirilmektedir.

1. İmzalama işleminde kullanılacak olan “keystore” dosyasının oluşturulması.
2. “Jarsign” aracı kullanılarak, oluşturulmuş olan “keystore” ile “.apk” dosyasının imzalanması.

“keytool -genkey -v -keystore newStoreName.keystore -alias aliasName -keyalg RSA -keysize 2048 -validity 10000” komutu kullanılarak “keystore” oluşturma işlemi gerçekleştirilmektedir.

![](11.png)

Daha sonra “jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore keyStoreName.keystore apkName.apk aliasName” komutu ile “.apk” dosyası imzalanmaktadır.

![](12.png)

Elde edilen “.apk” dosyası emulatör içerisine yüklenir ve uygulama çalıştırılarak güvenlik kontrollerinin kaldırıldığı görülür.

![](13.png)

Bir sonraki aşamada cihaz üzerinde çalışan uygulamaların listesi görüntülenmektedir.

![](14.png)

“owasp.mstg.uncrackable3”
paketine ait iki farklı PID değeri bulunduğu görülmektedir. Uygulama içerisinde
anti-debugging araçlarına karışı bu önlemin alındığı görülmektedir
(UnCrackable-Level2 uygulama çözümünde anti-debugging yöntemi ve bypass
edilmesine ilişkin detaylar bulunmaktadır)

Anti-Debugging kontrolünün gerçekleştirilmiş olduğu alanın tespit edilmesi için radare2 aracı ile “libfoo.so” kütüphanesi içerisinde bulunan ve JNI çağrısı ile dışarıdan çağırılan metotlar incelenir.

![](15.png)

“init” metodunun incelenmesi esnasında “fcn.00003250” fonksiyonunun çağırıldığı görülmektedir.

![](16.png)

“fcn.00003250” metodunun içeriği görüntülendiğinde ise “ptrace” sistem çağırısı kullanıldığı görülmektedir. “ptrace” kullanımı ile uygulamanın kendisine attach olduğu ve bu nedenle aynı paket adına ait iki farklı PID değeri ile karşılaştığımız anlaşılmaktadır.

![](17.png)

Tespit edilen kontrollerin, “0x0000325c” adresinde bulunan “call” metodu ile başladığı görülmektedir. Bu metot “call 0x3304” olacak şekilde güncellenir ve uygulamanın “ptrace” sistem çağrılarına uğramadan sonlanması sağlanılır.

![](18.png)

Daha sonra, güncellenen “libfoo.so” kütüphanesi emulatör içerisinde bulunan “/data/app/owasp.mstg.uncrackable3-1/lib/x86” dizini altı taşınır ve ardından uygulama çalıştırılır. Çalışan uygulamaların listesi görüntülendiğinde “owasp.mstg.uncrackable3” paketine ait tek PID değeri bulunduğu görülmektedir.

![](19.png)

Çözüme giden son adımda uygulama içerisinde bulunan gizli ifadenin tespiti gerçekleştirilecektir. İşe uygulama kaynak kodları incelenerek başlanır ve gizli ifade doğrulama işleminin “MainActivity” sınıfı içerisinde bulunan “verify” metodu ile gerçekleştirildiği tespit edilir.

![](20.png)

Metot içerisinde “CodeCheck” sınıfına ait “check_code” kontrolünün çağırıldığı görülmektedir. “CodeCheck” sınıfı incelendiğinde ise “check_code” metodunun içerisinde “bar” isimli bir metot çağırıldığı ve bu metodun native kütüphane içerisinde tanımlandığı anlaşılmaktadır.

![](21.png)

Bu bilginin ardından radare2 aracı ile native kütüphane – “libfoo.so” dosyası açılır ve içerisinde bulunan metotlar listelenir.

![](22.png)

“Java_sg_vantagepoint_uncrackable3_CodeCheck_bar” metoduna ait adres bilgisi bu şekilde elde edilir.

“s 0x000033b0” komutu ile imleç konumu “bar” metot adresine taşınır ve “VV” komutu ile metot içeriği görselleştirilir.

![](23.png)

Kullanıcıdan
alınan girdi ile uygulama içerisinde bulunan statik bir ifadenin
karşılaştırılması gerektiği bilinmektedir. Bu nedenle karşılaştırma
operatörleri dikkatli bir şekilde incelenir.

İlk olarak “0x00003432” adresinde bulunan kontrol incelenir.

![](24.png)

0x18 hexadecimal değeri ile “eax” kaydedicisi içerisinde bulunan değer karşılaştırılmaktadır. “eax” içerisinde bulunan değeri öğrenebilmek için GDB yardımı ile uygulama debug edilir.

![](25.png)

Öncelikle “gdb” içerisinde “info sharedlib” komutu çalıştırarak, native kütüphanenin yerleşmiş olduğu bellek adres bilgisi elde edilir.

![](26.png)

“0xaecb59e0” adresi, uygulamaya ait offset adres değerini de içermektedir. Offset değeri uygulamanın çalışmaya başlayacağı (entry point) değeri belirtmektedir. Radare2 aracı ile “ie” komutu kullanılarak uygulamanın entry point adresi elde edilebilir.

![](27.png)

Offset değeri”0x9e0″ olarak elde edilir. “0xaecb59e0” adres değerinden offset değerinin çıkartılması sonucunda uygulama için ayrılmış olan bellek alanının başlangıç değeri elde edilmektedir – “0xaecb5000”. Disassembler içerisinde tanımlanmış olan adres değerlerinin ofset değerini içermemesi nedeni ile adres dönüşüm işlemi gerçekleştirilmelidir.

Bu bilgilerin ardından “0x00003432” adresine break point yerleştirerek “eax” değerinin içeriğini elde edilir. Uygulama base adres değeri “0xaecb5000” olarak belirlenmişti, bu nedenle break point adresi “0xaecb8432” şeklinde dönüştürülür.

![](28.png)

Ara yüz üzerinde “1111” gibi rastgele bir ifade girerek uygulamanın break point noktasına yönlendirilmesi sağlanır.

![](29.png)

Amacımız “eax” kaydedicisinin içeriğini görüntülemek, bu nedenle “info registers” komutu ile kaydedicilere ait bilgilerin ekranda gösterilmesi sağlanılır.

![](30.png)

“eax” kaydedicisi içerisinde “4” decimal değerinin bulunduğu görülmektedir. Elde edilen değer, girmiş olduğumuz ifade uzunluğunu temsil etmektedir. Bu şekilde gizli ifadenin 0x18 hexadecimal (0x18 = 24 decimal) büyüklüğünde olması gerektiğini tespit ediyoruz.

Karakter boyutunun 24 olduğu durum sonrasında, uygulama “0x00003440” adresine yönlendirilmekte olup, “0x00003446” adresinde başka bir karşılaştırma metodu kullanıldığı görülmektedir. Karşılaştırma esnasında kullanılan kaydedici değerlerinin görüntülenebilmesi için GDB ile “0xaecb8440” adresine break point yerleştirilmektedir (Adres değerleri base adres üzerine eklenerek hesaplanmıştır)

![](31.png)

Daha sonra uygulama ara yüzü üzerinden 24 karakterden oluşan rastgele bir ifade girilir. Ardından, “0xaecb8440” adresinde bulunan break point noktasına erişim sağlanır ve “info register” komutu ile kaydedicilerin mevcut durumu görüntülenir.

![](32.png)

Radare2
üzerinden de görüleceği üzere “edx” kaydedicisine, “ecx”
kaydedicisi içerisinde bulunan adres değerinin barındırdığı ifade
aktarılmaktadır. Ardından “dl” kaydedicisi içerisinde bulunan ifade
ile “esp + eax” kaydedicileri içerisindeki adres değerlerinin toplamı
ile oluşan yeni adres değerinin içerdiği ifade “xor” işlemine tabi
tutulur ve sonuç “dl” kaydedicisi içerisine aktarılır (“dl”
kaydedicisi “edx” kaydedicisinin düşük değerli 8 bitlik bölümünü
içermektedir)

Son olarak da “dl” kaydedicisinin içerdiği değer ile, “esi + eax” kaydedicileri içerisinde adres değerlerinin toplanması ile oluşturulan adres değerinde bulunan ifadenin karşılaştırılması gerçekleştirilmektedir.

![](33.png)

“eax”
kaydedicisinin içeriğinin boş olduğu “info registers” komutu
sonucunda görülmektedir. Bu nedenle karşılaştırma işlemi “esp” ve
“esi” kaydedicileri içerisinde bulunan adreslerin içerdiği değerler
üzerinden gerçekleştirilmektedir.

“x/s 0xaecbb01c” komutu ile “ecx” kaydedicisi içerisinde bulunan adres değerinin içerdiği ifade ekranda gösterilmektedir.

![](34.png)

“x/s 0xbfb04450” komutu ile “esp” kaydedicisi içerisinde bulunan adres değerinin içerdiği ifade ekranda gösterilmektedir.

![](35.png)

Fakat elde edilen ifade çok anlamlı görünmemektedir. Bu nedenle “x/24x 0xbfb04450” komutu ile belirtilen adres üzerindeki ilk 24 değerin hexadecimal olarak ekranda görüntülenmesi sağlanmaktadır.

![](36.png)

Daha sonra “x/s 0xa380c08” komutu ile “esi” kaydedicisi içerisinde bulunan adres değerinin içerdiği ifade ekranda gösterilir ve kullanıcıdan alınan ifadeyi içerdiği tespit edilir.

![](37.png)

Uygulama tarafından gizli ifade doğrulamak için kullanılan tüm bileşenler bu şekilde elde edilmektedir. Geriye “esp” ile “dl” kaydedicileri içerisinden elde edilen değerlerin “xor” işleminde kullanılması ile sonucun elde edilmesi kalmıştır. Bu işlemin gerçekleştirilmesi için aşağıdaki python kodu geliştirilmiştir.

![](38.png)

Script çalıştırıldığında gizli ifadenin elde edildiği görülmektedir.

![](39.png)

Son olarak elde edilen değerin emulatör üzerinden doğruluğu test edilir.

![](40.png)
