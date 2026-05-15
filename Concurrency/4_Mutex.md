# Mutex'ler
- Mutexler, C++'ta eş zamanlı programlamanın temel parçalarından biridir.
- Paylaşılan kaynakları aynı anda birden fazla threadin erişimine karşı koruyan mekanizmalardır.
- Böylece **race condition** oluşmasını önlemeye yardımcı olur.
- Aynı anda yalnızda bir threadin bir kaynağa erişmesini sağlamamız gereken durumlarla karşılaşabiliriz.
- İşte burada **mutex** devreye girer.

## Mutex Nedir?
- **Mutual Exclusion** ifadesinin kısaltılmışıdır.
- Bir mutex, aynı anda yalnızca bir threadin belirli bir kaynağa erişmesine izin veren bir nesne diyebiliriz.
- Bunu bir odanın anahtarı gibi düşünelim:
   *Eğer anahtar bir kişideyse, o kişi anahtarı bırakıp odadan çıkana kadar başka kimse o odaya giremez*

- Mutex kullandığımızda, kodumuzdaki kritik bölümlerin, yani paylaşılan kaynaklara erişen kısımların, aynı anda yalnızca bir thread tarafından çalıştırılmasını sağlayabiliriz.
- Böylece race condition problemlerinin önüne geçebiliriz.
- Mutex nasıl oluşturulur ve nasıl kullanılır ona bakalım:

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx; //Paylaşılan kaynağı korumak için mutex
int sharedResource = 0;

void incrementResource() {
    mtx.lock();  // Paylaşılan kaynağa erişmeden önce mutexi kilitleyelim.
    ++sharedResource;
    mtx.unlock();  // İşlem bittikten sonra mutexin kilidini açalım. 
}

int main() {
    std::thread t1(incrementResource);
    std::thread t2(incrementResource);

    t1.join();
    t2.join()

    std::cout << "Final value of sharedResource: " << sharedResource << std::endl;

    return 0;
}
```
Çıktı:
```text
Final value of sharedResource: 2
```

- Bu örnekte, 2 thread tarafından artırılan basit bir paylaşılan kaynak var: `sharedResource`
- Threadler bu değişkene erişmeden önce mutex'i kilitliyoruz, işlem bittikten sonra kilidi açıyoruz. Böylece artırma işleminin aynı anda iki thread tarafından bozulmadan yapılmasını sağlıyoruz.
- İleriki aşamada şunu göreceğiz: Gerçek projelerde `lock()` ve `unlock()` doğrudan elle kullanılmaz. Daha güvenli olması için genelde `std::lock_guard` tercih edilir.

## Neden Mutex Kullanıyoruz ? 
- Çok threadli herhangi bir uygulamada mutex kullanmak birkaç önemli nedenden dolayı kritiktir:
  **1) Race Conditionları önlemek** : Birden fazla thread aynı anda paylaşılan veriye erişip onu değiştirdiğinde, threadler birbirinin işlemini bozabilir. Bu da tutarsız veya geçersiz veriye yol açabilir.  Mutexler, bu veriye erişimi sıraya koyarak race condition oluşmasını önlemeye yardımcı olur.
  **2) Veri bütünlüğünü korumak** : Mutexler, paylaşılan kaynaklara erişimi kontrol ederek verinin geçerli bir durumda kalmasını sağlar. Örneğin, iki thread aynı anda paylaşılan bir değişkene yazmaya çalışırsa, mutex olmadan bozulmuş bir değerle karşılaşabiliriz.
  **3) Deadlock önleme konusunda farkındalık sağlamak** : Mutexler dikkatli yönetilmezse deadlock'a yol açabilir. Ancak mutex kullanımını iyi anlamak, 2 veya daha fazla threadin sonsuza kadar birbirini beklediği durumları önleyecek sistemler tasarlanmasına yardımcı olur.

- Şimdi mutexlerin race conditionları nasıl etkili şekilde önleyebileceğini gösteren bir örneğe bakalım:
```
#include <iostream>
#include <thread>
#include <mutex>
#inclute <vector>

