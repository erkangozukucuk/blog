---
Title: C# ile Tek Yönlü Şifreleme (özet) Algoritmalarına Giriş
PublishDate: 12/12/2017
IsActive: True
Tags: C#
---

Veriyi şifreleme ile ilk yazımda konuya tek yönlü şifreleme mantığı ile başlamak istedim. Tek yönlü şifreleme adından da anlaşıldığı üzere şifreleme yapıldıktan sonra orijinal bilgiye ulaşılması imkansız(_ters işleminde şifreli metnin açtığınızda sonsuz karşılığı olduğu için_) olan şifreleme yöntemlerinin genel adıdır.

Tek yönlü şifrelemelerin oldukça yaygın kullanımı vardır. Orijinal bilgiye tekrar ulaşılamadığı için gizli bilgi iletiminde kullanılamayacakları açıktır. Peki nerelerde kullanıyoruz? Genel olarak "doğrulama" işlemlerinde kullanıyoruz diyebiliriz. 

* Bir bilgi gerçekten "Cihan" kullanıcısı tarafından mı üretilmiş? (sign)

* Bilgi iletim sırasında bozulmuş mu? (checksum) 

* Elimizdeki kod geçerli bir lisans kodu mu? 

* Kullanıcının parolasını bilmeyeyim ama yine de doğrulayabileyim; nasıl?

* Dağıtık bir yapıda ortak karar almayı nasıl sağlarız? (bizans generalleri problemi)

* Bir yığın bilgi içerisinde aradığımı veya  tekrar eden verileri  nasıl hızlı bulurum. (summary/hash)

  ​

Gibi soruların cevaplanması için oldukça işe yarar bir araçtan bahsediyoruz. Son soruya dikkat ederseniz tek yönlü şifreleme güvenlik amaçları dışında da kullanılabilmektedir. 

Mantığı kavramak için basit basit örnekler yapalım. Bir tam sayıyı tek yönlü olarak şifreleyelim. 

```csharp
public static int Sifrele (int sayi)
{
	 return sayi % 21;
}
```
Tamamen anlık bir uydurma sonucunda ürettiğim bu fonksiyon aldığı sayıdan başka bir sayı üretmektedir. Parametre olarak 1500 verdiğimde sonuç 9 çıkmaktadır. Bu 9 değerinden 1500 ün tekrar üretilmesi mümkün olmayacaktır. Mümkün olmayacaktır çünkü _sayi_ parametresinin bir çok değeri için 9 sonucu çıkacağından girilen değeri bilemenin imkanı yoktur, sadece bir sonuç kümesi elde edilebilir. Fakat 1 den 3000 e kadar olan sayılardan 143ü için 9 değeri çıkacaktır ki bu tip şifreleme yöntemlerinde hiç istenmeyen bir durumdur. Çünkü bu mantıkta, parametre için 21 sonuçtan birisi dönecektir.   Özellikle parola doğrulamalarında bu aynı sonuç üreten sayılar gerçek parola dışındaki değerlerinde doğrulanmasını sağlarlar. Parolam 1000 olsun, bu fonksiyon bana 13 sonucunu döndürecektir. Veritabanında 13ü saklarım. Bu sayede veritabanını ele geçiren saldırgan gerçek parolanın 1000 olduğunu bilemez. Fakat, 1000 değil de 3016 ile giriş yapılmaya çalışılırsa bu işlem onaylanacaktır. Bu aynı sonucu üreten parametrelerin nasıl engellendiğine bu ve gelecek yazılarda bahsedeceğim.

Gerçek Dünya'da bu işi genellikle bitlik ifadeler üzerinden ikili sayı sistemini (binary) kullanarak yapıyoruz. Tam burada bilmemiz gereken operatörler var, bunlar AND, OR, XOR, SHIFT operatörleri bunlardan da şifreleme açısından en önemlisi XOR operatördür. Bu konularda eksiğiniz varsa burada ufak bir ara verip araştırmanızı öneririm.

