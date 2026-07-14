import React, { useState, useEffect, useRef } from "react";
import { TrendingUp, Target, ArrowLeft, RotateCcw, ChevronDown } from "lucide-react";

// ---- Design tokens ----
// Navy / blue / gold — echoes the seminar deck's own palette.
const C = {
  navy: "#14284B",
  blue: "#1E4C8C",
  blueTint: "#E8F0FB",
  gold: "#C8891A",
  goldTint: "#FBF0DA",
  ink: "#1A1F2B",
  sub: "#5B6472",
  paper: "#F6F7FA",
  line: "#DCE2EC",
};

const FONT = `"Noto Sans JP","Yu Gothic Medium","Yu Gothic","Hiragino Sans",sans-serif`;
const MONO = `"Roboto Mono","Yu Gothic Medium","Consolas",monospace`;

// 一言コメントのプリセット（結果表示のたびにランダムで1つピックアップ）
const COMMENT_PRESETS = [
  "この株価、思ったより強気だった説。",
  "電卓の中で「がんばったね」の声がした。",
  "簿価と株価のギャップ、青春の温度差くらいある。",
  "この数字、ゼミで自信満々に発表してよし。",
  "ROCEが高いと気分まで高くなる、謎現象。",
];
function pickRandomComment() {
  return COMMENT_PRESETS[Math.floor(Math.random() * COMMENT_PRESETS.length)];
}

function fmt(n, digits = 2) {
  if (!isFinite(n)) return "—";
  return n.toLocaleString("ja-JP", { minimumFractionDigits: digits, maximumFractionDigits: digits });
}

// ---- Count-up number (the signature "ticker" reveal) ----
function TickerNumber({ value, suffix, digits, big }) {
  const [display, setDisplay] = useState(0);
  const raf = useRef(null);

  useEffect(() => {
    const start = performance.now();
    const duration = 1100;
    const from = 0;
    const to = value;
    function tick(now) {
      const t = Math.min(1, (now - start) / duration);
      const eased = 1 - Math.pow(1 - t, 3);
      setDisplay(from + (to - from) * eased);
      if (t < 1) raf.current = requestAnimationFrame(tick);
    }
    raf.current = requestAnimationFrame(tick);
    return () => cancelAnimationFrame(raf.current);
  }, [value]);

  return (
    <span
      style={{
        fontFamily: MONO,
        fontVariantNumeric: "tabular-nums",
        fontWeight: 700,
        fontSize: big ? "clamp(2.6rem,9vw,4.2rem)" : "2rem",
        color: C.navy,
        letterSpacing: "-0.01em",
      }}
    >
      {fmt(display, digits)}
      <span style={{ fontSize: "0.45em", marginLeft: 6, color: C.gold, fontFamily: FONT, fontWeight: 700 }}>
        {suffix}
      </span>
    </span>
  );
}

function Sparkles({ active }) {
  const dots = [
    { top: "10%", left: "8%", d: "0s", s: 6 },
    { top: "78%", left: "4%", d: "0.3s", s: 4 },
    { top: "18%", left: "92%", d: "0.15s", s: 5 },
    { top: "70%", left: "94%", d: "0.45s", s: 7 },
    { top: "4%", left: "48%", d: "0.6s", s: 4 },
    { top: "90%", left: "52%", d: "0.2s", s: 5 },
  ];
  return (
    <div style={{ position: "absolute", inset: 0, pointerEvents: "none", overflow: "hidden" }}>
      {dots.map((p, i) => (
        <span
          key={i}
          style={{
            position: "absolute",
            top: p.top,
            left: p.left,
            width: p.s,
            height: p.s,
            borderRadius: "50%",
            background: i % 2 === 0 ? C.gold : C.blue,
            opacity: active ? 0.9 : 0,
            transform: active ? "scale(1)" : "scale(0)",
            transition: `opacity 0.5s ease ${p.d}, transform 0.5s cubic-bezier(.2,1.4,.4,1) ${p.d}`,
          }}
        />
      ))}
    </div>
  );
}

// ---- Field ----
function Field({ label, unit, value, onChange, placeholder }) {
  return (
    <label style={{ display: "block", marginBottom: 18 }}>
      <div style={{ fontSize: 13, color: C.sub, marginBottom: 6, fontWeight: 600 }}>{label}</div>
      <div style={{ display: "flex", alignItems: "center", border: `1.5px solid ${C.line}`, borderRadius: 10, background: "#fff", overflow: "hidden" }}>
        <input
          type="number"
          inputMode="decimal"
          value={value}
          onChange={(e) => onChange(e.target.value)}
          placeholder={placeholder}
          style={{
            flex: 1,
            padding: "12px 14px",
            border: "none",
            outline: "none",
            fontFamily: MONO,
            fontSize: 16,
            color: C.ink,
            background: "transparent",
          }}
        />
        {unit && (
          <div style={{ padding: "0 14px", color: C.sub, fontSize: 14, fontFamily: FONT, borderLeft: `1px solid ${C.line}` }}>
            {unit}
          </div>
        )}
      </div>
    </label>
  );
}

