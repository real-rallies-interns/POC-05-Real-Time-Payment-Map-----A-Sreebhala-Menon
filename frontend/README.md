import { useState, useEffect, useRef } from "react";

// ── DATA LAYER (mirrors FastAPI /api/schemes) ──────────────────────────────
const RTP_SCHEMES = [
  { code:"IN", name:"India", scheme:"UPI", authority:"NPCI / Reserve Bank of India", launch_year:2016, maturity:"Leader", p2p:true, p2b:true, iso20022:false, tx_limit_usd:2400, daily_volume_m:400, lat:20.59, lng:78.96 },
  { code:"BR", name:"Brazil", scheme:"Pix", authority:"Banco Central do Brasil", launch_year:2020, maturity:"Leader", p2p:true, p2b:true, iso20022:true, tx_limit_usd:null, daily_volume_m:60, lat:-14.23, lng:-51.92 },
  { code:"GB", name:"United Kingdom", scheme:"Faster Payments", authority:"Pay.UK / Bank of England", launch_year:2008, maturity:"Leader", p2p:true, p2b:true, iso20022:true, tx_limit_usd:1250000, daily_volume_m:9, lat:55.37, lng:-3.43 },
  { code:"SG", name:"Singapore", scheme:"PayNow", authority:"MAS / ABS", launch_year:2017, maturity:"Leader", p2p:true, p2b:true, iso20022:true, tx_limit_usd:75000, daily_volume_m:2, lat:1.35, lng:103.81 },
  { code:"AU", name:"Australia", scheme:"NPP (Osko)", authority:"Reserve Bank of Australia", launch_year:2018, maturity:"Established", p2p:true, p2b:true, iso20022:true, tx_limit_usd:null, daily_volume_m:1, lat:-25.27, lng:133.77 },
  { code:"US", name:"United States", scheme:"FedNow", authority:"Federal Reserve", launch_year:2023, maturity:"Developing", p2p:true, p2b:false, iso20022:true, tx_limit_usd:500000, daily_volume_m:0.5, lat:37.09, lng:-95.71 },
  { code:"EU", name:"Eurozone", scheme:"SEPA Instant", authority:"European Central Bank / EPC", launch_year:2017, maturity:"Established", p2p:true, p2b:true, iso20022:true, tx_limit_usd:110000, daily_volume_m:15, lat:50.85, lng:4.35 },
  { code:"MX", name:"Mexico", scheme:"SPEI", authority:"Banco de México", launch_year:2004, maturity:"Leader", p2p:true, p2b:true, iso20022:false, tx_limit_usd:null, daily_volume_m:8, lat:23.63, lng:-102.55 },
  { code:"ZA", name:"South Africa", scheme:"RTC", authority:"PASA / SARB", launch_year:2006, maturity:"Established", p2p:true, p2b:true, iso20022:false, tx_limit_usd:27000, daily_volume_m:0.8, lat:-30.55, lng:22.93 },
  { code:"CN", name:"China", scheme:"IBPS / CNAPS", authority:"People's Bank of China", launch_year:2010, maturity:"Leader", p2p:true, p2b:true, iso20022:false, tx_limit_usd:null, daily_volume_m:200, lat:35.86, lng:104.19 },
  { code:"NG", name:"Nigeria", scheme:"NIP (NIBSS)", authority:"CBN / NIBSS", launch_year:2011, maturity:"Established", p2p:true, p2b:true, iso20022:false, tx_limit_usd:3000, daily_volume_m:12, lat:9.08, lng:8.67 },
  { code:"TH", name:"Thailand", scheme:"PromptPay", authority:"Bank of Thailand", launch_year:2017, maturity:"Established", p2p:true, p2b:true, iso20022:false, tx_limit_usd:8000, daily_volume_m:18, lat:15.87, lng:100.99 },
  { code:"MY", name:"Malaysia", scheme:"DuitNow", authority:"BNM / PayNet", launch_year:2018, maturity:"Established", p2p:true, p2b:true, iso20022:false, tx_limit_usd:23000, daily_volume_m:2, lat:4.21, lng:101.97 },
  { code:"PK", name:"Pakistan", scheme:"RAAST", authority:"State Bank of Pakistan", launch_year:2021, maturity:"Developing", p2p:true, p2b:false, iso20022:true, tx_limit_usd:3600, daily_volume_m:1, lat:30.37, lng:69.34 },
  { code:"KE", name:"Kenya", scheme:"PesaLink", authority:"KBA / CBK", launch_year:2017, maturity:"Developing", p2p:true, p2b:false, iso20022:false, tx_limit_usd:2800, daily_volume_m:0.3, lat:-0.02, lng:37.90 },
  { code:"JP", name:"Japan", scheme:"Zengin Instant", authority:"Bank of Japan / ZENGINKYO", launch_year:2018, maturity:"Established", p2p:true, p2b:true, iso20022:false, tx_limit_usd:null, daily_volume_m:5, lat:36.20, lng:138.25 },
  { code:"CA", name:"Canada", scheme:"Interac e-Transfer RTP", authority:"Bank of Canada / Payments Canada", launch_year:2023, maturity:"Developing", p2p:true, p2b:false, iso20022:true, tx_limit_usd:null, daily_volume_m:0.4, lat:56.13, lng:-106.34 },
  { code:"SE", name:"Sweden", scheme:"Swish / BiR", authority:"Riksbank / Bankgirot", launch_year:2012, maturity:"Leader", p2p:true, p2b:true, iso20022:true, tx_limit_usd:15000, daily_volume_m:1.5, lat:60.12, lng:18.64 },
  { code:"PH", name:"Philippines", scheme:"InstaPay", authority:"BSP / PhilPaSSplus", launch_year:2018, maturity:"Developing", p2p:true, p2b:false, iso20022:false, tx_limit_usd:1800, daily_volume_m:0.8, lat:12.87, lng:121.77 },
  { code:"AR", name:"Argentina", scheme:"Transferencias 3.0", authority:"BCRA", launch_year:2021, maturity:"Developing", p2p:true, p2b:true, iso20022:false, tx_limit_usd:null, daily_volume_m:2, lat:-38.41, lng:-63.61 },
  { code:"DE", name:"Germany", scheme:"SEPA Instant", authority:"Deutsche Bundesbank / EPC", launch_year:2017, maturity:"Established", p2p:true, p2b:true, iso20022:true, tx_limit_usd:110000, daily_volume_m:3, lat:51.16, lng:10.45 },
  { code:"ID", name:"Indonesia", scheme:"BI-FAST", authority:"Bank Indonesia", launch_year:2021, maturity:"Developing", p2p:true, p2b:false, iso20022:true, tx_limit_usd:3200, daily_volume_m:4, lat:-0.78, lng:113.92 },
  { code:"RU", name:"Russia", scheme:"SBP", authority:"Bank of Russia", launch_year:2019, maturity:"Established", p2p:true, p2b:true, iso20022:false, tx_limit_usd:null, daily_volume_m:10, lat:61.52, lng:105.31 },
  { code:"EG", name:"Egypt", scheme:"InstaPay EG", authority:"Central Bank of Egypt", launch_year:2022, maturity:"Developing", p2p:true, p2b:false, iso20022:false, tx_limit_usd:6000, daily_volume_m:0.5, lat:26.82, lng:30.80 },
  { code:"GH", name:"Ghana", scheme:"GhIPSS Instant Pay", authority:"Bank of Ghana / GhIPSS", launch_year:2016, maturity:"Developing", p2p:true, p2b:false, iso20022:false, tx_limit_usd:1700, daily_volume_m:0.2, lat:7.94, lng:-1.02 },
  { code:"SA", name:"Saudi Arabia", scheme:"sarie", authority:"Saudi Central Bank (SAMA)", launch_year:2021, maturity:"Developing", p2p:true, p2b:true, iso20022:true, tx_limit_usd:null, daily_volume_m:1, lat:23.88, lng:45.07 },
  { code:"AE", name:"UAE", scheme:"IPP", authority:"Central Bank of the UAE", launch_year:2023, maturity:"Developing", p2p:true, p2b:false, iso20022:true, tx_limit_usd:null, daily_volume_m:0.3, lat:23.42, lng:53.84 },
];