Az önceki fonksiyonumuzu biraz geliştirelim. Başarılı bir tek yönlü şifreleme algoritması hep aynı boyutta çıktı vermelidir. Bu sayede verinin boyutu hakkında bir bilgi vermemiş oluruz. Bizde sabit 32bit (4byte) çıktı verecen bir özetleyici yazalım. Bu uygulama girdi olarak UTF8 kodlamasında bir metin alsın. Öncelikle metni sayılarla ifade etmemiz gerekecek. Ardından metinin kendisini 4byte'lık parçalara ayırıp bir sonraki parçası ile XOR işlemine tutacağız. XOR işlemi eldesiz olduğundan 4byte XOR 4byte = 4byte olacağından sonucun 4byte olacağı garantidir. Tabi girilen metin 4byte'ın katı olmayabilir. Bunun için metni sonundan tamamlayacağız.

```csharp
private static int HashUret(string girdi)
{
    var bytes = Encoding.UTF8.GetBytes(girdi).ToList();

    var artan = 4 - bytes.Count % 4;
    for (var i = 0; i < artan; i++)
    {
        bytes.Add(default(byte));
    }

    bytes.TrimExcess();

    var hash = default(int);
    for (var i = 0; i < bytes.Count; i += 4)
    {
        var row = BitConverter.ToInt32(new[] { bytes[i], bytes[i + 1], bytes[i + 2], bytes[i + 3] }, 0);
        hash ^= row;
    }
    return hash;
}

```
Kodu açıklayayım, gelen yazıyı byte dizisine çevirip, dörder dörder okuyoruz (4byte = int). Okuduğum değerleri bir önceki değer ile özel veya (xor) işlemine sokuyoruz. Neticede metnin uzunluğu ne olursa olsun elimizde her zaman bir `int` oluyor.

Bu fonksiyonu uzun metinler ile deneyin. Sadece tek bir harf değiştirdiğinizde sonuçta büyük değişikler olmadığını fark edeceksiniz. Bu durum kötücül kişi için bir ışık yakacaktır. Biraz uğraşıp bizim dörtlü mantığımızı keşfedip, metinde yaptığı değişikleri hash kodunun aynı çıkmasını sağlayacak şekilde yapabilir. Bu da bu algoritmanın değişiklik kontrolüne karşı zayıf olması demektir.

Sonraki yazılarda inceleyeceğimiz MD5, SHA gibi bilindik algoritmaların bu problemleri nasıl çözdüğünü göreceğiz.

Zaman zaman "XXX kırıldı, bir siteye giriyorsun sana sonucu söylüyor" gibi sözler işitebilir/okuyabilirsiniz. Ben zaman zaman forumlarda buna denk geliyorum ve bir hash verip çözmelerini istiyorum. Tabii ki daha çözebilen çıkmadı. Bu algoritmalar tek yönlü oldukları için geriye döndürme işlemli gerçekten mümkün değildir. Peki ama bu siteler nasıl çalışıyor? Basit, ben "merhaba" yazısı için bir hash değeri alırsam ve bunu bir kenara yazarsam. Daha sonra bana bir hash değeri verildiğinde kenara yazdıklarım ile karşılaştırarak sonucu söyleyebilirim. Bu kenara yazma işlemi kısa metinler için mantıklıdır. Uzun metinlerde olasılıklar günümüz bilgisayarını aştığı için mümkün değildir. Bu sebeple bu konuda güvenlik sağlanması için hash değeri alınacak değere uzun ve karışık bir değer eklerseniz bu tip saldırılardan (rainbow attack) kolayca kurtulabilirsiniz.

Sorularımızı  yanıtlayalım:

### Bir bilgi gerçekten "Cihan" kullanıcısı tarafından mı üretilmiş?
Bu soruyu yanıtlamak için açık anahtar şifreleme yöntemlerin de fikir sahibi olunması gereklidir. Anahtar çifti yönteminde özel ve genel iki anahtar bulunur. Özel anahtar kimseyle paylaşılmaz. Genel anahtar ise herkes tarafından bilinir. Bir anahtarın şifrelediğini ancak diğer anahtar açabilir. Bir kişi bir bilgiyi kendi özel anahtarı ile şifrelerse, herkes o bilgiyi ilgili kişinin açık anahtarı ile açabilir. Bir belgenin o belgeye erişemeyeceği özeti çıkartılır ve bu özet gizli/özel anahtar ile şifrelenirse. Alıcı kişi özeti göndericinin açık anahtarı ile çözer ve bilginin özetini aldığında aynı değere ulaşırsa bu bilginin değişmediğini ve o kişiden geldiğinden emin olacaktır.

