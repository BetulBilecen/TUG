# REST API ve HTTP

## İçindekiler

- [REST API](#rest-api)
  - [API, Web API ve REST Nedir?](#api-web-api-ve-rest-nedir)
  - [REST Mimarisinin Temel Prensipleri](#rest-mimarisinin-temel-prensipleri)
  - [RESTful Servis](#restful-servis)
- [HTTP](#http)
  - [HTTP Metotları](#http-metotları)
  - [Klasik Bir GET Sorgusu Örneği](#klasik-bir-get-sorgusu-örneği)

---

## REST API

### API, Web API ve REST Nedir?
* **API (Application Programming Interface - Uygulama Programlama Arayüzü):** Farklı uygularamaların, yazılımların veya cihazların birbirine nasıl bağlanabileceğini ve birbiriyle
nasıl iletişim kuracağını tanımlayan kurallar bütünüdür.


* **WEB API (Web Application Programming Interface - Web Uygulama Programlama Arayüzü):** Web tabanlı yazılımların ve hizmetlerin diğer uygulamalarla ağ üzerinden iletişim kurmasını sağlayan bir yapıdır. İstemci (client), HTTP (Hypertext Transfer Protocol - Hiper Metin Transfer Protokolü) üzerinden bir istek (request) gönderir; API aracılığıyla hedeflenen kaynağa (resource) ulaşılır ve sunucu (server) istenen yanıtı genellikle JSON veya XML formatında geri döndürür.


* **REST (Representational State Transfer - Temsili Durum Transferi):** İstemci (client) ve sunucu (server) arasındaki iletişimi düzenleyen mimari bir yaklaşımdır. Kaynaklara erişmek ve işlem yapmak için URL'ler ve standart HTTP protokolü (GET, POST, PUT, DELETE vb.) kullanılır. Sunucudan dönen yanıtlar çoğunlukla JSON veya XML formatında iletilir; ağırlıklı olarak JSON formatı tercih edilir. REST mimarisinin 6 temel prensibi (kısıtı) şunlardır:

### REST Mimarisinin Temel Prensipleri

-----
🔹 **Stateless (Durum Bilgisizliği):** Sunucu (server), istemcinin (client) geçmiş durumunu, oturum bilgilerini veya önceki isteklerini hafızasında tutmaz. Her istek birbirinden bağımsızdır; bu nedenle istemci, her istek gönderdiğinde sunucunun işlemi tamamlayabilmesi için gereken kimlik/yetki bilgilerini (örneğin token, kullanıcı bilgisi vb.) isteğe tekrar eklemek zorundadır. Bu durum sunucunun ölçeklenebilirliğini artırıp yönetimini kolaylaştırırken, her istekte verilerin tekrar taşınması ağ yükü ve maliyet açısından dezavantaj oluşturabilir.

🔹 **Uniform Interface (Tek Tip Arayüz):** İstemcinin türüne (web, mobil, masaüstü vb.) bakılmaksızın aynı kaynağa yapılan tüm API istekleri standart bir formatta ve tutarlı URL yapıları üzerinden gerçekleştirilmelidir. Aynı işlemler için her platformda standart HTTP metotları (GET, POST vb.) ve aynı URL yapısı kullanılır. Sunucu yanıtları ise gereksiz veri karmaşasından arındırılarak istemcinin ihtiyaç duyduğu temel alanları (ad, soyad, e-posta vb.) standart ve eksiksiz bir yapıda sunmalıdır.

🔹 **Cacheable (Önbelleklenebilirlik):** Sık erişilen verilerin kopyalarını istek-yanıt hattı boyunca uygun önbelleklerde (istemci, CDN, proxy vb.) saklama yeteneğine denir. Önbellekleme olayı veri iletim hattında olduğu için _stateless_ kuralını bozmaz. İstemcinin talep ettiği veri önbellekte mevcutsa, istek sunucuya ulaşmadan doğrudan bu kopyadan yanıtlanır; verinin önbellekte bulunmadığı durumlarda ise istek sunucuya iletilir. Bu sayede sunucuyu fazla ağ trafiğinden kurtarılmış olur.

Kaynak sunucu tarafından bir yanıtın önbelleğe alınıp alınamayacağını, alınabiliyorsa kim tarafından ve ne kadar süreyle saklanabileceğini tanımlanır.

Örnek Tarih: ``Expires: Fri, 20 May 2027 19:20:50 GMT``

Önbelleklenebilen http komutu `GET`'dir. Diğer komutlar POST,PUT,DELETE vb. veri içeriğini değiştirdiği için uncacheable yani önbelleklenemez kabul edilir.

🔹 **Client-Server (İstemci-Sunucu Mimarisi):** İletişim sadece istemcinin başlattığı istekler ve sunucunun bunlara verdiği yanıtlar üzerinden yürür. Sunucu kendi başına istemciye veri akışı başlatmaz.

🔹 **Layered System (Katmanlı Sistem):** İstemci ve sunucu arasında güvenlik duvarı, proxy (vekil sunucu) veya önbellek (cache) katmanları bulunabilir. Araya katman girmesi durumunda her katman kendisi ile doğrudan iletişimde olan katmanı tanır.

🔹 **Code on Demand (İsteğe Bağlı Kod):** Opsiyonel bir kuraldır. Sunucunun istemciye çalıştırabileceği bir komut dosyası (kod) göndermesine olanak tanır.

----------

### RESTful Servis

* **RESTful Servis:** Yukarıdaki REST mimari prensiplerinin tümünü eksiksiz uygulayan servislere verilen addır. Popüler örneklere Twilio, Stripe ve Google Maps API servisleri gösterilebilir.

![Rest API](https://images.ctfassets.net/vwq10xzbe6iz/5sBH4Agl614xM7exeLsTo7/9e84dce01735f155911e611c42c9793f/rest-api.png)

---

## HTTP
HTTP (Hypertext Transfer Protocol - Köprü Metni Aktarım Protokolü), bir web tarayıcısı ile bir web sunucusu arasındaki iletişimi sağlayan bir protokoldür. İstemci-sunucu (Client-Server) mantığıyla çalışır.
İstek doğrultusunda sunucudan gelen HTML, CSS, JavaScript, JSON ve XML dosyalarının istemci/kullanıcı cihazında işlenmesini, çalıştırılmasını ve görüntülenmesini sağlar.

### HTTP Metotları

* **GET :** Belirtilen URL'deki kaynağı almak için kullanılır. Yalnızca okuma yapar, veride herhangi bir değişikliğe gitmez; bu nedenden dolayı önbelleklenebilir (cacheable).
* **HEAD :** GET ile aynı mantıkta çalışır fakat yanıtta Body (veri bloğu / JSON vb.) dönmez; yalnızca Header bilgileri (içerik türü, dosya boyutu, yetkilendirme durumu vb.) döner. Kaynağın varlığını veya en son ne zaman değiştirildiğini (Last-Modified) kontrol etmek için kullanılır.
* **POST :** Sunucuya veri gönderip yeni bir kaynak oluşturmak, göndermek veya controller kaynaklarını çalıştırmak için kullanılır. Veriler Body ile gönderilir.
* **PUT :** Hedef kaynağın bütünsel (tüm alanlarıyla) güncellenmesi veya belirtilen URI'da kaynak henüz yoksa sıfırdan oluşturulması (upsert) amacıyla kullanılır.
* **PATCH :** Kaynağın tamamını değil, yalnızca belirli bir parçasını güncellemek için kullanılır. Amaç maliyeti düşürmektir.
* **DELETE :** Kaynağı kalıcı olarak silmek için kullanılır. Kalıcı olmayan silmeler için uygulamaya özel bir controller kullanılır.
* **OPTIONS :** İlgili URL'nin hangi HTTP metotlarını desteklediğini öğrenmek için gönderilir. Desteklenen metotlar yanıttaki _Allow_ başlığında bildirilir.
* **QUERY :** QUERY 19 Haziran 2026'da yayımlanan, HHTP protokolüne eklenen bir istek metodudur. Bu metot geleneksel HTTP isteklerindeki GET ve POST yöntemlerinin sorgulama/arama esnasında yaşanılan şu sorunlara çözüm olarak geliştirilmiştir.

| GET | POST |
| :--- | :--- |
| • Url boyut sınırlamaları (8000 maksimum karakter kullanımı) ve karmaşık sorgular oluşturmanın zorluğu<br><br/>• Parametrelerin loglarda görünmesi bir güvenlik sorunu olarak açığa çıkar. | • Ağ bağlantısı kesilirse, istek güvenli bir şekilde yeniden gönderilemez. (POST İdempotent değildir)<br><br/>• Standart http önbelleklemesi (cache) yok |

QUERY sayesinde ağ bağlantısı kesilse dahi veride herhangibir bozulma olmadan ilgili isteği tekrar gönderebiliriz. Ayrıca karmaşık sorguların request body içerisinde gönderilebilmesi sayesinde URL uzunluğu sınırlamalarının ve sorgu parametrelerinin URL üzerinde açıkça görünmesinin önüne geçilmesi amaçlanır. Bu sayede özellikle büyük ve karmaşık sorguların daha uygun bir şekilde sunucuya iletilmesi sağlanır.

### Klasik Bir GET Sorgusu Örneği

![HTTPS Requst](https://www.cloud4y.ru/upload/medialibrary/4c0/hn5x5w7tx2pa0t3m1us71vh51dthf4kg/2.jpg)
Yukarıda klasik bir GET sorgusunun örneği verilmiştir.