// ---- Rate selector: dropdown of presets + custom ----
function RateSelect({ label, value, onChange }) {
  const presets = [5, 8, 10, 12, 15];
  const isPreset = presets.includes(Number(value));
  const [mode, setMode] = useState(isPreset || value === "" ? "preset" : "custom");

  return (
    <div style={{ marginBottom: 18 }}>
      <div style={{ fontSize: 13, color: C.sub, marginBottom: 6, fontWeight: 600 }}>{label}</div>
      <div style={{ display: "flex", gap: 8 }}>
        <div style={{ position: "relative", flex: mode === "preset" ? 1 : "0 0 auto" }}>
          <select
            value={mode === "preset" ? value || "" : ""}
            onChange={(e) => {
              if (e.target.value === "__custom") {
                setMode("custom");
                onChange("");
              } else {
                setMode("preset");
                onChange(e.target.value);
              }
            }}
            style={{
              appearance: "none",
              width: "100%",
              padding: "12px 36px 12px 14px",
              border: `1.5px solid ${C.line}`,
              borderRadius: 10,
              background: "#fff",
              fontFamily: FONT,
              fontSize: 15,
              color: C.ink,
              cursor: "pointer",
            }}
          >
            <option value="" disabled>
              選択してください
            </option>
            {presets.map((p) => (
              <option key={p} value={p}>
                {p}%
              </option>
            ))}
            <option value="__custom">カスタム…</option>
          </select>
          <ChevronDown size={16} style={{ position: "absolute", right: 12, top: "50%", transform: "translateY(-50%)", color: C.sub, pointerEvents: "none" }} />
        </div>
        {mode === "custom" && (
          <div style={{ display: "flex", alignItems: "center", border: `1.5px solid ${C.line}`, borderRadius: 10, background: "#fff", flex: 1 }}>
            <input
              type="number"
              inputMode="decimal"
              autoFocus
              value={value}
              onChange={(e) => onChange(e.target.value)}
              placeholder="任意の値"
              style={{ flex: 1, padding: "12px 14px", border: "none", outline: "none", fontFamily: MONO, fontSize: 16, color: C.ink, background: "transparent" }}
            />
            <div style={{ padding: "0 14px", color: C.sub, fontSize: 14, borderLeft: `1px solid ${C.line}` }}>%</div>
          </div>
        )}
      </div>
    </div>
  );
}

