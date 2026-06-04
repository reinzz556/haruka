# Message Builder Helper

**Thanks to [ValdazGT](https://gist.github.com/ValdazGT) or [Nixel](https://wa.me/6282139672290)**. The original owner of this message builder

---

## 📌 Cara Kerja & Arsitektur

Sebuah helper berbasis **Dynamic Proxy** untuk mempermudah pembuatan dan pengiriman berbagai jenis pesan interaktif (Rich Messages) pada bot WhatsApp (seperti Baileys/Socket). 

Helper ini mendukung *method chaining* (pemanggilan berantai) dan secara otomatis mengarahkan fungsi ke *instance* yang sesuai berdasarkan tipe yang ditentukan.

`messageBuilder` memanfaatkan JavaScript `Proxy` untuk menangkap setiap properti atau *method* yang dipanggil. 

1. **`setType(type)`**: Menentukan jenis builder yang ingin digunakan (`Button`, `ButtonV2`, `Carousel`, atau `AIRich`). Langkah ini **wajib** dilakukan pertama kali.
2. **Dynamic Forwarding**: Setelah tipe ditentukan, semua method berikutnya (seperti `.setTitle()`, `.addReply()`, dll) akan otomatis diteruskan (*forwarded*) ke *class instance* asli dari tipe tersebut.
3. **Promise Chaining**: Setiap *method* menghasilkan promise yang dirantai secara asinkron, memastikan urutan eksekusi data tetap aman sebelum akhirnya dikirim menggunakan `.send()`.

---

## 🛠️ API Penjelasan Fungsi

### `sock.messageBuilder(jid, options)`
Fungsi utama untuk menginisialisasi builder.
* `jid` *(String)*: ID Obrolan tujuan (misal: `m.chat` atau `remoteJid`).
* `options` *(Object, Opsional)*: Konfigurasi tambahan untuk pengiriman pesan (misal: `{ quoted: m }`).

### Inisiasi
Inisiasi semua fungsi atau class dari library
```javascript
import haruka, { addProperty, Button, ButtonV2, Carousel, AIRich } from "@ryuu-reinzz/haruka-lib
const sock = makeSocket({});
addProperty(sock, store);
```

### Method Bawaan Builder
* **.setType(type)**
    Mengatur jenis komponen pesan yang akan dibuat.
    * **Pilihan Tipe**: `'Button'`, `'ButtonV2'`, `'Carousel'`, `'AIRich'`.
    * *Return*: Mengembalikan proxy agar bisa dirantai kembali.
* **.send()**
    Mengirimkan pesan yang telah disusun ke `jid` yang ditentukan di awal. Harus dipanggil di paling akhir urutan.

---

## 🚀 Panduan Penggunaan (Contoh Implementasi)

Berikut adalah cara menggunakan `messageBuilder` wraper:

### 1. Membuat Pesan Interaktif (`Button`)
```javascript
await sock.messageBuilder(m.chat, { quoted: m })
    .setType('Button')
    .setTitle('🚀 Ryuu')
    .setSubtitle('Interactive Message')
    .setBody('Pilih menu di bawah')
    .setFooter('© Ryuu')
    .setImage('https://cdn.ornzora.eu.cc/b57c0d1e-d7a6-4277-8739-8f6b1d9894e6-FIORA.jpg')
    .addReply('📦 Menu', '.menu', { icon: 'DEFAULT' })
    .addReply('👤 Profile', '.profile', { icon: 'REVIEW' })
    .addUrl('🌐 Website', 'https://example.com', true, { icon: 'PROMOTION' })
    .addCopy('📋 Copy Code', 'RYUU-2026', { icon: 'DOCUMENT' })
    .addSelection('📚 Pilih Kategori')
    .makeSection('Main Menu')
    .makeRow('🔥 HOT', 'Downloader', 'Download social media', '.dl')
    .makeRow('⚡ FAST', 'AI Chat', 'Chat dengan AI', '.ai')
    .send();
```

### 2. Membuat Pesan Tombol (`ButtonV2`)
```javascript
await sock.messageBuilder(m.chat)
    .setType('ButtonV2')
    .setTitle('🚀 Ryuu')
    .setSubtitle('Buttons Message')
    .setBody('Halo dunia')
    .setFooter('Footer Message')
    .setThumbnail('https://cdn.ornzora.eu.cc/4d2905ce-3707-4ec0-998a-68a3d851629f-FIORA.jpg')
    .addButton('📦 Menu', '.menu')
    .addButton('👤 Profile', '.profile')
    .send();
```

### 3. Membuat Pesan Geser (`Carousel`)
```javascript
await sock.messageBuilder(m.chat, { quoted: m })
    .setType('Carousel')
    .setBody('🛍️ Product List')
    .setFooter('Swipe untuk lihat')
    .addCard(
        await new Button(conn)
            .setTitle('🍔 Burger')
            .setBody('Burger terenak')
            .setFooter('$5')
            .setImage('https://api.ryuu-dev.offc.my.id/src/assest/mahiru/Mahiru-11.jpg')
            .addReply('🛒 Buy', '.buy burger')
            .toCard()
    )
    .addCard(
        await new Button(conn)
            .setTitle('🍕 Pizza')
            .setBody('Pizza mozzarella')
            .setFooter('$7')
            .setImage('https://api.ryuu-dev.offc.my.id/src/assest/mahiru/Mahiru-11.jpg')
            .addReply('🛒 Buy', '.buy pizza')
            .toCard()
    )
    .send();
```

### 4. Membuat Pesan Kaya Fitur AI (`AIRich`)
```javascript
await sock.messageBuilder(m.chat, { quoted: m })
    .setType('AIRich')
    .setTitle('🚀 Ryuu')
    .setFooter('© Haruka')   
    .addSuggest(['Ryuu', 'Ryuu', 'Haruka'])    
    .addTip('Ini adalah text tip (Metadata Text)')
    .addText(`# Halo Dunia
=={ Yellow Text }==

Ini hyperlink:
[Google](https://google.com)

Ini auto citation:
[](https://openai.com)

Ini LaTeX:
[Shiroko|1429|1897]<https://cdn.ornzora.eu.cc/a3a756f2-6bb8-4814-a024-c325524a2308-FIORA.png>`)
    .addProduct({
        title: 'Nama Produk', 
        brand: 'Ryuu', 
        price: 'Rp 1000', 
        sale_price: 'Rp 0',
        product_url: 'https://wa.me/6288246552068', 
        icon_url: "https://static.nike.com/a/images/t_PDP_1280_v1/f_auto,q_auto:eco/additional_image_1.png",
        image_url: 'https://api.ryuu-dev.offc.my.id/src/assest/mahiru/Mahiru-10.jpg'
    })
    .addCode('javascript', `class Ryuu {
	static hello() {
		return 'Hello World';
	}
}`)
   .addText('HScroll Product (Array of Object Input):') 
   .addProduct(Array(5).fill({
		title: global.namabot, 
		brand: 'Ryuu', 
		price: 'Rp 1000', 
		sale_price: 'Rp 0', 
		url: 'https://wa.me/6288246552068', 
		icon: "https://static.nike.com/a/images/t_PDP_1280_v1/f_auto,q_auto:eco/additional_image_1.png", 
		image: "https://api.ryuu-dev.offc.my.id/src/assest/mahiru/Mahiru-10.jpg"
	}))
    .addTable([
        ['Nama', 'Role'],
        ['Ryuu', 'Developer'],
        ['Haruka', 'Assistant']
    ])
    .addSource([
        ['https://api.ryuu-dev.offc.my.id/src/assest/mahiru/Mahiru-09.jpg', 'https://github.com/reinzz556/', 'GitHub'],
        ['https://api.ryuu-dev.offc.my.id/src/assest/mahiru/Mahiru-09.jpg', 'https://api.ryuu-dev.my.id/', 'Haruka Botz']
    ])
    .addImage('https://api.ryuu-dev.offc.my.id/src/assest/mahiru/Mahiru-07.jpg')
    .addVideo("https://api.ryuu-dev.my.id/2026-06-01-064748592.mp4|10")
    .addReels(Array(5).fill({
        username: 'Ryuu',
        profile_url: 'https://api.ryuu-dev.my.id/logo.png',
        thumbnail: 'https://api.ryuu-dev.offc.my.id/src/assest/mahiru/Mahiru-08.jpg',
        url: 'https://api.ryuu-dev.my.id/',
        title: 'Demo Reel',
        like: 12000,
        share: 500,
        view: 999999,
        source: 'IG',
        verified: true
    }))
    .addPost(Array(5).fill({
        profile_url: "https://api.ryuu-dev.my.id/logo.png",
        username: 'Ryuu',
        title: "Demo Post",
        subtitle: 'Ryuu',
        caption: 'hii~ im haruka, just quietly observing things around here.',
        verified: true,
        url: 'https://api.ryuu-dev.my.id/',
        thumbnail: 'https://api.ryuu-dev.offc.my.id/src/assest/mahiru/Mahiru-07.jpg',
        source: 'INSTAGRAM',
        footer: 'Haruka',
        deeplink: 'https://api.ryuu-dev.my.id/',
        icon: "https://api.ryuu-dev.my.id/logo.png",
        orientation: 'LANDSCAPE',
        post_type: 'PHOTO',
        comment: 1,
        share: 1,
        like: 1
    }))
    .send();
```

---

## ⚠️ Penanganan Error (Error Handling)

Builder ini dilengkapi dengan proteksi *state* internal untuk menghindari salah urutan eksekusi:

1. **Memanggil method sebelum menentukan tipe:**
```javascript
sock.messageBuilder(jid).setTitle('Halo'); 
// Error: Kamu harus memanggil .setType() sebelum memanggil .setTitle()
```
2. **Langsung mengirim tanpa menentukan tipe:**
```javascript
sock.messageBuilder(jid).send(); 
// Error: Kamu harus menentukan .setType() terlebih dahulu.
```
3. **Memasukkan tipe yang tidak terdaftar:**
```javascript
sock.messageBuilder(jid).setType('Koran'); 
// Error: Type Koran tidak dikenali.
```
