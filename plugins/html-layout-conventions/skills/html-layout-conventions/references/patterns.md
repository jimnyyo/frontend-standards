# Worked patterns

Reference markup for the components that recur most. Copy the structure, rename the prefix to fit the component, keep the child abbreviations exactly as they are.

Contents:

1. Two-column sub page shell
2. GNB (global navigation)
3. Board list with mobile list fallback
4. Search box with absolutely positioned buttons
5. Card grid
6. Form rows
7. Numbered step / process list
8. Footer

---

## 1. Two-column sub page shell

```html
<div class="wrap">
  <header class="header">…</header>
  <nav class="gnb">…</nav>

  <div class="container">
    <div class="content">
      <h2 class="cont_tt">Notice</h2>
      …
    </div>
    <aside class="sidebox">
      <h3 class="side_tt">Information</h3>
      <nav class="side_mn"><a href="#" class="on">Notice</a><a href="#">Press</a></nav>
    </aside>
  </div>

  <footer class="footer">…</footer>
</div>
```

```css
.container { display: grid; grid-template-columns: 1fr 280px; gap: 40px; max-width: 1280px; margin: 0 auto; padding: 40px 20px; }
.side_tt { height: 120px; display: flex; align-items: center; padding: 0 24px; background: url('/img/bg_side.jpg') no-repeat center / cover; color: #fff; font-size: 22px; }
.side_mn { display: flex; flex-direction: column; }
.side_mn a { display: flex; align-items: center; height: 52px; padding: 0 24px; border-bottom: 1px solid var(--line); }
.side_mn a.on { color: var(--color-primary); font-weight: 700; }

@media (max-width: 1024px) {
  .container { grid-template-columns: 1fr; gap: 24px; }
  .container .sidebox { order: -1; }
}
```

One-column pages drop `.container` and `.sidebox` entirely and put sections straight under `.wrap`.

## 2. GNB

`nav > a` directly — no `ul > li > a`. The dropdown panel is a sibling of the trigger anchor inside the menu block.

```html
<nav class="gnb">
  <div class="gnb_mn">
    <a href="#">About</a>
    <div class="gnb_sub"><a href="#">Greeting</a><a href="#">History</a></div>
  </div>
  <div class="gnb_mn">
    <a href="#">Business</a>
    <div class="gnb_sub"><a href="#">Support</a><a href="#">Education</a></div>
  </div>
</nav>
```

```css
.gnb { display: flex; justify-content: center; gap: 48px; border-bottom: 1px solid var(--line); }
.gnb_mn > a { display: flex; align-items: center; height: 64px; font-size: 17px; font-weight: 600; }
.gnb_mn > a::after { content: ''; display: block; height: 3px; background: var(--color-primary); transform: scaleX(0); transition: .2s; }
.gnb_mn:hover > a::after { transform: scaleX(1); }
.gnb_sub { display: none; }
.gnb_mn:hover .gnb_sub { display: flex; flex-direction: column; position: absolute; }
```

## 3. Board list + mobile fallback

Semantic list, styled as a table row on desktop, reflowed to a stacked card below the breakpoint. Shared column defaults live on `.bd_item > *`.

```html
<ul class="bd_list">
  <li class="bd_item">
    <a href="#" class="bd_link">
      <span class="col_num">124</span>
      <span class="col_cat">Notice</span>
      <strong class="col_tt">2026 support program applications open</strong>
      <span class="col_dt">2026.08.21</span>
      <span class="col_vw">1,204</span>
    </a>
  </li>
</ul>
```

```css
.bd_list { border-top: 2px solid var(--color-text); }
.bd_item { border-bottom: 1px solid var(--line); }
.bd_link { display: grid; grid-template-columns: 90px 120px 1fr 130px 100px; align-items: center; }
.bd_item > * , .bd_link > * { padding: 20px 10px; font-size: 15px; color: var(--color-sub); text-align: center; }
.col_tt { padding: 20px 14px; font-size: 17px; color: var(--color-text); text-align: left; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
.bd_link:hover .col_tt { color: var(--color-primary); text-decoration: underline; }

@media (max-width: 768px) {
  .bd_link { grid-template-columns: auto 1fr; grid-template-areas: 'cat title' 'meta meta'; row-gap: 4px; padding: 16px 4px; }
  .bd_item .col_num, .bd_item .col_vw { display: none; }
  .bd_item .col_cat { grid-area: cat; padding: 0 10px 0 0; }
  .bd_item .col_tt { grid-area: title; padding: 0; white-space: normal; }
  .bd_item .col_dt { grid-area: meta; padding: 0; text-align: left; }
}
```

Note the whole row is one `<a>` — not one link per cell.

## 4. Search box

The two buttons' area is reserved by the parent's right padding, so the input text never slides under them.

```html
<div class="srch_box">
  <input type="text" class="srch_inp" placeholder="Enter a keyword">
  <button type="button" class="srch_clr"><span class="blind">Clear</span></button>
  <button type="submit" class="srch_btn"><span class="blind">Search</span></button>
</div>
```

