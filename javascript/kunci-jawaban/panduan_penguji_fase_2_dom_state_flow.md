# Modul Evaluasi JavaScript (Fase 2): Integrasi Pure Logic ke Manipulasi DOM

Dokumen ini merupakan kelanjutan dari **Fase 1 (Manipulasi Array & Object)**. Pada fase ini, kandidat diuji kemampuannya menghubungkan *pure functions* yang telah dibuat sebelumnya ke antarmuka pengguna (UI) menggunakan **Vanilla JavaScript (DOM API)** dengan pola *Unidirectional Data Flow*.

# BAGIAN 1: LEMBAR SOAL PESERTA (FASE 2)

*(Bagian ini dapat langsung dibagikan kepada peserta ujian/latihan)*

## Deskripsi Studi Kasus

Kamu telah berhasil membangun mesin logika karakter RPG (*State Management Layer*) pada Fase 1. Tugasmu sekarang adalah membangun antarmuka interaktif berbasis web menggunakan **Pure Vanilla JavaScript** tanpa mengubah satu baris pun logika inti dari 6 fungsi pada Fase 1.

## Aturan Main (Wajib Dipatuhi)

1. **Separation of Concerns (Pemisahan Logika & UI):** Dilarang menulis kalkulasi stat atau manipulasi array `inventory` di dalam fungsi *render* maupun *event listener*. Panggil fungsi-fungsi dari Fase 1 (`getTotalStats`, `equipItem`, `unequipItem`, `useConsumable`, `getInventorySummary`, `findBestUpgrade`).

2. **Unidirectional Data Flow (State → Render → Event → Update State):** Semua perubahan tampilan harus berasal dari satu sumber kebenaran (*Single Source of Truth*) bernama `appState`. Jangan memanipulasi teks/elemen DOM secara manual per tombol (seperti mengubah angka HP satu per satu saat tombol diklik); perbarui `appState`, lalu panggil ulang fungsi `render()`.

3. **Event Delegation:** Dilarang menggunakan atribut inline `onclick="..."` di dalam HTML dan dilarang memasang *event listener* di dalam *loop* untuk setiap kartu item. Gunakan satu *event listener* terpusat pada elemen pembungkus utama (`#app`).

## Starter Code (`index.html` & `app.js`)

Buat dua file dalam satu folder (`index.html` dan `app.js`), lalu salin kode dasar berikut:

### 1. File `index.html`

```
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
      --warning: #f59e0b;
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

### 2. File `app.js` (Kerangka Awal)

```
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

// TODO: Tempelkan 6 fungsi pure dari Fase 1 di sini
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

## Spesifikasi 4 Misi DOM

### Misi DOM 1: Dynamic Stat Panel & Equipment Slots (`renderStats` & `renderEquipped`)

* **Header & Stats:** Tampilkan nama dan level karakter pada `#char-header` (contoh: `Aether — Lv. 12`), lalu render ke-4 stat (`hp`, `atk`, `def`, `spd`) ke dalam `#stats-container` menggunakan hasil dari `getTotalStats(appState.character)`.

* **Equipment Slots:** Render 3 slot (`weapon`, `armor`, `accessory`) ke `#equipped-container`.

  * Jika slot terisi item, tampilkan nama item, rincian bonus stat, dan tombol **"Unequip"** (`data-action="unequip"` dan `data-slot="..."`).

  * Jika slot bernilai `null`, tampilkan teks *"Empty Slot"* tanpa tombol aksi.

### Misi DOM 2: Interactive Inventory & Smart Upgrade Badge (`renderInventory`)

* Render daftar item dari `appState.character.inventory` (yang sudah difilter berdasarkan `appState.activeRarity`) ke `#inventory-container`.

* **Conditional Action Button:**

  * Jika `item.slot === "consumable"`, tampilkan tombol **"Use"** (`data-action="consume"` dan `data-id="..."`).

  * Jika bukan `consumable`, tampilkan tombol **"Equip"** (`data-action="equip"` dan `data-id="..."`).

* **Smart Upgrade Indicator:** Manfaatkan fungsi `findBestUpgrade(appState.character, slot)` pada saat render. Jika suatu item di inventory merupakan *upgrade* terbaik untuk slot-nya, tampilkan badge hijau: `▲ Best Upgrade (+X)` di sebelah nama item.

