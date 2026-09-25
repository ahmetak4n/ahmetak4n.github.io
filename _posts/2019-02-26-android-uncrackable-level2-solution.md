---
title: OWASP Uncrackable Android Level 2
date: 2019-02-26 10:34:26 +0300
categories: [mobile]
tags: [android, frida, mobile, owasp uncrackable]
description: Owasp Uncrackable Android Level 2 uygulamasının incelenmesi ve içerdiği güvenlik önlemlerinin atlatılması.
media_subpath: /assets/img/posts/android-uncrackable-level2-solution
image: cover.png
---

İlk olarak uygulamanın emülatör üzerine kurulumu ve “.apk” dosyasının decompile edilmesi gerçekleştirilir. Bu işlemlerin detayları [UnCrackable-Level1](/android-uncrackable-level1-solution/) içerisinde anlatılmaktadır.

Daha sonra uygulamanın kaynak kodları elde edilerek incelenmeye başlanır. UnCrackable-Level1 uygulaması ile kıyaslandığında, tek farkının native kütüphane kullanımı olduğu görülmektedir.

![](01.png)

Daha sonra mobil cihaz içerisinde UnCrackable-Level2 uygulaması çalıştırılır ve çalışan uygulamaların PID değerleri görüntülenir.

![](02.png)

Bu noktada alışılmışın dışında bir durum ile karşılaşılır. Aynı paket adına ait iki farklı PID değeri bulunmaktadır. Daha sonra ilk PID değeri kullanılarak frida scripti ile uygulamaya attach olma denemesi gerçekleştirilir.

![](03.png)

Deneme sonucunda aşağıdaki hata mesajının alındığı görülmektedir.

![](04.png)

İkinci denemede, frida scripti içerisinde bulunan PID değeri “4720” olarak değiştirilir ve “System.exit” metodu içeriği güncellenir.

![](05.png)

Yazılan yeni script çalıştırıldığında herhangi bir hata mesajı alınmamaktadır, fakat script içerisinde tanımlanan işlemlerin uygulama çalışmasını etkilemediği tespit edilmiştir (Ekranda beliren butona tıklanması ile uygulamanın kapandığı görülmektedir)

Kaynak kodları incelediğimizde olağan dışı herhangi bir kontrol bulunmadığı görülmektedir. Geriye sadece native kütüphane içerisinde bulunan “init” metodunu incelemek kalmaktadır. Peki bu işlem nasıl gerçekleştirilir? Cevap: Disassembler araçları yardımı ile.

Disassembler araçları ile makine kodlarının
assembly koduna dönüştürülmesi mümkün olabilmektedir. Bu sayede uygulamanın
işleyişi hakkında daha kolay bilgi sahibi olunabilir ve üzerinde değişiklik gerçekleştirilebilir.
Radare2 ve Ida64 yaygın olarak kullanılan iki farklı disassembler aracıdır.
Google üzerinde daha farklı bir çok araç bulabilirsiniz fakat biz işlemlerimizi
radare2 kullanarak gerçekleştireceğiz.

İşlemlerimize başlamadan makine dili ile ilgili bir bilgiyi hatırlamamız gerekmektedir. C/C++ gibi dillerin derlenmesi ile oluşan çalıştırılabilir dosyalar makine kodlarına güzel birer örnektir. Burada gerçekleştirilen derleme işlemi mevcut işlemci mimarisine ve ABI (Application Binary Interface) türüne bağlı olarak gerçekleştirilmektedir. Çünkü içerisinde bulunan tüm işlemler, işlemci seviyesinde gerçekleştirilecek şekilde tanımlanmıştır. Bu nedenle x86 mimarisinde derlenmiş olan bir C/C++ (veya başka bir dil ile yazılmış uygulamanın) çalıştırılabilir dosyası, x64 veya arm64 mimarisinde çalıştırılamamaktadır.

Bu bilginin ardında android emulatör tarafından kullanılan mimari bilgisini öğrenmemiz gerekmektedir. Bunun için ayarlar ara yüzü içerisinden telefon bilgileri bölümüne gelerek kullanılmakta olan mimari bilgisi elde edilebilir.

![](06.png)

Cihazın “x86” mimarisine sahip olduğunu görülmektedir. Daha sonra “.apk” içerisinde bulunan dosyaların çıkartılmış olduğu dizine erişim sağlanır.

