---
Title: Farklı Türden Dosyaların Byte Dizisine Çevrilmesi
PublishDate: 11/12/2018
IsActive: True
Tags: C#
---

Şifreleme, kodlama, veri transferi, gibi birçok yöntemin kullanılmasında elimizdeki verinin `byte` dizisi olarak dönüştürülmüş olması gerekmektedir. Bu yazıda bilindik türleri nasıl byte dizisi olarak ele alabileceğimizi özetleyeceğim.

## Metin değişkenler

`String` türündeki bir değişken byte dizisine çevrilirken dikkat edilmesi gereken en önemli husus doğru kodlama (encoding) türünün seçilmesi olacaktır. Bir C# string değişkeni UTF16 (unicode) olarak bellekte saklanır. Bu metni diziye çevirirken UTF32, 16 veya 8 olarak saklayabilirsiniz. UTF32 de tek bir karakter 32bit (4byte) yer kaplar. UTF 16 da 16 bit (2byte) yer kaplar. UTF 8 de ise durum biraz değişiktir. Karakter ASCII olarak ifade edilebiliyorsa 8bit (1byte) olarak yer kaplarken ifade edilemeyen durumlarda bu artmaktadır.

Örnek kodu inceleyin:

```csharp
var metin = "Klasik bir UTF16 c# metni,fıstıkçı şahap oğlu";
 
var dizi32 		= Encoding.UTF32.GetBytes(metin); 		//  180
var dizi16 		= Encoding.Unicode.GetBytes(metin); 	//	 90
var dizi8 		= Encoding.UTF8.GetBytes(metin);      	//   51
var diziascii 	= Encoding.ASCII.GetBytes(metin); 		// 	 45
```

Metin değişkeni farklı kodlayıcılar kullanılarak byte dizisi haline çevrilmiştir. Yorum satırları ise ilgili byte dizisinin boyutunu belirtmektedir. 45 karakterlik bir metin dosyamız için UTF32 4 x 45 = 180 byte yer kaplarken ASCII 1 x 45 = 45byte yer kaplamaktadır. UTF-8 ise 51 byte yer kaplamaktadır. Eğer yazının sonuna eklediğim Türkçe'ye özgü karakterleri kullanmasaydım burada da sayı eşit olacaktı. Peki gelelim hangisini kullanacağımıza. Aslında cevap yok. Duruma göre kullanacaksınız. Şifreleme ve benzeri işlemlerde kullanacaksanız UTF-8 ve UTF-16 kullanmanız iyi olabilir. Karakter sayısına göre bir algoritmanız varsa metnin uzunluğunu dizinin boyutundan kolayca elde edebilmek adına UTF-16 kullanabilirsiniz. Eğer İngilizce dışında bir karakter kullanılmamış ise ASCII kullanabilirsiniz. Transfer işlemlerinde ise alıcı hangi kodlama biçimini destekliyorsa onu seçmelisiniz. Örneğe baktığımızda UTF-8 ile ASCII arasında boyut farkını göreceksiniz fakat bilin ki ASCII kodladığınız yerde bazı karakterler geri gelmeyecek şekilde yok oldu.  

```csharp
var yazi = Encoding.UTF32.GetString(dizi32);
var yazi16 = Encoding.Unicode.GetString(dizi16);

// Yanlış
var caprazTest1 = Encoding.Unicode.GetString(diziascii);
var caprazTest2 = Encoding.UTF8.GetString(dizi16);
```

Yukarıda _byte_ dizisinden geri çevrilme örneği görüyorsunuz. Alttaki iki örnekte farklı kodlamaları çapraz kullandığımız için elde edeceğimiz metinler anlamsız olacaktır.

## Dosyalar

Bir dosya ister metin dosyası ister başka bir türde olsun zaten aslında bir byte dizisidir.

```csharp
var dizi = File.ReadAllBytes(@"c:\c\test.txt");
```

Bu yöntem tüm dosyayı okuyup belleğe alacaktır. Bu yöntem görece küçük dosyalar için çok işe yararken büyük dosyalarda problemli olacaktır.

```csharp
using (var stream = File.OpenRead(@"c:\c\test.txt"))
{
	var buffer = new byte[1024];

	var tur = 0;
	var okunanAdet = 0;
	while(true)
	{
		okunanAdet = stream.Read(buffer, tur * buffer.Length, buffer.Length);
		//okunanAdet = await stream.ReadAsync(buffer, 0, buffer.Length);
		
		// !!! buffer da okunan parça var !!!
		
		if(okunanAdet < buffer.Length){
			break;
		}
		tur++;
	}
}
```
yukarıdaki örnek ilgili dosyayı 1024byte'lık parçalar halinde okumaktadır. Tüm dosyalar 1024'ün katı uzunlukta olmak zorunda değil tabi. Dosyanın sonuna doğru 1024´den daha az bir byte okunacaktır. Bu okunabilen byte adedi `okunanAdet` değişkenmizdedir. Read metodunun _async_ ve normal şekilde iki kullanımı var. Eğer dosya büyükse _async_ olanı tercih etmenizi öneririm. Küçük dosyalarda durum tartışmaya açıktır.

