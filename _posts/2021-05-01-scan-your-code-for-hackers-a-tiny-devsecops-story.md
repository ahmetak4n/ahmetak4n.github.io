---
title: Scan Your Codes for Hackers
date: 2021-05-01 09:15:00 +0300
categories: [web]
tags: [devsecops, web]
description: Uygulama geliştirme süreçlerinin güvenliğinin arttırılması esnasında yapılabilecek hatalı SonarQube entegrasyonlarının ne tür tehditlere sebebiyet verilebileceğinin incelenmesi
media_subpath: /assets/img/posts/scan-your-code-for-hackers-a-tiny-devsecops-story
image: cover.png
---

“Kötü niyetli kişilerin uygulamalarımızı analiz etmesi için nasıl kodlarımızı ve kod tarama sonuçlarımızı paylaşabiliriz” cümlesi içerisinde bir gariplik/yanlışlık olduğunu düşünebilirsiniz, düşünmeyin. Çünkü bazılarımız, farkında olmadan “DevSecOps” kapsamında bu olayın gerçekleşmesine izin verebiliyor.

#### Hazırlık

Konuya büyülü “DevOps” ifadesinden başlamakta fayda var. “DevOps”, müşterilerinize düzenli aralıklarla (çok ufak geliştirmeler bile olsa) çıktı verebilen, problemlerin daha ufak parçala ayrılarak çözülebildiği, her hangi bir T anında müşteri talebinin planlama ve geliştirme süreçlerine dahil olabileceği, hızlı/çevik süreçler elde edilmesini hedefleyen felsefe/düşünce olarak tanımlanabilir. (!Waterfall)

İşin teknik tarafında ise geliştirme, operasyon ve test (QA) ekipleri, “DevOps” mantalitesi ile tasarlanmış araçları kullanarak geliştirme, derleme, test etme, dağıtma vb. adımları ortak bir şekilde yürütebilmektedir.

