/**
 * GorillaModeShell.tsx
 * AgentSam IDE Terminal — game-feel PTY shell for Inner Animal Media dashboard
 * Drop into: src/core/gorilla-shell/GorillaModeShell.tsx
 *
 * Requires: @xterm/xterm @xterm/addon-fit lucide-react
 * CSS import: @xterm/xterm/css/xterm.css  (add to your entry)
 */

import React, {
  useEffect, useRef, useState, useImperativeHandle,
  forwardRef, useCallback, useMemo,
} from 'react';
import { Terminal } from '@xterm/xterm';
import { FitAddon } from '@xterm/addon-fit';
import {
  X, ChevronDown, ChevronUp, TriangleAlert, CircleCheck,
  Terminal as TerminalIcon, Bot, Send, Loader2,
  Wifi, WifiOff, RefreshCw, Zap, Skull, Shield,
} from 'lucide-react';

// ─── Types ────────────────────────────────────────────────────────────────────
export type ShellTab = 'terminal' | 'output' | 'problems';

export interface GorillaModeHandle {
  writeToTerminal: (text: string) => void;
  runCommand: (cmd: string) => void;
  setActiveTab: (t: ShellTab) => void;
  triggerPump: () => void;
  triggerError: () => void;
}

export interface GorillaModeProps {
  onClose: () => void;
  problems?: { file: string; line: number; msg: string; severity: 'error' | 'warning' }[];
  outputLines?: string[];
  iamOrigin?: string;
  workspaceCdCommand?: string;
  agentDashboardUrl?: string;
  showWelcomeBar?: boolean;
  workspaceLabel?: string;
  workspaceId?: string;
  productLabel?: string;
}

// ─── Pixel palette ────────────────────────────────────────────────────────────
const PS = 5;
const PAL: string[] = [
  'transparent','#060e1a','#0f2040','#1a3568','#6a6a90','#9898b8',
  '#2a1006','#8a5830','#ff1a1a','#ffd88a','#0e0400','#ffffff',
  '#ffd700','#ffaa00','#ff6600','#ff2200','#9B6E14','#6B4808',
  '#3a2604','#c09030','#ffee44','#cc8800','#44ff88','#ff4444',
];

const GORILLA: number[][] = [
  [0,0,0,2,2,2,2,2,2,2,2,2,2,2,0,0,0,0],
  [0,0,2,3,3,3,3,3,3,3,3,3,3,2,2,0,0,0],
  [0,2,2,3,3,3,3,3,3,3,3,3,3,3,2,2,0,0],
  [0,2,3,3,7,7,3,3,3,7,7,3,3,3,2,0,0,0],
  [2,2,3,7,7,8,7,3,3,7,8,7,7,3,3,2,2,0],
  [2,2,3,7,7,7,7,7,7,7,7,7,7,3,3,2,2,0],
  [2,3,3,7,6,10,6,7,7,6,10,6,7,3,3,3,2,0],
  [2,3,3,7,7,7,9,9,9,9,7,7,7,3,3,3,2,0],
  [0,2,3,3,3,3,3,3,3,3,3,3,3,3,3,2,0,0],
  [0,2,2,3,4,4,4,4,4,4,4,4,4,3,2,2,0,0],
  [2,2,2,3,4,5,5,5,5,5,5,5,4,3,3,2,2,2],
  [2,3,2,3,4,5,5,5,5,5,5,5,4,3,2,3,2,2],
  [2,3,3,3,3,4,4,5,5,4,4,3,3,3,3,3,2,0],
  [0,2,3,3,3,3,3,3,3,3,3,3,3,3,2,2,0,0],
  [0,0,2,3,3,3,3,3,3,3,3,3,3,2,2,0,0,0],
  [0,0,2,2,3,3,3,3,3,3,3,3,2,2,0,0,0,0],
  [0,0,0,2,2,3,3,0,0,3,3,2,2,0,0,0,0,0],
  [0,0,0,2,2,2,2,0,0,2,2,2,2,0,0,0,0,0],
  [0,0,0,0,2,2,2,0,0,2,2,2,0,0,0,0,0,0],
  [0,0,0,0,2,3,2,0,0,2,3,2,0,0,0,0,0,0],
];

const GORILLA_PUMP: number[][] = GORILLA.map((r, i) => {
  if (i === 8)  return [2,2,3,3,3,3,3,3,3,3,3,3,3,3,3,2,2,0];
  if (i === 9)  return [2,2,3,4,5,5,5,5,5,5,5,5,5,4,3,2,2,0];
  if (i === 10) return [0,2,3,4,5,5,5,5,5,5,5,5,5,4,3,2,0,0];
  if (i === 11) return [0,2,3,3,4,5,5,5,5,5,5,5,4,3,3,2,0,0];
  return r;
});

const COIN: number[][] = [
  [0,12,12,12,12,12,0],
  [12,20,20,20,20,20,12],
  [12,20,12,20,12,20,12],
  [12,20,20,20,20,20,12],
  [0,12,12,12,12,12,0],
  [0,0,21,21,0,0,0],
];

const FLAME_FRAMES: number[][][] = [
  [[0,0,14,0,0],[0,14,15,14,0],[14,15,13,14,0],[14,13,20,14,0],[0,14,13,14,0],[0,13,14,13,0]],
  [[0,14,0,14,0],[14,15,14,0,0],[14,13,15,14,0],[0,14,13,14,0],[0,13,14,0,0],[0,14,13,0,0]],
  [[0,14,14,0,0],[0,14,15,14,0],[14,15,13,15,0],[14,13,14,13,0],[0,14,13,14,0],[0,0,14,13,0]],
];

// ─── Themes ───────────────────────────────────────────────────────────────────
const THEMES = {
  NIGHT: {
    bg: '#030a14', panel: 'rgba(4,12,26,.97)', accent: '#00e5ff',
    dim: '#00e5ff22', glow: '0 0 14px #00e5ff55', border: '#00e5ff33',
    termBg: '#020810', termFg: '#839496', termCursor: '#00e5ff',
    termSel: '#0a2d38',
  },
  DAY: {
    bg: '#071a0a', panel: 'rgba(6,20,10,.97)', accent: '#44ff88',
    dim: '#44ff8822', glow: '0 0 14px #44ff8855', border: '#44ff8833',
    termBg: '#040e06', termFg: '#93a1a1', termCursor: '#44ff88',
    termSel: '#0a2d12',
  },
  LAVA: {
    bg: '#0e0003', panel: 'rgba(18,2,4,.97)', accent: '#ff4400',
    dim: '#ff440022', glow: '0 0 14px #ff440055', border: '#ff440033',
    termBg: '#080002', termFg: '#839496', termCursor: '#ff4400',
    termSel: '#2d0a0a',
  },
  VOID: {
    bg: '#06001a', panel: 'rgba(8,2,28,.97)', accent: '#aa44ff',
    dim: '#aa44ff22', glow: '0 0 14px #aa44ff55', border: '#aa44ff33',
    termBg: '#040012', termFg: '#839496', termCursor: '#aa44ff',
    termSel: '#1a0a2d',
  },
} as const;
type ThemeKey = keyof typeof THEMES;
const THEME_KEYS = Object.keys(THEMES) as ThemeKey[];

