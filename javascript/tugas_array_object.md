# Modul Evaluasi JavaScript: Manipulasi Array & Object (Studi Kasus Sistem RPG)

Dokumen ini disusun sebagai panduan lengkap bagi **Penguji**, mencakup lembar soal yang siap dibagikan kepada peserta, kunci jawaban menggunakan Pure JavaScript (ES6+), serta rubrik penilaian teknis.

---

# BAGIAN 1: LEMBAR SOAL PESERTA

*(Bagian ini dapat langsung disalin dan dibagikan kepada peserta ujian/latihan)*

## Deskripsi Studi Kasus
Kamu diminta membangun modul logika inti untuk mengelola status karakter, peralatan (*equipment*), dan tas penyimpanan (*inventory*) dalam sebuah game RPG menggunakan **Pure JavaScript (ES6+)**.

## Aturan Main (Wajib Dipatuhi)
1. **Strict Immutability (Dilarang Mutasi):** Jangan pernah mengubah data pada object `character` secara langsung (contoh yang dilarang: `char.equipped.weapon = ...`, `char.inventory.push(...)`, atau `item.qty -= 1`). Setiap fungsi yang memodifikasi data wajib mengembalikan **object `character` baru**.
2. **Functional Array & Object Methods:** Dilarang menggunakan perulangan konvensional (`for`, `while`, `do...while`, maupun `.forEach()` untuk mutasi). Selesaikan seluruh misi menggunakan `.map()`, `.filter()`, `.reduce()`, `.find()`, `.some()`, *spread operator* (`...`), *destructuring*, serta `Object.values()` / `Object.entries()`.

---

## Starter Code & Test Cases

Salin kode berikut ke dalam file `.js` (jalankan menggunakan Node.js atau Browser Console):

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
// SCRIPT PENGUJIAN (TEST CASES)
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

---

## Spesifikasi 6 Misi

### Misi 1: `getTotalStats(char)`
Menghitung total akhir seluruh stat karakter (`hp`, `atk`, `def`, `spd`) dengan menjumlahkan `baseStats` dan seluruh `statBonus` dari item yang sedang terpasang di `equipped`.
* **Edge case:** Slot di `equipped` bisa bernilai `null`, dan `statBonus` tiap item hanya mencantumkan stat yang terpengaruh saja.

### Misi 2: `equipItem(char, itemId)`
Memasang item dari `inventory` ke slot yang sesuai pada `equipped` dan mengembalikan state `char` baru.
* **Aturan:**
  1. Jika `itemId` tidak ada di `inventory` atau `slot`-nya adalah `"consumable"`, kembalikan `char` tanpa perubahan.
  2. Kurangi `qty` item di `inventory` sebanyak `1` (hapus item dari array jika `qty` menjadi `0`).
  3. Hapus properti `qty` saat item dipasang ke `equipped`.
  4. **Swap & Stack:** Jika slot tujuan sudah terisi item lama (tidak `null`), kembalikan item lama tersebut ke `inventory`. Jika item lama sudah ada di `inventory` (berdasarkan `id`), tambahkan `qty`-nya sebesar `1`; jika belum ada, masukkan dengan `qty: 1`.

### Misi 3: `unequipItem(char, slotName)`
Melepas item dari `equipped[slotName]`, mengubah slot tersebut menjadi `null`, dan mengembalikan item tersebut ke `inventory` dengan aturan *stacking* yang sama seperti Misi 2.
* **Edge case:** Jika `slotName` tidak valid atau sudah `null`, kembalikan `char` tanpa perubahan.

### Misi 4: `useConsumable(char, itemId)`
Menggunakan item dengan `slot: "consumable"` untuk menambah nilai `baseStats` karakter secara permanen.
* **Aturan:** Validasi keberadaan item dan pastikan `slot === "consumable"`. Tambahkan nilai `statBonus` ke `baseStats`, lalu kurangi `qty` item di `inventory` sebanyak `1` (hapus jika `0`).

### Misi 5: `getInventorySummary(char)`
Mengelompokkan isi `inventory` berdasarkan `rarity` menjadi object summary berisi total kuantitas (`totalQty`) dan daftar nama item (`items`).

### Misi 6: `findBestUpgrade(char, slotName)`
Mencari satu item di `inventory` yang cocok dengan `slotName` dan memberikan kenaikan total poin stat tertinggi dibandingkan item yang sedang terpasang di `equipped[slotName]`.
* **Aturan:** Skor dihitung dari penjumlahan seluruh angka dalam `statBonus` (jika slot saat ini `null`, skor pembanding adalah `0`). Kembalikan `{ itemName, scoreGain }`, atau `null` jika tidak ada opsi upgrade yang lebih baik.
