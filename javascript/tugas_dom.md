## Studi Kasus: Sistem Manajemen Karakter & Inventory RPG (Pure JavaScript)

Selamat datang di modul latihan pengembangan *Frontend Logic*. Pada studi kasus ini, kamu akan membangun mesin pengelola status karakter, perlengkapan (*equipment*), dan tas penyimpanan (*inventory*) untuk sebuah game RPG menggunakan **Pure JavaScript (ES6+)** tanpa bantuan *library* atau *framework* eksternal.

Pengerjaan dibagi menjadi **2 Tahap**:
1. **Tahap 1 — State & Logic Layer:** Mengolah struktur data bersarang (*nested array & object*) secara *immutable*.
2. **Tahap 2 — UI & DOM Integration:** Menghubungkan fungsi logika dari Tahap 1 ke tampilan web interaktif menggunakan *Vanilla DOM API*.

---

# TAHAP 1: LOGIKA ARRAY & OBJECT (PURE JAVASCRIPT)

Pada tahap ini, fokus utama kamu adalah membangun **6 fungsi inti** yang mengelola data karakter dengan aman dan terprediksi.

## Aturan Main Tahap 1 (Wajib Dipatuhi)

1. **Strict Immutability (Dilarang Mutasi Data):**
   Jangan pernah mengubah properti pada object `character` secara langsung (contoh yang dilarang: `char.equipped.weapon = ...`, `char.inventory.push(...)`, atau `item.qty -= 1`). Setiap fungsi yang memodifikasi data wajib mengembalikan **object `character` baru**.
2. **Functional Array & Object Methods:**
   Dilarang menggunakan perulangan konvensional (`for`, `while`, `do...while`, maupun `.forEach()` untuk memodifikasi data). Selesaikan seluruh misi menggunakan `.map()`, `.filter()`, `.reduce()`, `.find()`, `.some()`, *spread operator* (`...`), *destructuring*, serta `Object.values()` / `Object.entries()`.

## Starter Code & Test Cases Tahap 1

Buat file bernama `logic.js`, salin kode di bawah ini, lalu jalankan menggunakan **Node.js** (`node logic.js`) atau **Browser Console** untuk menguji hasil pekerjaanmu.

```javascript
const character = {
  name: "Aether",
  level: 12,
  baseStats: {
    hp: 200,
    atk: 35,
    def: 20,
    spd: 15,
  },
  equipped: {
    weapon: {
      id: "wpn-01",
      name: "Iron Broadsword",
      slot: "weapon",
      rarity: "common",
      statBonus: { atk: 12, spd: -2 },
    },
    armor: {
      id: "arm-01",
      name: "Leather Tunic",
      slot: "armor",
      rarity: "common",
      statBonus: { hp: 30, def: 8, spd: 2 },
    },
    accessory: null,
  },
  inventory: [
    {
      id: "wpn-02",
      name: "Flamebrand Katana",
      slot: "weapon",
      rarity: "rare",
      qty: 1,
      statBonus: { atk: 28, spd: 5 },
    },
    {
      id: "acc-01",
      name: "Ring of Vitality",
      slot: "accessory",
      rarity: "uncommon",
      qty: 2,
      statBonus: { hp: 50, def: 3 },
    },
    {
      id: "arm-02",
      name: "Mithril Chainmail",
      slot: "armor",
      rarity: "epic",
      qty: 1,
      statBonus: { hp: 80, def: 25, spd: -1 },
    },
    {
      id: "pot-01",
      name: "Elixir of Strength",
      slot: "consumable",
      rarity: "rare",
      qty: 3,
      statBonus: { atk: 5 },
    },
  ],
};

// ==========================================
// TULIS JAWABAN FUNGSI KAMU DI BAWAH INI
// ==========================================

function getTotalStats(char) {
  // TODO: Implementasi Misi 1
}

function equipItem(char, itemId) {
  // TODO: Implementasi Misi 2
}

function unequipItem(char, slotName) {
  // TODO: Implementasi Misi 3
}

function useConsumable(char, itemId) {
  // TODO: Implementasi Misi 4
}

function getInventorySummary(char) {
  // TODO: Implementasi Misi 5
}

function findBestUpgrade(char, slotName) {
  // TODO: Implementasi Misi 6
}

// ==========================================
// SCRIPT PENGUJIAN MANDIRI (TEST CASES)
// ==========================================

console.log("1. Initial Stats:", getTotalStats(character));
// Ekspektasi: { hp: 230, atk: 47, def: 28, spd: 15 }

const stateAfterWeapon = equipItem(character, "wpn-02");
const stateAfterAcc = equipItem(stateAfterWeapon, "acc-01");
console.log("2. Stats After Equip:", getTotalStats(stateAfterAcc));
// Ekspektasi: { hp: 280, atk: 63, def: 31, spd: 22 }
console.log("   Inventory Weapon Swap:", stateAfterAcc.inventory.find((i) => i.id === "wpn-01"));
// Ekspektasi: { id: "wpn-01", name: "Iron Broadsword", slot: "weapon", rarity: "common", statBonus: { atk: 12, spd: -2 }, qty: 1 }

const stateAfterUnequip = unequipItem(stateAfterAcc, "accessory");
console.log("3. Ring Qty After Unequip:", stateAfterUnequip.inventory.find((i) => i.id === "acc-01")?.qty);
// Ekspektasi: 2

const stateAfterPotion = useConsumable(stateAfterUnequip, "pot-01");
console.log("4. Base ATK After Elixir:", stateAfterPotion.baseStats.atk);
// Ekspektasi: 40

console.log("5. Inventory Summary:", getInventorySummary(character));
// Ekspektasi:
// {
//   rare: { totalQty: 4, items: ["Flamebrand Katana", "Elixir of Strength"] },
//   uncommon: { totalQty: 2, items: ["Ring of Vitality"] },
//   epic: { totalQty: 1, items: ["Mithril Chainmail"] }
// }

console.log("6. Best Armor Upgrade:", findBestUpgrade(character, "armor"));
// Ekspektasi: { itemName: "Mithril Chainmail", scoreGain: 64 }

console.log("7. Immutability Check - Weapon:", character.equipped.weapon.name === "Iron Broadsword");
console.log("   Immutability Check - Inventory:", character.inventory.length === 4);
```

