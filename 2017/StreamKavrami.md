---
Title: C# örnekleri ile Stream Kavramı
PublishDate: 18/11/2017
IsActive: True
Tags: C#
---



Stream (Akış) verinin bir bütün değil de parça parça alınması,işlenmesi olarak düşünülebilir. Pikap,cd, dvd,radyo... gibi düşünebilirsiniz. Okuyucu kafa yalnızca bulunduğu konumdaki veriden küçük parçalar okur veya yazar.  Akışlar geliştirdiğimiz hemen her programda bulunurlar. Biz doğrudan kullanmasak da ihtiyaç duyduğumuz kütüphaneler bunları sürekli kullanmaktadırlar. Internet üzerinden bir dosya indirirken bu dosya bize kalıp halinde değil de işleyip birleştirmemiz gereken parçalar haline gelecektir. Yine sabit diskten bir dosya okurken bu dosya aslında bir akış biçiminde parça parça gelmektedir. 

Akışlar farklı alt türlere ayrılabilir. Canlı bir yayın akışında akış üzerinde konum değiştirmek mümkün değilken, sabit diskten okunan bir video dosyasında konum değiştirmek basit bir iştir. Yine akışlar yönlerine göre de üçe ayrılabilirler. Bir akış sadece yazma yönündeyken diğeri okuma yönünde olabilir.  Ya da her ikisi birden olabilir.

