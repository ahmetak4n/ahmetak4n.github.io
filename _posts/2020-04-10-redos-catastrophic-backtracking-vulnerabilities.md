---
title: RE-DOS / Catastrophic Backtracking Vulnerabilities
date: 2020-04-10 09:15:00 +0300
categories: [web]
tags: [automata, catastrophic backtracking, re-dos, web]
description: Otomata teorisine ait DFA/NFA modelleri ve bu modellerin yanlış uygulanması sonucunda oluşabilecek sorunlardan bir tanesi olan Catastrophic Backtracking zafiyetinin incelenmesi
media_subpath: /assets/img/posts/redos-catastrophic-backtracking-vulnerabilities
image: cover.png
---

“[ReDOS](https://owasp.org/www-community/attacks/Regular_expression_Denial_of_Service_-_ReDoS) – Regex DOS” zafiyeti; uygulamalar içerisinde kullanılan “regex – regular expression” ifadelerinin, kullanıcı girdileri ile işleme girmesi sonucunda milyonlarca/milyarlarca kez iterasyona uğraması ve buna bağlı olarak aşırı miktarda “CPU” tüketmesi şeklinde tanımlanabilmektedir. Bu zafiyetin bir diğer ismi de “Catastrophic Backtracking” olarak bilinmektedir.

Genel tanımın ardından, “ReDOS” zafiyetinin tüm detaylarını incelemeye başlayabiliriz. Aslında bu noktada incelememiz gereken en temel şey “Regular Expression” adı verilen, bir çok insanın hayatında kullandığı (mail adresinin doğruluğunun kontrolü vb.) ama arka planında nelerin gerçekleştiği ile ilgilenmediği ifadelerdir.

Başlangıç noktamız “Finite Automata – FA” adı verilen bir matematiksel modeldir. FA sonlu sayıda durum içermekte ve bu durumların modellenmesinde kullanılmaktadır. İki farklı FA mevcuttur. İlk FA türü “Deterministik Finite Automata – DFA” olarak isimlendirilmektedir. DFA’nın bu isime sahip olmasının nedeni, bir giriş değeri ile geçilebilecek durum sayısının tek olmasıdır. Aşağıdaki çizimde bir DFA modeli bulunmaktadır.

![](01.webp)

Hızlı bir şekilde görselde bulunan ifadeleri anlatacak olursak, “q0, q1 ve q2” simgeleri durumları göstermektedir. “0, 1” ile değerleri ise geçişleri temsil etmektedir. Ayrıca geçiş değerleri, sahip olduğumuz alfabe içerisinden seçilebilmektedir. “q2” durumunun çift katlı çizgiye sahip olması, onun bir bitiş elemanı olduğunu göstermektedir. Amaç çok basit, elimizde bulunan alfabe (0,1) ile bitiş durumuna erişmek. Örneğin; “11”, “011”, 011101″ vb. tüm durumlar yukarıda tanımlı bulunan DFA tarafından tanınacaktır, çünkü hepsi “q2” durumunda sonlanmaktadır. Fakat “0100” değeri tanınmayacaktır, çünkü son olarak “q0” durumunda kalmaktadır.

Peki yukarıdaki FA neden “Deterministik” olarak adlandırılıyor. Bunun sebebi tek bir alfabe (0,1) elemanı ile sadece tek bir duruma geçiş sağlanabiliyor olmasıdır. Örneğin aşağıdaki tasarım “Deterministik” bir FA değildir.

![](02.webp)

“q1” durumu dikkatli incelendiğinde “0” değeri için hem “q0” hemde “q2” ye gidilebileceği görülmektedir. Bu durum tasarlanan FA’nın deterministik bir şekilde hareket etmeyeceğini göstermektedir. Bu nedenle yukarıda bulunan FA türüne verilen isim “Non-Deterministik Finite Automata – NFA” şeklindedir.

NFA ile DFA karşılaştırıldığında, NFA’lar çok daha fazla ifadeyi tanıma potansiyeline sahiptir. Bununda en büyük nedeni tüm durumları farklı geçiş değerleri için tarayarak verilen ifadeye uygun olup/olmadığını inceleyebilmektedir.

Tamda bu noktada “ReDOS” zafiyeti konusunu düşünecek olursak, tasarlanan bir “regular expression” ın beklenmedik sayıda iterasyona zorlanabilmesi için NFA türünde olması gerektiği görülmektedir. Çünkü NFA yapısı gereği, kendisine verilen ifadeyi olası tüm yolları deneyerek bulmak istemektedir.

“Regular Expression” öncesinde değinilmesi gereken son nokta “Regular Sets – Düzgün Kümeler” adı verilen ve FA tarafından tanınan durumları içeren kümelerdir. Bu küme içerisinde bulunan durumları ifade etme yöntemi ise “Regular Expression – Düzenli İfadeler” olarak adlandırılmaktadır. Örneğin; yukarıda tasarlanan DFA için 0*1(010*)*1(0+1)* regular expression olarak tanımlanabilmektedir. “*” karakteri boş küme veya ilgili karakterin herhangi bir sayıda tekrarını temsil etmektedir. “+” karakteri ise ilgili karakterin en az bir veya sonsuz tekrarı anlamında kullanılmaktadır.

Görüldüğü üzere tasarlanan FA üzerinde geçerli olan ifadeleri tespit edebilecek bir regular expression tasarladık, DFA ile NFA arasındaki farkları net bir şekilde görmüş olduk.

NFA türüne ait FA’lerin “ReDOS” kapsamında hedefimiz olduğunu belirtmiştik. “ReDOS” kapsamında inceleyeceğimiz modern regex motorları da NFA’ler üzerine geliştirilmişlerdir ve NFA’e ait olan, olası tüm yolları arama özelliğine sahiptirler. Bu özellik dokümanın ilerleyen bölümlerinde “backtracking” olarak adlandırılacaktır.

“Backtracking” özelliği sayesinde regex motorlarının çok daha güçlü olduğu bir gerçek, fakat her güç yanında bir takım tehlikeler barındırmaktadır.

İlk olarak “A(B|C+)+D” şeklinde bir regex ifademiz olduğunu varsayalım ve “backtracking” özelliğinin nasıl çalıştığını inceleyelim. Regex tarafından incelenmesini istediğimiz ifade “ABCCCCCCBD” şeklinde olsun. “[Regex101](https://regex101.com/)” adresi üzerinden ilgili regex ile girdimizin “debug” edilmesini sağlayarak her adımı tek tek incelemeye başlıyoruz.

![](03.webp)

6 numaralı adıma gelindiğinde “C+” ifadesine göre karşılaştırma yapılacağı görülmektedir. Bir sonraki adımda ise tüm “C” karakterlerinin tanındığı görülmektedir.

![](04.webp)

Regex motorumuzun “backtracking” özelliği olmasından dolayı bir sonraki işlem için bir adım geri gelinerek “B karakteri var mı” sorusuna cevap aranmaktadır. (“+” karakterinin kullanılmış olması bu duruma sebep olmaktadır)

![](05.webp)

Son “B” karakterinin bulunmasının ardından, son bir kontrol ile boşta “B” veya “C” karakteri kalıp/kalmadığının kontrolü gerçekleştirilir ve “D” karakterinin tespiti ile ifadenin tamamlanması sağlanır. Mutlu SON!

![](06.webp)

Maalesef her son mutlu bitmiyor:) Mutsuz son örneği olarak “A(B|C+)+D” regex ifadesi için “ABCCCCCCBK” değerini girdi olarak verelim. İfademizin ilgili regex tarafından tanınmayacağı görülmektedir. Çünkü “K” harfi ile bitiyor. Peki regex motoru bu durumu kaçıncı adımda tespit edebilecek? Olası tüm ihtimalleri denedikten sonra.

“Regex101” üzerinden işlem adımlarını tekrar inceleyerek, başarısız senaryolarda “backtracking” davranışını gözlemleyebiliriz. Bir önceki ifade için sadece “13” adım gerekirken, mevcut ifade için “398” adım denendiği görülmektedir.

![](07.webp)

12 numaralı adıma kadar gerçekleştirilen tüm işlemler, bir önceki girdi ile tamamen aynıdır. 11 numaralı adımda bir sonraki karakterin “D” olup/olmadığı kontrol edilmektedir.

![](08.webp)

Fakat karşılaştırma başarısız olmaktadır. Bu nedenle “backtracking” özelliği ile bir önceki duruma geçiş yapılmaktadır.

![](09.webp)

Regex motorunun şuan tek amacı farklı bir yol üretip, bitiş noktasına ulaşmaktır. Bu doğrultuda bir önceki karakter olan “B” ile, “D” karakterinin eşit olup olmadığı kontrol edilerek işlemlere devam edilmektedir.

![](10.webp)

Eşitlik sağlanamadığı için “backtracking” tarafından “(B|C+)” ifadesine dönüş sağlanmaktadır.

![](11.webp)

Dönüş sonrasında gerçekleştirilen ilerleme için “C” karakteri bulunup bulunmadığı kontrol edilmektedir ve biraz önce “backtracking” esnasında boşaltılmış olan “C” karakteri tespit edilmektedir.

![](12.webp)

İşlem devam ettiğinde herhangi bir “B” veya “C” karakteri bulunup/bulunmadığı kontrolü sağlanmaktadır. Tamda bu noktada bir önceki “backtracking” işleminde boşalmış olan “B” karakteri tespit edilir. Daha sonra başka “B” veya “C” karakteri bulunup/bulunmadığı kontrol edilir ve bir sonraki karakter olan “K” karakterinin karşılaştırılmasına geçilir.

![](13.webp)

“K” karakterinin eşitsizliğinin ardından bir kez daha “backtracking” devreye girer ve mevcut durum iki önceki konuma alınır.

![](14.webp)

Görüleceği üzere toplamda iki “C” ve bir “B” karakteri geriye dönülmüştür. Bu işlem tüm tekrarlayan karakterler için gerçekleştirilecek olup “A” karakterinin tespit edildiği ilk duruma dönülene kadar devam edecektir.

![](15.webp)

Bir sonraki aşamada, girdi olarak “ABCCCCCCCCBK” kullanacağız ve toplam iterasyon sayısındaki artışı inceleyeceğiz. Yeni eklenen iki “C” karakteri sonrasındaki toplam iterasyon sayısının “1552” olduğu görülmektedir.

![](16.png)

Toplam “C” sayısının 10 olduğu durumda 6162 iterasyon, “C” sayısının 20 olduğu durumda ise 857143 iterasyon gerçekleştirildiği görülmektedir.

![](17.webp)

Peki bu durum, bir web uygulamasının hizmet veremeyecek hale gelmesine neden olabilir mi? Kesinlikle!

Demo kapsamında hazırlanan .Net Core uygulamasının içeriği aşağıdaki şekildedir.

![](18.webp)

“Get” metoduna gönderilen “search” parametresi, daha önceden kullanmış olduğumuz regex ifadesi “A(B|C+)+D” kullanılarak karşılaştırılmaktadır. Elde edilen sonuç ve işlem süre bilgisi kullanıcıya cevap olarak döndürülmektedir.

![](19.webp)

Olası en kötü duruma ulaşabilmek için “C” karakter sayısını olabildiğince arttırıp, ifademizin son karakterini “K” harfi olacak şekilde güncellemekteyiz. Oluşturulan yeni ifade ile uygulamaya tekrar istek gönderdiğimizde, uygulamadan cevap alamadığımızı görülmektedir.

![](20.webp)

Kullandığım cihazın anlık CPU kullanımını incelediğimizde ise “dotnet” tarafından %99-%100 aralığında bir kullanım olduğu görülmektedir.

![](21.webp)

Bu süreçte uygulama tarafından hiçbir isteğe cevap verilmediği görülmüş olup, sadece tek bir “HTTP” isteği ile başarılı bir DOS işlemi gerçekleştirilebildiği doğrulanmıştır.

“ReDOS” zafiyetinin detayları ile anlaşılmasının ardından sıra çözüm önerilerinin bulunmasındadır. Bu noktada yapılabilecek ilk öneri “+” ve “*” gibi, “Quantifiers” olarak da bilinen karakterlerin zorunlu olmadıkça kullanılmaması şeklindedir.

Bir diğer öneri ise “Greedy” karşılaştırma ifadeleri yerine “Lazy” karşılaştırma ifadelerinin kullanılması olacaktır. Bu konu hakkında daha fazla bilgiye [link](https://mariusschulz.com/blog/why-using-the-greedy-in-regular-expressions-is-almost-never-what-you-actually-want) üzerinden erişebilirsiniz.

Son çözüm önerisi ise regex karşılaştırma işlemlerine “timeout” süresi atanmasıdır. Belirli bir süre üzerine çıkan işlemlerin iptal edilmesi ile olası “ReDOS” saldırılarının önüne geçilebilmektedir.

Geliştirmiş olduğumuz örnek uygulama içerisinde bulunan “Regex” kütüphanesi, içerisine aldığı “timeout” süresinin dolması ile bir hata fırlatmakta ve regex karşılaştırma işlemine devam etmemektedir. Bu özelliğin kullanımı aşağıdaki şekildedir.

![](22.webp)

Uygulamaya tekrar “ReDOS” gerçekleştirebilmek için istek gönderdiğimizde ise, “1 saniye” sonunda hata mesajı fırlatıldığı ve uygulamanın normal seyrine devam ettiği görülmektedir.

![](23.webp)

**Referans:**

- <https://owasp.org/www-community/attacks/Regular_expression_Denial_of_Service_-_ReDoS>
- <https://mariusschulz.com/blog/why-using-the-greedy-in-regular-expressions-is-almost-never-what-you-actually-want>