![](07.png)

“/lib” klasörü altında farklı türden mimarilere ait klasörler bulunduğu görülmektedir.

![](08.png)

“x86” mimarisi üzerinde işlemlerimizi gerçekleştireceğimiz için “x86” dizini altına erişim sağlayarak “libfoo.so” dosyasını elde ederiz.

![](09.png)

**Not:** Günümüzde android
işletim sistemi bir çok farklı firma ve işlemci mimarisi üzerinde
çalışmaktadır. Bu nedenle farklı mimarilere uyumlu native kütüphaneler uygulama
paketi ile içerisinde bulunmaktadır ve mevcut sistem mimarisine uygun olan
yükleme işlemi esnasında seçilerek kullanılmaktadır.

Elde edilen native kütüphane dosyası “radare2 libfoo.so” komutu kullanılarak açılmaktadır.

Bu işlemin ardından “aaa” komutu ile radare2 aracının native kütüphaneyi analiz etmesi sağlanır (Uygulama içerisindeki fonksiyon, string ve benzeri önemli bilgilerin düzenlenerek kullanıcıya gösterilmesi işlemini gerçekleştirir)

![](10.png)

(Soldaki ekranda kodların düzenlenerek
gösterilmiş hali bulunmaktadır. Görüleceği üzere metotlar birbirinden ayrılmış
bir şekilde ekranda gösterilmektedir)

Kaynak kod üzerinden kütüphaneye ait “init” metodunun çağırıldığı tespit edilmişti. Native kütüphane içerisinde bir metodun dışardan erişilebilir olması için o metodun export edilmesi gerekmektedir. Radare2 içerisinde, export edilmiş olan metotlar “iE” komutu ile tespit edilebilir.

![](11.png)

**NOT:** Android uygulama içerisinde kullanılacak olan metot tanımı, kullanılacak olan sınıfa ait adres bilgisini içererek, “Java_sg_vantagepoint_uncrackable2_MainActivity_init” şeklinde tanımlanmaktadır.

“init” metodunun tanımlı olduğu adres bilgisini bu şekilde elde etmiş bulunmaktayız – 0x00000fb0. “s 0x00000fb0” komutu ile imlecin belirtilen adrese taşıması gerçekleştirilir.

![](12.png)

“V” komutu kullanılarak bulunan adresten itibaren assembly kodlarının görselleştirilmesi sağlanır.

![](13.png)

Assembly kodlarını incelenirken 0x00000fc6 adresinde tanımlı
olan “call sub.fork_780” komutu dikkat çekmektedir. “s sub.fork_780”
komutu ile ilgili metodun tanımlandığı bellek adresine erişim sağlanır ve yine
“V” komutu yardımı ile metot içeriği ekranda görüntülenir.

**NOT:** Görsel kod ekranından, bellek adres alanına geçmek için “q” komutu kullanılmalıdır.

![](14.png)

“sub.fork_780” metodunun incelenmesi
esnasında “ptrace” isimli bir sistem çağrısı kullanıldığı
görülmektedir. “Ptrace” linux işletim sistemine ait bir sistem
çağrısıdır ve bir uygulamanın başka bir uygulamaya attach olması için
kullanılmaktadır.

Attach olma işlemi bir uygulamanın başka bir
uygulamaya ait işlemleri görebilmesi ve ona ait bellek alanları ile register
yapıları içerisine erişerek değiştirebilmesi olarak tanımlanabilir.

Peki bu komut bizim için önemli? Bir uygulamaya sadece bir uygulama attach olabilmekte, daha sonra hiçbir uygulama bu işleme dahil olamamaktadır. Bu nedenden ötürü UnCrackable-Level2 uygulamasına frida aracılığı ile bağlantı kurma işlemi gerçekleştirilememiştir.

Uygulama tarafından native kütüphanenin çağırılması ile birlikte aşağıdaki işlemler gerçekleştirilmektedir.

**NOT:** İşlemleri işleyiş sırasını takip etmek için “VV” komutu kullanılabilmektedir. Bu sayede radare2 tarafından dallanma işlemleri daha anlaşılır bir biçimde gösterilmektedir.

![](15.png)