### Bilgi iletim sırasında bozulmuş mu? (checksum) 
Herhangi bir dosyayı tek yönlü bir algoritma ile şifreleyip, bu değeri (checksum) de alıcıya iletirseniz. Dosya eline ulaştığında kendisi de aynı algoritma ile şifreleme yapıp sonuçları karşılaştırabilir. Eğer farklı değerler üretilmiş ise dosyayı tekrar talep eder.

### Elimizdeki kod geçerli bir lisans kodu mu? 
Bilinmeyen bir hash fonksiyonu oluşturduğunuzu düşünün. Bu durumda bu fonksiyondan X değerini döndüren değerleri doğru kabul ederseniz. Elinizde bir lisans kontrol mekanizması olur. Fonksiyon sizde olduğu için üretmeniz kolay olacaktır. Fakat kötücül kişinin bir zorlama saldırısı yapması günümüz bilgisayarları için zorlayıcı olacaktır. 

### Kullanıcının parolasını bilmeyeyim ama yine de doğrulayabileyim; nasıl?
En tipik tek yönlü şifreleme kullanım amaçlarından birisi de budur. Kullanıcının parola bilgisini elde edilebilir bir şekilde veritabanına koymak oldukça tehlikelidir. Bir çok insan aynı parola bilgisini farklı sistemlerde kullandıkları için sizin veritabanınız ele geçirilirse, aslında güvenlik sağlayan başka bir sisteminde girişini teslim etmiş olursunuz. Veya kötü bir şirket çalışanı bu parola'yı veritabanından okuyabilir. Bu sebeple kullanıcının parola bilgisini veritabanında hiç saklamamak gerekir. Bunun yerine önce tuzlayıp (salt) sonra özetini almak ve veritabanında bunu saklamak akılcı olacaktır.
Örneğin, kullanmak istediğim parola = "passw0rd" olsun. Bunu yukarıdaki fonksiyondan geçirirsem. 385962247 sayısı elde edilir. Fakat bunun güvensiz olduğundan bahsetmiştim. Bunun yerine "passw0rd" + "J^lB)S4Qad5y@[c&!Y|H7KvZH42*+&" gibi bir değer kullanıp hash işlemi yaparsam (1664574256) saldırılara karşı da önlem almış olurum.

### Dağıtık bir yapıda ortak karar almayı nasıl sağlarız?
Klasik bizanslı generaller probleminde birden fazla generalin çoğunluk sağlayarak bir karar alması gerekmektedir. Bu sebeple elçilerini bir birilerine göndererek kararlarını açıklarlar. Tüm gönderme işlemi bittiğinde karar verilmiş olur. Fakat, generallerden birisi bir generale evet diğerine hayır oyu gönderirse veya karar yolda değişirse ortak karara bağlanmamış olur. Bu problem dağıtık sistemler içinde aynı şekilde geçerlidir. Bu problem için bir çok çözüm olsa da en bilinen ve şu sıralarda popüler olsan blockchain yöntemi SHA özet fonksiyonunu kullanmaktadır. Merak edenler araştırabilir.


### Bir yığın bilgi içerisinde aradığımı ve tekrarlı dosyaları nasıl hızlı bulurum?
Bilgisayarınızdaki bir dizinde bir birinin aynısı olan dosyaları bulmak istiyor olun. 4 dosya varsa 6 tane karşılaştırma işlemi yapmanız gerekir. 100 tane varsa bu 4950 yapar. Her dosyanın 1gb olduğunu düşünün, çok fazla okuma ve yazma işlemi gerekecektir. Bu işi yapmak için aslında her dosyayı bir kere özetini çıkartarak okursam ve sonra bu özetleri karşılaştırırsam çok daha az I/O işlemi bu işi halledebilirim. Bu konudan bir gün yazmayı planladığım hashbucket yazısında detaylıca bahsetmek istiyorum.
