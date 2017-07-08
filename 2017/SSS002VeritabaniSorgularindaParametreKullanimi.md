---
Title: SSS002 Veritabanı Sorgularında Parametre Kullanımı
PublishDate: 08/07/2017
IsActive: True
Tags: C#, SQL
---

Yazılım geliştirme serüveninde veritabanları ile çalışılmaya başlayan kişilere baktığımda genelikle "hızlı" ilerleme görüyorum. Veritabanlarının nasıl çalıştığı veya T-SQL dilinin mantığının pek anlaşılmadan girişilen projelerde genellikle çok büyük güvenlik açıkları veya performans problemleri doğabiliyor. Bu sebeple en temelden başlayarak parametre kullanımını anlatmaya gayret edeceğim.

### T-SQL tarafı

Öncelikle C#'ı karıştırmadan ilerleyelim. Veritabanında basit bir tüm müşterileri getirecek sorgu yazalım :

```sql
SELECT * FROM Musteri
```
Bu sorguyu eğer bir programda kullanacaksak ve özel başka bir amacımız yoksa * dan kurtulmalıyız. Burada bir kaç sebebimiz var :

 * Sadece ihtiyacımız olan nitelikleri alarak veri boyutunu azaltmak
 * Olası şema değişikliklerinde istemci uygulamanın patlamasını engellemek
 * Performans (çalışma planı, indexler vs...)

Sorgumuz şu hale getirelim :

```sql
SELECT Ad, Soyad, CepTelefon FROM Musteri
```

Buraya kadar iyi ama ben "kişisel" bir son dokunuş yapmak istiyorum ve sorgu şu hale geliyor :

```sql
SELECT [M].[Ad], [M].[Soyad], [M].[CepTelefon] FROM [dbo].[Musteri] as [M];
```

Burada yaptığım dokunuşların sebepleri şunlar :

* `[]` kullanımı ile keywordler ile veritabanı nesnelerinin farkını rahat görmek.
* `dbo` ile scheme'ı özellikle belirtme ihtiyacı duyuyorum
* `as [M]` ile sorgunun bu sonucuna bir isim veriyorum ve bu sonuçtan dönen nitelikleri `[M].[Ad]`  gibi kullanıyorum. Daha sonra bu sorguya bir _JOIN, Subquery_ vb. eklediğimde karışıklıkların oluşmasını engellemiş oluyorum.
* `;` ise bir sorgunun bittiğini gösterdiği için uzun sorgularda oldukça işe yarıyor. (_WITH gibi keywordlerin kullanımında ise noktalı virgül zaten zorunlu olarak bulunmalıdır_)

> **Not:** İsteğe bağlı olarak, bu sorguda herhangi bir transaction derdiniz yoksa isolation seviyesini düşürecek hamleler ( `WITH (NOLOCK)` gibi)  yapabilirsiniz.

Asıl meseleye yavaştan gelelim, ben sadece adı "Cihan" olan müşterileri getirmek istiyor olayım. Bu durumda şöyle bir sorgu yazılması beklenir.

```sql
SELECT [M].[Ad], [M].[Soyad], [M].[CepTelefon] FROM [dbo].[Musteri] as [M] WHERE [M].[Ad] = 'Cihan';
```

Bu sorguyu tek sefer kullanmak için yazıyorsanız sıkıntı yok, fakat burada "Cihan" yerine farklı farklı değerler vererek sorgulamalar yapmayı amaçlıyorsanız bu yöntemi öneremeyeceğim. Çünkü hem yönetimi zor bir yöntem olacaktır hem de yine performans açısından sıkıntıları olacaktır. Sorguyu şu şekilde değiştiriyorum:

```sql
DECLARE @ad nvarchar(50) = 'Cihan';

SELECT [M].[Ad], [M].[Soyad], [M].[CepTelefon] FROM [dbo].[Musteri] as [M] WHERE [M].[Ad] = @ad;
```

Sorgum bu hale geldiğinde aslında bir kaç iş birden yaptım: 

Sorguda kullanılacak nesnenin tipini açıkça belirttim. Bu sayede örtülü tür dönüşümü yaşanmayacak.

