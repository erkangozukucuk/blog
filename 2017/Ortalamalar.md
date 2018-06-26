---
Title: Veriyi Anlamak - Ortalamalar
PublishDate: 29/08/2017
IsActive: True
Tags: C#, Veri Madenciliği
---

Bir yığın veriye bakıp karar vermek için gereken en temel bilgilerden birisi _ortalama_dır. Bir çok veri analiz yöntemi, makine öğrenme algoritmaları içerisinde ortalama hesabı barındırır. Günlük hayatta en çok ortanca ve aritmetik ortalama kullanılsa da iş veriyi anlamak olunca bir kaç farklı tür daha bulunmaktadır. Bu yazıda sık kullanılan ortalama türlerini nerede nasıl kullanmak gerektiğinden ve C# karşılıklarından bahsedeceğim. 

### Ortanca

Ortanca (Medyan,_Median_) bir dizinin tam ortasında bulunan elemana denilmektedir. Dizi çift sayıda eleman barındırıyorsa ortancanın seçimi analiz yapan kişinin tercihine kalmaktadır. Analizi yapan kişi dizide çift sayıda eleman olması durumunda şu yöntemleri tercih edebilir:   

* Ortadaki iki eleman birden kabul edilmesi
* Küçük olanın tercih edilmesi
* Büyük olanın tercih edilmesi 
* Ortadaki iki elemanın aritmetik ortalamasının kabul edilmesi

Ortanca aslında yazının konusunda olduğu gibi bir ortalama değildir. Fakat fikir vermesi açısından ve özellikle sıralı dizilerde hesaplaması kolay olduğundan demetleme ve sınıflama algoritmalarında uzaklık hesabında kullanılmaktadır.

Ortanca için C# da hazır bir metot bulunmamaktadır. En temel medyan bulma yönteminde dizi önce sıralanır ve eleman sayısına göre ortadaki indisteki eleman(lar) alınır. Örnekleyelim:

```csharp
public static double Ortanca(IEnumerable<double> sayilar)
{
    var siralanmisSeri = sayilar.OrderBy(s => s).ToList();
    var ortaSira = (siralanmisSeri.Count - 1D) / 2D;
    return (siralanmisSeri[(int)ortaSira] + siralanmisSeri[(int)(ortaSira +0.5D)]) / 2D;
}
```

> Bu algoritma sıralama algoritmasına bağımlı olarak O(n * log n) karmaşıklığında olacaktır. Biraz araştırırsanız O(n) seviyesinde medyan bulan algoritmalar bulabilirsiniz. Bu yazında hesaplaması ve anlaması kolay olduğu için ben bu yöntemi tercih ettim.



### Aritmetik Ortalama

En çok bilinen ve anlaması en kolay ortalama türüdür. Tüm sayıları toplayıp eleman sayısına bölmemiz sonucunda bulunur. 

$$\overline{X} = \frac{\sum{X_i}}{n}$$



Duruma göre elemanlara ağırlık verilerek ağırlıklı aritmetik ortalama bulunabilir. Ağırlıklı ortalamada ilgili eleman ağırlığı kadar fazla geçiyormuş gibi hesaplanır. Bu yöntemi kredili derslerin ortalamasının alınmasından hatırlayacaksınızdır.
C# da Linq kütüphanesi ile `Average` methodu aritmetik ortalamayı bulmak için kullanılır. Ağırlıklı ortalamada ise ağırlıklı çarpımların toplamını ağırlıklar toplamına bölmek yeterlidir.
```csharp
internal static class Program
{
    private static void Main(string[] args)
    {
        var notlar = new[] {new DersNotu("Türkçe", 5, 2), new DersNotu("Matematik", 3, 3)};
        var ortalama = notlar.Sum(x => x.Agirlik * x.Not) / notlar.Sum(x => x.Agirlik);
        Console.WriteLine(ortalama);
    }

    private struct DersNotu
    {
        public DersNotu(string ders, decimal not, int agirlik)
        {
            Ders = ders;
            Not = not;
            Agirlik = agirlik;
        }

        public string Ders { get; }
        public decimal Not { get; }
        public decimal Agirlik { get; }
    }
}
```
Bu örnek de O(2n) karmaşıklığında sonuç elde edilmiştir. Hesaplama şu şekilde yapılırsa daha iyi olacaktır:

```csharp
 private static void Main(string[] args)
 {
     var notlar = new[] { new DersNotu("Türkçe", 5, 2), new DersNotu("Matematik", 3, 3) };

     var agirlikliToplam = 0M;
     var agirliklarToplami = 0M;

     foreach (var dersnot in notlar)
     {
         agirlikliToplam += dersnot.Agirlik * dersnot.Not;
         agirliklarToplami += dersnot.Agirlik;
     }

     var ortalama = agirlikliToplam / agirliklarToplami;
     Console.WriteLine(ortalama);
 }
```



### Geometrik Ortalama

Geometrik ortalama, aritmetik kadar yaygın kullanılmasa da bir çok özel durumda kullanılmaktadır. Bu durumlar:

