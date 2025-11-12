import 'dotenv/config'
import fs from 'fs'
import path from 'path'
import express from 'express'
import session from 'express-session'
import bodyParser from 'body-parser'
import helmet from 'helmet'
import rateLimit from 'express-rate-limit'
import { fileURLToPath } from 'url'
import {
  Client,
  GatewayIntentBits,
  ChannelType,
  PermissionsBitField,
  EmbedBuilder,
  ActionRowBuilder,
  ButtonBuilder,
  ButtonStyle
} from 'discord.js'

const __filename = fileURLToPath(import.meta.url)
const __dirname  = path.dirname(__filename)
const app = express()
app.use(bodyParser.json())
app.use(bodyParser.urlencoded({ extended: true }))
app.use(session({ secret: process.env.SESSION_SECRET || 'v2dev_secret', resave: false, saveUninitialized: true }))
app.use(helmet({ contentSecurityPolicy: false }))
app.use(rateLimit({ windowMs: 60 * 1000, max: 120 }))

const STORE_PATH = path.join(__dirname, 'store.json')
if (!fs.existsSync(STORE_PATH)) fs.writeFileSync(STORE_PATH, JSON.stringify({ products: [], orders: [] }, null, 2))
const readStore  = () => JSON.parse(fs.readFileSync(STORE_PATH, 'utf-8'))
const writeStore = (data) => fs.writeFileSync(STORE_PATH, JSON.stringify(data, null, 2))

const boot = readStore()
if (!boot.products || boot.products.length === 0) {
  boot.products = [
    { id: 1, name: 'V2 – AntiCrash', description: 'حماية فايف إم متقدمة', priceCents: 1500, imageUrl: 'https://picsum.photos/seed/v2-1/1200/800' },
    { id: 2, name: 'V2 – Bot Manager', description: 'بوت إدارة متجر ديسكورد', priceCents: 2000, imageUrl: 'https://picsum.photos/seed/v2-2/1200/800' },
    { id: 3, name: 'V2 – Premium Script', description: 'سكربت حصري بخيارات كاملة', priceCents: 5000, imageUrl: 'https://picsum.photos/seed/v2-3/1200/800' }
  ]
  writeStore(boot)
}

const BOT = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    GatewayIntentBits.GuildMembers,
    GatewayIntentBits.GuildMessages
  ]
})
const GUILD_ID           = process.env.DISCORD_GUILD_ID
const TICKET_CATEGORY_ID = process.env.DISCORD_TICKET_CATEGORY_ID
const INVITE_URL         = process.env.DISCORD_INVITE || ''
const STAFF_ROLE_IDS     = (process.env.STAFF_ROLE_IDS || '').split(',').map(s=>s.trim()).filter(Boolean)

let memberCache = new Map()
async function warmMembers(){
  try {
    const guild = await BOT.guilds.fetch(GUILD_ID)
    const list = await guild.members.fetch({ withPresences: false })
    memberCache = new Map(list.map(m => [m.user.id, m]))
  } catch {}
}
BOT.once('ready', warmMembers)
setInterval(warmMembers, 10*60*1000)

