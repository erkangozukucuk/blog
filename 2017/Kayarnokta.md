---
Title:Kayar Nokta Kavramı
PublishDate: 18/09/2017
IsActive: True
Tags: C#, Veri Madenciliği
---

C# dahil olmak üzere çoğu programlama dilinde ondalıklı sayıları göstermek için bir çok farklı tür bulunur. Peki ama neden? Nerede `decimal,double,float` hangisini hangi senaryolarda kullanmak gerekir? Uzun uzun açıklamak istemediğimiz durumlarda şu cevabı veririz: “eğer işlem sonucunda çıkan sayının olması gereken sonuca çok yakın olmasında sakınca yoksa `double,float`, aksi durumda `decimal` kullan”. Bu cevabı bile ilk defa duyanlar vardır diye tahmin ediyorum. Biraz örnek yapıp neden bu sonuçları aldığımıza bakalım. Alıştırmaya `double` ile başlayalım.

```csharp
 double a = 0.4;
 double b = 0.3;
 double c = 0.7;
 Console.WriteLine((a + b) == c);
```

Bu kodun sonucu nedir? Cevap `true` olacak. Peki aynı örnekte sadece sayıları değiştirip şu hale getiriyorum bakalım sonuç değişecek mi:

```csharp
double a = 0.1;
double b = 0.7;
double c = 0.8;

Console.WriteLine((a + b) == c);
```

Aynı mantıkta yazdığım kod bu sefer `false` sonucunu dönmeye başladı. Aynı örneği bu sefer türleri değiştirerek yapayım :

```csharp
decimal a = 0.1M;
decimal b = 0.7M;
decimal c = 0.8M;

Console.WriteLine((a + b) == c);
```
Şimdi sonuç olması gerektiği gibi `true` halini aldı. 

>Not: C# da sayısal değişkenlere atama yapılırken. Tam sayılar için varsayılan tür `int` ondalıklı sayılar için `double` olmaktadır. `x = 1` atamasında 1 sayısının türü `int` dir. Benzer şekilde `x = 1.2` atamasında 1.2 sayısı `double` türündendir. Bunu değiştirmek istersek sonuna türe uygun bir harf eklemek gerekir. `x = 1L` durumunda 1 artık `long` türündendir. `x = 1.2M` durumunda ise `decimal` türündendir. `D` harfi `double`için kullanılmaktadır.

İşler biraz daha garipleşsin. Şu koda bakın:

```csharp
double toplam = 0;

for (int i = 0; i < 3; i++)
{
	toplam += 1D / 3D;
}

Console.WriteLine(toplam);
```
Basit bir `for` döngüsü 3 defa dönüyor. Her döngüde 1i 3e bölüp toplam değişkenine ekliyor. Kullanılan tüm sayılarda `double` türünde. Yukardaki notta açıklamıştım `3D` sayının `double` olduğunu söylüyor.

Peki sonuç nedir? Çalıştıracağınızda alacağınız sonuç `1` olacaktır. Şimdi ise türleri değiştirelim:

```csharp
decimal toplam = 0;

for (int i = 0; i < 3; i++)
{
	toplam += 1M / 3M;
}

Console.WriteLine(toplam);
```


Burada ise alacağımız sonuç `0.9999999999999999999999999999` olacaktır. 

Buraya kadar olan örneklerden şu sonuç kesinlikle çıkacaktır: Kafamıza göre double veya decimal diyip geçmemiz hatalı sonuçlar üreten bir kod yazmamıza sebep olacaktır. İlk örneğimizi düşünün, bu toplama işlemi bir sayaç uygulamasında oluyor olsaydı ve sayaç 0.8 değerine ulaşınca duruyor olması gerekseydi. Asla bu sonuca ulaşamayacağı için sayaç çalışmaya devam edecek ve sonlanması gereken bir uygulama sonlanmayacaktır. Benzer şekilde oranların toplamı ile çalışan bir uygulamamız olsa toplamda eksik sonuç vereceğinden yine hatalı bir uygulama yazmış oluruz. 

