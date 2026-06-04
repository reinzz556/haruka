# Message Builder Helper

Sebuah helper berbasis **Dynamic Proxy** untuk mempermudah pembuatan dan pengiriman berbagai jenis pesan interaktif (Rich Messages) pada bot WhatsApp (seperti Baileys/Socket). 

Helper ini mendukung *method chaining* (pemanggilan berantai) dan secara otomatis mengarahkan fungsi ke *instance* yang sesuai berdasarkan tipe yang ditentukan.

---

## 📌 Cara Kerja & Arsitektur

`messageBuilder` memanfaatkan JavaScript `Proxy` untuk menangkap setiap properti atau *method* yang dipanggil. 

1. **`setType(type)`**: Menentukan jenis builder yang ingin digunakan (`Button`, `ButtonV2`, `Carousel`, atau `AIRich`). Langkah ini **wajib** dilakukan pertama kali.
2. **Dynamic Forwarding**: Setelah tipe ditentukan, semua method berikutnya (seperti `.setTitle()`, `.addReply()`, dll) akan otomatis diteruskan (*forwarded*) ke *class instance* asli dari tipe tersebut.
3. **Promise Chaining**: Setiap *method* menghasilkan promise yang dirantai secara asinkron, memastikan urutan eksekusi data tetap aman sebelum akhirnya dikirim menggunakan `.send()`.

---

## 🛠️ API Penjelasan Fungsi

### `messageBuilder(jid, options)`
Fungsi utama untuk menginisialisasi builder.
* `jid` *(String)*: ID Obrolan tujuan (misal: `m.chat` atau `remoteJid`).
* `options` *(Object, Opsional)*: Konfigurasi tambahan untuk pengiriman pesan (misal: `{ quoted: m }`).

### Method Bawaan Builder
* **.setType(type)**
    Mengatur jenis komponen pesan yang akan dibuat.
    * **Pilihan Tipe**: `'Button'`, `'ButtonV2'`, `'Carousel'`, `'AIRich'`.
    * *Return*: Mengembalikan proxy agar bisa dirantai kembali.
* **.send()**
    Mengirimkan pesan yang telah disusun ke `jid` yang ditentukan di awal. Harus dipanggil di paling akhir urutan.

---

## 🚀 Panduan Penggunaan (Contoh Implementasi)

Berikut adalah cara menggunakan `messageBuilder` untuk menggantikan inisialisasi *class* manual:

### 1. Membuat Pesan Interaktif Lama (`Button`)
```javascript
await messageBuilder(m.chat, { quoted: m })
    .setType('Button')
    .setTitle('🚀 Ryuu')
    .setSubtitle('Interactive Message')
    .setBody('Pilih menu di bawah')
    .setFooter('© Ryuu')
    .setImage('https://cdn.ornzora.eu.cc/b57c0d1e-d7a6-4277-8739-8f6b1d9894e6-FIORA.jpg')
    .addReply('📦 Menu', '.menu', { icon: 'DEFAULT' })
    .addReply('👤 Profile', '.profile', { icon: 'REVIEW' })
    .addUrl('🌐 Website', 'https://example.com', true, { icon: 'PROMOTION' })
    .addCopy('📋 Copy Code', 'NIX-2026', { icon: 'DOCUMENT' })
    .addSelection('📚 Pilih Kategori')
    .makeSection('Main Menu')
    .makeRow('🔥 HOT', 'Downloader', 'Download social media', '.dl')
    .makeRow('⚡ FAST', 'AI Chat', 'Chat dengan AI', '.ai')
    .send();
```

### 2. Membuat Pesan Tombol Baru (`ButtonV2`)
```javascript
await messageBuilder(m.chat)
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
await messageBuilder(m.chat, { quoted: m })
    .setType('Carousel')
    .setBody('🛍️ Product List')
    .setFooter('Swipe untuk lihat')
    .addCard(
        await new Button(conn)
            .setTitle('🍔 Burger')
            .setBody('Burger terenak')
            .setFooter('$5')
            .setImage('https://cdn.ornzora.eu.cc/36df8c36-c74e-4dc2-bc03-87893f373cb4-FIORA.jpg')
            .addReply('🛒 Buy', '.buy burger')
            .toCard()
    )
    .addCard(
        await new Button(conn)
            .setTitle('🍕 Pizza')
            .setBody('Pizza mozzarella')
            .setFooter('$7')
            .setImage('https://cdn.ornzora.eu.cc/36df8c36-c74e-4dc2-bc03-87893f373cb4-FIORA.jpg')
            .addReply('🛒 Buy', '.buy pizza')
            .toCard()
    )
    .send();
```