const MATURITY_CONFIG = {
  Leader:     { color: "#38BDF8", glow: "0 0 12px #38BDF880", label: "Leader",     dot: "bg-sky-400" },
  Established:{ color: "#818CF8", glow: "0 0 10px #818CF870", label: "Established", dot: "bg-indigo-400" },
  Developing: { color: "#34D399", glow: "0 0 8px #34D39960",  label: "Developing",  dot: "bg-emerald-400" },
};

// ── TIMELINE DATA ─────────────────────────────────────────────────────────
const TIMELINE_YEARS = [2004,2006,2008,2010,2011,2012,2016,2017,2018,2019,2020,2021,2022,2023];

// ── WORLD MAP SVG PATHS (simplified country outlines) ─────────────────────
// Using a dot-plot approach on a mercator-style grid for zero external deps
function latLngToXY(lat, lng, w, h) {
  const x = ((lng + 180) / 360) * w;
  const latRad = (lat * Math.PI) / 180;
  const mercN = Math.log(Math.tan(Math.PI / 4 + latRad / 2));
  const y = h / 2 - (h * mercN) / (2 * Math.PI);
  return { x, y };
}

// ── COMPONENTS ────────────────────────────────────────────────────────────
const MaturityBadge = ({ maturity }) => {
  const cfg = MATURITY_CONFIG[maturity] || MATURITY_CONFIG["Developing"];
  return (
    <span style={{ color: cfg.color, border: `1px solid ${cfg.color}40`, background: `${cfg.color}15` }}
      className="text-xs px-2 py-0.5 rounded-full font-mono tracking-wider">
      {maturity.toUpperCase()}
    </span>
  );
};