* Serinin elemanları oransal bir değişiklik gösterdiğinde kullanılmaktadır.
* Sapan elemanların etkisini azaltma amacıyla kullanılmaktadır.
* n boyutlu şeklin büyüklüğünün (2 boyut için alan, 3 boyut için hacim gibi) aynı kalarak her bir boyutunun eşit uzunluğa sahip olduğu durumu bulmak için kullanılmaktadır. Bu durum daha açıklayıcı şekilde yazının devamında bulunmaktadır.
* Özellikleri farklı ölçüm birimi ile hesaplanmış bilgileri normalizasyon yapmadan basitçe karşılaştırmakta kullanılmaktadır.

Oransal değişiklik konusunu ile başlayalım. Basit bir şekilde elimizdeki dizi `2, 4, 8` şeklinde sürekli ikiye katlanarak artıyor olsun. Bu dizinin ortalamasına ne demeliyiz? Eğer aritmetik ortalama alırsak sonuç `4.66` çıkacaktır. Fakat oransal artışa göre baktığımızda ortalama `4` olmalıdır. Beş oranında artan  `1, 5, 25` durumuna da bakalım. Oransal artışın orta noktası `5` dir. Ama sayıların ortalamasına baktığımızda sonuç `10.33` çıkacaktır. Oran arttırdıkça aritmetik ortalama ve geometrik ortalama arasındaki farkın arttığına dikkat ediniz. 

Diğer kullanım durumlarına değinmeden formülü vereyim. Bu iş için kullanılan iki formül var.

$$\overline{X}_{geom} = (\prod_{i=1}^{N}{X_i})^{\frac{1}{N}}$$



ve

$$ \overline{X}_{geom} =\left({\frac{ \sum{log{X_i}}}{n}}\right)^e$$

Bunlardan ilki en sık kullandığımız yöntem. Her iki yöntem için C# kodu yazacak olsak şöyle olurdu :

```csharp
public static double GeometrikOrtalama(IList<double> dizi)
{
	return Math.Pow(dizi.Aggregate((x, y) => x * y), 1D / dizi.Count);
}

public static double GeometrikOrtalama2(IList<double> dizi)
{
	return Math.Pow(2, dizi.Sum(x => Math.Log(x,2)) / dizi.Count); //2 yerine herhangi bir sayı yazılabilir
}
```
> Linq de yer alan `Aggregate` metodu iki argüman verir.Başlangıçta ilk iki eleman olmak üzere birisi mevcut elemanı diğeri bir önceki hesaplamadan dönen değeri verir. Yani  `2,4,6` dizisi için `x*y` durumunda sırasıyla {2,4}, {8,16} şeklinde 2 tur dönecektir. Ve sonuç  8 * 16 = 128 olarak bulunacaktır. Bu metod diğer programlama dillerinde _reduce_ olarak geçmektedir.

Sapan (outlier) veri olduğunda geometrik ortalama ile bu verilerin etkisi azaltılabilir. Fakat bu amaçla genellikle birazdan değineceğim harmonik ortalama kullanılmaktadır. Yine de harmonik ortalamanın hesaplanamadığı durumlarda bu amaçla kullanılabilir.

İşin gerçekten geometri ile ilgili olan kullanımına gelirsek, 3 e 27 lik bir dikdörtgen düşünün. Bu şeklin alanı 27dir. Peki alanı 27 olan karenin bir kenar boyutu kaçtır? Geometrik ortalama yaptığımızda sonuç 9 çıkacaktır. Bunu her boyut için uygulamak mümkündür.

Normalizasyon tüm özelliklerin aynı birimden ifade edilmesidir. Fakat pratik kullanımda, karşılaştırmalarda geometrik ortalama kullanılabilir. Örneğin sınırlı miktarda paranız ve bir oyun alacaksınız. İki farklı oyun yorum sitesi buldunuz, sitelerden birisi 5 üzerinden diğeri 100 üzerinden puanlama yapıyor olsun. A oyunu için (7 - 85); B oyunu için (6 - 90) gibi puanlama yapılmış olsun. Eğer aritmetik ortalamalarını karşılaştırmak isterseniz sonuç A oyunu için 46, B için 48 çıkacaktır. Fakat geometrik ortalama da sonuçlar A için 24, B için 23 olduğundan tercih edilmesi gereken A olacaktır.  Eğer normalizasyon yapsaydık sonuç 0.8 A, 0.75 B çıkacaktı ve sonuç değişmeyecekti. 



### Harmonik Ortalama