- sym.imp.fork() metodu çağırılır ve sonucu sıfır ise 0x7d0 adresine sıçrama yapılır. 0x7a4 içerisinde bulunan sym.imp.fork() sistem çağrısı ile mevcut işlemin bir kopyası oluşturulur ve bu kopya işlemin adı “çocuk” işlem olarak tanımlanır. Gerçek işlem ismi ise “ana/ebeveyn” işlem şeklinde tanımlanır.

- sym.imp.fork() işleminin tamamlanmasının ardından geriye döndürdüğü sonuç eax kaydedicisi içerisine yazılır. 0x7b1 adresinde bulunan test komutu ise aldığı parametreleri AND işlemine tabi tutar ve sonuç sıfır ise ZF flag değerini “1” yapar. Parametre olarak sadece eax kaydedicisinin verildiği görülmektedir. Bu nedenle sonucun sıfır olabilmesi için eax içerisinde bulunan değerin sıfır olması gerekmektedir. Yani sym.imp.fork() metodu sonucu sıfır olarak elde edilmelidir. Sym.imp.fork() tanımı incelendiğinde, metodun ana işlem içerisinde çağırılması sonucunda çocuk işlem PID değerini döndürdüğü görülmektedir, fakat çocuk işlem içerisinde çağırılması durumunda sonuç “0” olarak döndürülmektedir.

- JE komutu ise ZF flag değerinin “1” olduğu durumlarda parametre olarak verilen adres değerine sıçrama işlemini gerçekleştirmektedir. Dolayısı ile çocuk işlem içerisinde bu kontrollerin gerçekleşmesi sonucunda belirlenen adrese sıçrama işlemi gerçekleşecektir.

- Şartlı dallanma sonucunun “true” olması durumunda 0x7d0 adresine gidilir ve ilk olarak sym.imp.getppid() metodu çağırılır. Ana işlemin PID değeri eax içerisine aktarılır. Daha sonra sym.imp.ptrace() metoduna parametre olarak aktarılacak olan değerler stack içerisine push edilirler (Stack LIFO yapısında olduğu için parametreler ters sırada stack içerisine aktarılırlar) Aktarılan parametreler ile sym.imp.ptrace(0x10, esi, 0, 0) metodu çağırılır. Ptrace metodunun kullanımı aşağıdaki şekildedir.

- ptrace (enum __ptrace_request request, pid_t pid, void *addr, void *data) Metot içerisine alınan parametreleri incelediğimizde ise ilk olarak 0x10 değerinin PTRACE_ATTACH enum değerine karşılık geldiği görülmektedir (“ptrace.h” kütüphanesi içerisinden bu bilgiye ulaşılır)

![](16.png)

- İkinci parametre olan esi kaydedicisi ise sym.imp.getppid() işleminin sonucunu içermektedir. Yani ana işlemin PID değeri parametre olarak aktarılmaktadır. Son iki parametrenin ise NULL olarak aktarıldığı görülmektedir.

- sym.imp.ptrace() metodu çalışmasının ardından farklı türde sonuçlar döndürebilmektedir. Bunlardan “-1” hata, “0” PTRACE_TRACEME (PID ile belirlenmiş olan işleme attach olunduğunu belirten değer), “1” PTRACE_PEEKTEXT vb. şekilde devam etmektedir. (ptrace.h kütüphanesi içerisinden diğer dönüş türleri incelenebilir)

![](17.png)

- “test eax, eax” komutu ile sym.imp.ptrace() metot sonucunun sıfır olup/olmadığının kontrolü yapılır ve sonuç sıfır ise 0x7ee adresinden uygulamaya çalışmaya devam eder (Sonucun sıfır olmadığı durumunda ise 0x858 adresine sıçrama yapılır ve uygulama bu noktadan çalışmaya devam eder)

- 0x7ee adresi ile başlayan metodu incelediğimizde üç adet parametre alan sym.imp.waitpid() metodunun çağırıldığı görülmektedir.

![](18.png)

- sym.imp.waitpid() ile belirlenen işlemin durdurulması/sonlanması beklemek için kullanılmaktadır. Alınan parametrelerden esi kaydedicisi içerisinde PID değerinin olduğu bilinmektedir. Bu nedenle sym.imp.waitpid() metodu ile ana işlem durumu takip edilmektedir ve ana işlemin sonlanması veya durdurulması durumundan çocuk işlem haberdar edilmektedir. Daha sonra ise yeniden sym.imp.ptrace() metodunun çağırıldığı görülmektedir. Bir önceki kullanımından farklı olarak “7” değeri parametre olarak aktarılmaktadır. “ptrace.h” kütüphanesine bakıldığında bu değer “PTRACE_CONT” ifadesine karşılık gelmektedir.