### Misi DOM 3: Live Rarity Filter Tabs (`renderFilters`)

* Render tombol filter ke `#filter-container` untuk kategori: `all`, `common`, `uncommon`, `rare`, dan `epic`.

* Gunakan fungsi `getInventorySummary(appState.character)` untuk menampilkan jumlah total kuantitas item pada setiap tab (contoh: `All (7)`, `Rare (4)`, `Common (0)`).

* Berikan class `.active` pada tab yang sesuai dengan `appState.activeRarity`.

### Misi DOM 4: Centralized Event Delegation, Activity Log & Reset

* Pasang **satu** `addEventListener("click", ...)` pada elemen `#app` untuk menangani seluruh aksi (`equip`, `unequip`, `consume`, `filter`, dan `reset`).

* Setiap kali pemain melakukan aksi (`equip`, `unequip`, atau `consume`), tambahkan pesan aktivitas baru ke bagian paling atas `appState.logs` (maksimal simpan 5 log terakhir secara *immutable*).

* Jika tombol **Reset Data** (`data-action="reset"`) diklik, kembalikan `appState` ke kondisi awal menggunakan `structuredClone(initialCharacter)`.

# BAGIAN 2: KUNCI JAWABAN LENGKAP (KHUSUS PENGUJI)

## Implementasi Lengkap `app.js`