// ─── CSS keyframes ────────────────────────────────────────────────────────────
const KEYFRAMES = `
@keyframes idle    { 0%,100%{transform:translateY(0)} 50%{transform:translateY(-4px)} }
@keyframes pump    { 0%{transform:scale(1)} 30%{transform:scale(1.12) translateY(-7px)} 70%{transform:scale(.97)} 100%{transform:scale(1)} }
@keyframes shake   { 0%,100%{transform:translateX(0)} 20%{transform:translateX(-4px)} 40%{transform:translateX(4px)} 60%{transform:translateX(-3px)} 80%{transform:translateX(3px)} }
@keyframes rise    { 0%{transform:translateY(0) scale(1);opacity:1} 100%{transform:translateY(-80px) scale(.3);opacity:0} }
@keyframes blink   { 0%,100%{opacity:1} 50%{opacity:0} }
@keyframes twinkle { 0%,100%{opacity:.7} 50%{opacity:.1} }
@keyframes flicker { 0%,100%{transform:scaleY(1)} 50%{transform:scaleY(1.12) scaleX(.9)} }
@keyframes scan    { 0%{opacity:.02} 50%{opacity:.05} 100%{opacity:.02} }
@keyframes fadein  { from{opacity:0;transform:translateY(6px)} to{opacity:1;transform:none} }
@keyframes errflash{ 0%,100%{box-shadow:none} 50%{box-shadow:0 0 30px #ff444488 inset} }
@keyframes slideup { from{opacity:0;transform:translateY(20px)} to{opacity:1;transform:none} }
@keyframes bootglow{ 0%,100%{text-shadow:0 0 8px currentColor} 50%{text-shadow:0 0 22px currentColor,0 0 40px currentColor} }
@keyframes spin    { to{transform:rotate(360deg)} }
@keyframes pulse-dot { 0%,100%{box-shadow:0 0 4px currentColor} 50%{box-shadow:0 0 12px currentColor} }
.gm-scen-btn { cursor:pointer; transition:all .12s; font-family:"Courier New",monospace; }
.gm-scen-btn:hover { transform:translateY(-1px); }
.gm-scen-btn:disabled { opacity:.35; cursor:not-allowed; transform:none; }
.gm-xterm .xterm-viewport { overflow-y:hidden !important; }
.gm-scrollbar { scrollbar-width:thin; }
`;

// ─── Sub-components ───────────────────────────────────────────────────────────

function Sprite({ data, scale = 1 }: { data: number[][], scale?: number }) {
  const ps = PS * scale;
  return (
    <svg
      width={data[0].length * ps}
      height={data.length * ps}
      style={{ imageRendering: 'pixelated', display: 'block' }}
    >
      {data.flatMap((row, y) =>
        row.map((c, x) =>
          c ? <rect key={`${x},${y}`} x={x * ps} y={y * ps} width={ps} height={ps} fill={PAL[c]} /> : null,
        ),
      )}
    </svg>
  );
}

function HudBar({ pct, color, label, val }: { pct: number; color: string; label: string; val: string }) {
  return (
    <div style={{ marginBottom: 9 }}>
      <div style={{ display: 'flex', justifyContent: 'space-between', fontSize: 8, letterSpacing: 2, color: 'rgba(255,255,255,.35)', marginBottom: 3 }}>
        <span>{label}</span>
        <span style={{ color }}>{val}</span>
      </div>
      <div style={{ height: 5, background: 'rgba(255,255,255,.07)', position: 'relative' }}>
        <div style={{ position: 'absolute', top: 0, left: 0, height: '100%', width: `${Math.max(0, Math.min(100, pct))}%`, background: color, transition: 'width .4s ease', boxShadow: `0 0 6px ${color}` }} />
      </div>
    </div>
  );
}

function BootScreen({ line, accent }: { line: string; accent: string }) {
  return (
    <div style={{ width: '100%', height: '100%', background: '#030a14', display: 'flex', flexDirection: 'column', alignItems: 'center', justifyContent: 'center', fontFamily: '"Courier New",monospace' }}>
      <div style={{ marginBottom: 28 }}>
        <Sprite data={GORILLA} scale={1.4} />
      </div>
      <div style={{ fontSize: 9, letterSpacing: 6, color: accent, opacity: .4, marginBottom: 12 }}>INNERANIMAL MEDIA</div>
      <div style={{ fontSize: 13, letterSpacing: 2, color: accent, animation: 'bootglow 1.4s ease-in-out infinite' }}>
        {line}<span style={{ animation: 'blink .7s infinite' }}>_</span>
      </div>
    </div>
  );
}

// ─── AI Buddy Panel ───────────────────────────────────────────────────────────
interface BuddyPanelProps {
  open: boolean;
  onClose: () => void;
  terminalBuffer: () => string;
  theme: typeof THEMES[ThemeKey];
  workspaceId?: string;
}

