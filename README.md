/* ============================================================
   FluorDB — Shared utilities for the static read-only site
   ============================================================ */

/** Convert a wavelength (nm) to an approximate visible-spectrum hex color. */
function nmColor(wl) {
  if (!wl) return '#aaaaaa';
  wl = Number(wl);
  if (wl < 380) return '#8B00FF';
  if (wl < 450) return '#4B0082';
  if (wl < 495) return '#0000FF';
  if (wl < 500) return '#00BFFF';
  if (wl < 570) return '#00FF00';
  if (wl < 590) return '#FFFF00';
  if (wl < 620) return '#FF7F00';
  if (wl < 750) return '#FF0000';
  return '#8B0000';
}

/** Slightly punchier palette used for the home-page scatter plot. */
function nmColorVivid(wl) {
  if (!wl) return '#aaaaaa';
  wl = Number(wl);
  if (wl < 380) return '#8B00FF';
  if (wl < 450) return '#6600cc';
  if (wl < 495) return '#3355ff';
  if (wl < 510) return '#00BFFF';
  if (wl < 540) return '#00cc66';
  if (wl < 570) return '#99ee00';
  if (wl < 590) return '#FFdd00';
  if (wl < 620) return '#FF8800';
  if (wl < 750) return '#FF2200';
  return '#990000';
}

/**
 * Binary-search linear interpolation of y at dataX within a sorted-by-x
 * array of {x, y} points. Returns null if dataX is outside the data range
 * or if there are fewer than 2 points.
 */
function interpY(pts, dataX) {
  if (!pts || pts.length < 2) return null;
  if (dataX < pts[0].x || dataX > pts[pts.length - 1].x) return null;
  let lo = 0, hi = pts.length - 1;
  while (lo < hi - 1) {
    const mid = (lo + hi) >> 1;
    if (pts[mid].x <= dataX) lo = mid; else hi = mid;
  }
  const loPt = pts[lo], hiPt = pts[hi];
  if (hiPt.x === loPt.x) return loPt.y;
  const t = (dataX - loPt.x) / (hiPt.x - loPt.x);
  return loPt.y + t * (hiPt.y - loPt.y);
}

/** Turn spectrum point pairs [[wl, intensity], ...] into sorted {x,y} objects. */
function toSortedXY(spectrum) {
  if (!spectrum) return null;
  return spectrum.map(p => ({ x: p[0], y: p[1] })).sort((a, b) => a.x - b.x);
}

/** Mirrors the admin tool's safe_filename_base(): spaces/dashes → underscores. */
function safeFilenameBase(name) {
  return name
    .toLowerCase()
    .trim()
    .replace(/[\s-]+/g, '_')
    .replace(/[^\w]/g, '');
}

/**
 * Render simple chemical-formula markup as real subscript/superscript HTML.
 * Syntax: _{...} -> <sub>...</sub>,  ^{...} -> <sup>...</sup>
 * e.g. "[Ag_{6}(DNA)*3H_{2}O]^{2+}" -> "[Ag<sub>6</sub>(DNA)*3H<sub>2</sub>O]<sup>2+</sup>"
 * Input is HTML-escaped first, so this is safe to use on user-entered text.
 */
function chemNotation(text) {
  if (!text) return '';
  const escaped = String(text)
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;');
  return escaped
    .replace(/_\{([^}]*)\}/g, '<sub>$1</sub>')
    .replace(/\^\{([^}]*)\}/g, '<sup>$1</sup>');
}

/**
 * Format a single lifetime value for compact table display:
 * - "IRF-limited" (any case) -> "IRF"
 * - a plain number -> "{value}{unit}", e.g. "5.2ns"
 * - any other free text -> shown as-is, no unit appended
 * Returns null if the value is empty.
 */
function formatLifetimeComponent(value, unit) {
  if (!value) return null;
  const v = String(value).trim();
  if (v.toLowerCase() === 'irf-limited' || v.toLowerCase() === 'irf') return 'IRF';
  if (v !== '' && !isNaN(Number(v))) return `${v}${unit}`;
  return v;
}

/** Combine lifetime_ns + lifetime_us into one compact label, e.g. "5.2ns/60µs". */
function lifetimeLabel(m) {
  const parts = [
    formatLifetimeComponent(m.lifetime_ns, 'ns'),
    formatLifetimeComponent(m.lifetime_us, 'µs'),
  ].filter(Boolean);
  return parts.length ? parts.join('/') : '—';
}

/** Fetch JSON with a friendly error if the file is missing (helps local testing). */
async function fetchJSON(path) {
  const res = await fetch(path);
  if (!res.ok) throw new Error(`Could not load ${path} (${res.status})`);
  return res.json();
}

/** Format a number, or return an em-dash if null/undefined/0. */
function fmt(value, digits) {
  if (value === null || value === undefined || value === '') return '—';
  return digits !== undefined ? Number(value).toFixed(digits) : value;
}