Eğer `double` kullanıyorsanız `==` kullanmaktan kaçının. Bunun yerine büyüklük kontrolü veya aralık kontrolü yapın. Sayaç örneğimizde `x > 0.699` gibi bir kullanım daha doğru olacaktır. `Double,float` gibi ikili (binary) kayar nokta mantığındaki sayılar üzerinde işlem yapmak hızlıdır. Bu sebeple tam değerler gerekmeyen ama performans gereken uygulamalarda özellikle oyunlarda bu türler kullanılır. Bu değerlerin tam doğru olmaması da oyunlarda “z-fighting” problemine yol açmaktadır. 

### Sabit Noktalı Sayılar
Kayar nokta mantığına gelmeden önce bu ihtiyacın oluşma sebebini anlamak için sabit noktalı sayılara bakmalıyız. Bir oyunun gigabytelarca büyüklükte olmadığı zamanlarda, her bir bit çok değerliydi. Aynı zamanda işlemciler, bellekler günümüze göre oldukça yavaşlardı. İşte bu kıtlık durumunda olduğumuzu varsayın. Bir değişken için ayırabileceğimiz bellek boyutunun 8bit olduğunu varsayın.

> Ondalıklı sayılar için 8bit çok nadir kullanılır. Mantık değişmediği için basit olması açısından 8bit ile devam edeceğim.

```
: 8 : 7 : 6 : 5 : 4 : 3 : 2 : 1 : 0 :
: 0 : 0 : 0 : 0 : 0 : 0 : 0 : 0 : 0 :
```
Üst satır bit sırasını alt satırda değeri (0 veya 1)  temsil ediyor olsun. Bu durumda yazabileceğim en büyük sayı

```
: 8 : 7 : 6 : 5 : 4 : 3 : 2 : 1 : 1 :
: 1 : 1 : 1 : 1 : 1 : 1 : 1 : 1 : 1 :

128 +64 +32 + 16 + 8 + 4 + 2  + 1 = 255
0 - 255
```
Tabi eğer sayının pozitif mi negatif mi olduğunu da tutmak istiyorsam bir bit de oraya gideceğinden sonuç `-128 - 127` e dönüşecektir. Peki ben bir de ondalıklı sayıları tutmak isteyim. İşaret (_sign_) bitini düşünmeden 8 biti 5 bit tam sayıyı, 3 bit ondalıklı kısmı gösterecek şekilde kullanmak istersem. Bu durumda noktanın sağı 2 nin negatif kuvvetlerine dönüşür.
```
: 5 : 4 : 3 : 2 : 1 : 0 : -1: -2: -3:
: 1 : 1 : 1 : 1 : 1 : 1 : 1 : 1 : 1 :

16 + 8 + 4 + 2 + 1 + 1/2 + 1/4 + 1/8
0 - 16,875
```

### Kayar Nokta
Gerçek dünyada çoğu durumda sayı büyüdükçe sayının küçük kısımları önemsizleşir. Bir iş sonucunda elde edilecek 1000TL ile 1500TL arasındaki fark çok ciddi iken 1.000.000TL ile 1.000.500TL arasındaki 500TL genellikle önemsizdir. Kayar nokta mantığıda buna benzer. Sayı 0'a ne kadar yakınsa ondalıklı kısmın önemi o kadar artar. Peki bu nasıl oluyor? Basit bir mantıkla tam kısım 0 a yaklaştıkça noktayı koyacağımız kısım sola kaymış oluyor. Tam tersi şekilde sayı büyüdükçe ondalık kısmın hassasiyeti gittikçe düşüyor. 

Elimizdeki 8 bit aşağıdaki gibi ayıralım.

* 1 bit'i işaret için (___sign___)
* 4 bit'i kuvveti tutmak için (___exponent___)
* 3 bit'i sayıyı (kök) tutmak için (___mantissa / significand/fraction ___)

Peki neden böyle üç parçaya ayırdık. Buradaki amaç _bilimsel gösterim_i bitler üzerinde modellemek. Yani ben 500 sayısından bahsediyorsam bu 5 * 10² ya da bilimsel gösterimle `5e+2` olarak gösterilir. Şayet 0.05 den bahsediyor olsaydım bu durumda `5e-2` demem yeterli olacaktır. Böylece 510000000 gibi büyük bir sayıyı `5.1e+8` ya da `51e+7` gibi göstermem mümkün olur. Dikkat ederseniz 51 ve 7 sayılarını saklamam yeterli olmakta. Yine dikkat ederseniz kuvvet kısmı negatif olabiliyor, bu durumda aslında kuvvet (_exponent_) parçasının da ilk biti işaret biti olarak kullanılabilir (_veya birazdan bahsedeceğimiz bias mantığı kullanılabilir_). Bu sebeple fazladan biti köke değil de kuvvete vermiş olduk.

