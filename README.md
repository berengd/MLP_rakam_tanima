# MLP_rakam_tanima
MNIST veri seti kullanılarak el yazısı rakamları tanıyan bir Multi-Layer Perceptron (MLP) modelinin tasarımı, eğitimi ve performans analizi

Bu projede, MNIST veri seti kullanılarak evrişimli sinir ağları (CNN) kullanılmadan sıfırdan **Multi-Layer Perceptron (MLP)** modelleri geliştirilmiş, farklı hiperparametre ve mimari yapılandırmalarının performansları karşılaştırılmış ve eğitilen model gerçek dünyadaki el yazısı öğrenci numarası görüntüleri üzerinde test edilmiştir.

## 1. Veri Seti ve Ön İşleme
* **Veri Seti:** MNIST (28x28 piksel, gri tonlamalı, 0-9 arası 10 sınıf).
* **Train / Validation / Test Ayrımı:** Eğitim verisi olan 60.000 görüntü **48.000 Train (%80)** ve **12.000 Validation (%20)** olarak ayrılmıştır. **10.000 Test** görüntüsü model geliştirme sürecinde hiç kullanılmamış, yalnızca eğitim tamamlandıktan sonra nihai performans ölçümü için kullanılmıştır.
* **Normalizasyon:** Piksel değerleri `[0, 255]` aralığından `[0.0, 1.0]` aralığına ölçeklenmiştir. Görüntüler MLP girişine uygun olması için `784` elemanlı 1 boyutlu vektörlere düzleştirilmiştir (`Flatten`).

## 2. Deneyler ve Kullanılan Hiperparametreler

Toplam 3 farklı model yapılandırması denenmiştir:
* **Deney 1 (Temel MLP):** 2 gizli katman (`128 -> 64`), `ReLU` aktivasyonu, `CrossEntropyLoss`, `Adam` optimizer (`lr=0.001`), Batch Size: `64`, Epoch: `20`.
* **Deney 2 (Derin MLP + Dropout):** 3 gizli katman (`256 -> 128 -> 64`), `ReLU` aktivasyonu, katmanlar arası `Dropout(0.2)`, `Adam` optimizer (`lr=0.001`), Batch Size: `64`, Epoch: `10`.
* **Deney 3 (Optimizer Karşılaştırması - SGD):** Temel MLP mimarisi (`128 -> 64`), `ReLU` aktivasyonu, `SGD` optimizer (`lr=0.01`), Batch Size: `64`, Epoch: `10`.

## 3. Deney Sonuçları

| Deney | Train Accuracy | Validation Accuracy | Test Accuracy | Eğitim Süresi (sn) |
| :--- | :--- | :--- | :--- | :--- |
| **Deney 1 (Temel MLP - Adam)** | %99.91 | %97.77 | %97.73 | 169.68 sn |
| **Deney 2 (Derin MLP + Dropout)** | %98.69 | %97.83 | %97.85 | 103.13 sn |
| **Deney 3 (Temel MLP - SGD)** | %90.2 | %90.69 | %90.56 | 80.18 sn |

### Deneylerin Değerlendirilmesi
1. **Optimizer Etkisi (Adam vs. SGD):** Deney 1 ve Deney 3 karşılaştırıldığında, adaptif öğrenme oranı kullanan `Adam` optimizer'ın çok daha hızlı yakınsadığı, sabit öğrenme oranlı `SGD`'nin ise aynı sürede daha düşük doğrulukta kaldığı gözlemlenmiştir.
2. **Mimari Derinliği ve Dropout Etkisi:** Deney 2'de katman sayısı artırılırken eklenen `%20 Dropout` katmanı sayesinde modelin eğitim verisini ezberlemesi (overfitting) engellenmiş ve doğrulama/test kümesinde yüksek genelleme başarısı elde edilmiştir.

## 4. Kendi El Yazısı Öğrenci Numaram Üzerinde Test ve Değerlendirme

Kağıt üzerine yazılan 11 haneli öğrenci numaramın her bir rakamı fotoğraflanmış ve MNIST formatına uygun hale getirilmiştir:
1. Görüntüler gri tonlamaya (`grayscale`) çevrilmiştir.
2. MNIST formatına uyum için renkler ters çevrilmiştir (`invert`: siyah arka plan, beyaz rakam).
3. Kağıt gölgelerini temizlemek için eşikleme (thresholding) uygulanmış, rakamlar en-boy oranı korunarak ortalanmış ve `28x28` piksele dönüştürülmüştür.

[ogrenci_no_tahmin.png]

### Gerçek Dünya Verisi ve Distribution Shift Analizi
* **Sonuç:** Eğitilen model, 11 haneli el yazısı öğrenci numaramın 10 hanesini %90.91 doğrulukla tahmin etmiştir.
* **Hata Analizi (Distribution Shift):** Yalnızca `1` rakamı model tarafından `9` olarak tahmin edilmiştir. Bunun sebebi, MNIST veri setindeki yazım stili ile kendi el yazım arasındaki dağılım farkı (distribution shift), kağıt-ışık-kalem gibi fiziksel koşullardan kaynaklanan piksel kalınlığı farkları ve MLP mimarisinin uzamsal (spatial) özellikleri koruyamamasıdır.
