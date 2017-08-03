---
Title: Pure Function ve Idempotent Function Kavramı Hakkında
PublishDate: 03/08/2017
IsActive: True
Tags: C#
---

Nesne yönelimli programlama mantığında yazılım geliştirirken SOLID gibi DRY gibi prensiplere uymamız gerekir. Yine bu prensiplere benzer şekilde,  oluşturduğumuz sınıfların, sınıf elemanlarının (_field_) ve metotlarında (_ortamımız nesne yönelimli olsun veya olmasın_) belirli prensiplere sahip olması gerekir.
İyi bir koddan beklentilerimizin başında o kodun hiçbir değişiklik yapılmaksızın tekrar kullanılabilir olması ve bakım masrafının düşük olması gelir. Daha açık olması için bakım masrafı kısmını genişletelim. Çalışma zamanında oluşacak bir hata veya beklenmeyen bir durumda problemin kaynağının çok hızlı şekilde bulunması ve ilgili parçanın düzeltilmeye çalışılmasının başka yerleri etkileme riski olmamasını bekleriz. Yine bununla ilgili olarak kodlarımızın kolayca test edilebilir olmasını bekleriz.
İşte bu gibi sebeplerle kodumuzun en temel parçalarından biri olan metotlarda da prensip sahibi olmanın faydaları olacaktır. Bu yazıda _pure function_ kavramından bahsedeceğim. Kendisi aslında bir kavramın adı olsa da bir metot _pure_ şekilde yazılma şansı varsa bu şekilde yazmak mevcut prensiplerin öğretilerini içermektedir.

Tanımlaması daha kısa olduğu için _idempotent_ fonksiyonlarla başlıyorum:

> **Idempotent functions**  (eşgüçlü olarak çevriliyor) saf (_pure_) olup olmadıklarından bağımsız olarak her çağrıldığında durum (_state_) değiştirmeyen ve aynı sonucu veren metotlara denilmektedir. Fonksiyonel programlamada ise tanım şu şekildedir:  `f(x) = y` için `f(f(x)) = y` durumunu sağlayan fonksiyonlardır. 

Programlama açısından en temel örnekler HTTP GET methodu örneklenebilir. Bunu standart bir html dosyası için düşündüğümüzde, kullanıcı arka arkaya bu sayfayı istediğinde sonuç veya durum değişmez. Tabii sizin bu davranışı bilinçli değiştirmediğinizi varsayıyorum.

Bu konuda daha anlaşılır örneklerin başlıcaları ise; öncelikle mutlak değer fonksiyonunu ele alalım `abs(-4) = 4` sonucunu verirken `abs(abs(abs(-4)))` de 4 sonucunu vermektedir. Matematikten devam edelim ve 0 ile çarpma yapan bir fonksiyon düşünelim. Sonuç hep 0 olacaktır. Yine bir metnin tüm harflerini büyük harfe çeviren bir fonksiyon, kendi kendisine çağrıldığında sonuç değişmeyecektir. Tüm metin kutularının içindeki metinleri temizleyen bir fonksiyon da buna örnek verilebilir.

Pure fonksiyonlar ile devam ediyorum.

> **Pure (Saf) functionlar** aldıkları argümanlar üzerinde, bellekte ve I/O da yan etkisi olabilecek hiç bir değişiklik yapmayan ve yan etkisi bulunmayan fonksiyonlardır. Bir pure functionın aynı argümanlar  ile her zaman, her makinede aynı sonucu vermesi beklenir ve bu argümanlar saklı, sınıf içindeki diğer elemanlar, kullanıcı girişi olmamalıdır.

En temelinden matematiksel fonksiyonlar bu kapsama dahildir. Örnekleyelim:

```csharp
using System;
namespace Pure
{
    internal class Program
    {
        private static void Main()
        {
            Console.WriteLine(Topla(5,6));
        }

        private static int Topla(int a, int b)
        {
            return a + b;
        }
    }
}
```
Burada `Topla` fonksiyonumuz her koşulda 11 sonucunu verecektir ve programın işleyişi açısından bir yan etkiye sebebiyet verecek bir işlem yapmamaktadır. Metot üzerinde ufak bir değişiklik yapalım ve *pure* olmaktan çıkartalım :

```csharp
private static int Topla(int a, int b)
{
    var sonuc =  a + b;
    Console.WriteLine(sonuc);
    return sonuc;
}
```
Burada metodun saflığını bozan şeyi görebildiniz mi? Bu farkı yaratan şey `sonuc` değişkeni değil. Çünkü _pure_ metotlar kendi içerisindeki işlemler için bellek kullanabilirler (_programın işleyişine karşı yan etkisi olmadan_). O zaman sorun geriye kalan `Console.WriteLine(sonuc);` satırında gibi gözüküyor. Evet öyle de. Peki ama neden? Bir saf fonksiyonun herhangi bir I/O işlemi yapması beklenmez. Burada uygulamaya ait _Console_ akışına (_stream_) müdahale edilmektedir. Eğer ki uygulamamızın içerisinde veya dışarısında bir kod parçası bu akışa göre işlem yapıyor olsa ona müdahale etmiş olurduk. Bu da saflık kurallarından birisini çiğnemek oluyor.