Framework içerisinde çoğu akış türü soyut [**Stream**](https://msdn.microsoft.com/en-us/library/system.io.stream%28v=vs.110%29.aspx) sınıfından türemişlerdir. Bazıları ise sadece arka taraftaki bir akış için sarmalayıcı yapıdadır.  Öncelikle bu sınıfa ait önemli metot ve özelliklere bakalım:

### Özellikler
**Position** : Yazının en başında akışları pikap,cd gibi düşünebilirsiniz demiştim. Bu özellik pikapın iğnesinin pikap üzerindeki konumuna karşılık gelmektedir.

**CanRead, CanWrite, CanSeek, CanTimeout** : Akışın sunduğu yetenekleri belirtirler. Okunabilir mi, yazılabilir mi, atlana bilir mi (akışın konumu değiştirilebilir mi), zaman aşımı uygulanabilir mi?

**Length** : Akışın uzunluğunu belirtir. Her akış için geçerli değildir. Canlı bir yayında veya bir cihaz ile bağlantı kurulduğunda bir boyuttan söz edilemez.

**ReadTimeout,WriteTimeout** : Milisaniye cinsinden zaman aşımı sürelerini belirtir.



### Metotlar
Sadece önemli olanları açıklayacağım:

**Read :** Belirtilen byte kadar bilgiyi parametre olarak verilen byte dizisine koyar. Geriye kaç byte okuyabildiyse onu döner. 105byte'lık bir akışı döngü ile 50şer byte ile okuduğumuzu varsayalım. İlk iki dönüşte 50 yanıtını alırken son turda 5 yanıtını alırız. 

**ReadByte :** Tek bir byte okur ve sıradakine geçer. **Geri dönüş türü adında olduğu gibi byte değil int'dir.**

**Write :** Belirtilen diziden belirtildiği kadar byte'ı akışa yazar ve konumu yazılan byte kadar ilerletir.

**WriteByte :** Tek bir byte yazar ve akış bir byte kayar.

**Seek :** Akışın konumu değiştirilir.

**Flush :** Bazı akışlar tampon bellek kullanabilir. Bu durumda yazılanlar akıştan önce bir tamponda tutulur. Bu metot çağrıldığında tampon bellekteki tüm veri doğrudan akışa yazılır.

**Close :** Akış kapatılır. (Tekrar açmayacaksınız `Dìspose` kullanın)



### MemoryStream
Anlaşılması en kolay akışlardandır. Genellikle iki farklı akış arasında köprü görevi kurmak için kullanılır.

```csharp
 var random = new Random(1300);
 using (var ms = new MemoryStream(10))
 {
     for (var i = 0; i < ms.Capacity; i++)
     {
         ms.WriteByte((byte)random.Next(0, 255));
     }
     
     ms.Seek(0, SeekOrigin.Begin); //ms.Position = 0;

     for (var i = 0; i < ms.Capacity; i++)
     {
         Console.WriteLine(ms.ReadByte());
     }
 }
```
Örnek kodda 10 adet rastgele sayı üretilip belleğe koyulmakta ve akış başa alınıp tekrar okunmakta. Dikkat ederseniz `Capacity` adında yeni özellik çıktı karşımıza. `Length` kaç eleman olduğunu söylerken, `Capacity` ise kaç eleman alınabileceğini söylemektedir.

### FileStream
Adı üzerinde, dosyalar ile akış işlemleri yapmamızı sağlar. Adım adım basit bir okuma örneği yapalım:

```csharp
 using (var fs = new FileStream(@"c:\cihan\test.txt", FileMode.Open))
 {
     for (int i = 0; i < fs.Length; i++)
     {
         Console.Write(fs.ReadByte());
     }
 }
```
Tek tek byteları okuyup ekrana basmaya çalıştım ve ekranım sayılarla doldu... Bu sebeple bu byteları öncelikle karakterlere dönüştürmem lazım:

```csharp
using (var fs = new FileStream(@"c:\cihan\test.txt", FileMode.Open))
{
    for (int i = 0; i < fs.Length; i++)
    {
        Console.Write((char)fs.ReadByte());
    }
}
```
Oldu gibi fakat **bu yöntem metin dosyaları için yalnızca ANSI/ASCII encodingde (her harf bir byte) çalışacaktır**.  Bu sebeple bu konuda yardımcı olacak sınıfları kullanmamız gerekecek. Onları inceleyeceğiz.

Okuduğumuz küçük bir metin dosyası değil de büyük binary bir dosya olsaydı bu durumda bir byte bir byte okumaya çalışmak zaman kaybı oluştururdu.  Bunun yerine dosyayı parçalar haline okumak ve yazmak önemli olacaktır. Basit bir dosya kopyalayıcı yazalım :

```csharp
var kaynakDosya = @"c:\cihan\test.txt";
var hedefDosya = @"c:\cihan\test2.txt";

using (var fsOkuyucu = new FileStream(kaynakDosya, FileMode.Open))
using (var fsYazici = new FileStream(hedefDosya, FileMode.CreateNew))
{
    fsYazici.SetLength(fsOkuyucu.Length);
    byte[] tampon = new byte[2048];
    int okunanByte = 0;
    while ((okunanByte = fsOkuyucu.Read(tampon, 0, tampon.Length)) > 0)
    {
        fsYazici.Write(tampon, 0, okunanByte);
    }
}
```
Adım adım ne yaptığımıza bakalım:
```csharp
using (var fsOkuyucu = new FileStream(kaynakDosya, FileMode.Open))
using (var fsYazici = new FileStream(hedefDosya, FileMode.CreateNew))
```
Okuma ve yazma işlemleri için ayrı akışlar açtık. 
```csharp
fsYazici.SetLength(fsOkuyucu.Length);
```
Elimizdeki akışın boyutu bilindiği için hedef akışın boyutunu da belirttik. Bu satır işlemi yapabilmemiz için şart değil, fakat işletim sistemine dosya boyutunu söylediğimiz için bizim için bu alanı tutacaktır. Özellikle büyük boyutlu dosyalarda işe yaramaktadır.

```csharp
byte[] tampon = new byte[2048];
```
2048 bytelık bir tampon bölge oluşturuyoruz. Dosya isterse 1TB olsun bizim bellek kullanımımız 2048'i aşmayacak.

```csharp
int okunanByte = 0;
    while ((okunanByte = fsOkuyucu.Read(tampon, 0, tampon.Length)) > 0)
```
Read metodu geriye kaç byte okumuşsa onu döndürdüğünden bahsetmiştik. Eğer bu değer 0 gelirse dosyanın sonuna ulaşmış oluyoruz. Kaç byte okunduğunu not alıyoruz çünkü dosyanın boyutu 2048'in katı olmak zorunda değil. Son döngüde kaç byte okuyabilmişsek o kadar yazacağız.

```csharp
 fsYazici.Write(tampon, 0, okunanByte);
```
 tampona koyduğumuz bytelardan okunanByte kadarını okuyuruz. 0 ne işe yarıyor diye sorarsanız, kaçıncı bytedan itibaren tampondan okunacağını belirtiyor. 

### Socket / NetworkStream
Soket kavramı bir ağ üzerinde erişimi anlatır. Bu ağ ve kullanılan protokoller çok farklı olabilmektedir. Socket'in kendisi Stream türünden kalıtım almamıştır. Fakat davranış şekilleri akışlara oldukça benzemektedir ki aslında arkaplanda akışlar kullanılmaktadır. NetworkStream ise bu yapıyı bildiğimiz akış yöntemleri ile kullanabilmemizi sağlar. Eğer TCP,UDP gibi bilindik protokolleri kullanacaksanız bu için özelleştirilmiş TCPClient gibi sınıfları kullanabilirsiniz. Biraz örnek yapalım, ilk iki örnek doğru kodu bulmak adına yazdığımız hatalı kodlar olacak aman kullanmayın :) :

```csharp
 	var adres = @"www.cihanyakar.com";
	var ipAdresi = Dns.GetHostEntry(adres).AddressList.First();
	
	var hedef = new IPEndPoint(ipAdresi, 80);
	using (var socket = new Socket(hedef.AddressFamily, SocketType.Stream, ProtocolType.Tcp))
	{
	    socket.Connect(hedef);
	    string istek = string.Join("\r\n",
	                        @"GET / HTTP/1.1",
	                        $"Host: {adres}",
	                        "Connection: Close", "", "");
	
	    byte[] istekByte = Encoding.UTF8.GetBytes(istek);
	
	    byte[] yanitTampon = new byte[2048];
	    int gelenBoyut = 0;
	    socket.Send(istekByte, istekByte.Length, SocketFlags.None);
	    do
	    {
	        gelenBoyut = socket.Receive(yanitTampon, yanitTampon.Length, 0);
	        Console.Write(Encoding.UTF8.GetString(yanitTampon, 0, gelenBoyut));
	    }
	    while (gelenBoyut > 0);
	
	    socket.Disconnect(false);
	}
```

bu örnekte `stream` e benzer metotlar kullandık fakat `write` yerine `send`, `read` yerine `receive` isimli metotlar kullandık. Aynı kodu sarmalayıcı bir stream kullanarak yazalım:

```csharp
var adres = @"www.cihanyakar.com";
var ipAdresi = Dns.GetHostEntry(adres).AddressList.First();

var hedef = new IPEndPoint(ipAdresi, 80);
using (var socket = new Socket(hedef.AddressFamily, SocketType.Stream, ProtocolType.Tcp))
{
    socket.Connect(hedef);

    using (var ns = new NetworkStream(socket))
    {
        string istek = string.Join("\r\n",
                                   @"GET / HTTP/1.1",
                                   $"Host: {adres}",
                                   "Connection: Close", "", "");

        byte[] istekByte = Encoding.UTF8.GetBytes(istek);
        ns.Write(istekByte,0,istekByte.Length);
        ns.Flush();

        while (!ns.DataAvailable)
        {
            Thread.SpinWait(1);
        }

        byte[] yanitTampon = new byte[2048];
        int gelenBoyut;

        while((gelenBoyut = ns.Read(yanitTampon, 0,yanitTampon.Length)) > 0)
        { 
            Console.Write(Encoding.UTF8.GetString(yanitTampon, 0, gelenBoyut));
        }
       
    }
}
```

İlk örnek saf Socket sınıfını kullanırken, ikinci örnekte NetworkStream kullanarak işi bildiğimiz yapıya çeviriyoruz. Fakat UTF8 konusunda yaptığımız işlem doğru değil. **UTF8 de harfler birkaç byte dan oluşabilir. Bu yöntemde tam birleştirme noktasına böyle bir harf gelirse bozulma olacaktır.** Kodumuzu şu hale getiriyoruz :

```csharp
var adres = @"www.cihanyakar.com";
var ipAdresi = Dns.GetHostEntry(adres).AddressList.First();

var hedef = new IPEndPoint(ipAdresi, 80);
using (var socket = new Socket(hedef.AddressFamily, SocketType.Stream, ProtocolType.Tcp))

{
    socket.Connect(hedef);

    using (var ns = new NetworkStream(socket))
    using (var sw = new StreamWriter(ns, Encoding.ASCII))
    using(var sr = new StreamReader(ns, Encoding.UTF8))
    {
        string istek = string.Join("\r\n",
                                   @"GET / HTTP/1.1",
                                   $"Host: {adres}",
                                   "Connection: Close", "", "");
    
        sw.Write(istek);
        sw.Flush();
        while (!ns.DataAvailable)
        {
            Thread.SpinWait(1);
        }
    
        char[] yanitTampon = new char[1024];
        int gelenBoyut;
    
        while ((gelenBoyut = sr.ReadBlock(yanitTampon,0, yanitTampon.Length)) > 0)
        {
            Console.Write(new string(yanitTampon,0,gelenBoyut));
        }
    
    }
 }
```

Artık gönderme ve alma işlemlerinde yardımcı sınıflar kullandık.  `while (!ns.DataAvailable)` kısmında verinin sunucudan geldiğini garanti altına almaya çalışıyorum.



### StreamReader, StreamWriter

 Kendileri stream olmasalar da özellikle akıştaki veri eğer metin ise kullanmamız işimizi kolaylaştırmaktadırlar.  Örneğin UTF-8 de ilk okuduğunuz byte değeri 126 nın üzerindeyse karakteri oluşturmak için ikinci byte bakmak zorunda kalırsınız bu byte da toplam 2046 nın üzerine çıkınca üçüncü byte işin içine girer 65534 un üzerine çıkarsanız dördüncü byte'ı da işin içine sokmak gerekir. Bu sebeple tampon belleğim bir karakteri ortadan kesebilir. Bu da istenmeyen sonuçların gösterilmesine veya gönderilmesine neden olabilir. Bunları bizim için kolay ve hızlı halleden StreamReader ve StreamWriter byte okuması yerine karakter karakter okuma yapar.  Örnek kod bir önceki başlıkta mevcut inceleyebilirsiniz.



### Console / Process

 Bizim en bildiğimiz akış ise şüphesiz `Console` dur diye tahmin ediyorum :). Aslında Console bir static sınıftır ve farklı streamler için sarmalayıcı görevi görür.  Console içinde üç temel akış vardır, Input, Output ve Error. Örneğin output akışını ele alalım:

