---
Title: SSS001 Rastgele Sayılar
PublishDate: 28/06/2017
IsActive: True
Tags: C#, Random
---


Forumlarda sürekli sorulan sorular bölümümün ilk konusunu için bu konuyu seçtim. Rastgele sayı üretmek C# tarafında sürekli yanlış bilgiler içermektedir. Bu sebeple bu konu ile ilgili özet bilgilerden bahsedeceğim.

####SORU : "Sürekli aynı sayı üretiliyor"
Kısaca nedeni : Seed kavramı.

Günümüzde bilgisayar dediğimiz cihaz teknik olarak "rastgele" bir sayı üretebilme yeteneğine sahip değildir. Şöyle düşünelim, hazır bir Random sınıfı olmasaydı nasıl bir kodla rastgele gibi görünen sayılar üretebilirdik. Bunun en pratik cevabı zaten hali hazırda rastgele gibi duran sayıları önceden hazırlayıp onların arasından kullanıcıya vermek olurdu. Fakat bu durumda da sayılar hep aynı sırayla aynı şekilde geliyor olurdu. Bunu engellemek için yine yapabileceğimiz en basit yöntem tüm bilgisayarlarda sürekli olarak değişen zaman bilgisini kullanmak olacaktır. .net de bu kurduğumuz yapıya benzer olarak rastgele sayı üretecek fonksiyonunu sürekli olarak zaman bilgisi argümanı ile çalıştırmaktadır. _(.net içerisinde kriptografik rastgele sayı üreticileride bulunmaktadır)_

Hatalı kod:

```csharp
for (int i = 0; i < 100; i++)
{
    var random = new Random();
    Console.WriteLine(random.Next());
}
```

Bu kodun çıktısı:

```
4565
4565
4565
4565
4565
124545
124545
124545
124545
124545
124545
…
```

Doğru kod:

```csharp
var random = new Random();
for (int i = 0; i < 100; i++)
{
    Console.WriteLine(random.Next());
}
```
Bu kodun çıktısı:

```
1995015542
557051673
704335828
1146562470
347634715
1782604740
1445944305
331395076
1365915347
1877460330
227513293
802360858
…
```

Burada ne değişti? _Random_ sınıfının yapıcı (constructor) methodu aslında _int_ tipinde bir _seed_ bilgisi almaktadır.  Bu bilgi verilmediğinde ise varsayılan olarak sistem zamanı kullanılmaktadır.  _For_ döngüsü ise oldukça hızlı çalıştığından  bu zaman bilgisinin değişmeden sürekli aynı _seed_ bilgisi ile sayı üretmektedir. Örneğin "doğru" olarak işaretlediğim kodda yapıcı methodu şu şekilde değiştirin :  `new Random(1111)` bu durumda döngüyü ne zaman çalıştırırsanız çalıştırın aynı 100 sayıyı aynı sırayla elde edeceksiniz. 

> **İpucu**: Oyun geliştiriyorsanız ve oyununuz rastgele olaylar üzerine kurulu ise oyuncuların kaydet/yükle şeklinde hile yapmasını istemiyorsanız her oyunu yeni bir _seed_ değeri ile başlatabilirsiniz. Bu durumda tüm rastgele olaylar oyun kayıttan yüklense bile aynı sırada yaşanacaktır.

####SORU : "Tekrarsız rastgele sayı üretmek istiyorum"

Bu soruda oldukça fazla soruluyor. Ama aslında beni sıkan bu soruya verilen yanıtlar oluyor. Çünkü genellikle şu yanıtı görürsünüz : "bir dizi oluştur, bu dizide varsa tekrar bak", evet bu görece doğrudur. Fakat, üretmeniz gereken sayı miktarı arttığında bu yöntem performans olarak oldukça kötü olacaktır. Biraz daha iyi kod yazmak gerekirse:

```csharp
//Kötü kod
public static IList<int> TekrarsizSayiUretKotu(int uretilecekAdet, int baslangic, int bitis)
{
    var sayilar = new List<int>();
    var random = new Random();
    do
    {
        var uretilen = random.Next(baslangic, bitis);
        if (sayilar.Contains(uretilen))
        {
            continue;
        }
        sayilar.Add(uretilen);
    } while (sayilar.Count < uretilecekAdet);
    return sayilar.ToList();
}

// Daha iyi kod
public static
 IList<int> TekrarsizSayiUret(int uretilecekAdet, int baslangic, int bitis)
{
    var sayilar = new HashSet<int>();
    var random = new Random();
    do
    {
        sayilar.Add(random.Next(baslangic, bitis));
    } while (sayilar.Count < uretilecekAdet);
    return sayilar.ToList();
}
```

Her iki metodu basitçe 
```csharp
void Main()
{
    var sonuc = TekrarsizSayiUret(100000, 1, 100000001);
}
```

şeklinde kronometre ile test ettiğimde. _HashSet_ kullanımında 100.000 sayı üretimi için 0,006saniye gerekirken bu _List_ kullanımında 7,68saniye sürdü. Çok daha hızlı bir yöntem bulursanız bana yazarsanız sevinirim.

Buradaki fark _HashSet_'in kullandığı kovalama yönteminden kaynaklanıyor. Hangi koleksiyon tipinin hangi senaryolarda kullanılması gerektiği de başka bir yazının konusu olsun. 

> **Not :**  _HashSet_ gibi _ISet_ uygulanmış koleksiyon sınıflarında aynı eleman iki defa bulunamaz. Aynı eleman ikinci defa eklenmeye çalışıldığında _Add_ metodu geriye _false_ değeri döner.


