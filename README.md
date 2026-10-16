# .NET Backend Geliştirme – Temel Bilgi ve Kavramlar

> **Amaç:** Bu dokümanda modern yazılım geliştirme süreçleri, .NET ekosistemi, backend geliştirme, ASP.NET Core, veritabanı işlemleri, güvenlik, loglama ve yazılım tasarım prensipleri temel örneklerle açıklanmıştır. Örnekler öğrenme amacıyla hazırlanmıştır; gerçek projelerde kullanılan sürüm ve yapılandırmalar projeye göre değişebilir.

## İçindekiler

1. [Modern Yazılım Geliştirme Pratikleri](#1-modern-yazılım-geliştirme-pratikleri)
2. [.NET Ekosistemi](#2-net-ekosistemi)
3. [Backend Geliştirme Temelleri](#3-backend-geliştirme-temelleri)
4. [ASP.NET Core](#4-aspnet-core)
5. [Veritabanı ve ORM](#5-veritabanı-ve-orm)
6. [Güvenlik ve Performans](#6-güvenlik-ve-performans)
7. [Logging ve Hata Yönetimi](#7-logging-ve-hata-yönetimi)
8. [Yazılım Geliştirme Prensipleri](#8-yazılım-geliştirme-prensipleri)

---

# 1. Modern Yazılım Geliştirme Pratikleri

## 1.1 Git ve GitHub nedir?

**Git**, kaynak koddaki değişiklikleri zaman içinde takip etmeyi sağlayan dağıtık bir sürüm kontrol sistemidir. Geliştirici eski sürümlere dönebilir, farklı özellikleri ayrı dallarda geliştirebilir ve değişiklikleri karşılaştırabilir.

**GitHub**, Git depolarının internet üzerinde barındırılmasını ve ekipçe kullanılmasını sağlayan bir platformdur. Pull request, kod inceleme, issue takibi ve otomasyon gibi özellikler sunar. Kısacası Git bir araç, GitHub ise Git depolarıyla çalışmayı kolaylaştıran çevrim içi bir hizmettir.

### Temel Git komutları

| Komut | Açıklama | Örnek |
|---|---|---|
| `git init` | Bulunulan klasörde yeni Git deposu oluşturur. | `git init` |
| `git clone URL` | Uzak depoyu bilgisayara kopyalar. | `git clone https://github.com/kullanici/proje.git` |
| `git add` | Değişiklikleri commit için hazırlama alanına ekler. | `git add README.md` |
| `git commit` | Hazırlanan değişiklikleri açıklama mesajıyla kaydeder. | `git commit -m "README eklendi"` |
| `git push` | Yerel commit'leri uzak depoya gönderir. | `git push origin main` |
| `git pull` | Uzak depodaki değişiklikleri alır ve mevcut dala entegre eder. | `git pull origin main` |
| `git branch` | Dalları listeler veya yeni dal oluşturur. | `git branch feature/login` |
| `git merge` | Bir daldaki değişiklikleri mevcut dala birleştirir. | `git merge feature/login` |

Örnek çalışma akışı:

```bash
git clone https://github.com/kullanici/trex_research.git
cd trex_research
# README.md dosyası oluşturulduktan veya düzenlendikten sonra:
git add README.md
git commit -m "Backend araştırma raporu eklendi"
git push origin main
```

`git add` değişikliği henüz depoya kalıcı olarak kaydetmez; yalnızca bir sonraki commit'e dahil edilmek üzere hazırlar. `git commit` yerel depoya kayıt yapar, `git push` ise bu kayıtları uzak depoya gönderir.

## 1.2 Merge conflict nedir?

Merge conflict, Git'in iki değişikliği otomatik olarak birleştiremediği durumda ortaya çıkar. Örneğin iki kişi aynı dosyanın aynı satırlarını farklı şekilde değiştirmiş olabilir.

Git çakışan bölümü genellikle şu işaretlerle gösterir:

```text
<<<<<<< HEAD
Mevcut daldaki metin
=======
Diğer daldaki metin
>>>>>>> feature/dal
```

Çözüm adımları:

1. Çakışan dosyayı açıp iki değişikliği incelemek.
2. Doğru son içeriği belirlemek ve Git'in işaretlerini kaldırmak.
3. Dosyayı test etmek.
4. `git add dosya-adi` ve `git commit` ile çözümü kaydetmek.

Çakışmayı körü körüne bir tarafı seçerek çözmek yerine, iki değişikliğin amacını anlamak gerekir.

## 1.3 CI/CD nedir?

**CI (Continuous Integration / Sürekli Entegrasyon)**, geliştiricilerin değişikliklerinin ortak depoya sık sık eklenmesi ve otomatik derleme/test süreçlerinden geçirilmesidir. Hataları erken fark etmeye yardımcı olur.

**CD (Continuous Delivery/Deployment)**, test edilmiş yazılımın yayıma hazır hâle getirilmesini veya uygun koşullarda otomatik olarak ortama dağıtılmasını ifade eder. Continuous Delivery'de üretime geçiş için onay adımı bulunabilir; Continuous Deployment'da başarılı kontrollerden sonra dağıtım otomatik yapılabilir.

.NET projesinde örnek akış:

1. Geliştirici kodu GitHub'a gönderir.
2. Pipeline .NET SDK'yı kurar.
3. `dotnet restore` bağımlılıkları indirir.
4. `dotnet build` projeyi derler.
5. `dotnet test` testleri çalıştırır.
6. Başarılıysa paket veya dağıtım çıktısı hazırlanabilir.

### GitHub Actions için örnek pipeline

Aşağıdaki örnek, depo kökünde `.github/workflows/dotnet.yml` dosyası olarak kullanılabilir. `YOUR_SOLUTION.sln` kısmı gerçek çözüm dosyasının adıyla değiştirilmelidir.

```yaml
name: .NET CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Kodu al
        uses: actions/checkout@v4

      - name: .NET SDK kur
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'

      - name: Bağımlılıkları yükle
        run: dotnet restore YOUR_SOLUTION.sln

      - name: Derle
        run: dotnet build YOUR_SOLUTION.sln --no-restore --configuration Release

      - name: Testleri çalıştır
        run: dotnet test YOUR_SOLUTION.sln --no-build --configuration Release
```

Azure DevOps Pipelines da benzer şekilde YAML üzerinden restore, build, test ve deployment adımlarını çalıştırabilir. Hangi platformun seçileceği ekipteki araçlara, izinlere ve dağıtım ortamına bağlıdır.

## 1.4 SDLC: Yazılım geliştirme yaşam döngüsü

SDLC, bir yazılımın fikir aşamasından bakımına kadar izlenen süreçtir.

| Aşama | Ne yapılır? | Yazılımcının rolü |
|---|---|---|
| Planlama | İhtiyaç, kapsam, zaman ve risk belirlenir. | Teknik yapılabilirlik ve tahmin sunar. |
| Analiz | Kullanıcı gereksinimleri ve iş kuralları netleştirilir. | Gereksinimleri teknik çözümlere dönüştürmeye yardım eder. |
| Tasarım | Mimari, veri modeli ve API tasarlanır. | Sınıfları, katmanları ve arayüzleri planlar. |
| Geliştirme | Kod yazılır. | Okunabilir ve test edilebilir kod üretir. |
| Test | Hatalar ve gereksinim uyumu kontrol edilir. | Birim, entegrasyon ve diğer testleri yazar/çalıştırır. |
| Dağıtım | Uygulama hedef ortama alınır. | Pipeline, yapılandırma ve sürüm süreçlerine destek verir. |
| Bakım | Hatalar giderilir, performans ve özellikler geliştirilir. | Logları inceler, düzeltme ve iyileştirme yapar. |

Bu aşamalar her zaman katı bir sırayla ve yalnızca bir kez gerçekleşmez. Özellikle çevik ekiplerde geri bildirim geldikçe analiz, tasarım ve geliştirme tekrar ele alınabilir.

## 1.5 Agile, Scrum ve Kanban

- **Agile (Çevik yaklaşım):** Değişen ihtiyaçlara uyum sağlamayı, sık geri bildirim almayı ve çalışan yazılımı küçük adımlarla sunmayı önemser.
- **Scrum:** İşlerin sprint adı verilen belirli süreli döngülerde planlandığı bir çerçevedir. Product Backlog, Sprint Planning, Daily Scrum, Sprint Review ve Retrospective gibi etkinlikleri bulunur.
- **Kanban:** İş akışını görselleştirir; örneğin “Yapılacak”, “Devam Ediyor” ve “Tamamlandı” sütunları kullanılır. Devam eden iş miktarını sınırlamak ve darboğazları görmek hedeflenir.

**Kısa yorum:** Küçük bir ekipte Kanban iş akışını görünür kılmak için pratik olabilir. Sprint hedefleri ve düzenli planlama isteyen ekipler Scrum'ı tercih edebilir. Her iki yaklaşım da ekibin ihtiyacına göre uyarlanmalıdır.

---

# 2. .NET Ekosistemi

## 2.1 .NET nedir?

.NET, Microsoft tarafından geliştirilen açık kaynaklı bir geliştirme platformudur. C# gibi dillerle web uygulamaları, API'ler, masaüstü uygulamaları, servisler ve çeşitli başka yazılımlar geliştirmek için kullanılır. Ortak çalışma zamanı, temel kütüphaneler, SDK ve araçlar sunar.

Tarihsel olarak Windows odaklı **.NET Framework** ortaya çıktı. Daha sonra farklı işletim sistemlerinde çalışmayı hedefleyen **.NET Core** geliştirildi. .NET Core 3.1 sonrasında ürün adı sadeleşerek .NET 5 ve devam eden sürümlerde **.NET** olarak kullanıldı.

## 2.2 .NET Framework, .NET Core ve modern .NET karşılaştırması

| Özellik | .NET Framework | .NET Core | Modern .NET (5 ve sonrası) |
|---|---|---|---|
| Temel hedef | Özellikle Windows uygulamaları | Platformlar arası uygulamalar | Platformlar arası, sürekli geliştirilen platform |
| İşletim sistemi | Windows | Windows, Linux, macOS | Windows, Linux, macOS |
| Kullanım | Mevcut/legacy uygulamalar, bazı Windows teknolojileri | Modern çapraz platform uygulamalarına geçiş dönemi | Yeni uygulamalar için genel tercih |
| Gelişim durumu | Bakım ve uyumluluk odaklı | .NET 5 ile devam eden çizgiye dönüştü | Düzenli sürümlerle devam ediyor |

.NET 7, .NET 8 gibi sürümlerin destek süreleri aynı değildir. Projeye başlanırken Microsoft'un güncel destek takvimine bakmak gerekir. Yeni bir proje için desteklenen bir LTS sürümü seçmek genellikle daha uzun bakım süresi sağlar.

## 2.3 Platformlar arası çalışma

.NET uygulamaları uygun SDK ve çalışma zamanı bulunduğunda Windows, Linux ve macOS üzerinde çalışabilir. Ancak uygulamanın kendisi işletim sistemine özel API'ler, dosya yolları veya bağımlılıklar kullanıyorsa ek düzenleme gerekebilir.

### `dotnet --info` çıktısını anlama

Terminalde şu komut çalıştırılır:

```bash
dotnet --info
```

Örnek olarak çıktı; SDK sürümünü, işletim sistemi ve mimari bilgisini, kurulu SDK'ları ve çalışma zamanlarını gösterebilir. Aşağıdaki değerler yalnızca örnektir, gerçek bilgisayar çıktısı değildir:

```text
.NET SDK:
 Version:   8.0.xxx
 OS Name:   Windows
 OS Version: 10.0.xxxxx
 Architecture: x64

Host:
 Version: 8.0.x
 Architecture: x64
```

- **SDK:** Uygulamayı oluşturmak, derlemek ve test etmek için gereken araçlar.
- **Runtime:** Derlenmiş .NET uygulamasını çalıştırmak için gereken bileşenler.
- **Architecture:** İşletim sisteminin/çalışma ortamının mimarisi; örneğin x64 veya arm64.

## 2.4 Senkron ve asenkron programlama

**Senkron işlemde** bir işlem bitmeden sonraki adım genellikle ilerlemez. **Asenkron programlama**, özellikle dosya, ağ ve veritabanı gibi bekleme içeren işlemlerde iş parçacığının beklerken başka işlere ayrılabilmesini kolaylaştırır.

```csharp
// Senkron örnek
string text = File.ReadAllText("data.txt");

// Asenkron örnek
string textAsync = await File.ReadAllTextAsync("data.txt");
```

`async`, bir metodu asenkron işlemler kullanabilecek şekilde işaretler. `await`, tamamlanması beklenen asenkron işlemi beklerken çağıran akışın uygun şekilde devam edebilmesini sağlar. `Task` veya `Task<T>`, devam eden bir asenkron işlemi temsil eder.

```csharp
public async Task<string> GetMessageAsync()
{
    await Task.Delay(500);
    return "İşlem tamamlandı";
}
```

`ConfigureAwait(false)`, özellikle yeniden kullanılabilir kütüphane kodlarında devamın önceki senkronizasyon bağlamına dönmesini istemediğimiz durumlarda kullanılabilir. ASP.NET Core'da klasik ASP.NET'teki gibi bir `SynchronizationContext` varsayımı genellikle yoktur; bu nedenle her yerde eklemek zorunlu değildir.

### `=>` ifadesi: lambda ve expression-bodied üyeler

C#'ta `=>` bir lambda ifadesinde veya kısa gövdeli bir üyede kullanılır. JavaScript'teki arrow function ile görünüş olarak benzese de C# dilinin kendi kuralları vardır.

```csharp
Func<int, int> kare = sayi => sayi * sayi;

public int Topla(int a, int b) => a + b;
```

İlk örnek bir lambda ifadesidir; ikinci örnek metodu kısa biçimde tanımlar.

**Ne zaman asenkron?** Bir API'nin veritabanından kayıt okuması veya başka bir servise HTTP isteği atması sırasında asenkron metotlar web sunucusunun kaynaklarını daha verimli kullanmasına yardımcı olabilir. CPU ağırlıklı bir hesaplamayı sadece `async` yapmak ise otomatik olarak hızlandırmaz.

---

# 3. Backend Geliştirme Temelleri

## 3.1 Backend ve frontend

**Frontend**, kullanıcının doğrudan etkileşimde bulunduğu arayüz bölümüdür. **Backend**, iş kurallarını uygulayan, verileri işleyen, kimlik ve yetki kontrollerini yapan ve çoğunlukla veritabanı ya da başka servislerle iletişim kuran bölümdür.

Örneğin bir e-ticaret sitesinde ürünleri ekranda göstermek frontend'in; siparişin fiyatını doğrulamak, stok kontrolü yapmak ve siparişi kaydetmek backend'in sorumluluğundadır. Güvenlik açısından kritik kurallar yalnızca frontend'e bırakılmamalıdır.

## 3.2 Web sunucusu ve API

**Web sunucusu**, istemciden gelen HTTP isteklerini karşılayıp yanıt döndürür. **API (Application Programming Interface)**, yazılım bileşenlerinin belirli kurallarla iletişim kurmasını sağlayan arayüzdür. Web API'leri genellikle HTTP üzerinden veri sunar.

Yaygın API yaklaşımları:
- REST: Kaynakları URL'lerle temsil eder ve HTTP yöntemlerinden yararlanır.
- SOAP: XML tabanlı mesajlaşma ve tanımlı sözleşmeler kullanan bir protokoldür.
- GraphQL: İstemcinin ihtiyaç duyduğu alanları sorgulayabildiği bir API sorgulama dilidir.

## 3.3 HTTP ve temel metotlar

HTTP, istemci ve sunucu arasında istek/yanıt biçiminde iletişim kurulmasını sağlayan protokoldür.

| Metot | Genel amaç | Örnek |
|---|---|---|
| `GET` | Kaynak okumak | `GET /api/products/5` |
| `POST` | Yeni kaynak oluşturmak veya işlem başlatmak | `POST /api/products` |
| `PUT` | Kaynağı genellikle bütünüyle güncellemek | `PUT /api/products/5` |
| `DELETE` | Kaynağı silmek | `DELETE /api/products/5` |

Gerçek API tasarımında HTTP durum kodları da önemlidir. Örneğin `200 OK` başarılı yanıtı, `201 Created` oluşturulan kaynağı, `400 Bad Request` hatalı isteği, `401 Unauthorized` kimlik doğrulama eksikliğini, `403 Forbidden` ise kullanıcının işlem için yetkili olmadığını belirtebilir.

## 3.4 RESTful servislerin mantığı

REST, ağ üzerinden kaynaklarla çalışmak için kullanılan mimari kısıtlar kümesidir. RESTful bir API'de kaynaklar çoğunlukla URL'lerle temsil edilir; HTTP metotları işlemin amacını belirtir ve yanıtlar çoğunlukla JSON biçimindedir.

Örnek uç noktalar:

```text
GET    /api/products       # Ürünleri listele
GET    /api/products/12    # 12 numaralı ürünü getir
POST   /api/products       # Yeni ürün oluştur
PUT    /api/products/12    # Ürünü güncelle
DELETE /api/products/12    # Ürünü sil
```

REST yalnızca URL'leri bu şekilde adlandırmak değildir; istemci-sunucu ayrımı, durum bilgisiz istekler ve standart arayüz gibi ilkeleri de vardır.

## 3.5 JSON nedir?

JSON, verileri anahtar-değer çiftleri ve dizilerle ifade eden, sistemler arasında yaygın kullanılan bir metin formatıdır.

```json
{
  "id": 12,
  "name": "Klavye",
  "price": 749.90,
  "inStock": true,
  "tags": ["aksesuar", "bilgisayar"]
}
```

Burada `id` sayı, `name` metin, `price` sayı, `inStock` mantıksal değer ve `tags` metin dizisidir. JSON, dil bağımsız olması ve okunabilirliği nedeniyle web API'lerinde sık tercih edilir. JSON bir veritabanı ya da protokol değil, veri gösterim formatıdır.

## 3.6 REST, SOAP ve GraphQL karşılaştırması

| Özellik | REST | SOAP | GraphQL |
|---|---|---|---|
| Tür | Mimari yaklaşım | Protokol | Sorgulama dili ve çalışma yaklaşımı |
| Veri biçimi | Çoğunlukla JSON; başka biçimler de olabilir | XML | Genellikle JSON yanıtı |
| Veri isteme | Kaynak URL'leri ve HTTP metotları | Tanımlı mesajlar/operasyonlar | Sorguda istenen alanlar belirtilir |
| Güçlü yön | Basit ve web ile uyumlu | Sözleşme ve kurumsal standartlar | İstemcinin ihtiyacı olan alanları seçebilmesi |
| Dikkat edilmesi gereken | Uç nokta ve yanıt tasarımı | Daha ayrıntılı mesaj yapısı | Sorgu karmaşıklığı ve kaynak tüketimi yönetilmelidir |

Her yaklaşım her durumda en iyi değildir. Karar; mevcut sistemler, istemci ihtiyaçları, güvenlik, performans ve ekip deneyimi göz önüne alınarak verilmelidir.

---

# 4. ASP.NET Core

## 4.1 ASP.NET ve ASP.NET Core

ASP.NET, Microsoft ekosisteminde web uygulamaları geliştirmek için kullanılan teknolojiler ailesidir. ASP.NET Core, modern .NET üzerinde çalışan, açık kaynaklı ve platformlar arası web geliştirme çatısıdır. Web API, MVC uygulamaları ve gerçek zamanlı uygulamalar geliştirmek için kullanılabilir.

## 4.2 MVC nedir?

MVC, uygulamayı üç temel sorumluluğa ayıran bir tasarım desenidir:

- **Model:** Veriyi ve ilgili iş kurallarını temsil eder.
- **View:** Kullanıcıya gösterilen arayüzdür.
- **Controller:** İstekleri karşılar, gerekli işlemleri koordine eder ve uygun yanıtı döndürür.

Bu ayrım kodun daha düzenli olmasını sağlar. Ancak JSON döndüren bir Web API projesinde klasik View katmanı kullanılmayabilir.

## 4.3 Middleware nedir?

Middleware, ASP.NET Core'un HTTP istek ve yanıt hattında çalışan bileşenlerdir. Her middleware isteği inceleyebilir, sonraki bileşene aktarabilir veya yanıtı kendisi oluşturabilir. Sıralama önemlidir.

Örnek `Program.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

app.Run();
```

Bu kod örneğinde `UseAuthentication` kullanılıyorsa ilgili kimlik doğrulama servisi de yapılandırılmalıdır; örneğin projeye uygun `AddAuthentication(...)` ayarı yapılır. Sıralama, kullanılan özelliklere göre düzenlenmelidir. Kimlik doğrulama, yetkilendirmeden önce çalışmalıdır.

- `UseHttpsRedirection`: HTTP isteklerini HTTPS'e yönlendirmeye yardımcı olur.
- `UseRouting`: İsteğe uygun endpoint'i belirler.
- `UseAuthentication`: Kullanıcının kimliğini doğrulamaya çalışır.
- `UseAuthorization`: Kullanıcının ilgili kaynağa erişim yetkisini kontrol eder.
- `MapControllers`: Controller endpoint'lerini eşler.

## 4.4 Dependency Injection (DI)

Dependency Injection, bir sınıfın ihtiyaç duyduğu bağımlılıkları kendisinin oluşturmak yerine dışarıdan almasıdır. ASP.NET Core yerleşik bir DI konteyneri sunar. Bu yaklaşım sınıfların birbirine sıkı bağlı olmasını azaltır ve test etmeyi kolaylaştırır.

```csharp
public interface IMessageService
{
    string GetMessage();
}

public class MessageService : IMessageService
{
    public string GetMessage() => "Merhaba";
}

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddScoped<IMessageService, MessageService>();
```

Bir controller içinde:

```csharp
[ApiController]
[Route("api/[controller]")]
public class MessageController : ControllerBase
{
    private readonly IMessageService _messageService;

    public MessageController(IMessageService messageService)
    {
        _messageService = messageService;
    }

    [HttpGet]
    public IActionResult Get()
    {
        return Ok(_messageService.GetMessage());
    }
}
```

DI yaşam süreleri:
- **Transient:** Her istendiğinde yeni örnek oluşturulur.
- **Scoped:** Genellikle her HTTP isteği için bir örnek kullanılır.
- **Singleton:** Uygulama yaşam döngüsü boyunca aynı örnek kullanılır.

Singleton servisler eşzamanlı erişime uygun tasarlanmalıdır ve scoped bağımlılıkları doğrudan içine almamalıdır.

## 4.5 Katmanlı mimari, Service ve Repository

Katmanlı mimari, sorumlulukları ayrı bölümlere ayırır:

```text
İstemci
   |
   v
Presentation / API
   |
   v
Business / Service
   |
   v
Data Access / Repository
   |
   v
Veritabanı
```

- **Presentation/API:** HTTP isteklerini alır ve yanıtları döndürür.
- **Business/Service:** İş kurallarını uygular.
- **Data Access:** Veritabanı işlemlerini yürütür.
- **Repository pattern:** Veri erişim ayrıntılarını bir arayüz arkasına almayı amaçlar.
- **Service pattern:** İş akışını ve iş kurallarını ayrı bir sınıfta toplar.

Her küçük uygulamada ayrı bir repository katmanı şart değildir. Entity Framework Core zaten birçok veri erişim işlemini sağlayabildiği için ek soyutlama ancak gerçek bir fayda sağlıyorsa kullanılmalıdır.

## 4.6 Clean Architecture

Clean Architecture'da temel iş kuralları, dış sistemlerden olabildiğince bağımsız tutulur. Yaygın bir düzen:

```text
API / Presentation
       |
       v
Application  ---> Domain
       ^
       |
Infrastructure
```

Daha doğru bağımlılık kuralı şudur: dış katmanlar iç katmanlara bağımlı olabilir; Domain katmanı veritabanı, web çatısı veya dış servis ayrıntılarına bağımlı olmamalıdır. Application katmanı iş akışlarını düzenler; Infrastructure veritabanı ve dış servis gibi teknik ayrıntıları uygular. Bağımlılıkların içe doğru yönelmesi test edilebilirliği ve değişiklik yapmayı kolaylaştırabilir, ancak küçük projelerde gereksiz karmaşıklık oluşturmamaya dikkat edilmelidir.

---

# 5. Veritabanı ve ORM

## 5.1 SQL ve veritabanı türleri

SQL, ilişkisel veritabanlarında veri tanımlamak, sorgulamak ve değiştirmek için kullanılan dildir. İlişkisel veritabanları verileri tablolar, satırlar ve sütunlarla düzenler; tablolar arasındaki ilişkiler anahtarlarla kurulabilir.

İlişkisel olmayan (NoSQL) veritabanları belge, anahtar-değer, geniş sütun veya grafik gibi farklı modeller kullanabilir. Esnek veri yapısı veya özel ölçekleme ihtiyaçlarında yararlı olabilirler. Seçim, yalnızca “hangisi daha yeni?” sorusuna göre değil, veri modeli, tutarlılık, sorgular ve operasyonel ihtiyaçlara göre yapılmalıdır.

## 5.2 ORM ve Entity Framework Core

**ORM (Object-Relational Mapping)**, nesne yönelimli kod ile ilişkisel veritabanı tabloları arasındaki eşlemeyi kolaylaştırır. **Entity Framework Core (EF Core)**, .NET uygulamalarında kullanılan bir ORM'dir. C# nesneleri üzerinden sorgu ve değişiklik yapmayı sağlar; uygun sorguları SQL'e dönüştürebilir.

### DbContext nedir?

`DbContext`, EF Core ile veritabanı arasında oturum ve çalışma birimi görevi görür. Hangi varlıkların kullanılacağını `DbSet<T>` üzerinden tanımlayabilir; değişiklikleri `SaveChanges()` veya `SaveChangesAsync()` ile kaydedebilir.

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public decimal Price { get; set; }
}

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options) { }

    public DbSet<Product> Products => Set<Product>();
}
```

Uygulama yapılandırmasında uygun veritabanı sağlayıcısı ile bağlantı dizesi tanımlanır. Bağlantı dizesi parola içeriyorsa kaynak kod deposuna düz metin olarak eklenmemelidir.

## 5.3 LINQ örnekleri

LINQ, koleksiyonlar ve EF Core gibi veri kaynakları üzerinde sorguları C# diliyle yazmayı sağlar.

```csharp
var products = await db.Products
    .Where(p => p.Price > 500)
    .OrderBy(p => p.Name)
    .Select(p => new { p.Id, p.Name, p.Price })
    .ToListAsync();
```

Bu sorgu fiyatı 500'den yüksek ürünleri filtreler, isme göre sıralar ve yalnızca seçilen alanları getirir. EF Core bu sorguyu çoğunlukla SQL'e dönüştürür. Üretilen SQL sağlayıcıya ve sorguya bağlıdır; örnek bir karşılık:

```sql
SELECT Id, Name, Price
FROM Products
WHERE Price > 500
ORDER BY Name;
```

Diğer sık kullanılan LINQ ifadeleri:
- `FirstOrDefault`: İlk eşleşeni veya varsayılan değeri getirir.
- `Any`: En az bir eşleşme var mı kontrol eder.
- `Count`: Eşleşen kayıt sayısını verir.
- `Select`: Sonuçta hangi alanların üretileceğini belirler.
- `OrderBy` / `OrderByDescending`: Sıralama yapar.
- `Skip` / `Take`: Sayfalama için kayıt atlar ve sınırlar.

## 5.4 Code-First ve Database-First

| Yaklaşım | Açıklama | Ne zaman uygun olabilir? |
|---|---|---|
| Code-First | Model ve yapılandırma kodda tanımlanır; migration'larla şema güncellenir. | Yeni uygulama geliştirirken ve şemayı kodla yönetirken |
| Database-First | Var olan veritabanından model/sınıflar üretilir veya yapılandırılır. | Önceden kurulmuş veritabanıyla çalışırken |

Code-First yaklaşımında migration üretmek veritabanını otomatik olarak her ortamda güvenle güncellemek anlamına gelmez; üretim ortamında değişiklikler incelenmeli ve kontrollü uygulanmalıdır.

## 5.5 Temel SQL sorguları

```sql
-- SELECT: kayıtları okur
SELECT Id, Name, Price
FROM Products
WHERE Price > 500;

-- INSERT: yeni kayıt ekler
INSERT INTO Products (Name, Price)
VALUES ('Klavye', 749.90);

-- UPDATE: mevcut kaydı günceller
UPDATE Products
SET Price = 799.90
WHERE Id = 1;

-- DELETE: kaydı siler
DELETE FROM Products
WHERE Id = 1;
```

`UPDATE` ve `DELETE` sorgularında `WHERE` koşulunu kontrol etmek önemlidir. Koşul unutulursa tablodaki çok sayıda veya tüm kayıtlar etkilenebilir. Kullanıcı girdilerini SQL metnine doğrudan birleştirmek yerine parametreli sorgular veya EF Core gibi güvenli yöntemler tercih edilmelidir.

---

# 6. Güvenlik ve Performans

## 6.1 Authentication ve Authorization

- **Authentication (Kimlik doğrulama):** Kullanıcının kim olduğunu doğrular. Örneğin oturum açma veya token doğrulama.
- **Authorization (Yetkilendirme):** Kimliği doğrulanmış kullanıcının hangi işlemi yapabileceğini kontrol eder. Örneğin yalnızca yöneticilerin kullanıcı silebilmesi.

Kimlik doğrulamanın başarılı olması her kaynağa erişim izni olduğu anlamına gelmez. Yetki kontrolü sunucu tarafında yapılmalıdır.

## 6.2 JWT nedir?

JWT (JSON Web Token), taraflar arasında belirli bilgileri taşımak için kullanılan, yaygın olarak token tabanlı kimlik doğrulama sistemlerinde görülen bir biçimdir. Genellikle üç bölümden oluşur:

```text
header.payload.signature
```

- **Header:** Token türü ve imza algoritması gibi bilgileri belirtir.
- **Payload:** `sub` (özne), `exp` (son kullanma zamanı) gibi claim'leri taşıyabilir.
- **Signature:** Token'ın bütünlüğünü doğrulamaya yardımcı olur.

JWT'nin payload bölümü çoğu yaygın kullanımda yalnızca Base64URL kodlamasıyla taşınır; bu, içeriğin gizli olduğu anlamına gelmez. Parola veya hassas veriler payload'a konulmamalıdır. Sunucu imzayı, süreyi, issuer/audience gibi gerekli değerleri doğrulamalıdır.

## 6.3 OAuth 2.0, OpenID Connect ve OpenIddict

- **OAuth 2.0:** Bir uygulamanın, kullanıcının parolasını paylaşmadan belirli kaynaklara sınırlı erişim elde etmesini sağlayan yetkilendirme çerçevesidir.
- **OpenID Connect (OIDC):** OAuth 2.0 üzerine kimlik doğrulama katmanı ekler; kullanıcı kimliği hakkında standart bilgiler sağlar.
- **OpenID:** Kimlik doğrulama standartları ailesiyle ilişkili bir isimdir; modern web oturum açma bağlamında genellikle OpenID Connect kastedilir.
- **OpenIddict:** .NET uygulamalarında OAuth 2.0/OpenID Connect sunucusu veya ilgili bileşenleri geliştirmeye yardımcı olan bir kütüphane/çerçevedir.

OAuth tek başına “kullanıcının kim olduğunu doğrulama protokolü” olarak düşünülmemelidir; kimlik doğrulama ihtiyacı için OIDC gibi katmanlar kullanılır.

## 6.4 OWASP Top 10 ve temel önlemler

OWASP Top 10, web uygulamalarındaki önemli güvenlik risklerini özetleyen bilinen bir farkındalık kaynağıdır. Sürümler arasında başlıklar değişebilir; aşağıdaki liste OWASP Top 10:2021 kategorilerini temel alır.

1. **Broken Access Control:** Kullanıcılar izinleri olmayan kayıtlara veya işlemlere erişebilir. Sunucu tarafında her istekte uygun yetki kontrolü yapılmalıdır.
2. **Cryptographic Failures:** Hassas veriler uygun şekilde korunmayabilir. HTTPS kullanılmalı, parolalar güvenli parola hash algoritmalarıyla saklanmalı ve sırlar korunmalıdır.
3. **Injection:** Güvenilmeyen girdiler sorgu veya komutların yapısını değiştirebilir. Parametreli sorgular ve uygun girdi doğrulaması kullanılmalıdır.
4. **Insecure Design:** Güvenlik ihtiyaçları tasarım aşamasında ele alınmamıştır. Tehdit modelleme ve güvenli tasarım ilkeleri erken aşamada uygulanmalıdır.
5. **Security Misconfiguration:** Hatalı varsayılan ayarlar, açık hata detayları veya gereksiz servisler riske neden olabilir. Ortam yapılandırmaları gözden geçirilmelidir.
6. **Vulnerable and Outdated Components:** Eski veya bilinen açıkları olan bağımlılıklar kullanılabilir. Paketler güncel tutulmalı ve güvenlik duyuruları takip edilmelidir.
7. **Identification and Authentication Failures:** Oturum ve kimlik doğrulama hataları hesapların ele geçirilmesine yol açabilir. Güvenli oturum yönetimi ve uygun çok faktörlü doğrulama düşünülmelidir.
8. **Software and Data Integrity Failures:** Kod, güncelleme veya veri bütünlüğü yeterince doğrulanmayabilir. Güvenilir kaynaklar, bütünlük kontrolleri ve güvenli CI/CD süreçleri kullanılmalıdır.
9. **Security Logging and Monitoring Failures:** Saldırılar kaydedilmeyebilir veya fark edilmeyebilir. Önemli olaylar loglanmalı, loglar izlenmeli ve hassas bilgiler loglara yazılmamalıdır.
10. **Server-Side Request Forgery (SSRF):** Sunucu, saldırganın yönlendirdiği beklenmeyen adreslere istek atabilir. Hedef adresler doğrulanmalı ve ağ erişimi sınırlandırılmalıdır.

### SQL Injection, XSS ve CSRF

- **SQL Injection:** Kullanıcı girdisinin sorgu yapısını değiştirmesiyle oluşabilir. Parametreli sorgular kullanmak temel önlemdir.
- **XSS (Cross-Site Scripting):** Saldırganın tarayıcıda çalışacak zararlı içerik enjekte etmesidir. Çıktı bağlama uygun şekilde encode edilmeli, güvenilmeyen HTML gelişigüzel çalıştırılmamalıdır.
- **CSRF (Cross-Site Request Forgery):** Kullanıcının tarayıcısındaki kimlik bilgileri kullanılarak istemeden istek yaptırılmasıdır. Cookie tabanlı oturumlarda anti-forgery token, uygun SameSite ayarları ve doğru HTTP metotları önemlidir.

ASP.NET Core'da model validation, yetkilendirme politikaları, anti-forgery mekanizmaları, güvenli kimlik doğrulama yapılandırması ve parametreli veri erişimi gibi önlemler kullanılabilir. “Input sanitization” tek başına bütün saldırıları engelleyen genel bir çözüm değildir; doğrulama ve çıktı kodlama kullanım bağlamına göre yapılmalıdır.

## 6.5 Performans artırma teknikleri

1. **AsNoTracking:** EF Core'da yalnızca okunacak veriler için değişiklik takibini kapatabilir ve bazı sorgularda maliyeti azaltabilir.

```csharp
var products = await db.Products
    .AsNoTracking()
    .Where(p => p.Price > 100)
    .ToListAsync();
```

2. **Asenkron veri erişimi:** `ToListAsync`, `SaveChangesAsync` gibi metotlar I/O beklemelerinde iş parçacığı kullanımını daha verimli hâle getirebilir.
3. **Caching (önbellekleme):** Sık okunan ve kısa süre içinde değişmeyen verileri yeniden kullanır. Bellek içi cache tek sunucuda uygun olabilir; birden fazla sunucu arasında ortak cache gerektiğinde Redis gibi çözümler değerlendirilebilir.
4. **Sorgu ve indeks optimizasyonu:** Gereksiz alanları çekmemek, sayfalama kullanmak ve sık filtrelenen sütunlar için uygun indeksleri değerlendirmek önemlidir.
5. **Profiling ve ölçüm:** Darboğaz tahmin edilmek yerine ölçülmelidir. CPU, bellek, yanıt süresi ve veritabanı sorgu süreleri incelenebilir.
6. **IAsyncEnumerable:** Büyük sonuç kümelerini parça parça/asenkron biçimde tüketmeye yardımcı olabilir. Kullanımı, verinin nasıl üretildiğine ve kaynakların ne kadar süre açık kaldığına göre değerlendirilmelidir.

Optimizasyon öncesi ölçüm yapmak gerekir. Cache kullanımı güncellik sorunlarına, yanlış sorgu optimizasyonu ise karmaşıklığa neden olabilir.

---

# 7. Logging ve Hata Yönetimi

## 7.1 Loglama neden yapılır?

Loglama, uygulamanın çalışırken neler yaptığını ve hangi hatalarla karşılaştığını anlamaya yardımcı olur. Üretim ortamındaki sorunları teşhis etmek, performans problemlerini araştırmak ve güvenlik olaylarını takip etmek için kullanılır.

Yaygın log seviyeleri:

| Seviye | Kullanım amacı |
|---|---|
| `Trace` | Çok ayrıntılı teşhis bilgileri |
| `Debug` | Geliştirme ve hata ayıklama bilgileri |
| `Information` | Normal çalışma akışıyla ilgili önemli olaylar |
| `Warning` | Beklenmeyen ama uygulamayı hemen durdurmayan durumlar |
| `Error` | Bir işlem veya istek başarısız olduğunda |
| `Critical` | Uygulamanın veya önemli bir bileşenin çalışmasını tehdit eden ciddi durumlar |

Parolalar, erişim token'ları, gizli anahtarlar ve gereksiz kişisel bilgiler loglara yazılmamalıdır. Logların erişimi ve saklama süresi de güvenlik açısından yönetilmelidir.

## 7.2 ILogger kullanımı

ASP.NET Core, `ILogger<T>` üzerinden yapılandırılabilir loglama sunar.

```csharp
public class OrderService
{
    private readonly ILogger<OrderService> _logger;

    public OrderService(ILogger<OrderService> logger)
    {
        _logger = logger;
    }

    public void ProcessOrder(int orderId)
    {
        _logger.LogInformation("Sipariş işleniyor. OrderId: {OrderId}", orderId);

        // İş mantığı burada çalışır.
    }
}
```

Şablondaki `{OrderId}` gibi alanlar yapılandırılmış loglamayı destekler. Bu, mesajları daha sonra aramayı ve filtrelemeyi kolaylaştırabilir.

## 7.3 Global exception handling

Hataları her controller'da ayrı ayrı ele almak yerine merkezi bir hata yönetimi kullanılabilir. Üretim ortamında ayrıntılı stack trace veya dahili hata mesajlarını istemciye döndürmek güvenlik riski oluşturabilir.

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.UseExceptionHandler(errorApp =>
{
    errorApp.Run(async context =>
    {
        context.Response.StatusCode =
            StatusCodes.Status500InternalServerError;
        context.Response.ContentType = "application/json";

        await context.Response.WriteAsJsonAsync(new
        {
            message = "Beklenmeyen bir hata oluştu."
        });
    });
});

app.MapGet("/", () => "Uygulama çalışıyor");
app.Run();
```

Bu temel örnek istemciye genel bir mesaj verir. Gerçek projede hata ayrıntısı sunucu tarafında `ILogger` ile kaydedilmeli; hata kimliği veya correlation ID ile istek ilişkilendirilebilmelidir. Bilinen iş hataları için uygun HTTP durum kodları ve kontrollü yanıtlar tasarlanmalıdır.

---

# 8. Yazılım Geliştirme Prensipleri

## 8.1 SOLID prensipleri

SOLID, nesne yönelimli yazılım tasarımını daha anlaşılır ve değiştirilebilir kılmayı amaçlayan beş prensibin baş harflerinden oluşur.

### S — Single Responsibility Principle
Bir sınıfın değişmek için tek bir ana sorumluluğu olmalıdır. Örneğin sipariş hesaplayan sınıfın aynı zamanda e-posta gönderme ve dosyaya log yazma görevlerini üstlenmesi yerine bu işler ayrı bileşenlere ayrılabilir.

### O — Open/Closed Principle
Yazılım birimleri yeni davranışlara genişletmeye açık, mevcut çalışan kodu sürekli değiştirmeye daha az ihtiyaç duyacak şekilde tasarlanmalıdır. Örneğin her ödeme türü için aynı metodu tekrar tekrar değiştirmek yerine bir ödeme arayüzünün farklı uygulamaları kullanılabilir.

### L — Liskov Substitution Principle
Bir alt tür, kullanıldığı yerde temel türün beklenen davranışını bozmadan yerine geçebilmelidir. Alt sınıf temel sınıfın sözleşmesini ihlal ediyorsa tasarım yeniden düşünülmelidir.

### I — Interface Segregation Principle
Sınıflar ihtiyaç duymadıkları metotları içeren büyük arayüzlere bağımlı olmamalıdır. Tek bir geniş arayüz yerine amaca yönelik küçük arayüzler daha uygundur.

### D — Dependency Inversion Principle
Üst seviye iş mantığı somut altyapı sınıflarına doğrudan bağlanmamalı; soyutlamalara bağımlı olmalıdır. Örneğin iş servisi doğrudan belirli bir e-posta sağlayıcısını oluşturmak yerine `IEmailSender` arayüzünü kullanabilir.

Bu prensipler katı kurallar gibi her projeye aynı biçimde uygulanmamalıdır. Amaç gereksiz soyutlama oluşturmak değil, değişiklik ve test maliyetini azaltmaktır.

## 8.2 Design Patterns

### Singleton

Bir uygulama kapsamında bir nesnenin tek örneğinin kullanılmasını amaçlayan desendir. ASP.NET Core DI'da `AddSingleton` ile kaydedilebilir. Paylaşılan durum içeren singleton nesnelerin eşzamanlı erişime uygun olması gerekir.

```csharp
builder.Services.AddSingleton<ITimeProvider, SystemTimeProvider>();
```

Singleton; yapılandırma veya durumsuz, paylaşılması güvenli servislerde uygun olabilir. Her servisi singleton yapmak doğru değildir.

### Repository

Veri erişimini bir arayüz arkasına alarak üst katmanların veritabanı ayrıntılarından ayrılmasına yardımcı olur. Örneğin `IProductRepository` ürünleri getirme ve kaydetme işlemlerini tanımlayabilir. Ancak EF Core'un `DbContext` ve `DbSet` yapıları zaten repository/unit-of-work benzeri özellikler taşıdığından, ek repository katmanı ihtiyaca göre tasarlanmalıdır.

### Factory

Nesne oluşturma mantığını ayrı bir yerde toplamaya yarar. Hangi sınıfın oluşturulacağı koşullara göre değişiyorsa factory kullanmak ana kodu sadeleştirebilir. Örneğin farklı bildirim türleri için uygun gönderici nesnesini seçen bir factory düşünülebilir.

## 8.3 Clean Code

Clean Code, kodun yalnızca bilgisayar tarafından çalıştırılmasını değil, insanlar tarafından da kolay okunup anlaşılmasını hedefler.

İyi uygulamalara örnekler:

- Anlamlı isimler kullanmak: `x` yerine bağlam uygunsa `productPrice`.
- Bir metoda tek ve net sorumluluk vermek.
- Tekrarlanan mantığı uygun bir yerde toplamak.
- Gereksiz yorumlar yerine kodun niyetini açık isimlerle anlatmak.
- Hataları sessizce yutmak yerine uygun biçimde ele almak.
- Birim testleri yazmak ve değişikliklerden sonra çalıştırmak.
- Biçimlendirme ve isimlendirme standartlarını ekip içinde tutarlı uygulamak.

Örnek:

```csharp
// Daha anlaşılır
decimal CalculateTotal(decimal unitPrice, int quantity)
{
    return unitPrice * quantity;
}
```

Bu örnekte isimler metodun ne yaptığını ve parametrelerin anlamını gösterir. Gerçek bir sipariş sisteminde indirim, vergi ve para birimi gibi kurallar da ayrıca ele alınabilir.

## 8.4 Yazılım mimarileri karşılaştırması

| Mimari | Temel fikir | Uygun olabileceği senaryo | Dikkat edilmesi gereken |
|---|---|---|---|
| Layered Architecture | Sistemi sunum, iş ve veri erişimi gibi katmanlara ayırır. | Küçük ve orta ölçekli kurumsal uygulamalar | Katmanlar arasında gereksiz bağımlılık oluşabilir. |
| Clean Architecture | İş kurallarını dış teknoloji ayrıntılarından bağımsız tutar. | Test edilebilirlik ve uzun vadeli bakım önemliyse | Küçük projelerde fazla soyutlama yaratabilir. |
| Microservices | Sistemi ayrı dağıtılabilen servislerden oluşturur. | Bağımsız ölçekleme ve ekip sahipliği gerekiyorsa | Dağıtık sistem, izleme, ağ ve veri tutarlılığı karmaşıklığı getirir. |
| Event-Driven | Bileşenler olaylar üzerinden iletişim kurar. | Asenkron iş akışları ve gevşek bağlılık gereken sistemler | Olay sırası, tekrar işleme ve hata yönetimi dikkat ister. |
| Hexagonal Architecture (Ports & Adapters) | İş mantığını dış sistemlere port ve adapter'larla bağlar. | Altyapıların değişebilmesi ve iş mantığının izole test edilmesi isteniyorsa | Arayüz ve adapter sayısı artabilir. |

Mimari seçimi uygulamanın boyutuna, iş gereksinimlerine, ekip deneyimine, dağıtım modeline ve bakım maliyetine göre yapılmalıdır. Mikroservisler her zaman daha iyi değildir; basit bir uygulama için iyi tasarlanmış tek bir uygulama daha az operasyonel yük getirebilir.

---

# Sonuç

.NET backend geliştirme yalnızca C# ile kod yazmaktan ibaret değildir. Sürüm kontrolü, otomatik test ve dağıtım, HTTP ve API tasarımı, veritabanı modelleme, güvenlik, loglama ve sürdürülebilir mimari birlikte düşünülmelidir.