## Spesifikasi Misi Tahap 1

### Misi 1: `getTotalStats(char)`
Menghitung total akhir seluruh stat karakter (`hp`, `atk`, `def`, `spd`) dengan menjumlahkan `baseStats` dan seluruh `statBonus` dari item yang sedang terpasang di `equipped`.
* **Edge case:** Salah satu slot di `equipped` bisa bernilai `null` (seperti `accessory`), dan `statBonus` tiap item hanya mencantumkan stat tertentu saja.

### Misi 2: `equipItem(char, itemId)`
Memasang item dari `inventory` ke slot yang sesuai pada `equipped` dan mengembalikan state `char` baru.
* **Aturan:**
  1. Jika `itemId` tidak ditemukan di `inventory` atau `slot`-nya adalah `"consumable"`, kembalikan `char` tanpa perubahan.
  2. Kurangi `qty` item di `inventory` sebanyak `1` (hapus item dari array jika `qty` menjadi `0`).
  3. Hapus properti `qty` saat item dipasang ke `equipped`.
  4. **Swap & Stack:** Jika slot tujuan sudah terisi item lama (tidak `null`), kembalikan item lama tersebut ke `inventory`. Jika item lama sudah ada di `inventory` (berdasarkan `id`), tambahkan `qty`-nya sebesar `1`; jika belum ada, masukkan ke `inventory` dengan `qty: 1`.

### Misi 3: `unequipItem(char, slotName)`
Melepas item dari `equipped[slotName]`, mengubah slot tersebut menjadi `null`, dan mengembalikan item tersebut ke `inventory` dengan aturan *stacking* yang sama seperti Misi 2.
* **Edge case:** Jika `slotName` tidak valid atau sudah `null`, kembalikan `char` tanpa perubahan.

### Misi 4: `useConsumable(char, itemId)`
Menggunakan item dengan `slot: "consumable"` untuk menambah nilai `baseStats` karakter secara permanen.
* **Aturan:** Validasi keberadaan item dan pastikan `slot === "consumable"`. Tambahkan nilai `statBonus` ke `baseStats`, lalu kurangi `qty` item di `inventory` sebanyak `1` (hapus jika `0`).

### Misi 5: `getInventorySummary(char)`
Mengelompokkan isi `inventory` berdasarkan `rarity` menjadi object ringkasan berisi total kuantitas (`totalQty`) dan daftar nama item (`items`).