```csharp
Console.WriteLine("Merhaba");
```

Metodu aslında

```csharp
Console.Out.WriteLine("Merhaba");
```

olarak da yazılabilir.  Aslında In ve Out özellikleri bir çeşit StreamReader/Writer ikilisidir. Bu reader ve writer'ın eriştiği akışlara bizde erişebiliriz. Bunun için kodu şöyle de yazabiliriz:

```csharp
 var cikisAkisi = Console.OpenStandardOutput();
 using (var sw = new StreamWriter(cikisAkisi))
 {
     sw.WriteLine("Merhaba");
 }
```

Bir Windows/DOS programı dış dünya ile iletişim için bu akışları kullanabilir. Kendi çağırdığımız bir process'e buradan komut gönderebilir ve çıktısını okuyabiliriz. Örneğin ping işlemini çalıştıralım ve çıktısını kendi programımız üzerinden yapalım:

```csharp
var islem = new Process
            {
                StartInfo = new ProcessStartInfo
                            {
                                FileName = "ping",
                                Arguments = "google.com",
                                RedirectStandardOutput = true,
                                UseShellExecute = false
                            }
            };

islem.Start();
string outputstring;
using (var sr = islem.StandardOutput)
{
    outputstring = sr.ReadToEnd();
}
Console.WriteLine(outputstring);


Console.ReadLine();
```

Benzer şekilde input stream üzerinden eğer program giriş kabul ediyor olsaydı gönderebilirdik. FFMPEG gibi bazı programlar çıkışları StandardOutput yerine StandardError üzerinden de yapabilirler.

Başka bir yazıda görüşmek üzere.
