---
title: Frida without Root/Jailbreak
date: 2019-03-24 16:19:00 +0300
categories: [mobile]
tags: [android, frida, iOS, mobile]
description: Frida aracının root/jailbreak olmayan cihazlar üzerinde nasıl kullanılabileceğinin incelenmesi
media_subpath: /assets/img/posts/frida-root-jailbreak-yapilmamis-cihazlar-uzerinde-kullanimi
image: cover.png
---

Frida hakkında genel bilgilere, [Frid](/frida-root-jailbreak-yapilmis-cihazlar-ile-kullanimi/)[a – Root/Jailbreak Yapılmamış Cihazlar Üzerinde Kullanımı](/frida-root-jailbreak-yapilmis-cihazlar-ile-kullanimi/) isimli yazı içerisinden erişebilirsiniz.

### android

Frida aracı ile, sistem üzerinde bulunan diğer uygulama bellek alanlarına ROOT yetkisine sahip olmadan erişilebildiğinden bahsedilmişti. Bu işlemin gerçekleştirilebilmesi için;

1. Hedef uygulamanın smali formatına dönüştürülmesi
2. “frida-gadget” kütüphanesinin hedef uygulama içerisine eklenmesi
3. “frida-gadget” kütüphanesinin çalıştırılmasını sağlayacak kodların, uygulama smali kodları içerisine eklenmesi
4. Manifest dosyası içerisine “<uses-permission android:name=”android.permission.INTERNET” />” izninin tanımlanması (frida-gadget tarafından soket açılabilmesi için bu izin gerekmektedir)
5. Gerekli değişikliklerin yapılmasının ardından, uygulama dosları kullanılarak yeni “.apk” dosyasının oluşturulması ve imzalanması

adımlarının
uygulanması gerekmektedir.

Hedef uygulamanın bir parçası olması, frida
aracının uygulama belleği üzerinde okuma ve yazma işlemlerini
gerçekleştirebilmesine olanak sağlamaktadır.

İlk olarak <https://github.com/frida/frida/releases> adresinden hedef sistem mimarisine uygun olan frida-gadget aracı indirilerek işlemlere başlanır (“frida-gadget-12.4.3-android-arm64.so.xz” üzerinden işlemlerin anlatılması gerçekleştirilecektir)

![](01.png)

**NOT:** “arm64” ile “aarch64” aynı mimariyi temsil etmektedir.

Örnek işlemler Owasp UnCrackable-Level2 uygulaması üzerinde gerçekleştirilecektir. Bu nedenle “apktool” aracı ile bu uygulamanın smali kodlarına dönüştürülmesi gerekmektedir.

![](02.png)

Daha sonra, elde edilen uygulama klasörü içerisinde bulunan “/lib/arm64-v8a” dizinine erişim sağlanır. “frida-gadget-12.4.3-android-arm64.so.xz” sıkıştırılmış dosyası içerisinden çıkartılan “frida-gadget-12.4.1-android-arm64.so” dosyası bu dizine taşınır. Buradaki önemli nokta, “frida-gadget-12.4.1-android-arm64.so” isminin “libfrida-gadget.so” şeklinde güncellenmesi gerektiğidir (JNI yapısı gereği “libfrida-gadget.so” şeklinde tanımlanmış bir kütüphaneye, uygulama içerisinden “frida-gadget” anahtar kelimesi ile erişim sağlanabilmektedir)

![](03.png)

Bir
sonraki adımda smali kodları içerisine ‘System.loadLibrary(“frida-gadget”)’ ifadesinin enjekte edilmesi gerekmektedir. Bu işlem için
“smali/sg/vantagepoint/uncrackable2” dizini altında bulunan
“MainActivity” sınıfına erişim sağlanır.

Uygulama içerisinde hali hazırda native-library kullanımı bulunduğu görülmektedir. Bu işleme benzer olarak “frida-gadget” kütüphanesinin çağırılmasına ilişkin smali kodları eklenmektedir.

Native-library tanımlaması ve kullanılacak metot tanımlamalarına ilişkin smali kodları aşağıdaki şekildedir.

![](04.png)

![](05.png)