(Konu hakkında bir çok detaylı yazı mevcut bknz. <https://medium.com/devopsturkiye>)

İlk başta her şey beklendiği gibi ilerlerken, “Hızlı geliştirme ve dağıtma çok güzel, peki geliştirilen uygulama güvenliğine yönelik bir kontrol yapıldı mı?” sorusu ile bir şeylerin eksik olduğu görülmüştür. Geliştiricilerin kodları geliştirme ortamlarına “push” etmesi, test script lerinin çalışması, “docker” imajlarının oluşturulması ve uygulamanın ortamdan ortama koşması esnasında uygulama güvenliğine en son sıra gelmekteydi (Sıra gelmeyen kurumlar olduğu da aşikar)

![](01.png)

Temel fikri hızlı değişim/aksiyon olan bu süreç uygulama güvenliği konusunda bir takım eksikler barındırmaktaydı. Çok temel zafiyetlerin tespiti için bile “Test/Pre-Prod/Prep” gibi ortamlara kadar süreçlerin ilerlemesi gerekmekteydi (Kurum içerisinde uygulama güvenlik testlerinin hiç yapılmıyor olması da muhtemel)

Bu eksikliğin giderilmesi adına, “DevOps” süreçlerinde yer alan “git”, “jenkins”, “selenium” gibi araçlara ek olarak;

- **SAST – Static Application Security Testing:** Geliştirilen uygulamaya ait kaynak kod ve “binary” dosyaları (Büyük bir çoğunluğu) analiz ederek “pre-defined” zafiyetleri tespit edebilen araçlar. (Checkmarx, Veracode, Fortify, ~SonarQube vb.)

- **SCA – Software Composition Analysis:** Uygulamanız içerisinde yer alan açık kaynaklı bileşenlerin versiyon ve lisans bilgilerini kontrol ederek zafiyet ve olası uygunsuz kullanımları tespit edebilen araçlar. (Whitesource, Dependency Check, Dependency Track, Synk, vb.)

- **DAST – Dynamic Application Security Testing**: Uygulamalarınız çalışma zamanlı olarak test edilmesine olanak sağlayan güvenlik araçları. (Owasp ZAP, Acunetix, Netsparker, BurpSuite, vb.)

eklenmesi ile “DevSecOps” yaklaşımı ortaya çıkmıştır.

Yukarıda bahsi geçemeyen “Vulnerability Management”, “Container Security”, “Git-Hooks”, “Vault”, vb. başka araç türleri de “DevSecOps” süreçlerinde kullanılmakta olup, internet üzerinde konu ile ilgili bir çok detaylı yazı bulunduğu için detaylarından bahsetmeyeceğim.

**Not:** Güvenlik araçlarının **yetkin güvenlik uzmanlarına destek amaçlı** kullanmanın fayda sağlayacağını, fakat araçların yetkin personelin alternatifi olmadığını unutmamak gerekir. Zira SAST ile tarama yaptık “güvendeyiz”, DAST ile taradık “uçuyoruz” demek pek mümkün değildir (A fool with a tool is still a fool)

“BlackHat 2019” etkinliğinde paylaşılan aşağıdaki görsel “DevSecOps” konusunu özetlemektedir.

![](02.png)

#### Ana Konu

“DevSecOps” ifadesinde yer alan “Sec” ifadesinin sadece uygulama ve uygulama çalışma ortam/sunucu güvenliği ile ilgilendiğini bir önceki başlık içerisinde belirtmiştik. Süreç içerisine dahil ettiğiniz SAST, DAST, SCA, Container Security vb. araçlarının güvenilir bir şekilde konumlandırılıp/konumlandırılmadığı “DevSecOps” sürecinin bir parçası olarak görülmemektedir.

Yazımızın ana konusu tam olarak bu olaya dayanmaktadır “Uygulamalarımızın güvenliği için kullanacağımız araçların güvenilir olmayan yöntemler/konfigürasyonlar ile konumlandırılması”. Bu yazı kapsamında ki hedef aracımız, “SonarQube”.

“SonarQube” bir tür kaynak kod analiz aracıdır (SAST). Güvenlik ile ilgili tespitleri kısıtlı (“Veracode”, “Checkmarx”, “Fortify” gibi ürünlere kıyasla) olmakla birlikte “Code Quality” alanında çok başarılı olduğunu söylemek yanlış olmayacaktır. “SonarQube” aracının en güzel yanı ücretsiz versiyonunun bulunmasıdır. Bu nedenle nerde bir “DevSecOps” süreci varsa orada “SonarQube/SonarCloud” bulunması muhtemeldir.

“SonarQube” aracı yaygın kullanımına ek olarak önemli bir özellik ile gelmektedir, “Tarama yaptığınız proje, SonarQube üzerine varsayılan olarak “Public-Açık Proje” şeklinde eklenmektedir”. Bunun anlamı, “SonarQube” uygulamanız internet üzerinden erişilebilir ve varsayılan konfigürasyonları ile konumlandırılmış ise (tarama yapılan projelerinizi “Private/Gizli” olacak şekilde özellikle belirtmediyseniz), tüm dünya tarama yaptığınız uygulama kodlarına ve tarama sonuçlarınıza erişim sağlayabilecektir.

Kısacası uygulamalarınızı saldırganlar için tarayıp, sonuçlarını tüm dünya ile paylaşmış olacaksınız.

#### Uygulama

Uygulama aşamasına geçemeden önce ilk yapmamız gereken şey, dış dünyaya açık “SonarQube” servislerini bulmak. Bu noktada “Shodan” arama motorundan yardım almak işleri oldukça kolaylaştırdı.

![](03.png)

Görüldüğü üzere 2,789 “SonarQube” sunucusu tespit edilmiştir. Fakat “Shodan” ücretsiz hesabı kullanmam sebebiyle 2,789 sonuçtan sadece 100 tanesi hakkında bilgi alabilmekteyim. Tarayıcı ara yüzü üzerinden tek-tek IP adreslerini ve projelerinin herkese açık olup/olmadığının kontrolünü yapmak zor olacağı için ufak bir uygulama ile bu işi otomatikleştirdim. (Ufak başlayıp büyüyen bir proje demek daha doğru olabilir)

![](04.png)

Aracın kaynak kodlarına <https://github.com/ahmetak4n/radar> üzerinden erişim sağlayabilirsiniz.

Radar aracı ile, “Shodan” tarafından tespit “SonarQube” servisleri üzerinde;

- Public/Açık projesi bulunup/bulunmadığının kontrol edilmesi
  - Public/Açık proje mevcut ise kaç adet olduğunun tespiti
    - Projelere ait toplam “Code Smell”, “Vulnerability”, “Bug” ve “Security Hotspot” sayılarının tespiti
- “Admin:admin” varsayılan kullanıcısı ile giriş denemesi

kontrolleri sağlanmaktadır.

#### Sonuç

Tarama sonucunda, **53** adet “SonarQube” servis projelerinin herkes tarafından erişilebilir olduğu tespit edilmiştir. Bu doğrultuda dış dünyaya açık olan “SonarQube” servislerinin **%53** oranında yanlış konfigürasyonla çalıştığını söylemek yanlış olmayacaktır.

Tespit edilen “SonarQube” servislerinden elde edilen proje sayısı ise **636** olarak belirlenmiştir. Ayrıca **14** “SonarQube” servisi ise hiçbir proje içermemektedir. Proje sayıları ile ilişkili “SonarQube” servis sayıları aşağıdaki gibidir.

| İçerdiği Proje Sayısı | Sonar Servis Sayısı |
| --- | --- |
| 0 | 14 |
| 1 | 11 |
| 2 | 8 |
| 3 | 3 |
| 4 | 1 |
| 5 | 2 |
| 6 | 1 |
| 7 | 2 |
| 10 | 2 |
| 12 | 1 |
| 14 | 1 |
| 15 | 1 |
| 22 | 2 |
| 57 | 1 |
| 80 | 1 |
| 123 | 1 |
| 201 | 1 |

Tespit edilen projeler üzerinde yapılan ilk kontrol, projenin açık kaynak kodlu bir uygulamaya mı yoksa bir ürüne/firmaya mı ait olduğunu tespit etmeye yönelikti. Bu kapsamda **53** “SonarQube” servisi içerisinden **10** tanesinin açık kaynak kodlu olmayan ürünlere ait olduğu tespit edilmiştir.

Açık kaynak kodlu olmadığı tespit edilen projelere ilişkin domain bilgilerini güvenlik amacıyla paylaşmam mümkün değil, fakat çoğunluğunun **Uzak Doğu** ve **İspanya/Portekiz/Latin Amerika** firmalarının oluşturduğunu söylemek yanlış olmayacaktır.

Bu firmaların sektörleri ise birbirlerinden oldukça farklı.

- **Yazılım Ürün/Çözüm Sağlayıcısı:** Web sayfalarında belirttikleri 6 ürünlerine ilişkin “SonarQube” tarama sonuçları ve kaynak kodları tespit edilmiştir.
- **Uzak Doğu Dil Öğrenme Platformu:** Dil platforma ait SonarQube” tarama sonuçları ve kaynak kodları tespit edilmiştir.
- **ERP Firması:** ERP çözümlerine ilişkin SonarQube” tarama sonuçları ve kaynak kodları tespit edilmiştir.
- **E-Ticaret Sitesi:** E-Ticaret uygulamalarına ilişkin SonarQube” tarama sonuçları ve kaynak kodları tespit edilmiştir.
- **Filo Yönetim Firması:** Filo yönetimi için geliştirilen uygulamalarının SonarQube” tarama sonuçları ve kaynak kodları tespit edilmiştir.
- **Avrupa da bir Üniversite:** Private/Gizli “github” hesabında yer alan bir projenin SonarQube” tarama sonuçları ve kaynak kodları tespit edilmiştir.
- **AI Danışmanlık Firması:** Analizlerinde kullandıkları projelerinin “SonarQube” tarama sonuçları ve kaynak kodları tespit edilmiştir.
- **Agile Danışmanlık Firması:** Kurum içi kullanılan uygulamalara ait projelerin SonarQube” tarama sonuçları ve kaynak kodları tespit edilmiştir.
- **Eczane ve Hastalar için E-Ticaret Sitesi:** Uygulamaya ve alt projelerine ait SonarQube” tarama sonuçları ve kaynak kodları tespit edilmiştir.

Ayrıca “çöp” olarak tanımlayabileceğimiz bir çok test, deneme uygulaması ile açık kaynak kodlu projelere ilişkin tarama sonuçları ve kaynak kodları da tespit edilmiştir. Bu projeler içerisinde sadece **1** adet **Go**, **1** adet **C#**, **6** adet **Python** projesi tespit edilmiştir. Geriye kalan projelerin büyük çoğunluğunu **Java (Spring)**, **Javascript** ve **Typescript** projeleri oluşturmaktadır.

Tespit edilen proje sayısının fazla olması sebebiyle, uygulamalara ait kaynak kodların incelenmesine henüz başlamış değilim. Fakat kodlar içerisinde statik olarak tanımlanmış hesap, şifre ve benzeri bilgilerin tespit edilmesine yönelik bir ön çalışma gerçekleştirilmiştir. Bu doğrultuda tespit edilen bilgiler aşağıdaki şekildedir.

**6** proje içerisinde statik tanımlanmış mail kullanıcı adı ve şifre bilgisi **(SMTP)** tespit edilmiştir. “SendGrid” gibi “Mail Campaigning” hesapları da tespitler içerisinde yer almaktadır.

**1** adet **veri tabanı** erişim bilgisi, gizli olması gerektiği düşünülen bir proje içerisinde tespit edilmiştir. İlgili veri tabanı içerisine uzaktan bağlantı sağlandığı doğrulanmıştır.

![](05.png)

**2** adet **AWS S3** API kullanıcı ID ve anahtar değeri tespit edilmiştir.

**1** adet **Azure DevOps** PAT (Personnel Access Token) değeri ve kullanıcısı tespit edilmiştir. Ayrıca proje içerisinde firma çalışan listesi ve hesaplarına ait “.csv” dokümanları yer almaktadır.

**1** adet **Redis** sunucusuna ait kullanıcı adı/şifre bilgisi tespit edilmiş olup, bu sunucuya başarılı bir şekilde erişim sağlandığı görülmektedir.

![](06.png)

**1** adet **Google AI** servis hesabına ilişkin ID ve anahtar değeri tespit edilmiştir.

Son olarak **1** adet **Docker Registry** erişim bilgisi ve **1** adet **OneSignal** ID ile anahtar değeri elde edilmiş olup, elde edilen bu anahtarlar ile erişim sağlanabildiği doğrulamıştır.

Özetle, bir çok firma veya bireyin farkında olmadan projelerine ait kodlarını ve kod tarama sonuçlarını kötü niyetli kişilerin bulabileceği şekilde internet konumlandırdığı görülmüştür.