std::mutex mtx;
std::vector<int> sharedData;

void addToSharedData(int value) {
    mtx.lock();
    sharedData.push_back(value);
    mtx.unlock();
}

int main() {
    std::thread threads[10];

    for (int i = 0; i < 10; ++i) {
        threads[i] = std::thread(addToSharedData, i);
    }

    for (auto& th : threads) {
        th.join();
    }

    std::cout << "Shared data contains:";
    for (const auto& val : sharedData) {
        std::cout << " " << val;
    }
    std::cout << std::endl;
    return 0;
}
```

Çıktı:
```text
Shared data contains: 0 1 2 3 4 5 6 7 8 9
```

- Mutexli iki satırı kaldırdığımızda yani `//mtx.lock();` ve `//mtx.unlock();` elde ettiğimiz çıktı ise: *0 yok, 2 sonda. Her kod çalıştığında farklı çıktı*

```text
Shared data contains: 1 3 4 5 6 7 8 9 2
```

- Gördüğümüz üzere eğer mutex kullanılmasaydı, `push_back()` işlemi race conditiona açık hale gelirdi. Bu da programın çökmesine, yanlış veri oluşmasına veya beklenmeyen sonuçlara neden olurdu. 

## C++'ta Mutex Türleri
- C++, standart kütüphanesinde farklı senaryolara uygun birkaç mutex türü sunar.

### std::mutex
- Şuana kadar gördüğümüz temel mutex türü  `std::mutex`'tir
- Bu, basit ve **recursive olmayan** bir mutextir. Threadler tarafından kilitlenebilir.
- Eğer bir threadi zaten kilitli olan mutex'i kilitlemeye çalışırsa, mutex'i alana kadar bekler.
- Yani `mtx.lock();` çağrısı yapıldığında mutex boşta değilse, thread orada bloklanır.

### std::recursive_mutex
- `std::recursive_mutex` , aynı threadin aynı mutexi birden fazla kez kilitlemesine izin verir ve deadlock oluşmasını engeller.
- Bu özellikle şu durumda işe yarar: Bir fonksiyon mutexi kilitler, sonra başka bir fonksiyonu çağırır. Çağrılan fonksiyonda aynı mutexi tekrar kilitlemeye çalışır.
- Normal `std::mutex` kullanılsaydı, aynı thread kendi kilitlediği mutexi tekrar almaya çalıştığı için deadlock oluşabilirdi.

Örnek:
```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::recursive_mutex rmtx;

void recursiveFunction(int count) {
    if (count <= 0) {
        return;
    }

    rmtx.lock();

    std::cout << "Locking count: " << count << std::endl;

    recursiveFunction(count - 1);

    rmtx.unlock();
}

int main() {
    std::thread t1(recursiveFunction, 3);
    t1.join();
    return 0;
}
```

Çıktı:
```text
Locking count: 3
Locking count: 2
Locking count: 1
```
- Bu örnekte `recursiveFunction`, aynı mutexi birden fazla kez kilitliyor.
- Normal `std::mutex` burada deadlock'a sebep olurdu. Çünkü fonksiyon kendisini tekrar çağırdığında aynı mutexi tekrar kilitlemeye çalışacaktı.
- Ama `std::recursive_mutex`, aynı threadin aynı mutexi tekrar tekrar kilitlemesine izin verdiği için bu durumu düzgün şekilde yönetir.
- Yine de küçük not: `recursive_mutex` her zaman ilk tercih olmamalı. Çoğu durumda kod tasarımını sadeleştirip normal `std::mutex` kullanmak daha sağlıklıdır.
- `resursive_mutex`, gerçekten iç içe kilitleme ihtiyacı varsa düşünülmelidir. 

