### Misi 6: `findBestUpgrade(char, slotName)`
Mencari satu item di `inventory` yang cocok dengan `slotName` dan memberikan kenaikan total poin stat tertinggi dibandingkan item yang sedang terpasang di `equipped[slotName]`.
* **Aturan:** Skor dihitung dari penjumlahan seluruh angka di dalam `statBonus` (jika slot saat ini `null`, skor pembanding adalah `0`). Kembalikan object `{ itemName, scoreGain }`, atau `null` jika tidak ada item di `inventory` yang lebih kuat daripada yang sedang dipakai.

---

# TAHAP 2: INTEGRASI STATE & MANIPULASI DOM

> **Syarat Masuk Tahap 2:** Pastikan seluruh 7 *Test Case* pada Tahap 1 sudah mengeluarkan hasil yang sesuai sebelum melanjutkan ke tahap ini.

Pada tahap ini, kamu akan menghubungkan **6 fungsi pure** yang sudah kamu buat di Tahap 1 ke antarmuka web menggunakan **Vanilla JavaScript (DOM API)**.

## Aturan Main Tahap 2 (Wajib Dipatuhi)

1. **Separation of Concerns (Pemisahan Logika & UI):**
   Jangan menulis ulang rumus penjumlahan stat atau logika manipulasi array `inventory` di dalam fungsi *render* maupun *event listener*. Wajib panggil fungsi-fungsi dari Tahap 1 (`getTotalStats`, `equipItem`, `unequipItem`, `useConsumable`, `getInventorySummary`, `findBestUpgrade`).
2. **Unidirectional Data Flow (`State → Render → Event → Update State`):**
   Semua perubahan tampilan harus berasal dari satu variabel pusat bernama `appState`. Jangan mengubah teks elemen DOM secara manual per tombol; perbarui `appState`, lalu panggil ulang fungsi `render()`.
3. **Event Delegation:**
   Dilarang menulis `onclick="..."` di HTML dan dilarang memasang `addEventListener` di dalam perulangan/loop kartu item. Gunakan **satu** *event listener* terpusat pada elemen pembungkus utama (`#app`).

## Starter Code Tahap 2 (`index.html` & `app.js`)

Buat dua file dalam satu folder (`index.html` dan `app.js`), lalu salin template berikut:

### 1. File `index.html`

