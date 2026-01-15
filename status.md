<div id="icmpCards" class="icmp-wrapper">
  {{#each data}}
    <div class="icmp-card">
      <span class="raw-name">{{Field}}{{Name}}</span>
      <span class="raw-last">{{Last}}</span>

      <div class="icmp-unit">{{Field}}{{Name}}</div>
      <div class="icmp-pill">—</div>
    </div>
  {{/each}}
</div>


JS

    const root = document.getElementById("icmpCards");
if (!root) return;

root.querySelectorAll(".icmp-card").forEach((card) => {
  const raw = (card.querySelector(".raw-last")?.textContent || "").trim();

  // vazio/null/undefined => NaN => UP
  const normalized =
    raw === "" || raw.toLowerCase() === "null" || raw.toLowerCase() === "undefined"
      ? "NaN"
      : raw;

  const value = Number(normalized);

  // DOWN somente se for 0
  const isDown = (value === 0);

  card.classList.remove("up", "down");
  card.classList.add(isDown ? "down" : "up");

  const pill = card.querySelector(".icmp-pill");
  if (pill) pill.textContent = isDown ? "DOWN" : "UP";
});

CSS

.icmp-wrapper{
  display:flex;
  flex-direction:column;
  gap:10px;
}

.icmp-card{
  background:#0f1117;
  border:1px solid #1f2430;
  border-radius:14px;
  padding:14px 16px;
  display:flex;
  align-items:center;
  justify-content:space-between;
  border-left:6px solid #6b7280;
  box-shadow:0 8px 22px rgba(0,0,0,.25);
}

/* ✅ ESSA É A CORREÇÃO DO DUPLICADO */
.raw-name,
.raw-last{
  display:none;
}

.icmp-unit{
  color:#EDEDED;
  font-size:15px;
  font-weight:900;
  letter-spacing:.3px;
  text-transform:uppercase;
  white-space:nowrap;
  overflow:hidden;
  text-overflow:ellipsis;
  max-width:520px;
}

.icmp-pill{
  min-width:88px;
  padding:8px 14px;
  border-radius:999px;
  font-weight:900;
  font-size:13px;
  text-align:center;
  background:rgba(255,255,255,.06);
  color:#cbd5e1;
}

/* UP */
.icmp-card.up{ border-left-color:#73BF69; }
.icmp-card.up .icmp-pill{ background:#73BF69; color:#05140b; }

/* DOWN */
.icmp-card.down{ border-left-color:#E02F44; }
.icmp-card.down .icmp-pill{ background:#E02F44; color:#1a070a; }
