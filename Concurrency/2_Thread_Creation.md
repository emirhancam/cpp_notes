# Thread Oluşturma
- Threadlerin nasıl oluşturulup yönetileceğini anlamak, verimli ve tepki verebilir uygulamalar geliştirmek için önemlidir.
- Bu bölümde thread oluşturmanın farklı yollarını göreceğiz.

## Thread Oluşturmak için `std::thread` Kullanımı
- C++'ta thread oluşturmanın en temel yolu, C++ standart kütüphanesindeki `std::thread` sınıfını kullanmaktır.
- Bu sınıf; bir fonksiyonu, lambda ifadesini veya bir sınıf metodunu çalıştıran yeni bir thread oluşturmamızı sağlar.

```cpp
#include <iostream>
#include <thread>

void hello() {
    std::cout << "Hello from a thread! << std::endl;
}

int main() {
    std::thread t(hello);  // hello fonksiyonunu çalıştıran yeni bir thread oluştur.
    t.join();              // Thread'in bitmesini bekle.
    return 0;
}
```
Çıktı:
```text
Hello from a thread!
```