```html
<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>RPG Character & Inventory Manager</title>
  <style>
    :root {
      --bg: #0f172a;
      --card: #1e293b;
      --border: #334155;
      --text: #f8fafc;
      --muted: #94a3b8;
      --primary: #3b82f6;
      --success: #10b981;
      --danger: #ef4444;
    }
    * { box-sizing: border-box; margin: 0; padding: 0; font-family: system-ui, sans-serif; }
    body { background: var(--bg); color: var(--text); padding: 2rem; line-height: 1.5; }
    .container { max-width: 1000px; margin: 0 auto; display: grid; gap: 1.5rem; }
    header { display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid var(--border); padding-bottom: 1rem; }
    .grid-2 { display: grid; grid-template-columns: 1fr 1.5fr; gap: 1.5rem; }
    .card { background: var(--card); border: 1px solid var(--border); border-radius: 10px; padding: 1.25rem; }
    h2 { font-size: 1.1rem; margin-bottom: 1rem; color: var(--muted); text-transform: uppercase; letter-spacing: 0.05em; }
    
    /* Stats */
    .stats-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 0.75rem; margin-bottom: 1.5rem; }
    .stat-box { background: var(--bg); padding: 0.75rem; border-radius: 8px; border: 1px solid var(--border); }
    .stat-label { font-size: 0.75rem; color: var(--muted); text-transform: uppercase; }
    .stat-value { font-size: 1.4rem; font-weight: 700; }

    /* Equipment Slots */
    .slot-item { display: flex; justify-content: space-between; align-items: center; background: var(--bg); padding: 0.75rem; border-radius: 8px; margin-bottom: 0.6rem; border: 1px solid var(--border); }
    .slot-title { font-size: 0.75rem; color: var(--muted); text-transform: uppercase; }
    .empty-text { color: var(--muted); font-style: italic; font-size: 0.9rem; }

    /* Filter Tabs */
    .filters { display: flex; flex-wrap: wrap; gap: 0.5rem; margin-bottom: 1rem; }
    .tab-btn { background: var(--bg); color: var(--muted); border: 1px solid var(--border); padding: 0.4rem 0.8rem; border-radius: 6px; cursor: pointer; font-size: 0.85rem; }
    .tab-btn.active { background: var(--primary); color: white; border-color: var(--primary); }

    /* Inventory List */
    .inv-item { display: flex; justify-content: space-between; align-items: center; background: var(--bg); padding: 0.85rem; border-radius: 8px; margin-bottom: 0.6rem; border: 1px solid var(--border); }
    .badge { display: inline-block; font-size: 0.7rem; padding: 0.15rem 0.45rem; border-radius: 4px; font-weight: 600; text-transform: uppercase; margin-left: 0.4rem; }
    .badge-upgrade { background: rgba(16, 185, 129, 0.15); color: var(--success); border: 1px solid var(--success); }
    .badge-rarity { background: #334155; color: #e2e8f0; }
    .bonus-text { font-size: 0.8rem; color: var(--muted); margin-top: 0.25rem; }

    /* Buttons & Log */
    button.action-btn { padding: 0.45rem 0.85rem; border: none; border-radius: 6px; font-weight: 600; cursor: pointer; font-size: 0.85rem; }
    .btn-equip { background: var(--primary); color: white; }
    .btn-use { background: var(--success); color: white; }
    .btn-unequip { background: var(--danger); color: white; }
    .btn-reset { background: transparent; border: 1px solid var(--border); color: var(--muted); padding: 0.45rem 0.9rem; border-radius: 6px; cursor: pointer; }
    .log-list { list-style: none; font-size: 0.85rem; color: var(--muted); max-height: 110px; overflow-y: auto; }
    .log-list li { padding: 0.25rem 0; border-bottom: 1px dashed var(--border); }
  </style>
</head>
<body>
  <div id="app" class="container">
    <header>
      <div>
        <h1 id="char-header">Character Sheet</h1>
      </div>
      <button class="btn-reset" data-action="reset">Reset Data</button>
    </header>

    <div class="grid-2">
      <!-- Kolom Kiri: Total Stats & Equipped -->
      <div class="card">
        <h2>Total Stats</h2>
        <div id="stats-container" class="stats-grid"></div>

        <h2>Equipped Gear</h2>
        <div id="equipped-container"></div>
      </div>

      <!-- Kolom Kanan: Filter & Inventory -->
      <div class="card">
        <h2>Inventory</h2>
        <div id="filter-container" class="filters"></div>
        <div id="inventory-container"></div>
      </div>
    </div>

    <!-- Log Aktivitas -->
    <div class="card">
      <h2>Activity Log</h2>
      <ul id="log-container" class="log-list"></ul>
    </div>
  </div>

  <script src="app.js"></script>
</body>
</html>
```

### 2. File `app.js`

```javascript
// ==========================================
// 1. INITIAL DATA & PHASE 1 PURE FUNCTIONS
// ==========================================
const initialCharacter = {
  name: "Aether",
  level: 12,
  baseStats: { hp: 200, atk: 35, def: 20, spd: 15 },
  equipped: {
    weapon: {
      id: "wpn-01",
      name: "Iron Broadsword",
      slot: "weapon",
      rarity: "common",
      statBonus: { atk: 12, spd: -2 },
    },
    armor: {
      id: "arm-01",
      name: "Leather Tunic",
      slot: "armor",
      rarity: "common",
      statBonus: { hp: 30, def: 8, spd: 2 },
    },
    accessory: null,
  },
  inventory: [
    { id: "wpn-02", name: "Flamebrand Katana", slot: "weapon", rarity: "rare", qty: 1, statBonus: { atk: 28, spd: 5 } },
    { id: "acc-01", name: "Ring of Vitality", slot: "accessory", rarity: "uncommon", qty: 2, statBonus: { hp: 50, def: 3 } },
    { id: "arm-02", name: "Mithril Chainmail", slot: "armor", rarity: "epic", qty: 1, statBonus: { hp: 80, def: 25, spd: -1 } },
    { id: "pot-01", name: "Elixir of Strength", slot: "consumable", rarity: "rare", qty: 3, statBonus: { atk: 5 } },
  ],
};

// TODO: Tempelkan 6 fungsi pure dari Tahap 1 di sini
// (getTotalStats, equipItem, unequipItem, useConsumable, getInventorySummary, findBestUpgrade)

// ==========================================
// 2. APPLICATION STATE & DOM RENDER ENGINE
// ==========================================
let appState = {
  character: structuredClone(initialCharacter),
  activeRarity: "all",
  logs: ["Karakter siap bertualang."],
};

// TODO: Implementasikan fungsi render() & event delegation di bawah ini
```