“frida-gadget” kütüphanesinin smali kodları içerisine eklenilmesi sonucunda elde edilen görüntü aşağıdaki şekildedir.

![](06.png)

Sırada “AndroidManifest.xml” dosyası içerisine “<uses-permission android:name=”android.permission.INTERNET” />” izninin eklenmesi işlemi bulunmaktadır. İşlem sonrasında “AndroidManifest.xml” dosyasının görüntüsü aşağıdaki şekilde olacaktır (“frida-gadget” ile “frida-client” arasında bağlantı kurma işlemi için bu izin bilgisinin tanımlanması gerekmektedir)

![](07.png)

Daha sonra uygulamanın yeniden paketlenmesi işlemi bulunmaktadır. “apktool” aracı ile bu işlem aşağıdaki şekilde gerçekleştirilmektedir.

![](08.png)

Son adım olarak

- “keytool -genkey -v -keystore newStoreName.keystore -alias aliasName -keyalg RSA -keysize 2048 -validity 10000”

komutu ile keystore oluşturulur.

![](09.png)

Oluşturulan keystore ile

- “jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore keyStoreName.keystore apkName.apk aliasName

komutu kullanılarak uygulamanın paketlenmesi işlemi tamamlanmış olur.

![](10.png)

Elde edilen “.apk” dosyası aşağıdaki şekilde hedef sistem üzerine yüklenir.

Sistem üzerine yüklenilen uygulamanın çalıştırılması ile birlikte “27042” portunun sistem üzerinde açıldığı görülmektedir.

![](11.png)

Hedef sistemin USB ile bağlı olduğu bilgisayar üzerinde frida aracı kullanılarak, hedef cihaz üzerinde bulunan uygulamaların listelenebildiği görülmektedir. Bu işlem sonucunda “Gadget” isimli bir uygulamanın bulunduğu tespit edilmektedir.

“frida -U Gadget” komutu kullanılarak frida ile hedef sistem arasında bağlantı kurulması gerçekleştirilir.

Kurulan bağlantı sonrasında gönderilen javascript kodlarının hedef sistem üzerinde çalıştırıldığı görülmektedir.

- Java.perform(function(){Java.enumerateLoadedClasses({“onMatch”:function(className){ console.log(className) },”onComplete”:function(){}})})

![](12.png)

### ios

**NOT:** Anlatılan işlemlerin gerçekleştirilebilmesi için Xcode aracının sisteminizde kurulu olması ve Jailbreak yapılmamış IOS cihazınız bulunması gerekmektedir.

JAILBREAK yapılmamış cihaz üzerinde bulunan hedef uygulama belleğine frida aracı ile erişim sağlayabilmek için, hedef uygulamaya ait “.ipa” dosyası üzerinde aşağıdaki işlem adımları gerçekleştirilmektedir.

1. Sıkıştırılmış formatının açılması (“.ipa” bir tür sıkıştırılmış – zip dosya şeklidir)
2. Frida-gadget kütüphanesinin hedef uygulama içerisine eklenilmesi
3. Uygulama kodları içerisinden frida-gadget kütüphanesinin çağırılması
4. Değiştirilmiş uygulama dosyalarının yeniden sıkıştırılması ve imzalanması

Elde edilen yeni “.ipa” dosyasının hedef sistem üzerinde çalıştırılması ile hedef uygulama belleğine frida aracı ile erişim sağlanabilecektir.

Örnek olarak Owasp “UnCrackable-Level1.ipa” uygulaması kullanılacaktır. İlk olarak UnCrackable-Level1.ipa uygulamasına ait dosyalar herhangi bir arşiv açma aracı yardımı ile “.ipa” formatı içerisinden çıkartılır.

![](13.png)

Daha sonra <https://github.com/frida/frida/releases> adresinden “frida-gadget-12.4.3-ios-universal.dylib.xz” arşiv dosyası indirilir ve içerisinde bulunan “frida-gadget-12.4.3-ios-universal.dylib” dosyası dışarı çıkartılır.

![](14.png)

Dışarı çıkartılmış olan “frida-gadget-12.4.3-ios-universal.dylib” dosyasının ismi “FridaGadget.dylib” olacak şekilde güncellenmektedir.