const page = (title, body, active = '') => `<!doctype html>
<html lang="ar" dir="rtl">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>${title}</title>
<link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' width='128' height='128'><rect rx='24' width='128' height='128' fill='%236ee7ff'/><text x='50%' y='54%' dominant-baseline='middle' text-anchor='middle' font-size='64' font-family='Arial' fill='%230b0d10'>V2</text></svg>">
<meta name="description" content="v2 dev — سكربتات وبوتات حصرية مع تكامل ديسكورد وتذاكر تلقائية.">
<meta property="og:title" content="v2 dev — متجر حصري">
<meta property="og:description" content="سكربتات وبوتات بجودة عالية وتكامل كامل مع ديسكورد.">
<meta property="og:image" content="https://picsum.photos/seed/v2-og/1200/630">
<meta property="og:type" content="website">
<style>
*{box-sizing:border-box}
:root{
  --bg:#0b0d10;--card:#0f1318;--muted:#9aa4af;--text:#e8eef6;--brand:#6ee7ff;--brand2:#a78bfa;--stroke:#1a2129;--ok:#10b981;--warn:#f59e0b
}
@media(prefers-color-scheme:light){
  :root{--bg:#f6f7fb;--card:#ffffff;--muted:#667085;--text:#0b0d10;--brand:#2563eb;--brand2:#7c3aed;--stroke:#e5e7eb}
}
body{margin:0;background:
radial-gradient(1200px 500px at 10% -10%, rgba(102,126,234,.25), transparent),
radial-gradient(1000px 500px at 100% 0%, rgba(118,75,162,.22), transparent),
linear-gradient(180deg, var(--bg), var(--bg));
color:var(--text);font-family:Inter,system-ui,Segoe UI,Arial;letter-spacing:.2px}
a{text-decoration:none;color:inherit}
.nav{position:sticky;top:0;z-index:50;backdrop-filter:saturate(150%) blur(8px);background:color-mix(in oklab, var(--bg) 70%, transparent)}
.container{max-width:1200px;margin:0 auto;padding:16px}
.brand{display:flex;align-items:center;gap:10px;font-weight:900}
.logo{width:34px;height:34px;border-radius:10px;background:conic-gradient(from 210deg,var(--brand),var(--brand2));box-shadow:0 0 30px color-mix(in oklab,var(--brand) 30%, transparent)}
.navbar{display:flex;justify-content:space-between;align-items:center;border-bottom:1px solid var(--stroke)}
.navlinks{display:flex;gap:14px;align-items:center}
.link{padding:8px 12px;border-radius:10px}
.link.active{background:color-mix(in oklab,var(--brand) 14%, transparent);color:var(--text)}
.btn{padding:10px 14px;border-radius:12px;border:1px solid var(--stroke);background:linear-gradient(180deg,color-mix(in oklab,var(--card) 94%, transparent),color-mix(in oklab,var(--card) 100%, transparent));color:var(--text);cursor:pointer;transition:transform .15s, box-shadow .15s}
.btn:hover{transform:translateY(-2px);box-shadow:0 10px 20px rgba(0,0,0,.15)}
.btn.p{border:0;background:linear-gradient(135deg,var(--brand),var(--brand2));color:#0b0d10;font-weight:700}
.hero{padding:60px 16px}
.hero-wrap{display:grid;grid-template-columns:1.2fr .9fr;gap:24px;align-items:center}
.hero h1{font-size:44px;line-height:1.1;margin:0 0 10px}
.hero p{color:var(--muted);margin:0 0 18px;font-size:18px}
.hero-card{background:linear-gradient(180deg,color-mix(in oklab,var(--card) 80%, transparent),color-mix(in oklab,var(--card) 96%, transparent));border:1px solid var(--stroke);border-radius:20px;overflow:hidden}
.mock{aspect-ratio:16/10;background:
radial-gradient(800px 300px at 60% -30%, color-mix(in oklab,var(--brand) 20%, transparent), transparent),
radial-gradient(600px 300px at 0% 0%, color-mix(in oklab,var(--brand2) 20%, transparent), transparent),
url('https://picsum.photos/seed/v2-hero/1200/800') center/cover;border-bottom:1px solid var(--stroke)}
.mock-bar{display:flex;gap:8px;padding:10px;border-bottom:1px solid var(--stroke);background:color-mix(in oklab,var(--card) 92%, transparent)}
.dot{width:10px;height:10px;border-radius:999px}
.grid{display:grid;gap:16px;grid-template-columns:repeat(auto-fill,minmax(260px,1fr))}
.card{background:linear-gradient(180deg,color-mix(in oklab,var(--card) 88%, transparent),color-mix(in oklab,var(--card) 100%, transparent));border:1px solid var(--stroke);border-radius:18px;overflow:hidden;transition:transform .18s, box-shadow .18s}
.card:hover{transform:translateY(-4px);box-shadow:0 20px 40px rgba(0,0,0,.25)}
.card .img{height:180px;background:#0b0d10}
.b{padding:14px}
.price{display:inline-flex;gap:8px;align-items:center;background:color-mix(in oklab,var(--brand) 12%, transparent);border:1px solid color-mix(in oklab,var(--brand) 28%, var(--stroke));border-radius:999px;padding:6px 10px;margin:6px 0;color:color-mix(in oklab,var(--brand) 90%, var(--text))}
.sec{padding:40px 16px}
.kpis{display:grid;grid-template-columns:repeat(4,1fr);gap:12px}
.kpis>div{background:linear-gradient(180deg,color-mix(in oklab,var(--card) 88%, transparent),color-mix(in oklab,var(--card) 100%, transparent));border:1px solid var(--stroke);border-radius:16px;padding:16px;text-align:center}
.kpis h3{margin:6px 0 0}
.badge{display:inline-block;border:1px solid var(--stroke);border-radius:999px;padding:4px 10px;color:var(--muted)}
.footer{padding:28px 16px;border-top:1px solid var(--stroke);color:var(--muted);text-align:center}
.toast{position:fixed;bottom:18px;left:50%;transform:translateX(-50%);background:color-mix(in oklab,var(--card) 95%, transparent);border:1px solid var(--stroke);color:var(--text);padding:10px 14px;border-radius:999px;display:none;z-index:60}
.cartpin{position:relative}
.cartpin b{position:absolute;top:-8px;inset-inline-end:-10px;background:linear-gradient(135deg,var(--brand),var(--brand2));color:#0b0d10;border-radius:999px;padding:2px 8px;font-size:12px;border:1px solid var(--stroke)}
@media(max-width:980px){.hero-wrap{grid-template-columns:1fr;}.kpis{grid-template-columns:repeat(2,1fr)}.hero h1{font-size:34px}}
@keyframes floaty{0%{transform:translateY(0)}50%{transform:translateY(-6px)}100%{transform:translateY(0)}}
.floaty{animation:floaty 6s ease-in-out infinite}
</style>
</head>
<body>
<header class="nav">
  <div class="container navbar">
    <div class="brand"><div class="logo"></div><span>v2 dev</span></div>
    <nav class="navlinks">
      <a class="link ${active==='home'?'active':''}" href="/">الرئيسية</a>
      <a class="link" href="#features">لماذا نحن</a>
      <a class="link" href="#store">المتجر</a>
      <a class="link cartpin" href="/cart">السلة <b id="cartN" style="display:none">0</b></a>
    </nav>
  </div>
</header>
${body}
<div id="toast" class="toast"></div>
<script>
function cartCount(){try{const c=JSON.parse(localStorage.getItem('cart')||'[]');const n=c.reduce((s,i)=>s+(i.qty||0),0);const el=document.getElementById('cartN');if(!el)return;n>0?(el.style.display='inline-block',el.textContent=n):(el.style.display='none')}catch{}}
function toast(m){const t=document.getElementById('toast');t.textContent=m;t.style.display='block';setTimeout(()=>t.style.display='none',1700)}
cartCount()
</script>
</body></html>`

