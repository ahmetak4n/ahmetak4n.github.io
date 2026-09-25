---
title: OWASP Uncrackable Android Level 1
date: 2019-02-14 20:52:42 +0300
categories: [mobile]
tags: [android, frida, mobile, owasp uncrackable]
description: Owasp Uncrackable Android Level 1 uygulamasının incelenmesi ve içerdiği güvenlik önlemlerinin atlatılması
media_subpath: /assets/img/posts/android-uncrackable-level1-solution
image: cover.png
---

İlk olarak “adb install UnCrackable-Level-1.apk” komutu ile uygulamanın emülatör üzerine kurulumu gerçekleştirilir.

Ardından uygulama “.apk” dosyasının içeriği herhangi bir arşiv açma aracı kullanarak dışarı çıkartılır ve uygulamaya ait “.dex” dosyası elde edilir. Uygulama kodlarını JD-GUI decompiler aracı ile inceleyebilmek için “.dex” dosyalarını “.jar” formatına dönüştürülmesi gerekmektedir. Dönüşüm işlemi “d2j-dex2jar” aracı yardımı ile gerçekleştirilmektedir.

(**NOT:** JADX decompiler aracı “.dex” formatında bulunan dosyaları decompile edebilme yeteneğine sahiptir)

“.jar” dosyasını elde edilmesinin ardından JD-GUI aracı ile uygulama kaynak kodları aşağıdaki şekilde görüntülenebilir.

![](01.png)

Daha sonra emülatör üzerinde uygulama çalıştırılır ve “Root detected!” uyarısının alındığı görülür.

![](02.png)

Uygulama kaynak kodları incelendiğinde ilgili kontrolün “c.c()” sınıfı içerisinde gerçekleştirildiği tespit edilmektedir.

![](03.png)

Uygulamanın başlatılması ile birlikte “onCreate” metodu çalıştırılmakta ve “root” ile “debug” kontrolleri gerçekleştirilmektedir. Herhangi bir bulgu tespit edildiğinde ise “a()” metodu kullanılarak ekranda bir uyarı gösterilmekte ve uygulama “System.exit(0)” metodu kullanılarak sonlandırılmaktadır.

“c” sınıfı ve içerisinde tespit edilen root kontrolleri aşağıdaki şekildedir.

![](04.png)

Tespit edilen root kontrollerinin atlatılması için çok çeşitli yöntemler bulunmaktadır. Örneğin, “.smali” kodları üzerinden güvenlik kontrollerinin çağırıldığı alanların silinmesi ve uygulamanın yeniden paketlenerek yüklenmesi işlemi ile uygulama içerisinde bulunan güvenlik özellikleri atlatılabilmektedir.

Bu yazıda ise FRIDA aracı ile run-time code injection yöntemini kullanarak root kontrolü atlatılacaktır. Kod injection işlemi, “java.lang.System” sınıfına ait olan “exit” metodunu içerisinde yapılan işlemlerin değiştirilmesi şeklinde gerçekleştirilecektir. Peki neden bu metod seçildi? Uygulamanın çalışmasının ardından root kontrolleri gerçekleştirilmekte ve son aşama olarak “a()” metodu ile kullanıcıya hata mesaj ekranı gösterilmektedir. Kullanıcının ekranda bulunan uyarı butonuna tıklamasının ardından da “exit” metodu çalıştırılmaktadır. Yani? “exit” metodunun istenilen şekilde güncellenmesi sonucunda uygulama kapanmak zorunda kalmayacaktır.

Frida ile geliştirilen script aşağıdaki şekildedir.

![](05.png)

Kısa bir şekilde tekrar edecek olursak, “java.lang.System” sınıfına ait “exit” metodunun düzenleneceği “implementation” anahtar kelimesi ile belirtilir. Java dokümantasyonları incelendiğinde “exit” metodu içerisinde “Runtime().getRuntime().exit(n)” metodunun çağırıldığı görülmektedir. Biz ise bu metodu hiçbir işlem yapmadan tamamlanacak şekilde güncelleriz.

Daha sonra UnCrackable-Level1 uygulaması ve geliştirmiş olduğumuz script sırayla çalıştırılır.

Uygulama çözümüne ait ilk aşama böylelikle tamamlanmış olmaktadır. İkinci aşamada ise bizden uygulama içerisinde ki gizli ifadeyi tespit etmemiz istenmektedir. Kaynak kodları incelediğimizde “MainActivity” sınıfı içerisinde bulunan “verify” metodu kullanılarak kullanıcıdan parametre alma işleminin gerçekleştirildiği görülmektedir.

![](06.png)

Kullanıcıdan alınan parametrenin, uygulama içerisinde bulunan gizli ifade ile karşılaştırıldığı görülmektedir. Bu işlem için “a” sınıfı içerisinde bulunan “a” metodunun kullanılmaktadır.

![](07.png)

“verify” metodunun son satırına dikkat edildiğinde, kullanıcıdan alınan parametre ile sistem içerisinde yapılan işlemler sonucunda elde edilen değerin “equals” metodu kullanılarak karşılaştırıldığı görülecektir. Yani gizli ifade “equals” metodu içerisine parametre olarak aktarılmaktadır. Bu noktada “equals” metodu içerisine alınan parametreyi FRIDA aracılığı ile ekrana yazdırma işlemi gerçekleştirilecektir.

“root_detection_bypass.py” isimli FRIDA kodunu aşağıdaki şekilde güncelleriz.

![](08.png)

Görüldüğü üzere “java.lang.String” sınıfına ait “equals” metodu içerisine alınan parametrenin “send” metodu ile konsol ekranına yazdırılması işlemi tanımlanmıştır. (“root_detection_bypass.py” isimli script içerisinde tanımlı olan “on_message” metoduna parametre aktarmak için “send” metodu kullanılmaktadır ve bu sayede uygulama içerisinde okunan parametre değeri ekrana yazdırılmaktadır)

Son olarak ara yüz üzerinden rastgele bir değer girerek yazılmış olan script test edilir.

![](09.png)

“equals” metodunun farklı işlemler içinde kullanılmasından ötürü gizli ifade dışında da parametre değerleri elde edilmektedir. Fakat aranılan değerin bulması çok zor değildir, “**I want to believe**“. Elde edilen değerin ara yüz üzerinden de doğrulanması ile uygulama çözümü tamamlanmaktadır.

![](10.png)