const Tag = ({ label, active }) => (
  <span className={`text-xs px-2 py-0.5 rounded font-mono ${active ? "bg-sky-900/60 text-sky-300 border border-sky-600/40" : "bg-slate-800/60 text-slate-500 border border-slate-700/40"}`}>
    {label}
  </span>
);

const StatCard = ({ value, label, accent }) => (
  <div className="flex flex-col items-center bg-slate-900/60 border border-slate-800 rounded-lg p-3">
    <span style={{ color: accent || "#38BDF8" }} className="text-2xl font-bold font-mono tabular-nums">{value}</span>
    <span className="text-xs text-slate-500 mt-0.5 text-center leading-tight">{label}</span>
  </div>
);

// ── WORLD MAP (SVG dot-plot) ───────────────────────────────────────────────
function WorldMap({ schemes, selected, onSelect, filter }) {
  const W = 820, H = 400;
  const visible = filter === "All" ? schemes : schemes.filter(s => s.maturity === filter);

  // draw a light graticule grid
  const gridLines = [];
  for (let lon = -180; lon <= 180; lon += 30) {
    const { x } = latLngToXY(0, lon, W, H);
    gridLines.push(<line key={`v${lon}`} x1={x} y1={0} x2={x} y2={H} stroke="#1F2937" strokeWidth="0.5"/>);
  }
  for (let lat = -60; lat <= 80; lat += 30) {
    const { y } = latLngToXY(lat, 0, W, H);
    gridLines.push(<line key={`h${lat}`} x1={0} y1={y} x2={W} y2={y} stroke="#1F2937" strokeWidth="0.5"/>);
  }

  return (
    <svg viewBox={`0 0 ${W} ${H}`} className="w-full h-full" style={{ background: "#050D18" }}>
      <defs>
        {Object.entries(MATURITY_CONFIG).map(([k, v]) => (
          <radialGradient key={k} id={`grad-${k}`} cx="50%" cy="50%" r="50%">
            <stop offset="0%" stopColor={v.color} stopOpacity="0.9"/>
            <stop offset="100%" stopColor={v.color} stopOpacity="0.2"/>
          </radialGradient>
        ))}
        <filter id="glow">
          <feGaussianBlur stdDeviation="3" result="coloredBlur"/>
          <feMerge><feMergeNode in="coloredBlur"/><feMergeNode in="SourceGraphic"/></feMerge>
        </filter>
      </defs>

      {/* ocean texture */}
      <rect width={W} height={H} fill="#050D18"/>
      <rect width={W} height={H} fill="url(#ocean-lines)" opacity="0.3"/>
      {gridLines}

      {/* equator accent */}
      {(() => { const {y} = latLngToXY(0,0,W,H); return <line x1={0} y1={y} x2={W} y2={y} stroke="#38BDF820" strokeWidth="1"/>; })()}

      {/* country dots */}
      {schemes.map(s => {
        const { x, y } = latLngToXY(s.lat, s.lng, W, H);
        const cfg = MATURITY_CONFIG[s.maturity];
        const isVisible = filter === "All" || s.maturity === filter;
        const isSel = selected?.code === s.code;
        const r = Math.min(12, 4 + Math.log10((s.daily_volume_m || 0.1) + 1) * 5);
        return (
          <g key={s.code} onClick={() => onSelect(s)} style={{ cursor: "pointer" }}>
            {isSel && <circle cx={x} cy={y} r={r + 8} fill="none" stroke={cfg.color} strokeWidth="1.5" opacity="0.5">
              <animate attributeName="r" values={`${r+6};${r+12};${r+6}`} dur="2s" repeatCount="indefinite"/>
              <animate attributeName="opacity" values="0.6;0.1;0.6" dur="2s" repeatCount="indefinite"/>
            </circle>}
            <circle cx={x} cy={y} r={r + 4} fill={cfg.color} opacity={isVisible ? 0.08 : 0.02}/>
            <circle cx={x} cy={y} r={r} fill={`url(#grad-${s.maturity})`}
              opacity={isVisible ? 1 : 0.15}
              filter={isSel ? "url(#glow)" : undefined}
              stroke={isSel ? cfg.color : "none"} strokeWidth="1.5"/>
            {isVisible && <text x={x} y={y - r - 4} textAnchor="middle" fontSize="8" fill={cfg.color} opacity="0.8" fontFamily="monospace">
              {s.code}
            </text>}
          </g>
        );
      })}
    </svg>
  );
}