Aynı değeri birden fazla kullandığımda tekrar tekrar yazmak zorunda kalmayacağım.

Ana sorgu parametreli hale gelmiş oldu bu sayede değer "Cihan" da olsa "Daron" da olsa aynı planı kullanacak. Aksi durumda veritabanınıza 10000 farklı isimle sorgu atıldığında her biri farklı planla yorumlanması gerekecek.

Tekrar hatırlatıyorum bu kullanımı gerçek senaryoda sürekli değişkenlik gösterecek durumlar için yapıyoruz. Örneğin veritabanında soft delete yani `SilindiMi` gibi bir alan ile bir kaydı silinmiş gibi gösteriyor olun. Bu durumda bu sorguda hep 0 değerini kullanacağınız için bu kısmı parametrik yapmanıza gerek olmayackatır:
Örnek:

```sql
DECLARE @ad nvarchar(50) = 'Cihan';

SELECT [M].[Ad], [M].[Soyad], [M].[CepTelefon] FROM [dbo].[Musteri] as [M] WHERE [M].[Ad] = @ad AND [M].[SilindiMi] = 0;
```
> *İPUCU*: Nadir olarak değişkenlik gösteren durumlar oluyorsa bu durumda parametre kullanımında [OPTIMIZE FOR](https://docs.microsoft.com/en-us/sql/t-sql/queries/hints-transact-sql-query) ile sık kullanılan değere göre optimizasyon sağlayabilirsiniz.

### Program Tarafı

C# tarafına dönelim ve aynı sorguyu yazalım :

```csharp
public async Task<IEnumerable<Musteri>> MusteriAra()
{
    using (var connection = new SqlConnection(Configuration.GetConnectionString()))
    using (var command = connection.CreateCommand())
    {
	    await connection.OpenAsync();
        command.CommandText = @"DECLARE @ad nvarchar(50) = 'Cihan';
                            SELECT [M].[Ad], [M].[Soyad], [M].[CepTelefon] 
                            FROM [dbo].[Musteri] as [M] 
                            WHERE[M].[Ad] = @ad AND [M].[SilindiMi] = 0;";
        var reader = await command.ExecuteReaderAsync();
        //...
    }
}
```

Bu kod çalışacaktır pek tabi, ama ben argümanı kullanıcının belirlemesini istiyorsam ne yapacağım? Burada sıklıkla yapılan hata SQL cümlesinin bir string olmasından dolayı + operatörü ile birleştirme yapmaya çalışmaktır.

```csharp
public async Task<IEnumerable<Musteri>> MusteriAra(string ad)
{
    using (var connection = new SqlConnection(Configuration.GetConnectionString()))
    using (var command = connection.CreateCommand())
    {
	    await connection.OpenAsync();
        command.CommandText = @"DECLARE @ad nvarchar(50) = '" + ad + @"';
                            SELECT [M].[Ad], [M].[Soyad], [M].[CepTelefon] 
                            FROM [dbo].[Musteri] as [M] 
                            WHERE[M].[Ad] = @ad AND [M].[SilindiMi] = 0;";
        var reader = await command.ExecuteReaderAsync();
        //...
    }
}
```

Burada hemen  uygulamanıza bir metin kutusu koyduğunuzu ve şu şekilde kod yazdığınızı düşünün :

``` csharp
//...
	command.CommandText = TextBox1.Text;
//...
```
Böyle bir özellikle kullanıcı veritabanınıza neler yapabilir? Cevap basit, SQL kullanıcısına ne yetki verdiyseniz onu yapabilir. Çoğu durumda veritabanını komple silebilir bile. **İşte + ile SQL cümlesi oluşturduğunuzda kullanıcıya aynı şekilde veritabanında tam yetki vermiş oluyorsunuz.** Nasıl mı?  Kullanıcının şu isimde müşteri aradığını düşünelim : ` '; DROP TABLE Musteri; --` bu durumda oluşacak SQL cümlesi:

```sql
DECLARE @ad nvarchar(50) = ''; DROP TABLE Musteri; --';

SELECT [M].[Ad], [M].[Soyad], [M].[CepTelefon] FROM [dbo].[Musteri] as [M] WHERE [M].[Ad] = @ad;
```
haline dönüşecektir, bu SQL cümlesi içinde parametre kullanmasanız da aynı şekilde tehlikelidir. Çoğu  bol yetki durumunda kullanıcı tabloyu silmiş olacaktır.

Bu fark edildiği anda durumu çözmek için daha vahim bir yöntem uygulanmaya çalışılır. Bu yöntem, tehlike oluşturabilecek karakterlerin argümanlardan temizlenmesidir. Örneğin `'` karakteri temizlenir ve kullanıcının bu kötü niyetli işleri yapması engellemesi amaçlanır. Bu aslında bir çözüm değildir. Bu kötü yöntemde  

* argüman dikkatli temizlenmezse ise kaçak olabilir 
* temizleme işlemi sorguyu bozabilir
* kullanıcı gerçekten bu karakterleri kullanmak istediğinde sıkıntılar çıkmaya başlar
* bu karakterler kullanıldığında sistemin hatalı çalışma durumları oluşabilir -ki genellikle oluşur  ve onlar için de tekrar tekrar replace'ler yazılır- 
* kod kalabalığı/kötü kokusu oluşturur

Çözüm nedir? Sorgumuzun içinde zaten değişkenler kullanmıştık, bu değişkenlerin değerlerini SQL Cümlesi ile değil de doğrudan `Command` nesnesi üzerinden verebiliriz. Bu durumda hem tüm karakterleri korkusuzca kullanabiliriz hem de  değişkenin türünden kaynaklı tırnak, tarihin stringe dönüştürülmesi gibi problemler ortaya çıkmaz.

```csharp
public async Task<IEnumerable<Musteri>> MusteriAra(string ad)
{
    using (var connection = new SqlConnection(Configuration.GetConnectionString()))
    using (var command = connection.CreateCommand())
    {
        await connection.OpenAsync();
        command.CommandText = @"SELECT [M].[Ad], [M].[Soyad], [M].[CepTelefon] 
                    FROM [dbo].[Musteri] as [M] 
                    WHERE[M].[Ad] = @ad AND [M].[SilindiMi] = 0;";

        //command.Parameters.AddWithValue("ad", ad);
        command.Parameters.Add(
            new System.Data.SqlClient.SqlParameter("@ad", SqlDbType.NVarChar, 40) {Value = ad});

        var reader = await command.ExecuteReaderAsync();
        //... mapping vb. işlemler burada
    }
}
```

Kodda görülebileceği gibi, SQL cümlesindeki değişken tanımlarını kaldırıyoruz. Bu tanımları Command nesnesi üzerinden tamamen statik olarak veriyoruz. Böylece argümanın içinde hangi karakterlerin olduğunun veritabanı güvenliği açısından bir sıkıntısı olmuyor. Yine string birleştirme yapmadığım için tarih ve sayısal alan gibi türlerde kültür bağımlılığından kurtulmuş oluyorum. Koda baktığınızda bir yorum satırı göreceksiniz. Parametre'nin tanımı farklı farklı Ado.net e söyleyebiliyorsunuz, yorum satırında da en basit söyleme şekli bulunuyor. Kendisi otomatik tür eşleştirmesi yapmış oluyor. Ben bu yöntemi sağlıklı bulmadığımdan parametrenin türü SQL'de nasıl olması gerekiyorsa bunu açıkça belirtmek için uzun yazımı tercih ediyorum. 

Son olarak, her zaman kullanıcıya kodu ve şifreleri açık şekilde vermişsiniz gibi kod yazmanızı öneririm. Kullanıcı istediği zaman  veritabanına erişip istediği sorguyu çalıştırabilecek, ekranda sizin pasif hale getirdiğiniz düğmeye tıklayabilecek gibi düşünün. Tüm güvenlik önlemlerini böyle gibi düşünerek alın, asla ama asla parametreli sorgu gönderiyorum, kodları, veritabanı bilgilerini şifreliyorum dolaysıyla uygulamam güvenli diye düşünmeyin.  Bu da başka bir yazının konusu olsun.