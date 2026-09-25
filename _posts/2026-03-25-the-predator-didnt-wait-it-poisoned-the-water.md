---
title: The Predator Didn’t Wait — It Poisoned the Water
date: 2026-03-25 16:21:20 +0300
categories: [supply-chain]
tags: [supply-chain-attack, watering-hole, teampcp]
description: Watering Hole ve yazılım tedarik zinciri saldırılarının birleştiği TeamPCP saldırılarının incelenmesi.
media_subpath: /assets/img/posts/the-predator-didnt-wait-it-poisoned-the-water
image: cover.jpg
canonical_url: https://blog-tr.peakcyber.com/the-predator-didnt-wait-it-poisoned-the-water-34b081c8ec47
---

Bir çoğumuz belgesellerde yırtıcı hayvanların su içme alanlarında kurdukları tuzakları ve bu tuzaklar sayesinde su içmeye gelen diğer hayvanları avladıklarını görmüşüzdür. Siber güvenlik dünyasında da bu yaklaşımdan yola çıkılarak kullanılan bir teknik mevcut. Watering Hole Attack.

Watering Hole Attack yaklaşımında saldırgan tarafından **hedeflenen** organizasyon/kurum kullanıcılarının erişim sağladığı sistemler, yapılar, araçlar tespit edilmektedir. Daha sonra saldırgan tespit ettiği bu sistemleri ele geçirerek veya kullanarak asıl hedefi olan organizasyon/kurum kullanıcılarına erişmeyi hedefler.

Tek bir saldırı ile birden fazla kurum/kullanıcının etkilendiği saldırı yöntemlerinden bir diğeri de Software Supply Chain saldırılarıdır. Saldırganlar yaygın kullanıma sahip açık/kapalı kaynak kodlu kütüphaneler içerisine ekledikleri zararlılar sayesinde hedef sistemler üzerinden uzaktan bağlantı, bilgi çalma ve benzeri birçok aksiyon gerçekleştirilebilmektedir.

Peki iki saldırının bir araya gelmesi nelere sebep olabilir? Şubat ve Mart 2026 ayları bu sorunun cevabını anlamayabilmemiz için bir çok örnek sundu.

## Şubat

Şubat ayı sonlarında yayınlanan [HackerBot-Claw](https://news.ycombinator.com/item?id=47205101) isimli bir AI bot hesabı, organizasyonların Github Action yapıları üzerinden GITHUB_TOKEN ve benzeri hassas verilerin yanlış yapılandırmalar sonucunda ele geçirilebildiğini gösterdi. Bu doğrultuda Microsoft, Data Dog ve Aqua Security — Trivy gibi organizasyonlara ait PAT (Personel Access Token) değerlerinin elde edilebildiği tespit edildi.

## Mart

19 Mart tarihine geldiğimizde Aqua Security — Trivy aracına ait Github, DockerHub ve ECR adresleri üzerinden, üzerinde çalıştığı sisteme ait hassas verileri, parolaları, ssh anahtarlarını, kripto cüzdanlarını ve benzeri bilgileri toplayan, bu bilgileri şifreleyip hedef C2 sunucusuna ileten zararlı bir güncelleme yayınladığı tespit edilmiştir — [GHSA-69fq-xp46-6x23](https://github.com/aquasecurity/trivy/security/advisories/GHSA-69fq-xp46-6x23)

23 Mart tarihinde hedef yine bir siber güvenlik ürünü Checkmarx — Kics. Saldırganlar Trivy örneği ile tamamen aynı şekilde işleyen bir zararlı içerik ile hassas bilgileri hedef almıştır — [Kicks Issue 152](https://github.com/Checkmarx/kics-github-action/issues/152)

> Tüm bu saldırılar TeamPCP adı verilen bir tehdit aktör tarafından gerçekleştirilmiştir. Aktörün [cloud-native](https://www.elastic.co/security-labs/teampcp-container-attack-scenario) alt yapılar üzerine yoğunlaşan çalışmaları olduğu bilinmektedir.

24 Mart, bu kez kurban [LiteLLM](https://github.com/BerriAI/litellm/issues/24518#issuecomment-4119055562). Trivy ve Checkmarx ile tamamen aynı saldırı senaryosu, üzerinde çalıştığın tüm ortama ait hassas bilgileri toplayıp, şifreleyerek ve C2 sunusuna göndermektedir — [LiteLLM Issue 24512](https://github.com/BerriAI/litellm/issues/24512)

Klasik bir Watering Hole saldırısı özü itibarıyla pasif bir yapıya sahiptir, avcı tuzağı kurar ve bekler. Ancak TeamPCP olayları, bu yaklaşımın evrilmiş bir versiyonunu gözler önüne sermektedir. Burada saldırgan, kurbanların ele geçirilmiş bir kaynağa rastlantısal olarak ulaşmasını beklemek yerine, kurbanların zaten kullanmak zorunda olduğu araçların tedarik zincirine dahil olmayı tercih ederek, kurbanlarına kaçmak için fırsat tanımayacak bir yapı dizayn etmiş oldu.

## Özet

Merkezi olarak konumlandırılan teknolojiler içerisinde meydana gelebilecek olası yanlış konfigürasyonların çok büyük boyutlarda kişi ve kurum güvenliğini tehdit edebileceği görülmektedir.

Günümüz dünyasında siber saldırıların kaçabileceğimiz, uzak durabileceğimiz bir kavram olmadığını bir kez daha görmüş bulunuyoruz. İstesekte/istemesekte siber saldırılar ile karşılaşacağız ve hack’leneceğiz. Tamda bu nedenle önemli olanın hacking vakası yaşandığında hızlı şekilde tespit ve aksiyon alabilecek yetenek ve yetkinlikte olmamız olduğu görülmektedir.

---
