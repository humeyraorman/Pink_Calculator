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

## Kaynak Kod (`calculator.py`)

```python
import tkinter as tk


class Calculator:
    def __init__(self, root):
        self.root = root
        self.root.title("Pink Calculator")
        self.root.geometry("380x620")
        self.root.resizable(False, False)

        # Tasarım renk paleti
        self.bg_color = "#F8C8DC"        # Ana arka plan pembesi
        self.display_bg = "#F5D9E4"      # Üst ekran kutusu
        self.num_bg = "#FBE3EC"          # Rakam tuşları
        self.op_bg = "#F39BC1"           # İşlem tuşları (C, ⌫, %, ÷, ×, -, +)
        self.eq_bg = "#EC407A"           # Eşittir (=) tuşu
        self.text_color = "#5A2A40"      # Koyu yazı rengi
        self.eq_text_color = "#FFFFFF"   # Eşittir tuşu beyaz yazı

        self.root.configure(bg=self.bg_color)

        # Hesaplama durumu (tasarım ve mantık bu değişken üzerinden bağlıdır)
        self.current_input = ""
        self.reset_next = False

        # Tasarım alanları
        self.display_canvas = tk.Canvas(
            self.root,
            width=340,
            height=135,
            bg=self.bg_color,
            highlightthickness=0
        )
        self.display_canvas.pack(pady=(24, 14))

        self.buttons_canvas = tk.Canvas(
            self.root,
            width=348,
            height=425,
            bg=self.bg_color,
            highlightthickness=0
        )
        self.buttons_canvas.pack(pady=(4, 20))

        # İlk çizim ve klavye bağlantısı
        self.draw_display()
        self.draw_buttons()
        self.bind_keys()

    # -------------------- TASARIM METODLARI --------------------

    def rounded_rect(self, canvas, x1, y1, x2, y2, r=20, **kwargs):
        """Köşeleri yuvarlatılmış dikdörtgen çizer."""
        points = [
            x1 + r, y1,
            x2 - r, y1,
            x2, y1,
            x2, y1 + r,
            x2, y2 - r,
            x2, y2,
            x2 - r, y2,
            x1 + r, y2,
            x1, y2,
            x1, y2 - r,
            x1, y1 + r,
            x1, y1
        ]
        return canvas.create_polygon(points, smooth=True, **kwargs)

    def draw_display(self):
        """Güncel hesaplama değerini (self.current_input) ekrana çizer."""
        self.display_canvas.delete("all")
        w = int(self.display_canvas["width"])
        h = int(self.display_canvas["height"])

        self.rounded_rect(self.display_canvas, 0, 0, w, h, r=24, fill=self.display_bg)

        display_text = self.current_input if self.current_input else "0"
        display_text = display_text.replace("*", "×").replace("/", "÷")
        font_size = 42 if len(display_text) <= 8 else 28

        self.display_canvas.create_text(
            w - 24,
            h - 28,
            text=display_text,
            anchor="se",
            fill=self.text_color,
            font=("Segoe UI", font_size, "bold")
        )

    def draw_buttons(self):
        """Butonları çizer ve her butonu doğrudan tıklama/mantık metoduna bağlar."""
        self.buttons_canvas.delete("all")

        layout = [
            ["C", "⌫", "%", "÷"],
            ["7", "8", "9", "×"],
            ["4", "5", "6", "-"],
            ["1", "2", "3", "+"],
            ["00", "0", ".", "="],
        ]

        start_x = 6
        start_y = 6
        gap = 10
        btn_w = 76
        btn_h = 74

        for row_idx, row in enumerate(layout):
            for col_idx, label in enumerate(row):
                x1 = start_x + col_idx * (btn_w + gap)
                y1 = start_y + row_idx * (btn_h + gap)
                x2 = x1 + btn_w
                y2 = y1 + btn_h

                if label == "=":
                    fill_color = self.eq_bg
                    fg_color = self.eq_text_color
                elif label in ["C", "⌫", "%", "÷", "×", "-", "+"]:
                    fill_color = self.op_bg
                    fg_color = self.text_color
                else:
                    fill_color = self.num_bg
                    fg_color = self.text_color

                tag = f"btn_{row_idx}_{col_idx}"

                self.rounded_rect(
                    self.buttons_canvas,
                    x1, y1, x2, y2,
                    r=20,
                    fill=fill_color,
                    tags=tag
                )

                self.buttons_canvas.create_text(
                    (x1 + x2) / 2,
                    (y1 + y2) / 2,
                    text=label,
                    fill=fg_color,
                    font=("Segoe UI", 20, "bold"),
                    tags=tag
                )

                # Tasarım elemanına tıklama olayını bağla
                self.buttons_canvas.tag_bind(
                    tag,
                    "<Button-1>",
                    lambda event, lbl=label: self.on_button_click(lbl)
                )

    # -------------------- MANTIK METODLARI --------------------

    def on_button_click(self, label):
        """Basılan tuşa göre state'i günceller ve tasarım ekranını yeniler."""
        if label == "C":
            self.current_input = ""
            self.reset_next = False

        elif label == "⌫":
            if self.reset_next:
                self.current_input = ""
                self.reset_next = False
            else:
                self.current_input = self.current_input[:-1]

        elif label == "=":
            self.calculate()

        elif label == "%":
            self.apply_percent()

        else:
            op_map = {"×": "*", "÷": "/"}
            val = op_map.get(label, label)

            if self.reset_next:
                if val.isdigit() or val in ["00", "."]:
                    self.current_input = "" if val != "." else "0."
                self.reset_next = False

            if val in ["+", "-", "*", "/"]:
                if not self.current_input:
                    if val == "-":
                        self.current_input = "-"
                elif self.current_input[-1] in ["+", "-", "*", "/"]:
                    self.current_input = self.current_input[:-1] + val
                else:
                    self.current_input += val
            else:
                self.current_input += val

        # Mantık işlemi bittikten sonra tasarımı yeniden çiz
        self.draw_display()

    def apply_percent(self):
        if not self.current_input:
            return
        try:
            allowed = set("0123456789+-*/. ")
            if not set(self.current_input).issubset(allowed):
                raise ValueError("Geçersiz ifade")
            result = eval(self.current_input) / 100
            self.current_input = self.format_number(result)
            self.reset_next = True
        except Exception:
            self.current_input = "Error"
            self.reset_next = True

    def calculate(self):
        if not self.current_input:
            return
        try:
            allowed = set("0123456789+-*/. ")
            if not set(self.current_input).issubset(allowed):
                raise ValueError("Geçersiz ifade")
            result = eval(self.current_input)
            self.current_input = self.format_number(result)
            self.reset_next = True
        except Exception:
            self.current_input = "Error"
            self.reset_next = True

    @staticmethod
    def format_number(num):
        if isinstance(num, float):
            if num.is_integer():
                return str(int(num))
            return f"{round(num, 8):g}"
        return str(num)

    def bind_keys(self):
        self.root.bind("<Key>", self.handle_keypress)

    def handle_keypress(self, event):
        key = event.char
        keysym = event.keysym

        if key in "0123456789+-*/.":
            map_back = {"*": "×", "/": "÷"}
            self.on_button_click(map_back.get(key, key))
        elif keysym in ("Return", "KP_Enter", "equal"):
            self.on_button_click("=")
        elif keysym == "BackSpace":
            self.on_button_click("⌫")
        elif keysym == "Escape":
            self.on_button_click("C")


if __name__ == "__main__":
    root = tk.Tk()
    app = Calculator(root)
    root.mainloop()
```

## Proje Yapısı

- `calculator.py`: Pencere ayarlarını, renk paletini, `Canvas` çizim metodlarını, hesaplama mantığını ve klavye olaylarını içeren ana uygulama dosyası.
- `README.md`: Proje açıklaması, renk tablosu, sınıf yapısı, kaynak kod ve çalıştırma adımları.