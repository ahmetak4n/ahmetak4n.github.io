---
title: OWASP Uncrackable iOS – Level1
date: 2019-03-26 19:58:58 +0300
categories: [mobile]
tags: [android, iOS, mobile, owasp uncrackable]
description: Owasp Uncrackable IOS Level 1 uygulamasının incelenmesi ve içerdiği güvenlik önlemlerinin atlatılması
media_subpath: /assets/img/posts/ios-uncrackable-level1-solution
image: cover.png
---

Uygulama analizine başlanılabilmesi için “UnCrackable_Level1.ipa” dosyasının hedef cihaz üzerine yüklenmesi gerekmektedir. Bu işlem;

- UnCrackable-Level1 uygulamasının imzalanması
- İmzalanan uygulamanın “Cydia Impactor” aracı ile hedef sisteme yüklenmesi

adımlarından oluşmaktadır.

**Not:** Testlerin gerçekleştirilebilmesi için
Jailbreak yapılmış IOS cihaz ve macbook gereksinimi bulunmaktadır.

İlk olarak “UnCrackable_Level1.ipa” içerisinde bulunan dosyalar, herhangi bir arşiv – zip aracı kullanılarak dışarı çıkartılmaktadır.

![](01.png)

İmzalama işlemine başlanmasından önce mevcut sertifika bilgisinin silinmesi gerekmektedir. Bu işlem

- rm -r Payload/UnCrackable\ Level\ 1.app/_CodeSignature/

komutu ile gerçekleştirilmektedir.

![](02.png)

İmzalama işleminin gerçekleştirilebilmesi için gerekli olan bir diğer adımda kullanıcı sertifika bilgisinin elde edilmesidir. Apple, imzalama işlemlerinde kullanılmakta olan iki sertifika türü barındırmaktadır. Bunlar;

- “Developer” : Belirli bir ücret ödenmesi sonucunda Apple Developer sistemi üzerinden elde edilen sertifika türü
- “Regular”: Apple icloud üzerinde bulunan ücretsiz hesaplara ait sertifika türü

şeklindedir. Xcode üzerinde ücretsiz icloud hesabı ile giriş yaparak “Regular” sertifika sahibi olunabilmektedir.

![](03.png)

Sahip olunan
sertifika bilgilerine konsol üzerinden;

- security find-identity

komutu ile erişim sağlanabilmektedir.

![](04.png)

Elde edilen sertifika bilgisi ve “codesign” aracı kullanılarak imzalama işlemi tamamlanılabilmektedir (“codesign” Xcode içerisinde uygulamaların imzalanması için kullanılan araçtır)

Bu işlem “codesign -f -s 170**************** Payload/UnCrackable\ Level\ 1.app”

![](05.png)

İmza bilgisi
güncellenmiş olan uygulamanın tekrar “.ipa” formatına dönüştürülmesi
için aşağıdaki komut kullanılmaktadır.

- “zip -qr $output_file $input_file”

![](06.png)

Son adım olarak “Cydia Impactor” aracı kullanılarak “resigned.ipa” dosyasının test cihazımız üzerine taşınması gerçekleştirilmektedir (“Cydia Impactor” aracı üzerine “resigned.ipa” dosyasının sürüklenmesi ile yükleme işlemi başlamaktadır)

![](07.png)

Başarılı bir şekilde uygulamanın yüklendiği görülmektedir.

![](08.png)

**Not:** Jailbreak yapılmış olan cihaz üzerinde frida aracının kurulu olması gerekmektedir. [Frida – Root/Jailbreak Yapılmış Cihazlar Üzerinde Kullanımı](/frida-root-jailbreak-yapilmis-cihazlar-ile-kullanimi/) isimli yazı içerisinde frida aracının kurulum adımlarını inceleyebilirsiniz.

“frida-ps -U” komutu ile USB üzerinden bağlı olunan IOS cihaz içerisinde çalışmakta olan uygulamaların listelenebildiği görülmektedir.

![](09.png)

UnCrackable-Level1 uygulamasının çalıştırılması sonucunda “A Secret is Found in The Hidden Label!” şeklinde bir mesajın ekranda bulunduğu görülmektedir.

![](10.png)

Verilen ipucu doğrultusunda aktif olan ekran üzerinde bulunan “hidden” label ögelerinin içeriklerinin incelenmesi gerçekleştirilmektedir. Bu işlem frida aracı kullanılarak gerçekleştirilecektir.

İlk olarak frida ile aktif olan ekran objesine erişim sağlanması gerekmektedir. Bu işlem “UIWindow.keyWindow()” komutu ile gerçekleştirilmektedir. IOS ekran sisteminde sadece bir adet “keyWindow” objesi bulunmaktadır ve mevcut olarak kullanılan ekranı temsil etmektedir.

Bu bilgiler doğrultusunda geliştirilen frida scripti aşağıdaki şekildedir.

![](11.png)

Script çalışması sonucunda “hidden” özelliği aktif olan label nesnelerinin listelendiği görülmektedir.

![](12.png)

İkinci satır dikkatli incelendiğinde gizli ifadenin saklandığı label nesnesi tespit edilebilmektedir – “i am groot!”

![](13.png)

Elde edilen gizli ifade ara yüz üzerinden girilerek doğrulanmaktadır.

![](14.png)