app.get('/', (req,res)=>{
  const { products } = readStore()
  const cards = products.map(p=>`
    <div class="card">
      <div class="img" style="background:url('${p.imageUrl}') center/cover"></div>
      <div class="b">
        <div style="display:flex;justify-content:space-between;align-items:center">
          <h3 style="margin:0;font-size:18px">${p.name}</h3>
          <span class="badge">جاهز للتسليم</span>
        </div>
        <div class="price">USD ${(p.priceCents/100).toFixed(2)}</div>
        <p style="color:var(--muted);margin:6px 0 10px">${p.description||''}</p>
        <button class="btn p" onclick='add(${p.id},"${p.name.replace(/"/g,'\\"')}",${p.priceCents})'>أضف للسلة</button>
      </div>
    </div>`).join('')

  res.send(page('v2 dev – متجر حصري', `
    <section class="hero">
      <div class="container hero-wrap">
        <div>
          <span class="badge">حصري</span>
          <h1>سكربتات وبوتات <span style="background:linear-gradient(135deg,var(--brand),var(--brand2));-webkit-background-clip:text;background-clip:text;color:transparent">v2 dev</span> بمعايير برو</h1>
          <p>تصاميم نظيفة، أداء قوي، ودمج مباشر مع ديسكورد. كل ما تحتاجه لتبيع وتدير طلباتك بسهولة.</p>
          <div style="display:flex;gap:10px;flex-wrap:wrap">
            <a class="btn p" href="#store">تصفح المنتجات</a>
            <a class="btn" href="/cart">السلة</a>
          </div>
        </div>
        <div class="hero-card floaty">
          <div class="mock-bar"><span class="dot" style="background:#ef4444"></span><span class="dot" style="background:#f59e0b"></span><span class="dot" style="background:#10b981"></span></div>
          <div class="mock"></div>
        </div>
      </div>
    </section>

    <section id="features" class="sec">
      <div class="container">
        <div class="kpis">
          <div><div class="badge">جودة</div><h3>أداء مستقر</h3><p style="color:var(--muted)">كود نظيف وعالي الاعتمادية.</p></div>
          <div><div class="badge">سهولة</div><h3>تذكرة تلقائية</h3><p style="color:var(--muted)">فتح تكت لحظي بعد الشراء.</p></div>
          <div><div class="badge">تصميم</div><h3>واجهة أنيقة</h3><p style="color:var(--muted)">تصميم حديث بخفة عالية.</p></div>
          <div><div class="badge">دعم</div><h3>متابعة الطلب</h3><p style="color:var(--muted)">إدارة الطلبات من داخل ديسكورد.</p></div>
        </div>
      </div>
    </section>

    <section id="store" class="sec">
      <div class="container">
        <div style="display:flex;justify-content:space-between;align-items:end;gap:10px;margin-bottom:10px">
          <h2 style="margin:0">المنتجات</h2>
          <span class="badge">تحديثات مستمرة</span>
        </div>
        <div class="grid">${cards}</div>
      </div>
    </section>

    <section class="sec">
      <div class="container" style="text-align:center">
        <h3 style="margin:0 0 8px">جاهز تبدأ؟</h3>
        <p style="color:var(--muted)">اختر منتجك وأكمل الطلب، والبوت يفتح لك تذكرتك مباشرة.</p>
        <a class="btn p" href="/cart">اذهب للسلة</a>
      </div>
    </section>

    <footer class="footer">© v2 dev</footer>

    <script>
      function add(id,name,priceCents){
        const c = JSON.parse(localStorage.getItem('cart')||'[]');
        const i = c.findIndex(x=>x.id===id);
        if(i>-1) c[i].qty+=1; else c.push({id,name,priceCents,qty:1});
        localStorage.setItem('cart', JSON.stringify(c));
        cartCount(); toast('أُضيف للسلة');
        setTimeout(()=>{ location.href='/cart' }, 350);
      }
    </script>
  `, 'home'))
})