function BuddyPanel({ open, onClose, terminalBuffer, theme: T, workspaceId }: BuddyPanelProps) {
  const [input, setInput] = useState('');
  const [messages, setMessages] = useState<{ role: 'user' | 'sam'; text: string }[]>([
    { role: 'sam', text: 'Agent Sam online. I have your terminal context. Ask me anything or pick a quick action.' },
  ]);
  const [loading, setLoading] = useState(false);
  const scrollRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (scrollRef.current) scrollRef.current.scrollTop = scrollRef.current.scrollHeight;
  }, [messages, loading]);

  const QUICK = [
    { label: 'Explain last error', prompt: 'Explain the most recent error in my terminal output.' },
    { label: 'Suggest fix', prompt: 'Based on my terminal output, what is the most likely fix?' },
    { label: 'Check D1 status', prompt: 'Are there any D1 database issues visible in my terminal output?' },
    { label: 'Summarize session', prompt: 'Give me a 3-line summary of what happened in this terminal session.' },
  ];

  const send = useCallback(async (text: string) => {
    if (!text.trim() || loading) return;
    const userMsg = text.trim();
    setInput('');
    setMessages(p => [...p, { role: 'user', text: userMsg }]);
    setLoading(true);

    const ctx = terminalBuffer().slice(-3000);
    const systemPrompt = `You are Agent Sam, the AI assistant for Inner Animal Media's developer dashboard. You have access to the user's terminal output context. Be concise, technical, and direct. Workspace: ${workspaceId ?? 'ws_inneranimalmedia'}.`;
    const fullPrompt = ctx
      ? `Terminal context (last session output):\n\`\`\`\n${ctx}\n\`\`\`\n\nUser: ${userMsg}`
      : userMsg;

    try {
      const res = await fetch('/api/agent/chat', {
        method: 'POST',
        credentials: 'same-origin',
        headers: { 'Content-Type': 'application/json', Accept: 'text/event-stream' },
        body: JSON.stringify({
          messages: [{ role: 'user', content: fullPrompt }],
          system: systemPrompt,
          stream: true,
          model_tier: 'T1',
        }),
      });

      if (!res.ok || !res.body) {
        throw new Error(`${res.status}`);
      }

      let samText = '';
      setMessages(p => [...p, { role: 'sam', text: '' }]);

      const reader = res.body.getReader();
      const decoder = new TextDecoder();

      while (true) {
        const { done, value } = await reader.read();
        if (done) break;
        const chunk = decoder.decode(value, { stream: true });
        const lines = chunk.split('\n');
        for (const line of lines) {
          if (!line.startsWith('data: ')) continue;
          const data = line.slice(6).trim();
          if (data === '[DONE]') break;
          try {
            const parsed = JSON.parse(data) as { delta?: { text?: string }; type?: string };
            const token = parsed.delta?.text ?? '';
            samText += token;
            setMessages(p => {
              const next = [...p];
              next[next.length - 1] = { role: 'sam', text: samText };
              return next;
            });
          } catch { /* skip malformed SSE chunks */ }
        }
      }
    } catch (err) {
      setMessages(p => [...p, { role: 'sam', text: `Error reaching Agent Sam: ${err instanceof Error ? err.message : String(err)}` }]);
    } finally {
      setLoading(false);
    }
  }, [loading, terminalBuffer, workspaceId]);

  if (!open) return null;

  return (
    <div style={{
      position: 'absolute', bottom: 0, right: 0, top: 0, width: 320,
      background: T.panel, borderLeft: `1px solid ${T.border}`,
      display: 'flex', flexDirection: 'column', zIndex: 20,
      animation: 'slideup .18s ease',
    }}>
      {/* Header */}
      <div style={{ padding: '10px 14px', borderBottom: `1px solid ${T.border}`, display: 'flex', alignItems: 'center', gap: 8, flexShrink: 0 }}>
        <Bot size={13} color={T.accent} />
        <span style={{ fontSize: 10, letterSpacing: 3, color: T.accent, fontFamily: '"Courier New",monospace', fontWeight: 900 }}>AGENT SAM</span>
        <div style={{ marginLeft: 'auto', display: 'flex', gap: 4, alignItems: 'center' }}>
          <div style={{ width: 6, height: 6, borderRadius: '50%', background: T.accent, animation: 'pulse-dot 2s infinite', boxShadow: T.glow }} />
          <button onClick={onClose} style={{ background: 'none', border: 'none', cursor: 'pointer', padding: 2, color: 'rgba(255,255,255,.4)', display: 'flex' }}>
            <X size={13} />
          </button>
        </div>
      </div>

      {/* Quick actions */}
      <div style={{ padding: '8px 12px', borderBottom: `1px solid ${T.border}`, display: 'flex', flexWrap: 'wrap', gap: 5, flexShrink: 0 }}>
        {QUICK.map(q => (
          <button key={q.label} onClick={() => send(q.prompt)} disabled={loading}
            style={{ padding: '3px 8px', background: T.dim, border: `1px solid ${T.border}`, color: T.accent, fontSize: 9, letterSpacing: 1, fontFamily: '"Courier New",monospace', cursor: 'pointer', opacity: loading ? .4 : 1 }}>
            {q.label}
          </button>
        ))}
      </div>

      {/* Messages */}
      <div ref={scrollRef} className="gm-scrollbar" style={{ flex: 1, overflowY: 'auto', padding: '12px 14px', display: 'flex', flexDirection: 'column', gap: 10 }}>
        {messages.map((m, i) => (
          <div key={i} style={{ animation: 'fadein .15s ease' }}>
            <div style={{ fontSize: 8, letterSpacing: 2, color: m.role === 'sam' ? T.accent : 'rgba(255,255,255,.4)', marginBottom: 4, fontFamily: '"Courier New",monospace' }}>
              {m.role === 'sam' ? '>> SAM' : '>> YOU'}
            </div>
            <div style={{ fontSize: 11, lineHeight: 1.65, color: m.role === 'sam' ? 'rgba(255,255,255,.85)' : 'rgba(255,255,255,.6)', fontFamily: '"Courier New",monospace', whiteSpace: 'pre-wrap', wordBreak: 'break-word' }}>
              {m.text}
              {i === messages.length - 1 && loading && m.role === 'sam' && (
                <span style={{ animation: 'blink .7s infinite' }}>▋</span>
              )}
            </div>
          </div>
        ))}
        {loading && messages[messages.length - 1]?.role === 'user' && (
          <div style={{ display: 'flex', alignItems: 'center', gap: 6, color: T.accent, fontSize: 10, fontFamily: '"Courier New",monospace' }}>
            <Loader2 size={10} style={{ animation: 'spin 1s linear infinite' }} />
            thinking…
          </div>
        )}
      </div>

      {/* Input */}
      <div style={{ padding: '8px 12px', borderTop: `1px solid ${T.border}`, display: 'flex', gap: 6, flexShrink: 0 }}>
        <input
          value={input}
          onChange={e => setInput(e.target.value)}
          onKeyDown={e => { if (e.key === 'Enter' && !e.shiftKey) { e.preventDefault(); send(input); } }}
          disabled={loading}
          placeholder="ask sam anything…"
          style={{ flex: 1, background: 'rgba(0,0,0,.5)', border: `1px solid ${T.border}`, color: 'rgba(255,255,255,.85)', fontSize: 11, padding: '6px 8px', fontFamily: '"Courier New",monospace', outline: 'none' }}
        />
        <button onClick={() => send(input)} disabled={loading || !input.trim()}
          style={{ padding: '6px 10px', background: T.dim, border: `1px solid ${T.border}`, color: T.accent, cursor: loading ? 'not-allowed' : 'pointer', display: 'flex', alignItems: 'center', opacity: (!input.trim() || loading) ? .4 : 1 }}>
          <Send size={11} />
        </button>
      </div>
    </div>
  );
}

// ─── Constants ────────────────────────────────────────────────────────────────
const MIN_HEIGHT   = 160;
const MAX_H_RATIO  = 0.82;
const DEFAULT_HEIGHT = 340;