Aynı örneği farklı şekilde yapalım:

```csharp
internal class Program
{
    private static int _sayi1;
    private static int _sayi2;
    private static int _sonuc;

    private static void Main()
    {
        int.TryParse(Console.ReadLine() ?? "0", out _sayi1);
        int.TryParse(Console.ReadLine() ?? "0", out _sayi2);
        Topla();
        Console.WriteLine(_sonuc);
    }

    private static void Topla()
    {
        _sonuc = _sayi1 + _sayi2;
    }
}
```
Bu örnekte ise `_sonuc` değişkeni metodun içerisinde olmadığı halde metot bu değişken üzerinde değişiklik yapmıştır. Bundan dolayı metot _pure_ değildir.

Benzer şekilde aşağıdaki metot örnekleri de saf (_pure_) değildir:

```csharp
private static int KalanGunSayisi(DateTime hedef)
{
    return (int)DateTime.Now.Subtract(hedef).TotalDays;
}

private static int ZarAt()
{
    return new Random().Next(1, 7);
}
```
`KalanGunSayisi` isimli method bugün için hep aynı değeri verirken, yarın çalıştırdığımda aynı sonucu vermeyeceği için saf kabul edilemez.
`ZarAt` metodu da benzer şekilde her çalıştırıldığında farklı sonuç vermektedir.

Peki şuna bakalım:

```csharp
internal class Program
{
    private static int _sayi1;
    private static int _sayi2;

    private static void Main()
    {
        int.TryParse(Console.ReadLine() ?? "0", out _sayi1);
        int.TryParse(Console.ReadLine() ?? "0", out _sayi2);
        Console.WriteLine(Topla());
    }

    private static int Topla()
    {
       return _sayi1 + _sayi2;
    }
}
```
Örnekteki `Topla` metodu için ne diyebiliriz? Burada `_sayi1` ve `_sayi2` değerleri açıkça argüman olarak gelmeseler de metot için gizli birer argümandırlar. Fakat tanımda açıkça bu argümanların saklı olmaması veya bir değişken olmaması gerektiği belirtilmektedir. Neticede `Topla()` fonksiyonunu her çağırdığımızda başka bir metodun etkisi ile `_sayi1`in değişip değişmediğini bilemeyeceğimizden her çağırdığımızda farklı sonuç dönme olasılığı vardır.

> **Not 1:** Erişilen sınıf üyesinin sabit (_const_) veya değişmez ve saltokunur (immutable and readonly) olması durumunda bu fonksiyonun _pure_ olacağı görüşü bulunmaktadır. 
>
> **Not 2:**  Saf bir metot not1 de yazdığımız durumlar dışında genellikle _static_ olarak tanımlanabiliyor olması gerekir. Tanımlanamıyorsa sınıf içerisindeki bir değişkene bağımlılığı var demektir ve bu onu artık saf yapmaz.

Bir fonksiyon pure olmayan başka bir fonksiyonu çağırdığında artık pure kabul edilmez. Örneğin :

```csharp
 private static int Topla(string a, string b)
 {
     Console.WriteLine(2131);
     int.TryParse(Console.ReadLine() ?? "0", out int sayi1);
     int.TryParse(Console.ReadLine() ?? "0", out int sayi2);
     return sayi1 + sayi2;
 }
```

Burada`TryParse` methodunun kendisi pure tanımı dışında olduğundan (_out kullandığınız an fonksiyon dışında bir bellek değişikliği yaptığımız için bu durum oluşur_) kendi methodunuz için de saf demek doğru olmayacaktır.

Kod yazarken bir fonksiyonu saf olarak yazabileceğinizi fark ettiğiniz an öyle yazmalıyız. Bunun faydaları:

* Bağımsız methodlar yazdığınız için hata ayıklama işleriniz kolaylaşacak.

* Metotlarınız için çok hızlı şekilde birim testi (_unit test_) yazabileceksiniz.

* _Static_ tanımlanmış methodlar genellikle örnek (_instance_) methodlara göre daha hızlı çalışmaktadır.

* Asenkron, parallel kodlar yazdığınızda tahmin edilemeyen garip sonuçlar almazsınız.

* Aynı girdi için her zaman aynı çıktı olacağından sonuçlar bir _cache_'e alınabilir. 

  ​



