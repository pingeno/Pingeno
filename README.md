<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PINGENO BREW — Pingin? Kami antar.</title>
    
    <!-- Google Fonts: Inter -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet">
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- Leaflet CSS & JS untuk Peta Interaktif -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY=" crossorigin="" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo=" crossorigin=""></script>

    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>

    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                    },
                    colors: {
                        brew: {
                            50: '#f4f7fc',
                            100: '#e5ecf6',
                            200: '#c7d7ee',
                            300: '#99b8e1',
                            400: '#6491cd',
                            500: '#3f74b9',
                            600: '#2d5b9b',
                            700: '#24497e',
                            800: '#1e3d67', // Biru Utama Elegan
                            900: '#193355',
                            950: '#112239',
                        },
                        gold: {
                            400: '#facc15',
                            500: '#eab308',
                            600: '#ca8a04',
                            700: '#a16207',
                        },
                        stevia: {
                            500: '#16a34a',
                            600: '#15803d',
                        }
                    }
                }
            }
        }
    </script>

    <style>
        #map {
            height: 300px;
            width: 100%;
            z-index: 10;
        }
        .fade-in {
            animation: fadeIn 0.25s ease-in-out;
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(4px); }
            to { opacity: 1; transform: translateY(0); }
        }
    </style>
</head>
<body class="bg-stone-50 text-stone-800 antialiased pb-28 selection:bg-brew-800 selection:text-white">

    <!-- =====================================================================
         KONFIGURASI TOKO (Dsn Kwarasan RT 01 RW 07 Ds Timbang Kec Leksono Kab Wonosobo)
         ===================================================================== -->
    <script>
        const CONFIG = {
            storeName: "PINGENO BREW.CO",
            whatsappNumber: "62895412300109", // Format internasional tanpa tanda +
            
            // Titik Pusat Kedai di Dusun Kwarasan RT 01 RW 07, Ds Timbang, Kec Leksono, Kab Wonosobo
            storeLocation: {
                lat: -7.382312,
                lng: 109.885438,
                label: "Dusun Kwarasan RT 01 RW 07, Ds. Timbang, Kec. Leksono, Kab. Wonosobo"
            },

            // Pengaturan Jarak & Ongkir (Diperluas hingga 5 km ke arah Timur / Timur Laut Perkotaan Wonosobo)
            baseRadiusKm: 3.0,          // Radius dasar segala arah (3 km)
            extendedEastRadiusKm: 5.0,  // Perluasan radius ke arah Timur / Timur Laut hingga 5 km (Perkotaan Wonosobo)
            baseShippingFee: 5000,      // Ongkir dekat (0 - 2 km)
            midShippingFee: 7000,       // Ongkir menengah (2 - 4 km)
            farShippingFee: 10000,      // Ongkir maksimal jarak 4 - 5 km (pembelian < 3 cup)
            promoMinCups: 3,            // Minimal beli 3 cup untuk mendapat potongan ongkir
            promoDiscountAmount: 5000,  // Potongan ongkir Rp5.000 jika beli >= 3 cup (khusus jarak > 2 km)
            
            // Daftar Menu sesuai Flayer (Cup 12 oz, pemanis stevia)
            menu: [
                {
                    id: "dark-roasted-milk-tea",
                    name: "Dark Roasted Milk Tea",
                    desc: "Oolong Tea Base • Creamy, manis alami dengan Natural Stevia (Cup 12 oz).",
                    price: 15000,
                    tag: "Oolong Base",
                    color: "bg-brew-900"
                },
                {
                    id: "jasmine-green-milk-tea",
                    name: "Jasmine Green Milk Tea",
                    desc: "Jasmine Green Tea Base • Segar, creamy, dengan Natural Stevia (Cup 12 oz).",
                    price: 15000,
                    tag: "Jasmine Base",
                    color: "bg-sky-900"
                }
            ],

            // Informasi Pembayaran Penjual (Hanya QRIS & Dompet Digital)
            paymentInfo: {
                qrisMerchant: "NURHUDA MASDYON, FASHION",
                ewallet: {
                    provider: "GoPay",
                    phoneNumber: "0895412300109",
                    accountHolder: "Nurhuda Masdyon"
                }
            }
        };
    </script>

    <!-- Header / Navbar -->
    <header class="sticky top-0 z-30 bg-white/95 backdrop-blur-md border-b border-brew-100 shadow-xs">
        <div class="max-w-xl mx-auto px-4 py-3.5 flex items-center justify-between">
            <div class="flex items-center gap-2.5">
                <!-- Ikon Cangkir Khas Pingeno (Stevia Leaf & Steam) -->
                <div class="w-10 h-10 rounded-xl bg-brew-900 text-white flex items-center justify-center shadow-sm border border-gold-400/40 shrink-0">
                    <svg viewBox="0 0 48 48" class="w-7 h-7" fill="none" xmlns="http://www.w3.org/2000/svg">
                        <!-- Uap Teh -->
                        <path d="M21 15C19.5 12.5 22.5 10.5 21 8" stroke="#facc15" stroke-width="2.2" stroke-linecap="round"/>
                        <path d="M26 14C24.5 11.5 27.5 9.5 26 7" stroke="#facc15" stroke-width="2.2" stroke-linecap="round"/>
                        <!-- Badan Cangkir -->
                        <path d="M12 19H33V26C33 32.0751 28.0751 37 22 37H23C16.9249 37 12 32.0751 12 26V19Z" fill="currentColor"/>
                        <!-- Gagang Cangkir -->
                        <path d="M33 21H36C38.2091 21 40 22.7909 40 25C40 27.2091 38.2091 29 36 29H33" stroke="currentColor" stroke-width="2.8" stroke-linecap="round"/>
                        <!-- Daun Stevia di Tengah Cangkir -->
                        <path d="M22.5 32C22.5 32 18 30 18.5 25.5C21.5 25.5 22.5 28.5 22.5 32Z" fill="#1e3d67"/>
                        <path d="M22.5 32C22.5 32 27.5 29.5 26.5 24.5C23.5 24.5 22.5 28 22.5 32Z" fill="#facc15"/>
                    </svg>
                </div>
                <div>
                    <span id="header-store-name" class="font-extrabold tracking-tight text-base text-brew-900 block leading-none">PINGENO BREW</span>
                    <span class="text-[11px] font-medium text-gold-600 flex items-center gap-1 mt-0.5">
                        <i data-lucide="sparkles" class="w-3 h-3 text-gold-500"></i> Premium Milk Tea With Stevia
                    </span>
                </div>
            </div>
            <span class="text-xs font-bold px-3 py-1 rounded-full bg-brew-100 text-brew-800 border border-brew-200 shrink-0">
                Leksono • Kota Wonosobo
            </span>
        </div>
    </header>

    <!-- Main Content Container -->
    <main class="max-w-xl mx-auto px-4 pt-6 space-y-6">

        <!-- Hero Section -->
        <section class="bg-gradient-to-br from-brew-950 via-brew-800 to-brew-900 text-white rounded-3xl p-6 shadow-xl relative overflow-hidden border border-gold-400/20">
            <div class="relative z-10">
                <div class="inline-flex items-center gap-1.5 px-3 py-1 rounded-full bg-gold-400/20 backdrop-blur-sm text-xs font-semibold text-gold-300 mb-3 border border-gold-400/30">
                    <i data-lucide="map-pin" class="w-3.5 h-3.5 text-gold-400"></i> Jangkauan s/d <span id="hero-radius-text">5</span> KM (Timur & Perkotaan Wonosobo)
                </div>
                <h1 class="text-3xl font-extrabold tracking-tight mb-2 text-white">Pingin? Kami Antar.</h1>
                <div class="space-y-1 text-brew-100 text-sm leading-relaxed">
                    <p>Your Focus on your Journey, we deliver your milk tea.</p>
                    <p class="font-semibold text-white">A little Comfort your Busy day.</p>
                    <p class="text-xs text-gold-300/95 pt-1 font-medium">Premium Milktea Olong Da hong Pao &amp; Jasmine Green Tea</p>
                </div>
            </div>
            <div class="absolute -right-8 -bottom-10 w-40 h-40 rounded-full bg-gold-400/10 blur-xl pointer-events-none"></div>
        </section>

        <!-- 1. PILIH PESANAN -->
        <section class="bg-white rounded-3xl p-5 shadow-sm border border-stone-200/70">
            <div class="flex items-center justify-between mb-1">
                <h2 class="text-lg font-bold text-stone-900 flex items-center gap-2">
                    <span class="w-6 h-6 rounded-full bg-brew-900 text-white text-xs flex items-center justify-center font-bold">1</span>
                    Pilih pesanan
                </h2>
                <span class="text-xs font-medium px-2.5 py-1 rounded-lg bg-stone-100 text-stone-600">
                    Cup 12 oz, natural stevia
                </span>
            </div>
            <p class="text-xs text-stone-500 mb-3 ml-8">Beli minimal <strong class="text-brew-800">3 cup</strong> dapatkan <strong class="text-emerald-700">potongan ongkir Rp5.000</strong> untuk jarak jauh!</p>

            <!-- Daftar Menu Render via JS -->
            <div id="menu-list" class="space-y-3"></div>
        </section>

        <!-- 2. TITIK ANTAR -->
        <section class="bg-white rounded-3xl p-5 shadow-sm border border-stone-200/70">
            <div class="mb-2">
                <h2 class="text-lg font-bold text-stone-900 flex items-center gap-2">
                    <span class="w-6 h-6 rounded-full bg-brew-900 text-white text-xs flex items-center justify-center font-bold">2</span>
                    Titik Antar
                </h2>
                <p class="text-xs text-stone-500 mt-1 ml-8">
                    Ketuk peta atau gunakan GPS. Area Antar menjangkau <strong class="text-stone-700">3 km</strong> sekitar kedai & diperluas hingga <strong class="text-brew-800">5 km ke arah Timur / Timur Laut (Perkotaan Wonosobo)</strong>.
                </p>
            </div>

            <!-- Tombol Gunakan Lokasi Saat Ini & Pencarian -->
            <div class="mb-3 flex flex-col sm:flex-row gap-2">
                <div class="relative flex-1">
                    <input type="text" id="map-search-input" placeholder="Cari lokasi (misal: Alun-alun Wonosobo, Rita, Timbang)..." 
                        onkeydown="if(event.key==='Enter'){ event.preventDefault(); searchAddressAPI(); }"
                        class="w-full pl-3.5 pr-9 py-2.5 rounded-xl border border-stone-300 text-xs focus:outline-none focus:ring-2 focus:ring-brew-800">
                    <button type="button" onclick="searchAddressAPI()" class="absolute right-2 top-1/2 -translate-y-1/2 text-brew-800 hover:text-brew-950 p-1.5" title="Cari Alamat">
                        <i data-lucide="search" class="w-4 h-4"></i>
                    </button>
                </div>
                <button type="button" id="btn-gps" onclick="useCurrentLocation()" class="inline-flex items-center justify-center gap-1.5 text-xs font-bold px-3.5 py-2.5 rounded-xl bg-brew-800 hover:bg-brew-900 text-white shadow-xs transition active:scale-95 shrink-0">
                    <i data-lucide="locate-fixed" class="w-4 h-4 text-gold-400"></i>
                    <span id="btn-gps-text">Gunakan Lokasiku</span>
                </button>
            </div>

            <!-- Hasil Pencarian Alamat API -->
            <div id="map-search-results" class="hidden mb-3 rounded-xl border border-brew-200 bg-brew-50/60 divide-y divide-brew-100 overflow-hidden text-xs"></div>

            <!-- Container Peta -->
            <div class="rounded-2xl overflow-hidden border border-stone-200 shadow-inner relative">
                <div id="map"></div>
                <!-- Legenda Area Peta -->
                <div class="bg-white/95 backdrop-blur-xs px-3 py-2 border-t border-stone-200 flex flex-wrap items-center justify-between gap-2 text-[11px] text-stone-600">
                    <div class="flex items-center gap-1.5">
                        <span class="w-2.5 h-2.5 rounded-full bg-brew-600 inline-block"></span>
                        <span>Area Utama (0-3 km)</span>
                    </div>
                    <div class="flex items-center gap-1.5">
                        <span class="w-2.5 h-2.5 rounded-full bg-emerald-500 inline-block"></span>
                        <span>Perluasan Timur / Kota Wonosobo (s/d 5 km)</span>
                    </div>
                </div>
            </div>

            <!-- Status Titik Dipilih -->
            <div id="location-status" class="mt-3 p-3.5 rounded-2xl bg-stone-100 border border-stone-200/80 flex items-center gap-3 transition-colors">
                <div id="location-icon-wrap" class="w-9 h-9 rounded-xl bg-stone-200 text-stone-500 flex items-center justify-center shrink-0">
                    <i data-lucide="map-pin" class="w-4 h-4"></i>
                </div>
                <div class="text-xs flex-1">
                    <p id="location-title" class="font-semibold text-stone-700">Belum ada titik dipilih.</p>
                    <p id="location-sub" class="text-stone-500 mt-0.5">Ketuk area jangkauan pada peta atau tekan "Gunakan Lokasiku".</p>
                </div>
            </div>
        </section>

        <!-- 3. DATA PENGANTARAN -->
        <section class="bg-white rounded-3xl p-5 shadow-sm border border-stone-200/70">
            <h2 class="text-lg font-bold text-stone-900 flex items-center gap-2 mb-4">
                <span class="w-6 h-6 rounded-full bg-brew-900 text-white text-xs flex items-center justify-center font-bold">3</span>
                Data pengantaran
            </h2>

            <div class="space-y-4">
                <!-- Nama -->
                <div>
                    <label for="input-name" class="block text-xs font-semibold text-stone-700 mb-1.5">Nama Penerima <span class="text-red-500">*</span></label>
                    <input type="text" id="input-name" placeholder="Contoh: Budi Santoso" 
                        class="w-full px-3.5 py-2.5 rounded-xl border border-stone-300 text-sm focus:outline-none focus:ring-2 focus:ring-brew-800 focus:border-brew-800 transition">
                </div>

                <!-- Alamat Lengkap -->
                <div>
                    <label for="input-address" class="block text-xs font-semibold text-stone-700 mb-1.5">Alamat Lengkap & Patokan <span class="text-red-500">*</span></label>
                    <textarea id="input-address" rows="2" placeholder="Nama jalan, gang, nomor rumah/kantor, warna pagar..." 
                        class="w-full px-3.5 py-2.5 rounded-xl border border-stone-300 text-sm focus:outline-none focus:ring-2 focus:ring-brew-800 focus:border-brew-800 transition"></textarea>
                </div>

                <!-- Catatan (Opsional) -->
                <div>
                    <label for="input-notes" class="block text-xs font-semibold text-stone-700 mb-1.5">Catatan <span class="text-stone-400 font-normal">(opsional)</span></label>
                    <input type="text" id="input-notes" placeholder="Contoh: Less ice, titip di pos satpam..." 
                        class="w-full px-3.5 py-2.5 rounded-xl border border-stone-300 text-sm focus:outline-none focus:ring-2 focus:ring-brew-800 focus:border-brew-800 transition">
                </div>

                <!-- Pilihan Pembayaran (Hanya QRIS & Dompet Digital) -->
                <div>
                    <label class="block text-xs font-semibold text-stone-700 mb-2">Metode Pembayaran <span class="text-red-500">*</span></label>
                    <div class="grid grid-cols-2 gap-2.5" id="payment-methods">
                        <button type="button" onclick="selectPayment('qris')" data-pay="qris" class="pay-opt active flex items-center gap-2.5 p-3 rounded-xl border-2 border-brew-900 bg-brew-50/70 text-left transition">
                            <i data-lucide="qr-code" class="w-4 h-4 text-brew-900 shrink-0"></i>
                            <div>
                                <span class="block text-xs font-bold text-stone-900">QRIS</span>
                                <span class="block text-[10px] text-stone-500">Semua e-wallet & bank</span>
                            </div>
                        </button>
                        <button type="button" onclick="selectPayment('ewallet')" data-pay="ewallet" class="pay-opt flex items-center gap-2.5 p-3 rounded-xl border border-stone-200 hover:border-stone-300 bg-white text-left transition">
                            <i data-lucide="smartphone" class="w-4 h-4 text-stone-600 shrink-0"></i>
                            <div>
                                <span class="block text-xs font-bold text-stone-900">GoPay</span>
                                <span class="block text-[10px] text-stone-500">E-Wallet • 0895412300109</span>
                            </div>
                        </button>
                    </div>
                </div>

                <!-- Info Tarif Ongkir & Promo -->
                <div class="text-xs text-stone-600 bg-amber-50/90 border border-amber-200/80 px-3.5 py-3 rounded-xl space-y-1">
                    <div class="flex items-center gap-1.5 font-semibold text-amber-900">
                        <i data-lucide="truck" class="w-4 h-4 text-amber-600 shrink-0"></i>
                        <span>Ketentuan Ongkir & Promo:</span>
                    </div>
                    <ul class="list-disc list-inside text-[11px] text-amber-800 space-y-0.5 pl-1">
                        <li>Jarak 0 – 2 km: <strong>Rp5.000</strong></li>
                        <li>Jarak 2 – 4 km: <strong>Rp7.000</strong></li>
                        <li>Jarak 4 – 5 km (Perkotaan Wonosobo): <strong>Rp10.000</strong></li>
                        <li>Pesan <strong>≥ 3 cup</strong>: <strong>Potongan ongkir Rp5.000!</strong></li>
                    </ul>
                </div>
            </div>
        </section>

        <!-- 4. RINGKASAN -->
        <section class="bg-white rounded-3xl p-5 shadow-sm border border-stone-200/70">
            <h2 class="text-lg font-bold text-stone-900 flex items-center gap-2 mb-4">
                <span class="w-6 h-6 rounded-full bg-brew-900 text-white text-xs flex items-center justify-center font-bold">4</span>
                Ringkasan
            </h2>

            <!-- Rincian item terpilih -->
            <div id="summary-items" class="space-y-2 mb-3 pb-3 border-b border-dashed border-stone-200 text-xs text-stone-600 hidden"></div>

            <div class="space-y-2.5 text-sm">
                <div class="flex justify-between items-center text-stone-600">
                    <span>Subtotal (<span id="summary-cup-count">0</span> cup)</span>
                    <span id="summary-subtotal" class="font-semibold text-stone-900">Rp0</span>
                </div>
                <div class="flex justify-between items-center text-stone-600">
                    <span id="summary-shipping-label">Ongkir</span>
                    <span id="summary-shipping" class="font-semibold text-stone-900">-</span>
                </div>
                <div id="summary-promo-row" class="hidden justify-between items-center text-emerald-700 text-xs font-semibold bg-emerald-50 px-2.5 py-1.5 rounded-lg">
                    <span class="flex items-center gap-1"><i data-lucide="badge-percent" class="w-3.5 h-3.5"></i> Promo Ongkir (≥ 3 Cup)</span>
                    <span id="summary-promo-amount">-Rp5.000</span>
                </div>
                <div class="pt-2.5 border-t border-stone-200 flex justify-between items-center">
                    <span class="font-bold text-stone-900 text-base">Total</span>
                    <span id="summary-total" class="font-extrabold text-brew-900 text-lg">Rp0</span>
                </div>
            </div>
        </section>

        <!-- 5. BAYAR -->
        <section class="bg-white rounded-3xl p-5 shadow-sm border border-stone-200/70">
            <div class="flex items-center justify-between mb-4">
                <h2 class="text-lg font-bold text-stone-900 flex items-center gap-2">
                    <span class="w-6 h-6 rounded-full bg-brew-800 text-gold-400 text-xs flex items-center justify-center font-bold">5</span>
                    Bayar
                </h2>
                <div class="text-right">
                    <span class="block text-[11px] text-stone-500">Total bayar</span>
                    <span id="pay-total-display" class="font-extrabold text-lg text-brew-800">Rp0</span>
                </div>
            </div>

            <!-- Panel QRIS -->
            <div id="panel-qris" class="pay-panel fade-in space-y-3">
                <div class="p-4 rounded-2xl bg-brew-50/70 border border-brew-200/80 text-center space-y-1.5">
                    <p class="text-xs font-bold text-brew-900">Pembayaran via QRIS</p>
                    <p class="text-xs text-stone-600 leading-relaxed">
                        Kode QRIS akan dikirimkan oleh admin setelah pesanan diteruskan melalui pesan <strong>WhatsApp</strong>.
                    </p>
                </div>
            </div>

            <!-- Panel Dompet Digital -->
            <div id="panel-ewallet" class="pay-panel hidden fade-in">
                <div class="p-4 rounded-2xl bg-stone-50 border border-stone-200 space-y-3 text-sm">
                    <div class="flex justify-between items-center">
                        <span class="text-xs text-stone-500">Dompet digital</span>
                        <span id="info-ewallet-name" class="font-bold text-stone-900">GoPay</span>
                    </div>
                    <div class="flex justify-between items-center">
                        <span class="text-xs text-stone-500">Nomor</span>
                        <div class="flex items-center gap-2">
                            <span id="info-ewallet-num" class="font-mono font-bold text-stone-900">0895412300109</span>
                            <button type="button" onclick="copyText('info-ewallet-num')" class="p-1.5 rounded-lg bg-white border border-stone-200 hover:bg-stone-100 text-stone-600" title="Salin nomor">
                                <i data-lucide="copy" class="w-3.5 h-3.5"></i>
                            </button>
                        </div>
                    </div>
                    <div class="flex justify-between items-center">
                        <span class="text-xs text-stone-500">Atas nama</span>
                        <span id="info-ewallet-holder" class="font-semibold text-stone-800">Nurhuda Masdyon</span>
                    </div>
                </div>
            </div>

            <p class="mt-4 text-xs text-stone-600 bg-emerald-50/80 border border-emerald-200/80 px-3.5 py-2.5 rounded-xl text-center leading-relaxed">
                Lakukan pembayaran via <strong>QRIS</strong> atau <strong>GoPay</strong> sesuai total tagihan, lalu klik tombol di bawah untuk meneruskan pesanan & bukti pembayaran melalui pesan <strong>WhatsApp</strong>.
            </p>
        </section>

    </main>

    <!-- Sticky Bottom Action Bar -->
    <div class="fixed bottom-0 inset-x-0 z-30 bg-white/95 backdrop-blur-md border-t border-brew-100 px-4 py-3 shadow-[0_-4px_20px_rgba(30,61,103,0.08)]">
        <div class="max-w-xl mx-auto flex items-center justify-between gap-3">
            <div>
                <span class="block text-[11px] font-medium text-stone-500">Total Tagihan</span>
                <span id="sticky-total" class="text-lg font-extrabold text-brew-900">Rp0</span>
            </div>
            <button type="button" onclick="submitOrder()" class="flex-1 max-w-xs bg-brew-800 hover:bg-brew-900 active:scale-[0.98] text-white font-bold text-sm py-3.5 px-5 rounded-2xl shadow-md flex items-center justify-center gap-2 transition border border-gold-400/40">
                <i data-lucide="message-circle" class="w-4 h-4 text-gold-400"></i>
                <span id="btn-submit-text">Lanjutkan Pembayaran via WhatsApp</span>
            </button>
        </div>
    </div>

    <!-- Custom Toast / Notification Box -->
    <div id="toast-box" class="fixed top-16 left-1/2 -translate-x-1/2 z-[60] pointer-events-none opacity-0 transition-all duration-300 transform -translate-y-2 w-[90%] max-w-md">
        <div id="toast-content" class="bg-stone-900 text-white text-xs font-medium px-4 py-3 rounded-2xl shadow-xl flex items-center gap-2.5 border border-stone-700">
            <span id="toast-icon-wrap" class="shrink-0 flex items-center justify-center">
                <i data-lucide="info" class="w-4 h-4 text-amber-400"></i>
            </span>
            <span id="toast-message">Pesan notifikasi</span>
        </div>
    </div>

    <!-- =====================================================================
         LOGIKA APLIKASI PEMBELI
         ===================================================================== -->
    <script>
        const state = {
            cart: {},
            selectedLocation: null,
            rawShippingFee: null,
            discountApplied: 0,
            finalShippingFee: null,
            paymentMethod: 'qris'
        };

        let map, userMarker, baseRadiusCircle, eastExtensionPolygon, routePolyline;
        let lastSearchResults = [];

        function formatRupiah(num) {
            return 'Rp' + Number(num || 0).toLocaleString('id-ID');
        }

        function refreshIcons() {
            if (typeof lucide !== 'undefined' && typeof lucide.createIcons === 'function') {
                lucide.createIcons();
            }
        }

        let toastTimeout;
        function showToast(message, type = 'info') {
            const box = document.getElementById('toast-box');
            const msgEl = document.getElementById('toast-message');
            const iconWrap = document.getElementById('toast-icon-wrap');
            if (!box || !msgEl || !iconWrap) return;

            msgEl.textContent = message;

            let iconName = 'info';
            let iconColor = 'text-amber-400';
            if (type === 'error') {
                iconName = 'alert-triangle';
                iconColor = 'text-red-400';
            } else if (type === 'success') {
                iconName = 'check-circle-2';
                iconColor = 'text-emerald-400';
            }

            iconWrap.innerHTML = `<i data-lucide="${iconName}" class="w-4 h-4 ${iconColor}"></i>`;
            refreshIcons();

            box.classList.remove('opacity-0', '-translate-y-2');
            box.classList.add('opacity-100', 'translate-y-0');

            clearTimeout(toastTimeout);
            toastTimeout = setTimeout(() => {
                box.classList.remove('opacity-100', 'translate-y-0');
                box.classList.add('opacity-0', '-translate-y-2');
            }, 3200);
        }

        // Ilustrasi SVG Cup Pingeno Brew.co
        function getCupIllustration(accentHex) {
            return `
                <svg viewBox="0 0 80 108" class="w-12 h-16 shrink-0 drop-shadow-xs" fill="none" xmlns="http://www.w3.org/2000/svg">
                    <path d="M12 16C12 13.5 16 12 40 12C64 12 68 13.5 68 16L70 20C70 22 66 23 40 23C14 23 10 22 10 20L12 16Z" fill="#1e293b"/>
                    <path d="M16 12C16 9 24 8 40 8C56 8 64 9 64 12H16Z" fill="#334155"/>
                    <path d="M16 23L23 96C23.5 100 28 102 40 102C52 102 56.5 100 57 96L64 23H16Z" fill="#ffffff" stroke="#cbd5e1" stroke-width="1.5"/>
                    <line x1="18" y1="29" x2="62" y2="29" stroke="${accentHex}" stroke-width="2.5" stroke-linecap="round"/>
                    <text x="40" y="43" font-family="Inter, sans-serif" font-weight="800" font-size="8" fill="#1e3d67" text-anchor="middle">pingeno</text>
                    <text x="40" y="50" font-family="Inter, sans-serif" font-weight="600" font-size="4.2" fill="#64748b" text-anchor="middle">Brew.co • 12 oz</text>
                    <path d="M20 74L32 60L44 72L54 63L60 72L57 96C56.5 100 52 102 40 102C28 102 23.5 100 23 96L20 74Z" fill="#2d5b9b" opacity="0.22"/>
                    <path d="M21 82L36 67L48 79L59 71L57 96C56.5 100 52 102 40 102C28 102 23.5 100 23 96L21 82Z" fill="#1e3d67" opacity="0.88"/>
                </svg>
            `;
        }

        // Render Daftar Menu dari CONFIG
        function renderMenu() {
            const container = document.getElementById('menu-list');
            if (!container) return;

            container.innerHTML = CONFIG.menu.map(item => {
                const qty = state.cart[item.id] || 0;
                const accentHex = item.id.includes('dark') ? '#eab308' : '#16a34a';
                return `
                    <div class="p-3.5 rounded-2xl border ${qty > 0 ? 'border-brew-800 bg-brew-50/50 shadow-xs' : 'border-stone-200/80 bg-white'} flex items-center justify-between gap-3 transition">
                        <div class="flex items-center gap-3">
                            <div class="w-14 h-16 rounded-xl bg-stone-50 border border-stone-200/70 flex items-center justify-center shrink-0 p-1">
                                ${getCupIllustration(accentHex)}
                            </div>
                            <div>
                                <div class="flex items-center gap-1.5 flex-wrap">
                                    <h3 class="font-bold text-sm text-stone-900">${item.name}</h3>
                                    <span class="text-[10px] font-semibold px-2 py-0.5 rounded-full bg-brew-100 text-brew-800">${item.tag}</span>
                                </div>
                                <p class="text-xs text-stone-500 mt-0.5 leading-snug">${item.desc}</p>
                                <p class="text-sm font-extrabold text-brew-900 mt-1.5">${formatRupiah(item.price)}</p>
                            </div>
                        </div>

                        <div class="flex items-center gap-1.5 shrink-0">
                            ${qty > 0 ? `
                                <button type="button" onclick="updateQty('${item.id}', -1)" class="w-8 h-8 rounded-xl bg-stone-100 hover:bg-stone-200 text-stone-800 font-bold flex items-center justify-center transition active:scale-90">
                                    <i data-lucide="minus" class="w-3.5 h-3.5"></i>
                                </button>
                                <span class="w-5 text-center text-sm font-bold text-stone-900">${qty}</span>
                            ` : ''}
                            <button type="button" onclick="updateQty('${item.id}', 1)" class="w-8 h-8 rounded-xl bg-brew-900 hover:bg-brew-950 text-gold-400 font-bold flex items-center justify-center shadow-xs transition active:scale-90">
                                <i data-lucide="plus" class="w-3.5 h-3.5"></i>
                            </button>
                        </div>
                    </div>
                `;
            }).join('');
            refreshIcons();
        }

        // Update Jumlah Pesanan
        function updateQty(itemId, delta) {
            const current = state.cart[itemId] || 0;
            const next = Math.max(0, current + delta);
            if (next === 0) {
                delete state.cart[itemId];
            } else {
                state.cart[itemId] = next;
            }
            renderMenu();
            calculateTotals();
            if (state.selectedLocation) {
                updateLocationStatusUI();
            }
        }

        // Menghitung titik tujuan dari koordinat asal, jarak (km), dan sudut arah (derajat)
        function destinationPoint(lat, lng, distanceKm, bearingDeg) {
            const R = 6371;
            const brng = bearingDeg * Math.PI / 180;
            const d = distanceKm / R;
            const lat1 = lat * Math.PI / 180;
            const lon1 = lng * Math.PI / 180;

            const lat2 = Math.asin(
                Math.sin(lat1) * Math.cos(d) +
                Math.cos(lat1) * Math.sin(d) * Math.cos(brng)
            );
            const lon2 = lon1 + Math.atan2(
                Math.sin(brng) * Math.sin(d) * Math.cos(lat1),
                Math.cos(d) - Math.sin(lat1) * Math.sin(lat2)
            );
            return [lat2 * 180 / Math.PI, lon2 * 180 / Math.PI];
        }

        // Menghitung sudut arah (bearing 0-360 derajat) dari kedai ke titik pembeli
        function getBearingDeg(lat1, lon1, lat2, lon2) {
            const p1 = lat1 * Math.PI / 180;
            const p2 = lat2 * Math.PI / 180;
            const dLon = (lon2 - lon1) * Math.PI / 180;
            const y = Math.sin(dLon) * Math.cos(p2);
            const x = Math.cos(p1) * Math.sin(p2) - Math.sin(p1) * Math.cos(p2) * Math.cos(dLon);
            return ((Math.atan2(y, x) * 180 / Math.PI) + 360) % 360;
        }

        // Rumus Haversine (Jarak garis lurus dalam km)
        function getDistanceKm(lat1, lon1, lat2, lon2) {
            const R = 6371;
            const dLat = (lat2 - lat1) * Math.PI / 180;
            const dLon = (lon2 - lon1) * Math.PI / 180;
            const a = Math.sin(dLat / 2) * Math.sin(dLat / 2) +
                      Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) *
                      Math.sin(dLon / 2) * Math.sin(dLon / 2);
            const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
            return R * c;
        }

        // Inisialisasi Peta Leaflet
        function initMap() {
            if (typeof L === 'undefined') {
                showToast("Peta gagal dimuat. Periksa koneksi internet Anda.", "error");
                return;
            }
            try {
                const { lat, lng, label } = CONFIG.storeLocation;
                map = L.map('map', {
                    center: [lat + 0.008, lng + 0.015],
                    zoom: 13,
                    zoomControl: true
                });

                L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                    attribution: '&copy; OpenStreetMap'
                }).addTo(map);

                const storeIcon = L.divIcon({
                    html: `<div class="w-8 h-8 rounded-full bg-brew-900 text-gold-400 border-2 border-white shadow-md flex items-center justify-center"><svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="M17 8h1a4 4 0 1 1 0 8h-1"/><path d="M3 8h14v9a4 4 0 0 1-4 4H7a4 4 0 0 1-4-4Z"/><line x1="6" x2="6" y1="2" y2="4"/><line x1="10" x2="10" y1="2" y2="4"/><line x1="14" x2="14" y1="2" y2="4"/></svg></div>`,
                    className: '',
                    iconSize: [32, 32],
                    iconAnchor: [16, 16]
                });

                L.marker([lat, lng], { icon: storeIcon })
                    .addTo(map)
                    .bindPopup(`<b>${CONFIG.storeName}</b><br>${label}`);

                // 1. Lingkaran Radius Utama 3 km
                baseRadiusCircle = L.circle([lat, lng], {
                    color: '#1e3d67',
                    fillColor: '#3f74b9',
                    fillOpacity: 0.14,
                    weight: 2,
                    dashArray: '5, 5',
                    radius: CONFIG.baseRadiusKm * 1000
                }).addTo(map);

                // 2. Sektor Perluasan 5 km ke arah Timur Laut - Timur (Perkotaan Wonosobo: sudut 10° sampai 115°)
                const sectorPoints = [[lat, lng]];
                for (let deg = 10; deg <= 115; deg += 5) {
                    sectorPoints.push(destinationPoint(lat, lng, CONFIG.extendedEastRadiusKm, deg));
                }
                sectorPoints.push([lat, lng]);

                eastExtensionPolygon = L.polygon(sectorPoints, {
                    color: '#16a34a',
                    fillColor: '#22c55e',
                    fillOpacity: 0.10,
                    weight: 2,
                    dashArray: '6, 6'
                }).addTo(map);

                map.on('click', function(e) {
                    setDeliveryPin(e.latlng.lat, e.latlng.lng);
                });
            } catch (err) {
                console.error("Gagal menginisialisasi peta:", err);
                showToast("Terjadi kendala saat memuat peta.", "error");
            }
        }

        // Pasang Pin Lokasi & Hitung Rute Jalan via OSRM API
        async function setDeliveryPin(lat, lng, addressLabel = '') {
            if (!map) return;

            const storeLat = CONFIG.storeLocation.lat;
            const storeLng = CONFIG.storeLocation.lng;
            const straightKm = getDistanceKm(storeLat, storeLng, lat, lng);
            const bearing = getBearingDeg(storeLat, storeLng, lat, lng);
            const isEastSector = bearing >= 10 && bearing <= 115;
            const maxAllowedKm = isEastSector ? CONFIG.extendedEastRadiusKm : CONFIG.baseRadiusKm;

            // Tampilkan status loading rute
            const titleEl = document.getElementById('location-title');
            const subEl = document.getElementById('location-sub');
            if (titleEl) titleEl.textContent = "Menghitung jarak rute jalan...";
            if (subEl) subEl.textContent = "Menghubungkan ke layanan peta...";

            let routeKm = straightKm;
            let routeGeometry = null;

            try {
                const url = `https://router.project-osrm.org/route/v1/driving/${storeLng},${storeLat};${lng},${lat}?overview=full&geometries=geojson`;
                const res = await fetch(url);
                const data = await res.json();
                if (data.code === 'Ok' && data.routes && data.routes.length > 0) {
                    routeKm = data.routes[0].distance / 1000;
                    routeGeometry = data.routes[0].geometry;
                }
            } catch (err) {
                // Fallback ke jarak garis lurus jika OSRM sedang sibuk
                routeKm = straightKm;
            }

            const withinRange = straightKm <= maxAllowedKm && routeKm <= (maxAllowedKm + 1.0);

            if (userMarker) map.removeLayer(userMarker);
            if (routePolyline) map.removeLayer(routePolyline);

            const pinColor = withinRange ? 'bg-emerald-600' : 'bg-red-600';
            const pinIcon = L.divIcon({
                html: `<div class="w-8 h-8 rounded-full ${pinColor} text-white border-2 border-white shadow-lg flex items-center justify-center"><svg xmlns="http://www.w3.org/2000/svg" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="M20 10c0 6-8 12-8 12s-8-6-8-12a8 8 0 0 1 16 0Z"/><circle cx="12" cy="10" r="3"/></svg></div>`,
                className: '',
                iconSize: [32, 32],
                iconAnchor: [16, 30]
            });

            userMarker = L.marker([lat, lng], { icon: pinIcon }).addTo(map);
            map.panTo([lat, lng]);

            if (routeGeometry) {
                routePolyline = L.geoJSON(routeGeometry, {
                    style: { color: withinRange ? '#1e3d67' : '#dc2626', weight: 4, opacity: 0.8 }
                }).addTo(map);
            } else {
                routePolyline = L.polyline([[storeLat, storeLng], [lat, lng]], {
                    color: withinRange ? '#1e3d67' : '#dc2626',
                    weight: 3,
                    dashArray: '5, 5'
                }).addTo(map);
            }

            state.selectedLocation = {
                lat,
                lng,
                distanceKm: routeKm,
                straightKm,
                maxAllowedKm,
                withinRange,
                label: addressLabel
            };

            if (addressLabel) {
                const addrInput = document.getElementById('input-address');
                if (addrInput && !addrInput.value.trim()) {
                    addrInput.value = addressLabel;
                }
            }

            calculateTotals();
            updateLocationStatusUI();

            if (!withinRange) {
                showToast(`Di luar jangkauan! Jarak ${routeKm.toFixed(1)} km (Maks ${maxAllowedKm} km untuk arah ini).`, "error");
            }
        }

        // Perbarui Tampilan Status Kotak Lokasi
        function updateLocationStatusUI() {
            const loc = state.selectedLocation;
            if (!loc) return;

            const statusBox = document.getElementById('location-status');
            const iconWrap = document.getElementById('location-icon-wrap');
            const titleEl = document.getElementById('location-title');
            const subEl = document.getElementById('location-sub');
            const totalCups = Object.values(state.cart).reduce((a, b) => a + b, 0);

            if (loc.withinRange) {
                statusBox.className = "mt-3 p-3.5 rounded-2xl bg-emerald-50 border border-emerald-200 flex items-center gap-3 transition-colors";
                iconWrap.className = "w-9 h-9 rounded-xl bg-emerald-600 text-white flex items-center justify-center shrink-0";
                titleEl.className = "font-bold text-emerald-900";
                titleEl.textContent = `Titik terpilih • Jarak rute ${loc.distanceKm.toFixed(2)} km`;

                let subText = `Masuk area antar! Ongkir ${formatRupiah(state.finalShippingFee)}.`;
                if (loc.distanceKm > 2.0 && totalCups < CONFIG.promoMinCups) {
                    const need = CONFIG.promoMinCups - totalCups;
                    subText += ` Tambah ${need} cup lagi agar dapat potongan ongkir Rp5.000!`;
                } else if (state.discountApplied > 0) {
                    subText += ` (Hemat ${formatRupiah(state.discountApplied)} promo ≥ 3 cup!)`;
                }
                subEl.className = "text-emerald-700 mt-0.5";
                subEl.textContent = subText;
            } else {
                statusBox.className = "mt-3 p-3.5 rounded-2xl bg-red-50 border border-red-200 flex items-center gap-3 transition-colors";
                iconWrap.className = "w-9 h-9 rounded-xl bg-red-600 text-white flex items-center justify-center shrink-0";
                titleEl.className = "font-bold text-red-900";
                titleEl.textContent = `Di luar jangkauan (${loc.distanceKm.toFixed(2)} km)`;
                subEl.className = "text-red-700 mt-0.5";
                subEl.textContent = `Maksimal ${loc.maxAllowedKm} km untuk arah ini. Area 5 km khusus ke arah Timur / Perkotaan Wonosobo.`;
            }
        }

        // Ambil Lokasi GPS Browser
        function useCurrentLocation() {
            if (!navigator.geolocation) {
                showToast("Browser tidak mendukung fitur GPS.", "error");
                return;
            }
            const btnText = document.getElementById('btn-gps-text');
            if (btnText) btnText.textContent = "Mencari GPS...";

            navigator.geolocation.getCurrentPosition(
                (pos) => {
                    if (btnText) btnText.textContent = "Gunakan Lokasiku";
                    setDeliveryPin(pos.coords.latitude, pos.coords.longitude);
                    showToast("Lokasi GPS berhasil dipasang!", "success");
                },
                () => {
                    if (btnText) btnText.textContent = "Gunakan Lokasiku";
                    showToast("Gagal mengakses GPS. Silakan ketuk peta langsung.", "error");
                },
                { enableHighAccuracy: true, timeout: 10000 }
            );
        }

        // Cari Alamat via Nominatim OpenStreetMap API
        async function searchAddressAPI() {
            const input = document.getElementById('map-search-input');
            const resultsBox = document.getElementById('map-search-results');
            if (!input || !resultsBox) return;

            const queryText = input.value.trim();
            if (!queryText) {
                showToast("Ketik nama desa atau tempat di Wonosobo terlebih dahulu.", "info");
                return;
            }

            resultsBox.classList.remove('hidden');
            resultsBox.innerHTML = `<div class="p-3 text-stone-500 text-center">Mencari lokasi...</div>`;

            try {
                const q = encodeURIComponent(`${queryText}, Wonosobo, Jawa Tengah`);
                const res = await fetch(`https://nominatim.openstreetmap.org/search?format=json&q=${q}&limit=5`);
                const data = await res.json();

                if (!data || data.length === 0) {
                    resultsBox.innerHTML = `<div class="p-3 text-stone-500 text-center">Lokasi tidak ditemukan. Coba ketuk langsung pada peta.</div>`;
                    return;
                }

                lastSearchResults = data.map(place => ({
                    lat: parseFloat(place.lat),
                    lng: parseFloat(place.lon),
                    label: place.display_name.split(',').slice(0, 3).join(', ')
                }));

                resultsBox.innerHTML = lastSearchResults.map((item, idx) => {
                    const safeLabel = item.label.replace(/</g, '&lt;').replace(/>/g, '&gt;');
                    return `
                        <button type="button" onclick="selectSearchResult(${idx})" 
                            class="w-full text-left p-2.5 hover:bg-brew-100/70 transition flex items-start gap-2">
                            <i data-lucide="map-pin" class="w-3.5 h-3.5 text-brew-800 shrink-0 mt-0.5"></i>
                            <span class="text-stone-700 font-medium leading-snug">${safeLabel}</span>
                        </button>
                    `;
                }).join('');
                refreshIcons();
            } catch (err) {
                resultsBox.innerHTML = `<div class="p-3 text-red-600 text-center">Gagal memuat pencarian alamat. Silakan ketuk peta langsung.</div>`;
            }
        }

        function selectSearchResult(index) {
            const selected = lastSearchResults[index];
            if (!selected || isNaN(selected.lat) || isNaN(selected.lng)) return;
            document.getElementById('map-search-results').classList.add('hidden');
            if (map) {
                map.setView([selected.lat, selected.lng], 15);
            }
            setDeliveryPin(selected.lat, selected.lng, selected.label);
        }

        // Pilih Metode Pembayaran (QRIS / GoPay)
        function selectPayment(method) {
            state.paymentMethod = method;

            document.querySelectorAll('.pay-opt').forEach(btn => {
                const isSelected = btn.getAttribute('data-pay') === method;
                const iconEl = btn.querySelector('i, svg');
                if (isSelected) {
                    btn.className = "pay-opt active flex items-center gap-2.5 p-3 rounded-xl border-2 border-brew-900 bg-brew-50/70 text-left transition";
                    if (iconEl) {
                        iconEl.classList.remove('text-stone-600');
                        iconEl.classList.add('text-brew-900');
                    }
                } else {
                    btn.className = "pay-opt flex items-center gap-2.5 p-3 rounded-xl border border-stone-200 hover:border-stone-300 bg-white text-left transition";
                    if (iconEl) {
                        iconEl.classList.remove('text-brew-900');
                        iconEl.classList.add('text-stone-600');
                    }
                }
            });

            document.querySelectorAll('.pay-panel').forEach(panel => panel.classList.add('hidden'));
            const activePanel = document.getElementById(`panel-${method}`);
            if (activePanel) {
                activePanel.classList.remove('hidden');
            }
        }

        // Kalkulasi Subtotal, Ongkir Dinamis (0-5 km), dan Promo >= 3 Cup (-Rp5.000)
        function calculateTotals() {
            let subtotal = 0;
            let totalCups = 0;
            const summaryItemsEl = document.getElementById('summary-items');
            const itemLines = [];

            CONFIG.menu.forEach(item => {
                const qty = state.cart[item.id] || 0;
                if (qty > 0) {
                    totalCups += qty;
                    const lineTotal = item.price * qty;
                    subtotal += lineTotal;
                    itemLines.push(`
                        <div class="flex justify-between items-center">
                            <span>${qty}x ${item.name}</span>
                            <span class="font-medium text-stone-800">${formatRupiah(lineTotal)}</span>
                        </div>
                    `);
                }
            });

            if (summaryItemsEl) {
                if (itemLines.length > 0) {
                    summaryItemsEl.innerHTML = itemLines.join('');
                    summaryItemsEl.classList.remove('hidden');
                } else {
                    summaryItemsEl.classList.add('hidden');
                }
            }

            const cupCountEl = document.getElementById('summary-cup-count');
            if (cupCountEl) cupCountEl.textContent = totalCups;

            const subtotalEl = document.getElementById('summary-subtotal');
            if (subtotalEl) subtotalEl.textContent = formatRupiah(subtotal);

            const shippingLabelEl = document.getElementById('summary-shipping-label');
            const shippingValEl = document.getElementById('summary-shipping');
            const promoRowEl = document.getElementById('summary-promo-row');
            const promoAmountEl = document.getElementById('summary-promo-amount');

            state.rawShippingFee = null;
            state.discountApplied = 0;
            state.finalShippingFee = null;

            if (state.selectedLocation && state.selectedLocation.withinRange) {
                const dist = state.selectedLocation.distanceKm;
                let rawFee = CONFIG.baseShippingFee;
                if (dist > 4.0) {
                    rawFee = CONFIG.farShippingFee;
                } else if (dist > 2.0) {
                    rawFee = CONFIG.midShippingFee;
                }

                let discount = 0;
                if (totalCups >= CONFIG.promoMinCups && dist > 2.0) {
                    discount = Math.min(rawFee, CONFIG.promoDiscountAmount);
                }

                state.rawShippingFee = rawFee;
                state.discountApplied = discount;
                state.finalShippingFee = Math.max(0, rawFee - discount);

                if (shippingLabelEl) shippingLabelEl.textContent = `Ongkir (${dist.toFixed(1)} km)`;
                if (shippingValEl) shippingValEl.textContent = formatRupiah(rawFee);

                if (promoRowEl) {
                    if (discount > 0) {
                        promoRowEl.classList.remove('hidden');
                        promoRowEl.classList.add('flex');
                        if (promoAmountEl) promoAmountEl.textContent = `-${formatRupiah(discount)}`;
                    } else {
                        promoRowEl.classList.add('hidden');
                        promoRowEl.classList.remove('flex');
                    }
                }
            } else {
                if (shippingLabelEl) shippingLabelEl.textContent = 'Ongkir';
                if (shippingValEl) shippingValEl.textContent = '-';
                if (promoRowEl) {
                    promoRowEl.classList.add('hidden');
                    promoRowEl.classList.remove('flex');
                }
            }

            const total = subtotal + (subtotal > 0 && state.finalShippingFee !== null ? state.finalShippingFee : 0);
            const formattedTotal = formatRupiah(total);

            const sumTotalEl = document.getElementById('summary-total');
            const payTotalEl = document.getElementById('pay-total-display');
            const stickyTotalEl = document.getElementById('sticky-total');

            if (sumTotalEl) sumTotalEl.textContent = formattedTotal;
            if (payTotalEl) payTotalEl.textContent = formattedTotal;
            if (stickyTotalEl) stickyTotalEl.textContent = formattedTotal;
        }

        // Salin Nomor Dompet Digital ke Clipboard
        function copyText(elementId) {
            const text = document.getElementById(elementId)?.textContent?.trim();
            if (!text) return;
            try {
                const tempInput = document.createElement('input');
                tempInput.value = text;
                document.body.appendChild(tempInput);
                tempInput.select();
                document.execCommand('copy');
                document.body.removeChild(tempInput);
                showToast(`Nomor ${text} berhasil disalin!`, "success");
            } catch (err) {
                showToast("Gagal menyalin nomor.", "error");
            }
        }

        // Kirim Pesanan ke WhatsApp
        function submitOrder() {
            const orderedItems = CONFIG.menu.filter(m => (state.cart[m.id] || 0) > 0);
            if (orderedItems.length === 0) {
                showToast("Pilih minimal 1 minuman terlebih dahulu.", "error");
                return;
            }

            if (!state.selectedLocation) {
                showToast("Ketuk peta atau gunakan GPS untuk menentukan titik antar.", "error");
                return;
            }
            if (!state.selectedLocation.withinRange) {
                showToast(`Lokasi di luar jangkauan maksimal ${state.selectedLocation.maxAllowedKm} km.`, "error");
                return;
            }

            const name = document.getElementById('input-name')?.value.trim() || '';
            const address = document.getElementById('input-address')?.value.trim() || '';
            const notes = document.getElementById('input-notes')?.value.trim() || '';

            if (!name) {
                showToast("Mohon isi nama penerima.", "error");
                document.getElementById('input-name')?.focus();
                return;
            }
            if (!address) {
                showToast("Mohon isi alamat lengkap & patokan.", "error");
                document.getElementById('input-address')?.focus();
                return;
            }

            let subtotal = 0;
            const itemsText = orderedItems.map(item => {
                const qty = state.cart[item.id];
                const sub = item.price * qty;
                subtotal += sub;
                return `• ${qty}x ${item.name} (${formatRupiah(sub)})`;
            }).join('\n');

            const finalShipping = state.finalShippingFee || 0;
            const total = subtotal + finalShipping;
            const paymentLabel = state.paymentMethod === 'qris'
                ? `QRIS (${CONFIG.paymentInfo.qrisMerchant})`
                : `GoPay (${CONFIG.paymentInfo.ewallet.phoneNumber} a.n ${CONFIG.paymentInfo.ewallet.accountHolder})`;

            const mapsLink = `https://www.google.com/maps?q=${state.selectedLocation.lat},${state.selectedLocation.lng}`;
            const promoLine = state.discountApplied > 0 ? `\n• Promo Ongkir (≥ 3 Cup): -${formatRupiah(state.discountApplied)}` : '';

            const waMessage =
`Halo *${CONFIG.storeName}*! Saya mau pesan antar:

*PESANAN (Cup 12 oz, Natural Stevia):*
${itemsText}

*DATA PENGANTARAN:*
• Nama: ${name}
• Alamat: ${address}
${notes ? `• Catatan: ${notes}\n` : ''}• Jarak Rute: ${state.selectedLocation.distanceKm.toFixed(2)} km
• Pin Peta: ${mapsLink}

*RINCIAN BIAYA:*
• Subtotal: ${formatRupiah(subtotal)}
• Ongkir: ${formatRupiah(state.rawShippingFee)}${promoLine}
• *Total Bayar: ${formatRupiah(total)}*
• Pembayaran: ${paymentLabel}

Mohon diproses ya kak, terima kasih!`;

            const waUrl = `https://wa.me/${CONFIG.whatsappNumber}?text=${encodeURIComponent(waMessage)}`;
            showToast("Membuka WhatsApp untuk konfirmasi pesanan...", "success");
            window.open(waUrl, '_blank');
        }

        // Inisialisasi saat halaman dimuat
        window.onload = function() {
            renderMenu();
            initMap();
            selectPayment('qris');
            calculateTotals();
            refreshIcons();
        };
    </script>
</body>
</html>