app.get('/cart',(req,res)=>{
  res.send(page('السلة – v2 dev', `
    <section class="sec">
      <div class="container" style="max-width:920px">
        <h2 style="margin:0 0 10px">سلة المشتريات</h2>
        <div id="cart"></div>
        <div style="display:grid;grid-template-columns:2fr 1fr;gap:16px;margin-top:14px">
          <div>
            <label class="badge" style="display:inline-block;margin-bottom:8px">اكتب معرفك في ديسكورد</label>
            <input id="discord" class="input" placeholder="ID رقمي، منشن، رابط بروفايل، أو اسمك داخل السيرفر">
            <div style="color:var(--muted);font-size:13px;margin-top:6px">يفضّل ID رقمي لضمان المنشن والصلاحيات مباشرة.</div>
          </div>
          <div style="display:flex;gap:10px;align-items:end;justify-content:flex-end">
            <a class="btn" href="/">متابعة التسوق</a>
            <button class="btn p" onclick="pay()">شراء الآن</button>
          </div>
        </div>
      </div>
    </section>
    <script>
      function render(){
        const c=JSON.parse(localStorage.getItem('cart')||'[]'); let t=0;
        if(!c.length){document.getElementById('cart').innerHTML='<div class="card" style="padding:18px;text-align:center">السلة فارغة</div>';return}
        const rows=c.map(it=>{t+=it.priceCents*it.qty;return '<tr><td style="padding:12px 8px">'+it.name+'</td><td style="padding:12px 8px">'+it.qty+'</td><td style="padding:12px 8px">USD '+(it.priceCents/100).toFixed(2)+'</td></tr>'}).join('');
        document.getElementById('cart').innerHTML='<div class="card" style="overflow:hidden"><div class="b" style="padding:0"><table style="width:100%;border-collapse:collapse"><thead><tr style="background:color-mix(in oklab,var(--card) 94%, transparent)"><th style="text-align:start;padding:12px 8px;border-bottom:1px solid var(--stroke)">المنتج</th><th style="text-align:start;padding:12px 8px;border-bottom:1px solid var(--stroke)">الكمية</th><th style="text-align:start;padding:12px 8px;border-bottom:1px solid var(--stroke)">السعر</th></tr></thead><tbody>'+rows+'</tbody></table></div></div><div style="display:flex;justify-content:flex-end;margin-top:10px"><div class="price" style="font-weight:700">الإجمالي: USD '+(t/100).toFixed(2)+'</div></div>';
      }
      async function pay(){
        const c=JSON.parse(localStorage.getItem('cart')||'[]');
        const discord = document.getElementById('discord').value.trim();
        if(!c.length){toast('السلة فارغة');return}
        const r=await fetch('/checkout',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify({items:c, discord})});
        const j=await r.json(); if(j.redirect){localStorage.removeItem('cart'); cartCount(); location.href=j.redirect;}
      }
      render();
    </script>
  `))
})