![](19.png)

- Seçilmiş olan enum türü, takip edilen bir uygulamanın sonlandırılması sonrasında yeniden başlatılmasını gerçekleştirmektedir. Bu sayede, ana işlem kapatılsa bile çocuk işlem tarafından yeniden başlatılabilmektedir.

![](20.png)

**NOT:** 6070 PID değerine sahip ana işlemi sonlandırma denemesinin “kill” metodu ile gerçekleştirilemediği görülmektedir

- İşlem 0x832 adresinden itibaren çalışmaya devam etmektedir. Burada tanımlanan metot içerisinde, sym.imp.waitpid() metodunun kullanıldığı görülmektedir. Ayrıca 0x832 ile 0x820 adres aralığının bir döngü oluşturacak şekilde tanımlandığı anlaşılmaktadır.

![](21.png)

- sym.imp.waitpid() işlemi sonucunun sıfır olması (yani çocuk işlem tarafından metodun kullanılması) durumunda 0x858 adres değerine sıçrama yapıldığı görülmektedir. 0x858 adres değeri içerisinde bulunan işlemler sonucunda tanımlanmış olan işlemlerin tamamlandığı görülmektedir. 0x864 adresinde bulunun “ret” komutu ile “sub.fork_780” metodunun işlemini bitirdiği ve “init” metoduna dönüş yapıldığı görülmektedir.

![](22.png)

- sym.imp.waitpid() işlemi sonucunun sıfır olmaması durumunda ise iki farklı sonuç meydana gelmektedir. İlk sonuç uygulamadan çıkış yapılması ile sonlanmaktadır. Diğer ihtimal ise sym.imp.ptrace() metodunun PTRACE_CONT enum değeri ile çağırılarak yeniden 0x832 adresine dönüş yapılması şeklindedir.

![](23.png)

Uygulama tarafından
sağlanmış olan kontrolün tespit edilmesinin ardından sıra geldi bu kontrolün
atlatılmasına. İlk olarak elde edilen “libfoo.so” dosyasını
“radare2 -Aw libfoo.so” komutu ile açarız. “-A” komutu
uygulamanın açılırken analiz edilmesini sağlamaktadır (uygulama içerisinden
girile “aaa” komutu ile aynı görevi yerine getirmektedir)

“-w” komutu ise dosyanın yazılabilir formatta açılmasını sağlamaktadır.

Çok farklı yöntemler ile alınmış olan kontrol mekanizması atlatmak mümkün olacaktır. Bizler “init” metodu içerisinde bulunan “call sub.fork_780” komut satırını farklı bir adres değerinde bulunan işlemleri çağıracak şekilde değiştireceğiz. Bunun için ilk olarak ilgili komut satırının adres değerini kopyalıyoruz.

![](24.png)

Daha sonra görsel ekrandan “q” komutu ile çıkıyoruz ve “s 0x00000fc6” komutu ile değişiklik yapmak istediğimiz komut satırına gidiyoruz.

![](25.png)

Son olarak “wa call 0x864” komutunu konsol üzerinden çalıştırıyoruz (“wa” komutu sayesinden uygulama içerisine assembly kodları eklenebilmektedir. “w?” komutu yardımı ile kod ekleme işlemine ait diğer özellikler görüntülenebilir)

![](26.png)

Metot içeriğinin değiştiği “V” görselleştirme komutu kullanılarak doğrulanabilmektedir.

![](27.png)

0x864 adres bölümünde “sub.fork_780” metodunun “init” metodu içerisine geri dönüş yapılmasını sağlayan bölüm bulunmaktadır.

İşlemlerin tamamlanmasının ardından radare2 uygulamasından çıkış yapılır. Uygulamanın çalıştığı emulatör üzerine komut satırı aracılığı ile bağlanılır ve “/data/app/owasp.mstg.uncrackable2-1/lib/x86” dizini altına erişim sağlanır.

![](28.png)

