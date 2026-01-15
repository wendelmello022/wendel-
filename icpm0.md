BTAntiInvisible.create({
  rootId: "icmpCards",
  itemSelector: ".icmp-card",
  cacheKey: "bt_icmp_cache_v1",

  readValue: (card) => {
    const raw = (card.querySelector(".raw-last")?.textContent || "").trim();
    const normalized =
      raw === "" || raw.toLowerCase() === "null" || raw.toLowerCase() === "undefined"
        ? "NaN"
        : raw;
    return Number(normalized);
  },

  isValid: (v) => typeof v === "number" && isFinite(v),

  getItemKey: (card, idx) => card.id || `icmp_${idx}`,

  apply: (card, value) => {
    const isDown = (value === 0);

    card.classList.remove("up", "down");
    card.classList.add(isDown ? "down" : "up");

    const pill = card.querySelector(".icmp-pill");
    if (pill) pill.textContent = isDown ? "DOWN" : "UP";
  }
});