```
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

// --- Helper & Pure Functions (Fase 1) ---
const mergeStats = (baseStats, bonusStats = {}) =>
  Object.entries(bonusStats).reduce(
    (acc, [key, val]) => ({ ...acc, [key]: (acc[key] || 0) + val }),
    { ...baseStats }
  );

const decrementInventoryItem = (inventory, itemId) =>
  inventory
    .map((item) => (item.id === itemId ? { ...item, qty: item.qty - 1 } : item))
    .filter((item) => item.qty > 0);

const incrementInventoryItem = (inventory, itemToReturn) => {
  const exists = inventory.some((item) => item.id === itemToReturn.id);
  return exists
    ? inventory.map((item) =>
        item.id === itemToReturn.id ? { ...item, qty: item.qty + 1 } : item
      )
    : [...inventory, { ...itemToReturn, qty: 1 }];
};

function getTotalStats(char) {
  return Object.values(char.equipped)
    .filter(Boolean)
    .reduce((total, item) => mergeStats(total, item.statBonus), { ...char.baseStats });
}

function equipItem(char, itemId) {
  const targetItem = char.inventory.find((i) => i.id === itemId);
  if (!targetItem || targetItem.slot === "consumable") return char;

  const targetSlot = targetItem.slot;
  const currentItem = char.equipped[targetSlot];
  const { qty, ...cleanItem } = targetItem;

  let nextInventory = decrementInventoryItem(char.inventory, itemId);
  if (currentItem) {
    nextInventory = incrementInventoryItem(nextInventory, currentItem);
  }

  return {
    ...char,
    equipped: { ...char.equipped, [targetSlot]: cleanItem },
    inventory: nextInventory,
  };
}

function unequipItem(char, slotName) {
  if (!Object.hasOwn(char.equipped, slotName) || !char.equipped[slotName]) return char;
  const itemToRemove = char.equipped[slotName];

  return {
    ...char,
    equipped: { ...char.equipped, [slotName]: null },
    inventory: incrementInventoryItem(char.inventory, itemToRemove),
  };
}

function useConsumable(char, itemId) {
  const targetItem = char.inventory.find((i) => i.id === itemId);
  if (!targetItem || targetItem.slot !== "consumable") return char;

  return {
    ...char,
    baseStats: mergeStats(char.baseStats, targetItem.statBonus),
    inventory: decrementInventoryItem(char.inventory, itemId),
  };
}

function getInventorySummary(char) {
  return char.inventory.reduce((summary, item) => {
    const group = summary[item.rarity] || { totalQty: 0, items: [] };
    return {
      ...summary,
      [item.rarity]: {
        totalQty: group.totalQty + item.qty,
        items: [...group.items, item.name],
      },
    };
  }, {});
}

function findBestUpgrade(char, slotName) {
  if (!Object.hasOwn(char.equipped, slotName)) return null;
  const getScore = (item) =>
    item?.statBonus ? Object.values(item.statBonus).reduce((s, v) => s + v, 0) : 0;

  const currentScore = getScore(char.equipped[slotName]);

  return char.inventory
    .filter((item) => item.slot === slotName)
    .map((item) => ({ itemName: item.name, scoreGain: getScore(item) - currentScore }))
    .filter((candidate) => candidate.scoreGain > 0)
    .reduce((best, curr) => (!best || curr.scoreGain > best.scoreGain ? curr : best), null);
}

// ==========================================
// 2. APPLICATION STATE & UI HELPERS
// ==========================================
let appState = {
  character: structuredClone(initialCharacter),
  activeRarity: "all",
  logs: ["Karakter siap bertualang."],
};

// Helper format stat bonus menjadi string (misal: "+28 ATK, +5 SPD")
const formatStatBonus = (statBonus = {}) =>
  Object.entries(statBonus)
    .map(([stat, val]) => `${val > 0 ? `+${val}` : val} ${stat.toUpperCase()}`)
    .join(", ");

// Helper menambahkan log secara immutable (maksimal 5 entri)
const prependLog = (logs, message) => {
  const timestamp = new Date().toLocaleTimeString("id-ID", { hour12: false });
  return [`[${timestamp}] ${message}`, ...logs].slice(0, 5);
};

// Central State Updater
function setState(updater) {
  appState = typeof updater === "function" ? updater(appState) : { ...appState, ...updater };
  render();
}

// ==========================================
// 3. DOM RENDER FUNCTIONS
// ==========================================
function renderStatsAndHeader() {
  const { character } = appState;
  const totalStats = getTotalStats(character);

  document.getElementById("char-header").textContent = `${character.name} — Lv. ${character.level}`;

  const statsHTML = Object.entries(totalStats)
    .map(
      ([statName, value]) => `
      <div class="stat-box">
        <div class="stat-label">${statName}</div>
        <div class="stat-value">${value}</div>
      </div>
    `
    )
    .join("");

  document.getElementById("stats-container").innerHTML = statsHTML;
}

function renderEquipped() {
  const { equipped } = appState.character;

  const equippedHTML = Object.entries(equipped)
    .map(([slotName, item]) => {
      if (!item) {
        return `
          <div class="slot-item">
            <div>
              <div class="slot-title">${slotName}</div>
              <div class="empty-text">Empty Slot</div>
            </div>
          </div>
        `;
      }

      return `
        <div class="slot-item">
          <div>
            <div class="slot-title">${slotName}</div>
            <strong>${item.name}</strong>
            <div class="bonus-text">${formatStatBonus(item.statBonus)}</div>
          </div>
          <button class="action-btn btn-unequip" data-action="unequip" data-slot="${slotName}">
            Unequip
          </button>
        </div>
      `;
    })
    .join("");

  document.getElementById("equipped-container").innerHTML = equippedHTML;
}

function renderFilters() {
  const { character, activeRarity } = appState;
  const summary = getInventorySummary(character);
  const totalAllQty = Object.values(summary).reduce((sum, group) => sum + group.totalQty, 0);

  const rarities = ["all", "common", "uncommon", "rare", "epic"];

  const filtersHTML = rarities
    .map((rarity) => {
      const count = rarity === "all" ? totalAllQty : summary[rarity]?.totalQty || 0;
      const isActive = activeRarity === rarity ? "active" : "";
      const label = rarity.charAt(0).toUpperCase() + rarity.slice(1);

      return `
        <button class="tab-btn ${isActive}" data-action="filter" data-rarity="${rarity}">
          ${label} (${count})
        </button>
      `;
    })
    .join("");

  document.getElementById("filter-container").innerHTML = filtersHTML;
}

function renderInventory() {
  const { character, activeRarity } = appState;

  // Hitung upgrade terbaik untuk tiap slot sekali saja saat render
  const bestUpgrades = {
    weapon: findBestUpgrade(character, "weapon"),
    armor: findBestUpgrade(character, "armor"),
    accessory: findBestUpgrade(character, "accessory"),
  };

  const visibleItems =
    activeRarity === "all"
      ? character.inventory
      : character.inventory.filter((item) => item.rarity === activeRarity);

  if (visibleItems.length === 0) {
    document.getElementById("inventory-container").innerHTML =
      `<p class="empty-text">Tidak ada item pada kategori ini.</p>`;
    return;
  }

  const inventoryHTML = visibleItems
    .map((item) => {
      const isConsumable = item.slot === "consumable";
      const bestForSlot = bestUpgrades[item.slot];
      const isBestUpgrade = !isConsumable && bestForSlot && bestForSlot.itemName === item.name;

      const upgradeBadge = isBestUpgrade
        ? `<span class="badge badge-upgrade">▲ Best Upgrade (+${bestForSlot.scoreGain})</span>`
        : "";

      const actionButton = isConsumable
        ? `<button class="action-btn btn-use" data-action="consume" data-id="${item.id}">Use</button>`
        : `<button class="action-btn btn-equip" data-action="equip" data-id="${item.id}">Equip</button>`;

      return `
        <div class="inv-item">
          <div>
            <div>
              <strong>${item.name}</strong> (x${item.qty})
              <span class="badge badge-rarity">${item.rarity}</span>
              ${upgradeBadge}
            </div>
            <div class="bonus-text">[${item.slot.toUpperCase()}] ${formatStatBonus(item.statBonus)}</div>
          </div>
          ${actionButton}
        </div>
      `;
    })
    .join("");

  document.getElementById("inventory-container").innerHTML = inventoryHTML;
}

function renderLogs() {
  const logsHTML = appState.logs.map((entry) => `<li>${entry}</li>`).join("");
  document.getElementById("log-container").innerHTML = logsHTML;
}

// Master Render Function
function render() {
  renderStatsAndHeader();
  renderEquipped();
  renderFilters();
  renderInventory();
  renderLogs();
}

// ==========================================
// 4. EVENT DELEGATION & INITIALIZATION
// ==========================================
document.getElementById("app").addEventListener("click", (event) => {
  const trigger = event.target.closest("[data-action]");
  if (!trigger) return;

  const { action, id, slot, rarity } = trigger.dataset;

  if (action === "equip") {
    const item = appState.character.inventory.find((i) => i.id === id);
    if (!item) return;

    setState((prev) => ({
      ...prev,
      character: equipItem(prev.character, id),
      logs: prependLog(prev.logs, `Memasang ${item.name} ke slot ${item.slot}.`),
    }));
  }

  if (action === "unequip") {
    const item = appState.character.equipped[slot];
    if (!item) return;

    setState((prev) => ({
      ...prev,
      character: unequipItem(prev.character, slot),
      logs: prependLog(prev.logs, `Melepas ${item.name} dari slot ${slot}.`),
    }));
  }

  if (action === "consume") {
    const item = appState.character.inventory.find((i) => i.id === id);
    if (!item) return;

    setState((prev) => ({
      ...prev,
      character: useConsumable(prev.character, id),
      logs: prependLog(prev.logs, `Menggunakan ${item.name} (${formatStatBonus(item.statBonus)}).`),
    }));
  }

  if (action === "filter") {
    setState({ activeRarity: rarity });
  }

  if (action === "reset") {
    setState({
      character: structuredClone(initialCharacter),
      activeRarity: "all",
      logs: prependLog([], "Data karakter di-reset ke kondisi awal."),
    });
  }
});

// Render perdana saat halaman dimuat
render();

```