### 4. Membuat Pesan Kaya Fitur AI (`AIRich`)
```javascript
await messageBuilder(m.chat, { quoted: m })
    .setType('AIRich')
    .setTitle('🚀 Ryuu')
    .setFooter('© Fiora Sylvie')   
    .addSuggest(['Ryuu', 'Ryuu', 'Fiora Sylvie'])    
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
        image_url: 'https://cdn.ornzora.eu.cc/152f4f0b-02fb-4d60-aacc-fc4cfa87ccdb-FIORA.jpg'
    })
    .addCode('javascript', `class Ryuu {
	static hello() {
		return 'Hello World';
	}
}`)
   .addText('HScroll Product (Array of Object Input):') 
   .addProduct(Array(5).fill({
		title: namebot, 
		brand: 'Ryuu', 
		price: 'Rp 1000', 
		sale_price: 'Rp 0', 
		url: 'https://wa.me/6282139672290', 
		icon: "https://static.nike.com/a/images/t_PDP_1280_v1/f_auto,q_auto:eco/additional_image_1.png", 
		image: "https://cdn.ornzora.eu.cc/152f4f0b-02fb-4d60-aacc-fc4cfa87ccdb-FIORA.jpg"
	}))
    .addTable([
        ['Nama', 'Role'],
        ['Ryuu', 'Developer'],
        ['Fiora Sylvie', 'Assistant']
    ])
    .addSource([
        ['https://cdn.ornzora.eu.cc/dc85c945-96f7-4d50-aaa4-1dff7249aaf4-FIORA.jpg', 'https://github.com/reinzz556/', 'GitHub'],
        ['https://cdn.ornzora.eu.cc/dc85c945-96f7-4d50-aaa4-1dff7249aaf4-FIORA.jpg', 'https://api.ryuu-dev.my.id/', 'Haruka Botz']
    ])
    .addImage('https://cdn.ornzora.eu.cc/d987ff9c-c16c-4f1e-a8d6-953e375f4aec-FIORA.jpg')
    .addVideo("https://cdn.ornzora.eu.cc/a1a3124d-533a-490d-8b56-517b8dccffb1-FIORA.mp4|10")
    .addReels(Array(5).fill({
        username: 'Ryuu',
        profile_url: 'https://cdn.ornzora.eu.cc/4d2905ce-3707-4ec0-998a-68a3d851629f-FIORA.jpg',
        thumbnail: 'https://cdn.ornzora.eu.cc/d6b36500-3b7e-49ee-9123-52bb1bf106be-FIORA.jpg',
        url: 'https://api.ryuu-dev.my.id/',
        title: 'Demo Reel',
        like: 12000,
        share: 500,
        view: 999999,
        source: 'IG',
        verified: true
    }))
    .addPost(Array(5).fill({
        profile_url: "https://cdn.ornzora.eu.cc/2498bf66-6870-4f8a-8421-0a77f7baa95b-FIORA.jpg",
        username: 'Ryuu',
        title: "Demo Post",
        subtitle: 'Ryuu',
        caption: 'hii~ im haruka sylvie, just quietly observing things around here.',
        verified: true,
        url: 'https://api.ryuu-dev.my.id/',
        thumbnail: 'https://cdn.ornzora.eu.cc/7048efb4-2abf-4081-bdd1-2f65972d793a-FIORA.jpg',
        source: 'INSTAGRAM',
        footer: 'Fiora Sylvie',
        deeplink: 'https://api.ryuu-dev.my.id/',
        icon: "https://cdn.ornzora.eu.cc/2498bf66-6870-4f8a-8421-0a77f7baa95b-FIORA.jpg",
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
   messageBuilder(jid).setTitle('Halo'); 
   // Error: Kamu harus memanggil .setType() sebelum memanggil .setTitle()
   ```
2. **Langsung mengirim tanpa menentukan tipe:**
   ```javascript
   messageBuilder(jid).send(); 
   // Error: Kamu harus menentukan .setType() terlebih dahulu.
   ```
3. **Memasukkan tipe yang tidak terdaftar:**
   ```javascript
   messageBuilder(jid).setType('Koran'); 
   // Error: Type Koran tidak dikenali.
   ```