Bilgisayarlar ise ikilik sistemde çalıştıkları için aynı mantığı bu sisteme taşımamız gerekir. Bu durumda 7.5 sayısını göstermek istiyor olayım. Bunu kayar nokta mantığı olmaksızın göstermek isteseydim. 0111,1 şeklinde (4 + 2 + 1 + 0.5) gösterebilirdim. Virgülün sağında bir tane 1 oluncaya kaydırma yapıyorum. Bu durumda virgülü 2 defa sola çekmem gerektiği için 1,111 * 2² demem doğru olacaktır (onluk sistemde düşünürsek ` 1 + (0.5) + (0.25) + (0.125) = 1.875  * 4 = 7.5` bu durumda  bitlerim `1  :  0 0 1 0 (2)  : 1 (0.5) 1 (0.25) 1 (0.125)` bu da   4 * 1.875 = 7.5 sonucunu verecektir (**mantissa kısmına sadece ondalıklı kısmı yazıyoruz**)

 #### IEEE 754 Standardı

Bir önceki yöntemimiz konuyu anlamak içindi fakat gerçek kullanımda IEEE 754 gibi standartlar bulunmaktadır. Bu standarda göre bit sayısına göre isimlendirme ve parça kullanımları aşağıdaki gibidir:

```
Half : SEEEEEMM MMMMMMMM   bias : 15
Single (C# float): SEEEEEEE EMMMMMMM MMMMMMMM MMMMMMMM bias: 127
Double (C# double): SEEEEEEE EEEEMMMM MMMMMMMM MMMMMMMM MMMMMMMM MMMMMMMM MMMMMMMM MMMMMMMM bias: 1023
```
IEEE standardında karşımıza **bias** adından yeni bir değer çıkıyor. Kuvvet kısmına önceden belirlenmiş bir sayı (genellikle yarısı) belirleniyor ve bu sayının üstü pozitif olarak değerlendiriliyor.
Hesap sırasında bias değerine kuvvet toplanıyor. Böylece işaret biti kullanılmamış oluyor ve bazı özel durumların gösterilmesi sağlanıyor. Özel durumlar ise:

* Eğer kuvvet bitlerinin hepsi 0 ise sayı 0 dır.
* Eğer kuvvet bitlerinin hepsi 1 ve kök bitlerinin hepsi 0 ise sonuç sonsuzu(_infinity_) belirtir. İşaret biti ise sonsuzluğun yönünü.
* Eğer tüm kuvvet bitleri 1 ve kök bitlerinin hepsi 0 değilse bu durumda sonuç bir sayı değildir (_NaN_)

Bu yeni bilgiler ışığında biraz alıştırma yapalım. Az önce tam olarak göstermeyi başaramadığımız 7.5 sayısı ile başlayalım. 
1,111 * 2² i zaten bulmuştuk.

C# float tipinde oluşturmak isteyelim. Bu durumda 16bit'e ihtiyacımız olacak.

```
SEEEEEEE EMMMMMMM MMMMMMMM MMMMMMMM
```
IEEE 754'e göre işaret biti 0 iken pozitif, 1 iken negatif sayıları belirtir. Bu durumda 0 diyeceğiz:
```
0EEEEEEE EMMMMMMM MMMMMMMM MMMMMMMM
```
Üs kısmında bias değeri 127 ile kuvvet değerimiz 2yi toplamamız gerekiyor.

‭01111111‬ + 10 = 10000001

```
01000000 1MMMMMMM MMMMMMMM MMMMMMMM
```

Elimizdeki sayı `1.111` haline gelmişti, bunun ondalıklı kısmını alıp kalan bitleri 0 ile dolduruyoruz:

```
01000000 11110000 00000000 00000000
```

Evet artık elimizde 7.5 sayısının tek duyarlı (single) kayar nokta hali var. Bunu c# ile gösterelim :

```csharp
float a = 7.5f;
var bytelar = new BitArray(BitConverter.GetBytes(a).ToArray());
Console.WriteLine(string.Concat(bytelar.Cast<bool>().Select(x=>x?"1":"0").Reverse()));
// 01000000111100000000000000000000

```
>Neden ters çevirdiğimi merak ediyorsanız _little ve big endian_ kavramlarını araştırabilirsiniz.

## Decimal Floating Point Kavramı ve C#
Para ile ilgili değerlerde ikilik sisteme göre olan bir kayar nokta aritmetiği sıkıntılı olduğu için aynı aritmetiğin onluk sistemdeki karşılığı da yapılmıştır. IEEE 754 tanımında decimal32,64… gibi tanımlar ve farklı encoding tipleri bulunmaktadır. Fakat konumuz C# olduğundan ben C#'ın yönteminden bahsedeceğim. C# 64bitlik bir onluk kayar nokta sistemi kullanır ve bitler aşağıdaki dizilimin tersidir (_big/little endian_)

```
SRRRRRRR
RRREEEEE
RRRRRRRR
RRRRRRRR
MMMMMMMM
MMMMMMMM
MMMMMMMM
MMMMMMMM
MMMMMMMM
MMMMMMMM
MMMMMMMM
MMMMMMMM
MMMMMMMM
MMMMMMMM
MMMMMMMM
MMMMMMMM
```
Mantissa kısmı 96bitlik bir tam sayıyı ve kuvvet (_exponent_) kısmı 5 bit olarak ve en fazla 28 değerine kadar kaç virgül sağa kaydırma yapılacağını tutmaktadır. Örneğin **7.5** sayısını tutmak isteyelim. Yine işaret bit i için 0 değerini pozitif sayıyı göstermek için kullanacağız. Kalan alanlar ise kullanılmamaktadır.

```
0RRRRRRR
RRREEEEE
RRRRRRRR
RRRRRRRR
MMMMMMMM
MMMMMMMM
MMMMMMMM
MMMMMMMM
MMMMMMMM
MMMMMMMM
MMMMMMMM
MMMMMMMM
MMMMMMMM
MMMMMMMM
MMMMMMMM
MMMMMMMM
```
Tam sayı kısmında virgülden kurtulmuş hali olan 75 sayısını (‭01001011‬) koyacağız ve kalan kısmı 0 ile dolduracağız.

```
0RRRRRRR
RRREEEEE
RRRRRRRR
RRRRRRRR
00000000
00000000
00000000
00000000
00000000
00000000
00000000
00000000
00000000
00000000
00000000
01001011‬
```
sağa doğru 1 tane kaydırma yaptığımız için kuvvet kısmına 1 yazacağız ve kalanı 0 ile tamamlayacağız:

```
00000000
00000001
00000000
00000000
00000000
00000000
00000000
00000000
00000000
00000000
00000000
00000000
00000000
00000000
00000000
01001011

```

Eveet 7.5 sayısını `decimal` türünden elde ettik. Bunu görmek için C# da bir kod yazalım :

```csharp
decimal a = 7.5M ;

var bits = decimal.GetBits(a);
var bytelar = new BitArray(bits.SelectMany(i=>BitConverter.GetBytes(i)).ToArray());

Console.WriteLine(string.Concat(bytelar.Cast<bool>().Select((x,i)=>(x?"1":"0") + (i%8 == 0? "\r\n" : "")).Reverse()));
```

## Sonuç
Fark ettiyseniz `decimal`türünde bir sayıyı birden fazla ifade etme şansı vardır. Gerçekten de eğer `7.5 ve 7.50` sayılarının bitlerine bakacak olursanız farklı olduklarını göreceksiniz. Fakat hesaplama anında bu sıkıntı oluşturmamaktadır. Yine de eğer yuvarlama (_rounding_) hataları sizin için önemli değilse hız için _double_ türünü kullanın.

Konunun daha net anlaşılması için konu başındaki sayıları (0.1;0.7…) her iki türde de bitlik yazılışlarını yazmaya çalışın. Bazı sayıları yazamadığınızı göreceksiniz. 

Sağlıcakla

