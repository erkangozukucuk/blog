---
Title: Immutable Type Kavramı Hakkında
PublishDate: 07/08/2017
IsActive: True
Tags: C#
---

[Pure fonksiyonları](http://cihanyakar.com/pure_function_ve_idempotent_function_kavrami) anlatırken değişmez (_immutable_) türlerin adını anmıştım. Değişmez türlerde tıpkı saf fonksiyonlar gibi uygulayabildiğimiz zaman kodlarımızın getireceği olası baş ağrılarını azaltmaktadır. Önce kısaca tanımı yapayım.
> **Immutable türler**, bir kere oluşturulduktan sonra durumu (_state_) değişmeyen sınıf veya yapılardır.

Bu şekilde olan türleri aslında sıklıkla kullanıyoruz. Örneğin, `int, string` gibi en sık kullandığımız türler bu şekildedir.

```csharp
string metin = "test";
metin = "deneme";
```
bu basit örnekte sağ tarafta `"test"` olarak bulunan kısımı  `new string(new []{'t','e','s','t'});` gibi düşünebilirsiniz. Birazcık terimlerle anlatmak gerekirse; aslında metin değişkeninin _heap_ de bulunan değerini değiştirmiyorsunuz. Bunun yerine _heap_ de yeni bir string oluşturulup o yeni adres _stack_ de yerini alıyor. 

Peki neden değişmez türleri kullanmak gerekir? Eğer bir türün değerinin oluşturulduktan sonra bir daha değişmeyeceğini bilirseniz, bunun için fazladan kontroller yapmanıza gerek kalmaz. Bu sayade de sınıfın iç bütünlüğü hiç bir zaman bozulmaz. Ek bir fayda olarak, bu türler kendiliğinden _thread safe_dir.

Biraz örnek vermek istiyorum: 

```csharp
namespace Immutable
{
    internal class Program
    {

       internal sealed class Tarih
        {
            public int Gun { get; set; }
            public int Ay { get; set; }
            public int Yil { get; set; }
        }

        internal sealed class Ogrenci
        {
            public string Ad { get; set; }
            public string Soyad { get; set; }

            public Tarih DogumTarihi { get;  set; }
        }

        private static void Main()
        {
            var cihan = new Ogrenci
                        {
                            Ad = "Cihan",
                            Soyad = "Yakar",
                            DogumTarihi = new Tarih {Gun = 3, Ay = 1, Yil = 1986}
                        };

            cihan.DogumTarihi.Yil = 1801;
        }
    }
}
```
Bu örnekte yer alan `Tarih` ve `Ogrenci` sınıflarının ikisi de değişmez yapıda değildir. Doğal olarak yapıcı (_constructor_) metot haricinde de ne zaman istenirse değişiklik yapılabilmektedir. Diyelim ki bir `Ogrenci` mutlaka 130 yaşından küçük olmalıdır diye bir şartımız olsun. Bu durumda son satırlara doğru olan `cihan.DogumTarihi.Yil = 1801;` atamasını nasıl engelleyebilirim? `Tarih` sınıfına bu sadece bu iş için bulaşmam pek doğru olmayacak gibi durmaktadır. Çünkü prensipler gereği her sınıf kendi işi ile ilgilenmelidir. Yani, _tarih_ sınıfında doğal olarak ay 12 den büyük olamaz gibi koşulları ekleyebilirim fakat başka bir sınıfın engellemesi gereken bir atamayı _tarih_ sınıfı içinde yapmamalıyım. En fazla bir değişkende değişiklik olduğu zaman bir `event` fırlatabilirim -ki o da güzel bir çözüm olmaz. Bu durumda bu işi sanki `Ogrenci` sınıfından halletmeliymişim gibi fakat `Tarih` içindeki `Yil` özelliğinin değiştiğini yakalamam pek mümkün değil (_pire için yorgan yakacak durumları saymıyorum_).

Tam bu noktada dikkati diğer iki özelliğe (_property_) çekemk istiyorum. Örneğin benzer bir durum _Ad_ özelliğinde olsaydı ve _ikinci karakter `X` olamaz_ şeklinde garip bir istek olsaydı. Bu `Ad` özelliğini `cihan.Ad = …` demeden değiştirebilir miydim?

```csharp
var cihan = new Ogrenci
            {
                Ad = "Cihan",
                Soyad = "Yakar",
                DogumTarihi = new Tarih {Gun = 3, Ay = 1, Yil = 1986}
            };

cihan.Ad[1] = "X";
```
Buradaki son satır hata verecektir. Özellikle doğrudan bellek erişimi gibi uçuk yollara gitmediğiniz sürece `Ad` özelliğine komple atama yapmadan bu değişikliği yapamazsınız.

Geri dönelim, bütün mesele aslında `Tarih` türünün değişmez olması gerekirken değişken (_mutable_) olarak tanımlanmasından çıkmaktadır. Kodu şu şekilde değiştirelim :

```csharp
internal class Program
{
    internal sealed class Tarih
    {
        public int Gun { get; }
        public int Ay { get; }
        public int Yil { get; }

        public Tarih(int gun, int ay, int yil)
        {
            Gun = gun;
            Ay = ay;
            Yil = yil;
        }
    }

    internal sealed class Ogrenci
    {
        public string Ad { get; set; }
        public string Soyad { get; set; }

        public Tarih DogumTarihi { get; set; }
    }

    private static void Main()
    {
        var cihan = new Ogrenci
        {
            Ad = "Cihan",
            Soyad = "Yakar",
            DogumTarihi = new Tarih(3, 1, 1986)
        };

        cihan.DogumTarihi.Yil = 1802;
    }
}
```
Bu durumda son satır artık hata verecektir. Bu durumda kullanıcı bu değişikliği şu şekilde yapmalı :

```csharp
private static void Main()
{
    var cihan = new Ogrenci
    {
        Ad = "Cihan",
        Soyad = "Yakar",
        DogumTarihi = new Tarih(3, 1, 1986)
    };

    cihan.DogumTarihi = new Tarih(gun: cihan.DogumTarihi.Gun,
                                   ay: cihan.DogumTarihi.Yil,
                                  yil: 1802);
}
```
Kod bu şekilde değişince doğrudan `DogumTarihi` özelliğine bir atama olduğu için ilgili _setter_ metotta değişiklik yakalanıp, kullanıcıya hata verilebilir. _Ogrenci_ türünün ise değişmez olup olmayacağı da kullanılacağı senaryoya bağlıdır. Bir sınıf değişmez olduğunda işinizi görüyorsa onu _değişmez_ yapın. Yine bir sınıf çok küçükse ve değişmez ise onu bu sefer bir yapı _struct_ olarak değerlendirin. Neticede `Tarih` yapısı şu hali alır :

```csharp
 internal struct Tarih
 {
     public int Gun { get; }
     public int Ay { get; }
     public int Yil { get; }

     public Tarih(int gun, int ay, int yil)
     {
         Gun = gun;
         Ay = ay;
         Yil = yil;
     }
 }
```

Peki ama _Ogrenci_ sınıfını değişmez yapmak isteseydim ve _Tarih_ sınıfı harici bir kütüphaneden geliyor olsaydı? Bu durumda ne yapmak gerekirdi? Örnek kodu vereyim:

```csharp
namespace Immutable
{
    internal class Program
    {
        // Bu sınıfın bir .dll ile geldiğini ve değiştiremediğimizi varsayıyoruz
        internal sealed class Tarih
        {
            public int Gun { get; set;}
            public int Ay { get;  set;}
            public int Yil { get; set; }
        }

        internal sealed class Ogrenci
        {
            public string Ad { get; }
            public string Soyad { get; }

            public Tarih DogumTarihi { get;  }

            public Ogrenci(string ad, string soyad, Tarih dogumTarihi)
            {
                Ad = ad;
                Soyad = soyad;
                DogumTarihi = dogumTarihi;
            }
        }

        private static void Main()
        {
            var cihan = new Ogrenci
            (
                ad: "Cihan",
                soyad: "Yakar",
                dogumTarihi: new Tarih {Gun= 3, Ay = 1, Yil = 1986 }
            );

            cihan.DogumTarihi.Yil = 1802;
        }
    }
}
```
Burada `Ogrenci` sınıfını değişmez yapmaya çalıştıysam da hain `Tarih` sınıfının yazarı buna imkan vermemektedir. Aynı durumun dizilerde ve diğer referans tiplerinde de olması mümkündür. C# 'da bunu engellemek için maalesef bir koruma bulunmamaktadır (_örnek: c++ const anahtar sözcüğü_). Yapılabilecek en iyi kullanım aşağıdaki gibidir:

```csharp
internal sealed class Ogrenci
{
    public string Ad { get; }
    public string Soyad { get; }

    private readonly Tarih _dogumTarihi;

    public int DogumTarihiGun => _dogumTarihi.Gun;
    public int DogumTarihiAy => _dogumTarihi.Ay;
    public int DogumTarihiYil => _dogumTarihi.Yil;

    public Ogrenci(string ad, string soyad, Tarih dogumTarihi)
    {
        Ad = ad;
        Soyad = soyad;
        _dogumTarihi = dogumTarihi;
    }
}

private static void Main()
{
    var cihan = new Ogrenci
    (
        ad: "Cihan",
        soyad: "Yakar",
        dogumTarihi: new Tarih {Gun= 3, Ay = 1, Yil = 1986 }
    );

    cihan.Yil = 1802;
    Console.ReadLine();
}
```

Bu durumda orijinal `Tarih` nesnesini sakladığımız için sınıfımız dışarıdan gelen değişikliklere kapalı olacaktır. Fakat maalesef gerçek dünyada kullanacağımız sınıflar bu kadar yalın olmayabiliyorlar.  Bu durumda kullanıcının erişim yöntemini biraz değiştirmeniz gerekecektir.

```csharp
internal sealed class Ogrenci
{
    public string Ad { get; }
    public string Soyad { get; }

    private readonly Tarih _dogumTarihi;

    public T DogumTarihi<T>(Expression<Func<Tarih, T>> erisimFonksiyonu)
    {
      // return erisimFonksiyonu.Compile()(_dogumTarihi); yan etkisi var
        var prop = (PropertyInfo)((MemberExpression)erisimFonksiyonu.Body).Member;
        return (T)prop.GetValue(_dogumTarihi); 
    }
    public Ogrenci(string ad, string soyad, Tarih dogumTarihi)
    {
        Ad = ad;
        Soyad = soyad;
        _dogumTarihi = dogumTarihi;
    }

}

private static void Main()
{
    var cihan = new Ogrenci
    (
        ad: "Cihan",
        soyad: "Yakar",
        dogumTarihi: new Tarih { Gun = 3, Ay = 1, Yil = 1986 }
    );
    var yil = cihan.DogumTarihi(t => t.Yil);
    //   var yil = cihan.DogumTarihi.Get(t => t.Yil);
    Console.WriteLine(yil);
    Console.ReadLine();
}
```
Burada Linq'in kullandığı yöntemibe benzer bir kullanım yapmış olduk. Kullanıcıdan `Tarih` sınıfından bir nesne verildiğinde geriye bir şey dönmesini istiyoruz. Kodda oldukça performanslı çalışacak `complie` satırını kapatıp işi yavaşlatacak _reflection_ yöntemine girişiyorum. Sebebi değişmezliği korumak çünkü kullanıcı isterse `cihan.DogumTarihi(t => 1808);` yazabilir veya daha kötüsü t üzerinde değişiklik yaptıracak bir metot üzerinden dolaştırmayı deneyebilir. Bu durumda yazılan ifadenin (_expression_) türü _MemberExpression_ olmayacağı için hata verecektir. Burada performansı düşürdüğüm açıktır. Eğer bir önceki yöntemi uygulayabiliyorsanız onu uygulamanızını öneririm.

Peki ya koleksiyon tipleri? Bir dizi, bir kuyruk vs. olduğunda bunlar için ne yapmak gerekiyor? Bir önceki yöntem normal sınıflar için işe yararken koleksiyon tipleri yorucu olacaktır. Burada ise Microsoft devreye giriyor ve bize şöyle harika bir kütüphane sunuyor : [https://msdn.microsoft.com/en-us/library/mt452182(v=vs.111).aspx](https://msdn.microsoft.com/en-us/library/mt452182(v=vs.111).aspx)
Bu kütüphanede neredeyse tüm koleksiyon tiplerinin değişmez karşılıkları bulunuyor. Kullanıcı bizim sınıfımızdan bir List aldığında `.Add(...)` methodunu yine çağrıabiliyor ama bu tıpkı `string.Replace` de olduğu gibi çalışıyor. Yani orijinal koleksiyonda hiç bir değişiklik yapmadan geriye yeni ve ilgili değişiklik yapılmış bir koleksiyon dönüyor. Mutlaka deneyin.

Bir kavram yazısının daha sonundayız. Özellikle değinmemi istediğiniz bir kavram varsa bana ulaşabilirsiniz.