![](15.png)

Bu noktaya kadar gerçekleştirilen işlemlerin hepsi Android üzerinde gerçekleştirilen işlemler ile aynıdır. “frida-gadget” kütüphanesinin hedef uygulama içerisine eklenmesi ve uygulama içerisinden çağırılması işlemleri Android üzerinde gerçekleştirilen işlemlerden tamamen farklıdır.

IOS ile
Android uygulamaları farklı binary yapılarına sahiptir. Android uygulamaları
ELF (Executable and Linkable Format) yapısına sahipken, IOS uygulamaları Mach-O
(Match Object) yapısına sahiptir. Peki neden, android uygulaması üzerine
“frida-gadget” kütüphanesi eklenilirken ELF yapısı hakkında bir
bilgiye ihtiyaç duyulmadı? Bunun nedeni “smali” kodları üzerinde
değişiklik yapılmış olmasıdır.

SDK
kullanılarak Android uygulamaları Java dilinde geliştirilebilmektedir ve
derlenen java sınıfları, uygulama “.apk” dosyası içerisinde
“.dex – Dalvik Executable” dosya formatında saklanmaktadır. Bir
uygulamanın Android cihaz içerisine yüklenmesi esnasında ise (Android 4.4 –
KitKat sonrasında) “.dex” dosyaları ELF formatına dönüştürülür ve bu
formatta çalıştırılır. “.dex” dosyalarının
“smali/backsmali” disassembler araçları yardımı ile okunabilir formata
dönüştürülmesi sonucunda, hedef uygulama içerisinden “frida-gadget”
kütüphanesi çağırılabilmektedir.

Fakat IOS uygulamaları üzerinde “smali” benzeri bir ara dil (intermediate language) bulunmamaktadır. Bu nedenle gerekli değişikliklerin gerçekleştirilebilmesi için Mach-O yapısı hakkında fikir sahibi olunması gerekmektedir. Mach-O, aşağıdaki bölümlerden oluşmaktadır.

![](16.png)

“Header” bölümü CPU tipi, binary boyutu ve benzeri genel bilgileri içermektedir. “Load Commands” bölümü binary içeriğin konumlarını barındıran bir tablo olarak düşünülebilir. Her bir segment konumları bu alanlar içerisinde saklanmaktadır. Ayrıca her bir segment içerisinde bulunan komutlara ilişkin tip, isim ve konum gibi bilgilerde bu alan içerisinde bulunmaktadır. “Data” bölümü ise uygulama kodları ve verilenin saklandığı alan olarak tanımlanmaktadır.

Bu bilgiler
doğrultusunda, uygulama içerisine yeni bir kütüphane ekleme işlemi;

1. Kütüphanenin uygulama yığını içerisine eklenmesi
2. “Load Commands” alanına, yeni eklenen kütüphanenin çağırılmasını sağlayacak olan binary ifadenin eklenmesi
3. “Header” bölümü içerisinde bulunan “Load Commands” sayacının bir arttırılması
4. “Header” bölümü içerisinde bulunan binary büyüklük değerinin arttırılması

adımlarından oluşmaktadır.

IOS
uygulamalarının düzenlenmesi esnasında karşılaşılan bir diğer zorluk ise imza
sirkülasyonudur. IOS uygulamaları içerisinde bulunan “provisioning
profile” ve “code signature header” alanlarında bulunabilecek
bir uyuşmazlık ilgili uygulamanın çalışmamasına neden olmaktadır. Bu nedenle
eklenilen kütüphane sonrasında “provisioning profile” ve “code
signature header” alanlarının da düzenlenmesi gerekmektedir.

“Provisioning
profile” Apple tarafından imzalanmış ve imzalayan bilgilerini içeren bir
“.plist” dosyasıdır. Bu dosya sayesinde Apple tarafından
geliştiriciye ait uygulamaların takibi yapılabilmektedir. İki farklı
“Provisioning profile” türü bulunmaktadır. Bunlar
“development” ve “regular” şeklindedir.
“Development”, Apple geliştirici sistemine belirli bir ücret ile
kayıt olunarak elde edilebilmektedir. “Regular” ise ücretsiz
oluşturulabilen Apple icloud hesabı kullanılarak elde edilebilir.
Gerçekleştirilecek olan örnek “regular” “provisioning
profile” kullanılarak gerçekleştirilecektir.

