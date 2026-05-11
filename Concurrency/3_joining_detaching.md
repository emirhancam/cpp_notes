# Join ve Detach Etme
- C++'ta eşzamanlı programlama söz konusu olduğunda, thread oluşturmak ve yönetmek işin sadece bir kısmıdır.
- Bir diğer konu ise bir thread görevini tamamladıktan sonra ne yapılacak ? 
- Threadin bitmesini beklemeli miyiz ? Yoksa bağımsız şekilde çalışmasına izin mi vermeliyiz ?
- İşte burada **joining** ve **detaching** devreye giriyor. Bu iki mekanizma, threadlerin çalışmasının tamamlandıktan sonra nasıl davranacağını kontrol ederler.

## Join Etmenin Temelleri

- Bir thread üzerinde `**join()**` çağırdığımızda, programa aslında şunu söylemiş oluyoruz:  
  *"Devam etmeden önce bu thread'in çalışmasını bitirmesini bekle."*

- Bu, thread tarafından hala kullanılıyor olabilecek kaynaklara erişim gibi sorunları önleyecektir.




