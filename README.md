<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>PrecisionTech3D • Print Management</title>
  <script src="https://cdn.tailwindcss.com"></script>

  <!-- ✅ Supabase client (CDN) -->
  <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
</head>

<body class="bg-slate-100 p-6">

  <div class="max-w-6xl mx-auto">

    <!-- ================= CALCULATOR ================= -->
    <div class="bg-white shadow-2xl rounded-2xl p-8 mb-8">
      <h1 class="text-2xl font-bold text-center mb-6">Print Order Calculator</h1>

      <div class="grid md:grid-cols-2 gap-4">
        <div>
          <label class="text-sm text-gray-600">Customer Name</label>
          <input id="customerName" type="text" placeholder="Enter customer name" class="w-full border rounded-lg p-2 mt-1" />
        </div>

        <div>
          <label class="text-sm text-gray-600">Filament Used (grams)</label>
          <input id="gramsInput" type="number" step="0.1" placeholder="e.g., 370" class="w-full border rounded-lg p-2 mt-1" />
        </div>

        <div>
          <label class="text-sm text-gray-600">Print Time (hours)</label>
          <input id="timeInput" type="number" step="0.1" placeholder="e.g., 20" class="w-full border rounded-lg p-2 mt-1" />
        </div>

        <div>
          <label class="text-sm text-gray-600">Selling Price per Gram (₹)</label>
          <input id="sellPerGramInput" type="number" step="0.1" value="7" class="w-full border rounded-lg p-2 mt-1" />
        </div>
      </div>

      <!-- COST SETTINGS (DETAILED MANUFACTURING) -->
      <details class="mt-6">
        <summary class="cursor-pointer text-sm font-semibold text-slate-800 select-none">Manufacturing Cost Settings (Detailed)</summary>
        <div class="mt-3 bg-slate-50 border rounded-2xl p-4">
          <div class="text-xs text-slate-600 mb-3">
            These settings are saved on this device (localStorage). Manufacturing cost =
            <b>Filament + Wastage + Electricity + Packaging</b>.
          </div>

          <div class="grid md:grid-cols-3 gap-4">
            <div>
              <label class="text-xs text-slate-600">Filament cost per gram (₹/g)</label>
              <input id="cfg_filamentPerGram" type="number" step="0.01" class="w-full border rounded-lg p-2 mt-1" />
            </div>

            <div>
              <label class="text-xs text-slate-600">Wastage buffer (%)</label>
              <input id="cfg_wastagePct" type="number" step="0.1" class="w-full border rounded-lg p-2 mt-1" />
            </div>

            <div>
              <label class="text-xs text-slate-600">Packaging per order (₹)</label>
              <input id="cfg_packaging" type="number" step="0.1" class="w-full border rounded-lg p-2 mt-1" />
            </div>

            <div>
              <label class="text-xs text-slate-600">Electricity rate (₹/kWh)</label>
              <input id="cfg_elecRate" type="number" step="0.1" class="w-full border rounded-lg p-2 mt-1" />
            </div>

            <div>
              <label class="text-xs text-slate-600">Printer average power (Watts)</label>
              <input id="cfg_powerW" type="number" step="1" class="w-full border rounded-lg p-2 mt-1" />
            </div>

            <div class="flex items-end gap-2">
              <button type="button" onclick="saveCostSettings()" class="px-4 py-2 rounded-xl bg-slate-900 text-white hover:bg-slate-800 text-sm">Save Settings</button>
              <button type="button" onclick="resetCostSettings()" class="px-4 py-2 rounded-xl bg-slate-200 hover:bg-slate-300 text-sm">Reset Defaults</button>
            </div>
          </div>

          <div id="cfgStatus" class="text-xs text-slate-600 mt-3"></div>
        </div>
      </details>

      <div class="flex gap-4 mt-6">
        <button type="button" onclick="calculatePrice()" class="flex-1 bg-black text-white py-2 rounded-xl hover:bg-gray-800">Calculate</button>
        <button type="button" onclick="finalizeOrder()" class="flex-1 bg-green-600 text-white py-2 rounded-xl hover:bg-green-700">Finalize & Save</button>
      </div>

      <div id="result" class="mt-6 hidden space-y-4">
        <div class="grid md:grid-cols-2 gap-4">
          <div class="bg-slate-50 border rounded-xl p-4">
            <div class="text-gray-500 text-sm">Manufacturing Cost (Total)</div>
            <div id="manufactureCost" class="text-2xl font-bold"></div>
            <div class="mt-3 space-y-1 text-sm text-slate-700">
              <div class="flex justify-between"><span>Filament</span><span id="m_filament">₹0</span></div>
              <div class="flex justify-between"><span>Wastage</span><span id="m_wastage">₹0</span></div>
              <div class="flex justify-between"><span>Electricity</span><span id="m_electricity">₹0</span></div>
              <div class="flex justify-between"><span>Packaging</span><span id="m_packaging">₹0</span></div>
            </div>
          </div>

          <div class="bg-white border rounded-xl p-4">
            <div class="text-gray-500 text-sm">Selling Price</div>
            <div id="sellingPrice" class="text-3xl font-bold"></div>
            <div class="mt-4">
              <div class="text-gray-500 text-sm">Profit</div>
              <div id="profit" class="text-2xl font-semibold text-green-600"></div>
            </div>
          </div>
        </div>
      </div>

      <p class="text-xs text-slate-500 mt-4">
        All values shown in ₹. Edit manufacturing settings in “Manufacturing Cost Settings (Detailed)”.
      </p>
    </div>

    <!-- ================= EXCEL STYLE SHEET ================= -->
    <div class="bg-white shadow-2xl rounded-2xl p-6">
      <div class="flex flex-col md:flex-row md:items-center md:justify-between gap-3 mb-4">
        <h2 class="text-xl font-bold">Saved Orders</h2>
        <button type="button" onclick="resetAllData()" class="text-xs px-3 py-2 rounded-lg bg-slate-200 hover:bg-slate-300">Reset all data</button>
      </div>

      <!-- FILTERS -->
      <div class="grid md:grid-cols-5 gap-4 mb-4">
        <input id="filterName" type="text" placeholder="Filter by customer" class="border rounded-lg p-2 text-sm" oninput="loadOrders()" />
        <input id="minProfit" type="number" placeholder="Min profit ₹" class="border rounded-lg p-2 text-sm" oninput="loadOrders()" />
        <input id="startDate" type="date" class="border rounded-lg p-2 text-sm" onchange="loadOrders()" />
        <input id="endDate" type="date" class="border rounded-lg p-2 text-sm" onchange="loadOrders()" />
        <button type="button" onclick="clearFilters()" class="bg-gray-300 hover:bg-gray-400 py-2 rounded-lg text-sm">Clear</button>
      </div>

      <!-- TOTALS (filtered) -->
      <div class="grid md:grid-cols-5 gap-4 mb-4 text-sm bg-gray-100 p-4 rounded-lg">
        <div>Total Grams: <span id="totalGrams" class="font-bold">0</span></div>
        <div>Total Hours: <span id="totalHours" class="font-bold">0</span></div>
        <div>Total Manufacturing: ₹<span id="totalManufacturing" class="font-bold">0</span></div>
        <div>Total Selling: ₹<span id="totalSelling" class="font-bold">0</span></div>
        <div>Total Profit: ₹<span id="totalProfit" class="font-bold text-green-600">0</span></div>
      </div>

      <!-- PARTNER SPLIT -->
      <div class="bg-slate-50 border rounded-2xl p-4 mb-4">
        <div class="flex flex-col md:flex-row md:items-center md:justify-between gap-3">
          <div>
            <div class="text-sm font-semibold text-slate-800">Partner Split (Tarun 60% • Vansh 40%)</div>
            <div class="text-xs text-slate-600">Based on totals above (after applying filters/date range)</div>
          </div>
          <div class="flex items-center gap-2">
            <label class="text-xs text-slate-600">Cost payer:</label>
            <select id="costMode" class="border rounded-lg p-2 text-sm" onchange="recalcSettlement()">
              <option value="tarun">Tarun pays all manufacturing cost (machine at Tarun)</option>
              <option value="split">Split manufacturing cost 60/40</option>
            </select>
          </div>
        </div>

        <div class="grid md:grid-cols-4 gap-4 mt-4 text-sm">
          <div class="bg-white rounded-xl border p-3">
            <div class="text-slate-500 text-xs">Revenue (Selling total)</div>
            <div class="text-lg font-bold">₹<span id="revTotal">0</span></div>
          </div>
          <div class="bg-white rounded-xl border p-3">
            <div class="text-slate-500 text-xs">Manufacturing total</div>
            <div class="text-lg font-bold">₹<span id="costTotal">0</span></div>
          </div>
          <div class="bg-white rounded-xl border p-3">
            <div class="text-slate-500 text-xs">Tarun gets</div>
            <div class="text-lg font-bold">₹<span id="tarunGets">0</span></div>
            <div id="tarunNote" class="text-xs text-slate-500 mt-1"></div>
          </div>
          <div class="bg-white rounded-xl border p-3">
            <div class="text-slate-500 text-xs">Vansh gets</div>
            <div class="text-lg font-bold">₹<span id="vanshGets">0</span></div>
            <div id="vanshNote" class="text-xs text-slate-500 mt-1"></div>
          </div>
        </div>

        <!-- Reimbursement Modal (ALL orders) -->
        <div id="reimbModal" class="fixed inset-0 hidden items-center justify-center bg-black/40 p-4 z-50">
          <div class="bg-white w-full max-w-6xl rounded-2xl shadow-2xl overflow-hidden max-h-[90vh] flex flex-col">
            <div class="flex items-center justify-between p-4 border-b">
              <div>
                <div class="text-sm font-semibold">Cost Reimbursement Details (Full Split)</div>
                <div class="text-xs text-slate-500">Shows ALL saved orders • Manufacturing split by component</div>
              </div>
              <button type="button" onclick="closeReimbursements()" class="px-3 py-1.5 rounded-lg bg-slate-200 hover:bg-slate-300 text-sm">Close</button>
            </div>

            <div class="p-4 overflow-y-auto" style="-webkit-overflow-scrolling: touch;">
              <div class="grid md:grid-cols-4 gap-3 text-sm mb-3">
                <div class="bg-slate-50 border rounded-xl p-3">
                  <div class="text-xs text-slate-500">Total reimbursement (Manufacturing)</div>
                  <div class="text-lg font-bold">₹<span id="reimbTotal">0</span></div>
                </div>
                <div class="bg-slate-50 border rounded-xl p-3">
                  <div class="text-xs text-slate-500">Orders count</div>
                  <div class="text-lg font-bold"><span id="reimbCount">0</span></div>
                </div>
                <div class="bg-slate-50 border rounded-xl p-3">
                  <div class="text-xs text-slate-500">Electricity</div>
                  <div class="text-lg font-bold">₹<span id="reimbElecTotal">0</span></div>
                </div>
                <div class="bg-slate-50 border rounded-xl p-3">
                  <div class="text-xs text-slate-500">Filament</div>
                  <div class="text-lg font-bold">₹<span id="reimbFilOnlyTotal">0</span></div>
                </div>
              </div>

              <div class="grid md:grid-cols-4 gap-3 text-sm mb-4">
                <div class="bg-slate-50 border rounded-xl p-3">
                  <div class="text-xs text-slate-500">Wastage</div>
                  <div class="text-lg font-bold">₹<span id="reimbWastTotal">0</span></div>
                </div>
              </div>

              <div class="grid md:grid-cols-4 gap-3 text-sm mb-4">
                <div class="bg-slate-50 border rounded-xl p-3">
                  <div class="text-xs text-slate-500">Packaging</div>
                  <div class="text-lg font-bold">₹<span id="reimbPackTotal">0</span></div>
                </div>
                <div class="bg-slate-50 border rounded-xl p-3">
                  <div class="text-xs text-slate-500">Filament + Wastage</div>
                  <div class="text-lg font-bold">₹<span id="reimbFilTotal">0</span></div>
                </div>
                <div class="bg-slate-50 border rounded-xl p-3">
                  <div class="text-xs text-slate-500">Other (Maint+Depr+Labor+Pack)</div>
                  <div class="text-lg font-bold">₹<span id="reimbOtherTotal">0</span></div>
                </div>
                <div class="bg-slate-50 border rounded-xl p-3">
                  <div class="text-xs text-slate-500">Note</div>
                  <div class="text-xs text-slate-600">For older orders without saved split, the split is recomputed from grams+hours using current settings.</div>
                </div>
              </div>

              <div class="overflow-x-auto border rounded-xl">
                <table class="min-w-full text-sm">
                  <thead class="bg-slate-100">
                    <tr>
                      <th class="p-2 text-left border-b">Date</th>
                      <th class="p-2 text-left border-b">Customer</th>
                      <th class="p-2 text-right border-b">Total (₹)</th>
                      <th class="p-2 text-right border-b">Filament</th>
                      <th class="p-2 text-right border-b">Wastage</th>
                      <th class="p-2 text-right border-b">Electricity</th>
                      <th class="p-2 text-right border-b">Pack</th>
                      <th class="p-2 text-center border-b">Recomputed?</th>
                    </tr>
                  </thead>
                  <tbody id="reimbTbody"></tbody>
                </table>
              </div>

              <details class="mt-3">
                <summary class="cursor-pointer text-xs font-semibold text-slate-700">Show full breakdown per order (cards)</summary>
                <div id="reimbFull" class="mt-2 text-xs text-slate-700 space-y-2"></div>
              </details>
            </div>
          </div>
        </div>
      </div>

      <div class="overflow-x-auto">
        <table class="min-w-full border text-sm" id="ordersTable">
          <thead class="bg-gray-200">
            <tr>
              <th class="border p-2">Date</th>
              <th class="border p-2">Customer</th>
              <th class="border p-2">Grams</th>
              <th class="border p-2">Hours</th>
              <th class="border p-2">Manufacturing</th>
              <th class="border p-2">Selling</th>
              <th class="border p-2">Profit</th>
              <th class="border p-2">Repeat?</th>
              <th class="border p-2">Action</th>
            </tr>
          </thead>
          <tbody></tbody>
        </table>
      </div>

      <div class="mt-6">
        <h3 class="font-semibold mb-2">Repeat Customers</h3>
        <div id="repeatCustomers" class="text-sm bg-gray-50 border p-3 rounded-lg">No repeat customers yet.</div>
      </div>

      <div id="status" class="mt-4 text-xs text-slate-500"></div>
    </div>

  </div>