export default function ValuationTool() {
  const [screen, setScreen] = useState("home"); // home | input | result
  const [mode, setMode] = useState(null); // 'growth' | 'return'

  useEffect(() => {
    if (document.getElementById("vt-font-link")) return;
    const link = document.createElement("link");
    link.id = "vt-font-link";
    link.rel = "stylesheet";
    link.href = "https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;500;700;800&family=Roboto+Mono:wght@500;700&display=swap";
    document.head.appendChild(link);
  }, []);

  const [p0, setP0] = useState("");
  const [b0, setB0] = useState("");
  const [e1, setE1] = useState("");
  const [div1, setDiv1] = useState("");
  const [rate, setRate] = useState(""); // required return (growth mode) or growth rate (return mode), in %

  const [result, setResult] = useState(null);
  const [error, setError] = useState("");
  const [comment, setComment] = useState("");
  const [reveal, setReveal] = useState(false);

  function goHome() {
    setScreen("home");
    setMode(null);
    setP0(""); setB0(""); setE1(""); setDiv1(""); setRate("");
    setResult(null); setError(""); setComment(""); setReveal(false);
  }

  function chooseMode(m) {
    setMode(m);
    setScreen("input");
    setResult(null); setError(""); setReveal(false);
  }

  function compute() {
    const P0 = parseFloat(p0), B0 = parseFloat(b0), E1 = parseFloat(e1);
    setError("");
    if (!isFinite(P0) || !isFinite(B0) || !isFinite(E1)) {
      setError("株価・簿価・予想利益をすべて入力してください。");
      return;
    }
    if (B0 === 0) {
      setError("簿価は0にできません。");
      return;
    }
    const ROCE = E1 / B0;

    if (mode === "growth") {
      const r = parseFloat(rate) / 100;
      if (!isFinite(r)) {
        setError("要求リターンを入力してください。");
        return;
      }
      if (P0 === B0) {
        setError("株価と簿価が同じだと成長率を求められません（分母が0になります）。");
        return;
      }
      const RE1 = E1 - r * B0;
      const g = r - RE1 / (P0 - B0);
      setResult({
        ROCE, RE1, r, g,
        headline: 1 + g,
        headlineSuffix: "",
        headlineDigits: 3,
        subHeadline: `= 成長率 ${fmt(g * 100, 2)}%`,
      });
    } else {
      const g = parseFloat(rate) / 100;
      if (!isFinite(g)) {
        setError("成長率を入力してください。");
        return;
      }
      const k = P0 / B0 - 1;
      const r = (ROCE + k * g) / (k + 1);
      setResult({
        ROCE, k, g, r,
        headline: r * 100,
        headlineSuffix: "%",
        headlineDigits: 2,
        subHeadline: `簿価・株価倍率 ${fmt(B0 / P0, 3)}`,
      });
    }
    setComment(pickRandomComment());
    setScreen("result");
    setReveal(false);
    setTimeout(() => setReveal(true), 60);
  }

  const isGrowth = mode === "growth";
  const accent = isGrowth ? C.blue : C.gold;
  const accentTint = isGrowth ? C.blueTint : C.goldTint;

  return (
    <div
      style={{
        minHeight: "100vh",
        background: C.paper,
        fontFamily: FONT,
        color: C.ink,
        display: "flex",
        justifyContent: "center",
        padding: "28px 16px",
      }}
    >
      <style>{`
        .vt-container { width: 100%; max-width: 480px; margin: 0 auto; }
        .vt-home-cards { display: flex; flex-direction: column; gap: 14px; }
        .vt-input-layout, .vt-result-layout { display: block; }
        @media (min-width: 780px) {
          .vt-container { max-width: 900px; }
          .vt-home-cards { flex-direction: row; gap: 20px; }
          .vt-home-cards > button { flex: 1; margin-bottom: 0 !important; }
          .vt-input-layout { display: grid; grid-template-columns: 1.15fr 0.85fr; gap: 24px; align-items: start; }
          .vt-result-layout { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; align-items: start; }
          .vt-result-layout .vt-result-hero { grid-row: 1 / 3; margin-bottom: 0 !important; }
        }
      `}</style>
      <div className="vt-container">
        {/* ---------------- HOME ---------------- */}
        {screen === "home" && (
          <div>
            <div style={{ textAlign: "center", marginBottom: 36, marginTop: 24 }}>
              <div style={{ fontSize: 12, letterSpacing: "0.2em", color: C.sub, fontWeight: 700, marginBottom: 10 }}>
                VALUATION TOOL
              </div>
              <h1 style={{ fontSize: 26, fontWeight: 800, color: C.navy, margin: 0, lineHeight: 1.4 }}>
                バリュエーションと<br />アクティブ投資
              </h1>
              <div style={{ width: 44, height: 3, background: C.gold, margin: "16px auto 0" }} />
            </div>

            <div className="vt-home-cards">
            <button
              onClick={() => chooseMode("growth")}
              style={{
                width: "100%",
                textAlign: "left",
                background: "#fff",
                border: `1.5px solid ${C.line}`,
                borderRadius: 16,
                padding: 20,
                marginBottom: 14,
                cursor: "pointer",
                display: "flex",
                alignItems: "center",
                gap: 16,
                transition: "border-color 0.15s, transform 0.15s",
              }}
              onMouseEnter={(e) => (e.currentTarget.style.borderColor = C.blue)}
              onMouseLeave={(e) => (e.currentTarget.style.borderColor = C.line)}
            >
              <div style={{ width: 48, height: 48, borderRadius: 12, background: C.blue, display: "flex", alignItems: "center", justifyContent: "center", flexShrink: 0 }}>
                <TrendingUp color="#fff" size={24} />
              </div>
              <div>
                <div style={{ fontWeight: 800, fontSize: 17, color: C.navy }}>インプライド成長率</div>
                <div style={{ fontSize: 13, color: C.sub, marginTop: 2 }}>株価に織り込まれた成長率を逆算する</div>
              </div>
            </button>

            <button
              onClick={() => chooseMode("return")}
              style={{
                width: "100%",
                textAlign: "left",
                background: "#fff",
                border: `1.5px solid ${C.line}`,
                borderRadius: 16,
                padding: 20,
                cursor: "pointer",
                display: "flex",
                alignItems: "center",
                gap: 16,
              }}
              onMouseEnter={(e) => (e.currentTarget.style.borderColor = C.gold)}
              onMouseLeave={(e) => (e.currentTarget.style.borderColor = C.line)}
            >
              <div style={{ width: 48, height: 48, borderRadius: 12, background: C.gold, display: "flex", alignItems: "center", justifyContent: "center", flexShrink: 0 }}>
                <Target color="#fff" size={24} />
              </div>
              <div>
                <div style={{ fontWeight: 800, fontSize: 17, color: C.navy }}>インプライド期待リターン</div>
                <div style={{ fontSize: 13, color: C.sub, marginTop: 2 }}>株価に織り込まれた期待リターンを逆算する</div>
              </div>
            </button>
            </div>
          </div>
        )}

        {/* ---------------- INPUT ---------------- */}
        {screen === "input" && (
          <div>
            <button onClick={goHome} style={{ display: "flex", alignItems: "center", gap: 6, background: "none", border: "none", color: C.sub, fontSize: 14, cursor: "pointer", padding: 0, marginBottom: 20 }}>
              <ArrowLeft size={16} /> はじめに戻る
            </button>

            <div style={{ display: "flex", alignItems: "center", gap: 10, marginBottom: 24 }}>
              <div style={{ width: 36, height: 36, borderRadius: 10, background: accent, display: "flex", alignItems: "center", justifyContent: "center" }}>
                {isGrowth ? <TrendingUp color="#fff" size={18} /> : <Target color="#fff" size={18} />}
              </div>
              <h2 style={{ fontSize: 19, fontWeight: 800, color: C.navy, margin: 0 }}>
                {isGrowth ? "インプライド成長率" : "インプライド期待リターン"}
              </h2>
            </div>

            <div className="vt-input-layout">
            <div style={{ background: "#fff", border: `1.5px solid ${C.line}`, borderRadius: 16, padding: 22 }}>
              <Field label="株価（第0期）" unit="円" value={p0} onChange={setP0} placeholder="例）133.71" />
              <Field label="簿価（第0期）" unit="円" value={b0} onChange={setB0} placeholder="例）100.00" />
              <Field label="予想利益（第1期）" unit="円" value={e1} onChange={setE1} placeholder="例）12.36" />
              <Field label="予想配当（第1期・任意）" unit="円" value={div1} onChange={setDiv1} placeholder="なければ空欄でOK" />

              <div style={{ height: 1, background: C.line, margin: "6px 0 18px" }} />

              <RateSelect
                label={isGrowth ? "要求リターン" : "成長率"}
                value={rate}
                onChange={setRate}
              />

              {error && (
                <div style={{ color: "#B3261E", fontSize: 13, marginBottom: 14, fontWeight: 600 }}>{error}</div>
              )}

              <button
                onClick={compute}
                style={{
                  width: "100%",
                  padding: "14px",
                  borderRadius: 12,
                  border: "none",
                  background: C.navy,
                  color: "#fff",
                  fontSize: 15,
                  fontWeight: 800,
                  letterSpacing: "0.02em",
                  cursor: "pointer",
                  marginTop: 4,
                }}
              >
                {isGrowth ? "インプライド成長率を導出する" : "インプライド期待リターンを導出する"}
              </button>
            </div>

            <div style={{ background: accentTint, border: `1.5px solid ${accent}44`, borderRadius: 16, padding: 20, marginTop: 18 }}>
              <div style={{ fontSize: 13, fontWeight: 800, color: C.navy, marginBottom: 10 }}>計算のしくみ</div>
              <div style={{ fontSize: 13, color: C.ink, lineHeight: 1.9 }}>
                ROCE = 利益 ÷ 簿価
                <br />
                残余利益 = 利益 − 要求リターン × 簿価
                {isGrowth ? (
                  <>
                    <br />
                    成長率 = 要求リターン − 残余利益 ÷（株価 − 簿価）
                  </>
                ) : (
                  <>
                    <br />
                    要求リターン = ( ROCE + k × 成長率 ) ÷ ( k + 1 )
                    <br />
                    <span style={{ color: C.sub, fontSize: 12.5 }}>k = 株価 ÷ 簿価 − 1</span>
                  </>
                )}
              </div>
              <div style={{ fontSize: 12, color: C.sub, marginTop: 14, lineHeight: 1.7 }}>
                残余利益モデル（Penman 方式）で、株価に織り込まれた{isGrowth ? "成長率" : "期待リターン"}を逆算しています。
              </div>
            </div>
            </div>
          </div>
        )}

        {/* ---------------- RESULT ---------------- */}
        {screen === "result" && result && (
          <div>
            <button onClick={() => setScreen("input")} style={{ display: "flex", alignItems: "center", gap: 6, background: "none", border: "none", color: C.sub, fontSize: 14, cursor: "pointer", padding: 0, marginBottom: 20 }}>
              <ArrowLeft size={16} /> 数値を修正する
            </button>

            <div className="vt-result-layout">
            <div
              className="vt-result-hero"
              style={{
                position: "relative",
                background: accentTint,
                border: `1.5px solid ${accent}55`,
                borderRadius: 20,
                padding: "38px 20px 30px",
                textAlign: "center",
                overflow: "hidden",
                marginBottom: 18,
              }}
            >
              <Sparkles active={reveal} />
              <div style={{ fontSize: 12, letterSpacing: "0.18em", color: C.sub, fontWeight: 700, marginBottom: 10 }}>
                {isGrowth ? "IMPLIED GROWTH RATE" : "IMPLIED EXPECTED RETURN"}
              </div>
              {reveal ? (
                <TickerNumber value={result.headline} suffix={result.headlineSuffix} digits={result.headlineDigits} big />
              ) : (
                <div style={{ height: "4.2rem" }} />
              )}
              <div style={{ marginTop: 10, fontSize: 14, color: C.navy, fontWeight: 600, opacity: reveal ? 1 : 0, transition: "opacity 0.4s ease 0.8s" }}>
                {result.subHeadline}
              </div>
              <div
                style={{
                  width: reveal ? 64 : 0,
                  height: 3,
                  background: C.gold,
                  margin: "16px auto 0",
                  transition: "width 0.6s ease 0.3s",
                }}
              />
            </div>

            <div style={{ background: "#fff", border: `1.5px solid ${C.line}`, borderRadius: 16, padding: 20, marginBottom: 18 }}>
              <div style={{ fontSize: 13, fontWeight: 700, color: C.sub, marginBottom: 12 }}>計算の内訳</div>
              <Row label="ROCE (%)" value={`${fmt(result.ROCE * 100, 2)}%`} />
              {isGrowth ? (
                <>
                  <Row label="残余利益" value={fmt(result.RE1, 2)} />
                  <Row label="要求リターン" value={`${fmt(result.r * 100, 2)}%`} />
                  <Row label="成長率" value={`${fmt(result.g * 100, 2)}%`} strong />
                </>
              ) : (
                <>
                  <Row label="成長率" value={`${fmt(result.g * 100, 2)}%`} />
                  <Row label="要求リターン" value={`${fmt(result.r * 100, 2)}%`} strong />
                </>
              )}
            </div>

            <div style={{ background: "#fff", border: `1.5px solid ${C.line}`, borderRadius: 16, padding: 20, marginBottom: 18 }}>
              <div style={{ fontSize: 13, fontWeight: 700, color: C.sub, marginBottom: 2 }}>一言コメント</div>
              <div style={{ fontSize: 11.5, color: C.sub, marginBottom: 8, opacity: 0.85 }}>自動で1つ入れてます。書き換えてOK</div>
              <textarea
                value={comment}
                onChange={(e) => setComment(e.target.value)}
                placeholder="この結果について気づいたことをメモ…"
                rows={3}
                style={{
                  width: "100%",
                  border: `1.5px solid ${C.line}`,
                  borderRadius: 10,
                  padding: 12,
                  fontFamily: FONT,
                  fontSize: 14,
                  color: C.ink,
                  resize: "vertical",
                  outline: "none",
                  boxSizing: "border-box",
                }}
              />
            </div>
            </div>

            <button
              onClick={goHome}
              style={{
                width: "100%",
                padding: "13px",
                borderRadius: 12,
                border: `1.5px solid ${C.line}`,
                background: "#fff",
                color: C.navy,
                fontSize: 14,
                fontWeight: 700,
                cursor: "pointer",
                display: "flex",
                alignItems: "center",
                justifyContent: "center",
                gap: 8,
              }}
            >
              <RotateCcw size={16} /> 最初からやり直す
            </button>
          </div>
        )}
      </div>
    </div>
  );
}

function Row({ label, value, strong }) {
  return (
    <div style={{ display: "flex", justifyContent: "space-between", padding: "9px 0", borderBottom: `1px solid ${C.line}` }}>
      <span style={{ fontSize: 14, color: strong ? C.navy : C.sub, fontWeight: strong ? 800 : 500 }}>{label}</span>
      <span style={{ fontFamily: MONO, fontSize: 14, color: strong ? C.navy : C.ink, fontWeight: strong ? 800 : 600 }}>{value}</span>
    </div>
  );
}
