# Pink Calculator

Görseldeki pastel pembe hesap makinesi tasarımını temel alan, `tkinter` ve `Canvas` kullanarak yuvarlatılmış ekran ve butonlar çizen bir Python masaüstü hesap makinesi uygulamasıdır.

## Özellikler

- **Pastel Pembe Arayüz ve Yuvarlatılmış Çizimler:** `Canvas` üzerinde `rounded_rect` metodu ile yumuşak köşeli ekran ve buton kutuları çizilir.
- **Tam Bağlı Tasarım ve Mantık:** Tuş tıklamaları ve klavye girişleri `self.current_input` durumunu günceller, her değişimde `draw_display()` çağrılarak ekran anlık yenilenir.
- **Temel İşlemler ve Yardımcı Tuşlar:** Toplama (`+`), çıkarma (`-`), çarpma (`×`), bölme (`÷`), yüzde (`%`), çift sıfır (`00`), ondalık nokta (`.`), tek karakter silme (`⌫`) ve ekran temizleme (`C`) desteklenir.
- **Klavye Kısayolları:** Rakam ve işlem tuşlarının yanı sıra `Enter` (`=`), `Backspace` (`⌫`) ve `Esc` (`C`) tuşları kullanılabilir.
- **Hata Yönetimi ve Sayı Biçimlendirme:** Sıfıra bölme veya geçersiz ifadelerde ekranda `"Error"` gösterilir; tam sayıya karşılık gelen ondalıklı sonuçlar sadeleştirilir ve uzun ondalıklar `8` basamağa yuvarlanır.

## Renk Paleti

| Bileşen | Hex Kodu | Açıklama |
| :--- | :--- | :--- |
| Arka Plan | `#F8C8DC` | Pencere ve buton boşluklarının pastel pembe zemin rengi |
| Ekran Kutusu | `#F5D9E4` | Üst sonuç/işlem ekranının dolgu rengi |
| Rakam Tuşları | `#FBE3EC` | `0`–`9`, `00` ve `.` tuşlarının dolgu rengi |
| İşlem Tuşları | `#F39BC1` | `C`, `⌫`, `%`, `÷`, `×`, `-` ve `+` tuşlarının dolgu rengi |
| Eşittir (`=`) Tuşu | `#EC407A` | Vurgulu koyu pembe dolgu rengi |
| Metin Rengi | `#5A2A40` | Ekran ve tuş yazılarının koyu mürdüm tonu (`=` yazısı `#FFFFFF`) |

## Kod Yapısı (`Calculator` Sınıfı)

- **Tasarım Metodları:**
  - `rounded_rect`: Belirlenen koordinatlar ve yarıçap (`r`) ile `create_polygon(..., smooth=True)` kullanarak yuvarlatılmış dikdörtgen çizer.
  - `draw_display`: `self.current_input` değerini okur, `*` ve `/` karakterlerini `×` ve `÷` olarak gösterir, metin uzunluğuna göre yazı boyutunu (`42` veya `28`) ayarlar ve ekranı sağ alt köşeye hizalar.
  - `draw_buttons`: 5 satır × 4 sütunluk buton düzenini çizer, renkleri atar ve her buton etiketine (`tag`) `<Button-1>` tıklama olayını bağlar.
- **Mantık ve Klavye Metodları:**
  - `on_button_click`: Basılan tuşa göre `self.current_input` ve `self.reset_next` durumunu yönetir, ardışık işlem işaretlerini düzenler ve sonunda `self.draw_display()` çağırır.
  - `calculate` ve `apply_percent`: İfade karakterlerini doğrular, `eval()` ile sonucu hesaplar (`apply_percent` ayrıca `100`'e böler) ve hatalarda `"Error"` üretir.
  - `format_number`: Ondalıklı değerleri tam sayıysa tam sayıya çevirir veya `8` basamağa yuvarlayarak biçimlendirir.
  - `bind_keys` ve `handle_keypress`: Fiziksel klavye tuşlarını hesap makinesi eylemlerine eşler.

## Kurulum ve Çalıştırma

Uygulama harici bir paket gerektirmez; Python 3 standart kütüphanesindeki `tkinter` modülüyle çalışır.

1. Kodu `calculator.py` adıyla kaydedin.
2. Terminal veya komut istemcisinde dosyanın bulunduğu klasöre geçin.
3. Uygulamayı başlatın:

```bash
python calculator.py
```

## Proje Yapısı

- `calculator.py`: Pencere ayarlarını, renk paletini, `Canvas` çizim metodlarını, hesaplama mantığını ve klavye olaylarını içeren ana uygulama dosyası.
- `README.md`: Proje açıklaması, renk tablosu, sınıf yapısı, kaynak kod ve çalıştırma adımları.
