---
title: "@Keyv/Mysql — Sql Injection"
date: 2026-01-19 09:45:32 +0300
categories: [web]
tags: [keyv-mysql, sql-injection, web]
description: "@keyv/mysql paketinin has metodunda tespit edilen SQL Injection zafiyeti ve analizi."
media_subpath: /assets/img/posts/keyv-mysql-sql-injection
image: cover.png
canonical_url: https://blog-tr.peakcyber.com/keyv-mysql-sql-injection-44fa62ec2c12
---

[Keyv](https://github.com/jaredwray/keyv) farklı veritabanları için key-value şeklinde veri saklama desteği veren bir kütüphanedir. Testlerin gerçekleştirildiği aralıkta [NPM](https://www.npmjs.com/package/keyv) üzerinden haftalık indirilme rakamının 62 milyondan daha fazla olduğu görülmektedir.

## Analiz

Gemini yardımıyla kaynak kod inceleme ve zafiyet bulmaya yönelik çalışmalar gerçekleştirirken NPM indirme listelerinde üst sıralarda karşılaştığım Keyv projesini incelemeye karar verdim. Gemini için tanımladığım prompt aşağıdaki şekildedir.

```json
{
  "task": "Vulnerability research",
  "target": "Current project source code",
  "tone": "Professional",
  "subject": ["remote code execution", "sql injection", "nosql injection", "orm injection", "header injection", "xss", "prototype pollution", "server side template injection", "insecure deserialization", "os command injection", "code injection", "denial of service"],
  "exclude": "Ignore test files",
  "verify": "Don't try verify findings",
  "output": {
    "task": "Write all findings in Vulnerability.md file",
    "subject": ["vulnerability topic", "vulnerability description", "location of vulnerable code", "poc", "impact", "remediation"]
  }
}
```

Gemini ile gerçekleştirilen analiz sonucunda elde edilen çıktı aşağıdaki şekildedir.

![](01.png)

Analiz sonucunda keyv/packages/mysql/src/index.ts dosyası içerisinde, 215 numaralı satırda bulunan has metodunun SQL Injection zafiyetine sahip olduğu, bu zafiyetin nasıl kötüye kullanabileceği ve zafiyetin nasıl çözülebileceğine ait örnekler Gemini tarafından oluşturulmuştur.

SQL Injection zafiyetine neden olan has metodu içeriği aşağıdaki şekildedir.

![](02.png)

Çok net bir zafiyet olsada çalışma zamanında zafiyetin kötüye kullanılabileceği göstermek adına aşağıdaki gibi ufak bir demo uygulaması geliştirilmiştir. Demo uygulama içerisinde [@keyv/mysql](https://www.npmjs.com/package/@keyv/mysql) kütüphanesinin entegre edilmesi ve zafiyete neden olan has metodunun kullanılması sağlandı.

![](03.png)

Zafiyete neden olan SQL sorgusu aşağıdaki şekildedir.

```sql
SELECT EXISTS ( SELECT * FROM ${this.opts.table!} WHERE id = '${key}' )
```

${key} değeri olarak göndereceğimiz ve mevcut sorgu yapısını değiştireceğimiz payload ise `1') UNION (SELECT SLEEP(10));-- ` şeklindedir.

Gönderilen payload sonucunda oluşacak SQL sorgusunun aşağıdaki gibi olmasını bekliyoruz.

```sql
SELECT EXISTS ( SELECT * FROM keyv WHERE id = '1' ) UNION (SELECT SLEEP(10));--
```

BurpSuite üzerinden ilgili payload değerini gönderdiğimizde sunucu cevabının >10 saniye sonrasında iletildiğini görmekteyiz.

![](04.png)

## Exploit

search parametresi yardımıyla göndermiş olduğumuz SQL sorgularının hedef sistem üzerinde çalıştığını doğrulamıştık. Bir sonraki adım olarak Keyv tarafından varsayılan olarak oluşturulan keyv tablosu dışında bir tablo üzerinden veri okuma işlemi gerçekleştirilmesi sağlanacaktır.

İlk olarak demo amaçlı oluşturduğumuz veri tabanı içerisine users isimli bir tablo ekliyoruz.

![](05.png)

Daha sonra aşağıdaki sorgu yardımıyla users tablosu içerisinden name değerinin karakter-karakter okumasını gerçekleştiriyoruz.

```
search=1%27+)+OR+IF(SUBSTR((SELECT+name+from+test_db%2eusers+LIMIT+1),1,1)%3d%27x%27,SLEEP(10),0)%3b--+
```

![](06.png)

Ekranda görüleceği üzere anethole değerine ait her karakter sonucunda sunucunun cevap süresinin >10 saniye olduğu görülmektedir.

## Çözüm

Tespit edilen zafiyetin bildirilmesinin ardından geliştirme ekibi hızlı bir şekilde giderilmesini sağladılar — [Commit](https://github.com/jaredwray/keyv/pull/1724/commits/b62579597a02a58da6abe5bb73f0ee80525fe2fb)

Dinamik olarak yazılmış olan SQL sorgusu, prepared-statement yöntemi kullanılarak yeniden yazılmış ve zafiyet giderilmesi sağlanmıştır.

**Note:** İlgili zafiyetin, 2022 yılında eklenen bir geliştirme sonucunda ortaya çıktığı gözlemlenmiştir — [Vulnerable PR](https://github.com/jaredwray/keyv/pull/244)

## Zafiyetten Etkilenen Sürümler

2.1.19 sürümünün altındaki tüm @keyv/mysql paketleri bu zafiyetten etkilenmektedir.

---