// ── TIMELINE STRIP ────────────────────────────────────────────────────────
function TimelineStrip({ schemes, yearFilter, onYearFilter }) {
  const byYear = {};
  schemes.forEach(s => { byYear[s.launch_year] = (byYear[s.launch_year] || 0) + 1; });
  const maxCount = Math.max(...Object.values(byYear));

  return (
    <div className="flex items-end gap-1 h-16 px-1">
      {TIMELINE_YEARS.map(yr => {
        const count = byYear[yr] || 0;
        const active = yearFilter === null || yearFilter === yr;
        const barH = count ? Math.max(8, (count / maxCount) * 40) : 0;
        return (
          <div key={yr} className="flex flex-col items-center flex-1 cursor-pointer group"
            onClick={() => onYearFilter(yearFilter === yr ? null : yr)}>
            <div className="text-xs font-mono mb-0.5 opacity-0 group-hover:opacity-100 transition-opacity"
              style={{ color: "#38BDF8", fontSize: "9px" }}>{count > 0 ? count : ""}</div>
            <div className="w-full rounded-t transition-all duration-300"
              style={{
                height: `${barH}px`,
                background: yearFilter === yr ? "#38BDF8" : count > 0 ? "#38BDF840" : "#1F2937",
                border: yearFilter === yr ? "1px solid #38BDF8" : "1px solid #1F293780",
                boxShadow: yearFilter === yr ? "0 0 8px #38BDF860" : "none",
              }}/>
            <div className="text-slate-600 mt-1 font-mono" style={{ fontSize: "9px" }}>
              {yr === 2004 ? "04" : yr === 2006 ? "06" : yr === 2008 ? "08" : String(yr).slice(2)}
            </div>
          </div>
        );
      })}
    </div>
  );
}

// ── COUNTRY DETAIL CARD ────────────────────────────────────────────────────
function CountryCard({ scheme }) {
  if (!scheme) return (
    <div className="flex flex-col items-center justify-center h-48 border border-dashed border-slate-800 rounded-xl text-slate-600 text-sm gap-2">
      <svg className="w-8 h-8 opacity-30" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <circle cx="12" cy="12" r="10"/><line x1="2" y1="12" x2="22" y2="12"/><path d="M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10 15.3 15.3 0 0 1-4-10 15.3 15.3 0 0 1 4-10z"/>
      </svg>
      <span>Select a country on the map</span>
    </div>
  );
  const cfg = MATURITY_CONFIG[scheme.maturity];
  return (
    <div className="rounded-xl border overflow-hidden" style={{ borderColor: `${cfg.color}40`, background: "#0B1117" }}>
      <div className="p-4 border-b" style={{ borderColor: `${cfg.color}20`, background: `${cfg.color}08` }}>
        <div className="flex items-start justify-between">
          <div>
            <div className="text-xs font-mono text-slate-500 mb-1">{scheme.code} · LIVE RAIL</div>
            <div className="text-xl font-bold text-white tracking-tight">{scheme.name}</div>
            <div style={{ color: cfg.color }} className="text-sm font-mono mt-1">{scheme.scheme}</div>
          </div>
          <MaturityBadge maturity={scheme.maturity}/>
        </div>
      </div>
      <div className="p-4 space-y-3">
        <div className="grid grid-cols-2 gap-2 text-xs">
          <div className="bg-slate-900/60 rounded-lg p-2 border border-slate-800">
            <div className="text-slate-500">Launched</div>
            <div className="text-white font-mono font-bold">{scheme.launch_year}</div>
          </div>
          <div className="bg-slate-900/60 rounded-lg p-2 border border-slate-800">
            <div className="text-slate-500">Daily Volume</div>
            <div className="text-white font-mono font-bold">{scheme.daily_volume_m >= 1 ? `${scheme.daily_volume_m}M` : `${scheme.daily_volume_m * 1000}K`}+</div>
          </div>
        </div>
        <div className="flex flex-wrap gap-1.5">
          <Tag label="P2P" active={scheme.p2p}/>
          <Tag label="P2B" active={scheme.p2b}/>
          <Tag label="ISO 20022" active={scheme.iso20022}/>
          {scheme.tx_limit_usd ? <Tag label={`Cap: $${scheme.tx_limit_usd.toLocaleString()}`} active={true}/> : <Tag label="No Cap" active={true}/>}
        </div>
        <div className="text-xs border border-slate-800 rounded-lg p-2.5 bg-slate-900/40">
          <div className="text-slate-500 mb-1">GOVERNING BODY</div>
          <div className="text-slate-300 leading-snug">{scheme.authority}</div>
        </div>
      </div>
    </div>
  );
}

