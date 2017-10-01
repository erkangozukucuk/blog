---
Title: C# 7.0 ve 7.1 yenilikleri
PublishDate: 1/10/2017
IsActive: True
Tags: C#, Xamarin, Etkinlik
---


## TUPLE YENİLİKLERİ

### Tuple neydi?
Önce Tuple kavramını hızlıca hatırlayalım. Tuple'lar yeni bir sınıf tanımlamadan elimizdeki verileri paylaşmamıza yaran basit yapılardır. En basit tanımı şöyle örneklenebilir:

```csharp
Tuple<string, string> city = new Tuple<string, string>("Ankara", "06");
Console.WriteLine(city.Item1);
```

Burada iki adet `string` türünden özelliğe sahip bir sınıfa ihtiyaç duyduğumuzu belirtiyoruz ve değer ataması yapıyoruz. Daha sonra bu değerlere erişmek istersek aynı sırayla `Item1,Item2...` şeklinde erişmemiz mümkün olacaktır. Bu yapı sayesinde bir method birden fazla değer döndürebilir :

```csharp
Tuple<bool,int> TryParse(string text)
{
	//...
	return new Tuple<bool,int>(true,23); 
}
```
Böylece `out` kullanmaksızın bir methodan birden fazla parametre dönmeniz mümkün olur. Yine LINQ sorgularınızda bir _projection_ (izdüşüm, `Select(x=> new {x.Ad, x.No})`) işlemi yaptığımız sonucu geriye dönmek doğrudan mümkün olmazken bu tuple'lar ile mümkün olmaktadır.

### Problemleri neydi?

