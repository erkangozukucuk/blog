---
Title: Semaphore
PublishDate: 5/11/2018
IsActive: False
Tags: C#
---

Threadler ya da Türkçe tabir ile "iş parçacıkları" bir uygulamanın (process) iş yapan birimleridir. Bir uygulamada en az bir adet thread olması gerekir. Bu thread genellikle "main thread" olarak isimlendirilir ve bu thread sonlandığında uygulamanın sonlaması beklenir. Threadler işletim sistemi tarafından önceliklerine göre belirli bir süre çalıştırılıp, dondurulurlar. Bu döngüye "spin" adı veriliyor. Bu işlem çok hızlı gerçekleştiği için bize görsel olarak sanki aynı zamanda birden fazla iş yapıldığı ilüzyonu oluşturur. Eğer birden fazla işlemci veya işlemci çekirdeği varsa bu gerçek anlamda işlemci çekirdeği sayısı kadar olacaktır.

Eğer tek çekirdekli işlemciye sahip makinede çalışıyorsak, teknik olarak birden fazla thread ile çalışmak çok mantıklı gelmeyebilir. Word gibi metin işlemci uygulamayı düşünün. Yazım denetimi siz yazarken çalışıyor olsaydı, muhtemelen uygulamanız sürekli kilitleniyor olacaktır. Bunun yerine yazım denetimi main thread haricinde düşük öncelikli bir thread olarak ayarlanırsa, yazım denetimi çok daha kısa süreler için çalıştırılacağından uygulama daha rahat çalışacaktır. Daha vurucu bir örnek vermek gerekirse, bir iletişim soketini dinleyen bir uygulamayı düşünün sokete bir ileti geldiğinde bunu kullanıcıya göstermesi gereksin. Bu uygulama eğer tüm zamanını soketi dinlemek için harcarsa uygulama kullanılamaz bir hale gelecektir. Bunu çözmek adına soket işlerini yapan kısım farklı bir threade taşınabilir. Burada "her iki örnek için thread açmaya gerek yok" diyenler olacaktır ve aslında haklılarda bu işi main thread için zaman paylaşımı yaparak çözmek mümkün olacaktır, fakat C# gibi üst seviye bir dilde bunları tek thread ile çözmeye çalışmak oldukça zahmetli olacaktır.

Main thread aynı zamanda işletim sistemi ile haberleşmeyi sağladığı için eğer bu arkadaş meşgul edilirse, işletim sisteminin emirlerini işlemekte sıkıntı çekebilir. Bunu denemek için bir button arkasında sonsuz döngü oluşturabilirsiniz. Düğmeye tıklanıldığında uygulama üzerindeki diğer nesneleri kullanamadığınızı ve uygulamayı hareket bile ettiremediğinizi göreceksiniz. Bir süre sonra işletim sistemi uygulamaya komutları işlemediği için uygulamanın "yanıt vermediği" yönünde sizi bilgilendirecektir. Bu sonsuz döngüyü bir başka thread oluşturup çalıştırsaydık veya döngü içinde thread içindeki diğer işlerin yapılması için (doevents) bir bekleme yapsaydık uygulamamız kilitlenmeyecekti.

Buraya kadar aslında tek işlemci, tek çekridek durumuna bakmış olduk. Tek çekirdek varken birden fazla thread kullanımında genellikle amacımız uygulamamızın akıcı olmasıdır ve açıkçası pek problem yaşamayız. Fakat günümüzde tek çekirdekli işlemci arasanız bulamayabilirsiniz. Birden fazla çekirdek olan durumda uygulamamız neden tek bir çekirdeğe hapsolsun ki? Word örneğinde yazım denetimi ben yazarken dahi çalışacaktır. Asıl önemli olan veri işlemede bize kazandırdığı performans artışıdır. Örneğin, bir sayı dizindeki tüm sayıların üç katını alacak bir döngümüz olsun.

```csharp
	var dizi = new[] {1,2,3,4,5,6,7,8,9,10,11,12};
	
	for (int i = 0; i < dizi.Length; i++)
	{
		dizi[i] *= 3;
	}
```

Bu kodda yer alan çarpma işlemi için 1 spin gerektiğini düşünelim, bu durumda işlem 12 işlemci döngüsünde gerçekleşecektir. İki çekirdekli bir işlemcimiz olduğu durumda eğer çarpma işlemlerini üleştirirsek bu durumda 12 / 2 = 6 döngüde aynı işlemi gerçekleştirmiş olabiliriz. Günümüzde 4 çekirdek kullanımı neredeyse standart olduğuna neden cpunun kalan 3/4nü kullanmayalım? C# buna değişime TPL, Parallel Linq, async/await gibi yapılar ile uyum sağlamaktadır. GO gibi daha güncel diller ise doğrudan bu çok çekirdekli dünyaya uygun olarak geliştirilmekteler.

Bu yazıda C# için her şeyin kökü olan Threading kütüphanesindeki Thread sınıfını örnekleyeceğim. Peşinden gelecek yazılarda işin çetrefilli yollarına dalacağız.