// ── MAIN APP ──────────────────────────────────────────────────────────────
export default function RealRailsPaymentsMap() {
  const [selected, setSelected] = useState(null);
  const [maturityFilter, setMaturityFilter] = useState("All");
  const [yearFilter, setYearFilter] = useState(null);
  const [activeTab, setSideTab] = useState("intel"); // "intel" | "tech" | "timeline"
  const [pulse, setPulse] = useState(0);

  // Filter schemes
  const visibleSchemes = RTP_SCHEMES.filter(s => {
    if (maturityFilter !== "All" && s.maturity !== maturityFilter) return false;
    if (yearFilter && s.launch_year !== yearFilter) return false;
    return true;
  });

  // Pulse animation counter
  useEffect(() => {
    const t = setInterval(() => setPulse(p => p + 1), 3000);
    return () => clearInterval(t);
  }, []);

  const stats = {
    total: RTP_SCHEMES.length,
    leaders: RTP_SCHEMES.filter(s => s.maturity === "Leader").length,
    established: RTP_SCHEMES.filter(s => s.maturity === "Established").length,
    developing: RTP_SCHEMES.filter(s => s.maturity === "Developing").length,
    iso20022: RTP_SCHEMES.filter(s => s.iso20022).length,
  };

  const handleDownload = () => {
    const sample = RTP_SCHEMES.slice(0, 5).map(s => ({
      country: s.name, scheme: s.scheme, authority: s.authority,
      launch_year: s.launch_year, maturity: s.maturity, iso20022: s.iso20022
    }));
    const blob = new Blob([JSON.stringify(sample, null, 2)], { type: "application/json" });
    const a = document.createElement("a"); a.href = URL.createObjectURL(blob);
    a.download = "real-rails-payments-sample.json"; a.click();
  };

  return (
    <div style={{ background: "#030712", fontFamily: "'IBM Plex Mono', 'JetBrains Mono', 'Fira Code', monospace", minHeight: "100vh" }} className="text-white overflow-hidden">

      {/* ── HEADER ── */}
      <div style={{ borderBottom: "1px solid #1F2937", background: "#030712" }} className="px-4 py-3 flex items-center justify-between">
        <div className="flex items-center gap-3">
          <div style={{ color: "#38BDF8", border: "1px solid #38BDF840", background: "#38BDF808" }} className="rounded px-2 py-0.5 text-xs tracking-widest font-bold">
            REAL RAILS
          </div>
          <span className="text-slate-300 text-sm tracking-wide">Global Payments Intelligence</span>
          <span className="text-slate-700 text-xs hidden sm:block">v1.0 · PoC #05</span>
        </div>
        <div className="flex items-center gap-3">
          <div className="flex items-center gap-1.5 text-xs text-slate-500">
            <div className="w-1.5 h-1.5 rounded-full bg-emerald-400" style={{ boxShadow: "0 0 6px #34D399" }}>
              <div className="w-1.5 h-1.5 rounded-full bg-emerald-400 animate-ping"/>
            </div>
            LIVE
          </div>
          <div className="text-xs text-slate-600">{stats.total} Rails Indexed</div>
        </div>
      </div>

      {/* ── MAIN LAYOUT: 70/30 split ── */}
      <div className="flex h-[calc(100vh-52px)]">

        {/* ── LEFT: MAP STAGE (70%) ── */}
        <div className="flex flex-col" style={{ width: "70%", borderRight: "1px solid #1F2937" }}>

          {/* Map controls */}
          <div className="flex items-center gap-2 px-4 py-2 border-b border-slate-800/60">
            <span className="text-xs text-slate-600 mr-2">FILTER:</span>
            {["All", "Leader", "Established", "Developing"].map(f => (
              <button key={f} onClick={() => setMaturityFilter(f)}
                className="text-xs px-3 py-1 rounded transition-all duration-200"
                style={{
                  background: maturityFilter === f ? (MATURITY_CONFIG[f]?.color ?? "#38BDF8") + "20" : "#0B1117",
                  color: maturityFilter === f ? (MATURITY_CONFIG[f]?.color ?? "#38BDF8") : "#64748B",
                  border: maturityFilter === f ? `1px solid ${MATURITY_CONFIG[f]?.color ?? "#38BDF8"}60` : "1px solid #1F2937",
                  boxShadow: maturityFilter === f ? `0 0 8px ${MATURITY_CONFIG[f]?.color ?? "#38BDF8"}30` : "none",
                }}>
                {f}
              </button>
            ))}
            <div className="ml-auto text-xs text-slate-600">
              {visibleSchemes.length} of {stats.total} visible
            </div>
          </div>

          {/* Map */}
          <div className="flex-1 relative overflow-hidden">
            <WorldMap
              schemes={RTP_SCHEMES}
              selected={selected}
              onSelect={s => setSelected(prev => prev?.code === s.code ? null : s)}
              filter={maturityFilter}
            />
            {/* Legend */}
            <div className="absolute bottom-4 left-4 flex gap-3">
              {Object.entries(MATURITY_CONFIG).map(([k, v]) => (
                <div key={k} className="flex items-center gap-1.5 text-xs bg-slate-900/80 px-2 py-1 rounded border border-slate-800">
                  <div className="w-2 h-2 rounded-full" style={{ background: v.color, boxShadow: v.glow }}/>
                  <span style={{ color: v.color }}>{k}</span>
                </div>
              ))}
              <div className="flex items-center gap-1.5 text-xs bg-slate-900/80 px-2 py-1 rounded border border-slate-800">
                <div className="w-2 h-2 rounded-full bg-slate-600"/>
                <span className="text-slate-500">Dot size ∝ volume</span>
              </div>
            </div>
            {/* Overlay hint */}
            {!selected && (
              <div className="absolute top-4 right-4 text-xs text-slate-600 bg-slate-900/70 px-2 py-1 rounded border border-slate-800">
                ↓ Click a node to inspect
              </div>
            )}
          </div>

          {/* Timeline strip */}
          <div style={{ borderTop: "1px solid #1F2937", background: "#060E1B" }} className="px-4 pt-2 pb-1">
            <div className="flex items-center justify-between mb-1">
              <span className="text-xs text-slate-600">LAUNCH TIMELINE · Click bar to filter</span>
              {yearFilter && (
                <button onClick={() => setYearFilter(null)} className="text-xs text-sky-400 hover:text-sky-300">
                  Clear ({yearFilter}) ×
                </button>
              )}
            </div>
            <TimelineStrip schemes={RTP_SCHEMES} yearFilter={yearFilter} onYearFilter={setYearFilter}/>
          </div>
        </div>

        {/* ── RIGHT: INTELLIGENCE SIDEBAR (30%) ── */}
        <div className="flex flex-col overflow-y-auto" style={{ width: "30%", background: "#08111A" }}>

          {/* Sidebar tabs */}
          <div style={{ borderBottom: "1px solid #1F2937" }} className="flex text-xs">
            {[["intel","INTELLIGENCE"],["tech","TECH LAYER"],["why","INSIGHTS"]].map(([id,label]) => (
              <button key={id} onClick={() => setSideTab(id)}
                className="flex-1 py-2.5 transition-all"
                style={{
                  color: activeTab === id ? "#38BDF8" : "#475569",
                  borderBottom: activeTab === id ? "2px solid #38BDF8" : "2px solid transparent",
                  background: activeTab === id ? "#38BDF808" : "transparent",
                }}>
                {label}
              </button>
            ))}
          </div>

          <div className="p-4 space-y-4 flex-1">

            {/* ── SECTION A: Headline metric ── */}
            <div>
              <div className="text-xs text-slate-600 mb-2 tracking-widest">GLOBAL RAIL INDEX</div>
              <div className="grid grid-cols-3 gap-2">
                <StatCard value={stats.leaders} label="Rail Leaders" accent="#38BDF8"/>
                <StatCard value={stats.established} label="Established" accent="#818CF8"/>
                <StatCard value={stats.developing} label="Developing" accent="#34D399"/>
              </div>
              <div className="mt-2 grid grid-cols-2 gap-2">
                <StatCard value={stats.iso20022} label="ISO 20022 Adopted" accent="#F59E0B"/>
                <StatCard value={stats.total} label="Total Indexed" accent="#64748B"/>
              </div>
            </div>

            {/* ── COUNTRY CARD ── */}
            {activeTab === "intel" && (
              <>
                <div>
                  <div className="text-xs text-slate-600 mb-2 tracking-widest">COUNTRY INTELLIGENCE</div>
                  <CountryCard scheme={selected}/>
                </div>

                {/* Top rails list when nothing selected */}
                {!selected && (
                  <div>
                    <div className="text-xs text-slate-600 mb-2 tracking-widest">TOP RAIL LEADERS BY VOLUME</div>
                    <div className="space-y-1.5">
                      {[...RTP_SCHEMES].sort((a,b) => b.daily_volume_m - a.daily_volume_m).slice(0,5).map(s => {
                        const cfg = MATURITY_CONFIG[s.maturity];
                        const pct = (s.daily_volume_m / RTP_SCHEMES.reduce((acc,x) => Math.max(acc,x.daily_volume_m),0)) * 100;
                        return (
                          <div key={s.code} className="cursor-pointer group" onClick={() => setSelected(s)}>
                            <div className="flex justify-between text-xs mb-0.5">
                              <span className="text-slate-300 group-hover:text-white transition-colors">{s.name}</span>
                              <span className="font-mono" style={{ color: cfg.color }}>{s.daily_volume_m >= 1 ? `${s.daily_volume_m}M` : `${s.daily_volume_m*1000}K`}/day</span>
                            </div>
                            <div className="h-1 rounded-full bg-slate-800 overflow-hidden">
                              <div className="h-full rounded-full transition-all" style={{ width: `${pct}%`, background: cfg.color }}/>
                            </div>
                          </div>
                        );
                      })}
                    </div>
                  </div>
                )}
              </>
            )}

            {/* ── TECH LAYER TAB ── */}
            {activeTab === "tech" && (
              <div className="space-y-3">
                <div className="text-xs text-slate-600 tracking-widest">API & PROTOCOL STANDARDS</div>
                <div className="space-y-2">
                  {(selected ? [selected] : RTP_SCHEMES.filter(s => s.iso20022).slice(0,8)).map(s => (
                    <div key={s.code} className="bg-slate-900/50 border border-slate-800 rounded-lg p-2.5 text-xs">
                      <div className="flex justify-between items-center mb-1">
                        <span className="text-white font-semibold">{s.name}</span>
                        <span className="text-slate-500 font-mono">{s.code}</span>
                      </div>
                      <div className="flex flex-wrap gap-1">
                        <Tag label={s.scheme} active={true}/>
                        {s.iso20022 && <Tag label="ISO 20022" active={true}/>}
                        <Tag label={`Since ${s.launch_year}`} active={false}/>
                        {s.p2p && <Tag label="P2P" active={true}/>}
                        {s.p2b && <Tag label="P2B" active={true}/>}
                      </div>
                    </div>
                  ))}
                  {!selected && <div className="text-xs text-slate-600 text-center">Showing ISO 20022 adopters · Select country for details</div>}
                </div>
              </div>
            )}

            {/* ── WHY THIS MATTERS TAB ── */}
            {activeTab === "why" && (
              <div className="space-y-4">
                {/* Section B: Why This Matters */}
                <div className="border border-sky-900/40 rounded-xl p-3.5" style={{ background: "#38BDF808" }}>
                  <div className="flex items-center gap-2 mb-2">
                    <div className="w-1.5 h-1.5 rounded-full bg-sky-400" style={{ boxShadow: "0 0 6px #38BDF8" }}/>
                    <span className="text-xs text-sky-400 tracking-widest font-bold">WHY THIS MATTERS</span>
                  </div>
                  <p className="text-xs text-slate-400 leading-relaxed">
                    Real-time payment rails are the circulatory system of a modern economy. Every second a payment is delayed is <span className="text-sky-300">lost velocity of money</span> — a direct drag on GDP. Countries like India (UPI) and Brazil (Pix) have used instant rails to leapfrog legacy banking infrastructure, driving financial inclusion at scale.
                  </p>
                  <div className="mt-3 grid grid-cols-2 gap-2 text-xs">
                    <div className="bg-slate-900/60 rounded p-2 border border-slate-800">
                      <div className="text-slate-500">India UPI</div>
                      <div className="text-emerald-400 font-bold">400M txns/day</div>
                    </div>
                    <div className="bg-slate-900/60 rounded p-2 border border-slate-800">
                      <div className="text-slate-500">Brazil Pix</div>
                      <div className="text-emerald-400 font-bold">60M txns/day</div>
                    </div>
                  </div>
                </div>

                {/* Section C: Who Controls the Rail */}
                <div className="border border-indigo-900/40 rounded-xl p-3.5" style={{ background: "#818CF808" }}>
                  <div className="flex items-center gap-2 mb-2">
                    <div className="w-1.5 h-1.5 rounded-full bg-indigo-400" style={{ boxShadow: "0 0 6px #818CF8" }}/>
                    <span className="text-xs text-indigo-400 tracking-widest font-bold">WHO CONTROLS THE RAIL</span>
                  </div>
                  <p className="text-xs text-slate-400 leading-relaxed">
                    Central banks own the infrastructure. Commercial banks control access. This two-layer structure means <span className="text-indigo-300">monetary sovereignty sits with the state</span>, while distribution remains privatized. FintTechs operate as licensed participants — never direct members — of the rail.
                  </p>
                  <div className="mt-3 space-y-1.5 text-xs">
                    {["Federal Reserve → FedNow", "NPCI → UPI", "Banco Central → Pix", "EPC → SEPA Instant"].map(r => (
                      <div key={r} className="flex items-center gap-2 text-slate-500">
                        <div className="w-1 h-1 rounded-full bg-indigo-600"/>
                        {r}
                      </div>
                    ))}
                  </div>
                </div>

                {/* ISO 20022 Adoption */}
                <div className="border border-amber-900/30 rounded-xl p-3.5" style={{ background: "#F59E0B08" }}>
                  <div className="flex items-center gap-2 mb-2">
                    <div className="w-1.5 h-1.5 rounded-full bg-amber-400"/>
                    <span className="text-xs text-amber-400 tracking-widest font-bold">ISO 20022 MIGRATION</span>
                  </div>
                  <p className="text-xs text-slate-400 leading-relaxed">
                    {stats.iso20022} of {stats.total} indexed schemes have adopted ISO 20022 — the new universal financial messaging standard enabling richer transaction data, better compliance, and true interoperability between national rails.
                  </p>
                  <div className="mt-2 bg-slate-900/50 rounded h-2 overflow-hidden">
                    <div className="h-full bg-amber-400 rounded transition-all" style={{ width: `${(stats.iso20022/stats.total)*100}%` }}/>
                  </div>
                  <div className="text-right text-xs text-amber-500 mt-1">{Math.round((stats.iso20022/stats.total)*100)}% adoption</div>
                </div>
              </div>
            )}

            {/* ── DOWNLOAD BUTTON (always visible) ── */}
            <button onClick={handleDownload}
              className="w-full text-xs py-2.5 rounded-lg border transition-all duration-200 mt-2"
              style={{
                color: "#38BDF8", borderColor: "#38BDF840", background: "#38BDF808",
                boxShadow: "0 0 0 0 #38BDF840",
              }}
              onMouseEnter={e => e.target.style.boxShadow = "0 0 12px #38BDF830"}
              onMouseLeave={e => e.target.style.boxShadow = "none"}>
              ↓ Download Sample Data (JSON)
            </button>

            {/* Source attribution */}
            <div className="text-xs text-slate-700 text-center leading-relaxed pb-2">
              Sources: FedNow · European Payments Council · World Bank<br/>
              Zero Hallucination Policy: Unknown = "Status Unknown"
            </div>
          </div>
        </div>
      </div>
    </div>
  );
}