Tuplelar bir `class` dır ve eşitlikleri referans noktaları üzerinden kontrol edildiğinden bire bir aynı değerlere sahip iki tuple eşit olarak kabul edilemez. Ve `immutable` yapıdadırlar.  Immutable yapılar ile ilgil yazı için [tıklayın](http://cihanyakar.com/immutable-type_kavrami). Dolasıyla bir kere oluşturulunca üzerinde değişiklik yapılamaz. 

```csharp
var city = new Tuple<string, string>("Ankara", "06");
city.Item1 = "İstanbul"; // Geçersiz
```

### Neler değişti?

#### Tanımlama ve yapı
Tanımlama şeklimiz oldukça kolaylaştı.

```csharp
var city = ("Ankara", "06");
```
Basit bir şekilde parantez içinde değerlerimizi verebiliyoruz. Ve dilersek her bir property'e _scope içinde kalacak_ isimler verebiliyoruz:

```csharp
var city = (Name: "Ankara", No: "06");
```
Artık her iki şekilde de property değerlerine erişmek mümkün ve değerlerinin değiştirilmesi de mümkün:

```csharp
var city = (Name: "Ankara", No: "06");
city.Name = "İstanbul"; // Geçerli
city.Item2 = "34"; // Geçerli
```

Değişiklik geriye yönelik bir bozulmama olmaması için () ile yazdığım Tuple'lar aslında **ValueTuple** türünde _struct_ türünden yapılar olarak hazırlanmış durumda.

```csharp
var city = (Name: "Ankara", No: "06");
// var city = new ValueTuple<string, string>("Ankara", "06");
city.Name = "İstanbul"; // Geçerli
city.Item2 = "34"; // Geçerli
```

#### Deconstruction

Tuplelar parçalanıp değerleri farklı değişkenlere gönderilebiliyorlar.

```csharp
public static (string, string) TupleDeclarationReturn()
{
	return ("Ankara", "06");
}

void Main()
{
	(var n, var p) = TupleDeclarationReturn();
	Console.WriteLine(n);
}
```

Bu özellik de _Tuple Swap_ adı verilen bir durumu ortaya çıkartıyor:

```csharp
var a = 5;
var b = 7;
(a, b) = (b, a);
// a = 7
// b = 5
```
Değişken tanımlarının ardındaki satırda hem bir tuple tanımlanıyor hem de bu tuple dağıtılıyor. Bu sayede sanki üçüncü bir değişken kullanılmadan yer değiştirme yapılmış ilüzyonu ortaya çıkıyor.

## Discard kavramı

Bir önceki örnekte bir fonksiyondan dönen Tuple türündeki bir nesnenin özelliklerini değişkenlere dağıtmıştık. Peki ama bana bu özelliklerden sadece bir kısmı gerekseydi? Bu durumda her biri için boşu boşuna değişken mi oluşturmam gerekecekti? Bunun önüne geçebilmek adına _discard_ ları kullanabiliyoruz. Örnek kodu inceleyelim:

```csharp
void Main()
{
	(var n, _) = TupleDeclarationReturn();
	Console.WriteLine(n);
}
```

Normalde C# da  " _ " karakteri ile  değişken isimi kullanabiliyoruz. Bu yeni kavramda eğer aynı isimde bir değişken yoksa " _ " _discard_ olarak kabul ediliyor. Yani... Atama işlemi çöp olarak kabul ediliyor ve bellekte bir yer ayrılmıyor .

Bu yeteneği geriye dönüş değerine ihtiyaç duymadığınız metotları çağırırken kullanabilirsiniz:

```csharp
int x;
_ = int.TryParse(Console.ReadLine(), out x);
```
Burada Tuple konusundaki kadar anlamlı bir kullanım olmadı gibi. Sonuçta eşitliği hiç koymasam kod çalışacaktır. Bu daha çok kodlarınızı sürekli bir kod analizi yapan araçlardan geçiriyorsanız; bunun muhtemelen bir kodlama hatası olarak gözükmesini engelleme amacıyla kullandığınızda işinize yarayacaktır. Ve birazdan değineceğimiz _out_ konusunda güzel bir kullanımı olacak.

## Local Functions

Yerel fonksiyonlar fonsiyon içinde fonksiyon tanımlama işlemleridir. .net framwork açısından düşündüğümüzde performansa genellikle (yanlış tanımlar yapılmadığı sürece) iyi yönde olmaktadır. Yine bir fonksiyonun içindeki döngü dışında kullanılmayacak bir fonksiyonu diğer fonksiyonların görebileceği bir yerlerde tanımlamakta genel OOP prensiplerine pek uymamaktadır. Fonsiyon içerisinde fonksiyon tanımlamasını zaten .net 2.0 dan bu yana yapabiliyoruz. [(Bu konuda hazırladığım örnekleri buraya tıklayarak inceyeyebilirsiniz)](https://github.com/cihanyakar/c-7/tree/master/7.0/2.%20Local%20Functions). C# 7 ile beraber bu tanımlamar daha doğal ve anlaşılır bir hale gelmiş durumda:

```csharp
public enum Operation
{
	Add,
	Divide
}

public static double Calculate(double a, double b, Operation operation)
{
	double add(double number1, double number2)
	{
		return number1 + number2;
	}

	double divide(double number1, double number2) => number1 / number2;

	switch (operation)
	{
		case Operation.Add:
			return add(a, b);
		case Operation.Divide:
			return divide(a, b);
		default:
			return default(double);
	}

}

void Main()
{
	Console.WriteLine(Calculate(5, 5, Operation.Add));
}
```

Örneği inceleyecek olursanız bu fonksiyonları istersek `lambda` olarakda tanımlayabiliyoruz. 

## Out

Bir methodun parametresini geriye dönüş amaçlı kullanmak için out sözcüğünden yararlanıyoruz. Bu sözcükten yararlanırken out olarak verilecek parametrenin mutlaka önceden bir değişken olarak tanımlanmış olması gerekiyordu. C# 7 ile birlikte bu değişken artık parametreye değer verildiği anda tanımlanıyor. Ve işin aslında en güzel yanına gelecek olursak bu dönüş parametresi ile işimiz yoksa _discard_ özelliğini kullanabiliyoruz.

```csharp
void Main()
{
	if (int.TryParse(Console.ReadLine(), out int value))
	{
		Console.WriteLine(value * 2);
	}
	else
	{
		Console.WriteLine(default(int));
	}
}
```
Örnek kodda tipik bir konsoldan girilen metnin sayıya dönüştürülmesi var. `value` değişkeni `TryParse` metodunun içinde tanımlanmış durumda. Bir satır bir satırdır :)

Benim amacım konsoldan girilen şeyin sadece bir sayı olup olmadığını anlamak olsaydı. Bu durumda dönüştürülmüş hali ile işim olmayacaktır. Bunun örneği de şöyle olacaktır:

```csharp
void Main()
{
	if (int.TryParse(Console.ReadLine(), out int _))
	{
		Console.WriteLine("sayı");
	}
	else
	{
		Console.WriteLine("sayı değil");
	}
}
```

## Ref
Metot parametreleri out dışında aynı zamanda  _ref_ olarak tanımlanabiliyorlar. Bu özellik nesnenin değerinin değil de referansının atılmasını sağlıyor. Artık C#7 ile birlikte _ref_ sözcüğünün yetenekleri artmış durumda. Bir fonksiyon geriye referans döndürebiliyor veya bir değişken referans olarak tanımlanabiliyor. 

Değişkenlerin referans olarak tanımlanmasına bir örnek:

```csharp
int a = 5;
ref int b = ref a;
a = 7;

Console.WriteLine(a); // 7
Console.WriteLine(b); // 7
```
Eğer ikinci satırdaki _ref_ belirteçleri olmasaydı. Bu kod 7 ve 5 değerlerini ekrana basacaktı. Ama ikinci satırda b değişkeni bir _int_ değil _int_ bir değişkenin referansı şeklinde tanımlanıyor ve ona bir int değişkenin referansı bağlanmış oluyor. Ama değer okumak istediğimde pointerlardaki gibi referans noktasını değil, değeri okuyoruz.

Bu özellik fonksiyonlarda da kullanılabilir örneğin:

```csharp
void Main()
{
	var array = new[]
			{
				5, 10, 15, 20, 25
			};

	Console.WriteLine(array[0]);
	Console.WriteLine(array[1]);
	Console.WriteLine(array[2]);
	Console.WriteLine(array[3]);
	Console.WriteLine(array[4]);
	
	ref int max = ref GetMax(array);
	max = 78;
	Console.ReadLine();
	
	Console.WriteLine(array[0]);
	Console.WriteLine(array[1]);
	Console.WriteLine(array[2]);
	Console.WriteLine(array[3]);
	Console.WriteLine(array[4]);
}

public static ref int GetMax(int[] array)
{
	int maxIndex= 0;
	for (int i = 0; i < array.Length; i++)
	{
		if (array[i] > array[maxIndex])
		{
			 maxIndex = i;
		}
	}
	return ref array[maxIndex];
}
```

`GetMax` metodu geriye bir _int_ referansı döndürmektedir. Bu da kendisine verilen dizinin en büyük indisli elemanının referansıdır. Bu durumda geriye dönen değer bir referans olduğu için değer ataması yaptığımda (`max = 78` satırı) dizinin içindeki ilgili elemanın değeri de değişmektedir. Oyunlarda, grafik işlemede vb. performans gerektiren noktalarda küçük optimizasyonlar yapmayı sağlayacaktır.

## Pattern Matching
Desen eşleme, yine zaten yapabildiğimiz bir işlemi daha kısa yoldan yapabilmemizi sağlayan bir yöntem. 

Klasik bir tür dönüşüm hikayesini ele alalım ve daha sonra nasıl kısalttığımıza bakalım:

```csharp
void Main()
{
	Console.WriteLine(Is("string"));
	Console.WriteLine(Is(5));
}

public static string Is(object obj)
{
	string s = obj as string;
	if (obj != null)
	{
		return s;
	}
	return "ERROR";
}
```
Bu örnekte gelen obje string türünde ise geriye dönüyoruz aksi halde başka bir string türünden nesne dönüyoruz. C# 7 ile baraber _out_ da olduğu gibi bu işi tek satıra indirebiliyoruz:

```csharp
public static string Is(object obj)
{
	if (obj is string s)
	{
		return s;
	}
	return "ERROR";
}
```
Bu işi sadece _is_ operatorü ile değil _switch_ operatörü ile de yapmam mümkün:

```csharp
void Main()
{
	Console.WriteLine(Switch("cihan"));
	Console.WriteLine(Switch(5));
	Console.WriteLine(Switch(5.7));
	Console.WriteLine(Switch(5.7M));
}

public static string Switch(object obj)
{
	switch (obj)
	{
		case string s:
			return "text :" + s;
		case int _: // discard
			return "integer number";
		case double b:
			return "binary floating point number";
		case decimal d:
			return "decimal floating point number";
	}
	return "unknown";
}
```

## Expression Bodied Members
Daha önceki c# sürümlerinden beri aslında _lambda_ ifadeler ile sınıf üyelerini tanımlamak mümkün hale gelmişti. C# 7 ile bu kapsam biraz daha genişletildi.

```csharp
public class Sample
{
	private static int _value;
	public Sample(int value)
	{
		_value = value;
	}

// Method
	public void AddOne() => _value += 1;


// readonly property
	public int Value => _value;
	
	// eskiden
	//public int Value
	//{
	//    get
	//    {
	//        return _value;

	//    }
	//}
}
```
## Async Main
Başlık yeterince açık. C# 7.1 den itibaren main methodlarını _async_ olarak tanımlamak mümkün.

## Target Typed Default Literals
Meali, _default_  keywordünün hedefinin türü belirli ise artık ayrıca tür belirtmemiz gerekmiyor. Bu özellikte C# 7.1 ile geldi.

```csharp
void Main()
{
	int a = default;
	Console.WriteLine(a);
}
```
Örnekteki `a` değişkeninin türü _int_ olduğundan ayrıca `default(int)` şeklinde belirtmeye ihtiyacımız yok. Benzer kullanımı generic ifadeler, metot dönüşleri gibi dönüş türü belirli olan her yerde kullanabilirsiniz.

Son olarak daha fazla detaya [Microsoft'un ilgili makalesinden](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-7) ulaşabilirsiniz.