app.post('/checkout', async (req,res)=>{
  const items = req.body.items||[]
  const discordInput = (req.body.discord||'').trim()
  if (!Array.isArray(items) || items.length === 0) return res.status(400).json({ error: 'السلة فارغة' })
  if (typeof discordInput !== 'string' || discordInput.length > 120) return res.status(400).json({ error: 'قيمة معرف الديسكورد غير صالحة' })
  const totalCents = items.reduce((s,i)=>s+i.priceCents*i.qty,0)
  const store = readStore()
  const id = (store.orders.at(-1)?.id||0)+1
  const order = { id, userId:null, username:discordInput||null, items, totalCents, ts:Date.now(), ticketId:null }
  store.orders.push(order); writeStore(store)
  const userId = await resolveUserId(discordInput).catch(()=>null)
  const chId = await openTicket({ order, userId })
  const idx = store.orders.findIndex(o=>o.id===id)
  if (idx>-1){ store.orders[idx].ticketId = chId; writeStore(store) }
  return res.json({ redirect: `/thanks?g=${encodeURIComponent(GUILD_ID)}&c=${encodeURIComponent(chId)}` })
})

app.get('/thanks',(req,res)=>{
  const g = req.query.g, c = req.query.c
  const channelLink = (g && c) ? `https://discord.com/channels/${g}/${c}` : '#'
  res.send(page('تم فتح تذكرتك', `
    <section class="sec">
      <div class="container" style="text-align:center;max-width:720px">
        <div class="card" style="padding:28px">
          <div style="font-size:22px;margin-bottom:6px">تم فتح تذكرتك 🎟️</div>
          <p style="color:var(--muted)">استخدم الأزرار أدناه للدخول للسيرفر ثم الذهاب إلى قناة تذكرتك مباشرة.</p>
          <div style="display:flex;gap:12px;justify-content:center;flex-wrap:wrap;margin-top:10px">
            ${INVITE_URL ? `<a class="btn" href="${INVITE_URL}" target="_blank" rel="noopener">دعوة السيرفر</a>` : ``}
            <a class="btn p" href="${channelLink}" target="_blank" rel="noopener">اذهب إلى التذكرة</a>
          </div>
        </div>
      </div>
    </section>
  `))
})

