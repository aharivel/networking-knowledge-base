# Line Rate Calculator

> Calculate the maximum packet rate (pps) for a given link speed and packet size

---

## ⚡ Calculator

<div id="line-rate-calc">
  <div class="calc-grid">
    <div class="calc-field">
      <label for="link-speed">Link Speed</label>
      <div class="input-row">
        <input type="number" id="link-speed" value="25" min="0.001" step="any" />
        <select id="speed-unit">
          <option value="1000000000" selected>Gbps</option>
          <option value="1000000">Mbps</option>
        </select>
      </div>
    </div>
    <div class="calc-field">
      <label for="pkt-size">Packet Size (bytes)</label>
      <div class="input-row">
        <input type="number" id="pkt-size" value="64" min="64" max="9216" step="1" />
        <div class="pkt-presets">
          <button onclick="setPktSize(64)">64</button>
          <button onclick="setPktSize(128)">128</button>
          <button onclick="setPktSize(256)">256</button>
          <button onclick="setPktSize(512)">512</button>
          <button onclick="setPktSize(1518)">1518</button>
          <button onclick="setPktSize(9000)">9K</button>
        </div>
      </div>
    </div>
  </div>
  <div class="calc-results">
    <div class="result-card primary">
      <span class="result-label">Packet Rate</span>
      <span class="result-value" id="result-pps">—</span>
    </div>
    <div class="result-card">
      <span class="result-label">Actual Throughput</span>
      <span class="result-value" id="result-throughput">—</span>
    </div>
    <div class="result-card">
      <span class="result-label">Ethernet Efficiency</span>
      <span class="result-value" id="result-efficiency">—</span>
    </div>
    <div class="result-card">
      <span class="result-label">Wire Size (with overhead)</span>
      <span class="result-value" id="result-wire-size">—</span>
    </div>
  </div>
  <div class="calc-breakdown" id="calc-breakdown"></div>
</div>