## Spesifikasi Misi Tahap 2

### Misi DOM 1: Dynamic Stat Panel & Equipment Slots
* **Header & Stats:** Tampilkan nama dan level karakter pada `#char-header` (contoh: `Aether — Lv. 12`), lalu render ke-4 stat (`hp`, `atk`, `def`, `spd`) ke dalam `#stats-container` menggunakan hasil dari `getTotalStats(appState.character)`.
* **Equipment Slots:** Render 3 slot (`weapon`, `armor`, `accessory`) ke `#equipped-container`:
  * Jika slot terisi item, tampilkan nama item, rincian bonus stat, dan tombol **"Unequip"** (`data-action="unequip"` dan `data-slot="..."`).
  * Jika slot bernilai `null`, tampilkan teks *"Empty Slot"* tanpa tombol aksi.

### Misi DOM 2: Interactive Inventory & Smart Upgrade Badge
* Render daftar item dari `appState.character.inventory` (yang sudah difilter berdasarkan `appState.activeRarity`) ke `#inventory-container`.
* **Conditional Action Button:**
  * Jika `item.slot === "consumable"`, tampilkan tombol **"Use"** (`data-action="consume"` dan `data-id="..."`).
  * Jika bukan `consumable`, tampilkan tombol **"Equip"** (`data-action="equip"` dan `data-id="..."`).
* **Smart Upgrade Indicator:** Manfaatkan fungsi `findBestUpgrade(appState.character, slot)` pada saat render. Jika suatu item di inventory merupakan *upgrade* terbaik untuk slot-nya, tampilkan badge hijau: `▲ Best Upgrade (+X)` di sebelah nama item.

### Misi DOM 3: Live Rarity Filter Tabs
* Render tombol filter ke `#filter-container` untuk kategori: `all`, `common`, `uncommon`, `rare`, dan `epic`.
* Gunakan fungsi `getInventorySummary(appState.character)` untuk menampilkan jumlah total kuantitas item pada setiap tab (contoh: `All (7)`, `Rare (4)`, `Common (0)`).
* Berikan class `.active` pada tab yang sesuai dengan `appState.activeRarity`.

### Misi DOM 4: Centralized Event Delegation, Activity Log & Reset
* Pasang **satu** `addEventListener("click", ...)` pada elemen `#app` untuk menangani seluruh aksi (`equip`, `unequip`, `consume`, `filter`, dan `reset`).
* Setiap kali pemain melakukan aksi (`equip`, `unequip`, atau `consume`), tambahkan pesan aktivitas baru ke bagian paling atas `appState.logs` (maksimal simpan 5 log terakhir secara *immutable*).
* Jika tombol **Reset Data** (`data-action="reset"`) diklik, kembalikan `appState` ke kondisi awal menggunakan `structuredClone(initialCharacter)`.

---

## Checklist Mandiri Sebelum Pengumpulan
Pastikan seluruh poin berikut sudah terpenuhi sebelum menyerahkan hasil pekerjaanmu kepada pembimbing:
- [ ] Tidak ada penggunaan `for`, `while`, atau mutasi data langsung (`.push()`, `.splice()`, ubah properti langsung).
- [ ] Seluruh 7 *Test Case* pada Tahap 1 berjalan tanpa *error* dan menghasilkan output yang tepat.
- [ ] Tombol **Equip**, **Unequip**, dan **Use** memperbarui Total Stats, slot equipment, jumlah item di inventory, dan badge *Best Upgrade* secara otomatis.
- [ ] Tab filter kategori *rarity* menampilkan angka jumlah item yang akurat dan tidak menghapus item saat berpindah tab.
- [ ] Hanya terdapat **satu** `addEventListener` pada elemen `#app`.
- [ ] Tombol **Reset Data** mengembalikan seluruh tampilan dan data ke kondisi semula.