async function resolveUserId(input){
  const raw = (input || '').trim()
  if (/^\d{16,20}$/.test(raw)) return raw
  const mention = raw.match(/^<@!?(\d{16,20})>$/)
  if (mention) return mention[1]
  try {
    const u = new URL(raw)
    const parts = u.pathname.split('/').filter(Boolean)
    const idx = parts.findIndex(p => p.toLowerCase()==='users')
    const id = idx > -1 ? parts[idx+1] : null
    if (id && /^\d{16,20}$/.test(id)) return id
  } catch {}
  if (!raw) return null
  for (const [,m] of memberCache) {
    const dn = (m.displayName||'').toLowerCase()
    const un = (m.user?.username||'').toLowerCase()
    if (dn.includes(raw.toLowerCase()) || un.includes(raw.toLowerCase())) return m.id
  }
  const guild = await BOT.guilds.fetch(GUILD_ID)
  const res = await guild.members.search({ query: raw, limit: 1 }).catch(()=>null)
  const member = res?.first?.() || null
  if (member) return member.id
  return null
}

async function openTicket({ order, userId }){
  const guild = await BOT.guilds.fetch(GUILD_ID)
  if (userId) { await guild.members.fetch(userId).catch(()=>null) }
  const overwrites = [{ id: guild.roles.everyone, deny:  [PermissionsBitField.Flags.ViewChannel] }]
  if (userId) overwrites.push({ id: userId, allow: [PermissionsBitField.Flags.ViewChannel, PermissionsBitField.Flags.SendMessages] })
  for (const rid of STAFF_ROLE_IDS) if (rid) overwrites.push({ id: rid, allow: [PermissionsBitField.Flags.ViewChannel, PermissionsBitField.Flags.SendMessages] })
  const ch = await guild.channels.create({
    name: `order-${order.id}`,
    type: ChannelType.GuildText,
    parent: TICKET_CATEGORY_ID,
    permissionOverwrites: overwrites
  })
  const lines = order.items.map(i=>`• ${i.name} × ${i.qty}`).join('\n') || '—'
  const embed = new EmbedBuilder()
    .setTitle(`طلب #${order.id}`)
    .setDescription(`المشتري: ${userId ? `<@${userId}>` : (order.username||'غير محدد')}\nالإجمالي: USD ${(order.totalCents/100).toFixed(2)}`)
    .addFields({ name:'المنتجات', value: lines })
    .setColor(0x2b2d31)
  const row = new ActionRowBuilder().addComponents(
    new ButtonBuilder().setCustomId(`ticket_close_${ch.id}`).setStyle(ButtonStyle.Danger).setLabel('إغلاق التذكرة')
  )
  await ch.send({ content: userId ? `<@${userId}> تم فتح تذكرتك` : 'تم فتح تذكرة للطلب (اكتب معرفك هنا)', embeds: [embed], components: [row] })
  return ch.id
}

BOT.on('interactionCreate', async (i) => {
  if (!i.isButton()) return
  if (!i.customId.startsWith('ticket_close_')) return
  const chId = i.customId.split('ticket_close_')[1]
  if (i.channelId !== chId) return i.reply({ content: 'هذه الأزرار تخص التذكرة الحالية فقط.', ephemeral: true })
  await i.reply({ content: 'سيتم إغلاق التذكرة خلال ثوانٍ...', ephemeral: true })
  setTimeout(async () => { try { await i.channel.delete('Closed by button') } catch(_){} }, 1500)
})

app.get('/health',(_,res)=>res.json({ok:true}))
const PORT = process.env.PORT || 3000
async function start(){
  await BOT.login(process.env.DISCORD_BOT_TOKEN)
  app.listen(PORT,()=>console.log(`v2 dev on http://localhost:${PORT}`))
}
start()
