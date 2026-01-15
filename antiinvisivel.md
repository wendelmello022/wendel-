/**
 * Business Text (Grafana) - Anti-Invisible Core
 * - Mantém último valor válido (cache em sessionStorage)
 * - Reaplica após re-render (MutationObserver)
 * - Intervalo de manutenção + burst inicial
 *
 * Como usar:
 * 1) Cole este arquivo no GitHub (raw) ou copie direto no painel
 * 2) Em cada painel, você chama BTAntiInvisible.create({...})
 */

(function (global) {
  const NS = "BTAntiInvisible";

  function safeJSONParse(s, fallback) {
    try { return JSON.parse(s); } catch { return fallback; }
  }

  function loadCache(cacheKey) {
    return safeJSONParse(sessionStorage.getItem(cacheKey) || "{}", {});
  }

  function saveCache(cacheKey, obj) {
    try { sessionStorage.setItem(cacheKey, JSON.stringify(obj)); } catch {}
  }

  function create(opts) {
    const {
      rootId,
      itemSelector,
      // função que lê e retorna um "valor atual" (qualquer tipo)
      readValue,
      // função que diz se valor é válido (default: finite number)
      isValid = (v) => (typeof v === "number" && isFinite(v)),
      // função que gera chave estável do item (por padrão: id ou índice)
      getItemKey = (el, idx) => el.id || `item_${idx}`,
      // função que aplica no DOM (recebe el, value, ctx)
      apply,
      cacheKey = `bt_cache_${rootId}_v1`,
      // timers
      burstTimes = 10,
      burstEveryMs = 800,
      tickEveryMs = 2000,
      // nomes globais (pra evitar duplicar)
      globalTimerKey = `__btTimer_${rootId}`,
      globalObsKey = `__btObs_${rootId}`,
    } = opts || {};

    const root = document.getElementById(rootId);
    if (!root) return;

    function applyOnce() {
      const cache = loadCache(cacheKey);
      let changed = false;

      root.querySelectorAll(itemSelector).forEach((el, idx) => {
        const key = getItemKey(el, idx);

        let value = readValue(el, idx);

        // anti-invisível: se inválido, tenta cache
        if (!isValid(value)) {
          if (cache[key] !== undefined) {
            value = cache[key];
          }
        } else {
          // valor válido -> salva
          if (cache[key] !== value) {
            cache[key] = value;
            changed = true;
          }
        }

        apply(el, value, { key, idx, root });
      });

      if (changed) saveCache(cacheKey, cache);
    }

    // limpa watcher/timer antigos
    if (global[globalObsKey] && global[globalObsKey].disconnect) {
      global[globalObsKey].disconnect();
    }
    if (global[globalTimerKey]) {
      clearInterval(global[globalTimerKey]);
    }

    // observer para re-render
    let raf = 0;
    const obs = new MutationObserver(() => {
      cancelAnimationFrame(raf);
      raf = requestAnimationFrame(applyOnce);
    });
    obs.observe(root, { childList: true, subtree: true, characterData: true });
    global[globalObsKey] = obs;

    // burst inicial
    let burst = 0;
    const init = setInterval(() => {
      applyOnce();
      if (++burst >= burstTimes) clearInterval(init);
    }, burstEveryMs);

    // manutenção
    applyOnce();
    global[globalTimerKey] = setInterval(applyOnce, tickEveryMs);
  }

  global[NS] = { create };
})(window);
