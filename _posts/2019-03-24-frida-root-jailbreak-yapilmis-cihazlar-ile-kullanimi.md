---
title: Frida with Root/Jailbreak
date: 2019-03-24 20:45:45 +0300
categories: [mobile]
tags: [android, frida, iOS, mobile]
description: Frida aracının root/jailbreak yapılmış cihazlar üzerinde nasıl kullanılabileceğinin incelenmesi
media_subpath: /assets/img/posts/frida-root-jailbreak-yapilmis-cihazlar-ile-kullanimi
image: cover.png
---

Frida
android, windows, IOS, macOS, GNU/Linux ve QNX platformları üzerinde çalışan
uygulamalara, javascript kodlarının enjekte edilmesini sağlayan bir araç olarak
tanımlanabilir.

Frida çekirdeği C dilinde yazılmıştır ve çalışma zamanında erişilmek istenilen hedef uygulama içerisine Google V8 javascript motorunu enjekte etmektedir. V8 motoru sayesinde kullanıcıdan alınan javascript kodları hedef uygulama/kütüphane üzerinde çalıştırılabilmektedir.

Buradaki
kritik nokta V8 javascript motorunun hedef uygulama içerisinde çalıştırılıyor
olmasıdır. Bu sayede hedef uygulama belleği üzerinde okuma ve yazma işlemleri
gerçekleştirilebilmektedir.

Frida, Server ve Client olmak üzere iki bileşenden oluşmaktadır. Server hedef cihaz üzerinde çalışan ve client üzerinden alınan javascript kodlarının hedef uygulama üzerine aktarılmasını sağlayan birimdir. Client ise javascript kodlarının server üzerine taşınmasını sağlayacak bileşen olarak tanımlanmaktadır. Python ve NodeJS dilleri içerisinde frida-client bileşeni kütüphane olarak bulunmaktadır ve geliştirilen javascript kodları bu diller aracılığı ile hedef uygulamaya aktarılabilmektedir.

![](01.png)

Örnek işlemlere geçilmeden önce python 3.x versiyonlarından birine sahip olunması gerekmektedir. Daha sonra “pip3 install frida-tools” komutu ile frida kütüphanesi python içerisine eklenilecektir.

Daha sonra konsol üzerinden “frida-ps” komutu ile mevcut cihaz üzerinde çalışmakta olan uygulamalar listelenebilmektedir.

![](02.png)

Artık frida aracı ile listelenen uygulamalara erişim sağlanabilmektedir. Buradaki önemli nokta, frida uygulamasını çalıştıran kullanıcı yetkisi dahilinde uygulama belleklerine erişim sağlanabileceğidir. Yani “owner” ve “group” değerleri root olan bir uygulamaya, standart kullanıcı yetkisi ile çalıştırılan frida aracı ile erişim sağlanamayacaktır. Yetki/Rol ayrımı özellikle mobil cihazlar üzerinde kritik bir önem göstermektedir. Mobil cihazlar üzerinde her bir uygulamanın ayrı bir kullanıcı olarak tanımlanması nedeniyle,

1. Frida uygulamasının root yetkisi ile çalıştırılması
2. Frida uygulamasının, hedef uygulamanın içerisinden çalıştırılması

Seçeneklerinden bir tanesi gerçekleştirilmelidir. Aksi taktirde uygulamaların belleklerine erişim sağlanamayacaktır.

Bu yazıda root yetkisi ile frida aracının Android ve IOS sistemler üzerinde çalıştırılması anlatılacaktır.

### android

Frida aracının direkt olarak yüklü bulunmadığı sistemler üzerinde işlem gerçekleştirebilmek için “frida-server” bileşeninin hedef sisteme kurulumu gerekmektedir. İlk olarak <https://github.com/frida/frida/releases> adresinden hedef sistem ile uyumlu server bileşeni indirilmektedir. Android x86 mimarisi üzerinde örnek işlemler gerçekleştirileceği için “frida-server-12.4.1-android-x86.xz” dosyası, kullanılan bilgisayar üzerine indirilir. Daha sonra elde edilen sıkıştırılmış dosya içerisindeki frida-server-12.4.1-android-x86 bileşeni çıkartılır ve aşağıdaki şekilde x86 mimarisinde çalışan android emulatörün “/data/local/tmp” dizinine taşınır.

![](03.png)

ADB aracılığı ile emulatör konsol ekranına erişim sağlanır ve frida-server bileşenin yüklenmiş olduğu dizine erişim sağlanır. “chmod 755 frida-server-12.4.1-android-x86” komutu ile uygulamaya çalışma izni tanımlanmaktadır.

![](04.png)

“./frida-server-12.4.1-android-x86 &” komutu ile frida-server çalıştırılır.

![](05.png)

“netstat -ant | grep LISTEN” komutu ile “27042” portunun açıldığı görülmektedir (27042 Frida-Server bileşeninin varsayılan port numarasıdır)

![](06.png)