<style>
#line-rate-calc {
  font-family: 'DM Sans', sans-serif;
  margin: 1.5rem 0;
}
.calc-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.25rem;
  margin-bottom: 1.5rem;
}
@media (max-width: 600px) { .calc-grid { grid-template-columns: 1fr; } }
.calc-field label {
  display: block;
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.7rem;
  font-weight: 600;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: #8b9dc3;
  margin-bottom: 0.4rem;
}
[data-theme="dark"] .calc-field label { color: #8b9dc3; }
.input-row {
  display: flex;
  gap: 0.5rem;
  align-items: center;
  flex-wrap: wrap;
}
.calc-field input[type="number"] {
  width: 120px;
  padding: 0.6rem 0.75rem;
  border: 1px solid #dce2ed;
  border-radius: 6px;
  font-family: 'JetBrains Mono', monospace;
  font-size: 1.1rem;
  font-weight: 600;
  color: #1b2a4a;
  background: #fff;
  outline: none;
  transition: border-color 0.15s;
}
.calc-field input:focus { border-color: #0af0ce; }
[data-theme="dark"] .calc-field input[type="number"] {
  background: #1a2744;
  border-color: #2d3f5f;
  color: #e8ecf2;
}
.calc-field select {
  padding: 0.6rem 0.5rem;
  border: 1px solid #dce2ed;
  border-radius: 6px;
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.85rem;
  color: #1b2a4a;
  background: #fff;
  cursor: pointer;
}
[data-theme="dark"] .calc-field select {
  background: #1a2744;
  border-color: #2d3f5f;
  color: #e8ecf2;
}
.pkt-presets {
  display: flex;
  gap: 0.25rem;
}
.pkt-presets button {
  padding: 0.35rem 0.55rem;
  border: 1px solid #dce2ed;
  border-radius: 4px;
  background: #f4f6fa;
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.7rem;
  font-weight: 500;
  color: #3d5a80;
  cursor: pointer;
  transition: all 0.12s;
}
.pkt-presets button:hover {
  background: #0af0ce;
  color: #0d1b2a;
  border-color: #0af0ce;
}
[data-theme="dark"] .pkt-presets button {
  background: #1a2744;
  border-color: #2d3f5f;
  color: #8b9dc3;
}
[data-theme="dark"] .pkt-presets button:hover {
  background: #0af0ce;
  color: #0d1b2a;
  border-color: #0af0ce;
}
.calc-results {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 0.75rem;
  margin-bottom: 1.25rem;
}
@media (max-width: 600px) { .calc-results { grid-template-columns: 1fr; } }
.result-card {
  background: #f4f6fa;
  border: 1px solid #dce2ed;
  border-radius: 8px;
  padding: 1rem 1.25rem;
  display: flex;
  flex-direction: column;
  gap: 0.3rem;
}
.result-card.primary {
  background: #0d1b2a;
  border-color: #0af0ce;
}
.result-card.primary .result-label { color: #8b9dc3; }
.result-card.primary .result-value { color: #0af0ce; }
[data-theme="dark"] .result-card {
  background: #1a2744;
  border-color: #2d3f5f;
}
[data-theme="dark"] .result-card.primary {
  background: #0d1b2a;
  border-color: #0af0ce;
}
.result-label {
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.65rem;
  font-weight: 600;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: #8b9dc3;
}
.result-value {
  font-family: 'JetBrains Mono', monospace;
  font-size: 1.3rem;
  font-weight: 700;
  color: #1b2a4a;
}
[data-theme="dark"] .result-value { color: #e8ecf2; }
.calc-breakdown {
  background: #f4f6fa;
  border: 1px solid #dce2ed;
  border-radius: 8px;
  padding: 1rem 1.25rem;
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.8rem;
  line-height: 1.8;
  color: #3d5a80;
}
[data-theme="dark"] .calc-breakdown {
  background: #1a2744;
  border-color: #2d3f5f;
  color: #8b9dc3;
}
.calc-breakdown .hl { color: #0af0ce; font-weight: 600; }
</style>

<script>
function setPktSize(size) {
  document.getElementById('pkt-size').value = size;
  calculate();
}

function formatNumber(n) {
  if (n >= 1e9) return (n / 1e9).toFixed(2) + ' Gpps';
  if (n >= 1e6) return (n / 1e6).toFixed(2) + ' Mpps';
  if (n >= 1e3) return (n / 1e3).toFixed(2) + ' Kpps';
  return n.toFixed(0) + ' pps';
}

function formatBits(bps) {
  if (bps >= 1e9) return (bps / 1e9).toFixed(2) + ' Gbps';
  if (bps >= 1e6) return (bps / 1e6).toFixed(2) + ' Mbps';
  if (bps >= 1e3) return (bps / 1e3).toFixed(2) + ' Kbps';
  return bps.toFixed(0) + ' bps';
}

function calculate() {
  var speed = parseFloat(document.getElementById('link-speed').value);
  var unit = parseFloat(document.getElementById('speed-unit').value);
  var pktSize = parseInt(document.getElementById('pkt-size').value);

  if (isNaN(speed) || isNaN(pktSize) || speed <= 0 || pktSize < 64) {
    document.getElementById('result-pps').textContent = '—';
    document.getElementById('result-throughput').textContent = '—';
    document.getElementById('result-efficiency').textContent = '—';
    document.getElementById('result-wire-size').textContent = '—';
    document.getElementById('calc-breakdown').innerHTML = '';
    return;
  }

  var linkBps = speed * unit;
  // Ethernet overhead: 7 preamble + 1 SFD + 4 FCS + 12 IFG = 24 bytes
  // But FCS (4B) is usually included in the packet size convention (64B min includes FCS)
  // So additional overhead on wire = 7 (preamble) + 1 (SFD) + 12 (IFG) = 20 bytes
  var overhead = 20;
  var wireSize = pktSize + overhead;
  var wireBits = wireSize * 8;
  var pps = linkBps / wireBits;
  var payloadBps = pps * pktSize * 8;
  var efficiency = (pktSize / wireSize) * 100;

  document.getElementById('result-pps').textContent = formatNumber(pps);
  document.getElementById('result-throughput').textContent = formatBits(payloadBps);
  document.getElementById('result-efficiency').textContent = efficiency.toFixed(1) + '%';
  document.getElementById('result-wire-size').textContent = wireSize + ' bytes';

  document.getElementById('calc-breakdown').innerHTML =
    '<span class="hl">Formula:</span> pps = link_speed / (wire_size × 8)<br>' +
    '<span class="hl">Wire size:</span> ' + pktSize + 'B (packet) + 7B (preamble) + 1B (SFD) + 12B (IFG) = <span class="hl">' + wireSize + ' bytes</span><br>' +
    '<span class="hl">Calculation:</span> ' + formatBits(linkBps) + ' / (' + wireSize + ' × 8) = ' + formatBits(linkBps) + ' / ' + wireBits + ' bits = <span class="hl">' + formatNumber(pps) + '</span>';
}

// Run on load and on any input change
document.addEventListener('DOMContentLoaded', calculate);
if (document.getElementById('link-speed')) {
  document.getElementById('link-speed').addEventListener('input', calculate);
  document.getElementById('pkt-size').addEventListener('input', calculate);
  document.getElementById('speed-unit').addEventListener('change', calculate);
  calculate();
}
</script>

---

## 📐 The Formula

For Ethernet, every frame has fixed overhead on the wire that's not part of the packet itself:

| Component | Size | Description |
|-----------|------|-------------|
| Preamble | 7 bytes | Synchronization pattern (10101010...) |
| SFD | 1 byte | Start Frame Delimiter (10101011) |
| Packet | 64–9216 bytes | Dst MAC + Src MAC + [VLAN] + EtherType + Payload + FCS |
| IFG | 12 bytes | Inter-Frame Gap (minimum silence between frames) |

```
Wire size = Packet size + 20 bytes (preamble + SFD + IFG)
Line rate (pps) = Link speed (bps) / (Wire size × 8)
```

**Note:** The 4-byte FCS (Frame Check Sequence) is typically included in the stated packet size. A "64-byte packet" means 64 bytes including FCS. The additional wire overhead is 7 + 1 + 12 = 20 bytes.

## 📊 Reference Table

Common line rates for standard link speeds:

| Packet Size | 1 GbE | 10 GbE | 25 GbE | 40 GbE | 100 GbE |
|------------|-------|--------|--------|--------|---------|
| 64 B | 1.49 Mpps | 14.88 Mpps | 37.20 Mpps | 59.52 Mpps | 148.81 Mpps |
| 128 B | 0.84 Mpps | 8.45 Mpps | 21.12 Mpps | 33.78 Mpps | 84.46 Mpps |
| 256 B | 0.45 Mpps | 4.53 Mpps | 11.33 Mpps | 18.12 Mpps | 45.29 Mpps |
| 512 B | 0.23 Mpps | 2.35 Mpps | 5.87 Mpps | 9.39 Mpps | 23.48 Mpps |
| 1518 B | 0.08 Mpps | 0.81 Mpps | 2.03 Mpps | 3.25 Mpps | 8.13 Mpps |
| 9000 B | 0.01 Mpps | 0.14 Mpps | 0.35 Mpps | 0.55 Mpps | 1.38 Mpps |

## 💡 Why This Matters

- **NFV testing**: when benchmarking a virtual switch or VNF, you need to know the theoretical max to calculate drop rate percentage
- **NIC validation**: verify a NIC reaches line rate at different packet sizes
- **TRex profiles**: set the correct `pps` target for your traffic streams
- **Capacity planning**: small packets are the worst case for pps — a firewall rated at 10 Mpps might handle 100 GbE with 1518B packets but choke at 64B

## 🔗 Related Topics

- [Cisco TRex](trex.md) — Traffic generation at line rate
- [Kernel Bypass](kernel-bypass.md) — DPDK for line-rate processing
- [Benchmarks & Tools](benchmarks.md) — Performance testing methodology