Eleman sayısının, elemanların çarpma işlemine göre tersinin toplamına bölümüne denir. Harmonik ortalamada dizinin elemanları bir bütünün eleman sayısı kadar eşit bölünmüş parçalarındaki ortalamalardır. Amacımızda genel ortalamayı bulmaktır. 
Genellikle zamansal ortalamalar alındığında kullanılır. En tipik örnekler sürat ile konularda verilir. 
A,B şehirleri arasındaki mesafenin yarısı 50km/s ile kalan yarısı 100km/s ile katedilmiş ise ortalama sürat ne kadardır? 
Mesafeyi bilmiyoruz ama sürelerin eşit olduğunu biliyoruz. Bu durumda ortalamaları alırken mesafenin ne olduğu sonucu değiştirmeyecektir. 200 diyelim. Bu durumda ilk yarısını 100 / 50 = 2 saat , kalan yarısını 100 / 100 = 1 saat de gitmiştir. Toplam süre 3 saat saat sürmüştür. 200 / 3 = 66 km/s ortalama sürat olarak bulunur. Ne alırsak değişmeyecek demiştik. Bu  durumda 200 yerine eleman sayısı olan 2 yi alalım. (1/50 + 1/100) saat sürecektir. 2 /(1/50 + 1/100) de bize 66 verecektir. Bu durumu formülleştirirsek:

$$\overline{X}_{harmonik} =\frac{n}{\sum{\frac{1}{X_i}}}$$



elde ederiz. Bunu C# ile yazmak istersek :

```csharp
public static double HarmonikOrtalama(IList<double> dizi)
{
	return (double)dizi.Count / (dizi.Sum(x=> 1D /x));
}
```

Şeklinde yazabiliriz. Geometrik ortalamada sapan elemanlar (outlier) dan bahsetmiştim kendileri veri kümesinin içerisinde diğer elemanlardan uyumsuz değerlere sahip olanlardır. Bu değerler doğruda olabilir, yanlış da fakat her türlü bilginin yorumlanmasına etkileri olacaktır. Özellikle örneklem boyutu azaldıkça bu etki doğrudan sonucu etkileyecektir. Yaşlara göre gelir dağılımı alınıyor olsun ve bunun için 1000 kişi seçilmiş olsun. Eğer bu 1000 kişinin içerisine 20 yaşında bir milyarder denk gelirse yapılacak yorumlamalarda problem çıkacağı açıktır. 20 yaşa sahip 10 kişi ve gelirleri şu şekilde olsaydı : `{ 2500, 3000, 2750, 5000, 1200, 3000, 1900, 1500, 150000 }` bu işlem için aritmetik ortalama bize `18983` sonucunu verecektir. Bu durumda 20 yaş için ortalama kazanç 18bindir demek pek mantıklı olmayacaktır. Fakat eğer işlemi geometrik ortalama ile yapacak olsaydık sonuç `3783` ile biraz daha mantıklı hale gelmiş olacaktı. Genellikle kullanılan harmonik ortalama ise bize `2456` değerini verecektir ki bu eleman listede olmasa aritmetik ortalama `2606` olacağından oldukça başarılı bir sonuç elde etmiş oluruz. Yine de gerçek senaryolarda mümkünse sapan değerler belirli bir sapmaya göre tespit edilip hesaplamaya alınmazlar.

Bölme işleminden dolayı harmonik ortalama 0 ve küçük olan değerlerde kullanılamaz. Yine sapan değer düşük değerlikli ise ortalamayı daha fazla etkilemektedir. Örneğin bir önceki değerler `{ 2, 3000, 2750, 5000, 1200, 3000, 1900, 1500, 1500 }` şeklinde olmuş olsunlar. Bir katılımcının geliri yanlışlıkla üç sıfır eksik yazılıp 2 olarak kayda geçtiğinde ortalamalar : `AO:2205.78 GO:1026.88 HO:17.86` değerlerini alır ki sonuç oldukça etkilenmiştir.



### Kareli Ortalama

Kareli ortalama (_root-mean-square_ , _rms_ veya _quadratic mean_) her elemanın karelerinin aritmetik ortalamasının karekökü olarak hesaplanır. 

$$\overline{X}_{kareli} =\sqrt{ \frac{\sum{{{X_i}^2}}}{n}}$$

Kareli ortalama bir çeşit uzaklıkların ortalamasını almayı sağlar. İstatistik de her bir elemanın ortalamadan uzaklığının kareli 

ortalamasının alınması standart sapmayı bulmayı sağlar ve istatistik standart sapmayı oldukça fazla kullandığı için bilinmesinde ben fayda görüyorum. 



### Kuvvet Ortalaması (_Hölder mean_)

Tek bir formülle tüm pisagorik ortalamaların (aritmetik,geometrik,harmonik,kareli) bulunmasını sağlar:

$${\bar {x}}(m)=\left({\frac {1}{n}}\cdot \sum _{i=1}^{n}{x_{i}^{m}}\right)^{\tfrac {1}{m}}$$

```csharp
public enum OrtalamaTuru
{
	Harmonik = -1,
	Geometrik = 0,
	Aritmetik = 1,
	Kareli = 2
}

public static double Ortalama(IList<double> dizi, OrtalamaTuru tur)
{
	return Math.Pow((1D / 
		dizi.Count * dizi.Sum(x => Math.Pow(x, (int)tur))), 1D / (double)tur);
}
```



### Sonuç

Konuda bahsi geçenler dışında daha bir çok ortalama türü bulunmaktadır. Fakat pisagorik ortalamalar genellikle yeterli olmaktadır. 