### Sayısal değişkenler

`BitConverter` sınıfı 10 farklı primitive tür için byte dizisine çevirmeyi ve tersine çevirmeyi sağlamaktadır. 

> &#128064; Primitive türler hakkında daha önce biraz bahsetmiştim. İlgili yazı için [tıklayın.](http://www.cihanyakar.com/primitive_type_kavrami)


```csharp
var sayi = 3423423;
var dizi = BitConverter.GetBytes(sayi);
// 191 60 52 0
```

Bu sınıfın `decimal` türünü içermediğini fark edeceksiniz. Bunun sebebi _decimal_ sınfının bir primitive type olmamasından kaynaklanıyor. `Decimal` türü için iki aşamalı bir yol izlememiz gerekiyor. Bu yönteme daha önce [kayar nokta kavramından](http://www.cihanyakar.com/kayarnokta) söz ettiğim yazımda değinmiştim. Kodu görelim :

```csharp
decimal a = 7.5M ;

var bits = decimal.GetBits(a);
var bytelar = bits.SelectMany(i=>BitConverter.GetBytes(i)).ToArray();
```
İlk aşamada _getbits_ metodu ilgili _decimal_ değişkeni _int_ parçalara ayırmakta. Daha sonra _bitconverter_ kullanarak her bir _int_ değişkeni _byte_ dizisine çevirip bunları uç uca ekliyoruz. 

### Resim türündeki değişkenler

System.Drawing kütüphanesini kullanarak çizim yapıyorsak bunun için aşağıdaki kod bloğu işimizi görecektir.

```csharp
byte[] dizi;
using (var bitmap = new Bitmap(16, 16))
{
	//örnek bir bitmap oluşturuyoruz
	using (var graphics = Graphics.FromImage(bitmap))
	{
		graphics.DrawString("C", new Font("Arial", 6), Brushes.Red, 0F, 0F);
	}


	// çevirme
	using (var stream = new MemoryStream())
	{
		bitmap.Save(stream, System.Drawing.Imaging.ImageFormat.Png);
		dizi = stream.ToArray();
	}
}
```

Kodun ilk yarısında 16*16 lik bir tuvale kırmızı bir C harfi basıyorum. Daha sonra bellek üzerinde bir stream açıyorum. Stream konusuna [stream kavramlarını anlattığım](http://www.cihanyakar.com/streamkavrami) yazıda değinmiştim. Detaylar için o yazımı da okuyabilirsiniz. Açtığım streame ilgili resmi dosyaya kaydeder gibi kaydediyorum. Kayıt esnasında da imajımın türünü seçiyorum. Ben örnekte _png_ formatını kullandım. Fakat dizi üzerinde işlem yaparak resim işleyecekseniz _bitmap_ kullanmanızda fayda var. Çalışma zamanında bit düzeyinde işlem yapmanız gerekiyorsa _unsafe_ metorları veya _bitmap.LockBits()_ metodunu kullanmanızı performans açısından öneririm.

### Sınıflar

Sınıfların byte dizisi haline getirilmesi için faklı yol bulunmaktadır. Her biri amacınıza göre değişecektir. Ben .net den çıkmadığınızı varsayarak basit bir serileştirme yöntemi önereceğim.

```csharp
[Serializable]
public sealed class Nesne
{
	public int Yas { get; set; }
	public string AdSoyad { get; set; }
}

static void Main()
{
	var ornek = new Nesne { Yas = 32, AdSoyad = "Cihan Yakar" };
	byte[] dizi;
	using (var memoryStream = new MemoryStream())
	{
		// System.Runtime.Serialization.Formatters.Binary
		BinaryFormatter bfWrite = new BinaryFormatter();
		bfWrite.Serialize(memoryStream, ornek);
		dizi = memoryStream.ToArray();
	}
}
```
Yukarıdaki kod bloğunda Nesne isimli bir sınıfım var. Bunun serileştirilebileceğini içeren bir nitelik almış durumda kendisi. _Main_ metodumda bu sınıftan bir örnek (instance) oluşturuyorum. Daha sonra bu sınıfı `BinaryFormatter` kullanarak serileştiriyorum. 

Bu byte dizisini daha sonra yine `BinaryFormatter`ın `Deserialize` metodunu kullanarak ete kemiğe büründürebiliriz. Örneğin bir nesneyi bu şekilde byte dizisi haline getirip internet üzerinden başka bir bilgisayara aktarabilirsiniz. Bu yöntemin JSON veya başka bir biçim kullanmaktan çok daha hızlı olması beklenir.

Yazının sonuna geldik. Fakat bu yazı güncellemenmeye gayet açık, geri dönüşlerinize göre şekillendirebilirim.

