import { useState, useRef } from "react";
const TODAY = new Date();
const initialFoods = [
{ id: 1, name: "Toyuq salatı", added: "2026-05-04", expiry: "2026-05-07", category: "Salat"
{ id: 2, name: "Düyü", added: "2026-05-05", expiry: "2026-05-10", category: "Dən", recipe:
{ id: 3, name: "Lobya yeməyi", added: "2026-05-04", expiry: "2026-05-08", category: "Ərzaq"
{ id: 4, name: "Makaron", added: "2026-05-06", expiry: "2026-05-13", category: "Pasta", rec
{ id: 5, name: "Süd", added: "2026-05-05", expiry: "2026-05-07", category: "Südlü", recipe:
];
const initialHistory = [
{ name: "Kərə yağı", onTime: true },
{ name: "Yoğurt", onTime: true },
{ name: "Pomidor", onTime: false },
{ name: "Pendir", onTime: true },
{ name: "Yumurta", onTime: true },
{ name: "Alma", onTime: true },
{ name: "Çörək", onTime: false },
{ name: "Ayran", onTime: true },
];
const CAT_EMOJI = { Salat: " ", Dən: " ", Ərzaq: " ", Pasta: " ", Südlü: " ", Meyvə: "
function daysLeft(expiry) {
return Math.ceil((new Date(expiry) - TODAY) / 86400000);
}
function Badge({ days }) {
const cfg = days <= 1 ? { bg: "#ef4444", label: "Kritik" }
: days <= 3 ? { bg: "#f59e0b", label: "Tezliklə" }
: { bg: "#22c55e", label: "Normal" };
return (
<span style={{ background: cfg.bg, color: "#fff", fontSize: 10, fontWeight: 800, borderRa
{cfg.label}
</span>
);
}
async function fetchAIRecipe(foodName, category) {
const response = await fetch("https://api.anthropic.com/v1/messages", {
method: "POST",
headers: { "Content-Type": "application/json" },
body: JSON.stringify({
model: "claude-sonnet-4-20250514",
max_tokens: 1000,
messages: [{
role: "user",
content: `Sən bir aşpaz köməkçisisən. "${foodName}" (kateqoriya: ${category}) məhsulu
}]
})
});
const data = await response.json();
return data.content?.[0]?.text || "Resept tapılmadı.";
}
export default function App() {
const [tab, setTab] = useState(0);
const [foods, setFoods] = useState(initialFoods);
const [history, setHistory] = useState(initialHistory);
const [showAdd, setShowAdd] = useState(false);
const [detail, setDetail] = useState(null);
const [editingRecipe, setEditingRecipe] = useState(false);
const [editRecipeText, setEditRecipeText] = useState("");
const [newFood, setNewFood] = useState({ name: "", expiry: "", category: "Salat", recipe: "
const [toast, setToast] = useState("");
const [scanning, setScanning] = useState(false);
const [dark, setDark] = useState(false);
const [aiLoading, setAiLoading] = useState(false);
const [aiLoadingDetail, setAiLoadingDetail] = useState(false);
const [statFilter, setStatFilter] = useState(null);
// ── YENİ: Profil state-ləri ──
const [profilePhoto, setProfilePhoto] = useState(null);
const [profileName, setProfileName] = useState("İstifadəçi");
const [editingName, setEditingName] = useState(false);
const [tempName, setTempName] = useState("");
const [showPhotoOptions, setShowPhotoOptions] = useState(false);
const fileInputRef = useRef(null);
const expiring = foods.filter(f => daysLeft(f.expiry) <= 3);
const safe = foods.filter(f => daysLeft(f.expiry) > 3);
const critical = foods.filter(f => daysLeft(f.expiry) <= 1);
const totalDeleted = history.length;
const onTimeCount = history.filter(h => h.onTime).length;
const t = {
bg: dark ? "#0f172a" : "#f0fdf4",
card: dark ? "#1e293b" : "#ffffff",
border: dark ? "#334155" : "#e5e7eb",
borderRed: dark ? "#7f1d1d" : "#fca5a5",
text: dark ? "#f1f5f9" : "#1e293b",
sub: dark ? "#94a3b8" : "#999999",
sub2: dark ? "#64748b" : "#aaaaaa",
tabBg: dark ? "#1e293b" : "#ffffff",
tabBorder: dark ? "#334155" : "#e5e7eb",
inputBg: dark ? "#1e293b" : "#ffffff",
inputBorder: dark ? "#475569" : "#e5e7eb",
recipeBlock: dark ? "#0f2a1f" : "#f0fdf4",
shadow: dark ? "rgba(0,0,0,0.4)" : "rgba(0,0,0,0.06)",
};
function notify(msg) {
setToast(msg);
setTimeout(() => setToast(""), 2500);
}
// ── Profil şəkli funksiyaları ──
function handlePhotoSelect(e) {
const file = e.target.files?.[0];
if (!file) return;
if (!file.type.startsWith("image/")) { notify(" const reader = new FileReader();
reader.onload = (ev) => {
setProfilePhoto(ev.target.result);
notify(" Profil şəkli yeniləndi!");
Yalnız şəkil faylı seçin!"); return; }
};
reader.readAsDataURL(file);
setShowPhotoOptions(false);
}
function handleRemovePhoto() {
setProfilePhoto(null);
setShowPhotoOptions(false);
notify(" Profil şəkli silindi.");
}
function handleSaveName() {
if (tempName.trim()) {
setProfileName(tempName.trim());
notify(" Ad saxlanıldı!");
}
setEditingName(false);
}
async function handleAIRecipeForNew() {
if (!newFood.name) { notify(" Əvvəlcə məhsul adını yazın!"); return; }
setAiLoading(true);
try {
const recipe = await fetchAIRecipe(newFood.name, newFood.category);
setNewFood(p => ({ ...p, recipe }));
notify(" AI resept hazırladı!");
} catch { notify(" Xəta baş verdi."); }
setAiLoading(false);
}
async function handleAIRecipeForDetail() {
if (!detail) return;
setAiLoadingDetail(true);
try {
const recipe = await fetchAIRecipe(detail.name, detail.category);
setFoods(p => p.map(f => f.id === detail.id ? { ...f, recipe } : f));
setDetail(d => ({ ...d, recipe }));
notify(" AI resept yeniləndi!");
} catch { notify(" Xəta baş verdi."); }
setAiLoadingDetail(false);
}
function saveEditedRecipe() {
setFoods(p => p.map(f => f.id === detail.id ? { ...f, recipe: editRecipeText } : f));
setDetail(d => ({ ...d, recipe: editRecipeText }));
setEditingRecipe(false);
notify(" Resept saxlanıldı!");
}
function addFood() {
if (!newFood.name || !newFood.expiry) return;
setFoods(p => [...p, {
id: Date.now(), name: newFood.name,
added: new Date().toISOString().split("T")[0], expiry: newFood.expiry,
category: newFood.category,
recipe: newFood.recipe || "Resept hələ əlavə edilməyib."
}]);
setNewFood({ name: "", expiry: "", category: "Salat", recipe: "" });
setShowAdd(false);
notify(" Məhsul əlavə edildi!");
}
function deleteFood(id) {
const food = foods.find(f => f.id === id);
if (food) {
const onTime = daysLeft(food.expiry) >= 0;
setHistory(p => [...p, { name: food.name, onTime }]);
}
setFoods(p => p.filter(f => f.id !== id));
setDetail(null);
setEditingRecipe(false);
notify(food && daysLeft(food.expiry) >= 0 ? " Vaxtında istifadə edildi!" : " Məhsul b
}
function scan() {
setScanning(true);
setTimeout(() => { setScanning(false); setShowAdd(true); notify(" QR uğurla oxundu!");
}
const tabs = [
{ icon: " { icon: " { icon: " { icon: " ", label: "Əsas" },
", label: "Ərzaqlar" },
", label: "Reseptlər" },
", label: "Profil" },
];
const cardStyle = (urgent) => ({
background: t.card, borderRadius: 14, margin: "0 0 8px",
padding: "12px 14px", display: "flex", alignItems: "center", gap: 12,
boxShadow: urgent ? "0 2px 12px rgba(239,68,68,0.2)" : `0 2px 8px ${t.shadow}`,
border: urgent ? `1.5px solid ${t.borderRed}` : `1.5px solid ${t.border}`,
cursor: "pointer", transition: "transform 0.15s",
});
const cardStyleMargin = (urgent) => ({
...cardStyle(urgent), margin: "0 16px 8px",
});
const sectionTitle = { padding: "12px 16px 6px", fontSize: 14, fontWeight: 800, color: t.te
const inputStyle = {
width: "100%", border: `1.5px solid ${t.inputBorder}`, borderRadius: 10,
padding: "11px 13px", fontSize: 14, marginBottom: 10, outline: "none",
background: t.inputBg, color: t.text,
};
const modalBox = {
background: t.card, width: "100%", maxWidth: 420,
margin: "0 auto", borderRadius: "20px 20px 0 0", padding: "24px 20px 36px",
maxHeight: "90vh", overflowY: "auto",
};
const aiBtnStyle = (loading) => ({
width: "100%", background: loading ? "#94a3b8" : "linear-gradient(135deg,#6366f1,#8b5cf6)
color: "#fff", border: "none", borderRadius: 10, padding: "10px",
fontSize: 13, fontWeight: 700, cursor: loading ? "not-allowed" : "pointer",
marginBottom: 10, display: "flex", alignItems: "center", justifyContent: "center", gap: 6
});
const statFilterFoods = statFilter === "umumi" ? foods
: statFilter === "tezlikle" ? expiring
: statFilter === "kritik" ? critical
: [];
const statFilterTitle = statFilter === "umumi" ? " Bütün Məhsullar"
: statFilter === "tezlikle" ? " Tezliklə Bitəcək"
: " Kritik Məhsullar";
return (
<div style={{ minHeight: "100vh", background: t.bg, fontFamily: "'Nunito', sans-serif", m
<style>{`
@import url('https://fonts.googleapis.com/css2?family=Nunito:wght@400;600;700;800;900
* { box-sizing: border-box; margin: 0; padding: 0; }
::-webkit-scrollbar { display: none; }
.card:hover { transform: translateY(-1px); }
textarea:focus { outline: none; }
@keyframes spin { to { transform: rotate(360deg); } }
.spin { animation: spin 1s linear infinite; display: inline-block; }
@keyframes fadeIn { from { opacity: 0; transform: scale(0.95); } to { opacity: .fadeIn { animation: fadeIn 0.2s ease; }
`}</style>
1; tra
{/* Hidden file input */}
<input
ref={fileInputRef}
type="file"
accept="image/*"
style={{ display: "none" }}
onChange={handlePhotoSelect}
/>
{/* TOAST */}
{toast && (
<div style={{ position: "fixed", top: 16, left: "50%", transform: "translateX(-50%)",
{toast}
</div>
)}
{/* HEADER */}
<div style={{ background: "#00A550", color: "#fff", padding: "20px 20px 16px", display:
<div>
<div style={{ fontSize: 22, fontWeight: 900, letterSpacing: -0.5 }}> AntiWaste</d
<div style={{ fontSize: 12, opacity: 0.85, marginTop: 2 }}>
{new Date().toLocaleDateString("az-AZ", { day: "numeric", month: "long", year: "n
</div>
</div>
<div style={{ display: "flex", gap: 8, alignItems: "center" }}>
{/* Header-də kiçik profil şəkli */}
<div
onClick={() => setTab(3)}
style={{ width: 36, height: 36, borderRadius: "50%", overflow: "hidden", border:
>
{profilePhoto
? <img src={profilePhoto} alt="profil" style={{ width: "100%", height: "100%",
: <span style={{ fontSize: 18 }}> </span>
}
</div>
<button onClick={() => setDark(d => !d)} style={{ background: "rgba(255,255,255,0.2
{dark ? " " : " "}
</button>
</div>
</div>
<div style={{ flex: 1, overflowY: "auto", paddingBottom: 74 }}>
{/* ANA SƏHİFƏ */}
{tab === 0 && (
<div>
{[
<div style={{ display: "flex", gap: 10, padding: "16px 16px 8px" }}>
{ label: "Ümumi", val: foods.length, bg: "#0066B3", key: "umumi" },
{ label: "Tezliklə", val: expiring.length, bg: "#EF3340", key: "tezlikle" },
{ label: "Kritik", val: critical.length, bg: "#00A550", key: "kritik" },
].map(s => (
<div key={s.label} onClick={() => setStatFilter(s.key)}
style={{ flex: 1, background: s.bg, borderRadius: 14, padding: "14px onMouseEnter={e => e.currentTarget.style.transform = "scale(1.04)"}
onMouseLeave={e => e.currentTarget.style.transform = "scale(1)"}
10px",
>
<div style={{ fontSize: 28, fontWeight: 900, color: "#fff" }}>{s.val}</div>
<div style={{ fontSize: 11, color: "rgba(255,255,255,0.9)", marginTop: 2 }}
</div>
))}
</div>
<div onClick={scan} style={{ margin: "8px 16px", background: "linear-gradient(135
<div style={{ fontSize: 36 }}>{scanning ? " " : " "}</div>
<div style={{ color: "#fff" }}>
<div style={{ fontWeight: 900, fontSize: 15 }}>QR Kodu Skan Et</div>
<div style={{ fontSize: 12, opacity: 0.9, marginTop: 2 }}>Qabın üzərindəki ko
</div>
</div>
{expiring.length > 0 && (
<>
<div style={sectionTitle}> Tezliklə Bitəcək</div>
{expiring.map(f => (
<div key={f.id} className="card" style={cardStyleMargin(true)} onClick={()
<div style={{ fontSize: 30 }}>{CAT_EMOJI[f.category] || " "}</div>
<div style={{ flex: 1 }}>
<div style={{ fontWeight: 800, fontSize: 15, color: t.text }}>{f.name}<
<div style={{ marginTop: 4 }}><Badge days={daysLeft(f.expiry)} /></div>
</div>
<div style={{ textAlign: "right" }}>
<div style={{ fontSize: 22, fontWeight: 900, color: daysLeft(f.expiry)
<div style={{ fontSize: 10, color: t.sub }}>gün</div>
</div>
</div>
))}
</>
)}
<div style={sectionTitle}> Normal Vəziyyətdə</div>
{safe.length === 0 && <div style={{ padding: "8px 16px", color: t.sub, fontSize:
{safe.map(f => (
<div key={f.id} className="card" style={cardStyleMargin(false)} onClick={() =>
<div style={{ fontSize: 30 }}>{CAT_EMOJI[f.category] || " "}</div>
<div style={{ flex: 1 }}>
<div style={{ fontWeight: 800, fontSize: 15, color: t.text }}>{f.name}</div
<div style={{ fontSize: 12, color: t.sub, marginTop: 2 }}>{f.category}</div
</div>
<div style={{ textAlign: "right" }}>
<div style={{ fontSize: 20, fontWeight: 900, color: "#22c55e" }}>{daysLeft(
<div style={{ fontSize: 10, color: t.sub }}>gün</div>
</div>
</div>
))}
</div>
)}
{/* ƏRZAQLAR */}
{tab === 1 && (
<div>
<div style={sectionTitle}>Bütün Ərzaqlar ({foods.length})</div>
{[...foods].sort((a, b) => daysLeft(a.expiry) - daysLeft(b.expiry)).map(f => (
<div key={f.id} className="card" style={cardStyleMargin(daysLeft(f.expiry) <= 3
<div style={{ fontSize: 30 }}>{CAT_EMOJI[f.category] || " "}</div>
<div style={{ flex: 1 }}>
<div style={{ fontWeight: 800, fontSize: 14, color: t.text }}>{f.name}</div
<div style={{ fontSize: 12, color: t.sub2, marginTop: 2 }}>Bitmə: {f.expiry
</div>
<Badge days={daysLeft(f.expiry)} />
</div>
))}
</div>
)}
bitmir
{/* RESEPTLƏR */}
{tab === 2 && (
<div style={{ padding: 16 }}>
<div style={{ fontSize: 14, fontWeight: 800, color: t.text, marginBottom: 12 }}>
{expiring.length === 0 ? (
<div style={{ textAlign: "center", padding: "40px 0", color: t.sub }}>
<div style={{ fontSize: 52 }}> </div>
<div style={{ fontWeight: 700, marginTop: 12 }}>Heç bir ərzaq tezliklə </div>
) : expiring.map(f => (
<div key={f.id} style={{ background: t.card, borderRadius: 16, padding: 16, mar
<div style={{ display: "flex", alignItems: "center", gap: 10, marginBottom: 1
<div style={{ fontSize: 32 }}>{CAT_EMOJI[f.category] || " "}</div>
<div>
<div style={{ fontWeight: 900, fontSize: 15, color: t.text }}>{f.name}</d
<div style={{ fontSize: 11, color: "#f59e0b", fontWeight: 700 }}>{daysLef
</div>
<button onClick={() => { setDetail(f); setEditingRecipe(false); }} style={{
</div>
<div style={{ background: t.recipeBlock, borderRadius: 10, padding: "10px 12p
{f.recipe}
</div>
</div>
))}
</div>
)}
{/* PROFİL */}
{tab === 3 && (
<div style={{ padding: 16 }}>
{/* Profil Kart */}
<div style={{ background: "linear-gradient(135deg,#16a34a,#4ade80)", borderRadius
{/* Profil Şəkli */}
<div style={{ position: "relative", display: "inline-block", marginBottom: 12 }
<div
onClick={() => setShowPhotoOptions(true)}
style={{
width: 90, height: 90, borderRadius: "50%", overflow: "hidden",
border: "3px solid rgba(255,255,255,0.8)", cursor: "pointer",
background: "rgba(255,255,255,0.25)", display: "flex",
alignItems: "center", justifyContent: "center", margin: "0 auto",
transition: "transform 0.15s",
}}
onMouseEnter={e => e.currentTarget.style.transform = "scale(1.05)"}
onMouseLeave={e => e.currentTarget.style.transform = "scale(1)"}
>
{profilePhoto
? <img src={profilePhoto} alt="profil" style={{ width: "100%", height: "1
: <span style={{ fontSize: 44 }}> </span>
}
</div>
{/* Kamera ikonu */}
<div
onClick={() => setShowPhotoOptions(true)}
style={{
position: "absolute", bottom: 2, right: 2,
background: "#fff", borderRadius: "50%", width: 26, height: 26,
display: "flex", alignItems: "center", justifyContent: "center",
cursor: "pointer", boxShadow: "0 2px 6px rgba(0,0,0,0.2)", fontSize: 13,
}}
>
</div>
</div>
{/* Ad redaktəsi */}
{editingName ? (
<div style={{ display: "flex", gap: 6, justifyContent: "center", alignItems:
<input
value={tempName}
onChange={e => setTempName(e.target.value)}
onKeyDown={e => e.key === "Enter" && handleSaveName()}
autoFocus
style={{ background: "rgba(255,255,255,0.3)", border: "1.5px solid placeholder="Adınızı yazın"
rgba(2
/>
</div>
) : (
<div
<button onClick={handleSaveName} style={{ background: "rgba(255,255,255,0.3
onClick={() => { setTempName(profileName); setEditingName(true); }}
style={{ fontWeight: 900, fontSize: 20, color: "#fff", marginBottom: 4, cur
>
{profileName}
<span style={{ fontSize: 14, opacity: 0.7 }}> </div>
</span>
)}
<div style={{ opacity: 0.85, fontSize: 13, color: "#fff" }}>AntiWaste Üzvü · 20
</div>
{/* Stat kartlar */}
{[
{ icon: " { icon: " { icon: " { icon: " ", label: "Qənaət", val: `₼ ${(onTimeCount * 3).toFixed(2)}` },
", label: "İsraf miqdarı", val: `${totalDeleted - onTimeCount} məhsu
", label: "İzlənən ərzaq", val: `${foods.length} məhsul` },
", label: "Xal", val: `${onTimeCount * 20 + foods.length * 10}` },
].map(item => (
<div key={item.label} style={{ background: t.card, borderRadius: 14, padding: "
<div style={{ fontSize: 24 }}>{item.icon}</div>
<div style={{ flex: 1, fontWeight: 600, color: t.text, fontSize: 14 }}>{item.
<div style={{ fontWeight: 900, color: "#16a34a", fontSize: 15 }}>{item.val}</
</div>
))}
</div>
)}
</div>
{/* TAB BAR */}
<div style={{ position: "fixed", bottom: 0, left: "50%", transform: "translateX(-50%)",
{tabs.slice(0, 2).map((tb, i) => (
<div key={i} onClick={() => setTab(i)} style={{ flex: 1, padding: "10px 0 8px", tex
<div style={{ fontSize: 20 }}>{tb.icon}</div>
<div>{tb.label}</div>
</div>
))}
<div style={{ flex: 1, display: "flex", justifyContent: "center", alignItems: "center
<button onClick={() => setShowAdd(true)} style={{ background: "#16a34a", color: "#f
</div>
{tabs.slice(2).map((tb, i) => (
<div key={i + 2} onClick={() => setTab(i + 2)} style={{ flex: 1, padding: "10px 0 8
<div style={{ fontSize: 20 }}>{tb.icon}</div>
<div>{tb.label}</div>
</div>
))}
</div>
{/* FOTO SEÇİM MODAL */}
{showPhotoOptions && (
<div onClick={() => setShowPhotoOptions(false)} style={{ position: "fixed", inset: 0,
<div onClick={e => e.stopPropagation()} className="fadeIn" style={{ background: t.c
<div style={{ fontWeight: 900, fontSize: 17, color: t.text, marginBottom: 6, text
<div style={{ fontSize: 12, color: t.sub, textAlign: "center", marginBottom: 20 }
<button
onClick={() => fileInputRef.current?.click()}
style={{ width: "100%", background: "#16a34a", color: "#fff", border: "none", b
>
Qalereyadan Seç
</button>
{profilePhoto && (
<button
onClick={handleRemovePhoto}
style={{ width: "100%", background: "#ef4444", color: "#fff", border: "none",
>
Şəkli Sil
</button>
)}
<button
onClick={() => setShowPhotoOptions(false)}
style={{ width: "100%", background: dark ? "#334155" : "#f1f5f9", color: t.text
>
Ləğv et
</button>
</div>
</div>
)}
{/* STAT FİLTER MODAL */}
{statFilter && (
<div onClick={() => setStatFilter(null)} style={{ position: "fixed", inset: 0, backgr
<div onClick={e => e.stopPropagation()} style={{ background: t.card, width: "100%",
<div style={{ display: "flex", alignItems: "center", justifyContent: "space-betwe
<div style={{ fontWeight: 900, fontSize: 17, color: t.text }}>{statFilterTitle}
<button onClick={() => setStatFilter(null)} style={{ background: "none", border
</div>
{statFilterFoods.length === 0 ? (
<div style={{ textAlign: "center", padding: "30px 0", color: t.sub }}>
<div style={{ fontSize: 40 }}> </div>
<div style={{ fontWeight: 700, marginTop: 8 }}>Bu kateqoriyada məhsul yoxdur<
</div>
) : statFilterFoods.map(f => (
<div key={f.id} className="card" style={cardStyle(daysLeft(f.expiry) <= 3)} onC
<div style={{ fontSize: 30 }}>{CAT_EMOJI[f.category] || " "}</div>
<div style={{ flex: 1 }}>
<div style={{ fontWeight: 800, fontSize: 15, color: t.text }}>{f.name}</div
<div style={{ marginTop: 4 }}><Badge days={daysLeft(f.expiry)} /></div>
</div>
<div style={{ textAlign: "right" }}>
<div style={{ fontSize: 20, fontWeight: 900, color: daysLeft(f.expiry) <= 1
<div style={{ fontSize: 10, color: t.sub }}>gün</div>
</div>
</div>
))}
</div>
</div>
)}
{/* ADD MODAL */}
{showAdd && (
<div onClick={() => setShowAdd(false)} style={{ position: "fixed", inset: 0, backgrou
<div onClick={e => e.stopPropagation()} style={modalBox}>
<div style={{ fontWeight: 900, fontSize: 18, color: t.text, marginBottom: 16 }}>
<input value={newFood.name} onChange={e => setNewFood(p => ({ ...p, name: e.targe
<input type="date" value={newFood.expiry} onChange={e => setNewFood(p => ({ ...p,
<select value={newFood.category} onChange={e => setNewFood(p => ({ ...p, category
{Object.keys(CAT_EMOJI).map(c => <option key={c}>{c}</option>)}
</select>
<button onClick={handleAIRecipeForNew} disabled={aiLoading} style={aiBtnStyle(aiL
{aiLoading ? <span className="spin"> </span> : " "} {aiLoading ? "AI resept h
</button>
<textarea value={newFood.recipe} onChange={e => setNewFood(p => ({ ...p, recipe:
<button onClick={addFood} style={{ width: "100%", background: "#16a34a", color: "
<button onClick={() => setShowAdd(false)} style={{ width: "100%", background: dar
</div>
</div>
)}
{/* DETAIL MODAL */}
{detail && (
<div onClick={() => { setDetail(null); setEditingRecipe(false); }} style={{ position:
<div onClick={e => e.stopPropagation()} style={modalBox}>
<div style={{ textAlign: "center", marginBottom: 16 }}>
<div style={{ fontSize: 54 }}>{CAT_EMOJI[detail.category] || " "}</div>
<div style={{ fontWeight: 900, fontSize: 20, color: t.text, marginTop: 8 <div style={{ marginTop: 6 }}><Badge days={daysLeft(detail.expiry)} /></div>
</div>
}}>{de
{[
[" Əlavə tarixi", detail.added],
[" Bitmə tarixi", detail.expiry],
[" Kateqoriya", detail.category],
[" Qalan gün", `${daysLeft(detail.expiry)} gün`]
].map(([k, v]) => (
<div key={k} style={{ display: "flex", justifyContent: "space-between", padding
<span style={{ fontSize: 13, color: t.sub }}>{k}</span>
<span style={{ fontSize: 13, fontWeight: 700, color: t.text }}>{v}</span>
</div>
))}
<div style={{ marginTop: 14 }}>
style=
<div style={{ display: "flex", alignItems: "center", justifyContent: "space-bet
<div style={{ fontWeight: 800, color: "#16a34a", fontSize: 13 }}> Resept</d
<div style={{ display: "flex", gap: 6 }}>
<button onClick={handleAIRecipeForDetail} disabled={aiLoadingDetail} {aiLoadingDetail ? " " : " AI"}
</button>
<button onClick={() => { setEditingRecipe(true); setEditRecipeText(detail.r
Redaktə
</button>
</div>
</div>
{editingRecipe ? (
<div>
<textarea value={editRecipeText} onChange={e => setEditRecipeText(e.target.
<div style={{ display: "flex", gap: 8 }}>
<button onClick={saveEditedRecipe} style={{ flex: 1, background: "#16a34a
<button onClick={() => setEditingRecipe(false)} style={{ flex: 1, backgro
</div>
</div>
) : (
<div style={{ background: t.recipeBlock, borderRadius: 12, padding: 12, fontS
{detail.recipe}
</div>
)}
</div>
<button onClick={() => deleteFood(detail.id)} style={{ width: "100%", background:
<button onClick={() => { setDetail(null); setEditingRecipe(false); }} style={{ wi
</div>
</div>
)}
</div>
);
