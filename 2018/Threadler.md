---
Title: Thread'lere Kısa Bakış
PublishDate: 27/11/2018
IsActive: True
IsListed: True
Tags: C#
---

Threadler ya da Türkçe tabir ile "iş parçacıkları" bir uygulamanın (process) iş yapan birimleridir. Bir uygulamada en az bir adet thread olması gerekir. Bu mecburi thread genellikle "main thread" olarak isimlendirilir ve bu thread sonlandığında normal şartlarda uygulamanın sonlaması beklenir (tam burada foreground ve background thread kavramlarını not alın, bu yazıdan sonra araştrın). Threadler işletim sistemi tarafından önceliklerine göre belirli bir süre çalıştırılıp, dondurulurlar. Her bir thread için ayrılan süre ise "time slice" olarak isimlendiriliyor. Bu işlem çok hızlı gerçekleştiği için biz birçok uygulamanın aslında aynı anda iş yaptığını düşünürüz. Fakat aynı anda yapılan iş sayısı elimizdeki işlemci çekirdekleri sayısı kadardır.

Detaylara inmeden yazının Windows işletim sistemi için ve .net ortamı için hazırlandığını hatırlatayım, diğer işletim sistemlerinde ve ortamlarda süreçler daha farklı olabilir. Hemen yazıya devam edelim. Eğer görev yöneticisini açıp, mevcut thread sayısına bakarsanız binlerce threadin koştuğunu göreceksiniz. Bunların eş zamanlı gibi çalışmasını sağlayan "öncelik tabanlı round-robin" algoritmasıdır. Her thread için sırayla belirli bir süre ayrılır bu süre dolarsa veya threadin yapacak işi yoksa sıradaki thread çalıştırılır. Bir diğer kısıtta eğer bir thread çalışırken daha yüksek öncelikli bir process veya thread iş için hazırsa mevcut çalışan thread beklemeden hemen dondurulur ve öncelikli thread çalışmaya devam eder (_context-switch_). 

Biz .net tarafına doğru hızlıca dönelim. Main thread aynı zamanda işletim sistemi ile haberleşmeyi sağladığı için eğer bu arkadaş meşgul edilirse, işletim sisteminin emirlerini işlemekte sıkıntı çekebilir. Bunu denemek için bir button arkasında sonsuz döngü oluşturabilirsiniz. Düğmeye tıklanıldığında uygulama üzerindeki diğer nesneleri kullanamadığınızı ve uygulamayı hareket bile ettiremediğinizi göreceksiniz. Bir süre sonra işletim sistemi uygulama komutları işlemediği için uygulamanın "yanıt vermediği" yönünde sizi bilgilendirecektir.

C# tarafında yeni bir `thread` oluşturmak basittir. Yeni bir thread oluşturduğumuzda yapıcı metoda parametresiz almayan bir `ThreadStart` ve tek bir object parametresi olan `ParameterizedThreadStart` delegelerinden bir argüman vermeniz beklenir. Opsiyonel olarak da stack boyutu belirtebilirsiniz.

```csharp
static void Main()
{
	var thread = new Thread(YaziYaz);
	thread.Start();
	
	Console.ReadKey();
}

static void YaziYaz()
{
	Console.WriteLine("Merhaba");
}
```

Yeni bir thread oluştur diyoruz ve çalıştırmasını istediğimiz giriş metodunu veriyoruz. Yine bu thread e parametre vermek istersek metodumuz `object` türünden bir parametre almalı.

```csharp
static void Main()
{
	var thread = new Thread(YaziYaz);
	thread.Start("Merhabalar");
	
	Console.ReadKey();
}

static void YaziYaz(object yazi)
{
	Console.WriteLine(yazi);
}
```
 
Tek çekirdek varken birden fazla thread kullanımında genellikle amacımız uygulamamızın akıcı olmasıdır ve açıkçası kodlama kısmında pek problem yaşamayız. Fakat günümüzde tek çekirdekli işlemci arasanız bulamayabilirsiniz. Birden fazla çekirdek olan durumda uygulamamız neden tek bir çekirdeğe hapsolsun ki? Word örneğinde yazım denetimi ben yazarken dahi çalışacaktır. Asıl önemli olan veri işlemede bize kazandırdığı performans artışıdır. Örneğin, bir sayı dizindeki tüm sayıların üç katını alacak bir döngümüz olsun.

