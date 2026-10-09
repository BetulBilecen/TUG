# gRPC

## İçindekiler

- [gRPC Nedir?](#grpc-nedir)
  - [gRPC'yi Geleneksel RPC'den Ayıran Nedir?](#grpcyi-geleneksel-rpcden-ayıran-nedir)
- [Protocol Buffers (Protobuf) Nedir?](#protocol-buffers-protobuf-nedir)
  - [gRPC'nin Temel Özellikleri](#grpcnin-temel-özellikleri)
  - [Dil Bağımsızlığı Nasıl Sağlanır?](#dil-bağımsızlığı-nasıl-sağlanır)
- [HTTP/2'nin HTTP/1.1'den Farkları](#http2nin-http11den-farkları)
- [gRPC Haberleşme Yöntemleri](#grpc-haberleşme-yöntemleri)
  - [Deadline](#deadline)
  - [Cancellation](#cancellation)
- [gRPC Status Codes](#grpc-status-codes)
- [gRPC'nin Avantajları](#grpcnin-avantajları)
- [gRPC'nin Dezavantajları](#grpcnin-dezavantajları)
- [gRPC'nin Implementasyonu (Uygulaması)](#grpcnin-implementasyonu-uygulaması)
- [gRPC ve REST Karşılaştırması](#grpc-ve-rest-karşılaştırması)
  - [Peki Madem gRPC Bu Kadar Güçlü, Neden Hâlâ REST Kullanıyoruz?](#peki-madem-grpc-bu-kadar-güçlü-neden-hâlâ-rest-kullanıyoruz)
- [Kaynaklar](#kaynaklar)

---

## gRPC Nedir?

Google tarafından 2015 yılında geliştirilen ve ardından CNCF'ye (Cloud Native Computing Foundation) devredilen **gRPC**, kökleri 1970'lere dayanan geleneksel **Remote Procedure Call (RPC)** mimarisine dayanan modern bir framework'tür. İstemci-sunucu (client-server) modeliyle çalışan bu mimari; uzaktaki bir sunucu veya serviste çalışan bir metodu, sanki projemizin içindeki yerel bir fonksiyonmuş gibi doğrudan çağırabilmemizi sağlar.

### gRPC'yi Geleneksel RPC'den Ayıran Nedir?

Eski RPC uygulamaları çoğunlukla tek bir programlama diline bağımlıydı, hantal metin formatları yüzünden yavaştı, güvenlik duvarlarını aşmakta zorlanan özel ağ protokolleri kullanıyordu ve yalnızca tek yönlü istek-yanıt (request-response) modeliyle sınırlıydı.

gRPC ise bu sınırları yıkarak hantal metinler yerine **Protocol Buffers (Protobuf)** ile ikili (binary) serileştirme yapar, taşıma protokolü olarak **HTTP/2** kullanarak çift yönlü veri akışını (streaming) ve çoklamayı (multiplexing) standart hale getirir.

Kısacası gRPC; Protobuf'ın kompakt ikili yapısı ve HTTP/2'nin getirdiği güçle 50 yıllık RPC fikrini yeniden tanımlayarak, modern mikroservis ve bulut mimarilerinde servisler arası iletişimi hızlı, hafif ve düşük gecikmeli hale getirir.

## Protocol Buffers (Protobuf) Nedir?

gRPC için kullanılan verinin **binary olarak serialization işlemini gerçekleştiren bir veri serileştirme formatıdır.** Bir adet `.proto` dosyasına ilgili servisin hangi metotlara sahip olacağı ve bu metotların hangi mesajları alıp hangi mesajları döndüreceği yazılır. gRPC ile birlikte kullanılan `protoc` derleyicisi bu `.proto` dosyasını okuyarak hedeflediğimiz programlama dilinde yüzlerce satırlık hazır kod üretir.

> **Binary Serialization:** Bir verinin ağ üzerinden gönderilebilmesi veya saklanabilmesi için binary (byte) dizisine dönüştürülmesi işlemidir.

Bu kod bloklarında temel olarak şu işlemler gerçekleştirilir:
* İlgili veri sınıflarını otomatik olarak oluşturur.
* Karşı taraftaki sunucuya, sanki kendi bilgisayarımızdaki yerel bir fonksiyonu çağırıyormuşuz gibi istek atmamızı sağlayan istemci sınıfını (stub) üretir.
* Sunucu tarafında gelen gRPC isteklerini karşılayıp ilgili fonksiyona yönlendirebilmemiz için gerekli sunucu altyapısını oluşturur.

Bu özellik sayesinde REST mimarilerinde olduğu gibi HTTP isteklerini manuel olarak oluşturma, verileri parse etme ve ağ iletişiminin birçok detayını kendimiz yönetme gibi işlemlerle uğraşmamıza gerek kalmaz. gRPC ve Protobuf bu işlemlerin büyük bir kısmını bizim için gerçekleştirir. HTTP/2 ve Protobuf'un binary yapısı sayesinde de servisler arası iletişimin hızlı, hafif ve düşük gecikmeli olmasına katkı sağlar.


![Protocol Buffer](Images/Protocol%20Buffer.jpg)


Bizim yapmamız gereken son olarak ilgili iş mantığını yazmak, yani implementasyonları gerçekleştirmektir. **Buradaki implementasyon, oluşturulan metotların ne yapacağını belirleyip ilgili kodları yazmak anlamına gelir.** Örneğin `.proto` dosyasında `GetUser` adında bir metot tanımladıysak, gRPC bizim için bu metodun kullanılabilmesi için gerekli yapıyı oluşturur. Ancak bu metodun veritabanından kullanıcıyı nasıl bulacağı, hangi kontrolleri yapacağı ve hangi veriyi döndüreceği gibi **iş mantığını bizim yazmamız gerekir.** Yani burada oluşturulan hazır sunucu sınıflarından kalıtım alarak veya bu sınıfların sağladığı yapıları kullanarak oluşturmak istediğimiz fonksiyonların içeriğini geliştiririz.

İstemci tarafında da aynı kolaylık vardır; istemci, sanki kendi yerel dosyasındaki bir metodu çağırır gibi oluşturulan **stub** üzerinden tek satırda sunucuya istek gönderebilir.

### gRPC'nin Temel Özellikleri

Protocol Buffers ile kullanılan gRPC'nin özellikleri:
* Herhangi bir programlama diline bağlı değildir.
* Farklı programlama dillerine kolayca entegre olabilir.
* Veriler binary formatta serileştirilir. HTTP/1.1 ile kullanılan JSON gibi metin tabanlı formatlara göre daha kompakt bir yapı sunar.
* Aynı bağlantı üzerinden birden fazla isteğin gönderilebilmesini sağlayan multiplexing (çoğullama) desteğine sahiptir.
* İstemci ve sunucunun aynı bağlantı üzerinden karşılıklı olarak veri gönderebilmesini sağlayan çift yönlü iletişim desteğine sahiptir.
* Streaming özelliği sayesinde verilerin parça parça gönderilmesine olanak sağlar. Bu da sürekli veri akışı gereken durumlarda kullanılmasını sağlar.
* TLS/SSL desteği sayesinde şifreli ve güvenli iletişim kurulabilir.
* Loglama, izleme (monitoring) ve kimlik doğrulama gibi ara katman işlevleri için zengin Interceptor desteği sunar.

### Dil Bağımsızlığı Nasıl Sağlanır?

Peki bu dil bağımsızlığını gRPC nasıl sağlar? gRPC'de `.proto` dosyası hazırlanırken C#, Python gibi belirli bir programlama dili kullanılmaz. Bunun yerine Interface Definition Language (IDL) kullanılır. Böylece servislerin hangi metotlara sahip olacağı, bu metotların hangi parametreleri alacağı ve hangi verileri döndüreceği herhangi bir programlama diline bağlı kalmadan tanımlanabilir.

Daha sonra `protoc` derleyicisi bu `.proto` dosyasını okuyarak hedeflediğimiz programlama diline uygun kodları oluşturur. Böylece örneğin sunucu C# ile, istemci ise Python ile geliştirilse bile ikisi de aynı `.proto` dosyasındaki servis tanımına göre çalışabilir.

## HTTP/2'nin HTTP/1.1'den Farkları

Peki bu bahsettiğimiz HTTP/2'nin HTTP/1.1'den farkı nedir?

HTTP, web tarayıcıları ile sunucular arasında veri alışverişi yapılmasını sağlayan temel iletişim protokolüdür. HTTP/2 ise bu protokolün hız, performans ve verimlilik odaklı geliştirilmiş ikinci ana sürümüdür.

HTTP/2, internetin temel iletişim protokolü olan HTTP'nin HTTP/1.1 sürümünde ortaya çıkan bazı performans sorunlarını, gecikmeleri ve kaynak kullanımını azaltmak amacıyla 2015 yılında IETF tarafından standartlaştırılan modern bir ağ protokolüdür.

İyileştirdiği sorunlar ve getirdiği özellikler şunlardır:
* **Binary yapı:** HTTP/1.1 metin tabanlı bir protokol olarak çalışırken HTTP/2 iletişimi daha verimli hale getirmek için binary bir yapı kullanır. Bu yapı sayesinde verilerin işlenmesi daha verimli hale gelir ve protokoldeki gereksiz boşluk, satır sonu gibi karakterlerden kaynaklanan ek yük azaltılır.
* **Multiplexing (çoğullama):** HTTP/2, tek bir TCP bağlantısı üzerinden paralel olarak birden fazla isteğin ve yanıtın taşınmasına olanak sağlar. HTTP/1.1'de isteklerin işlenme şekli nedeniyle ortaya çıkabilen sıra başı engelleme (Head-of-Line Blocking) sorununu azaltır. Sıra başı engelleme, bir sıranın en başındaki isteğin veya paketin gecikmesi nedeniyle arkasında bekleyen diğer işlemlerin de beklemek zorunda kalmasıdır.
* **Header Compression (Başlık sıkıştırma):** HTTP isteklerinde tekrar tekrar gönderilen başlık bilgilerini sıkıştırarak veri miktarını azaltır. Bu sayede ağ üzerinden daha az veri taşınmasına ve iletişimin daha verimli hale gelmesine yardımcı olur.
* **Server Push:** HTTP/2, sunucunun istemcinin daha sonra ihtiyaç duyacağını tahmin ettiği bazı kaynakları, istemciden ayrıca istek gelmesini beklemeden gönderebilmesine olanak sağlar. Böylece bazı durumlarda ek isteklerin oluşturacağı gecikme azaltılabilir.

Bu özelliklerin yanında HTTP/2, HTTP/1.1'e göre daha verimli bir iletişim yapısı sunar. Ancak burada önemli bir nokta vardır: HTTP/2'nin kendisi yalnızca HTTPS üzerinden çalışmak zorunda değildir. Fakat tarayıcılar HTTP/2'yi pratikte büyük ölçüde güvenlik açısından HTTPS üzerinden kullanır. gRPC tarafında ise HTTP/2 kullanılması zorunludur.

## gRPC Haberleşme Yöntemleri

![gRPC](Images/gRPC.jpg)

Daha önce gRPC'nin bir streaming özelliği olduğundan bahsetmiştik. gRPC'nin streaming yapısı, HTTP/2'nin sağladığı sürekli veri akışı ve aynı bağlantı üzerinden birden fazla mesajın taşınabilmesi gibi özelliklerden yararlanır. gRPC'de temel olarak dört farklı iletişim türü bulunur:
* **Unary:** Client'ın server'a tek bir istek gönderdiği ve server'ın tek bir yanıt verdiği en basit RPC türüdür.

```text
İstemci          Sunucu        
    │── istek ─────▶│              
    │◀── cevap ─────│           
```
* **Server → Client Streaming:** Client server'a tek bir istek gönderir. Server ise bu isteğe karşılık birden fazla mesajı stream halinde client'a gönderir.
````text
 İstemci          Sunucu            
    │── istek ─────▶ │
    │◀── cevap 1 ─── │
    │◀── cevap 2 ─── │
    │◀── cevap N ─── │
````
* **Client → Server Streaming:** Client bir stream açarak server'a birden fazla mesaj gönderir. Client mesajlarını göndermeyi tamamladıktan sonra server tek bir yanıt gönderir.
````text
İstemci          Sunucu       
    │── istek 1 ───▶│                
    │── istek 2 ───▶│          
    │── istek N ───▶│           
    │◀── cevap ─────│
````
* **Bi-directional Streaming:** İstemci ile sunucu arasında sürekli bir iletişim akışı kurulur. İstemci bir veya birden fazla mesaj gönderirken server da buna karşılık bir veya birden fazla mesaj gönderebilir. İki tarafın stream'i birbirinden bağımsız çalışır; yani client mesaj gönderirken server'ın cevap vermesini beklemek zorunda değildir. Bu sayede iki taraf aynı bağlantı üzerinden karşılıklı olarak veri gönderebilir.
````text
 İstemci          Sunucu          
    │── istek 1 ───▶│
    │◀── cevap 1 ───│    
    │── istek 2 ───▶│
    │◀── cevap 2 ───│
````

### Deadline 

gRPC'nin bir diğer önemli özelliği Deadline kullanımıdır. Client, bir gRPC isteğinin tamamlanması için en fazla ne kadar süre beklemek istediğini belirleyebilir. Bunun temel nedeni, bir serviste meydana gelen gecikmenin veya kilitlenmenin diğer servisleri de etkileyerek tüm sistemi yavaşlatmasını ve domino etkisi (Cascading Failure) oluşturmasını engellemektir. Örneğin bir servis yanıt vermediğinde diğer servisler bu yanıtı beklemeye devam ederse gereksiz CPU ve bellek kullanımı oluşabilir ve zamanla diğer uygulamaların çalışması da etkilenebilir.

Bu nedenle client tarafından belirlenen en son tamamlanma süresine Deadline denir. Server da bir RPC'nin deadline'ının dolup dolmadığını veya işlemi tamamlamak için ne kadar süre kaldığını kontrol edebilir. Böylece uzun süre bekleyen isteklerin oluşturabileceği yığılma ve gereksiz kaynak tüketiminin önüne geçilebilir.

### Cancellation 

gRPC'nin bir diğer önemli özelliği ise RPC'lerin iptal edilebilmesidir. Client ve server arasındaki iletişim durumunu takip ederek ihtiyaç duyulmayan bir RPC isteği iptal edilebilir. RPC iptal edildiğinde, ilgili işlemin durdurulması sağlanır ve bu istek için gereksiz yere kaynak tüketilmesinin önüne geçilir.

Neden gönderdiğimiz bir isteğe ihtiyaç duymayalım ve onu iptal edelim derseniz, bunu bir örnekle açıklayabiliriz. Bir arama motorunda “çay” kelimesini aratmak istediğinizi düşünelim. Önce “ç” karakterini, sonra “a” ve son olarak “y” karakterini gireriz. Web siteleri, biz “ç” harfini girdiğimizde arka planda “ç” ile ilgili önerileri arar. Ardından “a” harfini girdiğimizde “ça”, son olarak “y” harfini girdiğimizde ise “çay” ile ilgili sonuçları arar.

Diyelim ki aradığımız sonucu bulduk ve ilgili web sitesine girdik. Bu sırada arka planda en az üç istek gönderilmiş olabilir. Ancak “çay” kelimesini yazdığımızda önceki “ç” ve “ça” isteklerine artık ihtiyaç kalmaz. Cancellation özelliği sayesinde bu istekler iptal edilebilir ve sunucunun gereksiz yere çalışmaya devam etmesinin önüne geçilebilir. Böylece sunucunun CPU ve RAM gibi kaynakları boş yere tüketmesi azaltılabilir.

![Cancellation Example](Images/Cancellation%20Example.jpg)

## gRPC Status Codes
![gRPC Status Codes](Images/gRPC%20Status%20Codes.jpg)

gRPC arka planda HTTP/2 kullansa da, HTTP seviyesindeki `200 OK` yanıtı sadece ağ taşıma katmanının (transport) başarılı olduğunu gösterir. Yani paket sunucuya sağ salim varmıştır. Fakat sunucu tarafındaki fonksiyonun içinde ne olduğu (kullanıcı bulunamadı mı, işlem zaman aşımına mı uğradı, yetki mi yok) HTTP katmanının konusu değildir. Bu nedenle gRPC, fonksiyon çağrılarına özel 0 ile 16 arasında standart durum kodları (Status Codes) tanımlamıştır:

- **0: OK** – İşlem başarılı.
- **1: CANCELLED** – İstemci isteği iptal etti.
- **2: UNKNOWN** – Bilinmeyen veya sunucuda yakalanmamış genel bir hata oluştu.
- **3: INVALID_ARGUMENT** – İstemci geçersiz bir parametre veya argüman gönderdi.
- **4: DEADLINE_EXCEEDED** – Belirlenen işlem süresi (deadline) aşıldı / zaman aşımı.
- **5: NOT_FOUND** – İstenen kaynak (kullanıcı, dosya vb.) bulunamadı.
- **6: ALREADY_EXISTS** – Oluşturulmak istenen kaynak zaten mevcut.
- **7: PERMISSION_DENIED** – İstemcinin bu işlemi yürütmek için yetkisi yok.
- **8: RESOURCE_EXHAUSTED** – Kota, bellek veya disk gibi sistem kaynakları tükendi.
- **9: FAILED_PRECONDITION** – Sistem durumu, işlemin yürütülmesi için uygun değil.
- **10: ABORTED** – Eşzamanlılık çakışması veya transaction iptali nedeniyle işlem durduruldu.
- **11: OUT_OF_RANGE** – Bir değer geçerli aralığın dışına çıktı.
- **12: UNIMPLEMENTED** – Çağrılan servis veya metot sunucuda henüz tanımlanmamış/desteklenmiyor.
- **13: INTERNAL** – Ciddi bir dahili sistem/sunucu hatası oluştu.
- **14: UNAVAILABLE** – Servis şu anda geçici olarak kapalı veya ulaşılamıyor.
- **15: DATA_LOSS** – Kurtarılamaz veri kaybı veya veri bozulması meydana geldi.
- **16: UNAUTHENTICATED** – İstek geçerli kimlik doğrulama bilgisi (token, sertifika vb.) içermiyor.

İstemci ile sunucu bu kodları, yanıtın en sonunda yer alan HTTP başlıkları (HTTP/2 Trailers) içindeki `grpc-status` alanı üzerinden değiş tokuş eder.

## gRPC'nin Avantajları
* **Yüksek Performanslı:** Binary serialization işlemi ve HTTP/2'nin hızını birleştirdiğimizde mesaj iletimi için oldukça hızlı ve performanslı bir iletişim altyapısı sağlanmış olur.
* **Otomatik Kod Üretimi:** gRPC, `.proto` dosyasında tanımlanan servisler ve mesajlar için otomatik olarak gerekli kodları oluşturabilir. Bu da haberleşme işlemleri için zaman kaybetmemizin önüne geçer. Çünkü REST mimarisi gibi yapılarda gerekli istekleri, sınıfları ve bazı iletişim yapılarını kendimiz oluştururken gRPC sayesinde temel iletişim yapısı otomatik olarak oluşturulur. Biz daha çok hangi servis ve metotların bulunması gerektiğini ve bu metotların iş akışını belirlemeye odaklanırız.
* **Spesifikasyonları vardır ve belirsizliği ortadan kaldırır:** `.proto` dosyasında servislerin hangi metotlara sahip olacağı, bu metotların hangi parametreleri alacağı ve hangi verileri döndüreceği önceden belirlenir. Böylece client ve server taraflarının aynı servis tanımına göre çalışması sağlanır.

## gRPC'nin Dezavantajları
* **Browser desteğinin sınırlı olması:** Tarayıcıların sunduğu standart web API'leri gRPC'nin HTTP/2 üzerindeki tüm özelliklerine doğrudan erişim sağlamaz. Bu nedenle gRPC, tarayıcılar üzerinden doğrudan yapılan isteklerde REST API'ler kadar kolay kullanılamaz. Web uygulamalarında browser ile gRPC arasında iletişim kurmak için genellikle gRPC-Web gibi ek çözümlere ihtiyaç duyulur. Bu nedenle dışarıdan gelen browser tabanlı çağrılarda REST API'ler hâlâ yaygın olarak tercih edilmektedir.
* **Okunabilirliğinin düşük olması:** HTTP API istekleri genellikle JSON gibi metin tabanlı formatlarda gönderildiği için insanlar tarafından kolayca okunabilir ve oluşturulabilir. gRPC mesajları ise varsayılan olarak Protobuf ile binary formatta kodlandığından, bir insan tarafından okunup anlaşılması HTTP API isteklerine göre daha zordur.
* **Entegrasyonunun daha karmaşık olması:** gRPC'nin `.proto` dosyaları, kod üretimi, Protobuf yapısı ve HTTP/2 gibi kendi çalışma yapıları bulunduğu için REST API'lere göre projeye ilk kez entegre edilmesi biraz daha fazla öğrenme ve yapılandırma gerektirebilir. Bu nedenle bazı durumlarda implementasyonu REST API'lere göre daha karmaşık olabilir.

## gRPC'nin Implementasyonu (Uygulaması)
Anlattığımız kavramların kod üzerinde nereye denk geldiğini görmek için basit bir uygulama yazalım. gRPC'nin dört iletişim tipini ASP.NET Core ile uygulayıp Postman üzerinden deneyeceğiz.
```text
GrpcDemo/
├── Protos/demo.proto          # sözleşme
├── Services/DemoService.cs    # sunucu mantığı
├── Program.cs                 # servis kaydı
└── GrpcDemo.csproj
```

Reflection paketini ekledik, böylece Postman `.proto` dosyasını elle yüklemeden metotları kendisi listeler:

```bash
dotnet add package Grpc.AspNetCore.Server.Reflection
```

### Sözleşme: `demo.proto`

```proto
syntax = "proto3";
option csharp_namespace = "GrpcDemo";

package demo;

// protoc bu tanımdan Demo.DemoBase sınıfını üretir (sunucu altyapısı)
service Demo {
  rpc Unary (UserRequest) returns (UserResponse);                      // Unary: 1 istek -> 1 cevap
  rpc ServerStream (UserRequest) returns (stream UserResponse);        // Server Streaming: 1 istek -> N cevap
  rpc ClientStream (stream NumberRequest) returns (SumResponse);       // Client Streaming: N istek -> 1 cevap
  rpc BidiStream (stream ChatMessage) returns (stream ChatMessage);    // Bi-directional Streaming: N istek <-> N cevap
}

// Alan numaraları (= 1, = 2): Protobuf binary formatta alan adı yerine bu numaraları kullanır
message UserRequest   { string name = 1; int32 count = 2; }
message UserResponse  { string message = 1; }
message NumberRequest { int32 value = 1; }
message SumResponse   { int32 total = 1; }
message ChatMessage   { string user = 1; string text = 2; }
```

Dört tipi ayıran tek şey `stream` kelimesinin yeridir. `.proto` dosyasının dilden bağımsız tanım (IDL) olması da burada görülüyor, içinde C# ya da başka bir programlama dili yok.


### Sunucu: `DemoService.cs`

```csharp
using Grpc.Core;
namespace GrpcDemo;

// Demo.DemoBase: protoc'un .proto dosyasından ürettiği sınıf.
// Biz sadece iş mantığını (implementasyonu) yazıyoruz.
public class DemoService : Demo.DemoBase
{
    // UNARY: en basit RPC türü, tek istek tek cevap
    // ServerCallContext: meta veriler, deadline ve cancellation bilgisi burada taşınır (bu örnekte kullanmadık)
    public override Task<UserResponse> Unary(UserRequest request, ServerCallContext context)
    {
        var response = new UserResponse { Message = $"Merhaba {request.Name}" };
        return Task.FromResult(response);
    }


    // SERVER STREAMING: 1 istek -> N cevap
    // responseStream: sunucunun istemciye mesaj yazdığı kanal
    public override async Task ServerStream(UserRequest request, IServerStreamWriter<UserResponse> responseStream, ServerCallContext context)
    {
        for(int k = 1; k <= request.Count; k++){
            await responseStream.WriteAsync(
                new UserResponse {Message = $"{request.Name} - mesaj {k}" });

            await Task.Delay(500);   // akışı gözle görebilmek için
        }
    }


    // CLIENT STREAMING: N istek -> 1 cevap
    // requestStream: istemci mesaj gönderdikçe buraya düşer, End Streaming ile döngü biter
    public override async Task<SumResponse> ClientStream(IAsyncStreamReader<NumberRequest> requestStream, ServerCallContext context)
    {
        int total = 0;
        await foreach (var _number in requestStream.ReadAllAsync()){
            total += _number.Value;
        }

        return new SumResponse{Total= total};
    }


    // BIDIRECTIONAL: N istek <-> N cevap
    // Okuma (requestStream) ve yazma (responseStream) aynı anda açık, iki taraf birbirini beklemeden mesaj gönderebilir
    public override async Task BidiStream(IAsyncStreamReader<ChatMessage> requestStream, IServerStreamWriter<ChatMessage> responseStream, ServerCallContext context)
    {
        await foreach ( var _msg in requestStream.ReadAllAsync()){
            await responseStream.WriteAsync(
                new ChatMessage {User = "Sunucu", Text = $"Echo: {_msg.Text}"}
            );
        }
    }
}
```

### Servis Kaydı: `Program.cs`

```csharp
using GrpcDemo;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddGrpc();              // gRPC altyapısı (HTTP/2 üzerinde çalışır)
builder.Services.AddGrpcReflection();    // Postman'in metotları keşfetmesi için

var app = builder.Build();

app.MapGrpcService<DemoService>();       // gelen gRPC çağrılarını DemoService'e yönlendirir
app.MapGrpcReflectionService();          // reflection uç noktası

app.Run();
```

### Çalıştırma ve Postman ile Test
```bash
dotnet run --urls http://localhost:5001
```
![Postman gRPC istek oluşturma](Images/Postman%20gRPC%20istek%20oluşturma.jpg)

gRPC'yi tarayıcı doğrudan konuşamadığı için adresi tarayıcıdan açıp deneyemeyiz. Postman'de yeni bir gRPC isteği açıp adrese `localhost:5001` yazıyoruz (TLS kapalı) ve **Use server reflection** seçiyoruz. Dört metot otomatik listelenir.

![grpc method types](Images/grpc-method-types.png)

> Postman'in web sürümünde gRPC isteklerinin `localhost`'a ulaşabilmesi için **Postman Desktop Agent**'ın çalışıyor olması gerekir.


| Metot | Mesaj | Sonuç |
|---|---|---|
| `Unary` | `{"name":"Betul","count":0}` → **Invoke** | `Merhaba Betul` |
| `ServerStream` | `{"name":"Betul","count":5}` → **Invoke** | 5 cevap, yarım saniye arayla |
| `ClientStream` | **Invoke**, sonra `{"value":5}` ve `{"value":10}` için **Send**, en son **End Streaming** | `{"total":15}` |
| `BidiStream` | **Invoke**, sonra `{"user":"Betul","text":"merhaba"}` için **Send** | Her mesaja anında `Echo: merhaba` |

**Unary**

![Unary](Images/Unary.png)

**ServerStream**

![ServerStream](Images/ServerStream.png)

**ClientStream**

![ClientStream](Images/ClientStream.png)

**BidiStream**

![BidiStream](Images/BidiStream.png)

---

## gRPC ve REST Karşılaştırması

Önceki yazılarda [REST mimarisinden](https://github.com/BetulBilecen/TUG/blob/main/REST-API-ve-HTTP.md) bahsetmiştik. Şimdi ağda veri iletişimi için kullanılan bu iki mimariyi karşılaştıralım.

![gRPC vs REST](Images/gRPC%20vs%20REST.jpg)

Kısaca hem gRPC hem de REST, API tasarımında yaygın olarak kullanılan mimari stillerdir. Her ikisi de istemci/sunucu mimarisini takip eder, HTTP tabanlı iletişime dayanır ve programlama dillerinden bağımsızdır.

REST ve gRPC mimarilerinin temel farklarını şöyle sıralayabiliriz:

- **Veri Formatı:** REST API'leri JSON ve XML gibi düz metin formatlarını kullanır. gRPC ise verileri ikili (binary) formata dönüştürüp kodlamak için Protobuf kullanır. Binary format sayesinde karakter çözümlemeyle uğraşmadan, Server Stub gelen veriyi çok daha hızlı bir şekilde decode eder (anlaşılır nesnelere geri dönüştürür).
- **İletişim Modeli:** gRPC; Unary, Server Streaming, Client Streaming ve Bi-directional Streaming olmak üzere 4 farklı iletişim modelini destekler. REST mimarisi ise temelde tek yönlü istek-yanıt (Unary) mekanizmasını benimser.
- **Kod Üretimi:** gRPC yerleşik kod üretimi sunar; `.proto` sözleşmesinden hem istemci hem sunucu iskelet kodları otomatik üretilir. REST'te bu özellik varsayılan olarak yoktur; ancak bu ihtiyacı karşılamak için harici araçlar (OpenAPI Generator, Swagger Codegen vb.) kullanılır.
- **Tasarım Modeli:** gRPC, işlemlerin servis ve fonksiyon olarak tanımlandığı servis/eylem odaklı bir yapıya sahiptir. REST'te ise tasarım; URL'ler ile tanımlanan kaynaklar (resources) ve bunlara uygulanan standart HTTP metotları (GET, POST vb.) etrafında şekillenir.
- **Bağlaşım (Coupling):** gRPC, ortak `.proto` sözleşmesine dayandığı için istemci ve sunucu arasında sıkı sıkıya bağlıdır (tight coupling); köklü şema değişikliklerinde iki tarafın da güncellenmesini gerektirir ancak derleme anında (compile-time) güçlü tip güvenliği sunar. REST ise gevşek bağlıdır (loose coupling); iletişim esnek JSON formatı ve evrensel HTTP fiilleri üzerinden yürür. Sunucunun yanıtına yeni bir alan eklenmesi veya iç mantığının değişmesi istemciyi etkilemez; istemci sadece ilgilendiği veriyi okumaya devam eder. Bu sayede ekipler birbirinin kodunu beklemeden büyük ölçüde bağımsız geliştirme yapabilir.
- **Protokol ve Tarayıcı Desteği:** gRPC varsayılan olarak HTTP/2 kullanırken, REST API'leri genellikle HTTP/1.1 (veya isteğe bağlı HTTP/2) üzerinden çalışır. Modern tarayıcılar HTTP/2'yi desteklese de, tarayıcı API'leri (`fetch`, `XHR`) gRPC'nin ihtiyaç duyduğu alt düzey HTTP/2 özelliklerine doğrudan erişim izni vermez. Bu nedenle gRPC, web tarayıcılarında doğrudan çalışmak için ek bir köprüye (`gRPC-Web` ve Envoy proxy) ihtiyaç duyar; bu durum gRPC'yi web ön yüzleri yerine doğrudan arka uç (backend-to-backend) mikroservis iletişimi için çok daha cazip kılar.
- **Cancellation:** Klasik REST/HTTP mimarisinde client bir istek attıktan sonra tarayıcıyı kapatsa veya internet bağlantısı kopsa bile server, bu durumdan haberdar olmayarak işlemi çalıştırmaya devam edebilir. gRPC'deki Cancellation özelliği ise devam eden bir RPC isteğinin iptal edilmesini sağlar. Böylece artık ihtiyaç duyulmayan işlemlerin durdurulmasına ve gereksiz kaynak tüketiminin önlenmesine yardımcı olur.

### Peki Madem gRPC Bu Kadar Güçlü, Neden Hâlâ REST Kullanıyoruz?

Madem performans ve hız tarafında bu kadar belirgin farklar var, neden tüm sistemleri gRPC ile kurmuyoruz? Çünkü yazılım mimarisinde "en iyi" teknoloji yoktur; **ihtiyaca en uygun araç** vardır. Her iki mimari de farklı kullanım senaryolarında öne çıkar.

**REST**, günümüzde web servisleri ve genel API entegrasyonları için hâlâ en popüler mimaridir. İnsan tarafından kolayca okunabilen JSON yapısı, HTTP standartlarına doğrudan oturması, esnekliği ve öğrenme eğrisinin düşüklüğü sayesinde hem yeni başlayanlar hem de büyük ekipler için geliştirmesi son derece kolaydır.

**REST özellikle şu senaryolar için idealdir:**
- **Web Ön Yüzleri ve Tarayıcılar:** Ekstra bir vekil sunucuya (proxy) ihtiyaç duymadan doğrudan tarayıcıdan (`fetch`/`axios`) çağrılabilen sistemler.
- **Halka Açık (Public) API'ler:** Dış dünyadaki üçüncü taraf geliştiricilerin kolayca anlayıp test edebileceği, entegrasyonu zahmetsiz genel servisler.
- **Standart CRUD İşlemleri:** Karmaşık veri akışlarına ihtiyaç duymayan, basit veri alışverişi gerektiren uygulamalar.


---

**gRPC** ise özellikle dağıtık sistemler, mikroservis mimarileri ve yüksek veri hacmine sahip iç ağlar için tasarlanmış bir güç merkezidir. Ağ gecikmesinin (latency) kritik olduğu, farklı dillerin bir arada çalıştığı ve sistemlerin birbiriyle kesintisiz konuştuğu ortamlarda fark yaratır.

**gRPC özellikle şu senaryolar için idealdir:**
- **İç Mikroservis İletişimi (Backend-to-Backend):** Şirket içi servislerin birbirleriyle minimum kaynak ve maksimum hızla haberleşmesi gereken altyapılar.
- **Gerçek Zamanlı Veri ve Akış (Streaming):** Finansal veriler, sohbet sistemleri, IoT/sensör verileri gibi tek veya çift yönlü sürekli veri akışı gerektiren uygulamalar.
- **Yüksek Performans ve Düşük Gecikme:** Her milisaniyenin ve bayt boyutunun maliyet yarattığı büyük ölçekli dağıtık mimariler.
- **Çok Dilli (Polyglot) Ekipler:** Go, Python, C#, Java gibi farklı dillerle yazılmış servislerin ortak bir sözleşme (`.proto`) üzerinden sorunsuz entegre olması gereken projeler.

---

## Kaynaklar
- https://gokhana.medium.com/grpc-nedir-ve-nas%C4%B1l-uygulan%C4%B1r-microservice-mimarisi-ile-grpc-9f1dc0847475
- https://www.imperva.com/learn/performance/http2/
- https://www.ibm.com/think/topics/grpc
- https://aws.amazon.com/tr/compare/the-difference-between-grpc-and-rest/