<script>
  // =========================
  // ✅ SUPABASE (shared + realtime)
  // =========================
  // Put these in Vercel env ideally, but for your single HTML you can paste here:
  const SUPABASE_URL = "https://YOUR_PROJECT_ID.supabase.co";
  const SUPABASE_ANON_KEY = "sb_publishable_..."; // your publishable/anon key
  const USE_SUPABASE = SUPABASE_URL.includes("supabase.co") && SUPABASE_ANON_KEY && !SUPABASE_ANON_KEY.includes("...");

  const ORDERS_TABLE = "print_orders"; // create this in Supabase

  const supabase = (USE_SUPABASE && window.supabase)
    ? window.supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY)
    : null;

  let realtimeChannel = null;

  async function dbReadOrders() {
    if (!supabase) return readOrdersLocal();
    const { data, error } = await supabase
      .from(ORDERS_TABLE)
      .select("*")
      .order("created_at", { ascending: false });

    if (error) {
      console.error(error);
      return [];
    }
    // map DB rows -> your app shape
    return (data || []).map(r => normalizeOrder({
      id: r.id,
      date: r.date,
      name: r.name,
      grams: r.grams,
      hours: r.hours,
      manufacturingCost: r.manufacturing_cost,
      sellingPrice: r.selling_price,
      profit: r.profit,
      manufacturingBreakdown: r.manufacturing_breakdown
    }));
  }

  async function dbInsertOrder(order) {
    if (!supabase) {
      const orders = readOrdersLocal();
      orders.push(normalizeOrder(order));
      writeOrdersLocal(orders);
      return;
    }
    const row = normalizeOrder(order);
    const payload = {
      date: row.date,
      name: row.name,
      grams: row.grams,
      hours: row.hours,
      manufacturing_cost: row.manufacturingCost,
      selling_price: row.sellingPrice,
      profit: row.profit,
      manufacturing_breakdown: row.manufacturingBreakdown || null,
    };
    const { error } = await supabase.from(ORDERS_TABLE).insert(payload);
    if (error) console.error(error);
  }

  async function dbDeleteOrderById(id) {
    if (!supabase) return; // local delete is handled separately
    const { error } = await supabase.from(ORDERS_TABLE).delete().eq("id", id);
    if (error) console.error(error);
  }

  function initRealtime() {
    if (!supabase) return;

    // remove old
    if (realtimeChannel) {
      supabase.removeChannel(realtimeChannel);
      realtimeChannel = null;
    }

    realtimeChannel = supabase
      .channel("pt3d-orders-realtime")
      .on("postgres_changes",
        { event: "*", schema: "public", table: ORDERS_TABLE },
        () => {
          // whenever Tarun/partner adds/edits/deletes -> refresh table
          loadOrders();
        }
      )
      .subscribe();
  }

  // =========================
  // Existing code (unchanged UI)
  // =========================
  const DEFAULT_COST_SETTINGS = {
    filamentPerGram: 1.30,
    wastagePct: 5.0,
    packaging: 0,
    elecRate: 10,
    powerW: 120,
  };

  const TARUN_SHARE = 0.60;
  const VANSH_SHARE = 0.40;
  let lastTotals = { selling: 0, manufacturing: 0, profit: 0 };
  let lastCalculation = null;

  function toNum(v, fallback = 0) {
    const n = typeof v === 'number' ? v : parseFloat(v);
    return Number.isFinite(n) ? n : fallback;
  }
  function fmt0(v) { return toNum(v, 0).toFixed(0); }
  function fmt1(v) { return toNum(v, 0).toFixed(1); }
  function todayISO() { return new Date().toISOString().split('T')[0]; }

  function getCostSettings() {
    const raw = localStorage.getItem('pt3d_costSettings');
    if (!raw) return { ...DEFAULT_COST_SETTINGS };
    try {
      const s = JSON.parse(raw);
      return {
        filamentPerGram: toNum(s.filamentPerGram, DEFAULT_COST_SETTINGS.filamentPerGram),
        wastagePct: toNum(s.wastagePct, DEFAULT_COST_SETTINGS.wastagePct),
        packaging: toNum(s.packaging, DEFAULT_COST_SETTINGS.packaging),
        elecRate: toNum(s.elecRate, DEFAULT_COST_SETTINGS.elecRate),
        powerW: toNum(s.powerW, DEFAULT_COST_SETTINGS.powerW),
      };
    } catch {
      return { ...DEFAULT_COST_SETTINGS };
    }
  }

  function setCostSettings(s) {
    localStorage.setItem('pt3d_costSettings', JSON.stringify(s));
  }

  function hydrateCostSettingsUI() {
    const s = getCostSettings();
    const setVal = (id, v) => { const el = document.getElementById(id); if (el) el.value = v; };

    setVal('cfg_filamentPerGram', s.filamentPerGram);
    setVal('cfg_wastagePct', s.wastagePct);
    setVal('cfg_packaging', s.packaging);
    setVal('cfg_elecRate', s.elecRate);
    setVal('cfg_powerW', s.powerW);

    const st = document.getElementById('cfgStatus');
    if (st) st.textContent = 'Settings loaded.';
  }

  function saveCostSettings() {
    const s = {
      filamentPerGram: toNum(document.getElementById('cfg_filamentPerGram')?.value, DEFAULT_COST_SETTINGS.filamentPerGram),
      wastagePct: toNum(document.getElementById('cfg_wastagePct')?.value, DEFAULT_COST_SETTINGS.wastagePct),
      packaging: toNum(document.getElementById('cfg_packaging')?.value, DEFAULT_COST_SETTINGS.packaging),
      elecRate: toNum(document.getElementById('cfg_elecRate')?.value, DEFAULT_COST_SETTINGS.elecRate),
      powerW: toNum(document.getElementById('cfg_powerW')?.value, DEFAULT_COST_SETTINGS.powerW),
    };

    setCostSettings(s);
    const st = document.getElementById('cfgStatus');
    if (st) st.textContent = 'Saved ✅';
  }

  function resetCostSettings() {
    setCostSettings({ ...DEFAULT_COST_SETTINGS });
    hydrateCostSettingsUI();
    const st = document.getElementById('cfgStatus');
    if (st) st.textContent = 'Reset to defaults ✅';
  }

  function computeManufacturing(grams, hours) {
    const s = getCostSettings();

    const filament = grams * s.filamentPerGram;
    const wastage = filament * (s.wastagePct / 100);

    const kWh = hours * (s.powerW / 1000);
    const electricity = kWh * s.elecRate;

    const packaging = s.packaging;

    const total = filament + wastage + electricity + packaging;

    return { filament, wastage, electricity, packaging, total, settingsSnapshot: { ...s } };
  }

  function setManufacturingBreakdownUI(b) {
    const set = (id, v) => { const el = document.getElementById(id); if (el) el.textContent = '₹' + fmt0(v); };
    set('m_filament', b.filament);
    set('m_wastage', b.wastage);
    set('m_electricity', b.electricity);
    set('m_packaging', b.packaging);
  }

  function isISODate(s) {
    return typeof s === 'string' && /^[0-9]{4}-[0-9]{2}-[0-9]{2}$/.test(s);
  }

  function normalizeOrder(o) {
    const date = isISODate(o?.date) ? o.date : todayISO();
    const name = (typeof o?.name === 'string' && o.name.trim()) ? o.name.trim() : 'Unknown';

    const grams = toNum(o?.grams, 0);
    const hours = toNum(o?.hours, 0);

    let breakdown = (o && typeof o.manufacturingBreakdown === 'object' && o.manufacturingBreakdown)
      ? o.manufacturingBreakdown
      : null;

    let manufacturingCost = toNum(o?.manufacturingCost, NaN);
    let sellingPrice = toNum(o?.sellingPrice, NaN);
    let profit = toNum(o?.profit, NaN);

    if (!Number.isFinite(manufacturingCost)) {
      const b = (breakdown && Number.isFinite(toNum(breakdown.total, NaN)))
        ? { ...breakdown, total: toNum(breakdown.total, 0) }
        : computeManufacturing(grams, hours);
      breakdown = breakdown || b;
      manufacturingCost = toNum(b.total, 0);
    }

    if (!Number.isFinite(sellingPrice)) sellingPrice = 0;
    if (!Number.isFinite(profit)) profit = sellingPrice - manufacturingCost;

    return { id: o?.id, date, name, grams, hours, manufacturingCost, sellingPrice, profit, manufacturingBreakdown: breakdown };
  }

  // --------------------
  // ✅ Local fallback (kept)
  // --------------------
  function readOrdersLocal() {
    const raw = JSON.parse(localStorage.getItem('printOrders') || '[]');
    const normalized = Array.isArray(raw) ? raw.map(normalizeOrder) : [];
    localStorage.setItem('printOrders', JSON.stringify(normalized));
    return normalized;
  }

  function writeOrdersLocal(orders) {
    localStorage.setItem('printOrders', JSON.stringify(orders));
  }

  // --------------------
  // Settlement unchanged
  // --------------------
  function computeSettlement(revenue, cost, profit, mode) {
    if (mode === 'tarun') {
      return {
        tarunGets: cost + (profit * TARUN_SHARE),
        vanshGets: profit * VANSH_SHARE,
        tarunNote: `Cost reimbursement ₹${fmt0(cost)} + 60% profit (₹${fmt0(profit * TARUN_SHARE)})`,
        vanshNote: `40% profit only`
      };
    }
    return {
      tarunGets: revenue * TARUN_SHARE,
      vanshGets: revenue * VANSH_SHARE,
      tarunNote: `60% of revenue (cost split)`,
      vanshNote: `40% of revenue (cost split)`
    };
  }

  function recalcSettlement() {
    const modeEl = document.getElementById('costMode');
    if (!modeEl) return;

    const mode = modeEl.value;
    const revenue = toNum(lastTotals.selling, 0);
    const cost = toNum(lastTotals.manufacturing, 0);
    const profit = toNum(lastTotals.profit, 0);

    const s = computeSettlement(revenue, cost, profit, mode);

    const revEl = document.getElementById('revTotal');
    const costEl = document.getElementById('costTotal');
    const tarEl = document.getElementById('tarunGets');
    const vanEl = document.getElementById('vanshGets');
    const tarNote = document.getElementById('tarunNote');
    const vanNote = document.getElementById('vanshNote');

    if (!revEl || !costEl || !tarEl || !vanEl || !tarNote || !vanNote) return;

    revEl.textContent = fmt0(revenue);
    costEl.textContent = fmt0(cost);
    tarEl.textContent = fmt0(s.tarunGets);
    vanEl.textContent = fmt0(s.vanshGets);

    if (mode === 'tarun') {
      tarNote.innerHTML = `
        <button type="button" onclick="openReimbursements()" class="text-xs text-blue-700 underline hover:text-blue-900">
          Cost reimbursement ₹${fmt0(cost)} (click)
        </button>
        <span class="text-xs text-slate-500"> + 60% profit (₹${fmt0(profit * TARUN_SHARE)})</span>
      `;
      vanNote.textContent = '40% profit only';
    } else {
      tarNote.textContent = s.tarunNote;
      vanNote.textContent = s.vanshNote;
    }
  }

  function calculatePrice() {
    const name = document.getElementById('customerName').value.trim();
    const grams = parseFloat(document.getElementById('gramsInput').value);
    const hours = parseFloat(document.getElementById('timeInput').value);
    const sellPerGram = parseFloat(document.getElementById('sellPerGramInput').value);

    if (!name || Number.isNaN(grams) || Number.isNaN(hours) || Number.isNaN(sellPerGram)) {
      alert('Please fill all fields.');
      return;
    }

    const b = computeManufacturing(grams, hours);
    const manufacturingCost = b.total;
    const sellingPrice = grams * sellPerGram;
    const profit = sellingPrice - manufacturingCost;

    document.getElementById('manufactureCost').textContent = '₹' + fmt0(manufacturingCost);
    setManufacturingBreakdownUI(b);

    document.getElementById('sellingPrice').textContent = '₹' + fmt0(sellingPrice);
    document.getElementById('profit').textContent = '₹' + fmt0(profit);
    document.getElementById('result').classList.remove('hidden');

    lastCalculation = { date: todayISO(), name, grams, hours, manufacturingCost, manufacturingBreakdown: b, sellingPrice, profit };
  }

  // ✅ NOW SAVES TO SUPABASE (shared) + triggers realtime for both users
  async function finalizeOrder() {
    if (!lastCalculation) {
      alert('Calculate first.');
      return;
    }

    await dbInsertOrder(lastCalculation);
    await loadOrders();
    alert('Order saved!');
  }

  // ✅ NOW LOADS FROM SUPABASE (shared)
  async function loadOrders() {
    const table = document.getElementById('ordersTable');
    const tbody = table ? table.querySelector('tbody') : null;
    if (!tbody) {
      console.error('ordersTable/tbody not found in DOM');
      return;
    }

    const orders = await dbReadOrders();
    tbody.innerHTML = '';

    const nameFilter = (document.getElementById('filterName')?.value || '').toLowerCase().trim();
    const minProfit = parseFloat(document.getElementById('minProfit')?.value || '');
    const startDate = document.getElementById('startDate')?.value || '';
    const endDate = document.getElementById('endDate')?.value || '';

    let totalGrams = 0, totalHours = 0, totalManufacturing = 0, totalSelling = 0, totalProfit = 0;

    const customerCount = {};
    orders.forEach(o => { customerCount[o.name] = (customerCount[o.name] || 0) + 1; });

    let shown = 0;

    orders.forEach((rawOrder, index) => {
      const order = normalizeOrder(rawOrder);

      if (nameFilter && !order.name.toLowerCase().includes(nameFilter)) return;
      if (!Number.isNaN(minProfit) && toNum(order.profit, 0) < minProfit) return;
      if (startDate && order.date < startDate) return;
      if (endDate && order.date > endDate) return;

      shown += 1;

      totalGrams += order.grams;
      totalHours += order.hours;
      totalManufacturing += order.manufacturingCost;
      totalSelling += order.sellingPrice;
      totalProfit += order.profit;

      const repeatBadge = customerCount[order.name] > 1
        ? '<span class="text-red-600 font-bold">Yes</span>'
        : 'No';

      const b = order.manufacturingBreakdown;
      const tip = b ?
        `Filament ₹${fmt0(b.filament)} | Wastage ₹${fmt0(b.wastage)} | Elec ₹${fmt0(b.electricity)} | Pack ₹${fmt0(b.packaging)}`
        : '';

      const delHandler = supabase
        ? `deleteOrderById("${order.id}")`
        : `deleteOrderLocal(${index})`;

      tbody.insertAdjacentHTML('beforeend', `
        <tr>
          <td class="border p-2">${order.date}</td>
          <td class="border p-2">${order.name}</td>
          <td class="border p-2">${fmt0(order.grams)}</td>
          <td class="border p-2">${fmt1(order.hours)}</td>
          <td class="border p-2" title="${tip}">₹${fmt0(order.manufacturingCost)}</td>
          <td class="border p-2">₹${fmt0(order.sellingPrice)}</td>
          <td class="border p-2 text-green-600">₹${fmt0(order.profit)}</td>
          <td class="border p-2 text-center">${repeatBadge}</td>
          <td class="border p-2 text-center">
            <button type="button" onclick='${delHandler}' class="bg-red-500 text-white px-2 py-1 rounded text-xs hover:bg-red-600">Delete</button>
          </td>
        </tr>
      `);
    });

    document.getElementById('totalGrams').textContent = fmt0(totalGrams);
    document.getElementById('totalHours').textContent = fmt1(totalHours);
    document.getElementById('totalManufacturing').textContent = fmt0(totalManufacturing);
    document.getElementById('totalSelling').textContent = fmt0(totalSelling);
    document.getElementById('totalProfit').textContent = fmt0(totalProfit);

    lastTotals = { selling: totalSelling, manufacturing: totalManufacturing, profit: totalProfit };
    recalcSettlement();

    const repeats = Object.entries(customerCount)
      .filter(([, c]) => c > 1)
      .sort((a, b) => b[1] - a[1]);

    const repeatBox = document.getElementById('repeatCustomers');
    if (repeatBox) {
      repeatBox.innerHTML = repeats.length
        ? repeats.map(([n, c]) => `${n} (${c} orders)`).join('<br>')
        : 'No repeat customers yet.';
    }

    const status = document.getElementById('status');
    if (status) status.textContent =
      (supabase ? `✅ Supabase shared DB ON • ` : `⚠️ Supabase OFF (local only) • `) +
      `Showing ${shown} order(s) • Total loaded: ${orders.length}`;
  }

  // ✅ Delete in Supabase
  async function deleteOrderById(id) {
    if (!id) return;
    if (!confirm('Delete this order?')) return;
    await dbDeleteOrderById(id);
    await loadOrders();
  }

  // Local delete fallback
  function deleteOrderLocal(index) {
    const orders = readOrdersLocal();
    if (!confirm('Delete this order?')) return;
    orders.splice(index, 1);
    writeOrdersLocal(orders);
    loadOrders();
  }

  function clearFilters() {
    document.getElementById('filterName').value = '';
    document.getElementById('minProfit').value = '';
    document.getElementById('startDate').value = '';
    document.getElementById('endDate').value = '';
    loadOrders();
  }

  // ✅ Reset (Supabase or local)
  async function resetAllData() {
    if (!confirm('This will delete ALL saved orders. Continue?')) return;

    if (supabase) {
      // delete all rows
      const { error } = await supabase.from(ORDERS_TABLE).delete().neq("id", "00000000-0000-0000-0000-000000000000");
      if (error) console.error(error);
    } else {
      localStorage.removeItem('printOrders');
    }
    await loadOrders();
  }

  // ---------- Reimbursement Modal (ALL orders) ----------
  async function openReimbursements() {
    const modal = document.getElementById('reimbModal');
    const tbody = document.getElementById('reimbTbody');
    const full = document.getElementById('reimbFull');
    if (!modal || !tbody || !full) return;

    const orders = await dbReadOrders(); // ALL orders (shared if Supabase)

    function getBreakdownSafe(order) {
      const b0 = order?.manufacturingBreakdown;
      const hasSplit = b0 && typeof b0 === 'object' && ['filament','wastage','electricity','packaging','total']
        .every(k => Number.isFinite(toNum(b0[k], NaN)));
      if (hasSplit) {
        return { b: {
          filament: toNum(b0.filament, 0),
          wastage: toNum(b0.wastage, 0),
          electricity: toNum(b0.electricity, 0),
          packaging: toNum(b0.packaging, 0),
          total: toNum(b0.total, 0)
        }, recomputed: false };
      }
      const b = computeManufacturing(toNum(order.grams, 0), toNum(order.hours, 0));
      return { b, recomputed: true };
    }

    let total = 0, elecTotal = 0, filOnlyTotal = 0, wastTotal = 0, packTotal = 0;

    const rows = orders.map(o => {
      const order = normalizeOrder(o);
      const { b, recomputed } = getBreakdownSafe(order);
      total += toNum(order.manufacturingCost, toNum(b.total, 0));
      filOnlyTotal += toNum(b.filament, 0);
      wastTotal += toNum(b.wastage, 0);
      elecTotal += toNum(b.electricity, 0);
      packTotal += toNum(b.packaging, 0);
      return { order, b, recomputed };
    });

    const filTotal = filOnlyTotal + wastTotal;
    const otherTotal = packTotal;

    document.getElementById('reimbTotal').textContent = fmt0(total);
    document.getElementById('reimbCount').textContent = String(rows.length);
    document.getElementById('reimbElecTotal').textContent = fmt0(elecTotal);
    document.getElementById('reimbFilOnlyTotal').textContent = fmt0(filOnlyTotal);
    document.getElementById('reimbWastTotal').textContent = fmt0(wastTotal);
    document.getElementById('reimbPackTotal').textContent = fmt0(packTotal);
    document.getElementById('reimbFilTotal').textContent = fmt0(filTotal);
    document.getElementById('reimbOtherTotal').textContent = fmt0(otherTotal);

    tbody.innerHTML = '';
    full.innerHTML = '';

    rows.forEach(({ order, b, recomputed }) => {
      tbody.insertAdjacentHTML('beforeend', `
        <tr class="border-t">
          <td class="p-2">${order.date}</td>
          <td class="p-2">${order.name}</td>
          <td class="p-2 text-right font-semibold">${fmt0(order.manufacturingCost)}</td>
          <td class="p-2 text-right">${fmt0(b.filament)}</td>
          <td class="p-2 text-right">${fmt0(b.wastage)}</td>
          <td class="p-2 text-right">${fmt0(b.electricity)}</td>
          <td class="p-2 text-right">${fmt0(b.packaging)}</td>
          <td class="p-2 text-center">${recomputed ? '<span class="text-amber-700 font-semibold">Yes</span>' : 'No'}</td>
        </tr>
      `);

      full.insertAdjacentHTML('beforeend', `
        <div class="bg-slate-50 border rounded-xl p-3">
          <div class="flex items-center justify-between">
            <div class="font-semibold">${order.date} • ${order.name} ${recomputed ? '<span class="ml-2 text-amber-700">(recomputed)</span>' : ''}</div>
            <div class="font-bold">Total ₹${fmt0(order.manufacturingCost)}</div>
          </div>
          <div class="mt-2 grid md:grid-cols-4 gap-2">
            <div>Filament: ₹${fmt0(b.filament)}</div>
            <div>Wastage: ₹${fmt0(b.wastage)}</div>
            <div>Electricity: ₹${fmt0(b.electricity)}</div>
            <div>Packaging: ₹${fmt0(b.packaging)}</div>
          </div>
        </div>
      `);
    });

    modal.classList.remove('hidden');
    modal.classList.add('flex');
  }

  function closeReimbursements() {
    const modal = document.getElementById('reimbModal');
    if (!modal) return;
    modal.classList.add('hidden');
    modal.classList.remove('flex');
  }

  document.addEventListener('click', (e) => {
    const modal = document.getElementById('reimbModal');
    if (!modal || modal.classList.contains('hidden')) return;
    if (e.target === modal) closeReimbursements();
  });

  hydrateCostSettingsUI();
  loadOrders().then(() => {
    initRealtime(); // ✅ realtime refresh for both Tarun + partner
  });

</script>

</body>
</html>