## Catatan Teknis untuk Penguji (Poin Kritis Evaluasi DOM)

1. **Pemisahan UI State vs Domain State:**
   Perhatikan bagaimana kandidat menyimpan status tab filter (`activeRarity`). Kandidat yang baik menyimpannya di variabel terpisah di dalam `appState` dan hanya memfilter array saat proses `renderInventory()` berlangsung—**bukan** menimpa/menghapus isi `appState.character.inventory`.

2. **Penggunaan `event.target.closest("[data-action]")`:**
   Dalam *event delegation*, kandidat pemula sering menulis `if (event.target.dataset.action === ...)`. Jika di dalam `<button>` terdapat elemen anak seperti `<span>` atau ikon SVG, `event.target` akan merujuk ke elemen anak tersebut dan gagal membaca `data-action`. Penggunaan `.closest("[data-action]")` menunjukkan pemahaman DOM yang matang.

3. **Zero Logic Duplication:**
   Pastikan kandidat tidak menulis ulang rumus penjumlahan stat atau logika pengurangan `qty` di dalam fungsi render maupun event listener. Semua harus didelegasikan ke fungsi Fase 1.

4. **Penggunaan `structuredClone` saat Reset:**
   Jika kandidat melakukan reset dengan `appState.character = initialCharacter` (tanpa kloning), hal itu masih aman *selama* seluruh fungsi Fase 1 benar-benar 100% *immutable*. Namun penggunaan `structuredClone(initialCharacter)` adalah praktik defensif yang sangat baik.

