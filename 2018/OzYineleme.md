---
Title: Özyineleme (Recursion) Kavramı
PublishDate: 25/07/2018
IsActive: True
Tags: C#
---

Özyinelemeli (Recursive) algoritmalar, yazılıma ilk merhaba denilen günlerde anlaşılması biraz zor olan bir kavramdır. Programlamaya ısındıkça durum tersine döner ve karmaşık problemler için özyinelemeli bir yöntem varsa kod hızlıca bu şekilde yazılır geçilir. Kavram, bir metodun çalışma süresi içerisinde kendisini çağırması durumu olarak ifade edilir. Tekrar çağırılma sayısı ve hangi satırda çağrıldığına göre doğrusal, kuyruk, ikili, vb. kategorilere ayrılmaktadır.  

Özyinelemeli algoritmalar çoğu zaman yazılımcıyı kolayca çözüme götüren ve okunabilirliği yüksek kodlardır. Fakat geliştirme yaptığımız dilin bu kodlara ne kadar uyumlu olduğu önemlidir. Bir çok derleyicide bazı özyineleme türleri optimize edilir. Özellikle Haskell gibi fonksiyonel programlama dillerinde ise zaten prosödürel dillerden alıştığımız döngü kavramı olmadığından, özyineleme şart olmaktadır. Bu yazı C# özelinde ilerleyecektir ve C# dilinde genellikle özyineleme kullanımı "tehlikelidir". Bunun sebeplerine değineceğim .Şimdi aşağıdaki kodu inceleyin:

```csharp
    class Program
    {
        static void Main(string[] args)
        {
            For(100);
            Console.ReadLine();
        }

        private static void For(int hedef, int n = 0)
        {
            if (n == hedef)
            {
                return;
            }
            Console.WriteLine(n);
            For(hedef, ++n);
        }
    }
```
Yukarıdaki örnekte döngü kullanmadan n sayıda komut işletmiş olduk. Bu haliyle örneği çalıştırdığınız zaman çok güzel şekilde çalıştığını göreceksiniz. Fakat, `hedef` değerine `100` yerine `100000` yazarsak kod çalışmayacaktır. Tek göreceğimiz oldukça ünlü "System.StackOverflowException" hatası olacaktır. Eğer Visual Studio kullanıyorsanız tam bu esnada "Call Stack" pencersine göz atın. Binlerce kez bu metodun geçtiğini göreceksiniz. Hata mesajının bize dediği gibi, "call stack" için ayrılan bellek bölgemiz tükenmiş durumda. C# (CLR) bir metot başka bir metodu çağırdığı zaman parametre değerlerini korumak ve geriye dönüş yapmak için bu bellek bölgesini kullanacaktır. Biz iç içe çok fazla metot çağırdığımız için bu belleğin tükenmesine yol açmış oluyoruz. 

"StackOverflowException" hatasının ilginç bir yanı da bu hatanın `try/catch`bloğu ile yakalanmıyor olmasıdır. Uzak durmak için bir sebep daha!

Peki, nasıl kurtulacağız bu dertten :) yukarıdaki örneğimize bakın. Metodun ismini boşu boşuna `For` koymadım. Kurtulmanın yolu döngü mekanizmalarını akılcı olarak kullanmaktan geçiyor. Tüm özyinelemeli metotlar teorik olarak döngüler ile ifade edilebilirler. Örneğimizi düzelterek başlayalım:

```csharp
private static void For(int hedef, int n = 0)
{
    while (n != hedef)
    {
        Console.WriteLine(n);
        n = ++n;
    }
}
```
Bu kalıp bir çok özyinelemeli fonksiyonu döngü olarak çevirmemizi sağlamaktadır.  Öncelikle bir `while` döngüsü kuruyorum. Bu while döngüsü ile aslında metodun tamamını dönerek metot içindeki kodları yinelemiş oluyorum  Özyinelemeyi kıran şartımı bu döngüyü kıracak şekilde ayarlıyorum ve artık çok daha iyi bir koda sahibim. Burada aslında`for` döngüsü kurabilirdim fakat kalıbı anlatabilmek için bu şekilde ilerledim.

Bu yazıyı yazmama sebep olan forum sorusunda özyinelemeli olarak bir dosya listelemesi yapılıyordu. Bahsettiğim sorudaki koda oldukça benzeyen br örnek hazırladım. Bir klasör ve onun altındaki tüm klasörlerdeki dosyaları listeleyen şu örneği inceleyin:

```csharp
static void Main(string[] args)
{
    DosyaListele("c:\\c\\");
    Console.ReadLine();
}

public static void DosyaListele(string dizin)
{
    foreach (var dosya in Directory.GetFiles(dizin))
    {
        Console.WriteLine(dosya);
    }
    foreach (var altDizin in Directory.GetDirectories(dizin))
    {
        DosyaListele(altDizin);
    }
}

```

İlk döngüde `Directory.GetFiles` ile mevcut dizindeki dosyalar tek tek ekrana basılıyor. Daha sonra `Directory.GetDirectories`ile alt dizinler için metodun kendisi tekrar çağırılıyor.  Klasörlerin ağaç yapısı genellikle çok fazla dallanmadığı için aslında bu şekilde kod yazmanızın sıkıntı çıkartacağını düşünmüyorum. Ama biraz takıntılı davranalım ve bu kodu daha iyi hale getirelim. Önerim, aşağıdaki çözüme bakmadan kendiniz yapmanız yönünde olacaktır.

Çözdüyseniz veya vakit ayırmak istemediyseniz çözüme geçebilirim. Kalıbımızı kullanarak bu özyinelemeyi kaldırdığımızda kodlarımız aşağıdaki gibi olacaktır:

```csharp
public static void DosyaListele(string dizin)
{
    var yigin = new Stack<string>();
    yigin.Push(dizin);
    while (yigin.Any())
    {
        var mevcutDizin = yigin.Pop();
        foreach (var dosya in Directory.GetFiles(mevcutDizin))
        {
            Console.WriteLine(dosya);
        }

        foreach (var altDizin in Directory.GetDirectories(mevcutDizin))
        {
          yigin.Push(altDizin);
        }
    }
}
```
Kalıbımızda olduğu gibi yine bir `while` döngüsü açıyorum. Orijinal yöntemde özyinelemenin kırılması alt dizinlerin bitmesi ile gerçekleşiyordu. Alt dizinlerin sayısı da derine indikçe artıyordu. Öncelikle bu dizinleri tutacak bir diziye ihtiyacım var. Her döngüde bu diziden okuduğum bir kayıdı o diziden çıkartmam gerekecektir. Bu durumu benim için yapacak sınıflar .net de hali hazırda bulunuyor. Tek thread çalışacağım için Stack koleksiyonu benim için yeterli olacaktır. 

Özetlersek, özyinelemeli metotlar çok daha okunaklı kodlar yazmamızı sağlamaktadır. Özellikle "böl ve fethet" mantığındaki algoritmalarda işleri oldukça kolaylaştırmaktadır. Fakat bu yetenek beraberinde bellek problemleri oluşturabilir. Eğer okunabilirlik oldukça düşmüyorsa ve yinelemenin derinleşme ihtimali varsa C# için özyinelemeli işlemlerden uzak durmak güvenilir olacaktır. Performans açısından durum değişkenlik göstermektedir.