// ─── Main Component ───────────────────────────────────────────────────────────
export const GorillaModeShell = forwardRef<GorillaModeHandle, GorillaModeProps>(
  (
    {
      onClose,
      problems = [],
      outputLines = [],
      iamOrigin,
      workspaceCdCommand,
      agentDashboardUrl: agentDashboardUrlProp,
      showWelcomeBar = true,
      workspaceLabel = '',
      workspaceId,
      productLabel = 'Agent Sam',
    },
    ref,
  ) => {
    // ── Config resolution ────────────────────────────────────────────────────
    const [resolvedOrigin, setResolvedOrigin] = useState(iamOrigin ?? (typeof window !== 'undefined' ? window.location.origin : ''));
    const [resolvedCdCmd, setResolvedCdCmd]   = useState(workspaceCdCommand);

    const agentDashboardUrl = agentDashboardUrlProp ?? `${resolvedOrigin.replace(/\/$/, '')}/dashboard/agent`;

    useEffect(() => {
      void fetch('/api/agentsam/config', { credentials: 'same-origin' })
        .then(r => (r.ok ? r.json() : Promise.reject()))
        .then((data: { workspace_cd_command?: string; iam_origin?: string }) => {
          if (workspaceCdCommand === undefined && data.workspace_cd_command) setResolvedCdCmd(data.workspace_cd_command);
          if (iamOrigin === undefined && data.iam_origin) setResolvedOrigin(data.iam_origin);
        })
        .catch(() => {});
    }, []);

    // ── Terminal refs ────────────────────────────────────────────────────────
    const terminalRef   = useRef<HTMLDivElement>(null);
    const xtermRef      = useRef<Terminal | null>(null);
    const fitAddonRef   = useRef<FitAddon | null>(null);
    const socketRef     = useRef<WebSocket | null>(null);
    const ptySessionRef = useRef<string | null>(null);
    const bufferRef     = useRef<string>('');

    // ── UI State ─────────────────────────────────────────────────────────────
    const [themeIdx, setThemeIdx]   = useState(0);
    const [height, setHeight]       = useState(DEFAULT_HEIGHT);
    const [collapsed, setCollapsed] = useState(false);
    const [activeTab, setActiveTab] = useState<ShellTab>('terminal');
    const [wsStatus, setWsStatus]   = useState<'connecting' | 'online' | 'offline'>('connecting');
    const [sessionId, setSessionId] = useState<string | null>(null);
    const [uptime, setUptime]       = useState(0);

    // ── Game State ───────────────────────────────────────────────────────────
    const [booted, setBooted]       = useState(false);
    const [bootLine, setBootLine]   = useState('');
    const [pump, setPump]           = useState(false);
    const [errorFlash, setErrorFlash] = useState(false);
    const [coinCount, setCoinCount] = useState(0);
    const [floatCoins, setFloatCoins] = useState<{ id: number; x: number; y: number; dx: number }[]>([]);
    const [flameTick, setFlameTick] = useState(0);
    const [progress, setProgress]   = useState(0);
    const [hudStatus, setHudStatus] = useState('READY');
    const [aiOpen, setAiOpen]       = useState(false);

    // ── Tunnel ───────────────────────────────────────────────────────────────
    const [tunnelHealth, setTunnelHealth] = useState<{ healthy: boolean; connections: number } | null>(null);
    const [restarting, setRestarting]     = useState(false);

    const T = THEMES[THEME_KEYS[themeIdx]];

    // ── Stars (static) ───────────────────────────────────────────────────────
    const stars = useMemo(() =>
      Array.from({ length: 35 }, (_, i) => ({
        id: i, x: Math.random() * 100, y: Math.random() * 55,
        s: Math.random() < .2 ? 2 : 1, d: Math.random() * 4,
      })), []);

    // ── Boot sequence ────────────────────────────────────────────────────────
    useEffect(() => {
      const BOOT = [
        'GORILLA MODE v1.0',
        `>> WORKSPACE: ${workspaceLabel || 'ws_inneranimalmedia'}`,
        '>> D1: inneranimalmedia-business',
        '>> MCP TOOLS: LOADING…',
        '>> AGENT SAM: ONLINE',
        '>> READY',
      ];
      let i = 0;
      const t = setInterval(() => {
        if (i < BOOT.length) { setBootLine(BOOT[i]); i++; }
        else { clearInterval(t); setTimeout(() => setBooted(true), 300); }
      }, 320);
      return () => clearInterval(t);
    }, [workspaceLabel]);

    // ── Flame tick ───────────────────────────────────────────────────────────
    useEffect(() => {
      const t = setInterval(() => setFlameTick(n => (n + 1) % 3), 190);
      return () => clearInterval(t);
    }, []);

    // ── Uptime ───────────────────────────────────────────────────────────────
    useEffect(() => {
      if (wsStatus !== 'online') { setUptime(0); return; }
      const t = setInterval(() => setUptime(s => s + 1), 1000);
      return () => clearInterval(t);
    }, [wsStatus]);

    const fmtUptime = (s: number) => {
      const h = Math.floor(s / 3600);
      const m = Math.floor((s % 3600) / 60);
      const sec = s % 60;
      return h > 0 ? `${h}h${String(m).padStart(2,'0')}m` : `${String(m).padStart(2,'0')}:${String(sec).padStart(2,'0')}`;
    };

    // ── Game helpers ─────────────────────────────────────────────────────────
    const appendBuffer = useCallback((text: string) => {
      bufferRef.current = (bufferRef.current + text).slice(-8000);
      // Scan for success/error patterns in PTY output
      const lower = text.toLowerCase();
      if (/deployed successfully|✓|gate passed|benchmark.*pass/i.test(text)) {
        setPump(true); setTimeout(() => setPump(false), 700);
        spawnCoins(4);
        setProgress(100);
      }
      if (/error|failed|✗|exception/i.test(lower)) {
        setErrorFlash(true); setTimeout(() => setErrorFlash(false), 600);
      }
    }, []);

    const spawnCoins = useCallback((n: number) => {
      if (!n) return;
      const batch = Array.from({ length: n }, (_, i) => ({
        id: Date.now() + i,
        x: 5 + Math.random() * 20,
        y: 40 + Math.random() * 15,
        dx: (Math.random() - .5) * 50,
      }));
      setFloatCoins(p => [...p, ...batch]);
      setCoinCount(c => c + n);
      setTimeout(() => setFloatCoins(p => p.filter(c => !batch.find(b => b.id === c.id))), 1400);
    }, []);

    const triggerPump  = useCallback(() => { setPump(true); setTimeout(() => setPump(false), 700); spawnCoins(2); }, [spawnCoins]);
    const triggerError = useCallback(() => { setErrorFlash(true); setTimeout(() => setErrorFlash(false), 600); }, []);

    // ── Tunnel status ────────────────────────────────────────────────────────
    const fetchTunnelStatus = useCallback(() => {
      void fetch('/api/tunnel/status', { credentials: 'same-origin' })
        .then(r => r.json())
        .then((j: { healthy?: boolean; connections?: number }) => {
          setTunnelHealth({ healthy: j?.healthy === true, connections: j?.connections ?? 0 });
        })
        .catch(() => setTunnelHealth(null));
    }, []);

    const handleTunnelRestart = useCallback(async () => {
      setRestarting(true);
      xtermRef.current?.writeln('\r\n\x1b[38;5;208m  ◌ Requesting tunnel restart…\x1b[0m');
      try {
        const res = await fetch('/api/tunnel/restart', { method: 'POST', credentials: 'same-origin' });
        const data = await res.json() as { ok?: boolean; error?: string };
        if (data.ok) {
          xtermRef.current?.writeln('\x1b[38;5;82m  ✓ Restart requested — rechecking in 4s…\x1b[0m');
          setTimeout(fetchTunnelStatus, 4000);
        } else {
          xtermRef.current?.writeln(`\x1b[38;5;196m  ✗ ${data.error ?? 'Failed'}\x1b[0m`);
        }
      } catch (e) {
        xtermRef.current?.writeln(`\x1b[38;5;196m  ✗ ${e instanceof Error ? e.message : String(e)}\x1b[0m`);
      } finally {
        setRestarting(false);
      }
    }, [fetchTunnelStatus]);

    // ── WebSocket connect ────────────────────────────────────────────────────
    useEffect(() => {
      if (collapsed || activeTab !== 'terminal') return;
      let mounted = true;

      const connect = async () => {
        try {
          ptySessionRef.current = null;
          setSessionId(null);
          setWsStatus('connecting');
          setHudStatus('CONNECTING');

          const [socketPack, resumeJson, cfgJson] = await Promise.all([
            fetch('/api/agent/terminal/socket-url', { credentials: 'same-origin', headers: { Accept: 'application/json' } })
              .then(async r => ({ r, j: await r.json().catch(() => ({})) as Record<string, unknown> })),
            fetch('/api/terminal/session/resume', { credentials: 'same-origin' })
              .then(r => r.json().catch(() => ({ resumable: false }))),
            fetch('/api/agent/terminal/config-status', { credentials: 'same-origin' })
              .then(r => r.json().catch(() => ({}))),
          ]);

          if (!mounted) return;

          const url = (socketPack.j as { url?: string }).url;
          if (!socketPack.r.ok || !url) {
            if (mounted) { setWsStatus('offline'); setHudStatus('OFFLINE'); }
            xtermRef.current?.writeln(`\r\n\x1b[1;31m  ✗ Terminal URL failed: ${(socketPack.j as { error?: string }).error ?? socketPack.r.status}\x1b[0m`);
            return;
          }

          const ws = new WebSocket(url);
          socketRef.current = ws;

          ws.onopen = () => {
            if (!mounted) return;
            setWsStatus('online');
            setHudStatus('ONLINE');
            setProgress(100);

            if (xtermRef.current) {
              const term = xtermRef.current;
              term.clear();
              // ANSI welcome banner
              const C = '\x1b[38;5;51m', Y = '\x1b[38;5;226m', DK = '\x1b[38;5;238m',
                    MD = '\x1b[38;5;244m', R = '\x1b[0m', B = '\x1b[1m', DM = '\x1b[2m';

              const wsLine = (workspaceLabel || 'Workspace').slice(0, 38);
              const banner = [
                '',
                `${DK}  ╔════════════════════════════════════════════════════╗${R}`,
                `${B}${Y}  ██╗ █████╗ ███╗   ███╗${R}`,
                `${B}${Y}  ██║██╔══██╗████╗ ████║${R}`,
                `${B}${Y}  ██║███████║██╔████╔██║${R}`,
                `${B}${Y}  ██║██╔══██║██║╚██╔╝██║${R}`,
                `${B}${Y}  ╚═╝╚═╝  ╚═╝╚═╝     ╚═╝${R}`,
                ``,
                `${B}${C}  ██████╗  ██████╗ ██████╗ ██╗██╗     ██╗      █████╗ ${R}`,
                `${B}${C}  ██╔════╝ ██╔═══██╗██╔══██╗██║██║     ██║     ██╔══██╗${R}`,
                `${B}${C}  ██║  ███╗██║   ██║██████╔╝██║██║     ██║     ███████║${R}`,
                `${B}${C}  ██║   ██║██║   ██║██╔══██╗██║██║     ██║     ██╔══██║${R}`,
                `${B}${C}  ╚██████╔╝╚██████╔╝██║  ██║██║███████╗███████╗██║  ██║${R}`,
                `${B}${C}   ╚═════╝  ╚═════╝ ╚═╝  ╚═╝╚═╝╚══════╝╚══════╝╚═╝  ╚═╝${R}`,
                ``,
                `${DM}  Workspace : ${wsLine}${R}`,
                workspaceId ? `${DM}  ID        : ${workspaceId}${R}` : '',
                `${DM}  PTY       : iam-pty bridge active${R}`,
                `${DK}  ╚════════════════════════════════════════════════════╝${R}`,
                '',
              ].filter(Boolean).join('\r\n');

              term.writeln(banner);

              const cfgOk = (cfgJson as { terminal_configured?: boolean }).terminal_configured === true;
              term.writeln(`  ${cfgOk ? '\x1b[38;5;82m◈' : '\x1b[38;5;196m◈'}\x1b[0m Worker: TERMINAL_WS_URL + TERMINAL_SECRET ${cfgOk ? '\x1b[38;5;82mOK\x1b[0m' : '\x1b[38;5;196mMISSING\x1b[0m'}`);

              if ((resumeJson as { resumable?: boolean }).resumable) {
                const sid = (resumeJson as { session_id?: string }).session_id ?? '';
                term.writeln(`  \x1b[38;5;240m◈ Resumed: session ${sid.slice(0, 8)}…\x1b[0m`);
              }

              // Key handler
              const onData = term.onData(data => {
                if (ws.readyState === WebSocket.OPEN) ws.send(data);
                // Ctrl+A → toggle AI buddy
                if (data === '\x01') setAiOpen(v => !v);
              });
              const onResize = term.onResize(({ cols, rows }) => {
                if (ws.readyState === WebSocket.OPEN) ws.send(JSON.stringify({ type: 'resize', cols, rows }));
              });

              // Memory greeting
              void fetch('/api/agent/memory/list', { credentials: 'same-origin' })
                .then(r => r.json())
                .then((data: unknown) => {
                  const items = Array.isArray(data) ? data as { key?: string; value?: string }[] : [];
                  const greeting = items.find(m => m.key === 'STARTUP_GREETING')?.value;
                  if (greeting) term.writeln(`\r\n\x1b[1;36m  › ${greeting}\x1b[0m`);
                  fetchTunnelStatus();
                })
                .catch(() => fetchTunnelStatus());

              ws.onclose = (e) => {
                onData.dispose();
                onResize.dispose();
                if (mounted) { setWsStatus('offline'); setHudStatus('OFFLINE'); setProgress(0); }
                xtermRef.current?.writeln('\r\n\x1b[1;31m  ✗ Connection closed. Press Ctrl+A for AI assist.\x1b[0m');
              };
            }
          };

          ws.onmessage = (event) => {
            try {
              const msg = JSON.parse(event.data as string) as { type?: string; session_id?: string; data?: string };
              if (msg.type === 'session_id' && msg.session_id) {
                ptySessionRef.current = msg.session_id;
                if (mounted) setSessionId(msg.session_id);
                return;
              }
              if (msg.type === 'output') {
                const text = msg.data ?? '';
                appendBuffer(text);
                xtermRef.current?.write(text);
                return;
              }
            } catch { /* not JSON */ }
            appendBuffer(event.data as string);
            xtermRef.current?.write(event.data as string);
          };

          ws.onerror = () => { if (mounted) { setWsStatus('offline'); setHudStatus('OFFLINE'); } };

        } catch {
          if (mounted) { setWsStatus('offline'); setHudStatus('ERROR'); }
        }
      };

      connect();

      return () => {
        mounted = false;
        socketRef.current?.close();
        socketRef.current = null;
      };
    }, [collapsed, activeTab, workspaceLabel, workspaceId, appendBuffer, fetchTunnelStatus]);

    // ── Terminal init ────────────────────────────────────────────────────────
    useEffect(() => {
      if (!terminalRef.current || collapsed || activeTab !== 'terminal') return;

      const term = new Terminal({
        theme: {
          background: T.termBg, foreground: T.termFg,
          cursor: T.termCursor, selectionBackground: T.termSel,
          black: '#002b36', brightBlack: '#657b83',
          red: '#dc322f', brightRed: '#cb4b16',
          green: '#859900', brightGreen: '#586e75',
          yellow: '#b58900', brightYellow: '#657b83',
          blue: '#268bd2', brightBlue: '#839496',
          magenta: '#d33682', brightMagenta: '#6c71c4',
          cyan: '#2aa198', brightCyan: '#93a1a1',
          white: '#eee8d5', brightWhite: '#fdf6e3',
        },
        fontFamily: '"JetBrains Mono","Fira Code","SF Mono",Menlo,monospace',
        fontSize: 12, lineHeight: 1.45,
        cursorBlink: true, cursorStyle: 'block',
        allowTransparency: true, scrollback: 5000,
      });

      term.open(terminalRef.current);
      const fitAddon = new FitAddon();
      term.loadAddon(fitAddon);
      fitAddon.fit();
      xtermRef.current  = term;
      fitAddonRef.current = fitAddon;

      const onResize = () => requestAnimationFrame(() => fitAddonRef.current?.fit());
      window.addEventListener('resize', onResize);
      const ro = new ResizeObserver(() => requestAnimationFrame(() => fitAddonRef.current?.fit()));
      if (terminalRef.current) ro.observe(terminalRef.current);

      return () => {
        window.removeEventListener('resize', onResize);
        ro.disconnect();
        term.dispose();
        xtermRef.current = null;
      };
    }, [collapsed, activeTab, T.termBg]);

    // ── Theme reactivity → xterm ─────────────────────────────────────────────
    useEffect(() => {
      if (!xtermRef.current) return;
      xtermRef.current.options.theme = {
        ...xtermRef.current.options.theme,
        background: T.termBg, foreground: T.termFg,
        cursor: T.termCursor, selectionBackground: T.termSel,
      };
    }, [themeIdx]);

    // ── Drag resize ──────────────────────────────────────────────────────────
    const handleDragStart = (e: React.MouseEvent) => {
      e.preventDefault();
      const startY = e.clientY, startH = height;
      const maxH = window.innerHeight * MAX_H_RATIO;
      const onMove = (me: MouseEvent) => {
        setHeight(Math.max(MIN_HEIGHT, Math.min(startH + (startY - me.clientY), maxH)));
        requestAnimationFrame(() => fitAddonRef.current?.fit());
      };
      const onUp = () => {
        document.removeEventListener('mousemove', onMove);
        document.removeEventListener('mouseup', onUp);
      };
      document.addEventListener('mousemove', onMove);
      document.addEventListener('mouseup', onUp);
    };

    // ── Quick actions (fire real PTY commands) ───────────────────────────────
    const sendCmd = useCallback((cmd: string) => {
      setCollapsed(false); setActiveTab('terminal');
      if (socketRef.current?.readyState === WebSocket.OPEN) {
        socketRef.current.send(cmd + '\r');
      } else {
        xtermRef.current?.writeln('\r\n\x1b[1;31m  Terminal offline — cannot run command.\x1b[0m\r\n');
      }
    }, []);

    const QUICK_ACTIONS = [
      { n: '1', label: 'Workspace', fn: () => resolvedCdCmd ? sendCmd(resolvedCdCmd) : null },
      { n: '2', label: 'Agent',     fn: () => window.open(agentDashboardUrl, '_blank', 'noopener,noreferrer') },
      { n: '3', label: 'Tools',     fn: () => { setCollapsed(false); setActiveTab('terminal'); xtermRef.current?.writeln('\r\n\x1b[38;5;51m  MCP: https://mcp.inneranimalmedia.com/mcp\x1b[0m\r\n\x1b[38;5;240m  PTY: pm2 restart iam-pty (port 3099)\x1b[0m\r\n'); } },
      { n: '4', label: 'Theme',     fn: () => setThemeIdx(i => (i + 1) % THEME_KEYS.length) },
      { n: '5', label: 'Diag',      fn: () => {
        const cmds = [
          `echo "═ ${productLabel} diagnostics ═"`,
          'node --version', 'npm --version',
          'npx wrangler --version 2>/dev/null || echo "wrangler: not found"',
          'pm2 status 2>/dev/null || echo "pm2: not found"',
        ];
        cmds.forEach((c, i) => setTimeout(() => sendCmd(c), i * 240));
      }},
    ] as const;

    // ── Expose handle ─────────────────────────────────────────────────────────
    useImperativeHandle(ref, () => ({
      writeToTerminal: (text: string) => {
        setCollapsed(false); setActiveTab('terminal');
        xtermRef.current?.writeln(`\r\n\x1b[2m${text}\x1b[0m`);
      },
      runCommand: (cmd: string) => {
        setCollapsed(false); setActiveTab('terminal');
        if (socketRef.current?.readyState === WebSocket.OPEN) {
          socketRef.current.send(cmd + '\r');
          return;
        }
        const sid = ptySessionRef.current;
        xtermRef.current?.writeln(`\r\n\x1b[33m  WS offline → POST /api/agent/terminal/run\x1b[0m`);
        void fetch('/api/agent/terminal/run', {
          method: 'POST', credentials: 'same-origin',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ command: cmd, session_id: sid }),
        })
          .then(async r => {
            const j = await r.json().catch(() => ({})) as { output?: string; command?: string; execution_id?: string; error?: string };
            if (!xtermRef.current) return;
            if (!r.ok) { xtermRef.current.writeln(`\r\n\x1b[1;31m  run ${r.status}: ${j.error ?? 'error'}\x1b[0m`); return; }
            xtermRef.current.writeln(`\r\n\x1b[36m  $ ${j.command ?? cmd}\x1b[0m`);
            const out = j.output ?? '';
            appendBuffer(out);
            xtermRef.current.writeln(out.trim() ? out : '  (no output)');
            if (j.execution_id) {
              void fetch('/api/agent/terminal/complete', {
                method: 'POST', credentials: 'same-origin',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ execution_id: j.execution_id, status: 'completed', output_text: out, exit_code: 0 }),
              }).catch(() => {});
            }
          })
          .catch(() => xtermRef.current?.writeln('\r\n\x1b[1;31m  terminal/run: network error\x1b[0m'));
      },
      setActiveTab: (t: ShellTab) => { setActiveTab(t); setCollapsed(false); },
      triggerPump,
      triggerError,
    }));

    // ── Derived ───────────────────────────────────────────────────────────────
    const errorCount   = problems.filter(p => p.severity === 'error').length;
    const warnCount    = problems.filter(p => p.severity === 'warning').length;
    const gorillaAnim  = errorFlash ? 'shake .5s ease' : pump ? 'pump .6s ease' : 'idle 2.8s ease-in-out infinite';
    const gorillaFilter = errorFlash ? 'brightness(1.5) saturate(2) hue-rotate(-20deg)' : 'none';

    // ── Pre-boot ──────────────────────────────────────────────────────────────
    if (!booted) {
      return (
        <div style={{ position: 'fixed', inset: 0, zIndex: 9999 }}>
          <style>{KEYFRAMES}</style>
          <BootScreen line={bootLine} accent={T.accent} />
        </div>
      );
    }

    // ── Main render ───────────────────────────────────────────────────────────
    return (
      <>
        <style>{KEYFRAMES}</style>

        {/* Floating coins */}
        {floatCoins.map(c => (
          <div key={c.id} style={{ position: 'fixed', left: `${c.x}%`, top: `${c.y}%`, animation: 'rise 1.3s ease-out forwards', transform: `translateX(${c.dx}px)`, pointerEvents: 'none', zIndex: 9999 }}>
            <Sprite data={COIN} scale={.9} />
          </div>
        ))}

        <div
          style={{
            position: 'relative',
            height: collapsed ? 36 : height,
            display: 'flex',
            flexDirection: 'column',
            borderTop: `1px solid ${T.accent}55`,
            boxShadow: `0 -4px 24px rgba(0,0,0,.5)`,
            background: T.bg,
            transition: 'height .25s ease',
            flexShrink: 0,
            overflow: 'hidden',
            fontFamily: '"Courier New",monospace',
          }}
        >
          {/* Stars */}
          {!collapsed && stars.map(s => (
            <div key={s.id} style={{ position: 'absolute', left: `${s.x}%`, top: `${s.y}%`, width: s.s, height: s.s, background: '#fff', animation: `twinkle ${2 + s.d}s infinite`, animationDelay: `${s.d}s`, pointerEvents: 'none', zIndex: 0 }} />
          ))}

          {/* Scanlines */}
          <div style={{ position: 'absolute', inset: 0, pointerEvents: 'none', zIndex: 1, backgroundImage: `repeating-linear-gradient(0deg,${T.dim.replace('22','06')} 0,${T.dim.replace('22','06')} 1px,transparent 1px,transparent 3px)`, animation: 'scan 5s infinite' }} />

          {/* Resize handle */}
          {!collapsed && (
            <div style={{ height: 4, cursor: 'ns-resize', zIndex: 10, flexShrink: 0, display: 'flex', alignItems: 'center', justifyContent: 'center' }} onMouseDown={handleDragStart}>
              <div style={{ width: 48, height: 2, background: T.border, borderRadius: 2 }} />
            </div>
          )}

          {/* ── Toolbar ── */}
          <div style={{ height: 36, minHeight: 36, display: 'flex', alignItems: 'center', justifyContent: 'space-between', padding: '0 8px 0 12px', borderBottom: `1px solid ${T.border}`, background: T.panel, flexShrink: 0, zIndex: 10, position: 'relative' }}>

            {/* Left */}
            <div style={{ display: 'flex', alignItems: 'center', gap: 12, minWidth: 0 }}>
              {/* Tabs */}
              <div style={{ display: 'flex', alignItems: 'stretch' }}>
                {(['terminal', 'output', 'problems'] as ShellTab[]).map(tab => {
                  const badge = tab === 'problems' && (errorCount + warnCount) > 0 ? errorCount > 0 ? errorCount : warnCount : null;
                  const active = activeTab === tab;
                  return (
                    <button key={tab} onClick={() => setActiveTab(tab)}
                      style={{ position: 'relative', padding: '0 12px', height: 36, background: 'none', border: 'none', cursor: 'pointer', color: active ? T.accent : 'rgba(255,255,255,.35)', fontSize: 9, letterSpacing: 2, fontWeight: 900, fontFamily: '"Courier New",monospace', display: 'flex', alignItems: 'center', gap: 5, textTransform: 'uppercase', transition: 'color .15s' }}>
                      {tab === 'terminal' && <TerminalIcon size={9} />}
                      {tab}
                      {badge !== null && (
                        <span style={{ padding: '1px 4px', background: 'rgba(255,68,68,.15)', color: '#ff4444', fontSize: 8, border: '1px solid rgba(255,68,68,.3)' }}>{badge}</span>
                      )}
                      {active && <span style={{ position: 'absolute', bottom: 0, left: 4, right: 4, height: 2, background: T.accent, boxShadow: T.glow }} />}
                    </button>
                  );
                })}
              </div>

              {/* WS status */}
              <div style={{ display: 'flex', alignItems: 'center', gap: 6, fontSize: 9, fontFamily: '"Courier New",monospace' }}>
                {wsStatus === 'connecting' && (
                  <span style={{ color: '#ffcc00', display: 'flex', alignItems: 'center', gap: 5 }}>
                    <span style={{ width: 6, height: 6, borderRadius: '50%', background: '#ffcc00', animation: 'blink .7s infinite', display: 'inline-block' }} />
                    CONNECTING
                  </span>
                )}
                {wsStatus === 'online' && (
                  <span style={{ color: '#44ff88', display: 'flex', alignItems: 'center', gap: 5 }}>
                    <span style={{ width: 6, height: 6, borderRadius: '50%', background: '#44ff88', boxShadow: '0 0 6px #44ff88', display: 'inline-block', animation: 'pulse-dot 2s infinite' }} />
                    ONLINE · {fmtUptime(uptime)}
                    {sessionId && <span style={{ color: 'rgba(255,255,255,.25)' }}> · {sessionId.slice(0, 6)}…</span>}
                  </span>
                )}
                {wsStatus === 'offline' && (
                  <span style={{ color: '#ff4444', display: 'flex', alignItems: 'center', gap: 5 }}>
                    <WifiOff size={9} /> OFFLINE
                  </span>
                )}
              </div>

              {/* Tunnel */}
              {tunnelHealth && (
                <div style={{ display: 'flex', alignItems: 'center', gap: 5 }}>
                  {tunnelHealth.healthy ? <Wifi size={9} color="#44ff88" /> : <WifiOff size={9} color="#ff4444" />}
                  <span style={{ fontSize: 9, color: tunnelHealth.healthy ? '#44ff88' : '#ff4444', fontFamily: '"Courier New",monospace' }}>
                    {tunnelHealth.healthy ? `TUNNEL ×${tunnelHealth.connections}` : 'TUNNEL ✗'}
                  </span>
                  <button onClick={handleTunnelRestart} disabled={restarting}
                    style={{ background: 'none', border: 'none', cursor: restarting ? 'not-allowed' : 'pointer', padding: 2, color: 'rgba(255,255,255,.35)', display: 'flex' }}>
                    <RefreshCw size={9} style={restarting ? { animation: 'spin 1s linear infinite' } : {}} />
                  </button>
                </div>
              )}
            </div>

            {/* Right */}
            <div style={{ display: 'flex', alignItems: 'center', gap: 2, flexShrink: 0 }}>
              {/* Coin counter */}
              <div style={{ display: 'flex', alignItems: 'center', gap: 4, padding: '2px 8px', border: `1px solid ${T.border}`, marginRight: 6 }}>
                <Sprite data={COIN} scale={.42} />
                <span style={{ color: T.accent, fontSize: 10, fontFamily: '"Courier New",monospace' }}>×{coinCount}</span>
              </div>

              {/* Theme cycle */}
              <button onClick={() => setThemeIdx(i => (i + 1) % THEME_KEYS.length)}
                style={{ padding: '3px 7px', background: 'transparent', border: `1px solid ${T.border}`, color: T.accent, fontSize: 8, letterSpacing: 2, cursor: 'pointer', fontFamily: '"Courier New",monospace' }}>
                {THEME_KEYS[themeIdx]}
              </button>

              {/* AI buddy */}
              <button onClick={() => setAiOpen(v => !v)} title="Agent Sam AI (Ctrl+A)"
                style={{ padding: '4px 8px', background: aiOpen ? T.dim : 'transparent', border: `1px solid ${aiOpen ? T.accent + '66' : T.border}`, color: aiOpen ? T.accent : 'rgba(255,255,255,.4)', fontSize: 9, display: 'flex', alignItems: 'center', gap: 4, cursor: 'pointer', fontFamily: '"Courier New",monospace', letterSpacing: 1 }}>
                <Bot size={11} /> SAM
              </button>

              <button onClick={() => setCollapsed(v => !v)}
                style={{ padding: 6, background: 'none', border: 'none', cursor: 'pointer', color: 'rgba(255,255,255,.4)', display: 'flex' }}>
                {collapsed ? <ChevronUp size={14} strokeWidth={2} /> : <ChevronDown size={14} strokeWidth={2} />}
              </button>
              <button onClick={onClose}
                style={{ padding: 6, background: 'none', border: 'none', cursor: 'pointer', color: 'rgba(255,255,255,.4)', display: 'flex' }}>
                <X size={14} strokeWidth={2} />
              </button>
            </div>
          </div>

          {/* ── Content area ── */}
          {!collapsed && (
            <div style={{ flex: 1, minHeight: 0, display: 'flex', overflow: 'hidden', position: 'relative', zIndex: 5 }}>

              {/* ── LEFT: Gorilla HUD ── */}
              <div style={{ width: 160, minWidth: 160, borderRight: `1px solid ${T.border}`, background: T.panel, display: 'flex', flexDirection: 'column', alignItems: 'center', padding: '12px 10px 10px', gap: 0, flexShrink: 0 }}>

                {/* Gorilla sprite */}
                <div style={{ animation: gorillaAnim, filter: gorillaFilter, marginBottom: 4 }}>
                  <Sprite data={pump ? GORILLA_PUMP : GORILLA} scale={.95} />
                </div>

                {/* Flames */}
                <div style={{ display: 'flex', justifyContent: 'center', gap: 50, marginTop: -2, marginBottom: 8 }}>
                  {[0, 1].map(i => (
                    <div key={i} style={{ animation: 'flicker .45s infinite', animationDelay: `${i * .2}s` }}>
                      <Sprite data={FLAME_FRAMES[flameTick]} scale={.8} />
                    </div>
                  ))}
                </div>

                {/* Status label */}
                <div style={{ fontSize: 8, letterSpacing: 3, color: T.accent, marginBottom: 8, textAlign: 'center', textShadow: T.glow, fontWeight: 900 }}>
                  {hudStatus}
                  {wsStatus === 'connecting' && <span style={{ animation: 'blink .7s infinite', marginLeft: 2 }}>_</span>}
                </div>

                {/* Bars */}
                <div style={{ width: '100%' }}>
                  <HudBar pct={progress} color={T.accent} label="PROGRESS" val={`${progress}%`} />
                  <HudBar pct={Math.max(0, 100 - errorCount * 10)} color="#ff4444" label="HEALTH" val={`${Math.max(0, 100 - errorCount * 10)}%`} />
                  <HudBar pct={wsStatus === 'online' ? 100 : 0} color="#44ff88" label="AGENT" val={wsStatus === 'online' ? 'RDY' : 'OFF'} />
                </div>

                {/* Workspace info */}
                <div style={{ width: '100%', marginTop: 8, paddingTop: 8, borderTop: `1px solid ${T.border}`, fontSize: 8, color: 'rgba(255,255,255,.25)', letterSpacing: 1, lineHeight: 1.9, wordBreak: 'break-all' }}>
                  <div style={{ color: 'rgba(255,255,255,.4)' }}>{(workspaceLabel || 'workspace').slice(0, 20)}</div>
                  {workspaceId && <div>{workspaceId.slice(0, 14)}…</div>}
                  <div>branch: main</div>
                </div>

                {/* Quick action buttons (numbered like game menu) */}
                <div style={{ width: '100%', marginTop: 10, display: 'flex', flexDirection: 'column', gap: 3 }}>
                  {QUICK_ACTIONS.map(({ n, label, fn }) => (
                    <button key={n} className="gm-scen-btn" onClick={fn}
                      style={{ padding: '4px 6px', background: 'transparent', border: `1px solid ${T.border}`, color: 'rgba(255,255,255,.5)', fontSize: 9, letterSpacing: 1, textAlign: 'left', display: 'flex', gap: 5 }}>
                      <span style={{ color: T.accent }}>{n}.</span>{label}
                    </button>
                  ))}
                </div>
              </div>

              {/* ── RIGHT: Terminal + tabs ── */}
              <div style={{ flex: 1, display: 'flex', flexDirection: 'column', minHeight: 0, minWidth: 0 }}>

                {/* Terminal tab */}
                {activeTab === 'terminal' && (
                  <div
                    ref={terminalRef}
                    className="gm-xterm"
                    style={{ flex: 1, minHeight: 0, width: '100%', background: T.termBg, animation: errorFlash ? 'errflash .5s ease' : 'none' }}
                  />
                )}

                {/* Output tab */}
                {activeTab === 'output' && (
                  <div className="gm-scrollbar" style={{ flex: 1, overflowY: 'auto', padding: '14px 18px', fontFamily: '"Courier New",monospace', fontSize: 11, lineHeight: 1.7, color: 'rgba(255,255,255,.75)', background: T.termBg }}>
                    {outputLines.length === 0
                      ? <p style={{ color: 'rgba(255,255,255,.2)', fontStyle: 'italic' }}>No output yet.</p>
                      : outputLines.map((line, i) => (
                        <div key={i} style={{ paddingLeft: 8, borderLeft: `2px solid transparent`, marginBottom: 3 }}>{line}</div>
                      ))
                    }
                  </div>
                )}

                {/* Problems tab */}
                {activeTab === 'problems' && (
                  <div className="gm-scrollbar" style={{ flex: 1, overflowY: 'auto', padding: 16, background: T.termBg, display: 'flex', flexDirection: 'column', gap: 8 }}>
                    {problems.length === 0 ? (
                      <div style={{ display: 'flex', flexDirection: 'column', alignItems: 'center', justifyContent: 'center', height: '100%', color: 'rgba(255,255,255,.2)', gap: 8 }}>
                        <CircleCheck size={24} />
                        <p style={{ fontSize: 11, fontFamily: '"Courier New",monospace' }}>NO PROBLEMS DETECTED</p>
                      </div>
                    ) : (
                      problems.map((p, i) => (
                        <div key={i} style={{ padding: '8px 10px', background: 'rgba(0,0,0,.4)', borderLeft: `2px solid ${p.severity === 'error' ? '#ff4444' : '#ffcc00'}`, display: 'flex', gap: 8, alignItems: 'flex-start' }}>
                          <TriangleAlert size={12} color={p.severity === 'error' ? '#ff4444' : '#ffcc00'} style={{ flexShrink: 0, marginTop: 1 }} />
                          <div>
                            <div style={{ fontSize: 11, color: 'rgba(255,255,255,.85)', fontFamily: '"Courier New",monospace' }}>{p.msg}</div>
                            <div style={{ fontSize: 10, color: 'rgba(255,255,255,.35)', fontFamily: '"Courier New",monospace' }}>{p.file}:{p.line}</div>
                          </div>
                        </div>
                      ))
                    )}
                  </div>
                )}
              </div>

              {/* ── AI Buddy Panel ── */}
              <BuddyPanel
                open={aiOpen}
                onClose={() => setAiOpen(false)}
                terminalBuffer={() => bufferRef.current}
                theme={T}
                workspaceId={workspaceId}
              />
            </div>
          )}
        </div>
      </>
    );
  },
);

GorillaModeShell.displayName = 'GorillaModeShell';
