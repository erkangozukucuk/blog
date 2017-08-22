---
Title: String Interning Kavramı Hakkında
PublishDate: 22/08/2017
IsActive: True
Tags: C#
---

Öncelikle bu yazıyı kavramak için _immutable tiplerin_ bilinmesi gerekecektir. Okumadıysanız [ilgili yazıma](http://cihanyakar.com/immutable-type_kavrami) ve diğer kaynaklara bakmanızı tavsiye ederim. 

Muhtemelen aşağıdaki gibi bir kod görmüşsünüzdür:

```csharp
private string isim = string.Empty;
```
Aynı kod şu şekilde de yazılabilirdi:

```csharp
private string isim = "";
```
ama yazılmamış. Peki ama neden?

Bu kodu gördüğünüzde akla ilk olarak boş _string_in farklı platformlarda farklı değerler aldığı gelebilir. Fakat durum böyle olsaydı tıpkı yeni satır karakteri gibi _System.Environment _ sınıfının altında veya sisteme özel olduğunu belirten başka bir şekilde tanımlanmış olması gerekirdi. Yani bu tahmin yanlıştır.
Daha fazla tahmin ile sizi uğraştıracak değilim. Burada amaçlanan uyum veya kodun daha güzel görünmesi vb. değil, amaç kısaca .net 2.0 öncesinde performans artışı ve gereksiz bellek kullanımını azaltmayı sağlamaktır.
Her şeyin toz bulutu olduğu duruma dönelim ve .net dünyasında iki nesnenin eşit olup olmadığını karşılaştırmak isteyelim. Bu karşılaştırmayı hem bir referans türü hem de bir değer türü ile ayrı ayrı yapacak şu kodları yazmış olalım:

```csharp
class Program
{

    internal class ReferansTur
    {
        public int Sayi { get; set; }
    }

    internal struct DegerTur
    {
        public int Sayi { get; set; }
    }

    static void Main(string[] args)
    {
        var a = new ReferansTur {Sayi = 4};
        var b = new ReferansTur {Sayi = 4};
        Console.WriteLine(object.Equals(a, b));
        Console.WriteLine(object.ReferenceEquals(a, b));
		Console.WriteLine(object.ReferenceEquals(a, a));

        var x = new DegerTur {Sayi = 4};
        var y = new DegerTur {Sayi = 4};
        Console.WriteLine(object.Equals(x,y));
        Console.WriteLine(object.ReferenceEquals(x, y));
        Console.WriteLine(object.ReferenceEquals(x, x));

        Console.ReadKey();
    }
}
```
Bu işlemin sonucu:

```
False // object.Equals(a, b)
False // object.ReferenceEquals(a, b)
True  // object.ReferenceEquals(a, a)

True  // object.Equals(x,y)
False // object.ReferenceEquals(x, y)
False // object.ReferenceEquals(x, x)
```

Birinci karşılaştırma _false_ sonucunu vermiştir. Çünkü referans türlerinde siz eşitliğin nasıl olacağını tanımlamadığınız sürece aslında eşitlik karşılaştırması iki nesnenin _heap_ bellek adreslerinin eşit olup olmadığına göre yapılır. Yani aslında _ReferenceEquals_ çağrılır.
İkinci karşılaştırma _false_ dur. Çünkü bu iki nesne referans türü oldukları için _new_ ile bellekte farklı alanlardadır.
Üçüncü karşılaştırmada a nın hedef bellek adresi a'ya eşit mi diye bakıldığında aynı sonucu vereceğinden _true_ sonucu gelmektedir.
Dördüncü karşılaştırmada değer türlerine geçiyoruz. Bu iki nesne _stack_ üzerinde farklı yerlerde de olsa değer türleri özelliklerinin tek tek bir birine eşit olup olmamalarına göre eşitlik kontrolü yapılır. Minik performans iyileştirmelerinde özelliklerin tanım sırası bu sebeple önemlidir.
Dördüncü ve beşinci karşılaştırmalarda ise nesnenin kendisi dahi verilse referansların eşit olmadığını görüyorsunuz. Zaten elimizdekiler değer türüydü doğal olarak bir referansları da yok. Ama burada _false_ sonucunun gelmesinin temel sebebi _ReferenceEquals_ methodunun parameterlerinin türünün _object_ olmasından kaynaklanıyor. Burada bir kapalı _boxing_ işlemi gerçekleşiyor ve iki _object_ türünden yeni nesne oluşuyor ve ikinci karşılaştırmadaki durumun aynısı oluşuyor. 

Şimdi hemen hızlıca konumuza geri dönelim. String bildiğiniz gibi bir referans türüdür. Bu durumda referans türlerinin davranışları gereği aşağıdaki eşitliğin mümkün olmaması gerekir :

```csharp
static void Main(string[] args)
{
    string a = "cihan";
    string b = "cihan";

    Console.WriteLine(object.Equals(a,b)); // TRUE
    Console.WriteLine(object.ReferenceEquals(a,b)); // TRUE

    Console.ReadKey();
}
```
Burada gelen sonuçlar bir önceki örneğimize uymamaktadır. Eğer _string_ referans türü olsaydı iki eşitliğinde _false_ olması beklenirdi. Eğer değer türü olsaydı ilki _true_ ikincisi _false_ olmalıydı. Olayı biraz daha açıklığa kavuşturmak için şunu yazalım:

```csharp
 static void Main(string[] args)
 {
     string a = Console.ReadLine(); // "cihan" yazıyorum
     string b = Console.ReadLine(); // "cihan" yazıyorum

     Console.WriteLine(object.Equals(a,b)); // TRUE
     Console.WriteLine(object.ReferenceEquals(a,b)); // FALSE

     Console.ReadKey();
 }
```
Bu örnekte girdiyi kullanıcıdan almaktayım (_dosyadan da okuyabilirdim veya internet üzerinden de alabilirdim sonuç değişmezdi_). Birebir aynı değeri girdiğim halde _ReferenceEquals_ artık _false_ değeri almaktadır. Peki ne değişti? Kodu şu şekilde değiştirmiş olalım:

```csharp
static void Main(string[] args)
{
    string a = string.Intern(Console.ReadLine()); // "cihan" yazıyorum
    string b = string.Intern(Console.ReadLine()); // "cihan" yazıyorum

    Console.WriteLine(object.Equals(a,b)); // TRUE
    Console.WriteLine(object.ReferenceEquals(a,b)); // TRUE

    Console.ReadKey();
}
```

Burada ise ilk durumun aynı sonucu almış oldum. Aslında ilk durumda olanda buradaki örneğin aynısı. Sanki ilk  kodumuz şöyle yazılmış gibi çalışmaktadır :

```csharp
static void Main(string[] args)
{
    string a = string.Intern("Cihan"); 
    string b = string.Intern("Cihan");  

    Console.WriteLine(object.Equals(a,b)); // TRUE
    Console.WriteLine(object.ReferenceEquals(a,b)); // TRUE

    Console.ReadKey();
}
```

_Interning_ işlemi bir metin için bellekte bir yer zaten ayrıldıysa o metin için hep o bellek bölgesinin kullanılmasını sağlayarak bellekten kazanma ve karşılaştırma işleminde metnin içeriğine harf harf bakmak yerine sadece adresine bakarak işlem zamanı kazanmayı sağlar. Güncel .net Framework sürümlerinde kod içerindeki metin kalıplarına (_string literals_) otomatik olarak bu işlem uygulanır.

Konunun en başına dönecek olursak, eski frameworklerde bu özellik olmadığı için örneğin ekrandaki tüm metin kutularını boşaltmak adına şöyle bir atama yaptığınızda :

```csharp
foreach (var textBox in TextBoxes)
{
    textBox.Text = "";
}
```
Her döngüde yeni bir string üretilmekteydi. Bunun önüne geçmek için boş metin gibi özel metinler ayrıca tanımlanmıştır. Güncel sürümlerde `string.Empty` ve `""` tamamen aynıdır. 

Peki `string.Intern` ü ne gibi durumlarda kullanmak gerekir. Eğer kodda geçmeyen bir metin ifadeyi tekrar tekrar kullanmanız gerekiyorsa, örneğin dil dosyasından yaptığınız okumalar varsa veya metin dosyalarında karşılaştırma, arama gibi işlemler yapıyorsanız işinize yarayacaktır. 

> Eğer bir _string_ boş mu diye bakmak istiyorsanız metnin _null_ olmadığından eminseniz en hızlı yol `.Length == 0` şeklinde kontrol etmektir. Emin değilseniz `string.IsNullOrEmpty` metodunu kullanabilirsiniz.
