<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>公众号版 · App Store 地区总表生成器</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#eef1f5;color:#1f2328;
  font:14px/1.6 -apple-system,BlinkMacSystemFont,"Segoe UI","PingFang SC","Hiragino Sans GB","Microsoft YaHei",sans-serif}
header{position:sticky;top:0;z-index:10;background:#fff;border-bottom:1px solid #e5e8ec;padding:12px 16px}
.wrap{max-width:780px;margin:0 auto}
h1{font-size:15px;margin:0 0 10px;font-weight:600}
h1 small{font-weight:400;color:#8a8f99;font-size:12px;margin-left:8px}
.row{display:flex;gap:6px;flex-wrap:wrap;align-items:center;margin-bottom:6px}
.row:last-child{margin-bottom:0}
.lbl{font-size:12px;color:#8a8f99;margin-right:2px;flex:0 0 auto}
button{font:inherit;cursor:pointer;border-radius:8px;border:1px solid #d0d7de;
  background:#fff;color:#1f2328;padding:6px 11px;transition:.15s;font-size:13px}
button:hover{background:#f6f8fa;border-color:#afb8c1}
button.on{background:#1f6feb;border-color:#1f6feb;color:#fff}
button.on:hover{background:#388bfd;border-color:#388bfd}
button.primary{background:#1a7f37;border-color:#1a7f37;color:#fff;font-weight:600}
button.primary:hover{background:#2da44e;border-color:#2da44e}
.seg{display:inline-flex;border:1px solid #d0d7de;border-radius:8px;overflow:hidden;background:#fff;flex-wrap:wrap}
.seg button{border:0;border-radius:0;background:transparent;color:#57606a;padding:6px 10px}
.seg button.on{background:#1f6feb;color:#fff}
.seg button:hover:not(.on){background:#f6f8fa}
main{padding:16px}
#preview{max-width:420px;margin:0 auto;background:#fff;padding:20px 16px;border-radius:12px;
  box-shadow:0 2px 12px rgba(0,0,0,.06)}
.hint{max-width:420px;margin:0 auto 12px;font-size:12px;color:#8a8f99;text-align:center;line-height:1.7}
#toast{position:fixed;left:50%;bottom:24px;transform:translate(-50%,20px);background:#1a7f37;color:#fff;
  padding:9px 18px;border-radius:999px;font-size:13px;opacity:0;pointer-events:none;transition:.25s;z-index:99;
  box-shadow:0 6px 22px rgba(0,0,0,.2)}
#toast.on{opacity:1;transform:translate(-50%,0)}
</style>
</head>
<body>

<header>
  <div class="wrap">
    <h1>公众号版生成器 <small>链接单独成行 · 长按即可整段复制</small></h1>

    <div class="row">
      <span class="lbl">范围</span>
      <div class="seg" id="scope">
        <button data-scope="hot" class="on">热门 30</button>
        <button data-scope="all">全部 175</button>
        <button data-scope="asia">亚太</button>
        <button data-scope="europe">欧洲</button>
        <button data-scope="africa">非洲</button>
        <button data-scope="na">北美</button>
        <button data-scope="ca">中美</button>
        <button data-scope="carib">加勒比</button>
        <button data-scope="sa">南美</button>
        <button data-scope="oceania">大洋洲</button>
      </div>
    </div>

    <div class="row">
      <span class="lbl">视图</span>
      <div class="seg" id="view">
        <button data-view="card" class="on">卡片</button>
        <button data-view="plain">纯链接列表</button>
      </div>
      <span class="lbl">格式</span>
      <div class="seg" id="fmt">
        <button data-fmt="web" class="on">https</button>
        <button data-fmt="itms">itms-apps</button>
      </div>
      <button class="primary" id="copyRich">复制富文本</button>
      <button id="copyPlain">复制纯链接</button>
      <button id="downloadBtn">下载 HTML</button>
    </div>
  </div>
</header>

<main>
  <p class="hint">
    点「复制富文本」→ 公众号编辑器 <b>Ctrl/⌘+V</b> 粘贴<br>
    读者端：<b>长按链接那一行</b>即可整段选中复制；「纯链接列表」模式一行一条，适合批量存到备忘录
  </p>
  <div id="preview"></div>
</main>

<div id="toast"></div>

<script>
/* ============ 中文名 ============ */
const ZH = {
  ae:'阿联酋', af:'阿富汗', ag:'安提瓜和巴布达', ai:'安圭拉', al:'阿尔巴尼亚', am:'亚美尼亚',
  ao:'安哥拉', ar:'阿根廷', at:'奥地利', au:'澳大利亚', az:'阿塞拜疆',
  ba:'波黑', bb:'巴巴多斯', be:'比利时', bf:'布基纳法索', bg:'保加利亚', bh:'巴林',
  bj:'贝宁', bm:'百慕大', bn:'文莱', bo:'玻利维亚', br:'巴西', bs:'巴哈马', bt:'不丹',
  bw:'博茨瓦纳', by:'白俄罗斯', bz:'伯利兹',
  ca:'加拿大', cd:'刚果（金）', cg:'刚果（布）', ch:'瑞士', ci:'科特迪瓦', cl:'智利',
  cm:'喀麦隆', cn:'中国', co:'哥伦比亚', cr:'哥斯达黎加', cv:'佛得角', cy:'塞浦路斯', cz:'捷克',
  de:'德国', dk:'丹麦', dm:'多米尼克', do:'多米尼加', dz:'阿尔及利亚',
  ec:'厄瓜多尔', ee:'爱沙尼亚', eg:'埃及', es:'西班牙',
  fi:'芬兰', fj:'斐济', fm:'密克罗尼西亚', fr:'法国',
  ga:'加蓬', gb:'英国', gd:'格林纳达', ge:'格鲁吉亚', gh:'加纳', gm:'冈比亚', gr:'希腊',
  gt:'危地马拉', gw:'几内亚比绍', gy:'圭亚那',
  hk:'中国香港', hn:'洪都拉斯', hr:'克罗地亚', hu:'匈牙利',
  id:'印度尼西亚', ie:'爱尔兰', il:'以色列', in:'印度', iq:'伊拉克', is:'冰岛', it:'意大利',
  jm:'牙买加', jo:'约旦', jp:'日本',
  ke:'肯尼亚', kg:'吉尔吉斯斯坦', kh:'柬埔寨', kn:'圣基茨和尼维斯', kr:'韩国', kw:'科威特',
  ky:'开曼群岛', kz:'哈萨克斯坦',
  la:'老挝', lb:'黎巴嫩', lc:'圣卢西亚', lk:'斯里兰卡', lr:'利比里亚', lt:'立陶宛',
  lu:'卢森堡', lv:'拉脱维亚', ly:'利比亚',
  ma:'摩洛哥', md:'摩尔多瓦', me:'黑山', mg:'马达加斯加', mk:'北马其顿', ml:'马里',
  mm:'缅甸', mn:'蒙古', mo:'中国澳门', mr:'毛里塔尼亚', ms:'蒙特塞拉特', mt:'马耳他',
  mu:'毛里求斯', mv:'马尔代夫', mw:'马拉维', mx:'墨西哥', my:'马来西亚', mz:'莫桑比克',
  na:'纳米比亚', ne:'尼日尔', ng:'尼日利亚', ni:'尼加拉瓜', nl:'荷兰', no:'挪威',
  np:'尼泊尔', nr:'瑙鲁', nz:'新西兰', om:'阿曼',
  pa:'巴拿马', pe:'秘鲁', pg:'巴布亚新几内亚', ph:'菲律宾', pk:'巴基斯坦', pl:'波兰',
  pt:'葡萄牙', pw:'帕劳', py:'巴拉圭', qa:'卡塔尔',
  ro:'罗马尼亚', rs:'塞尔维亚', ru:'俄罗斯', rw:'卢旺达',
  sa:'沙特阿拉伯', sb:'所罗门群岛', sc:'塞舌尔', se:'瑞典', sg:'新加坡', si:'斯洛文尼亚',
  sk:'斯洛伐克', sl:'塞拉利昂', sn:'塞内加尔', sr:'苏里南', st:'圣多美和普林西比',
  sv:'萨尔瓦多', sz:'斯威士兰',
  tc:'特克斯和凯科斯', td:'乍得', th:'泰国', tj:'塔吉克斯坦', tm:'土库曼斯坦', tn:'突尼斯',
  to:'汤加', tr:'土耳其', tt:'特立尼达和多巴哥', tw:'中国台湾', tz:'坦桑尼亚',
  ua:'乌克兰', ug:'乌干达', us:'美国', uy:'乌拉圭', uz:'乌兹别克斯坦',
  vc:'圣文森特', ve:'委内瑞拉', vg:'英属维尔京群岛', vn:'越南', vu:'瓦努阿图',
  xk:'科索沃', ye:'也门', za:'南非', zm:'赞比亚', zw:'津巴布韦'
};

/* ============ 数据：cc | en | dsf ============ */
const DATA = [
  {key:'asia', short:'亚太', label:'亚洲 / 亚太', color:'#f0883e', rows:`
ae|UAE|143481
af|Afghanistan|143610
am|Armenia|143524
az|Azerbaijan|143568
bh|Bahrain|143559
bn|Brunei|143560
bt|Bhutan|143577
cn|China|143465
ge|Georgia|143615
hk|Hong Kong|143463
id|Indonesia|143476
il|Israel|143491
in|India|143467
iq|Iraq|143617
jo|Jordan|143528
jp|Japan|143462
kg|Kyrgyzstan|143586
kh|Cambodia|143579
kr|South Korea|143466
kw|Kuwait|143493
kz|Kazakhstan|143517
la|Laos|143587
lb|Lebanon|143497
lk|Sri Lanka|143486
mm|Myanmar (Burma)|143570
mn|Mongolia|143592
mo|Macao|143515
mv|Maldives|143488
my|Malaysia|143473
np|Nepal|143484
om|Oman|143562
ph|Philippines|143474
pk|Pakistan|143477
qa|Qatar|143498
sa|Saudi Arabia|143479
sg|Singapore|143464
th|Thailand|143475
tj|Tajikistan|143603
tm|Turkmenistan|143604
tr|Türkiye|143480
tw|Taiwan|143470
uz|Uzbekistan|143566
vn|Vietnam|143471
ye|Yemen|143571`},

  {key:'europe', short:'欧洲', label:'欧洲', color:'#4f9cf9', rows:`
al|Albania|143575
at|Austria|143445
ba|Bosnia & Herzegovina|143612
be|Belgium|143446
bg|Bulgaria|143526
by|Belarus|143565
ch|Switzerland|143459
cy|Cyprus|143557
cz|Czechia|143489
de|Germany|143443
dk|Denmark|143458
ee|Estonia|143518
es|Spain|143454
fi|Finland|143447
fr|France|143442
gb|United Kingdom|143444
gr|Greece|143448
hr|Croatia|143494
hu|Hungary|143482
ie|Ireland|143449
is|Iceland|143558
it|Italy|143450
lt|Lithuania|143520
lu|Luxembourg|143451
lv|Latvia|143519
md|Moldova|143523
me|Montenegro|143619
mk|North Macedonia|143530
mt|Malta|143521
nl|Netherlands|143452
no|Norway|143457
pl|Poland|143478
pt|Portugal|143453
ro|Romania|143487
rs|Serbia|143500
ru|Russia|143469
se|Sweden|143456
si|Slovenia|143499
sk|Slovakia|143496
ua|Ukraine|143492
xk|Kosovo|143624`},

  {key:'africa', short:'非洲', label:'非洲', color:'#e3b341', rows:`
ao|Angola|143564
bf|Burkina Faso|143578
bj|Benin|143576
bw|Botswana|143525
cd|Congo - Kinshasa|143613
cg|Congo - Brazzaville|143582
ci|Côte d’Ivoire|143527
cm|Cameroon|143574
cv|Cape Verde|143580
dz|Algeria|143563
eg|Egypt|143516
ga|Gabon|143614
gh|Ghana|143573
gm|Gambia|143584
gw|Guinea-Bissau|143585
ke|Kenya|143529
lr|Liberia|143588
ly|Libya|143567
ma|Morocco|143620
mg|Madagascar|143531
ml|Mali|143532
mr|Mauritania|143590
mu|Mauritius|143533
mw|Malawi|143589
mz|Mozambique|143593
na|Namibia|143594
ne|Niger|143534
ng|Nigeria|143561
rw|Rwanda|143621
sc|Seychelles|143599
sl|Sierra Leone|143600
sn|Senegal|143535
st|São Tomé & Príncipe|143598
sz|Eswatini|143602
td|Chad|143581
tn|Tunisia|143536
tz|Tanzania|143572
ug|Uganda|143537
za|South Africa|143472
zm|Zambia|143622
zw|Zimbabwe|143605`},

  {key:'na', short:'北美', label:'北美洲', color:'#a371f7', rows:`
bm|Bermuda|143542
ca|Canada|143455
mx|Mexico|143468
us|United States|143441`},

  {key:'ca', short:'中美', label:'中美洲', color:'#3fb950', rows:`
bz|Belize|143555
cr|Costa Rica|143495
gt|Guatemala|143504
hn|Honduras|143510
ni|Nicaragua|143512
pa|Panama|143485
sv|El Salvador|143506`},

  {key:'carib', short:'加勒比', label:'加勒比', color:'#db61a2', rows:`
ag|Antigua & Barbuda|143540
ai|Anguilla|143538
bb|Barbados|143541
bs|Bahamas|143539
dm|Dominica|143545
do|Dominican Rep.|143508
gd|Grenada|143546
jm|Jamaica|143511
kn|St. Kitts & Nevis|143548
ky|Cayman Islands|143544
lc|St. Lucia|143549
ms|Montserrat|143547
tc|Turks & Caicos|143552
tt|Trinidad & Tobago|143551
vc|St. Vincent|143550
vg|British Virgin Is.|143543`},

  {key:'sa', short:'南美', label:'南美洲', color:'#f85149', rows:`
ar|Argentina|143505
bo|Bolivia|143556
br|Brazil|143503
cl|Chile|143483
co|Colombia|143501
ec|Ecuador|143509
gy|Guyana|143553
pe|Peru|143507
py|Paraguay|143513
sr|Suriname|143554
uy|Uruguay|143514
ve|Venezuela|143502`},

  {key:'oceania', short:'大洋洲', label:'大洋洲', color:'#39c5cf', rows:`
au|Australia|143460
fj|Fiji|143583
fm|Micronesia|143591
nr|Nauru|143606
nz|New Zealand|143461
pg|Papua New Guinea|143597
pw|Palau|143595
sb|Solomon Islands|143601
to|Tonga|143608
vu|Vanuatu|143609`}
];

/* 热门（中国大陆用户常用） */
const HOT = ['us','hk','tw','jp','kr','sg','gb','ca','au','de','fr','th','my','mo','tr','in','ae','nz','ie','nl','ch','se','ru','br','mx','it','es','id','ph','vn'];

/* ============ 工具 ============ */
const esc = s => String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');

const ALL = [];
DATA.forEach(g=>{
  g.rows.trim().split('\n').forEach(r=>{
    const [cc,en,dsf] = r.split('|');
    ALL.push({ cc, en, dsf, zh: ZH[cc] || en, region: g.key, regionLabel: g.label, color: g.color });
  });
});

const webUrl  = cc => `https://apps.apple.com/${cc}/store`;
const itmsUrl = (cc, dsf) => `itms-apps://itunes.apple.com/WebObjects/MZStore.woa/wa/resetAndRedirect?dsf=${dsf}&cc=${cc}`;
const urlOf   = it => fmt === 'itms' ? itmsUrl(it.cc, it.dsf) : webUrl(it.cc);

/* ============ 状态 ============ */
let scope = 'hot';
let view  = 'card';
let fmt   = 'web';
let currentLinks = []; // {zh, en, cc, dsf, url}

/* ============ 渲染 ============ */
function toast(msg){
  const t = document.getElementById('toast');
  t.textContent = msg;
  t.classList.add('on');
  clearTimeout(toast._t);
  toast._t = setTimeout(()=>t.classList.remove('on'), 2000);
}

/* 卡片版：链接独占一行，蓝色，长按可整段选中 */
function cardItem(it){
  const url = urlOf(it);
  return `<section style="margin:0 0 8px 0;padding:9px 12px;background:#fafbfc;border:1px solid #ebeef5;border-left:3px solid ${it.color};border-radius:6px;">
    <section style="font-size:14px;font-weight:bold;color:#1a1a1a;line-height:1.45;">
      ${esc(it.zh)}
      <span style="font-weight:normal;color:#8a8f99;font-size:12px;"> ${esc(it.en)} · ${esc(it.cc)}</span>
    </section>
    <section style="font-size:13px;color:#1f6feb;margin-top:3px;line-height:1.65;word-break:break-all;">${esc(url)}</section>
  </section>`;
}

/* 纯链接版：国家名一行 + 链接一行，链接整段可长按选中 */
function plainItem(it){
  const url = urlOf(it);
  return `<section style="margin:0 0 2px 0;font-size:12px;color:#8a8f99;line-height:1.5;">${esc(it.zh)} ${esc(it.en)}</section>
<section style="margin:0 0 10px 0;font-size:14px;color:#1a1a1a;line-height:1.7;word-break:break-all;">${esc(url)}</section>`;
}

function buildGroup(label, color, items){
  let out = `<section style="margin:22px 0 10px 0;padding:0 0 0 10px;border-left:4px solid ${color};">
    <section style="font-size:16px;font-weight:bold;color:#1a1a1a;line-height:1.4;">
      ${esc(label)}
      <span style="font-size:12px;font-weight:normal;color:#8a8f99;background:#f2f3f5;padding:2px 9px;border-radius:10px;margin-left:8px;">${items.length}</span>
    </section>
  </section>`;
  items.forEach(it=>{ out += (view === 'plain' ? plainItem(it) : cardItem(it)); });
  return out;
}

function buildHeader(count){
  return `<section style="margin:0 0 16px 0;padding:18px 16px;background:linear-gradient(135deg,#1f6feb,#4f46e5);border-radius:12px;color:#fff;">
    <section style="font-size:20px;font-weight:bold;line-height:1.35;">App Store 地区商店总表</section>
    <section style="font-size:13px;opacity:.92;margin-top:6px;line-height:1.6;">${count} 个国家 / 地区 · 链接长按即可复制</section>
  </section>

  <section style="margin:0 0 16px 0;padding:12px 14px;background:#fff8e6;border:1px solid #ffe6a8;border-radius:8px;font-size:13px;color:#8a6d1f;line-height:1.75;">
    <b>怎么用</b><br>
    ① 复制下方链接到 Safari 打开，确认该区有没有你要的 App<br>
    ② iOS 上把 https 链接换成 itms-apps 链接可直接唤起 App Store<br>
    ③ 打开商店 ≠ 改 Apple ID 地区，真正转区见文末
  </section>`;
}

function buildFooter(){
  return `<section style="margin:20px 0 0 0;padding-top:14px;border-top:1px dashed #d0d7de;font-size:12px;color:#8a8f99;line-height:1.9;">
    <b>注意</b>：打开商店链接 ≠ 修改 Apple ID 地区。真正转区请访问<br>
    https://apps.apple.com/choose-country-region<br>
    或 https://appleid.apple.com/account/manage<br>
    <br>
    iOS 上更新 App 需登录下载时所用的 Apple ID，请勿频繁切换账号。
  </section>`;
}

function render(){
  let groups;
  currentLinks = [];

  if(scope === 'all'){
    groups = DATA.map(g=>({
      label: g.label, color: g.color,
      items: ALL.filter(it=>it.region === g.key)
    }));
  } else if(scope === 'hot'){
    const list = ALL.filter(it=>HOT.includes(it.cc));
    list.sort((a,b)=>HOT.indexOf(a.cc) - HOT.indexOf(b.cc));
    groups = [{ label:'热门地区 · 精选 30', color:'#1f6feb', items:list }];
  } else {
    const g = DATA.find(x=>x.key === scope);
    groups = [{ label:g.label, color:g.color, items: ALL.filter(it=>it.region === g.key) }];
  }

  const total = groups.reduce((s,g)=>s+g.items.length,0);
  groups.forEach(g=>g.items.forEach(it=>{
    currentLinks.push({ zh:it.zh, en:it.en, cc:it.cc, dsf:it.dsf, url:urlOf(it) });
  }));

  let html = buildHeader(total);
  groups.forEach(g=>{ html += buildGroup(g.label, g.color, g.items); });
  html += buildFooter();

  document.getElementById('preview').innerHTML = html;
}

/* ============ 复制富文本 ============ */
async function copyRich(){
  const el = document.getElementById('preview');
  try{
    if(navigator.clipboard && window.ClipboardItem){
      await navigator.clipboard.write([new ClipboardItem({
        'text/html': new Blob([el.innerHTML], {type:'text/html'}),
        'text/plain': new Blob([el.innerText], {type:'text/plain'})
      })]);
      toast('已复制富文本，去公众号编辑器粘贴');
      return;
    }
  }catch(e){}

  try{
    const range = document.createRange();
    range.selectNodeContents(el);
    const sel = window.getSelection();
    sel.removeAllRanges();
    sel.addRange(range);
    const ok = document.execCommand('copy');
    sel.removeAllRanges();
    toast(ok ? '已复制富文本，去公众号编辑器粘贴' : '复制失败，请手动全选');
  }catch(e){
    toast('复制失败，请手动全选复制');
  }
}

/* ============ 复制纯链接（一行一条，适合存备忘录 / 批量打开） ============ */
async function copyPlain(){
  if(!currentLinks.length){ toast('没有可复制的链接'); return; }
  const text = currentLinks.map(x=>x.url).join('\n');
  try{
    await navigator.clipboard.writeText(text);
    toast(`已复制 ${currentLinks.length} 条链接（一行一条）`);
  }catch(e){
    const ta = document.createElement('textarea');
    ta.value = text;
    ta.style.cssText = 'position:fixed;top:0;left:0;opacity:0';
    document.body.appendChild(ta);
    ta.select();
    try{ document.execCommand('copy'); }catch(_){}
    ta.remove();
    toast(`已复制 ${currentLinks.length} 条链接（一行一条）`);
  }
}

/* ============ 下载 HTML ============ */
function downloadHTML(){
  const html = `<!DOCTYPE html><html><head><meta charset="utf-8"><title>App Store 地区总表</title></head><body style="margin:0;padding:20px;background:#f5f5f5;"><div style="max-width:420px;margin:0 auto;background:#fff;padding:20px 16px;font-family:-apple-system,BlinkMacSystemFont,'PingFang SC','Microsoft YaHei',sans-serif;">${document.getElementById('preview').innerHTML}</div></body></html>`;
  const blob = new Blob([html], {type:'text/html;charset=utf-8'});
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = 'appstore-regions-wechat.html';
  a.click();
  setTimeout(()=>URL.revokeObjectURL(a.href), 1000);
  toast('已下载 HTML');
}

/* ============ 交互 ============ */
document.getElementById('scope').addEventListener('click', e=>{
  const b = e.target.closest('[data-scope]'); if(!b) return;
  scope = b.dataset.scope;
  document.querySelectorAll('#scope button').forEach(x=>x.classList.toggle('on', x===b));
  render();
});

document.getElementById('view').addEventListener('click', e=>{
  const b = e.target.closest('[data-view]'); if(!b) return;
  view = b.dataset.view;
  document.querySelectorAll('#view button').forEach(x=>x.classList.toggle('on', x===b));
  render();
});

document.getElementById('fmt').addEventListener('click', e=>{
  const b = e.target.closest('[data-fmt]'); if(!b) return;
  fmt = b.dataset.fmt;
  document.querySelectorAll('#fmt button').forEach(x=>x.classList.toggle('on', x===b));
  render();
});

document.getElementById('copyRich').addEventListener('click', copyRich);
document.getElementById('copyPlain').addEventListener('click', copyPlain);
document.getElementById('downloadBtn').addEventListener('click', downloadHTML);

render();
</script>
</body>
</html>
