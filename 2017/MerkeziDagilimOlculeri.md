---
Title: Veriyi Anlamak - Merkezi Dağılım Ölçüleri
PublishDate: 4/09/2017
IsActive: True
Tags: C#, Veri Madenciliği
---

Merkezi dağılma ölçüleri verinin düzlemde ortalamalarına göre nasıl dağıldıkları hakkında sayısal bilgilerdir. Şu iki dağılımı inceleyelim :

```csharp
var dizi1 = new[] { 5, 6, 14, 17, 18, 19, 20, 25, 26, 27, 29, 38, 80, 89, 95, 100 };
var dizi2 = new[] { 5, 25, 27, 28, 29, 30, 32, 33, 34, 35, 37, 38, 40, 45, 70, 100 };
```
Biraz yardımcı olması için sayı doğrusu üzerinde göstereyim:

![d1.png](media/merkezidagilim/d1.png)

Yeşil olarak gösterdiğim kısım her iki serininde arimetik ortalaması olan (38)in konumunu gösteriyor. Yani elimizdeki iki dizinin dağılımları farklı ama ortalamaları aynıdır. [Entropi kavramında](http://cihanyakar.com/entropi) serideki verilerin ne kadar tahmin edilemez dağıldığına bakmıştık. Bu sefer daha çok ortalamaya göre nasıl dağıldıklarını gösteren metrikleri anlatmaya çalışacağım. Tabi yine örnekleri C# üzerinde yapıyor olacağım.



### Değişim

Değişim ya da varyans (_Variance_) değeri her bir elemanın aritmetik ortalamaya olan farklarının karelerinin ortalamasıdır. Örneğin ` 1, 2, 3` serisinde aritmetik ortalama 2 dir. Sırayla elemanların bu değere uzaklıkları -1,0,1 dir. Bunların karelerini aldığımızda `1,0,1` değerlerini elde ederiz. Toplayıp (2) eleman sayısına (3) böldüğümüzde değişim sayısı `0.66` olarak çıkar. Eğer elimizdeki sayılar bir kitlenin içinden alınmış bir örnek ise (örneklem, _sample_) bu durumda eleman sayısının bir eksiğine bölmek gerekir.

Kitle(_population_) değişimi :
$$σ^2 = \frac{\sum(X_i - μ)^2}{n}$$

Örneklem (sample) değişimi :

$$s^2 = \frac{\sum(X_i - \overline{X})^2}{n-1}$$

Bunu koda dökelim :

```csharp
public enum VeriTur
{
	Kitle = 0,
	Ornek = 1
}

public static double Degisim(IList<double> dizi, VeriTur kitleTuru)
{
	var ortalama = dizi.Average();
	return dizi.Sum(x => Math.Pow(ortalama - x, 2)) 
			/ (dizi.Count - (int)kitleTuru);
}
```

Bunu her iki serimize _kitle_ olarak uyguladığımızda (_elimizdeki veri kullanacağımız verinin tamamı_) sonuçlar şu şekilde olacaktır :

```
dizi1 : ~1011
dizi2 : ~414
```
Buradan çıkartacağımız sonuç, dizi1'in elemanları dizi2'ye göre ortalamanın daha uzağına dağılmışlardır.



### Standart Sapma

Standart sapma  (_Standard Deviation_) varyansın kareköküdür. Değişim sayısında elde ettiğimiz değer gerçek ölçü birimi ne ise onun karesi olarak oluşmaktadır. Örneğimizdeki sayılar cm olursa sonuç cm² cinsinden olacaktır.

Kitle(_population_) standart sapması:
$$σ = \sqrt{\frac{\sum(X_i - μ)^2}{n}}$$

Örneklem (sample) değişimi :

$$s = \sqrt{\frac{\sum(X_i - \overline{X})^2}{n-1}}$$

Bunu koda dökelim :

```csharp
public enum VeriTur
{
	Kitle = 0,
	Ornek = 1
}

public static double StandartSapma(IList<double> dizi, VeriTur kitleTuru)
{
	var ortalama = dizi.Average();
	return Math.Sqrt(dizi.Sum(x => Math.Pow(ortalama - x, 2))
			/ (dizi.Count - (int)kitleTuru));
}
```
> Burada [ortalamalar yazımdaki](http://cihanyakar.com/ortalamalar) kareli ortalama formülüne bakacak olursanız aynı formülün kullanıldığını göreceksiniz. Standart sapma yapı olarak bir kareli ortalamalıdır. 


Yeni formülümüzü her iki serimize _kitle_ olarak uyguladığımızda (_elimizdeki veri kullanacağımız verinin tamamı_) sonuçlar şu şekilde olacaktır :

```
dizi1 : ~32
dizi2 : ~20
```

Buradan değişimde çıkardığımız sonuca ek olarak şu fikre de ulaşırız. Dizi1 için ortalamanın 32 altı ve 32üstüne kadar olan bölge bu dizi için normal, altında ve üstünde kalan bölgeler normal dışı (çok küçük, çok büyük) kabul edilebilir. Aynı cümleyi dizi2 için 20 olarak kurabiliriz. 



### Değişim Katsayısı

Değişim katsayısı veya bağıl standart sapma (_Coefficient Of Variation_ , _Relative Standard Deviation_); standart sapmanın aritmetik ortalamaya bölümü ile elde edilir.
Standart sapma bize tek bir dizi ilgili fikir verirken farklı dizilerin karşılaştırılmasında pek fazla fikir verememektedir. Örneğin, iki farklı işletmenin cirolarındaki değişkenliği karşılaştırmak isteyelim. Birisi mahalle bakkalı diğeri büyük market olsun. Doğal olarak market daha düzenli bile gitse standart sapması bakkala oranla çok yüksek olacaktır. Şöyle iki dizimiz olsun.

```csharp
var dizi1 = new double[] { 5, 6, 14, 17, 18, 19, 20, 25, 26, 27, 29, 38, 80, 89, 95, 100 };
var dizi2 = new double[] { 50, 250, 270, 280, 290, 300, 320, 330, 340, 350, 370, 380, 400, 450, 700, 1000 };
```
Fark ettiyseniz, ikinci dizi ilk örneğimizdekinin sadece 10 ile çarpılmış hali.

Bu ikisinin standart sapmalarını aldığımızda, sonuçlar aşağıdaki gibi çıkacaktır.

```
dizi1 : ~32
dizi2 : ~204
```
Bu durumda dizi1 daha dizi2'den daha az değişkenlik gösterir veya göstermez diyemeyiz. Bu ikisini standart bir hale getirmek gerekir.  Fark ettiyseniz her bir elemanın 10 kat arttırılması sonucu da 10 kat arttırmıştır. Bu durumda her iki standart sapmayı bu etkiden kurtarırsak o zaman değişimi gösterecek bir sonuç elde ederiz. O da aritmetik ortalamadır. 

$$\%değişim = \frac{σ}{\overline{X}}$$

C# dilinde yazacak olursak:

```csharp
public static double Degisim(IList<double> dizi, VeriTur kitleTuru)
{
	var ortalama = dizi.Average();
	return (Math.Sqrt(dizi.Sum(x => Math.Pow(ortalama - x, 2))
			/ (dizi.Count - (int)kitleTuru))) / ortalama;
}
```
Sonuçlar:
```
dizi 1: ~0.84
dizi 2: ~0.54
```
Çarpmanın etkisinden ayırdığımız sonuçlarda dizi2 deki değişimin %54 olduğunu ve ortalamayı tutturma konusunda daha tutarlı olduğunu söyleyebiliriz.