Daha sonra local bilgisayar üzerinden “frida-ps -U” komutu ile “frida-server” bileşeninin üzerinde çalıştığı sisteme ait uygulama listesinin elde edildiği görülmektedir (“U” parametresi “USB” ile bağlı olunan sistem üzerine erişim sağlanacağını belirtmek için kullanılmaktadır)

![](07.png)

Örnek işlemlerde kullanılmak üzere Owasp “UnCrackable-Level1” uygulaması emulatör üzerine aşağıdaki şekilde yüklenmektedir.

![](08.png)

UnCrackable-Level1 uygulaması çalıştırılması sonrasında, “frida-ps -U” komutu ile uygulamanın paket ismi elde edilir.

![](09.png)

“frida -U owasp.mstg.uncrackable1” komutu ile ilgili uygulama işlemi içerisine V8 javascript motorunun eklenmesi sağlanır. Artık javascript kodları ile uygulama üzerinde işlem gerçekleştirilebilecektir.

![](10.png)

“frida” aracına ait CLI üzerinden aşağıdaki javascript kodu ile uygulamaya ait class isimlerinin listelenmesi işlemi gerçekleştirilebilmektedir.

- “Java.perform(function(){Java.enumerateLoadedClasses({“onMatch”:function(className){ console.log(className) },”onComplete”:function(){}})})”

![](11.png)

Gerçekleştirilen bu işlem, Python içerisinde bulunan “frida” kütüphanesi kullanılarak yazılan script yardımıyla da gerçekleştirilebilmektedir. Geliştirilen örnek python scripti aşağıdaki şekildedir.

![](12.png)

Yazılan script çalıştırıldığında aynı sonucun elde edildiği görülmektedir.

![](13.png)

Uygulama ve native-library içerisinde bulunan metotların değiştirilmesi, sınıfların listelenmesi, parametrelerin içeriklerinin okunması ve benzeri bir çok işlem bu şekilde gerçekleştirilebilmektedir.

### iOS

IOS cihazlar üzerinde gerçekleştirilen Jailbreak işlemi sonrasında da ilk olarak frida server bileşeninin cihaz üzerine kurulması gerekmektedir. Bunun için Cydia ara yüzünde bulunan “Kaynaklar” tabı içerisine yeni bir paket kaynağı tanımlamalıdır. Paket kaynak adresi olarak “<https://build.frida.re>” ifadesi girilerek aşağıdaki görüntü elde edilmektedir.

![](14.png)

Kaynak tanımlamasının ardından “Arama” tabına gelerek “Frida” anahtar kelimesi aratılır ve arama sonucunda bulunan paket cihaz üzerine indirilir.

![](15.png)

Tüm bu işlemler sonunda OpenSSH ile jailbreak yapılmış cihaza erişim sağlandığında “27042” portunun açılmış olduğu görülmektedir.

![](16.png)

Bir önceki örnekte, android cihaz ile kullanılan bilgisayarın USB kablo ile bağlanması sonucunda frida CLI ve python kütüphanesi üzerinden frida-server ile iletişim kurulması işlemi gerçekleştirilmişti. Aynı işlem IOS cihazın USB kablosu ile kullanılan bilgisayarın bağlanması sonucunda da gerçekleştirilebilmektedir.

Bu yönteme
ek olarak, SSHTunnel yöntemi ile frida-server kurulu cihaz üzerinde javascript
komutları çalıştırılması gerçekleştirilebilmektedir (Bu yöntemde hem android
hemde IOS platformları için geçerlidir)

SSHTunnel yöntemi “ssh -L <local_port>:<remote_interface>:<remote_port> <remote_address_user>@<remote_ip>” komutu ile aşağıdaki şekilde gerçekleştirilmektedir.

![](17.png)

Bağlantının başlatılmış olduğu bilgisayar üzerinde bulunan portlar kontrol edildiğinde, SSHTunnel işleminin başarılı bir şekilde gerçekleştirildiği görülmektedir.

![](18.png)

Artık frida ile cihaz üzerinde çalışan uygulamaların listelenmesi sağlanabilir. Bunun için “frida-ps -R” komutu kullanılmaktadır (“R” – Remote, SSHTunnel üzerinden gerçekleştirilen bağlantı türlerinde kullanılmaktadır)

![](19.png)

Örnek olarak “BlueTool” isimli uygulamaya frida ile bağlantı denemesi aşağıdaki şekilde gerçekleştirilmektedir.

![](20.png)

“frida”
aracına ait CLI üzerinden aşağıdaki javascript kodu girilerek uygulamaya ait
class isimlerinin listelenmesi işlemi gerçekleştirilebilmektedir.

- “for(var className in ObjC.classes){if (ObjC.classes.hasOwnProperty(className)){console.log(className);} }”

![](21.png)

Benzer şekilde aşağıdaki python scripti ile uygulama içerisinde kullanılan class isimlerinin listelenmesi gerçekleştirilebilmektedir.

![](22.png)

Geliştirilen script ile uygulamaya ait sınıf isimlerinin listelenebildiği görülmektedir.

![](23.png)
