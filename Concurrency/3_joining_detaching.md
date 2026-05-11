# Join ve Detach Etme
- C++'ta eşzamanlı programlama söz konusu olduğunda, thread oluşturmak ve yönetmek işin sadece bir kısmıdır.
- Bir diğer konu ise bir thread görevini tamamladıktan sonra ne yapılacak ? 
- Threadin bitmesini beklemeli miyiz ? Yoksa bağımsız şekilde çalışmasına izin mi vermeliyiz ?
- İşte burada **joining** ve **detaching** devreye giriyor. Bu iki mekanizma, threadlerin çalışmasının tamamlandıktan sonra nasıl davranacağını kontrol ederler.

## Join Etmenin Temelleri

- Bir thread üzerinde `join()` çağırdığımızda, programa aslında şunu söylemiş oluyoruz:  
  *"Devam etmeden önce bu thread'in çalışmasını bitirmesini bekle."*

- Bu, thread tarafından hala kullanılıyor olabilecek kaynaklara erişim gibi sorunları önleyecektir.

### Ne Zaman `join()` Kullanılır ?
- Ana program devam etmeden önce bir threadin görevini tamamladığından emin olmamız gerekiyorsa `join()` kullanmalıyız.
- Bu özellikle şu durumlarda önemlidir:
   1) Thread paylaşılan veriyi güncelliyorsa
   2) Hesaplama yapıyorsa
   3) Kritik bir işi yürütüyorsa

```cpp
#include <iostream>
#include <thread>

void printNumbers() {
   for (int i = 0; i < 5; ++i) {
       std::cout << "Number: " << i << std::endl;
   }
}

int main() {
   std::thread t(printNumbers); // Yeni thread başlatıyoruz.
   t.join();                    // Threadin bitmesini bekliyoruz.

   std::cout << "Thread has finished execution." << std::endl;
   return 0;
}
```
Çıktı

```text
Number: 0
Number: 1
Number: 2
Number: 3
Number: 4
Thread has finished execution.
```
- Bu örnekte `main thread`, son mesajı yazdırmadan önce `t` threadinin çalışmasını bitirmesini bekler.
- Bu kritik bir noktadır. Çünkü eğer `t.join()` çağırmasaydık, ana program thread tamamlanmadan önce sonlanabilirdi.
- Yukarıdaki örnekte `t.join()` kısmını yorum satırına alıp kodu çalıştırdığımızda elde edilen çıktı:

```text
Thread has finished execution.
terminate called without an active exception
```