Uygulamanın mobil cihaz üzerine yüklenmesi sonrasında “libfoo.so” native kütüphanesinin “/data/app/owasp.mstg.uncrackable2-1/lib/x86” içerisine yerleştiği görülmektedir. Daha sonra burada bulunan “libfoo.so” dosyasını “rm -r libfoo.so” komutu kullanılarak silinir. Ardından değiştirilmiş olan “libfoo.so” dosyası “/data/app/owasp.mstg.uncrackable2-1/lib/x86” dizini altına taşınır.

Konsol üzerinden “frida-ps -U” komutu ile emulatör üzerinde çalışan uygulamaların listelenmesi gerçekleştirilir ve “owasp.mstg.uncrackable2” uygulamasına ait sadece bir adet PID değeri bulunduğu görülür.

![](29.png)

Gerçekleştirilmiş olan bu işlemin adı “binary patching” olarak bilinmektedir. Artık frida kodları uygulama içerisine enjekte edilebilecektir.

Aşağıdaki şekilde yazılmış olan frida kodları ile uygulama içerisinde bulunan root kontrolü atlatılmaktadır (Burada gerçekleştirilen işlemler UnCrackable-Level1 uygulaması ile birebir aynı olduğu için detaylı olarak bahsedilmemiştir)

![](30.png)

Son adım olarak uygulama içerisinde bulunan gizli ifadenin bulunması istenmektedir. İlk olarak decompile edilmiş olan uygulama kodları içerisinde gizli ifade kontrolünün nasıl yapıldığı incelenmektedir. “MainActivity” sınıfı içerisinde bulunan “verify” metodu içeriği aşağıdaki şekildedir.

![](31.png)

Ara yüz üzerinden alınan parametrenin “m.a()” isimli metot içerisine aktarıldığı görülmektedir. Metot içeriği aşağıdaki şekildedir.

![](32.png)

Gerçekleştirilen işlemlerin “libfoo.so” kütüphanesi içerisinde bulunan “bar” isimli metot yardımı ile gerçekleştirildiği görülmektedir. Radare2 aracı yardımı ile native kütüphane içerisinde bulunan “bar” metodu içeriği incelenir.

![](33.png)

Dikkatli olarak incelendiğinde 0x00001014 ile 0x00001043 adresleri arasında gizli ifadenin bulunduğu görülmektedir – ‘Thanks for all the fish’. Fakat elde edilen değer ara yüz üzerinden doğrulanmak istendiğinde aşağıdaki hata mesajı alınmaktadır.

![](34.png)

Hatanın sebebini bulmak için “init” ve “bar” metotları incelenir.

![](35.png)

“init” metoduna ait 0x00000fcb adresi incelendiğinde “[ebx + 0x40]” bellek değeri içerisine “1” değerinin aktarıldığı görülmektedir. “bar” metoduna ait 0x00001004 adresi içerisinde de “[ebx + 0x40]” bellek değerinde bulunan değerin “1” değeri ile karşılaştırıldığı görülmektedir. Binary patching nedeni ile ebx kaydedicisinin içeriğinde değişiklik olabileceği yada olması gereken değişikliklerin olmaması ihtimali bulunmaktadır. Bu nedenle ikinci bir binary patching işlemi ile 0x0000100b adresinde bulunan “jne” dallanma ifadesi değiştirilecektir.

![](36.png)

İlk olarak “q” komutu ile görsel bölümden çıkış yapılır ve daha sonra “s 0x0000100b” komutu ile işaretçi ilgili metot adresine taşınır. Son olarak “wa je 0x1093” komutunun ilgili adrese yazılması işlemi gerçekleştirilir. “bar” metodunun güncellenmiş versiyonu aşağıdaki şekildedir.

![](37.png)

Yeniden binary patching işlemi uygulanmış olan “libfoo.so” kütüphanesi, android cihaz içerisinde bulunan “/data/app/owasp.mstg.uncrackable2-1/lib/x86” dizini altına taşınmaktadır (Mevcut olan “libfoo.so” kütüphanesinin silinmesinin ardından taşıma işlemi gerçekleştirilmelidir)

Önce UnCrackable-Level2 uygulaması daha sonra da frida ile yazılmış olan script çalıştırılır ve radare2 üzerinden elde edilen gizli ifade ‘Thanks for all the fish’ değeri ara yüz üzerinden girilir.

![](38.png)

Tüm bu işlemler sonucunda “Success!” değeri elde edildiği görülmektedir.