```csharp
var dizi = new[] {0,1,2,3,4,5,6,7,8,9,10,11};
for (int i = 0; i < dizi.Length; i++)
{
	dizi[i] *= 3;
}
 
```

Bu kodda yer alan çarpma işlemi için 1 tur gerektiğini düşünelim, bu durumda işlem 12 işlemci döngüsünde gerçekleşecektir. İki çekirdekli bir işlemcimiz olduğu durumda eğer çarpma işlemlerini üleştirirsek bu durumda 12 / 2 = 6 döngüde aynı işlemi gerçekleştirmiş olabiliriz. Günümüzde 4 çekirdek kullanımı neredeyse standart olduğuna göre, neden CPUnun kalan 3/4nü kullanmayalım? Bunu gerçekleştirmek adına dizinin farklı bölgelerinden başlayan threadler ile döngüler kurabiliriz. Haydi yapalım...

```csharp
var dizi = new[] {0,1,2,3,4,5,6,7,8,9,10,11};

void islem(object state){

	var index = (int)state;
	for (int i = 0; i < 3; i++)
	{
		dizi[i + index] *= 3;
	}
}

var t0 = new Thread(islem);
var t1 = new Thread(islem);
var t2 = new Thread(islem);
var t3 = new Thread(islem);

t0.Start(0);
t1.Start(3);
t2.Start(6);
t3.Start(9);

t0.Join();
t1.Join();
t2.Join();
t3.Join();
```

Epey uzun bir kod oldu değil mi? Merak etmeyin, bu kodlar güncel C# sürümlerinde oldukça kısa yazılabilmektedir. Ama önce ne yaptığımızdan bahsedelim. `islem` adında bir metodumuz var bu metot `object` türünden bir parametre alıyor. Bunu ister benim gibi inline metot olarak tanımlarsınız ister, normal metot olarak hiç fark etmez. Sonra 4 adet thread tanımlıyorum ve her birinin başlangıç metodu `islem` adını verdiğim metot. Burda ufak bir soru sormak isterim. Peki, bizim oluşturduğumuz normal programlardaki main thread'in başlangıç metodu ne? Cevap aslında soruda gizli. Eğer siz bunu özellikle değiştirmezseniz, `Program.cs` dosyası içinde göreceğiniz `Main()` metodu başlangıç metodumuzdur diyerek kodun açıklamasına devam edeyim. Peşinden 4 adet `Start` metodu başlatıyoruz. Her birine de thread'e aktarmak için bir parametre bilgisi geçiyoruz. Bu dizinin kaçıncı elemanından itibaren iş yapacağı oldu bu senaryoda, siz her türden nesneyi buraya verebilirsiniz. Arkasından da 4 adet `join` metodu geldiğini göreceksiniz. Bu metot çalışan bir thread bitene kadar beklenmesini sağlar. Hepsini alt alta yazdığımız için tümünün bitmesini bekliyor oluyoruz. Eğer beklemeden ekrana bu dizinin içeriğini basarsak tüm sayılar için işlem yapılmadığını görürüz.

Yukarıda yazdığımız kodu TPL (Task Parallel Library) sayesinde aşağıdaki gibi çok daha kısa yazılabileceğini ileriki yazılarda anlatıyor olacağım. Aşağıdaki kodun birebir karşılık olmadığını ve thread sayısına kendisinin karar verdiğini not edeyim.

```csharp
var dizi = new[] { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11 };
Parallel.For(0, dizi.Length, i =>
{
  dizi[i] *= 3;
});
```

Peki, bu işlemi 4 farklı threade böldüm. Kodum 4 kat hızlandı mı? Cevap vereyim, yukarıdaki kod için yaptığımız işlem yavaşladı. Sebebine gelince, bu kod zaten tek thread de kendisine ayrılan süre içerisinde bitebilecekken biz bunu 4 parçaya böldük. Aslında yapılan iş sayısını arttırmış olduk. Ama, dizinin her bir elemanı için işletilen kod daha zorlayıcı bir kod olaydı o zaman çok ciddi performans artışları sağlayabilirdik.

Aynı anda birçok iş yapmak güzel şeyler ama threadlerin kötü yüzleri de var. Bunlara ileride değinmek üzere...