Elde edilen
bilgiler doğrultusunda, UnCrackable-Level1.ipa uygulaması içerisine
“frida-gadget” aracının eklenmesi işlemlerine başlanılabilir.

İlk olarak Xcode üzerinden “uncrackable_repackage” isimli yeni bir uygulama oluşturulur.

![](17.png)

Elimizde bulunan IOS cihazın USB ile kullanılan bilgisayar üzerine bağlanması gerçekleştirilir. Daha sonra Xcode üzerinde oluşturulan yeni projenin USB ile bağlı olan cihaz üzerinde çalıştırılması sağlanır.

![](18.png)

Uygulamanın derlenmesi ile “mobilprovision” dosyasının Xcode tarafından üretildiği görülmektedir.

![](19.png)

“cp embedded.mobilprovision ~/Desktop/UnCrackable-Level1” komutu ile “mobilprovision” dosyası hedef “.ipa” dosyası ile aynı dizin içerisine taşınmaktadır.

![](20.png)

Elde edilen “embedded.mobilprovision -(provisioning profile)” dosyası içerisinde bulunan sertifika, cihaz ve uygulama yetkileri gibi bilgilerin düzgün bir formatta elde edilmesi gerekmektedir. Bunun için ilk olarak; “security” aracı ile

- “security cms -D -i embedded.mobilprovision > profile.plist”

komutu kullanılarak kriptografik bilgilerin decode edilerek “profile.plist” dosyası içerisine yazılması sağlanmaktadır (“-D” flag değeri “decode”, “-i” flag değeri interaktif-shell özelliklerinin kullanılacağını belirtmektedir)

![](21.png)

Daha sonra “PlistBuddy” aracı kullanılarak “profile.plist” içerisinde bulunan “Entitlements” bölümü “entitlements.plist” dosyası içerisine kopyalanmaktadır. Bu işlem

- “/usr/libexec/PlistBuddy -x -c ‘Print :Entitlements’ profile.plist > entitlements.plist” komutu ile gerçekleştirilmektedir.

![](22.png)

