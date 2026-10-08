# Dağıtık Sistemlerde Konfigürasyon, Servis Keşfi ve Gizli Bilgi Yönetimi

Bu doküman; etcd, Consul, ZooKeeper, Kubernetes ConfigMap/Secret, Spring Cloud Config, HashiCorp Vault, AWS Secrets Manager, AWS Parameter Store ve Google Cloud Secret Manager araçlarını tek bir çatı altında anlatır.

---

## İçindekiler

**Giriş**
- [Bu araçlara neden ihtiyacımız var?](#bu-araçlara-neden-ihtiyacımız-var)
- [Dağıtık sistem nedir?](#dağıtık-sistem-nedir)
- [Konfigürasyon nedir?](#konfigürasyon-nedir)
- [Secret nedir? Secret yönetimi nedir?](#secret-nedir-secret-yönetimi-nedir)
- [Konfigürasyon ile secret arasındaki fark](#konfigürasyon-ile-secret-arasındaki-fark)

**A. Koordinasyon ve servis keşfi**

1. [etcd](#1-etcd)
   - [1.1 Genel bakış](#11-genel-bakış)
   - [1.2 Temel özellikleri](#12-temel-özellikleri)
   - [1.3 Raft algoritması](#13-raft-algoritması)
   - [1.4 Lider seçimi](#14-lider-seçimi)
   - [1.5 Disk ve performans](#15-disk-ve-performans)
   - [1.6 etcd ve Kubernetes](#16-etcd-ve-kubernetes)
   - [1.7 etcd'yi kimler kullanır?](#17-etcdyi-kimler-kullanır)
   - [1.8 Özet tablo](#18-özet-tablo)
2. [Consul](#2-consul)
   - [2.1 Genel bakış](#21-genel-bakış)
   - [2.2 Consul ile etcd/benzeri araçlar arasındaki fark](#22-consul-ile-etcdbenzeri-araçlar-arasındaki-fark)
   - [2.3 Servis discovery: servise nasıl ulaşılır?](#23-servis-discovery-servise-nasıl-ulaşılır)
   - [2.4 Consul'un özellikleri](#24-consulun-özellikleri)
   - [2.5 Consul cluster mimarisi](#25-consul-cluster-mimarisi)
   - [2.6 Consul Agent](#26-consul-agent)
   - [2.7 Önemli ayrım: state kimde?](#27-önemli-ayrım-state-kimde)
   - [2.8 Özet](#28-özet)
3. [Apache ZooKeeper](#3-apache-zookeeper)
   - [3.1 Apache ZooKeeper nedir?](#31-apache-zookeeper-nedir)
   - [3.2 Mimari yapı](#32-mimari-yapı)
   - [3.3 Veri modeli: znode kavramı](#33-veri-modeli-znode-kavramı)
   - [3.4 Znode türleri](#34-znode-türleri)
   - [3.5 Avantajları ve kullanım alanları](#35-avantajları-ve-kullanım-alanları)
   - [3.6 Lider seçimi](#36-lider-seçimi)
   - [3.7 2026 yılında tespit edilen güvenlik açıkları](#37-2026-yılında-tespit-edilen-güvenlik-açıkları)

**B. Kubernetes'te konfigürasyon ve secret**

4. [Kubernetes ConfigMap, Pod, Deployment ve HPA](#4-kubernetes-configmap-pod-deployment-ve-hpa)
   - [4.1 ConfigMap nedir?](#41-configmap-nedir)
   - [4.2 ConfigMap verilerinin kullanılması](#42-configmap-verilerinin-kullanılması)
   - [4.3 Pod](#43-pod)
   - [4.4 Kubernetes Deployment](#44-kubernetes-deployment)
   - [4.5 Deployment ve ConfigMap birlikte nasıl çalışır?](#45-deployment-ve-configmap-birlikte-nasıl-çalışır)
   - [4.6 Kullanıcı isteği geldiğinde ne olur?](#46-kullanıcı-isteği-geldiğinde-ne-olur)
   - [4.7 Replica sayısının güncellenmesi](#47-replica-sayısının-güncellenmesi)
   - [4.8 HPA (Horizontal Pod Autoscaler)](#48-hpa-horizontal-pod-autoscaler)
5. [Kubernetes Secret](#5-kubernetes-secret)
   - [5.1 Kubernetes Secret nedir?](#51-kubernetes-secret-nedir)
   - [5.2 Kim tarafından, ne amaçla geliştirildi?](#52-kim-tarafından-ne-amaçla-geliştirildi)
   - [5.3 Secret nasıl kullanılır?](#53-secret-nasıl-kullanılır)
   - [5.4 Secret'lar nerede saklanır?](#54-secretlar-nerede-saklanır)
   - [5.5 `data` ve `stringData` farkı](#55-data-ve-stringdata-farkı)
   - [5.6 ConfigMap'e göre güvenlik avantajı: RBAC](#56-configmape-göre-güvenlik-avantajı-rbac)
   - [5.7 etcd'de encryption at rest](#57-etcdde-encryption-at-rest)

**C. Uygulama seviyesinde konfigürasyon**

6. [Spring Cloud Config](#6-spring-cloud-config)
   - [6.1 Neden ihtiyaç var?](#61-neden-ihtiyaç-var)
   - [6.2 Spring Cloud Config Server nedir?](#62-spring-cloud-config-server-nedir)
   - [6.3 Nasıl çalışır?](#63-nasıl-çalışır)
   - [6.4 Ön bilgi: Bean nedir?](#64-ön-bilgi-bean-nedir)
   - [6.5 Çalışma anında yapılandırmayı yenileme](#65-çalışma-anında-yapılandırmayı-yenileme)
   - [6.6 Client'ın Config Server'ı bulma yöntemleri](#66-clientın-config-serverı-bulma-yöntemleri)
   - [6.7 Kubernetes kullanılıyorsa?](#67-kubernetes-kullanılıyorsa)

**D. Secret yönetimi**

7. [HashiCorp Vault](#7-hashicorp-vault)
   - [7.1 Vault nedir?](#71-vault-nedir)
   - [7.2 Kim tarafından, ne zaman geliştirildi?](#72-kim-tarafından-ne-zaman-geliştirildi)
   - [7.3 Temel özellikleri](#73-temel-özellikleri)
   - [7.4 Dinamik gizli bilgiler nasıl çalışır?](#74-dinamik-gizli-bilgiler-nasıl-çalışır)
   - [7.5 Sızıntı problemi ve dinamik gizli bilgiler](#75-sızıntı-problemi-ve-dinamik-gizli-bilgiler)
   - [7.6 Entegrasyonlar](#76-entegrasyonlar)
   - [7.7 Nasıl çalışır?](#77-nasıl-çalışır)
   - [7.8 Depolama](#78-depolama)
8. [AWS Secrets Manager](#8-aws-secrets-manager)
   - [8.1 AWS Secrets Manager nedir?](#81-aws-secrets-manager-nedir)
   - [8.2 Temel özellikleri](#82-temel-özellikleri)
   - [8.3 Nasıl çalışır?](#83-nasıl-çalışır)
   - [8.4 Fiyatlandırma](#84-fiyatlandırma)
   - [8.5 HashiCorp Vault ile fark](#85-hashicorp-vault-ile-fark)
   - [8.6 Parameter Store ile fark](#86-parameter-store-ile-fark)
   - [8.7 Spring Cloud Config ile ilişkisi](#87-spring-cloud-config-ile-ilişkisi)
9. [AWS Parameter Store](#9-aws-parameter-store)
   - [9.1 AWS Parameter Store nedir?](#91-aws-parameter-store-nedir)
   - [9.2 Temel özellikleri](#92-temel-özellikleri)
   - [9.3 Avantajları](#93-avantajları)
   - [9.4 Standart ve gelişmiş (Advanced) katman](#94-standart-ve-gelişmiş-advanced-katman)
   - [9.5 Nasıl çalışır?](#95-nasıl-çalışır)
   - [9.6 Fiyatlandırma](#96-fiyatlandırma)
   - [9.7 Secrets Manager ile fark](#97-secrets-manager-ile-fark)
   - [9.8 Spring ile ilişkisi](#98-spring-ile-ilişkisi)
10. [Google Cloud Secret Manager](#10-google-cloud-secret-manager)
    - [10.1 Secret Version (gizli bilgi sürümleri)](#101-secret-version-gizli-bilgi-sürümleri)
    - [10.2 Encryption (şifreleme)](#102-encryption-şifreleme)
    - [10.3 IAM (Identity and Access Management)](#103-iam-identity-and-access-management)
    - [10.4 Replication (çoğaltma)](#104-replication-çoğaltma)
    - [10.5 Secret Manager ile etcd ve Consul'un birlikte kullanılması](#105-secret-manager-ile-etcd-ve-consulun-birlikte-kullanılması)

**Ek:** [Kaynakça](#kaynakça)


> 💡 **Raft** algoritması bu dokümanda birkaç yerde karşımıza çıkar. Ayrıntılı anlatımı [etcd](#1-etcd) bölümündedir; [Consul](#2-consul) ve [Vault](#7-hashicorp-vault) da Raft kullanır.

---

## Bu araçlara neden ihtiyacımız var?

Tek bir makinede çalışan basit bir uygulamanın ayarları bir dosyada durabilir. Ancak uygulama onlarca mikroservise ve yüzlerce makineye bölündüğünde şu sorular ortaya çıkar:

1. **Ayarlar nerede tutulacak?** Her servisin kendi ayar dosyasını taşıması, servis sayısı arttıkça yönetilemez hale gelir.
2. **Servisler birbirini nasıl bulacak?** Servislerin birbirinin adresini elle bilmesi hem bağımlılığı hem karmaşıklığı artırır.
3. **Birden fazla makine nasıl koordine olacak?** Örneğin aynı anda yalnızca tek bir makinenin işlem yapması (kilit) ya da bir lider seçilmesi gerekebilir.
4. **Şifreler ve API anahtarları nasıl güvenli saklanacak?** Bu bilgileri kodun ya da config dosyasının içine yazmak güvenli değildir.


Bu dokümandaki araçlar bu soruların farklı kısımlarına cevap verir.

### Dağıtık sistem nedir?

**Dağıtık sistem**, birden fazla bilgisayarın (düğümün) ağ üzerinden haberleşerek, dışarıdan tek bir sistem gibi görünecek şekilde birlikte çalışmasıdır. Birbiriyle konuşan mikroservisler, bir Kubernetes kümesi ya da birden fazla düğümden oluşan bir veritabanı kümesi buna örnektir.

Avantajı, yükün makineler arasında paylaştırılabilmesi (ölçeklenebilirlik) ve bir makine çökse bile sistemin çalışmaya devam edebilmesidir. Bedeli ise yönetim karmaşıklığıdır: makineler birbirinin belleğini göremez, aralarındaki ağ her zaman güvenilir değildir ve her makinenin kendi durumu vardır. Yukarıdaki dört soru (ayarlar, servis keşfi, koordinasyon, gizli bilgiler) tam olarak bu karmaşıklıktan doğar.

### Konfigürasyon nedir?

**Konfigürasyon**, uygulamanın kodunu değiştirmeden davranışını belirleyen ayarlardır. Port numarası, log seviyesi, veritabanı adresi ya da bir özelliğin açık/kapalı olması buna örnektir.

Bu değerler koda gömülmez, koddan ayrı tutulur. Böylece aynı uygulama farklı ortamlarda (geliştirme, test, production) farklı ayarlarla çalışabilir ve bir ayarı değiştirmek için uygulamayı yeniden derlemek gerekmez. Tek makinede bu ayarlar bir dosyada durabilir; dağıtık sistemde ise yüzlerce servisin ayarını tek tek dosyalarda tutmak yönetilemez hale geldiği için bu dokümandaki araçlara ihtiyaç duyulur.

### Secret nedir? Secret yönetimi nedir?

**Secret (gizli bilgi)**, bir sisteme ya da veriye erişmeyi sağlayan ve başkalarının eline geçmemesi gereken hassas bilgidir. Dijital sertifikalar, veritabanı kimlik bilgileri, parolalar ve API şifreleme anahtarları buna örnektir. Bu bilgi kötü niyetli birinin eline geçerse, o kişi bizim yerimize sisteme girebilir.

Uygulamaların çalışabilmesi için bu bilgilere ihtiyacı vardır. Örneğin bir uygulama veritabanına bağlanmak için şifreye muhtaçtır. Bu şifrenin nerede saklanacağı, uygulamaya nasıl ulaştırılacağı, kimlerin görebileceği ve ne zaman değiştirileceği ise ayrı bir problemdir. **Secret yönetimi (secrets management)**, gizli bilgilerin güvenli şekilde saklanması, dağıtılması, erişiminin kontrol edilmesi ve düzenli olarak yenilenmesi işlerinin tamamına verilen isimdir.

Secret'ı kodun içine ya da config dosyasına yazmak en basit yol gibi görünür ama güvenli değildir (Git'e yüklenebilir, loglara düşebilir, değiştirmek zordur). Vault, Secrets Manager ve Secret Manager gibi araçlar bu probleme çözüm olarak geliştirilmiştir.

### Konfigürasyon ile secret arasındaki fark

Dokümanın ana eksenlerinden biri bu ayrımdır:

| | Konfigürasyon | Secret (gizli bilgi) |
|---|---|---|
| Örnek | `PORT=8080`, `LOG_LEVEL=INFO`, `DB_HOST=postgres` | `DB_PASSWORD`, `API_KEY`, `JWT_SECRET` |
| Gizli mi? | Hayır | Evet |
| Sızarsa | Genellikle sorun olmaz | Saldırgan bizim yerimize sisteme girebilir |
| Uygun araçlar | etcd, Consul, ZooKeeper, ConfigMap, Spring Cloud Config, Parameter Store | Vault, Secrets Manager, GCP Secret Manager, Kubernetes Secret |

---

# A. Koordinasyon ve Servis Keşfi

---

## 1. etcd

### 1.1 Genel bakış

**etcd**, dağıtık sistemlerin çalışması için gereken verileri (yapılandırma, durum bilgisi vb.) saklayan, açık kaynaklı bir **anahtar-değer (key-value) veri deposudur.**

#### İsmi nereden geliyor?

| Parça | Anlamı |
|---|---|
| `etc` | Linux'taki `/etc` klasörü (yapılandırma dosyaları) |
| `d` | **d**istributed (dağıtık) |

`/etc` tek bir makinenin ayarlarını tutar. etcd ise büyük ölçekli, çok makineli sistemlerin ayarlarını tutar. Yani etcd, "dağıtık /etc"dir.

#### Kısa tarihçe

- etcd, **CoreOS** tarafından Container Linux'un birden fazla kopyasını eş zamanlı koordine etmek ve uygulamaların kesintisiz çalışmasını sağlamak için **Raft** algoritması temel alınarak geliştirildi.
- Aralık 2018'de **CNCF**'ye (Cloud Native Computing Foundation) bağışlandı. CNCF, kâr amacı gütmeyen ve tarafsız bir kuruluştur.
- CoreOS daha sonra Red Hat tarafından satın alındı.

### 1.2 Temel özellikleri

- **Replicated (çoğaltılmış):** Her düğüm, veri deposunun tamamına sahiptir.
- **Consistent (tutarlı):** Her okuma işlemi en güncel veriyi döndürür.
- **Yüksek erişilebilirlik:** Düğümlerin çoğunluğu ayakta kaldığı sürece, ağ veya donanım sorunlarında bile kesintisiz çalışabilir.
- **Basit kullanım:** Standart HTTP/JSON araçlarıyla okuma ve yazma yapılabilir. İç iletişimde **gRPC** kullanılır.
- **Güvenlik:** İsteğe bağlı SSL istemci sertifikası ile kimlik doğrulama yapılabilir. SSL tamamen opsiyoneldir.

> **gRPC nedir?** Google'ın geliştirdiği, farklı bilgisayarlar veya servisler arasında hızlı ve güvenli iletişim kurmayı sağlayan açık kaynaklı bir **RPC (Remote Procedure Call)** çerçevesidir. RPC ise bir programın, başka bir makinedeki fonksiyonu sanki kendi makinesindeymiş gibi çağırmasını sağlayan yöntemdir.

### 1.3 Raft algoritması

etcd, **Raft** konsensüs algoritması üzerine inşa edilmiştir. Bir etcd kümesinde aynı anda yalnızca:

- **1 Lider (Leader)** bulunur,
- diğer tüm düğümler **Takipçi (Follower)** olur.

#### Yazma işlemi nasıl olur?

Örnek: Küme 4 düğümden oluşuyor ve hepsinde `7` değeri var. Client, bu değeri `17` yapmak istiyor.

```text
Client ──(7 → 17 yap)──▶ Lider
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
      Takipçi 1       Takipçi 2       Takipçi 3
     (7 → 17 yapar)  (7 → 17 yapar)  (7 → 17 yapar)
          │               │               │
          └───────────────┼───────────────┘
                          ▼
                 Lider onayları alır
                          │
                 Kendi verisini günceller
                          │
Client ◀──(işlem başarılı)─┘
```

Adım adım:

1. Client lider ile iletişime geçer.
2. Lider **kendi verisini hemen değiştirmez**, isteği takipçilere iletir.
3. Takipçiler değeri günceller ve lidere bildirir.
4. Lider onayları aldıktan sonra kendi verisini günceller.
5. Client'a "işlem başarılı" yanıtı gönderilir.

> 💡 Pratikte Raft, tüm düğümleri beklemez. Düğümlerin **çoğunluğunun (quorum)** onayı yeterlidir. Bu yüzden küme birkaç düğümünü kaybetse bile çalışmaya devam edebilir. Örneğin 4 düğümlü bir kümede çoğunluk 3'tür ve çoğunluk işlemi onayladıysa işlem tamamlanmış sayılır. Hata toleransı hesabında çoğunluk N/2 + 1'dir (4 düğümde 3). Dolayısıyla 3 düğümün çalışması sistemi ayakta tutar ve yalnızca 1 düğüm kaybı tolere edilir. 3 düğümlü küme de aynı şekilde 1 düğüm kaybını tolere ettiği için 4. düğüm ek bir hata toleransı sağlamaz. Bu yüzden pratikte genellikle tek sayıda (3 veya 5) düğüm tercih edilir. 2 veya daha fazla düğüm kaybedilirse çoğunluk sağlanamaz ve küme çalışmaz.

#### Okuma işlemi nasıl olur?

**Soru:** Client henüz güncellenmemiş bir takipçiye okuma isteği gönderirse ne olur?

Takipçi, bu isteği tek başına cevaplamaya yetkili olmadığını bilir. İsteği **lidere iletir**, lider güncel değerin `17` olduğunu bildirir ve takipçi de Client'a `17` yanıtını verir. Böylece Client hiçbir zaman eski veriyi görmez, yani **tutarlılık** korunur.

#### Log yapısı

Yapılan tüm değişiklikler Raft sayesinde **log** olarak tutulur. Tüm düğümlerde loglar **aynı sırayla** olmalıdır:

```text
Düğüm A:             Düğüm B:
1. SET x=10          1. SET x=10
2. SET y=20          2. SET y=20
3. SET x=15          3. SET x=15
```

Yazma isteği geldiğinde lider işlemi önce kendi **Raft loguna** ekler, log girdisini takipçilere **çoğaltır** ve çoğaltma doğrulandıktan sonra veriyi **BoltDB tabanlı MVCC veritabanına** kaydeder. Bu sayede konsensüs sağlanmadan hiçbir yazma işlemi kalıcı olmaz.

### 1.4 Lider seçimi

#### Lider nasıl seçilir?

1. Başlangıçta tüm düğümler **Follower** durumundadır ve liderden mesaj (heartbeat) bekler.
2. Belirli bir süre boyunca mesaj gelmezse, düğüm liderde sorun olduğunu anlar ve **Candidate (aday)** durumuna geçer.
3. Aday, diğer düğümlerden oy ister ("ben lider olmaya hazırım"). **Düğümlerin çoğunluğunun (quorum) oyunu alan düğüm lider olur.**

Her düğümün zaman aşımı (timeout) süresi birbirinden farklıdır. Süresi ilk dolan düğüm adaylığını ilk duyurur ve lider olma şansı artar. Böylece aynı anda birden fazla aday çıkıp oyların bölünmesi azaltılır.

Lider seçildikten sonra periyodik olarak takipçilere heartbeat gönderir. Bu heartbeat'ler takipçilerin zaman aşımı sayaçlarını sıfırlar, böylece yeni seçim başlamaz. Lider çökerse takipçiler heartbeat alamaz ve yukarıdaki seçim süreci yeniden işleyerek yeni bir lider seçilir.

#### Term (dönem) kavramı

Raft'ta her liderlik dönemi bir **term** numarasıyla tutulur (1. lider, 2. lider...).

Eski lider düğümün sorunu çözülüp sisteme geri döndüğünde, ortamda daha yüksek term'e sahip yeni bir lider olduğunu görür ve **Follower olarak kalır.**

### 1.5 Disk ve performans

etcd saniyede yaklaşık **10.000 yazma işlemi** yapabilir ve bunları **diske kaydeder.** Dolayısıyla performans, her düğümün disk hızına doğrudan bağlıdır.

| Konu | Öneri |
|---|---|
| Disk türü | En az **SSD** olmalı |
| Ağ tabanlı disk (network storage) | ❌ **Uygun değil** |

#### Network tabanlı disk neden sorun?

Disk erişim süresi şu gecikmelerin toplamına dönüşür:

```text
Disk latency
+ Network latency
+ Network congestion
+ Storage system latency
```

Yüksek veya değişken I/O gecikmeleri etcd'nin performansını ve **küme kararlılığını** bozabilir (örneğin heartbeat'ler gecikir, gereksiz lider seçimleri başlar).

### 1.6 etcd ve Kubernetes

Kubernetes (K8s), konteynerleştirilmiş uygulamaların dağıtımını, ölçeklenmesini ve yönetimini otomatikleştiren açık kaynaklı bir **orkestrasyon platformudur.** Yüzlerce veya binlerce konteyneri elle yönetmek imkansız olduğu için bir orkestra şefi gibi çalışır.

#### Kubernetes kümesinin bileşenleri

- **Control Plane (Kontrol Düzlemi):** Kümenin beynidir. Karar alma, planlama ve genel durumu yönetme işlerini yapar.
- **Worker Node (Çalışan Düğüm):** Uygulamaları çalıştıran fiziksel veya sanal makinelerdir.
- **Pod:** Kubernetes'in en küçük yapı taşıdır, uygulamaların çalıştığı birimdir ([4.3 Pod bölümünde](#43-pod) ayrıntılı anlatılıyor).

#### etcd'nin rolü

etcd, Kubernetes için şunları saklar:

- **Yapılandırma verileri** (istenen durum)
- **Durum verileri** (mevcut durum)
- **Meta veriler**

Bir **izleme (watch)** fonksiyonu sayesinde Kubernetes, istenen durum ile mevcut durumu birbirinden ayrı tutup karşılaştırabilir. İkisi arasında fark oluştuğunda küme buna göre yeniden yapılandırılır. Akış şöyledir:

```text
etcd (değişiklik olur) ──▶ Kubernetes API ──▶ Küme buna göre yeniden yapılandırılır
```

### 1.7 etcd'yi kimler kullanır?

- **Kubernetes**
- **Rook**
- **CoreDNS**
- **M3**

### 1.8 Özet tablo

| Kavram | Açıklama |
|---|---|
| etcd | Dağıtık, tutarlı, açık kaynaklı anahtar-değer deposu |
| Algoritma | Raft |
| Roller | Leader, Follower, Candidate |
| Replicated | Her düğümde verinin tam kopyası vardır |
| Consistent | Her okuma en güncel veriyi verir |
| Term | Kaçıncı liderlik döneminde olunduğunu gösteren sayaç |
| Depolama | BoltDB tabanlı MVCC veritabanı |
| İletişim | gRPC (HTTP/JSON ile de erişilebilir) |
| Disk | Minimum SSD, ağ tabanlı disk önerilmez |
| Ana kullanım | Kubernetes'in yapılandırma ve durum verileri |

---

## 2. Consul

### 2.1 Genel bakış

**Consul**, dağıtık ortamlarda servislerin **kaydedilmesini, bulunmasını (discovery), sağlık durumunun izlenmesini ve güvenli iletişimini** sağlayan bir **kontrol düzlemidir (control plane)**.

Fiziksel sunucular, bulut örnekleri, sanal makineler veya konteynerler gibi **düğüm (node) kümeleri** üzerinde çalışan dağıtık bir sistemdir. İçinde servisler ve IP adresleri için merkezi bir **kayıt defteri (registry)** tutar.

> **Neden kullanılır?** Mikroservis mimarisinde servislerin birbiriyle konuşması hem bağımlılığı hem karmaşıklığı artırır. Servislerin birbirinin adresini elle bilmesi yerine, adresleri Consul'a sorarlar.

### 2.2 Consul ile etcd/benzeri araçlar arasındaki fark

| | etcd gibi araçlar | Consul |
|---|---|---|
| Odak | Konfigürasyon / anahtar-değer verisi | **Servis bilgisi** (adres, port) + KV |
| Servis kaydı | Yerleşik değil (elle yapılır) | Var, yerleşik |
| Health check | Yerleşik değil | Var, yerleşik |
| Discovery | Yerleşik değil | HTTP ve DNS ile |

Yani Consul'da sadece konfigürasyon dosyaları değil, **servislerin adresi ve port numarası** da saklanır. İki servisin birbiriyle konuşması gerektiğinde ilgili adresler Consul üzerinden paylaşılır.

### 2.3 Servis discovery: servise nasıl ulaşılır?

Bir servis Consul'a **iki yolla** ulaşabilir: **HTTP** ve **DNS**.

#### HTTP ile

```text
Order Service
      │
      │ HTTP
      ↓
Consul API
      │
      ↓
Payment Service adresi
```

#### DNS ile

Uygulama Consul'un DNS özelliğini kullanabilir:

```text
payment.service.consul
          ↓
       Consul
          ↓
192.168.1.20:8080
```

#### Birden fazla instance örneği

Payment Service'in 3 instance'ı olsun:

```text
Payment-1 → 192.168.1.20:8080
Payment-2 → 192.168.1.21:8080
Payment-3 → 192.168.1.22:8080
```

Hepsi Consul'a **register** edilir:

```text
             Consul
          /     |     \
         ↓      ↓      ↓
     Payment1 Payment2 Payment3
```

Order Service *"Payment Service nerede?"* diye sorduğunda Consul uygun instance'ı bulmasına yardımcı olur.

#### Bir instance çökerse?

Health check başarısız olursa:

```text
Payment-1 ❌
Payment-2 ✅
Payment-3 ✅
```

Consul sağlıksız instance'ı discovery sonucuna dahil etmez. Böylece trafik çöken servise yönlendirilmez.

### 2.4 Consul'un özellikleri

1. **Service Discovery:** Servisleri kaydetme ve bulma
2. **Health Checking:** Servislerin ve node'ların sağlık kontrolü
3. **KV Store:** Anahtar-değer deposu
4. **Secure Service Communication:** Servisler arası güvenli iletişim
5. **Multi Datacenter:** Birden fazla veri merkezi desteği

#### Health checking detayı

Consul client'ları iki tür kontrol yapabilir:

- **Servis ile ilgili:** Web sunucusu 200 dönüyor mu?
- **Local node ile ilgili:** Bellek kullanımı %90'ın altında mı?

Bu bilgi iki amaçla kullanılır:

- Operatör, cluster'ın sağlığını **izlemek** için
- Discovery bileşenleri, **sağlıksız host'lara trafik yönlendirmemek** için

### 2.5 Consul cluster mimarisi

Consul cluster, **Server** ve **Client** olmak üzere iki tür yapıdan oluşur.

```text
                 Consul Cluster
              ┌─────────────────┐
              │  Server  Server │
              │      Server     │
              └────────┬────────┘
                       │
              ┌────────┴─────────┐
              ↓                  ↓
        Consul Client      Consul Client
              │                  │
        Order Service      Payment Service
```

#### Consul Server

- Anahtar-değer verilerini, servis ve node bilgilerini **saklar ve replike eder**. Yani bilgi birden fazla Consul server'da tutulur.
- Tek server de kullanılabilir, ancak **lider seçiminin sağlıklı çalışması** ve **hata (failure) senaryoları** için birden fazla server önerilir.
- Raft çoğunluk (quorum) ile çalıştığı için production'da genellikle **3 veya 5 server** önerilir ([etcd](#1-etcd) bölümündeki quorum açıklamasına bakınız). Çift sayıdaki server ek bir hata toleransı sağlamaz.

> **Failure tolerance:** Sistemde bazı parçalar bozulduğunda sistemin çalışmaya devam edebilme kapasitesidir.

#### Consul Client

- Node'un ve üzerindeki servislerin **sağlık durumunu kontrol eder**.
- Görevi, bilgiyi **en hızlı biçimde server'a iletmektir**.
- Veriyi saklama (data store) gibi kritik görevlerden sorumlu **değildir**.
- Uygulamalar discovery sorgularını genellikle yerel client agent'a yapar, o da bunu server'a iletir. Yani client discovery'de **aracı** olur, veri sahibi olmaz.

### 2.6 Consul Agent

Server ve Client aslında **iki farklı program değildir**. İkisinde de çalışan yapı **Consul Agent**'tır.

> **Consul Agent**, bir node üzerinde sürekli çalışan Consul sürecidir.
> **Node nedir?** Consul açısından Consul Agent'ın çalıştığı makine/instance.

```text
                 Consul Agent
                      │
             ┌────────┴────────┐
             │                 │
        Client Mode       Server Mode
             │                 │
             ↓                 ↓
      Consul Client      Consul Server
```

Yani Consul tek bir programdır; bir makinede çalıştırıldığında Agent olur ve **Client modunda** ya da **Server modunda** çalışır.

#### Neden her node'da Agent var?

Consul'un bütün işlerini tek bir merkezi server'a yaptırmak yerine, her node'da bir Agent çalıştırılarak mimari **dağıtılır**.

```text
                    Consul Servers
               ┌───────┬───────┬───────┐
               │       │       │       │
              S1      S2      S3
               └───────┴───────┴───────┘
                       ▲
                       │
                 Consul Cluster
                       │
          ┌────────────┼────────────┐
          │            │            │
          ↓            ↓            ↓
      Client A     Client B     Client C
```

Agent'ın görevleri: servisleri **register** etmek, **health check** çalıştırmak, Consul cluster'ıyla **iletişim** kurmak, **service discovery** işlemlerine aracılık etmek ve **local node** hakkında bilgi tutmak.

### 2.7 Önemli ayrım: state kimde?

```text
Order Service
      ↓
Consul Client Agent
      ↓
Consul Server
```

- **Client Agent**, cluster state'inin **sahibi değildir**. Cluster'a katılan client tarafındaki bir temsilci gibi davranır.
- **Asıl dağıtık state**, Consul Server'larda (Server 1, 2, 3) tutulur ve server'lar arasında **Raft** ile yönetilir.

Server agent'lar Raft'a katılarak şunları yapar:

| Eylem | Anlamı |
|---|---|
| **Leader election** | Bir lider server seçilir |
| **Log replication** | Değişiklikler diğer server'lara kopyalanır |
| **Commit** | Çoğunluk onaylayınca değişiklik kesinleşir |
| **Consensus** | Tüm server'lar aynı durumda uzlaşır |

### 2.8 Özet

- Consul = **servis kayıt defteri + health check + KV store + güvenli iletişim**
- Servisler birbirini **HTTP API** veya **DNS** ile bulur; sağlıksız instance'lar discovery sonucundan çıkarılır
- Her node'da **Consul Agent** çalışır; **Client** veya **Server** modunda
- **Server'lar** state'i tutar ve **Raft** ile yönetir; **Client'lar** kayıt ve health check yapıp bilgiyi server'a iletir

---

## 3. Apache ZooKeeper

### 3.1 Apache ZooKeeper nedir?

Apache ZooKeeper; dağıtık sistemlerde yapılandırma yönetimi, isimlendirme ve senkronizasyon işlemlerini yürüten merkezi bir **koordinasyon servisidir**. Büyük verileri değil, sistemin durumunu belirten küçük boyutlu kritik verileri (meta veri) depolamak için tasarlanmış ve özellikle yüksek okuma performansına göre optimize edilmiştir.

Java ile yazılan proje, başlangıçta Yahoo! bünyesindeki dağıtık sistemleri koordine etmek için tasarlanmış, günümüzde ise **Apache Software Foundation**'ın en önemli projelerinden biri olmuştur. Hadoop ve HBase gibi popüler dağıtık mimariler için standart bir koordinasyon aracıdır.

#### İsmi nereden geliyor?

Hadoop ekosistemindeki projelerin çoğu hayvan isimleriyle (Pig, Hive vb.) anılıyordu. Dağıtık sistemlerdeki bu "hayvanat bahçesini" düzenli tutan ve koordine eden servise de doğal olarak **ZooKeeper** (Hayvan Bakıcısı) adı verilmiştir.

### 3.2 Mimari yapı

ZooKeeper kümesi (cluster), **1 Lider (Leader)** ve **birden fazla Takipçi (Follower)** sunucudan oluşan bir mimariyle çalışır.

Hiyerarşik bir dosya sistemine benzeyen veri yapısı (znode ağacı) sunucu rollerinden bağımsızdır: znode ağacı verinin nasıl organize edildiğini gösterir ve kümedeki tüm sunucularda aynı kopya (replika) olarak tutulur.

> 💡 **Önemli not:** ZooKeeper'da okuma işlemleri varsayılan olarak doğrudan bağlanılan sunucudan (takipçiden) yapılır. Eğer o sunucu liderin gerisinde kalmışsa **eski veri (stale data)** dönebilir. Mutlaka en güncel veriye ulaşmak gerekiyorsa, okuma işleminden önce `sync` çağrılmalıdır. Bu yönüyle etcd gibi varsayılan olarak "güçlü tutarlılık" (strong consistency) sunan araçlardan ayrılır.

### 3.3 Veri modeli: znode kavramı

ZooKeeper veriyi, tıpkı standart bir bilgisayar dosya sistemi gibi **hiyerarşik bir ağaç** yapısında saklar. Bu ağaçtaki her bir düğüme **znode** denir.

```text
/
├── app1
│   ├── config
│   └── workers
│       ├── worker-0001
│       └── worker-0002
└── app2
    └── leader
```

Znode'lar hem kendi içlerinde **veri taşıyabilir** hem de **alt düğümlere (children)** sahip olabilirler. Bir znode içinde tutulan veri boyutu küçüktür (varsayılan sınır yaklaşık **1 MB**). Çünkü ZooKeeper bir veritabanı değil, koordinasyon aracıdır.

### 3.4 Znode türleri

Znode'lar, oluşturulurken seçilen ve sonradan değiştirilemeyen yaşam döngüsü modlarına sahiptir. Temel olarak şu türler vardır:

| Tür | Davranış |
|---|---|
| **Persistent (Kalıcı)** | Düğüm, kullanıcı tarafından açıkça silinene kadar sistemde kalır. |
| **Ephemeral (Geçici)** | Düğümü oluşturan istemcinin (client) oturumu koptuğunda veya bittiğinde otomatik silinir. *Not: Ephemeral düğümlerin alt düğümü (çocuğu) olamaz.* |
| **Sequential (Sıralı)** | Düğüm isminin sonuna otomatik olarak artan bir sayaç numarası eklenir (örn: `lock-0000000001`). Bu özellik Persistent veya Ephemeral ile birlikte kullanılabilir. |

Bunun yanı sıra, ZooKeeper 3.5.3 sürümüyle birlikte sisteme iki gelişmiş znode türü daha eklenmiştir:

#### 1. CONTAINER znode

Dağıtık kilit (lock) veya lider seçimi mekanizmalarında, altına sürekli geçici (ephemeral) düğümlerin eklenip silindiği bir ana "klasör" düğümüne ihtiyaç duyulur (örneğin `/locks` düğümü).

Eğer `/locks` düğümü normal bir "Persistent" düğüm olursa, altındaki tüm istemci işlemleri bitip düğümler silindiğinde içi boş kalır. Bu durum zamanla sistemde devasa bir "boş klasör (çöp)" birikmesine yol açar. Klasörü "Ephemeral" yapmak ise çözüm değildir, çünkü ephemeral düğümlerin alt düğümü olamaz.

**Container znode**, bu sorunu çözmek için tasarlanmıştır. İçerisindeki **son çocuk düğüm de silindiğinde, Container düğümü periyodik bir kontrol mekanizması tarafından otomatik olarak silinmeye aday olur**. Böylece sistem kendi çöpünü temizlemiş olur.

#### 2. TTL (Time-To-Live) znode

TTL znode, belirli bir süre (milisaniye cinsinden) güncellenmezse sistem tarafından otomatik silinen kalıcı (persistent) düğümlerdir. Geçici oturumlar, önbellek (cache) kayıtları veya süresi dolan token'lar gibi işlemler için idealdir. (Not: Zamanlayıcı milisaniye hassasiyetinde değildir, periyodik kontrollerle çalışır.)

*TTL düğümleri varsayılan olarak kapalıdır. Kullanmak için sunucu yapılandırmasında `zookeeper.extendedTypesEnabled=true` ayarı yapılmalıdır.*

### 3.5 Avantajları ve kullanım alanları

**Avantajları**

- **Merkezi yönetim ve dinamik güncelleme:** Yapılandırma ayarları tek bir merkezde tutulur. Makineler yeniden başlatılmadan konfigürasyon değişiklikleri anında (dinamik olarak) sisteme yansıtılır.
- **Değişiklik izleme (watch mekanizması):** Bir znode'da değişiklik olduğunda, o düğümü izleyen tüm istemcilere tek seferlik bir bildirim gönderilir. İstemci yeni veriyi çeker ve gerekiyorsa izlemeyi (watch) tekrar aktif eder.
- **Yüksek erişilebilirlik (HA) ve hata toleransı:** ZooKeeper sunucuları çoğaltılmış (replicated) çalışır. Kümedeki sunucuların çoğunluğu (quorum) ayakta kaldığı sürece sistem kesintisiz hizmet verir. Ephemeral düğümler sayesinde ağdan kopan veya arızalanan makineler, oturum zaman aşımı (session timeout) süresi dolduğunda tespit edilebilir.

**Kullanım alanları**

1. **Merkezi yapılandırma yönetimi:** Tüm servislerin konfigürasyonlarını tek bir noktadan okuması.
2. **Grup üyeliği ve lider seçimi (Leader Election):** Bir makine grubuna kimlerin dahil olduğunu takip etme ve aralarından bir yönetici (lider) atama işlemleri.
3. **Senkronizasyon ve dağıtık kilitler (Distributed Locks):** Dağıtık sistemlerde aynı anda tek bir servisin işlem yapmasını güvence altına alma.

#### Grup üyeliği örneği

5 adet sunucumuz olduğunu düşünelim. Bunlardan biri çöktü ve aktif olarak 4 sunucu kaldı. ZooKeeper bunu takip eder. Bu takip, ZooKeeper'a özgü znode yapısı sayesinde gerçekleşir. Her sunucu ZooKeeper'a kendisini temsil eden geçici (ephemeral) bir znode oluşturabilir:

```text
/services/workers/server1
```

Sunucu çalıştığı sürece znode hep vardır. Sunucu çalışmayı bırakırsa ya da bağlantı kopukluğu uzun sürüp oturum zaman aşımına (session timeout) uğrarsa, ephemeral znode otomatik olarak silinir. Kısa süreli bir kopuklukta client süre dolmadan yeniden bağlanırsa oturum ve znode korunur. Bu sayede znode'ların varlığına bakarak kaç adet aktif sunucu olduğunu takip edebiliriz.

### 3.6 Lider seçimi

Peki ZooKeeper'da 1 lider ve takipçileri var demiştik, bu lider seçimi nasıl yapılıyor? Burada iki ayrı lider seçimi olduğunu bilmek gerekiyor.

ZooKeeper sunucularının kendi aralarındaki lider seçimi ZooKeeper'ın iç işidir ve ZAB protokolü ile oylama yoluyla yapılır. Aşağıda anlatılan ise **ZooKeeper'ı kullanan uygulamaların** kendi aralarında yaptığı lider seçimidir. Yani uygulama, ZooKeeper'ı sadece bir araç olarak kullanır.

Yine 5 adet sunucumuz olduğunu düşünelim (bunlar ZooKeeper'ın değil, uygulamanın sunucuları). Her sunucuya bir ephemeral znode atandığını söylemiştik. Bu znode'lar aslında hem ephemeral hem de sequential olarak, yani bir sıra numarasıyla oluşturulur ve numaralar otomatik verilir:

```text
/leader/server-00000001
/leader/server-00000002
/leader/server-00000003
/leader/server-00000004
/leader/server-00000005
```

En küçük znode numarasına sahip olan sunucu lider olur. Bu sunucu arızalanırsa znode'u silineceği için, doğal olarak ondan sonraki en küçük numaraya sahip sunucu lider olur.

#### Thundering herd problemi

Liderin öldüğünü ve yeni liderin sıra numarasına göre seçileceğini düşünelim. Bunun diğer sunuculara bildirilmesi için iki yöntem düşünülebilir.

**1. Herkes lideri izlesin.** Bütün takipçi sunucuların lider sunucuyu izlediğini düşünelim. Lider ölünce ZooKeeper bunu bütün sunuculara aynı anda haber vermek zorunda kalır. Uyanan sunucuların hepsi de yeni liderin kim olduğunu öğrenmek için aynı anda ZooKeeper'a istek gönderir (hepsi `getChildren` çağırır). Bu da ZooKeeper üzerinde gereksiz bir yük oluşturur. Bu probleme **thundering herd** problemi denir.

**2. Herkes sadece kendinden önceki znode'u (sunucuyu) izlesin.** Bu yaklaşım, ilk yöntemin yarattığı yükü ortadan kaldırmak için geliştirilmiştir.

```text
server-2 → server-1'i izler
server-3 → server-2'yi izler
server-4 → server-3'ü izler
server-5 → server-4'ü izler
```

Lider (server-1) öldüğünde sadece server-2 uyandırılır. server-2 çocukları listeler, en küçük numaranın kendisi olduğunu görür ve lider olur. Diğerlerinin uyanmasına gerek yoktur. Yani ZooKeeper kimin lider olduğunu "söylemez", sunucular listeye bakıp kendi sıralarını kendileri belirler. Ortadaki bir sunucu (örneğin server-2) çökerse, onu izleyen server-3 uyandırılır, hâlâ lider olmadığını görür ve bu sefer bir önceki yaşayan sunucuyu (server-1) izlemeye başlar.

### 3.7 2026 yılında tespit edilen güvenlik açıkları

2026 yılında Apache ZooKeeper'da sistemi doğrudan etkileyen kritik zafiyetler tespit edildi. Bu açıklar **3.9.0 - 3.9.5** ve **3.8.0 - 3.8.6** sürümlerini etkilemektedir ve **3.9.6** ile **3.8.7** yamalarıyla kapatılmıştır.

- **CVE-2026-79993 (Konteyner düğümü yetki atlatması):** ZooKeeper'ın belgelenmemiş (dahili) bir protokol işleyicisindeki bu açık, sistemin en ciddi zafiyetlerinden biridir. Saldırgan `deleteContainer` istek yolunu kullanarak hem oturum kontrolünü hem de DELETE ACL yetkilendirmesini devre dışı bırakabilir. Kimlik doğrulaması yapmamış bir saldırgan, 2181 portu üzerinden ham protokol mesajları göndererek boş durumdaki persistent, container ve TTL znode'ları silebilir. Resmi client'ta bu işlem için bir API olmadığından ham protokol kullanılır.
- **CVE-2026-84501 (Log satırı enjeksiyonu):** Kimlik doğrulaması yapmamış bir saldırgan, `EnsembleAuthenticationProvider` bileşenine yeni satır (newline) karakterleri içeren `add_auth("ensemble", ...)` istekleri göndererek operasyonel log dosyalarına sahte kayıtlar ekleyebilir. Bu kayıtlar zaman damgası, log seviyesi ve hata mesajı gibi alanları taklit ettiği için gerçek loglardan ayırt edilemez. Bu durum, güvenlik ekiplerinin olaya müdahale (incident response) süreçlerini manipüle etmek için kullanılabilir.
- **CVE-2026-24281 (Ters DNS üzerinden sunucu taklidi):** ZKTrustManager alt sistemindeki TLS sertifikası doğrulamasında, IP SAN doğrulaması başarısız olursa sistem ters DNS (PTR kaydı) sorgusuna yönelir. PTR kayıtlarını manipüle edebilen bir saldırgan, geçerli bir sertifikası olmasa bile güvenilir bir ZooKeeper sunucusu veya istemcisi gibi davranabilir.
- **CVE-2026-59739 (Yeniden bağlanmada bilgi sızıntısı):** İstemci yeniden bağlanırken tetiklenen izleme (watch) mekanizmasında ACL kontrolü yapılmaması yüzünden ortaya çıkar. Saldırgan, var olmayan yollar üzerinde izleyiciler oluşturarak erişimi kısıtlanmış znode isimlerini, hassas kullanıcı adlarını ve dahili tanımlayıcıları öğrenebilir.
- **CVE-2026-24308 (Log dosyalarına hassas veri sızması):** ZKConfig bileşeninin yapılandırma değerlerini yanlış işlemesinden kaynaklanır. İstemci logları INFO seviyesinde tutulurken, yapılandırmadaki gizli bilgiler düz metin olarak loglara yazılır.
- **CVE-2026-59969 (FIPS modunda sertifika doğrulama hatası):** FIPS modunda ve quorum TLS açıkken, host adı doğrulamaları açık olsa bile quorum bağlantısı, CA tarafından güvenilen ama SAN alanı bağlanılan host ile uyuşmayan bir sertifikayı kabul eder.

Ulusal Siber Olaylara Müdahale Merkezi (USOM) da bu zafiyetlerin aktif sistemleri tehdit ettiğini doğrulamıştır. ZooKeeper 3.7 ve önceki sürümlerin desteği sona erdiği için bu yamalardan faydalanamamaktadır.

---

# B. Kubernetes'te Konfigürasyon ve Secret

---

## 4. Kubernetes ConfigMap, Pod, Deployment ve HPA

> 💡 Bu bölümde ConfigMap anlatılırken Pod, Deployment ve HPA gibi kavramlar da geçer. Bunların açıklamaları bölümün devamındadır.

### 4.1 ConfigMap nedir?

Kubernetes içindeki **ConfigMap**, uygulamanın kodundan ayrı olarak hassas olmayan, yani gizli olmayan konfigürasyon verilerini saklamaya yarayan Kubernetes nesnesidir. **ConfigMap gizlilik veya şifreleme sağlamaz!** Şifre, API key, token gibi gizli bilgiler için [Kubernetes Secret](#5-kubernetes-secret) veya harici bir secret yönetim sistemi kullanılmalıdır.

Örneğin:

```text
PORT=8080
LOG_LEVEL=INFO
APP_NAME=payment-service
DB_HOST=postgres
```

Buradaki amaç, bu değerleri doğrudan uygulamanın koduna veya Docker image'ına yazmak yerine uygulamadan ayrı olarak yönetmektir. Böylece aynı uygulama farklı ortamlarda farklı konfigürasyonlarla çalıştırılabilir:

```text
Development:
DB_HOST=dev-postgres

Production:
DB_HOST=prod-postgres
```

Uygulamanın kendisi değişmez, sadece kullanılan ConfigMap değişir.

ConfigMap'in adı geçerli bir DNS alt alan adı olmalıdır. `data` veya `binaryData` alanının altındaki her anahtar; alfanümerik karakterlerden, `-`, `_` veya `.` karakterlerinden oluşmalıdır. `data` içinde saklanan anahtarlar, `binaryData` alanındaki anahtarlarla çakışmamalıdır.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  # property-like keys; each key maps to a simple value
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  # file-like keys
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
```

Bir Pod içindeki bir container'ı yapılandırmak için ConfigMap'i kullanmanın dört farklı yolu vardır:

1. Bir container'ın içindeki komut ve argümanlar
2. Bir container için ortam değişkenleri
3. ConfigMap'i salt okunur bir volume olarak ekleyip uygulamanın dosya olarak okumasını sağlamak
4. Pod içinde çalışacak ve Kubernetes API'sini kullanarak ConfigMap'i okuyacak bir kod yazmak

İlk üç yöntem için, kubelet Pod için container(lar) başlatırken ConfigMap'ten gelen verileri kullanır.

Dördüncü yöntem, ConfigMap'i ve verilerini okumak için kod yazmanız gerektiği anlamına gelir. Ancak Kubernetes API'sini doğrudan kullandığınız için uygulamanız ConfigMap değiştiğinde güncellemeleri almak için abone olabilir ve bu gerçekleştiğinde tepki verebilir. Kubernetes API'sine doğrudan erişim sayesinde bu teknik farklı bir ad alanındaki (namespace) ConfigMap'e erişmenizi de sağlar.

### 4.2 ConfigMap verilerinin kullanılması

ConfigMap'teki veriler Pod içerisinde temel olarak iki şekilde kullanılabilir:

```text
ConfigMap
    │
    ├── Environment Variable
    │
    └── Dosya
```

Her iki yöntemde de ConfigMap içerisindeki veriler **key-value** mantığıyla tutulur.

#### Environment variable olarak

Örneğin ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: app-config

data:
  LOG_LEVEL: "INFO"
  PORT: "8080"
```

şeklinde oluşturulabilir.

Daha sonra Pod'un veya Deployment'ın tanımında bu ConfigMap'in kullanılacağı belirtilir:

```yaml
envFrom:
  - configMapRef:
      name: app-config
```

Böylece ConfigMap'teki değerler container içerisinde environment variable olarak kullanılabilir:

```text
Container Environment

LOG_LEVEL=INFO
PORT=8080
```

#### Dosya olarak

ConfigMap'teki bilgiler bir dosya şeklinde de Pod'a aktarılabilir.

Örneğin:

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: app-config

data:
  application.conf: |
    port=8080
    log_level=INFO
    db_host=postgres
```

Bu ConfigMap bir volume olarak Pod'a bağlandığında container içerisinde `/etc/config/application.conf` gibi bir dosya olarak görülebilir. Pod tanımında bu bağlama şöyle yapılır:

```yaml
spec:
  containers:
    - name: app
      image: my-app:v1
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: app-config
```

> 💡 ConfigMap sonradan değiştirilirse, environment variable olarak verilen değerler çalışan Pod'da kendiliğinden güncellenmez (Pod'un yeniden başlaması gerekir). Volume olarak bağlanan dosyalar ise bir süre sonra güncellenir (`subPath` ile bağlananlar hariç).

### 4.3 Pod

**Pod**, Kubernetes'teki uygulamaların çalıştırıldığı en temel birimdir. Bir veya birden fazla container'ın birlikte çalıştığı Kubernetes çalışma birimidir.

Basit bir Pod:

```text
Pod
┌─────────────────────┐
│                     │
│     Container       │
│   ┌───────────────┐ │
│   │  Python App   │ │
│   └───────────────┘ │
│                     │
└─────────────────────┘
```

Kubernetes sadece container çalıştırmakla kalmaz; container'ların ağ, depolama ve yaşam döngüsü gibi kaynaklarını da yönetir. Pod, Kubernetes'in bu kaynakları container'lara sunabilmesi için oluşturduğu çalışma ortamıdır.

Pod içerisindeki container'lar:

- Aynı network namespace'i paylaşabilir.
- Birbirlerine `localhost` üzerinden ulaşabilir.
- Ortak volume kullanabilir.

ConfigMap ile birlikte düşündüğümüzde: ConfigMap uygulamanın kullanacağı konfigürasyonu sağlar, Pod ise container'ın çalıştığı ortamı oluşturur.

```text
ConfigMap ──▶ Pod ──▶ Container ──▶ Uygulama
```

### 4.4 Kubernetes Deployment

**Deployment**, Kubernetes'e uygulamanın kaç kopyasının çalışacağını ve bu Pod'ların nasıl yönetileceğini söyleyen Kubernetes nesnesidir.

Örneğin:

```yaml
spec:
  replicas: 3
```

yazarsak, bu uygulamadan **3 Pod'un çalışmasını istediğimizi** belirtmiş oluruz. Buradaki Pod'ları uygulamanın birbirinin replikaları gibi düşünebiliriz.

Deployment doğrudan Pod'ları yönetmek yerine genellikle **ReplicaSet** üzerinden yönetir:

```text
Deployment
     │
     ↓
ReplicaSet
     │
     ├── Pod 1
     ├── Pod 2
     └── Pod 3
```

#### ReplicaSet

**ReplicaSet**, Deployment tarafından belirlenen istenen sayıda Pod'un çalışmasını sağlamaya çalışır.

- İstenen 3, mevcut 3 ise herhangi bir işlem yapmaz.
- Bir Pod çökerse mevcut sayı 2'ye düşer; ReplicaSet eksik olan Pod'u yeniden oluşturur ve sayı tekrar 3 olur.

### 4.5 Deployment ve ConfigMap birlikte nasıl çalışır?

Örneğin bir SER-AI uygulamamız olduğunu düşünelim. Öncelikle uygulamanın konfigürasyon bilgilerini içeren bir ConfigMap oluşturulur:

```text
serai-config

MQTT_HOST = 192.168.1.10
MQTT_PORT = 1883
LOG_LEVEL = INFO
```

Daha sonra Deployment oluşturulur ve bu uygulamadan 3 Pod çalıştırılması istenir:

```text
Image: serai-app:v1
Replicas: 3
ConfigMap: serai-config
```

Çalışma süreci:

```text
1. ConfigMap oluşturulur
        ↓
2. Deployment oluşturulur (3 Pod ister)
        ↓
3. Deployment → ReplicaSet → 3 Pod oluşturulur
        ↓
4. Pod'ların içindeki container'lar başlatılır
        ↓
5. ConfigMap'teki konfigürasyonlar container'a sağlanır
        ↓
6. Uygulama bu konfigürasyonlarla çalışır
```

Sonuçta yapı şu hale gelir:

```text
                         Kubernetes
                              │
                 ┌────────────┴────────────┐
                 │                         │
             Deployment                ConfigMap
                 │                         │
                 ↓                         │
             ReplicaSet                    │
                 │                         │
        ┌────────┼────────┐                │
        ↓        ↓        ↓                │
      Pod 1    Pod 2    Pod 3              │
        │        │        │                │
        ↓        ↓        ↓                │
    Container Container Container          │
        │        │        │                │
        └────────┼────────┘                │
                 ↓                         │
             Uygulama  ←───────────────────┘
```

Burada **ConfigMap Pod sayısını belirlemez**; Pod'ların ve dolayısıyla container'ların kullanacağı konfigürasyonu sağlar. Pod sayısını ve Pod'ların yönetimini ise Deployment belirler.

### 4.6 Kullanıcı isteği geldiğinde ne olur?

Pod'lar uygulama çalışmaya başlamadan önce oluşturulmuş durumdadır. Kullanıcı uygulamaya istek gönderdiğinde yeni bir Pod oluşturulmaz. **Service**, gelen isteği mevcut ve uygun Pod'lardan birine yönlendirir:

```text
Kullanıcı
    ↓
Service
    ↓
Pod 2
    ↓
Container
    ↓
Uygulama
```

### 4.7 Replica sayısının güncellenmesi

Deployment içindeki `replicas` değeri, çalıştırılacak Pod sayısını belirler. Uygulamanın yükü artar ve daha fazla Pod'a ihtiyaç duyulursa bu sayı artırılabilir. Bu işlem:

- **Manuel olarak** yapılabilir (`replicas: 3` → `replicas: 10` ile 3 Pod'dan 10 Pod'a çıkılır),
- **HPA** kullanılarak otomatik olarak yapılabilir.

### 4.8 HPA (Horizontal Pod Autoscaler)

**HPA**, uygulamanın çalışma sırasında kullandığı kaynakları ve tanımlanan diğer metrikleri izleyerek Pod sayısını otomatik olarak artırıp azaltmaya yarayan Kubernetes nesnesidir.

Buradaki **Horizontal** ifadesi, mevcut Pod'u güçlendirmek yerine **Pod sayısının artırılıp azaltılmasını** ifade eder.

Örneğin HPA için şöyle bir hedef belirlenebilir:

```text
CPU hedefi: %60
Minimum Pod: 3
Maksimum Pod: 10
```

HPA metrikleri sürekli kontrol eder. CPU kullanımı hedefin üzerine çıkarsa replica sayısını artırır (örneğin 3 Pod → 5 Pod), yük azaldığında ise tekrar düşürür (5 Pod → 3 Pod). Böylece yoğunluk arttığında yük dağıtılır, yoğunluk azaldığında gereksiz Pod'lar azaltılır.

HPA'nın temel çalışma mantığı:

```text
Uygulama
  ↓
CPU / Bellek / Diğer metrikler
  ↓
HPA
  ↓
Deployment'ın replica sayısını güncelle
  ↓
ReplicaSet
  ↓
Pod sayısını artır veya azalt
```

Önemli nokta: **HPA doğrudan Pod oluşturmaz**, Deployment'ın replica sayısını değiştirir; Pod'ları Deployment'ın ReplicaSet'i oluşturur.

---

## 5. Kubernetes Secret

### 5.1 Kubernetes Secret nedir?

Secret'ın genel tanımı için [giriş bölümüne](#secret-nedir-secret-yönetimi-nedir) bakınız. **Kubernetes Secret**, bu ihtiyacı Kubernetes içinde karşılayan nesnedir: uygulamanın ihtiyaç duyduğu hassas bilgileri (parola, token, sertifika, API anahtarı vb.) Kubernetes içerisinde saklar ve Pod'lara sağlar. Böylece bu bilgilerin container image'ına ya da Pod/Deployment tanımına gömülmesi gerekmez.

### 5.2 Kim tarafından, ne amaçla geliştirildi?

Kubernetes, **Google** tarafından başlatılan ve 2014'te açık kaynak olarak duyurulan bir projedir. 1.0 sürümü Temmuz 2015'te yayımlanmış ve proje **CNCF**'ye bağışlanmıştır. Secret nesnesi de Kubernetes'in ilk dönemlerinden beri API'nin bir parçasıdır.

Secret'ın tasarım amacı, parola ve anahtar gibi bilgilerin container'lara, container'ın kendisini değiştirmeden dağıtılmasıdır. Tasarım dokümanında şu ihtiyaç öne çıkar: container'lar Kubernetes master'ı, git depoları ya da veritabanları gibi iç ve dış kaynaklara erişmek için secret'lara ihtiyaç duyar. İlk tasarımda bu bilgiler container'a özel bir volume türüyle (dosya olarak) sağlanıyordu. Ortam değişkeni olarak kullanma ise sonradan eklenmiştir.


### 5.3 Secret nasıl kullanılır?

Secret'ın kullanılma akışı:

**1. Secret oluşturulur**

**2. Deployment (veya Pod), Secret'i kullanacağını belirtir.** ConfigMap'te olduğu gibi iki yol vardır:

- **Environment variable olarak:**

```yaml
envFrom:
  - secretRef:
      name: database-secret
```

Tek bir anahtarı almak için `secretKeyRef` kullanılır:

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: database-secret
        key: DB_PASSWORD
```

- **Dosya olarak (volume):**

```yaml
spec:
  containers:
    - name: app
      image: my-app:v1
      volumeMounts:
        - name: secret-volume
          mountPath: /etc/secrets
          readOnly: true
  volumes:
    - name: secret-volume
      secret:
        secretName: database-secret
```

**3.** Pod oluşturulur

**4.** Secret bilgileri container'a sağlanır

> 💡 ConfigMap'teki davranış Secret için de geçerlidir: Secret değişirse environment variable olarak verilen değerler çalışan Pod'da kendiliğinden güncellenmez (Pod'un yeniden başlaması gerekir). Volume olarak bağlanan dosyalar bir süre sonra güncellenir (`subPath` ile bağlananlar hariç). Volume olarak bağlanan Secret'lar node'un diskine değil, belleğe (tmpfs) yazılır.

Secret genellikle ConfigMap ile birlikte kullanılır: normal ayarlar ConfigMap'te, hassas bilgiler Secret'ta durur.

```text
                 Kubernetes
                     │
          ┌──────────┴──────────┐
          │                     │
      ConfigMap              Secret
          │                     │
     DB_HOST               DB_USERNAME
     DB_PORT               DB_PASSWORD
          │                     │
          └──────────┬──────────┘
                     ↓
              Pod / Container
                     ↓
                 Uygulama
```

### 5.4 Secret'lar nerede saklanır?

Kubernetes kendi verilerini saklamak için [etcd](#1-etcd) kullanır. Secret oluşturduğunuzda Kubernetes'in bu bilgiyi bir yerde saklaması gerekir ve bu bilgiler de etcd'ye kaydedilir.

Peki o zaman Secret'a ne gerek var diye düşünebilirsiniz. Burada etcd'nin tuttuğu Kubernetes'in kendi verileridir; Secret ise kullanıcı uygulamalarındaki ya da servislerindeki şifre gibi gizli tutulması gereken bilgileri temsil eden nesnedir.

### 5.5 `data` ve `stringData` farkı

Secret YAML'ları iki şekilde yazılabilir:

**1. `data` şeklinde** (değer base64 ile kodlanmış olarak yazılır):

```yaml
data:
  DB_PASSWORD: bXktcGFzc3dvcmQ=
```

**2. `stringData` şeklinde** (değer düz metin olarak yazılır):

```yaml
stringData:
  DB_PASSWORD: my-password
```

`stringData` kullanıldığında Kubernetes, düz metin değeri otomatik olarak base64 formatına çevirip `data` alanı altında saklar. Yani iki yazım şekli de sonuçta aynı biçimde saklanır.

Burada yapılan işlem **şifreleme değil, encoding'dir** (kodlama). Bu encoding işlemini herkes geri çevirebilir, yani decode edebilir. Dolayısıyla bu bir veri gizleme işleminden ziyade, veriyi belirli bir karakter formatında temsil etme işlemidir. Base64, verinin Kubernetes API/YAML içerisindeki veri formatına uygun şekilde taşınabilmesi için kullanılır.

### 5.6 ConfigMap'e göre güvenlik avantajı: RBAC

RBAC (Role-Based Access Control) hem ConfigMap hem Secret için geçerlidir. Ancak Secret'lar ayrı bir kaynak türü olduğu için erişim ayrı ayrı yönetilebilir. Örneğin bir kullanıcıya ConfigMap'leri okuma izni verip Secret'ları okuma izni vermemek mümkündür. Böylece kullanıcı Secret içerisindeki hassas verileri okuyamaz.

Servisler tarafında ise **ServiceAccount** kullanılır. Pod'lar bir ServiceAccount ile çalışır ve RBAC kuralları ile bu ServiceAccount'un hangi Secret'lara erişebileceği belirlenebilir.

### 5.7 etcd'de encryption at rest

Bir Secret'ımız olduğunu düşünelim:

```text
DB_PASSWORD = my-password
```

Kubernetes bunu Secret resource'u olarak yönetiyor ve etcd'de Base64 olarak saklıyor. Ama burada hâlâ herhangi bir şifreleme söz konusu değil:

```text
my-password
      ↓
Base64
      ↓
bXktcGFzc3dvcmQ=
```

Bu değeri gören biri Base64 decode ederek tekrar `my-password` elde edebilir.

**Encryption at Rest:** etcd'ye yazılan Secret verilerini disk üzerinde şifreler. Uygun bir anahtar sayesinde şifreleme yapılır ve disk üzerinde artık doğrudan `my-password` gibi okunabilir bir veri yerine şifrelenmiş veri bulunur.

```text
Secret
  │
  │ DB_PASSWORD=my-password
  ↓
Kubernetes API
  │
  │ Encryption
  ↓
etcd
  │
  ↓
Disk
```

#### Secret zaten etcd'de, neden ayrıca şifreliyoruz?

Eğer etcd'nin depolama alanına yetkisiz biri erişirse ve Secret'ler at rest encryption olmadan saklanıyorsa, Secret verilerine ulaşma riski vardır. Bu yüzden her katman farklı bir riske karşı koruma sağlar:

```text
                    Kubernetes
                         │
                    Secret
                         │
                ┌────────┴────────┐
                │                 │
              TLS             API/RBAC
                │                 │
                ↓                 ↓
          Network güvenliği   Erişim kontrolü
                                  │
                                  ↓
                                etcd
                                  │
                         Encryption at Rest
                                  │
                                  ↓
                               Disk
```

> ⚠️ Encryption at rest otomatik olarak "Secret kullandım, artık şifreli" anlamına gelmez. Cluster yöneticisinin bunu yapılandırması gerekir.

Rotasyon, dinamik secret gibi daha gelişmiş ihtiyaçlar için harici sistemler kullanılır: [Vault](#7-hashicorp-vault), [AWS Secrets Manager](#8-aws-secrets-manager), [Google Cloud Secret Manager](#10-google-cloud-secret-manager).

---

# C. Uygulama Seviyesinde Konfigürasyon

---

## 6. Spring Cloud Config

### 6.1 Neden ihtiyaç var?

Mikro hizmetler mimarisinde birçok mikro hizmet bir araya gelerek bir uygulamayı oluşturur ve bunlar genellikle farklı ekipler tarafından geliştirilip ayrı ayrı dağıtılır. Her servisin kendi yapılandırma dosyasını taşıması, servis sayısı arttıkça yönetimi zorlaştırır. Spring Cloud Config bu soruna çözüm olarak yapılandırmayı merkezi bir yerde toplar.

### 6.2 Spring Cloud Config Server nedir?

Config Server modülü, yapılandırma dosyalarımızı uzak bir depodan (GitHub, GitLab, Bitbucket vb.), yerel (local) bir dizinden ya da farklı servisler üzerinden (AWS, HashiCorp Vault vb.) okumamıza olanak sağlar.

Config Server'a bağlanıp ayarlarını oradan alan servislere de **Config Client** denir.

etcd ve ZooKeeper da yapılandırma saklayabilir ama onlar genel amaçlı koordinasyon araçlarıdır. Spring Cloud Config ise sadece yapılandırma yönetimi için tasarlanmıştır ve Spring ekosistemiyle doğrudan entegredir.

#### Avantajları

- **Merkezi yönetim:** Tüm servislerin ayarları tek bir yerde durur.
- **Dinamik yeniden yükleme:** Ayarlar değiştiğinde servisi yeniden başlatmadan güncellenebilir (aşağıda anlatılıyor).
- **Çeşitli kaynaklardan yükleme:** Git, yerel dizin, Vault, AWS gibi farklı kaynaklar kullanılabilir.

![Spring Cloud Config Server Kullanımı](https://miro.medium.com/v2/resize:fit:720/format:webp/1*e5Fg_XwEj9eK7BU0l55YwQ.png)

*Resim kaynağı: https://kayhanozturk.medium.com/spring-cloud-config-server-nedir-01a268d8c6e6*

### 6.3 Nasıl çalışır?

```text
Git deposu ──▶ Config Server ──(HTTP)──▶ Servis A
                                    ├──▶ Servis B
                                    └──▶ Servis C
```

1. Ayar dosyaları bir Git deposunda tutulur (örneğin `order-service.yml`, `order-service-prod.yml`).
2. Config Server bu depoyu okur.
3. Bir servis açılırken Config Server'a istek atar ve kendi ayarlarını alır.
4. Servis bu ayarlarla ayağa kalkar.

### 6.4 Ön bilgi: Bean nedir?

Yapılandırmanın canlı yenilenmesini anlamak için önce Spring'deki **bean** kavramını bilmek gerekiyor.

Bean, Spring'de uygulamanın içindeki, Spring'in kendisinin oluşturup yönettiği nesnelerdir. Java'da bir nesneyi `new` ile biz oluştururuz, Spring'de ise bu işi Spring üstlenir.

```java
// Java
OrderService service = new OrderService();
```

```java
// Spring
@Service
public class OrderService {
    public void siparisVer() { ... }
}
```

`@Service` (ya da `@Component`, `@Repository`, `@Controller`) ile işaretlenen sınıf için Spring, uygulama açılırken bir nesne oluşturur ve kendi içindeki bir havuzda tutar. Bu havuza **Spring Container** (ya da *ApplicationContext*) denir. Havuzdaki her nesne bir bean'dir.

Başka bir sınıf bu nesneye ihtiyaç duyarsa kendisi `new` demez, Spring'den ister. Buna **bağımlılık enjeksiyonu (Dependency Injection)** denir:

```java
@RestController
public class OrderController {

    private final OrderService orderService;

    public OrderController(OrderService orderService) {
        this.orderService = orderService;   // Spring hazır bean'i buraya verir
    }
}
```

#### Bean'lerin varsayılan davranışı: Singleton

Spring varsayılan olarak her bean'den **tek bir tane** oluşturur ve uygulama boyunca herkes aynı nesneyi kullanır. Bean uygulama açılırken bir kez oluşturulur ve sonradan yeniden oluşturulmaz.

### 6.5 Çalışma anında yapılandırmayı yenileme

Normalde servisler ayarları **sadece başlarken** okur. Bir bean ayarı oluşturulurken bir kez okuduğu için, Config Server'daki ayar sonradan değişse bile bean eski değerle devam eder. Ayar değişince servisi yeniden başlatmamak için iki yol vardır:

- **Actuator `/actuator/refresh`:** Tek bir servisi elle tetikler. Yenilenecek bean'ler `@RefreshScope` ile işaretlenir.
- **Spring Cloud Bus:** Mesajlaşma altyapısı (RabbitMQ veya Kafka) üzerinden **tüm servislere** tek seferde "yapılandırma değişti" haberi yayar.

#### `@RefreshScope` nasıl çalışır?

`@RefreshScope` ile işaretlenen bean, `/actuator/refresh` çağrıldığında silinir ve yeniden oluşturulur. Yeni oluşan bean de güncel ayarları okur.

```java
@RestController
@RefreshScope
public class MesajController {

    @Value("${mesaj}")
    private String mesaj;

    @GetMapping("/mesaj")
    public String getMesaj() {
        return mesaj;
    }
}
```

```text
Config Server'da mesaj değişir
        │
        ▼
POST /actuator/refresh
        │
        ▼
@RefreshScope'lu bean silinir, yeniden oluşturulur
        │
        ▼
Yeni bean güncel "mesaj" değerini okur
```

Servisi yeniden başlatmaya gerek kalmaz. Ama yalnızca `@RefreshScope` ile işaretli bean'ler yenilenir, diğer bean'ler eski değerlerle devam eder.

> 💡 `/actuator/refresh` endpoint'i varsayılan olarak web üzerinden açık değildir. Kullanmak için Actuator bağımlılığını eklemek ve endpoint'i `management.endpoints.web.exposure.include=refresh` ayarıyla açmak gerekir.

ZooKeeper ve etcd'den farkı şudur: onlarda değişiklik **watch** ile otomatik bildirilir. Spring Cloud Config'te ise yenileme mekanizmasını (refresh ya da Bus) biz kurarız.

### 6.6 Client'ın Config Server'ı bulma yöntemleri

| Yaklaşım | Açıklama |
|---|---|
| **Config First** | Client, Config Server adresini kendi ayarında sabit olarak bilir (`spring.config.import`) |
| **Discovery First** | Client, Config Server'ı Eureka/Consul gibi bir servis keşfi üzerinden isminden bulur |

İkincisi, Config Server adresi değiştiğinde client ayarlarını güncellemeyi gerektirmez.

### 6.7 Kubernetes kullanılıyorsa?

Kubernetes kullanılan ortamlarda [ConfigMap ve Secret](#4-kubernetes-configmap-pod-deployment-ve-hpa) zaten yapılandırma yönetimi sağlar. Bu yüzden şu soru sık sorulur: "Spring Cloud Config'e hâlâ ihtiyaç var mı?"

- Uygulamalar yalnızca Kubernetes'te çalışıyorsa ConfigMap/Secret (ve Spring Cloud Kubernetes) çoğu zaman yeterli ve daha sadedir.
- Spring Cloud Config'in öne çıktığı durumlar şunlardır: Git ile ayar geçmişi ve geri alma ihtiyacı, Kubernetes dışı ortamlar, çoklu ortam yönetimi.

---

# D. Secret Yönetimi

Secret ve secret yönetimi kavramları için [giriş bölümüne](#secret-nedir-secret-yönetimi-nedir) bakınız.

---

## 7. HashiCorp Vault

### 7.1 Vault nedir?

HashiCorp Vault, gizlilik açısından önemli olan konfigürasyonları ve projelerin hassas bilgilerini merkezi ve güvenli şekilde, ayrı path'lerde tutan bir yapıdır.

![Vault](https://web-unified-docs-hashicorp.vercel.app/api/assets/vault/latest/img/how-vault-works.png)

### 7.2 Kim tarafından, ne zaman geliştirildi?

HashiCorp (Mitchell Hashimoto ve Armon Dadgar) tarafından başlangıçta şirket içi ihtiyaçlar için geliştirilen Vault, Nisan 2015'teki ilk sürümünden itibaren "dinamik gizli bilgi üretme" yeteneğiyle öne çıkmıştır. 2018'de kararlı 1.0 sürümüne ulaşan ve Şubat 2025'teki satın almayla IBM bünyesine katılan araç; günümüzde kaynak kodu açık (2023'ten itibaren BSL lisansıyla) ücretsiz "Community" ve ücretli "Enterprise" seçenekleriyle kullanılmaya devam etmektedir.

### 7.3 Temel özellikleri

#### 1. Anahtar/değer sırlarını şifreli saklama

Vault, rastgele anahtar/değer sırlarını saklayabilir. Bu bilgileri saklamadan önce şifreler. Bu sayede depolama alanına erişim olsa bile düz metin sürümüne ulaşılamaz, yani sırlar açığa çıkmaz.

#### 2. Şifrelemeyi hizmet olarak sunma (Transit)

Vault, şifrelemeyi bir hizmet olarak sunar. **Transit** gizli veri motoruyla Vault, verileri kendisinde depolamadan, geçiş halindeki veriler üzerinde şifreleme işlemlerini gerçekleştirebilir. Esasen bu, Vault'un bir aracı görevi gördüğü, hizmet olarak şifrelemedir. Bu sayede şifreleme/şifre çözme işlemleri uygulama geliştiricilerine değil Vault'a ait olur ve şifrelenmiş verileri SQL veritabanı gibi başka bir yere yazabiliriz.

#### 3. Dinamik gizli bilgiler

HashiCorp Vault'u diğer şifrelenmiş anahtar-değer depolarından ayıran özellik budur: Vault, istek geldiğinde (on-demand) gizli bilgi üretebilir. Aşağıda ayrıntılı anlatılıyor.

### 7.4 Dinamik gizli bilgiler nasıl çalışır?

#### Önce problem: sabit şifre

Mesela bir uygulamanın veritabanına bağlanması gerekiyor. Normalde şifre uygulamanın config dosyasına yazılır:

```yaml
database:
  username: app_user
  password: 123456
```

```text
Uygulama
   │
   │ username: app_user
   │ password: 123456
   ↓
Veritabanı
```

Buradaki problemler şunlar:

- Bu şifre uzun süre geçerli olabilir.
- Birden fazla uygulama aynı şifreyi kullanabilir.
- Şifre loglara, hata kayıtlarına veya başka yerlere sızabilir.
- Şifreyi değiştirmek manuel işlem gerektirebilir.
- Şifre çalınırsa saldırgan uzun süre kullanabilir.

#### Vault'un çözümü

Vault burada şunu yapıyor: "Bu şifreyi aylarca sabit tutmak yerine, uygulama istediği zaman ona özel bir kimlik bilgisi oluşturayım."

```text
Uygulama
   │
   │ "Database credential lazım"
   ↓
 Vault
   │
   │ username: temp_user_8472
   │ password: X7f...92K
   ↓
Uygulama
```

Bu kullanıcı adı ve şifre önceden hazırlanmış olmak zorunda değil. Dinamik gizli bilgiler, biri okuyana (istek gelene) kadar oluşturulmaz. Örneğin Vault'ta `database/creds/my-role` diye bir rol tanımladık ama henüz kimse istemedi; bu durumda henüz hiçbir credential yoktur. Uygulama "Bana my-role için database credential ver." dediğinde Vault o anda oluşturur.

#### Her servise farklı şifre, belirli süre geçerlilik

Aynı konfigürasyona 2 farklı servis erişmek istiyor diyelim. İkisine de farklı bir şifre gider, çünkü yapı, istek geldiği an dinamik olarak benzersiz bir şifre belirleyip göndermek üzerine kuruludur.

Bu şifreler her zaman bir **lease (kira) süresiyle** gelir. Servis belirli bir süre kadar erişebilir, süre bittikten sonra Vault ilgili kimlik bilgisini iptal eder ve şifre kullanılamaz hale gelir. İstenirse süre dolmadan elle iptal (revoke) de edilebilir.

> ⚠️ Şifre, kullanıldıktan sonra otomatik olarak iptal olmaz. Geçerlilik süresi dolana ya da iptal edilene kadar çalışmaya devam eder. Kısa süre ayarlamak bu yüzden önemlidir.

### 7.5 Sızıntı problemi ve dinamik gizli bilgiler

Uygulamalar genellikle gizli bilgileri günlük dosyalarında veya kayıt sistemlerinde bırakır. HashiCorp'un kurucu ortağı Armon Dadgar'a göre gizli bilgiler ayrıca harici izleme sistemlerine gönderilen istisna izleme kayıtlarında veya çökme raporlarında yakalanabilir, ya da bir hatayla karşılaşıldıktan sonra hata ayıklama uç noktaları ve teşhis sayfaları aracılığıyla sızdırılabilir.

Örneğin uygulama hata verdi ve log şöyle oldu:

```text
ERROR:
Database connection failed
username=app_user
password=ABC123
```

Artık password log sistemine gitmiş olabilir. Secret uygulamada bir kere bulundu mu şu yerlere sızabilir:

```text
Uygulama
   │
   ├── Log
   ├── Error report
   ├── Crash report
   ├── Debug endpoint
   ├── Monitoring
   └── Diagnostic page
```

Dolayısıyla secret'ı sadece config dosyasından çıkarmak tek başına bütün problemi çözmüyor. Ama dynamic secret kullanırsak, sızan credential'ın ömür süresi kısa olur.

### 7.6 Entegrasyonlar

Vault; GitHub, Kubernetes, Microsoft SQL Server EKM sağlayıcısı ve ServiceNow gibi sistemlerle de entegre edilebilir.

### 7.7 Nasıl çalışır?

1. İstemciler (client), manuel olarak oluşturulan belirteçler (token), LDAP gibi protokoller veya Azure ve AWS gibi üçüncü taraf sağlayıcılar aracılığıyla kimlik doğrulaması yapar.
2. Vault, istemci isteğini dahili bir varlığa ve geçerli güvenlik politikalarına bağlayan bir erişim belirteci oluşturur.
3. İstemciler, Vault'ta bulunan kaynak yollarına (path) dayalı olarak gizli bilgiler ve şifreleme işlemleriyle etkileşim kurar.
4. Vault, istemci isteğini kaynak yolunda belirlenen politikalara göre yetkilendirir ve buna göre erişim izni verir veya erişimi reddeder.

```text
İstemci ──(kimlik doğrulama)──▶ Vault ──▶ Erişim belirteci (token)
İstemci ──(token + path)──────▶ Vault ──▶ Politika kontrolü ──▶ İzin ver / Reddet
```

### 7.8 Depolama

Vault verileri bir depolama arka ucunda (storage backend) tutar. Seçeneklerden biri **Integrated Storage (Raft)**'tır. Burada veriler, [etcd](#1-etcd) bölümünde de gördüğümüz Raft algoritmasıyla Vault düğümleri arasında çoğaltılarak depolanır.

---

## 8. AWS Secrets Manager

### 8.1 AWS Secrets Manager nedir?

AWS Secrets Manager, API anahtarları, veri tabanı kimlik bilgileri ve şifreleme anahtarları gibi bilgilerin güvenli bir şekilde depolanması ve yönetimi için tasarlanmış, AWS üzerindeki hazır bir secret yönetimi servisidir.

Servis tamamen **AWS tarafından yönetilir** (managed service). Yani [HashiCorp Vault](#7-hashicorp-vault)'un aksine kendimiz sunucu kurup bakımını yapmayız, sadece kullandığımız kadar öderiz.

#### Kim tarafından, ne zaman geliştirildi?

AWS Secrets Manager, **Amazon Web Services (AWS)** tarafından geliştirilmiştir ve **4 Nisan 2018'de**, San Francisco'daki AWS Summit etkinliğinde duyurulmuştur. İlk çıktığında MySQL, PostgreSQL ve Amazon Aurora için hazır rotasyon entegrasyonu ile geliyordu. Yani HashiCorp Vault'tan (Nisan 2015) yaklaşık üç yıl sonra çıkmıştır.

### 8.2 Temel özellikleri

#### 1. KMS ile şifreleme

KMS ile sırların built-in olarak şifrelenmesini ve şifrelerinin çözülmesini sağlar.

> **KMS (Key Management Service)**, AWS'nin şifreleme anahtarlarını oluşturup yönettiği, tamamen yönetilen servistir. Secret'lar Secrets Manager'a yazılırken bu anahtarlarla şifrelenir, okunurken çözülür. Düz metin olarak saklanamaz. Varsayılan AWS yönetimli anahtar yerine kendi oluşturduğumuz KMS anahtarı da seçilebilir. [Parameter Store](#9-aws-parameter-store) da aynı KMS servisini kullanır.

#### 2. IAM ile erişim kontrolü

IAM politikaları aracılığıyla ayrıntılı erişim kontrolü sağlar ve kimlik doğrulama ve yetkilendirme için IAM ile entegre çalışır.

> **AWS IAM (Identity and Access Management)**, AWS üzerindeki kaynaklara (Secrets Manager, Parameter Store, KMS, S3 vb.) kimlerin erişebileceğini ve bu kaynaklarla neler yapabileceğini güvenli bir şekilde yönetmenizi sağlayan merkezi kimlik ve erişim yönetimi servisidir.

Pratikte şöyle çalışır: Bir uygulamaya (örneğin bir EC2 makinesi ya da Lambda fonksiyonu) bir **IAM rolü** verilir. Bu role, sadece belirli secret'ı okuma izni veren bir **politika** bağlanır. Böylece uygulama için ayrıca bir şifre saklamaya gerek kalmaz, kimliği AWS zaten bilir.

```json
{
  "Effect": "Allow",
  "Action": "secretsmanager:GetSecretValue",
  "Resource": "arn:aws:secretsmanager:eu-west-1:123456789012:secret:prod/db-credentials-*"
}
```

Bu politika şunu söyler: bu rol sadece `prod/db-credentials` secret'ının değerini okuyabilir, başka bir şeye dokunamaz. Ayrıca secret'ın kendisine de politika (resource policy) yazılabilir. Bu sayede başka bir AWS hesabından erişim (cross-account) de verilebilir.

#### 3. Otomatik rotasyon (döndürme)

Secrets Manager'ın en güçlü özelliği budur. Bir secret'ı (örneğin veritabanı şifresini) belirlediğimiz aralıkla (örneğin her 30 günde bir) **otomatik olarak değiştirir.** Bu işi bir **AWS Lambda** fonksiyonu yapar.

```text
Zamanlayıcı süresi doldu
        │
        ▼
Secrets Manager rotasyon Lambda'sını çalıştırır
        │
        ├─ 1. createSecret → yeni şifre üretilir
        ├─ 2. setSecret    → yeni şifre veritabanında ayarlanır
        ├─ 3. testSecret   → yeni şifreyle bağlantı denenir
        └─ 4. finishSecret → yeni sürüm "güncel" olarak işaretlenir
```

Amazon RDS, Aurora gibi servisler için hazır şablonlar vardır. Diğer sistemler için Lambda fonksiyonunu kendimiz yazarız. Uygulama her seferinde API'den secret'ı çektiği için kod değiştirmeden her zaman en güncel şifreyi alır (secret'ı cache'liyorsak, cache süresi dolunca).

#### 4. Sürümleme

Bir secret'ın aynı anda birden fazla sürümü bulunabilir ve her sürüm bir etiketle işaretlenir (örneğin `AWSCURRENT` ve `AWSPREVIOUS`). Rotasyon sırasında eski sürüm bir süre yedek kalır, bu yüzden yeni şifre çalışmazsa geri dönmek mümkündür.

#### 5. Diğer özellikler

- **Çapraz bölge replikasyon:** Secret başka AWS bölgelerine çoğaltılabilir. Her kopya ayrı bir secret olarak ücretlendirilir.
- **Denetim (audit):** Secret'a kimin ne zaman eriştiği AWS'nin loglama servislerine (CloudTrail) yazılır.
- **Boyut sınırı:** Bir secret en fazla **64 KB** olabilir. Düz metin, JSON veya binary olabilir.

### 8.3 Nasıl çalışır?

```text
Uygulama (IAM rolü ile)
        │
        │ GetSecretValue isteği (HTTPS)
        ▼
 Secrets Manager
        │
        ├── IAM: bu rol bu secret'ı okuyabilir mi?
        ├── KMS: secret'ın şifresini çöz
        ▼
Uygulama secret'ı alır
```

Basit bir örnek (AWS CLI):

```bash
# Secret oluştur
aws secretsmanager create-secret \
  --name prod/db-credentials \
  --secret-string '{"username":"app_user","password":"..."}'

# Secret'ı oku
aws secretsmanager get-secret-value --secret-id prod/db-credentials
```

### 8.4 Fiyatlandırma

Kullandıkça ödeme modeli vardır:

- Secret başına aylık **0,40 dolar** (kullanılan süreye göre saatlik hesaplanır)
- Her **10.000 API çağrısı** için **0,05 dolar**
- Rotasyon Lambda'sı için standart Lambda fiyatı
- Her çoğaltılan kopya (replica) ayrı bir secret gibi ücretlendirilir

> ⚠️ Fiyatlar bölgeye ve zamana göre değişebilir. Güncel rakamlar için AWS'nin resmi fiyat sayfasına bakmak gerekir.

Bu yüzden uygulamada secret'ı **her istekte** yeniden çekmek yerine bir kez okuyup bellekte (cache) tutmak hem maliyeti hem gecikmeyi azaltır.

### 8.5 HashiCorp Vault ile fark

| | AWS Secrets Manager | HashiCorp Vault |
|---|---|---|
| Çalıştırma | AWS yönetir | Kendimiz kurarız (ya da HCP) |
| Bulut | Sadece AWS odaklı | Çoklu bulut / şirket içi |
| Gizli bilgi yenileme | **Rotasyon:** secret belirli aralıkla değiştirilir | **Dinamik secret:** her istekte benzersiz, kısa ömürlü kimlik bilgisi üretilir |
| Erişim kontrolü | IAM | Kendi politika sistemi |

Önemli fark şudur: Secrets Manager mevcut bir secret'ı **zaman zaman değiştirir**, ama aynı anda onu okuyan tüm servisler aynı geçerli değeri görür. Vault'un dinamik secret özelliğinde ise her servis, istek geldiği anda **kendine özel** bir kimlik bilgisi alır.

### 8.6 Parameter Store ile fark

AWS'de bir de **Systems Manager Parameter Store** vardır. İkisi benzer görünür ama amaçları farklıdır. Karşılaştırma [Parameter Store bölümünde](#97-secrets-manager-ile-fark) verilmiştir.

### 8.7 Spring Cloud Config ile ilişkisi

[Spring Cloud Config Server](#6-spring-cloud-config), yapılandırma kaynağı olarak AWS Secrets Manager'ı da kullanabilir. Kullandığımız Spring Cloud sürümüne göre ayar adları değişebileceği için resmi dokümana bakmak gerekir.

---

## 9. AWS Parameter Store

### 9.1 AWS Parameter Store nedir?

Parameter Store; sunucu, servis ya da uygulama ayarları ve ortam değişkenleri gibi hassas olan veya olmayan veriler dahil olmak üzere parametreleri depolamak ve yönetmek için kullanılır. Ayarları kodun dışında, merkezi bir yerde tutar ve uygulamalar oradan okur; böylece her ayar değişikliğinde uygulamayı yeniden derlemek gerekmez.

Parameter Store, **AWS Systems Manager (SSM)** servisinin bir parçasıdır. Servis AWS tarafından yönetilir (managed service), yani sunucu kurup bakımını yapmamız gerekmez.

#### Kim tarafından, ne zaman geliştirildi?

Parameter Store, **Amazon Web Services (AWS)** tarafından geliştirilmiştir. AWS re:Invent 2016 etkinliğinde, o dönem adı **EC2 Systems Manager** olan servisin bir özelliği olarak duyurulmuştur. Servis sonradan AWS Systems Manager adını almıştır. Sürümleme desteği ise Ekim 2017'de eklenmiştir. Yani AWS Secrets Manager'dan (Nisan 2018) önce çıkmıştır.

### 9.2 Temel özellikleri

#### 1. Key-value yapısı ve hiyerarşi

Key-value (anahtar-değer) şeklinde veriler saklanır. Komut dosyalarımızda, komutlarımızda, SSM belgelerimizde ve yapılandırma ve otomasyon iş akışlarımızda, Systems Manager parametrelerine, parametreyi oluştururken belirttiğimiz benzersiz adı kullanarak başvurabiliriz.

Yapılandırma verileri ve şifrelenmiş dizeler **hiyerarşik yapıda** saklanabilir. Bu, klasör yapısına benzer bir isimlendirmedir:

```text
/prod/order-service/db-url
/prod/order-service/db-password
/prod/payment-service/api-key
/dev/order-service/db-url
```

Bu sayede ortama ve servise göre ayrım yapılır ve tek bir istekle bir yolun altındaki tüm parametreler çekilebilir (`get-parameters-by-path`).

#### 2. Parametre türleri

| Tür | Açıklama |
|---|---|
| **String** | Düz metin değer. Şifrelenmez |
| **StringList** | Virgülle ayrılmış değerler listesi |
| **SecureString** | Değeri KMS ile şifrelenen hassas parametre (şifre, token vb.) |

#### 3. KMS ile şifreleme

**SecureString** türündeki parametrelerin değerleri KMS anahtarıyla şifrelenerek saklanır (encryption at rest). Parametrenin adı düz metin kalır, sadece değeri şifrelenir. Varsayılan olarak AWS'nin yönettiği `alias/aws/ssm` anahtarı kullanılır, istenirse kendi KMS anahtarımız seçilebilir. KMS'in ne olduğu için [Secrets Manager bölümüne](#8-aws-secrets-manager) bakınız.

**Şifre çözme nasıl olur?** Şifreleme ve çözme işini Parameter Store, KMS'i kullanarak kendisi yapar. Uygulamanın kendi başına şifre çözmesi gerekmez. Ancak SecureString parametreyi alırken **şifresinin çözülmesini istediğimizi** belirtmemiz gerekir. Bu seçenek `WithDecryption=true` (CLI'da `--with-decryption`) şeklindedir. Bu seçenek verilmezse parametre **şifreli (ciphertext) olarak** gelir.

> ⚠️ Bu seçeneğin adı `WithDecryption`'dır (şifre çözme), `WithEncryption` değildir. Ayrıca çözme işleminin yapılabilmesi için uygulamanın rolünün KMS anahtarı üzerinde de yetkisi olmalıdır.

#### 4. Sürümleme

Parametre değişikliklerini otomatik olarak takip eder, her güncellemede yeni bir sürüm numarası oluşur. Böylece önceki sürümlere erişebilir ve gerekirse eski değeri tekrar kullanabiliriz.

#### 5. IAM ile erişim kontrolü ve denetim

IAM politikalarını kullanarak parametrelere erişimi kısıtlar. Hiyerarşik isimler burada işe yarar: bir servise sadece kendi yolunun altına erişim verilebilir.

```json
{
  "Effect": "Allow",
  "Action": ["ssm:GetParameter", "ssm:GetParametersByPath"],
  "Resource": "arn:aws:ssm:eu-west-1:123456789012:parameter/prod/order-service/*"
}
```

Bu politika, sadece `/prod/order-service/` altındaki parametrelerin okunmasına izin verir. Parameter Store'a yapılan tüm çağrılar AWS CloudTrail'e kaydedilir ve böylece denetlenebilir.

#### 6. Yüksek erişilebilirlik

Parametre deposu, bir AWS bölgesindeki birden fazla kullanılabilirlik bölgesinde (availability zone) barındırıldığı için parametreler güvenilir bir şekilde saklanır. Servis bölgeseldir, yani parametreler oluşturulduğu bölgeye aittir.

#### 7. AWS servisleriyle entegrasyon

AWS Lambda, EC2, ECS ve diğer hizmetlerle sorunsuz yapılandırma yönetimi için birlikte çalışır. Ayrıca CloudFormation, CodePipeline ve Systems Manager'ın Run Command gibi özelliklerinden de parametrelere başvurulabilir.

### 9.3 Avantajları

- Sunucu yönetimi gerektirmeyen, güvenli, ölçeklenebilir ve barındırılan (managed) bir yapılandırma ve gizli bilgi yönetimi hizmeti kullanırız.
- Verilerimizi kodumuzdan ayırarak güvenlik durumumuzu iyileştiririz.
- Erişim kontrolünü ve denetimini ayrıntılı seviyelerde yaparız.

### 9.4 Standart ve gelişmiş (Advanced) katman

Parameter Store'da iki katman vardır. Her parametre için ayrı ayrı seçilir ve varsayılan olarak standart katman kullanılır.

| | Standart | Gelişmiş (Advanced) |
|---|---|---|
| Parametre sayısı (bölge/hesap başına) | 10.000 | 100.000 |
| Değer boyutu | 4 KB | 8 KB |
| Ücret | Ücretsiz | Ücretli |
| Parametre politikaları | ❌ | ✅ |

- **Parametre politikaları (parameter policies):** Sadece gelişmiş katmanda vardır. Bir parametreye son kullanma tarihi koymayı, süre dolmadan önce bildirim almayı ya da uzun süredir değişmeyen parametreler için uyarı almayı sağlar.
- **Intelligent-Tiering:** Parametreyi sistemin ihtiyaca göre otomatik olarak gelişmiş katmana geçirmesini sağlayan seçenektir.
- Standarttan gelişmişe geçilebilir, **gelişmişten standarda geri dönülemez.** Dönmek için parametreyi silip yeniden oluşturmak gerekir.

### 9.5 Nasıl çalışır?

```text
Uygulama (IAM rolü ile)
        │
        │ GetParameter isteği (WithDecryption=true)
        ▼
 Parameter Store
        │
        ├── IAM: bu rol bu parametreyi okuyabilir mi?
        ├── KMS: (SecureString ise) değeri çöz
        ▼
Uygulama değeri alır
```

Basit bir örnek (AWS CLI):

```bash
# Hassas parametre oluştur
aws ssm put-parameter \
  --name /prod/order-service/db-password \
  --type SecureString \
  --value "..."

# Parametreyi oku (şifresi çözülmüş halde)
aws ssm get-parameter \
  --name /prod/order-service/db-password \
  --with-decryption

# Bir yolun altındaki tüm parametreleri oku
aws ssm get-parameters-by-path \
  --path /prod/order-service \
  --recursive \
  --with-decryption
```

### 9.6 Fiyatlandırma

- Standart parametreler **ücretsizdir.**
- Gelişmiş parametreler, parametre başına aylık küçük bir ücrete tabidir.
- Yüksek verimli (higher throughput) API çağrıları ayrıca ücretlendirilir.

> ⚠️ Fiyatlar zamanla değişebilir. Güncel rakamlar için AWS'nin resmi fiyat sayfasına bakmak gerekir.

### 9.7 Secrets Manager ile fark

[AWS Secrets Manager](#8-aws-secrets-manager) ile Parameter Store benzer görünür ama amaçları farklıdır:

| | Parameter Store | Secrets Manager |
|---|---|---|
| Asıl amaç | Yapılandırma değerleri ve küçük secret'lar | Secret saklama + rotasyon |
| Otomatik rotasyon | ❌ Yok | ✅ Var |
| Boyut sınırı | 4 KB (standart), 8 KB (gelişmiş) | 64 KB |
| Ücret | Standart parametreler ücretsiz | Secret başına 0,40 $/ay |
| Şifreleme | Sadece `SecureString` türünde | Her zaman |
| Çapraz bölge çoğaltma | ❌ Hazır değil | ✅ Hazır |

Kısaca: rotasyon gereken gizli bilgiler (veritabanı şifresi gibi) için Secrets Manager, rotasyon gerekmeyen sıradan ayarlar ve küçük secret'lar için Parameter Store daha uygundur.

### 9.8 Spring ile ilişkisi

Spring tarafında **Spring Cloud AWS**, Parameter Store'u uygulamanın yapılandırma kaynağı olarak kullanabilir. Böylece parametreler `@Value` gibi mekanizmalarla okunur. Kullanılan sürüme göre ayar adları değişebileceği için resmi dokümana bakmak gerekir.

---

## 10. Google Cloud Secret Manager

Secret Manager, API anahtarları, kullanıcı adları, parolalar, sertifikalar ve token'lar gibi hassas verileri güvenli bir şekilde saklamaya ve yönetmeye olanak tanıyan bir gizli bilgi ve kimlik bilgisi yönetimi hizmetidir.

Dağıtık bir sistemde birden fazla uygulama (Backend 1, Backend 2, Worker) aynı gizli bilgilere (örneğin `DB_PASSWORD`) ihtiyaç duyabilir. Bu bilgileri her uygulamanın koduna ayrı ayrı yazmak yerine Secret Manager merkezi bir kaynak olur: bir şifre değiştiğinde tek yerde güncellenir, Git deposuna yanlışlıkla yüklenme riski azalır ve erişim izinleri tek noktadan yönetilir.

```text
                 Secret Manager
                       │
                 DB_PASSWORD
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
      Backend 1    Backend 2      Worker
```

Uygulamalar, gerekli izinlere sahip olduklarında gizli bilgileri Secret Manager üzerinden alır.

### 10.1 Secret Version (gizli bilgi sürümleri)

Secret Manager, gizli bilgilerin farklı sürümlerini yönetmeyi sağlar. Sürümler, kademeli dağıtımları ve gerektiğinde geri almayı kolaylaştırır.

Örneğin, bir veritabanı şifresi değiştirildiğinde eski sürümün üzerine yazmak yerine yeni bir sürüm oluşturulabilir.

```text
DATABASE_PASSWORD
    ├── Version 1 → İlk şifre
    ├── Version 2 → Önceki şifre
    └── Version 3 → Güncel şifre
```

Bu sayede gizli bilgi yanlışlıkla değiştirilirse veya yeni sürümle ilgili bir sorun yaşanırsa önceki sürüme geri dönülebilir. Ayrıca artık ihtiyaç duyulmayan sürümler devre dışı bırakılabilir veya silinebilir.

### 10.2 Encryption (şifreleme)

Secret Manager'da gizli bilgiler hem aktarım sırasında hem de depolanırken şifrelenir.

- **Aktarım sırasında şifreleme (Encryption in transit):** Gizli bilgiler, istemci ile hizmet arasında aktarılırken TLS kullanılarak korunur.
- **Depolama sırasında şifreleme (Encryption at rest):** Gizli bilgiler depolanırken varsayılan olarak AES-256 tabanlı şifreleme kullanılır. Bu, Google'ın varsayılan şifreleme mekanizmasının bir parçasıdır.

Daha ayrıntılı şifreleme anahtarı kontrolü isteyen kullanıcılar, **Customer-Managed Encryption Keys (CMEK)** kullanabilir. Bu yöntemde kullanıcı, Cloud KMS aracılığıyla yönettiği şifreleme anahtarını Secret Manager ile ilişkilendirir.

Varsayılan şifreleme ile CMEK arasındaki temel fark, şifrelemenin varlığı değil, kullanılan anahtar üzerindeki kontrol düzeyidir.

### 10.3 IAM (Identity and Access Management)

IAM, kullanıcıların ve servislerin hangi kaynaklar üzerinde hangi işlemleri yapabileceğini belirler.

Secret Manager'da IAM rolleri ve koşulları kullanılarak gizli bilgilere erişim sınırlandırılabilir. Bir kullanıcıya veya servise Secret Manager üzerinde genel yönetici yetkisi vermek yerine yalnızca ihtiyaç duyduğu secret üzerinde belirli bir rol verilebilir.

Örneğin, bir backend servisinin yalnızca `MQTT_PASSWORD` gizli bilgisine erişmesi istenebilir. Bunun için ilgili secret üzerinde uygun IAM rolü ve izinleri tanımlanır; backend'in diğer secret'lara erişmesi engellenir.

```text
Backend
   ↓
  IAM
   ↓
Secret Manager
   ↓
MQTT_PASSWORD
```

**Least Privilege (En Az Ayrıcalık):** Kullanıcılara ve servislere yalnızca görevlerini yerine getirmek için ihtiyaç duydukları izinleri verme prensibidir.

TLS ve IAM farklı amaçlara hizmet eder:

- **TLS:** Verinin aktarım sırasında korunmasını sağlar.
- **IAM:** Kimlerin hangi kaynaklara erişebileceğini ve hangi işlemleri yapabileceğini kontrol eder.

### 10.4 Replication (çoğaltma)

Secret Manager, gizli bilgileri farklı konumlarda çoğaltarak yüksek kullanılabilirliği ve dayanıklılığı destekler. Böylece tek bir konumda yaşanan sorunların hizmet üzerindeki etkisi azaltılabilir.

Secret Manager'da iki temel çoğaltma türü vardır.

#### Automatic Replication (otomatik çoğaltma)

Google Cloud, çoğaltma konumlarını kendisi yönetir. Kullanıcının belirli bölgeleri tek tek seçmesi gerekmez. Fiyatlandırma, otomatik çoğaltma için geçerli ücretlendirme modeline göre yapılır.

#### User-managed Replication (kullanıcı tarafından yönetilen çoğaltma)

Kullanıcı, gizli bilgilerin hangi Google Cloud bölgelerinde çoğaltılacağını belirleyebilir. Çoğaltma konumları ihtiyaca göre seçilir ve coğrafi konum ile veri yerleşimi gereksinimleri üzerinde daha fazla kontrol sağlanır. Fiyatlandırma, seçilen konumlara ve geçerli ücretlendirme modeline bağlıdır.

### 10.5 Secret Manager ile etcd ve Consul'un birlikte kullanılması

[etcd](#1-etcd) ve [Consul](#2-consul) normal konfigürasyonun, Secret Manager ise hassas bilgilerin yönetimine odaklanır. Bu yüzden birbirinin alternatifi olmak zorunda değildir; aynı sistem içinde birlikte çalışabilir (normal ve gizli bilgi ayrımı için [giriş bölümündeki tabloya](#konfigürasyon-ile-secret-arasındaki-fark) bakınız):

```text
                    Uygulama
                       │
             ┌─────────┴─────────┐
             ↓                   ↓
         Consul / etcd      Secret Manager
             │                   │
             ↓                   ↓
      Normal konfigürasyon    Gizli bilgiler
             │                   │
             ├── PORT            ├── DB_PASSWORD
             ├── LOG_LEVEL       ├── API_KEY
             ├── DB_HOST         └── JWT_SECRET
             └── SERVICE_URL
```

---

# Kaynakça

### etcd

- [YouTube: etcd anlatımı](https://www.youtube.com/watch?v=OmphHSaO1sE)
- [etcd.io: Why etcd?](https://etcd.io/docs/v3.1/learning/why/)
- [Medium: etcd nedir, operasyonel detaylar ve monitoring](https://medium.com/@mustafaakgundev/etcd-nedir-baz%C4%B1-operasyonel-detaylar-ve-%C3%B6nemli-monitoring-i%CC%87pu%C3%A7lar%C4%B1-a3a7a6683186)
- [IBM: etcd](https://www.ibm.com/think/topics/etcd)
- [etcd.io](https://etcd.io/)

### Consul

- [Consul 101 (Trendyol Tech)](https://medium.com/trendyol-tech/consul-101-d645dc4ea13f)
- [YouTube: Consul anlatımı](https://www.youtube.com/watch?v=mxeMdl0KvBI&t=5s)
- [Consul ile Service Discovery Dünyasına Bakış](https://medium.com/@selcukusta/consul-i%CC%87le-service-discovery-d%C3%BCnyas%C4%B1na-bak%C4%B1%C5%9F-60d81c06a45d)

### Apache ZooKeeper

- [What the hell is ZooKeeper? (Medium)](https://medium.com/@vinciabhinav7/what-the-hell-is-zookeeper-46a1799c828c)
- [Apache ZooKeeper nedir, ne işe yarar, nasıl kullanılır (TurkMMO Forum)](https://forum.turkmmo.com/konu/3776182-apache-zookeeper-nedir-ne-ise-yarar-nasil-kullanilir-genel-bakis/)
- [USOM Güvenlik Bildirimi TR-26-1115](https://siberguvenlik.gov.tr/guvenlik-bildirimleri/detay/tr-26-1115)
- [Apache ZooKeeper: Security](https://zookeeper.apache.org/security/)
- [Apache ZooKeeper: Releases](https://zookeeper.apache.org/releases/)

### Kubernetes ConfigMap, Pod, Deployment, HPA

- [Kubernetes ConfigMap oluşturma ve kullanma (DevOps Türkiye)](https://medium.com/devopsturkiye/kubernetes-configmap-olu%C5%9Fturma-ve-kullanma-be71b6f3df72)
- [Kubernetes Docs: ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)

### Kubernetes Secret

- [Kubernetes Docs: Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Kubernetes Docs: Encrypting Confidential Data at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)

### Spring Cloud Config

- [Spring Cloud Config Server nedir? (Kadir Demirel)](https://medium.com/@kadirdemirell/spring-cloud-config-server-nedir-fdd3381fd182)
- [Spring Cloud Config Server nedir? (Kayhan Öztürk)](https://kayhanozturk.medium.com/spring-cloud-config-server-nedir-01a268d8c6e6)
- [GeeksforGeeks: Managing Configuration for Microservices with Spring Cloud Config](https://www.geeksforgeeks.org/advance-java/managing-configuration-for-microservices-with-spring-cloud-config/)

### HashiCorp Vault

- [Devoteam: What is HashiCorp Vault?](https://www.devoteam.com/expert-view/what-is-hashicorp-vault/)
- [HashiCorp Developer: What is Vault?](https://developer.hashicorp.com/vault/docs/about-vault/what-is-vault)

### AWS Secrets Manager

- [AWS: Introducing AWS Secrets Manager](https://aws.amazon.com/de/about-aws/whats-new/2018/04/introducing-aws-secrets-manager)
- [InfoQ: AWS Secrets Manager](https://www.infoq.com/news/2018/04/aws-secret-manager-manage/)
- [AWS: Secrets Manager now supports larger size for secrets](https://aws.amazon.com/about-aws/whats-new/2020/03/aws-secrets-manager-now-supports-larger-size-for-secrets-and-higher-request-rate-for-getsecretvalue-api)
- [GeeksforGeeks: AWS Secrets Manager](https://geeksforgeeks.org/aws-secrets-manager)
- [AWS Secrets Manager Docs](https://docs.aws.amazon.com/secretsmanager/)

### AWS Parameter Store

- [AWS Secrets Manager ve Parameter Store (Oğuz Türkay)](https://oguz-turkay.medium.com/aws-secrets-manager-ve-parameter-store-anahtar-y%C3%B6netimi-ve-konfig%C3%BCrasyon-i%C3%A7in-do%C4%9Fru-servisi-7bbdd705b093)
- [AWS Parameter Store (Joud Wawad)](https://joudwawad.medium.com/aws-parameter-store-43185b34af92)
- [AWS Systems Manager Parameter Store Docs](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)
- [AWS KMS Docs: Parameter Store](https://docs.aws.amazon.com/kms/latest/developerguide/services-parameter-store.html)
- [AWS Blog: Parameter Store adds support for parameter versions](https://aws.amazon.com/blogs/mt/amazon-ec2-systems-manager-parameter-store-adds-support-for-parameter-versions/)

### Google Cloud Secret Manager

- [Secret Manager Overview](https://docs.cloud.google.com/secret-manager/docs/overview)
- [Secret Manager: Customer-Managed Encryption Keys (CMEK)](https://docs.cloud.google.com/secret-manager/docs/cmek)