# BAGIAN 3: RUBRIK PENILAIAN FASE 2 & PERTANYAAN LANJUTAN

| Kriteria Penilaian | Bobot | Indikator Sangat Baik (85–100%) | Indikator Cukup (60–84%) | Indikator Kurang (<60%) | 
| ----- | ----- | ----- | ----- | ----- | 
| **1. Arsitektur State & Separation of Concerns** | **30%** | Memisahkan *domain state* (`character`) dan *UI state* (`activeRarity`); memanggil 100% fungsi Fase 1 tanpa duplikasi logika. | Menggunakan sebagian fungsi Fase 1, namun menulis ulang beberapa logika filter/kalkulasi langsung di dalam fungsi render. | Mencampuradukkan kalkulasi stat dan mutasi data langsung di dalam *event listener* DOM. | 
| **2. Event Delegation & Interaktivitas** | **30%** | Menggunakan satu *event listener* terpusat di `#app` dengan `.closest("[data-action]")` dan atribut `data-*` yang bersih. | Menggunakan *event delegation* namun memakai `event.target` langsung tanpa `.closest()`, atau memasang beberapa listener per container. | Memasang `addEventListener` di dalam `.map()` / loop setiap kali `render()` dipanggil (berpotensi *memory leak* / *double trigger*). | 
| **3. Kelengkapan Fitur & Edge Cases UI** | **25%** | Seluruh 4 misi DOM berjalan mulus: badge *Best Upgrade* muncul dinamis, counter rarity akurat, pesan kosong tampil saat filter nihil, serta log & reset berfungsi. | Fitur utama (equip/unequip/use) berjalan, namun badge *Best Upgrade* atau counter pada tab filter tidak sinkron setelah state berubah. | Tampilan rusak atau *error* saat slot kosong (`null`) atau saat item terakhir di inventory habis terpakai. | 
| **4. Kebersihan & Modularitas Kode UI** | **15%** | Memecah proses render menjadi fungsi-fungsi kecil yang terfokus (`renderStats`, `renderEquipped`, `renderInventory`, dll.). | Seluruh HTML digabung menjadi satu blok string raksasa di dalam satu fungsi `render()` yang panjang. | Kode sulit dibaca, banyak `document.querySelector` berulang-ulang tanpa struktur yang jelas. | 

### Pertanyaan Diskusi Lanjutan untuk Kandidat

1. *"Saat ini fungsi `render()` mengganti seluruh `innerHTML` di setiap container setiap kali state berubah. Apa kelemahan pendekatan ini jika aplikasi berkembang menjadi jauh lebih besar (misalnya punya 500 item dan input pencarian teks yang sedang diketik user)?"*

   * **Arah Jawaban:** Kehilangan fokus (*focus loss*) pada elemen `<input>`, hilangnya posisi *scroll*, dan biaya *re-parse / repaint* DOM yang besar. Solusinya di Vanilla JS adalah melakukan *targeted DOM patching* (hanya memperbarui bagian yang berubah) atau menggunakan konsep *Virtual DOM / fine-grained reactivity* seperti pada framework modern.

2. *"Jika data `item.name` berasal dari input pengguna (misalnya fitur rename senjata), mengapa menyisipkan `${item.name}` langsung ke dalam `innerHTML` berbahaya, dan bagaimana cara mengamankannya di Vanilla JS?"*

   * **Arah Jawaban:** Rentan terhadap serangan **XSS (Cross-Site Scripting)**. Cara mengamankannya adalah dengan menggunakan `document.createElement()` dan `.textContent` untuk data dinamis dari user, atau membuat fungsi *HTML sanitizer / escaping* sebelum disisipkan ke `innerHTML`.