# ⚡ STM32 Embedded Microprocessor System Project

### Gelişmiş STM32F103C8T6 Çok Fonksiyonlu Mikroişlemci Takip ve Denetim Sistemi

Bu proje; **STM32F103C8T6 (Blue Pill)** mikrodenetleyicisi üzerinde **GPIO, EXTI (Harici Kesme), Timer (PWM), ADC ve USART** çevre birimlerinin entegre biçimde koordine edildiği gömülü sistem dönem sonu projesidir.

---

## 🛠️ Donanım Mimarisi ve Çevre Birimleri

| Çevre Birimi     | Pin / Kanal Tanımı      | Bağlantı & Fonksiyon                                          |
| :--------------- | :---------------------- | :------------------------------------------------------------ |
| **GPIO**         | `PA0` - `PA7` (Output)  | Çift 7-Segment ortak katot/anot pin sürüşü                    |
| **Multiplexing** | `PB5`, `PB6` (Output)   | 7-Segment birler ve onlar basamağı transistör/GND seçimi      |
| **EXTI**         | `PB0`, `PB10`, `PB11`   | Butonlar (Arttırma, Azaltma, Sıfırlama) için donanımsal kesme |
| **ADC1**         | `PB1` (ADC1_IN9)        | Potansiyometreden 12-bit (0-4095) analog açı/konum okuma      |
| **TIM1 (PWM)**   | `PA8` (TIM1_CH1)        | 50 Hz PWM frekansı ile SG90 Servo motor açı kontrolü          |
| **USART1**       | `PA9` (TX), `PA10` (RX) | 9600 Baud hızıyla PC seri terminale telemetri aktarımı        |

---

## 📐 Matematiksel Hesaplamalar ve Formüller

### 1. Timer / PWM Frekans Hesabı (Servo Motor - 50 Hz)

Servo motorlar tipik olarak **50 Hz (20 ms periyot)** sinyal gerektirir:

$$f_{PWM} = \frac{f_{CLK}}{(PSC + 1) \times (ARR + 1)}$$

- Sistem Saat Kaynağı ($f_{CLK}$): $8\text{ MHz}$
- Prescaler ($PSC$): $15 \implies \frac{8\text{ MHz}}{16} = 500\text{ kHz}$
- Counter Period ($ARR$): $9999 \implies \frac{500\text{ kHz}}{10000} = 50\text{ Hz}$

### 2. ADC $\to$ Servo PWM Dönüşümü

ADC 12-bit çözünürlükle $0 - 4095$ arasında değer üretir. Servo puls genişliği ise $250 - 1250$ tick aralığına eşlenir:

$$PWM = 250 + \frac{ADC}{4.1}$$

### 3. USART Açı Gönderimi

Okunan potansiyometre değeri eş zamanlı olarak $0^\circ - 180^\circ$ açı formatına dönüştürülür ve Termite seri arayüzüne aktarılır:

$$Angle = \frac{ADC \times 180}{4096}$$

---

## 💻 Temel Gömülü Kod Blokları

### Harici Kesme (EXTI) Rutinleri

```c
void EXTIO_IRQHandler(void) {
    sayi++;
    HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_0);
}

void EXTI15_10_IRQHandler(void) {
    if (__HAL_GPIO_EXTI_GET_IT(GPIO_PIN_10) != RESET) {
        sayi = 0; // Reset
        HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_10);
    } else if (__HAL_GPIO_EXTI_GET_IT(GPIO_PIN_11) != RESET) {
        sayi--; // Azaltma
        HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_11);
    }
}

float calculateAngleFromADC(void) {
    HAL_ADC_Start(&hadc1);
    if (HAL_ADC_PollForConversion(&hadc1, 1000) == HAL_OK) {
        uint32_t adcValue = HAL_ADC_GetValue(&hadc1);
        float angle = ((float)adcValue * 180.0f) / 4096.0f;
        return angle;
    }
    return 0.0f;
}

Copyright (c) 2026 Muhammed Emin Korkunç. All Rights Reserved.

Bu projenin tüm analizleri, STM32 çevre birim mimarisi, hesaplamaları
ve rapor içeriği Muhammed Emin Korkunç'a aittir. Yazarın yazılı izni
olmaksızın kısmen veya tamamen kopyalanması, paylaşılması veya akademik/ticari
amaçla izinsiz kullanımı kesinlikle yasaktır.

👨‍💻 Geliştirici / Author
Muhammed Emin Korkunç

Email: muhammedemin.korkunc@gmail.com

LinkedIn: Muhammed Emin Korkunç

GitHub: @muhammedkorkunc

Fatih Sultan Mehmet Vakıf Üniversitesi — Bilgisayar Mühendisliği Bölümü


```