“FridaGadget.dylib” kütüphanesinin uygulama içerisine eklenmesi işlemi için “[optool](https://github.com/alexzielenski/optool.git)” isimli bir araç kullanılacaktır. Araç kurulumu aşağıdaki şekilde gerçekleştirilebilmektedir.

![](23.png)

**Not:** “build/Release/optool” dizini altında uygulamanın çalıştırılabilir dosyası bulunmaktadır.

“cp FridaGadget.dylib Payload/UnCrackable\ Level \ 1.app” komutu ile frida kütüphanesi hedef uygulama içerisine taşınır.

![](24.png)

Bir sonraki
adımda, uygulama içerisinde frida kütüphanesinin çağırılmasını sağlayacak olan
işlemler “optool” aracı ile aşağıdaki şekilde gerçekleştirilir.

- “optool install -c load -p “@executable_path/FridaGadget.dylib” -t Payload/UnCrackable \Level \1.app/UnCrackable\ Level\ 1″

![](25.png)

Uygulama
içerisine frida kütüphanesinin eklenmesi işlemi bu şekilde tanımlanmış
bulunmaktadır. Bundan sonraki adımların hepsi imza sirkülasyonunu tamamlamak
amacı ile gerçekleştirilecektir.

İlk olarak oluşturmuş olduğumuz uygulama içerisinden alınan “embedded.mobilprovision” dosyasının hedef uygulama içerisine taşınması gerçekleştirilir.

- “cp embedded.mobilprovision Payload/Uncrackable\ Level\ 1.app/embedded.mobilprovision”

![](26.png)

Daha sonra hedef uygulama içerisinde bulunan “Info.plist” dosyası üzerinde, yeni kullanıcı profiline ait bilgilerin güncellenmesi işlemi

- “/usr/libexec/PlistBuddy -c “Set :CFBundleIdentifier Aora.uncrackable-repackage” Payload/UnCrackable\ Level\ 1.app/Info.plist”

komutu gerçekleştirilmektedir.

![](27.png)

Burada dikkat edilmesi gereken nokta “-c” parametresi ile aktarılan değerdir. Hedef uygulama içerisine, Xcode üzerinde oluşturulan uygulamaya ait “mobilprovision” dosyası aktarılmıştı. Bu nedenle “Info.plist” içerisinde bulunan bilgilerde Xcode üzerinde oluşturulan uygulama ile uyumlu olacak şekilde güncellenmelidir. Bu nedenle “CBundleIdentifier” değeri, oluşturulan uygulama içerisindeki “Bundle Identifier” olacak şekilde belirlenmiştir.

![](28.png)

Sıradaki işlemde hedef uygulama içerisinde bulunan “_CodeSignature” klasörü silinerek uygulamaya ait olan orijinal imza bilgileri temizlenmektedir.

![](29.png)

Orijinal imza bilgilerinin silinmesinin ardından hedef uygulamanın imzalanması işlemi gerçekleştirilebilmektedir. Bu işlem için “codesign” isimli araç kullanılmaktadır. Xcode tarafından gerçekleştirilen imzalama işlemlerinde de “codesign” aracı kullanılmaktadır.

İmzalama işlemi için “mobilprovisioning” profili içerisinde bulunan “signing identity” bilgisine ihtiyaç duyulmaktadır. Bu bilgi konsol üzerinden

- “security find-identity -v”

komutu çalıştırılarak öğrenilebilmektedir.

![](30.png)

“Signing
Identity” bilgisinin elde edilmesi sonrasında aşağıdaki komut kullanılarak
“FridaGadget.dylib” kütüphanesinin imzalanması işlemi
gerçekleştirilmektedir.

- “codesign -f -s 170***************** Payload/UnCrackable\ Level\ 1.app/FridaGadget.dylib”

![](31.png)

Son olarak oluşturmuş olduğumuz uygulama mobilprovision dosyası içerisinden alınan “entitlements” bilgileri kullanılarak uygulamanın tamamının imzalanması gerçekleştirilecektir.

- “codesign -f -s 170***************** –entitlements entitlements.plist Payload/UnCrackable\ Level\ 1.app/UnCrackable\ Level\ 1”

![](32.png)

Artık uygulama hedef sistem üzerine yüklenmeye hazırdır. Bu işlem için “ios-deploy” isimli bir araç kullanılacaktır. Araç kurulumu aşağıdaki şekilde gerçekleştirilmektedir.

![](33.png)

**Not:** “build/Release” dizini altında
uygulamanın çalıştırılabilir dosyası bulunmaktadır.

“ios-deploy” aracının kurulmasının ardından aşağıdaki komut ile USB bağlantısı üzerinden hedef cihaz üzerine güncellenmiş “UnCrackable-Level1.ipa” uygulamasının yüklenmesi gerçekleştirilmektedir.

- “ios-deploy –debug –bundle Payload/UnCrackable\ Level\ 1.app/”

![](34.png)

Görüleceği üzere uygulamanın debug modu açık bir şekilde hedef cihaz üzerinde çalıştırılması gerçekleştirilmiştir. Lokal sistem üzerinde bulunan frida aracı ile “frida-ps -U” komutu kullanılarak cihaz üzerinde bulunan “Gadget” kütüphanesinin çalışmakta olduğu doğrulanmaktadır.

Daha sonra “frida -U Gadget” komutu ile frida CLI kullanımı gerçekleştirilebilmektedir.

“frida” aracına ait CLI üzerinden aşağıdaki javascript kodu girilerek uygulamaya ait class isimlerinin listelenmesi gerçekleştirilebilmektedir.

- “for(var className in ObjC.classes){if (ObjC.classes.hasOwnProperty(className)){console.log(className);} }”

![](35.png)

Bu işlemler sonucunda, ROOT kullanıcısına sahip olunmasada frida aracı ile çalışma zamanlı kod enjeksiyonu gerçekleştirilebildiği doğrulanmaktadır.