```css
.srch_box { position: relative; padding: 0 85px 0 16px; border: 1px solid var(--btn-border); border-radius: 24px; background: #fff; }
.srch_inp { width: 100%; border: none; padding: 0; }
.srch_clr { position: absolute; top: 0; left: calc(100% - 82px); width: 32px; height: 48px; background: url('/img/ic_clear.svg') no-repeat center; }
.srch_btn { position: absolute; top: 0; right: 10px; width: 32px; height: 48px; background: url('/img/ic_search.svg') no-repeat center; }
```

`.srch_inp` overrides `border` and `padding` with one class each — possible only because the form defaults are `:where()`-wrapped.

## 5. Card grid

`a.card` directly; thumbnail rounded with `clip-path`, so no wrapper div.

```html
<div class="card_list">
  <a href="#" class="card">
    <img src="/img/thumb01.jpg" alt="" class="card_thumb">
    <span class="card_cat">Education</span>
    <strong class="card_tt">Practical data analysis course</strong>
    <span class="card_dt">2026.09.01 – 09.30</span>
  </a>
</div>
```

```css
.card_list { display: grid; grid-template-columns: repeat(3, 1fr); gap: 28px; }
.card { display: grid; grid-template-rows: auto auto auto auto; gap: 8px; }
.card_thumb { width: 100%; aspect-ratio: 16/10; object-fit: cover; clip-path: inset(0 round 12px); }
.card_cat { font-size: 13px; color: var(--color-primary); }
.card_tt { font-size: 18px; line-height: 1.4; }
.card_dt { font-size: 14px; color: var(--color-sub); }
.card:hover .card_tt { text-decoration: underline; }

@media (max-width: 768px) { .card_list { grid-template-columns: 1fr; gap: 20px; } }
```

## 6. Form rows

```html
<div class="frm_list">
  <div class="frm_row">
    <label class="frm_lb" for="nm">Name</label>
    <div class="frm_val"><input type="text" id="nm"></div>
  </div>
  <div class="frm_row">
    <span class="frm_lb">Type</span>
    <div class="frm_val">
      <label><input type="radio" name="tp"> Individual</label>
      <label><input type="radio" name="tp"> Organization</label>
    </div>
  </div>
</div>
```

```css
.frm_row { display: grid; grid-template-columns: 160px 1fr; align-items: center; gap: 16px; padding: 14px 0; border-bottom: 1px solid var(--line); }
.frm_lb { font-weight: 600; }
.frm_val { display: flex; align-items: center; gap: 20px; }
.frm_val label { display: flex; align-items: center; gap: 6px; }

@media (max-width: 768px) { .frm_row { grid-template-columns: 1fr; gap: 6px; } }
```

Radio size and `accent-color` come from the reset — not declared here.

## 7. Numbered step list

The number is generated, not typed.

```html
<ol class="step_list">
  <li class="step_item"><strong class="step_tt">Apply</strong><p class="step_desc">Submit the online form.</p></li>
  <li class="step_item"><strong class="step_tt">Review</strong><p class="step_desc">Documents are screened.</p></li>
</ol>
```

```css
.step_list { counter-reset: step; display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 20px; }
.step_item { counter-increment: step; position: relative; padding: 28px 20px 20px; border-radius: 12px; background: var(--color-bg-soft); }
.step_item::before { content: counter(step, decimal-leading-zero); position: absolute; top: 14px; left: 20px; font-size: 14px; font-weight: 700; color: var(--color-primary); }
.step_tt { display: block; font-size: 18px; }
.step_desc { margin-top: 6px; font-size: 15px; color: var(--color-sub); }
```

## 8. Footer

```html
<footer class="footer">
  <nav class="ft_policy"><a href="#">Terms</a><a href="#"><strong>Privacy</strong></a><a href="#">Sitemap</a></nav>
  <address class="ft_info">123 Example-ro, Seoul · Tel 02-000-0000</address>
  <p class="ft_copy">© Company. All rights reserved.</p>
  <div class="sns"><a href="#" class="sns_yt"></a><a href="#" class="sns_ig"></a></div>
</footer>
```

```css
.footer { padding: 40px 20px; background: var(--color-bg-dark); color: #cfd3d8; }
.ft_policy { display: flex; gap: 20px; }
.ft_policy a:hover { text-decoration: underline; }
.ft_info { margin-top: 16px; font-size: 14px; font-style: normal; }
.ft_copy { margin-top: 8px; font-size: 13px; }
.sns a { width: 36px; height: 36px; border-radius: 50%; background-color: rgba(255,255,255,.1); }
.sns_yt { background-image: url('/img/ic_yt.svg'); background-repeat: no-repeat; background-position: center; }
.sns_yt:hover { background-image: url('/img/ic_yt_on.svg'); }
```

`.sns` keeps `background-color` and `background-image` separate so the hover swap does not wipe the circle behind the icon.
