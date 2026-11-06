# Dekoratorlar-yordamida-funksiyalar-to-plami
Python tilida funksiyalarni o'zgartirmasdan ularga qo'shimcha imkoniyatlar (vaqtni o'lchash, logga yozish va xatolik yuz berganda qayta ishga tushirish) qo'shuvchi kichik utilita moduli.

Quyida talablarga to'liq javob beradigan kod keltirilgan:

```python
import time
import functools

# 1. Timer dekoratori: bajarilish vaqtini o'lchaydi
def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.perf_counter()
        result = func(*args, **kwargs)
        end_time = time.perf_counter()
        elapsed_time = end_time - start_time
        print(f"[TIMER] '{func.__name__}' funksiyasi {elapsed_time:.6f} sekundda bajarildi.")
        return result
    return wrapper

# 2. Log calls dekoratori: argumentlar va qaytarilgan natijani chop etadi
def log_calls(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"[LOG] '{func.__name__}' chaqirildi. Argumentlar: args={args}, kwargs={kwargs}")
        result = func(*args, **kwargs)
        print(f"[LOG] '{func.__name__}' natijasi: {result}")
        return result
    return wrapper

# 3. Retry dekoratori (dekoratorlar fabrikasi): istisno bo'lganda N martagacha qayta ishga tushiradi
def retry(times):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            last_exception = None
            for attempt in range(1, times + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    last_exception = e
                    print(f"[RETRY] '{func.__name__}' xatolik berdi ({e}). Urinish {attempt}/{times}...")
            print(f"[RETRY] '{func.__name__}' barcha {times} ta urinishda ham muvaffaqiyatsiz tugadi.")
            raise last_exception
        return wrapper
    return decorator


# --- DEMO VA TEKSHIRUV ---

if __name__ == "__main__":
    
    # 1-namuna: Bir nechta dekoratorni birgalikda qo'llash
    @timer
    @log_calls
    def multiply_numbers(a, b):
        """Ikki sonni ko'paytiradi va sun'iy kechikish qo'shadi."""
        time.sleep(0.2)  # Vaqtni o'lchashni ko'rish uchun kechikish
        return a * b

    print("--- 1-NAMUNA: Oddiy funksiya (timer va log_calls) ---")
    # __name__ va __doc__ saqlanganini isbotlash
    print(f"Funksiya nomi (__name__): {multiply_numbers.__name__}")
    print(f"Hujjat (__doc__): {multiply_numbers.__doc__}")
    print("-" * 50)
    
    result = multiply_numbers(6, 7)
    print(Yakuniy natija: {result}\n)


    # 2-namuna: retry dekoratorini sinab ko'rish
    attempt_counter = 0

    @retry(times=3)
    @timer
    def unstable_operation(x):
        """Ba'zida xato beruvchi, sinov uchun mo'ljallangan funksiya."""
        nonlocal attempt_counter
        attempt_counter += 1
        print(f"Urinish soni: {attempt_counter}")
        if attempt_counter < 3:
            raise ConnectionError("Tarmoqqa ulanishda xatolik!")
        return f"Muvaffaqiyatli ma'lumot: {x * 2}"

    print("--- 2-NAMUNA: Retry dekoratori (xatolikni qayta tiklash) ---")
    print(f"Funksiya nomi (__name__): {unstable_operation.__name__}")
    print(f"Hujjat (__doc__): {unstable_operation.__doc__}")
    print("-" * 50)

    try:
        res = unstable_operation(21)
        print(f"Yakuniy natija: {res}")
    except Exception as e:
        print(f"Barcha urinishlar muvaffaqiyatsiz yakunlandi. Oxirgi xato: {e}")

```

### Kodning asosiy qismlari izohi:

1. **`functools.wraps(func)`**: Barcha dekoratorlar ichida ishlatilgan bo'lib, o'ralayotgan funksiyaning asl `__name__` (nomi) va `__doc__` (hujjati) yo'qolib qolmasligini ta'minlaydi.
2. **`timer`**: `time.perf_counter()` yordamida funksiyaning bajarilishiga ketgan aniq vaqtni o'lchaydi.
3. **`log_calls`**: Funksiyaga uzatilgan pozitsion (`*args`) va kalit so'zli (`**kwargs`) argumentlarni, shuningdek, funksiya qaytargan natijani terminalga chiqaradi.
4. **`retry(times)`**: Tashqi funksiya parametr (`times`) qabul qiladi va ichki dekoratorni qaytaradi (dekoratorlar fabrikasi patterni). Agar funksiya ichida xatolik (`Exception`) yuzaga kelsa, sikl yordamida uni belgilangan martagacha qaytadan chaqiradi.
