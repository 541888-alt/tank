<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>坦克对战 — Tank Battle</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    background: #2a2418;
    color: #f6efdd;
    font-family: "Zpix", "Fusion Pixel", "方正像素", Consolas, "Microsoft YaHei", monospace;
    display: flex; flex-direction: column;
    align-items: center; justify-content: center;
    min-height: 100vh; gap: 10px;
    user-select: none;
    image-rendering: pixelated;
  }
  #hud {
    display: flex; gap: 20px; align-items: center;
    font-size: 13.5px; letter-spacing: .5px;
    background: #5c5638;
    padding: 8px 18px;
    border: 3px solid #2a2418;         /* 像素风：直角 + 厚描边 */
    box-shadow: inset 0 -4px 0 #45412c;
  }
  #hud b { color: #ffd84d; font-weight: 600; }
  #hud .warn { color: #ff8a5c; }
  #hud .ok { color: #b6e06a; }
  canvas {
    background: #f0d9a8;
    border: 4px solid #2a2418;         /* 像素风：无圆角 */
    cursor: crosshair;
    display: block;
    image-rendering: pixelated;
    box-shadow: 0 6px 0 #1b1810;
  }
  #tip { font-size: 12.5px; color: #b8a985; }
  kbd {
    background: #5c5638; border: 2px solid #2a2418;
    padding: 1px 6px; font-size: 12px; color: #fff6d8;
  }
</style>
</head>
<body>

<div id="hud">
  <span>炮弹 <b id="hudShells">0 / 5</b></span>
  <span>冷却 <b id="hudCool">就绪</b></span>
  <span>敌军 <b id="hudEnemy">1</b></span>
  <span>空中弹 <b id="hudLive">0</b></span>
  <span>种子 <b id="hudSeed">—</b></span>
  <span>胜利 <b id="hudWins">0</b></span>
</div>

<canvas id="cv" width="960" height="640"></canvas>

<div id="tip">
  <kbd>W</kbd><kbd>A</kbd><kbd>S</kbd><kbd>D</kbd> 移动　·　鼠标瞄准　·　<kbd>右键</kbd> 发射　·　<kbd>R</kbd> 重开　·　
  <span style="color:#8a6a1f">炮弹可反弹，最多 3 次</span>
</div>

<script>
"use strict";

/* =========================================================================
   1. 配置
   ========================================================================= */
const CFG = {
  W: 960, H: 640,
  CELL: 20,                     // 寻路网格单元（20px ≈ 坦克车身宽度）
  TANK_R: 16,                   // 坦克碰撞半径
  PLAYER_SPEED: 175,            // 像素/秒
  PLAYER_TURN: 4.6,             // 车头跟随鼠标的最大转速（弧度/秒）
  NPC_SPEED: 96,
  SHELL_SPEED: 430,
  SHELL_R: 3.5,
  MAX_BOUNCES: 3,               // 炮弹最多反弹次数（撞墙/障碍各计一次）
  BOUNCE_KEEP: 0.94,            // 每次反弹后保留的速度比例（略微衰减）
  MAX_SHELLS: 5,                // 炮弹一次上限 5 发（玩家与 NPC 同一套规则）
  COOLDOWN_MS: 10000,           // 打空后冷却 10 秒才能续弹
  NPC_BURST_GAP: 140,           // NPC 每次扣扳机之间的硬下限（毫秒），避免同一帧打出多发
  NPC_PAUSE_MIN: 620,           // NPC 两发之间的思考间隔（毫秒）——不是连发，是"打一发看一发"
  NPC_PAUSE_MAX: 1350,
  NPC_MISS_PAUSE: 520,          // 上一发没打中 → 额外多等这么久再打（相当于在调整）
  NPC_LOS_RANGE: 700,           // 有视线时认为可以直射的距离
  NPC_AIM_TOL: 0.075,           // 瞄准方向与目标方向的允许误差（弧度，约 4.3°）
  NPC_BANK_MAX_DIST: 320,       // 尝试反弹射击时，目标点距 NPC 的最大距离
  NPC_BANK_COOLDOWN: 3200,      // 两次"反弹射击"之间的最小间隔（这种机会不会连续出现）
  AIM_SENSITIVITY: 14,          // 炮塔跟随鼠标的灵敏度（越大越跟手；1 = 完全平滑）
  NPC_FIRE_RANGE: 620,
  OBSTACLE_TRIES: 30,           // 生成尝试次数（保证可达）
  COUNTDOWN_SECONDS: 3,         // 每局开始前的倒计时秒数
  RESULT_WIN_DELAY: 2000,       // 胜利后停留多久再自动进入下一局（毫秒）
  RESULT_LOSE_DELAY: 2500,      // 失败后停留多久再返回主界面（毫秒）
};

/* =========================================================================
   2. 工具
   ========================================================================= */
function mulberry32(a) {
  return function () {
    a |= 0; a = a + 0x6D2B79F5 | 0;
    let t = Math.imul(a ^ a >>> 15, 1 | a);
    t = t + Math.imul(t ^ t >>> 7, 61 | t) ^ t;
    return ((t ^ t >>> 14) >>> 0) / 4294967296;
  };
}
const clamp = (v, a, b) => v < a ? a : v > b ? b : v;

/* =========================================================================
   3. 关卡生成：随机障碍 + 连通性校验（保证玩家 ↔ NPC 可达）
   ========================================================================= */
const SPAWN_MARGIN = 62;
const PLAYER_SPAWN = { x: SPAWN_MARGIN, y: CFG.H - SPAWN_MARGIN };          // 左下
const NPC_SPAWN    = { x: CFG.W - SPAWN_MARGIN, y: SPAWN_MARGIN };          // 右上

function makeObstacles(rnd) {
  const list = [];
  const count = 9 + Math.floor(rnd() * 6);          // 9–14 个障碍
  let guard = 0;
  while (list.length < count && guard++ < 300) {
    const type = rnd() < 0.55 ? "rect" : "circle";
    if (type === "rect") {
      const w = 40 + rnd() * 150;
      const h = 34 + rnd() * 130;
      const o = {
        kind: "rect",
        x: 60 + rnd() * (CFG.W - 120 - w),
        y: 60 + rnd() * (CFG.H - 120 - h),
        w, h,
      };
      o.cx = o.x + w / 2; o.cy = o.y + h / 2;
      o.rr = Math.hypot(w, h) / 2;                  // 外接圆半径（寻路时保守占用）
      list.push(o);
    } else {
      const r = 22 + rnd() * 34;
      const o = {
        kind: "circle",
        cx: 60 + r + rnd() * (CFG.W - 120 - 2 * r),
        cy: 60 + r + rnd() * (CFG.H - 120 - 2 * r),
        r,
      };
      o.rr = r;
      list.push(o);
    }
  }
  return list;
}

/** 点是否落在障碍内（用于出生点校验） */
function pointInObstacle(px, py, o, pad) {
  pad = pad || 0;
  if (o.kind === "circle") {
    return Math.hypot(px - o.cx, py - o.cy) < o.r + pad;
  }
  return px > o.x - pad && px < o.x + o.w + pad && py > o.y - pad && py < o.y + o.h + pad;
}

/** 圆（坦克/炮弹）与障碍是否相交。加 0.05 容差，避免浮点误差让炮弹从"贴边"处漏过 */
const HIT_EPS = 0.05;
function circleHitsObstacle(cx, cy, r, o) {
  if (o.kind === "circle") {
    return Math.hypot(cx - o.cx, cy - o.cy) < r + o.r - HIT_EPS;
  }
  const nx = clamp(cx, o.x, o.x + o.w);
  const ny = clamp(cy, o.y, o.y + o.h);
  return Math.hypot(cx - nx, cy - ny) < r - HIT_EPS;
}

/** 线段是否被障碍挡住（NPC 视线判定，采样法） */
function lineBlocked(x1, y1, x2, y2, obstacles) {
  const dist = Math.hypot(x2 - x1, y2 - y1);
  const steps = Math.ceil(dist / 6);
  for (let i = 1; i < steps; i++) {
    const t = i / steps;
    const px = x1 + (x2 - x1) * t;
    const py = y1 + (y2 - y1) * t;
    for (const o of obstacles) if (pointInObstacle(px, py, o, 0)) return true;
  }
  return false;
}

/** 网格寻路（8 邻域 BFS）——用外接圆保守占用，确保留出的通道真的能过 */
function buildGrid(obstacles) {
  const cols = Math.ceil(CFG.W / CFG.CELL);
  const rows = Math.ceil(CFG.H / CFG.CELL);
  const blocked = new Uint8Array(cols * rows);
  for (let gy = 0; gy < rows; gy++) {
    for (let gx = 0; gx < cols; gx++) {
      const wx = gx * CFG.CELL + CFG.CELL / 2;
      const wy = gy * CFG.CELL + CFG.CELL / 2;
      // 半径取坦克半径，保证"能过"的通道坦克真的能过
      for (const o of obstacles) {
        if (pointInObstacle(wx, wy, o, CFG.TANK_R)) { blocked[gy * cols + gx] = 1; break; }
      }
    }
  }
  return { cols, rows, blocked };
}

function cellOf(pt) {
  return {
    gx: clamp(Math.floor(pt.x / CFG.CELL), 0, Math.ceil(CFG.W / CFG.CELL) - 1),
    gy: clamp(Math.floor(pt.y / CFG.CELL), 0, Math.ceil(CFG.H / CFG.CELL) - 1),
  };
}

/** BFS：返回路径数组（世界坐标）或 null */
function findPath(grid, from, to) {
  const { cols, rows, blocked } = grid;
  const s = cellOf(from), g = cellOf(to);
  const si = s.gy * cols + s.gx, gi = g.gy * cols + g.gx;
  if (blocked[si] || blocked[gi]) return null;

  const prev = new Int32Array(cols * rows).fill(-1);
  const seen = new Uint8Array(cols * rows);
  const queue = [si];
  seen[si] = 1;
  const DIRS = [[1,0],[-1,0],[0,1],[0,-1],[1,1],[1,-1],[-1,1],[-1,-1]];

  while (queue.length) {
    const cur = queue.shift();
    if (cur === gi) break;
    const cx = cur % cols, cy = (cur - cx) / cols;
    for (const [dx, dy] of DIRS) {
      const nx = cx + dx, ny = cy + dy;
      if (nx < 0 || ny < 0 || nx >= cols || ny >= rows) continue;
      const ni = ny * cols + nx;
      if (seen[ni] || blocked[ni]) continue;
      // 斜向不允许贴角穿过
      if (dx && dy && (blocked[cy * cols + nx] || blocked[ny * cols + cx])) continue;
      seen[ni] = 1;
      prev[ni] = cur;
      queue.push(ni);
    }
  }
  if (!seen[gi]) return null;

  const path = [];
  let cur = gi;
  while (cur !== -1) {
    const cx = cur % cols, cy = (cur - cx) / cols;
    path.push({ x: cx * CFG.CELL + CFG.CELL / 2, y: cy * CFG.CELL + CFG.CELL / 2 });
    cur = prev[cur];
  }
  return path.reverse();
}

/** 生成一个"必然有解"的关卡 */
function generateLevel(seed0) {
  let seed = seed0 >>> 0;
  for (let attempt = 0; attempt < CFG.OBSTACLE_TRIES; attempt++) {
    const rnd = mulberry32(seed);
    const obstacles = makeObstacles(rnd);

    // 出生点必须干净
    const dirty =
      obstacles.some(o => pointInObstacle(PLAYER_SPAWN.x, PLAYER_SPAWN.y, o, CFG.TANK_R + 10)) ||
      obstacles.some(o => pointInObstacle(NPC_SPAWN.x, NPC_SPAWN.y, o, CFG.TANK_R + 10));
    if (dirty) { seed = (seed * 1664525 + 1013904223) >>> 0; continue; }

    const grid = buildGrid(obstacles);
    const path = findPath(grid, PLAYER_SPAWN, NPC_SPAWN);
    if (path && path.length > 6) {
      return { seed, obstacles, grid, path, attempts: attempt + 1 };
    }
    seed = (seed * 1664525 + 1013904223) >>> 0;   // 换种子重试
  }
  // 极端兜底：不放障碍
  const obstacles = [];
  const grid = buildGrid(obstacles);
  return { seed, obstacles, grid, path: findPath(grid, PLAYER_SPAWN, NPC_SPAWN), attempts: CFG.OBSTACLE_TRIES };
}

/** 线段是否与障碍实体相交（考虑炮弹半径）。两端各跳过 SKIP px，
 *  因为弹着点本来就贴在障碍表面上，不跳过会把合法解误判为穿墙。 */
function segmentHitsAnyObstacle(x1, y1, x2, y2, obstacles) {
  const dist = Math.hypot(x2 - x1, y2 - y1);
  const SKIP = CFG.SHELL_R + 6;
  if (dist <= SKIP * 2 + 2) return false;
  const steps = Math.max(3, Math.ceil(dist / 4));
  const tStart = SKIP / dist, tEnd = 1 - SKIP / dist;
  for (let i = 1; i < steps; i++) {
    const t = i / steps;
    if (t < tStart || t > tEnd) continue;
    const px = x1 + (x2 - x1) * t;
    const py = y1 + (y2 - y1) * t;
    for (const o of obstacles) {
      if (circleHitsObstacle(px, py, CFG.SHELL_R, o)) return true;
    }
  }
  return false;
}

/**
 * 解一次反射路径：从 A 出发、经某个障碍表面反弹一次后能否到达 B。
 *
 * 原理：把 A 关于该表面所在直线做镜像 A'，若线段 A'B 与该表面相交，
 * 则交点就是弹着点 P，且 A→P→B 满足"入射角 = 反射角"。
 *
 * @returns {x, y, dist} 弹着点与总路程；无解返回 null
 */
﻿function solveBankShot(ax, ay, bx, by, obstacles, maxDist) {
  // 若本就有直线通路，就不该"绕一圈"：直接返回无解，让调用方走直射分支。
  if (!lineBlocked(ax, ay, bx, by, obstacles)) return null;

  // 所有可反弹的"面"：场地四壁 + 矩形障碍的四个面
  // n 为朝外的单位法线 —— 炮弹实际是在"表面内移一个炮弹半径"处反弹的，
  // 用镜面点算方向时必须把这个偏移算进去，否则反射方向会有偏差。
  const sides = [
    { axis: "v", X: 0,     lo: 0, hi: CFG.H, nx: -1, ny: 0, owner: null },
    { axis: "v", X: CFG.W, lo: 0, hi: CFG.H, nx: 1,  ny: 0, owner: null },
    { axis: "h", X: 0,     lo: 0, hi: CFG.W, nx: 0,  ny: -1, owner: null },
    { axis: "h", X: CFG.H, lo: 0, hi: CFG.W, nx: 0,  ny: 1,  owner: null },
  ];
  for (const o of obstacles) {
    if (o.kind !== "rect") continue;
    sides.push({ axis: "v", X: o.x,       lo: o.y,       hi: o.y + o.h, nx: -1, ny: 0, owner: o });
    sides.push({ axis: "v", X: o.x + o.w, lo: o.y,       hi: o.y + o.h, nx: 1,  ny: 0, owner: o });
    sides.push({ axis: "h", X: o.y,       lo: o.x,       hi: o.x + o.w, nx: 0,  ny: -1, owner: o });
    sides.push({ axis: "h", X: o.y + o.h, lo: o.x,       hi: o.x + o.w, nx: 0,  ny: 1,  owner: o });
  }

﻿  let best = null;
  /**
   * 判定一个反弹方案。几何关系（简洁表述）：
   *   ① 把目标 B 关于该表面镜射成 B'
   *   ② 直线 A→B' 与表面的交点就是弹着点 Q
   *   ③ 瞄准点 P 取在 A→Q 这条射线的延长线上，保证 A→P 的直线必然经过 Q
   *   ④ 由于 B' 在该射线上，反射后的射线必然朝向 B
   * 校验时用命中环上的点 C（而非 B 本身）作为反射目标，避免只擦到坦克边缘。
   */
  const consider = (q, side) => {
    const qx = q.x, qy = q.y;
    if (qx < 0 || qx > CFG.W || qy < 0 || qy > CFG.H) return;
    if (side && side.owner && pointInObstacle(qx, qy, side.owner, -1)) return;

    const distToB = Math.hypot(bx - qx, by - qy);
    if (distToB < CFG.TANK_R + 10) return;
    const ux = (bx - qx) / distToB, uy = (by - qy) / distToB;
    const inset = CFG.TANK_R * 0.75;
    const cx = bx - ux * inset, cy = by - uy * inset;
    if (pointInObstacle(cx, cy, obstacles, 0)) return;

    // 瞄准点：从弹着点 Q 朝射手 A 的方向回退一段（必须朝 A，才能保证 A→P 的射线经过 Q）
    const vaX = ax - qx, vaY = ay - qy;
    const vaLen = Math.hypot(vaX, vaY);
    if (vaLen < 1) return;
    const aimLen = Math.min(vaLen * 0.9, 140);
    const px = qx + (vaX / vaLen) * aimLen;
    const py = qy + (vaY / vaLen) * aimLen;
    if (px < -40 || px > CFG.W + 40 || py < -40 || py > CFG.H + 40) return;

    const total = vaLen + distToB;
    if (total > maxDist + 80) return;
    if (segmentHitsAnyObstacle(ax, ay, px, py, obstacles)) return;
    if (segmentHitsAnyObstacle(qx, qy, bx, by, obstacles)) return;
    if (best && total >= best.dist) return;
    best = { x: px, y: py, dist: total };
  };

  // ① 竖直面（场地左右墙、矩形障碍左右面）
  for (const s of sides) {
    if (s.axis !== "v") continue;
    const surface = s.X - s.nx * CFG.SHELL_R;      // 炮弹实际反弹的平面
    const bMirror = 2 * surface - bx;              // 目标关于该平面的镜像
    const dxA = bMirror - ax;
    if (Math.abs(dxA) < 1e-9) continue;
    // A 与 B' 必须在平面两侧；交点参数 t 必然 > 1（B' 远在另一侧）
    if ((ax - surface) * (bMirror - surface) >= 0) continue;
    const t = (surface - ax) / dxA;
    if (t <= 0) continue;
    const hy = ay + (by - ay) * t;
    if (hy < s.lo || hy > s.hi) continue;
    consider({ x: surface, y: hy }, s);
  }

  // ② 水平面（场地上下墙、矩形障碍上下面）
  for (const s of sides) {
    if (s.axis !== "h") continue;
    const surface = s.X - s.ny * CFG.SHELL_R;
    const bMirror = 2 * surface - by;
    const dyA = bMirror - ay;
    if (Math.abs(dyA) < 1e-9) continue;
    if ((ay - surface) * (bMirror - surface) >= 0) continue;
    const t = (surface - ay) / dyA;
    if (t <= 0) continue;
    const hx = ax + (bx - ax) * t;
    if (hx < s.lo || hx > s.hi) continue;
    consider({ x: hx, y: surface }, s);
  }

  // ③ 圆形障碍：用切点方向反解（取 16 个方向上的切点做候选）
  for (const o of obstacles) {
    if (o.kind !== "circle") continue;
    for (let k = 0; k < 16; k++) {
      const a2 = k * Math.PI / 8;
      const nx = Math.cos(a2), ny = Math.sin(a2);
      const px = o.cx + nx * (o.r + CFG.SHELL_R);   // 炮弹实际反弹点
      const py = o.cy + ny * (o.r + CFG.SHELL_R);
      // 入射方向必须从圆外指向该切点
      const ix = px - ax, iy = py - ay;
      const len = Math.hypot(ix, iy);
      if (len < 1) continue;
      const dot = (ix / len) * nx + (iy / len) * ny;
      if (dot > -0.02) continue;                     // 必须从正面射来
      // 反射方向是否指向目标
      const rx = ix / len - 2 * dot * nx, ry = iy / len - 2 * dot * ny;
      const tx = bx - px, ty = by - py;
      const tlen = Math.hypot(tx, ty);
      if (tlen < CFG.TANK_R + 10) continue;
      if ((rx * tx + ry * ty) / tlen < 0.985) continue;   // 反射方向须基本对准目标
      consider({ x: px, y: py }, o);
    }
  }

  return best;
}

/* =========================================================================
   4. 实体
   ========================================================================= */
class Tank {
  constructor(x, y, isPlayer) {
    this.x = x; this.y = y;
    this.r = CFG.TANK_R;
    this.isPlayer = isPlayer;
    this.alive = true;
    this.angle = isPlayer ? -Math.PI / 4 : Math.PI * 0.75;   // 炮口朝向（跟鼠标）
    this.aimAngle = this.angle;                              // 炮塔的目标角（灵敏度跟踪用）
    this.hull = this.angle;                                  // 车体朝向（与炮塔分离）
    this.track = 0;                                          // 履带动画相位
    // 弹药：玩家与 NPC 同一套规则
    this.shotsLeft = CFG.MAX_SHELLS;                         // 一次上限 5 发
    this.cooldownEnd = 0;                                    // 打空后开始 10 秒冷却
    this.lastShot = 0;                                       // NPC 连发节流用
    this.shotsFired = 0;                                     // 统计
    // NPC 决策状态（打一发看一发）
    this.nextDecisionAt = 0;                                 // 下次判断"该不该开火"的时刻
    this.lastMissed = false;                                 // 上一发是否落空
    this.lastBankAt = -1e9;                                  // 上次反弹射击的时刻
    this.aiming = false;                                     // 是否正在瞄准（尚未开火）
    this.bankTarget = null;                                  // 正在瞄的弹着点（用于预警显示）
    this.orbitDir = Math.random() < 0.5 ? 1 : -1;            // 绕行方向（每辆不同）
    this.nailongPhase = Math.random() * Math.PI * 2;         // 奶龙的呼吸/摆尾相位
    this.recoil = 0;                                         // 开火后坐（0–1，绘制用）
  }
  /** 带滑动的移动：先整体尝试，再分轴尝试（贴墙时仍能沿墙滑行） */
  move(dx, dy, obstacles, tanks) {
    const step = (nx, ny) => {
      for (const o of obstacles) if (circleHitsObstacle(nx, ny, this.r, o)) return false;
      if (nx < this.r || ny < this.r || nx > CFG.W - this.r || ny > CFG.H - this.r) return false;
      for (const t of tanks) {
        if (t === this || !t.alive) continue;
        if (Math.hypot(nx - t.x, ny - t.y) < this.r + t.r) return false;
      }
      return true;
    };
    let moved = false;
    if (step(this.x + dx, this.y + dy)) { this.x += dx; this.y += dy; moved = true; }
    else if (step(this.x + dx, this.y)) { this.x += dx; moved = true; }
    else if (step(this.x, this.y + dy)) { this.y += dy; moved = true; }
    // 兜底夹紧：即使初始位置异常，也不会永久卡在边界外
    this.x = clamp(this.x, this.r, CFG.W - this.r);
    this.y = clamp(this.y, this.r, CFG.H - this.r);
    if (moved) this.track += Math.hypot(dx, dy) * 0.35;
    return moved;
  }
}

/**
 * 计算炮弹与障碍的碰撞法线（用于反弹）。
 * 矩形：比较四个方向的穿透深度，取最浅的一侧作为法线方向。
 * 圆形：法线 = 圆心连线方向。
 * @returns {x, y, nx, ny, depth} 或 null（未碰撞）
 */
function shellObstacleHit(sx, sy, r, o) {
  const EPS = 0.05;
  if (o.kind === "circle") {
    const dx = sx - o.cx, dy = sy - o.cy;
    const d = Math.hypot(dx, dy) || 1e-6;
    const pen = (r + o.r - EPS) - d;
    if (pen <= 0) return null;
    const nx = dx / d, ny = dy / d;
    return { nx, ny, depth: pen, x: o.cx + nx * (o.r + r - EPS), y: o.cy + ny * (o.r + r - EPS) };
  }
  // 矩形：包围盒用 (r + EPS) 而不是 (r - EPS)。
  // 若用 r-EPS，则"恰好贴在扩展半径上"的斜向碰撞会被漏判（斜边比直角边长），
  // 表现为 45° 入射时偶尔穿角而过。
  if (sx < o.x - r - EPS || sx > o.x + o.w + r + EPS ||
      sy < o.y - r - EPS || sy > o.y + o.h + r + EPS) return null;

  const penL = sx - (o.x - r);            // 从左面穿入的深度
  const penR = (o.x + o.w + r) - sx;
  const penT = sy - (o.y - r);
  const penB = (o.y + o.h + r) - sy;
  const min = Math.min(penL, penR, penT, penB);

  if (min === penL) return { nx: -1, ny: 0, depth: penL, x: o.x - r, y: sy };
  if (min === penR) return { nx: 1, ny: 0, depth: penR, x: o.x + o.w + r, y: sy };
  if (min === penT) return { nx: 0, ny: -1, depth: penT, x: sx, y: o.y - r };
  return { nx: 0, ny: 1, depth: penB, x: sx, y: o.y + o.h + r };
}

class Shell {
  constructor(x, y, angle, owner, shooter) {
    this.x = x; this.y = y;
    this.vx = Math.cos(angle) * CFG.SHELL_SPEED;
    this.vy = Math.sin(angle) * CFG.SHELL_SPEED;
    this.owner = owner;              // 'player' | 'npc'（统计与染色用）
    this.shooter = shooter || null;  // 发射者对象：命中判定只认它，避免阵营字符串误判
    this.alive = true;
    this.trail = [];
    this.bounces = 0;                // 已反弹次数
    this.hitWall = false;
    this.ricochet = false;           // 是否已经弹过（用于变色）
  }

  /** 反弹一次；返回 false 表示弹数用尽，炮弹应当消失 */
  bounce(nx, ny) {
    if (this.bounces >= CFG.MAX_BOUNCES) return false;
    const dot = this.vx * nx + this.vy * ny;
    if (dot < 0) {                                  // 只处理"撞进去"的情况
      this.vx -= 2 * dot * nx;
      this.vy -= 2 * dot * ny;
    }
    this.vx *= CFG.BOUNCE_KEEP;
    this.vy *= CFG.BOUNCE_KEEP;
    this.bounces++;
    this.ricochet = true;
    return true;
  }

  update(dt, obstacles, tanks, onHit, onDone) {
    const steps = 3;                                  // 子步进，避免高速穿透
    for (let i = 0; i < steps && this.alive; i++) {
      this.x += this.vx * dt / steps;
      this.y += this.vy * dt / steps;

      // ① 边界：按轴反射
      let bounced = false;
      if (this.x < CFG.SHELL_R) {                     // 左墙
        this.x = CFG.SHELL_R;
        bounced = this.bounce(1, 0);
        if (!bounced) { this.alive = false; this.hitWall = true; break; }
      } else if (this.x > CFG.W - CFG.SHELL_R) {      // 右墙
        this.x = CFG.W - CFG.SHELL_R;
        bounced = this.bounce(-1, 0);
        if (!bounced) { this.alive = false; this.hitWall = true; break; }
      }
      if (this.y < CFG.SHELL_R) {                     // 上墙
        this.y = CFG.SHELL_R;
        bounced = this.bounce(0, 1);
        if (!bounced) { this.alive = false; this.hitWall = true; break; }
      } else if (this.y > CFG.H - CFG.SHELL_R) {      // 下墙
        this.y = CFG.H - CFG.SHELL_R;
        bounced = this.bounce(0, -1);
        if (!bounced) { this.alive = false; this.hitWall = true; break; }
      }

      // ② 障碍：按法线反射
      let hitInfo = null;
      for (const o of obstacles) {
        const info = shellObstacleHit(this.x, this.y, CFG.SHELL_R, o);
        if (info) { hitInfo = info; break; }
      }
      if (hitInfo) {
        this.x = hitInfo.x; this.y = hitInfo.y;       // 先退出障碍，避免卡住
        const okBounce = this.bounce(hitInfo.nx, hitInfo.ny);
        this.lastHit = { x: this.x, y: this.y };
        if (!okBounce) {
          this.alive = false;
          this.hitWall = true;                        // 弹数用尽 → 就地爆开
          break;
        }
      }
      if (!this.alive) break;

      // ③ 命中坦克（反弹后的炮弹同样致命；仍不打自己）
      for (const t of tanks) {
        if (!t.alive) continue;
        // 只放过发射者自己（避免自伤）；其余坦克无论阵营都会被击中。
        // 之前按 owner 字符串 + isPlayer 过滤，会把 NPC 的炮弹错当己方而漏判命中。
        if (this.shooter && t === this.shooter) continue;
        if (Math.hypot(this.x - t.x, this.y - t.y) < t.r + CFG.SHELL_R) {
          this.alive = false;
          this.hitTank = t;
          onHit(t);
          break;
        }
      }
    }

    // 拖尾：反弹过的炮弹拖尾更长，便于观察轨迹
    const maxTrail = this.bounces > 0 ? 14 : 6;
    this.trail.push({ x: this.x, y: this.y });
    while (this.trail.length > maxTrail) this.trail.shift();

    // 生命周期结束 → 回调（用于 NPC 判断"这一发打没打中"）
    if (!this.alive && !this.reported) {
      this.reported = true;
      if (onDone) onDone(this, !!this.hitTank);
    }
  }
}

/* =========================================================================
   5. 游戏状态
   ========================================================================= */
const cv = document.getElementById("cv");
const ctx = cv.getContext("2d");

const G = {
  level: null,
  player: null,
  npcs: [],
  shells: [],
  particles: [],
  decor: [],                          // 场地里散落的奶龙（纯装饰，不参与判定）
  keys: Object.create(null),          // 只由键盘写入，只被位移计算读取
  mouse: { x: CFG.W * 0.7, y: CFG.H * 0.3 },   // 只由鼠标写入，只被瞄准/开火/准星读取
  state: "playing",        // playing | win | lose（对局状态）
  screen: "title",         // title | countdown | game —— 开屏 / 倒计时 / 对局中
  title: null,             // 标题页的静态场景（一次性生成，避免每帧抖动）
  titleT: 0,               // 标题页动画时间
  message: "",
  time: 0,
  winCount: 0,             // 连胜次数（胜利 +1，失败清零）
  countdownT: 0,           // 倒计时剩余秒数
  resultAt: 0,             // 胜负结果出现的时间戳（0 = 未结算）
};

/** 在空地上撒一些装饰奶龙（避开出生点与障碍） */
function placeDecor(seed) {
  const rnd = mulberry32((seed ^ 0x9e3779b9) >>> 0);
  // 各种黄色的奶龙 + 少量"异色款"（参考图里也有白/奶油色的）
  const palettes = [
    { body: "#ffce21", belly: "#fffbe8" },
    { body: "#ffc107", belly: "#fff8dd" },
    { body: "#ffdb63", belly: "#fffdf2" },
    { body: "#fff0c9", belly: "#ffffff" },
    { body: "#ffa93a", belly: "#fff0d4" },
  ];
  const out = [];
  let guard = 0;
  while (out.length < 16 && guard++ < 500) {
    const x = 34 + rnd() * (CFG.W - 68);
    const y = 34 + rnd() * (CFG.H - 68);
    if (Math.hypot(x - PLAYER_SPAWN.x, y - PLAYER_SPAWN.y) < 120) continue;
    let blocked = false;
    for (const o of G.level.obstacles) {
      if (o.kind === "circle") {
        if (Math.hypot(x - o.cx, y - o.cy) < o.r + 26) { blocked = true; break; }
      } else {
        const nx = Math.max(o.x, Math.min(x, o.x + o.w));
        const ny = Math.max(o.y, Math.min(y, o.y + o.h));
        if (Math.hypot(x - nx, y - ny) < 26) { blocked = true; break; }
      }
    }
    if (blocked) continue;
    const pal = palettes[Math.floor(rnd() * palettes.length)];
    out.push({
      x, y,
      body: pal.body, belly: pal.belly,
      scale: 0.50 + rnd() * 0.24,           // 大小略有差异，更像散落一地
      rot: (rnd() - 0.5) * 1.1,             // 轻微歪头
      phase: rnd() * Math.PI * 2,
      alpha: 0.88 + rnd() * 0.12,
    });
  }
  G.decor = out;
}

/** 生成一局的地形、玩家、NPC（不含流程状态），可指定种子用于复现 */
function setupLevel(seed) {
  const s = (seed === undefined || seed === null)
    ? (Math.floor(Math.random() * 0xffffffff) >>> 0)
    : (seed >>> 0);
  G.level = generateLevel(s);
  G.player = new Tank(PLAYER_SPAWN.x, PLAYER_SPAWN.y, true);
  // NPC 沿路径中段巡逻起步，增加变化；只用 1 个 NPC，保证"一个炮弹即败一个坦克"
  const start = G.level.path[Math.min(G.level.path.length - 1, Math.floor(G.level.path.length * 0.6))];
  G.npcs = [new Tank(start.x, start.y, false)];
  G.shells = [];
  G.particles = [];
  placeDecor(s);                      // 撒装饰奶龙（需要 G.level 已就绪）
  document.getElementById("hudSeed").textContent = G.level.seed;
}

/** 完全重新开始：回到开屏页，连胜清零 */
function newGame(seed) {
  setupLevel(seed);
  G.state = "playing";
  G.message = "";
  G.winCount = 0;
  G.countdownT = 0;
  G.resultAt = 0;
  G.screen = "title";
  G.titleT = 0;
  buildTitleScene();
}

/** 开始一局（进入倒计时）。胜利后连打下一局也走这里，不再回开屏页 */
function startRound() {
  setupLevel();                       // 新随机种子
  G.state = "playing";
  G.message = "";
  G.countdownT = CFG.COUNTDOWN_SECONDS;
  G.resultAt = 0;
  G.screen = "countdown";
  G.keys = Object.create(null);
  last = performance.now();
}

/* =========================================================================
   6. 输入
   ========================================================================= */

/* -------------------------------------------------------------------------
   输入隔离说明（两套输入完全独立，互不读写对方的状态）
     · 键盘：只写 G.keys，只被"位移计算"读取
     · 鼠标：只写 G.mouse，只被"炮塔朝向 / 开火方向 / 准星"读取
   任何一边的改动都不会影响另一边的行为。
   ------------------------------------------------------------------------- */

/* ---------- 键盘：仅负责移动 ---------- */
addEventListener("keydown", e => {
  // 开屏页：任意键（用 R 重开时不重复触发）
  if (G.screen === "title") {
    if (e.key !== "r" && e.key !== "R") { startGame(); return; }
  }
  G.keys[e.key.toLowerCase()] = true;
  if (e.key === "r" || e.key === "R") newGame();
  if (["w", "a", "s", "d", " "].includes(e.key.toLowerCase())) e.preventDefault();
});
addEventListener("keyup", e => { G.keys[e.key.toLowerCase()] = false; });

/* ---------- 鼠标：仅负责瞄准与开火 ---------- */
/** 把鼠标事件换算到画布坐标（标题页与对局共用） */
function toCanvas(e) {
  const r = cv.getBoundingClientRect();
  return {
    x: (e.clientX - r.left) * (CFG.W / r.width),
    y: (e.clientY - r.top) * (CFG.H / r.height),
  };
}
cv.addEventListener("mousemove", e => {
  const p2 = toCanvas(e);
  G.mouse.x = p2.x; G.mouse.y = p2.y;
});
cv.addEventListener("contextmenu", e => {
  e.preventDefault();
  if (G.screen === "title") { startGame(); return; }   // 右键也算"任意位置"
  tryFire();
});
// 左键：开屏页点击开始；对局中不参与开火（开火只认右键）
cv.addEventListener("mousedown", e => {
  if (G.screen !== "title") return;
  e.preventDefault();
  startGame();
});
// 触屏：同样"点任意位置开始"
cv.addEventListener("touchstart", e => {
  if (G.screen !== "title") return;
  e.preventDefault();
  startGame();
}, { passive: false });

/** 离开开屏页：进入倒计时（鼠标停在原地不会立刻开火，需要玩家主动点右键） */
function startGame() {
  if (G.screen !== "title") return;
  G.screen = "countdown";
  G.state = "playing";
  G.countdownT = CFG.COUNTDOWN_SECONDS;
  G.resultAt = 0;
  G.keys = Object.create(null);            // 清掉开局瞬间可能残留的按键
  last = performance.now();
}

/* =========================================================================
   7. 瞄准：炮塔跟随鼠标
      灵敏度 AIM_SENSITIVITY 控制"跟手程度"：
        · 值越大 → 越立刻贴到鼠标（越灵敏）
        · 值越小 → 越平滑（炮塔转动有延迟，手感更重）
      = 1 时约为原来的平滑度；= 14 时几乎瞬时贴合（默认）
   ========================================================================= */
function aimTurret(tank, targetX, targetY, dt) {
  const want = Math.atan2(targetY - tank.y, targetX - tank.x);
  const diff = ((want - tank.angle + Math.PI * 3) % (Math.PI * 2)) - Math.PI;
  if (dt === undefined || dt <= 0) {
    tank.angle = want;                       // 瞬时贴合（开火前调用，保证打得准）
    return;
  }
  // 指数逼近：k = 1 - e^(-灵敏度 × dt)，与帧率无关
  //   灵敏度 = 1   → 每 1/60 秒只走掉约 1.6% 的差值（明显平滑）
  //   灵敏度 = 14  → 每 1/60 秒走掉约 21% 的差值，几乎瞬时贴合
  const k = 1 - Math.exp(-CFG.AIM_SENSITIVITY * dt);
  tank.angle += diff * k;
}

/* =========================================================================
   8. 弹药规则：5 发上限，逐发无冷却；打空后 10 秒冷却才能续弹
      —— 玩家与 NPC 共用同一套逻辑，规则完全一致
   ========================================================================= */

/** 弹药是否已补满（打空后冷却结束 → 回满 5 发） */
function checkReload(tank, now) {
  if (tank.shotsLeft === 0 && now >= tank.cooldownEnd) {
    tank.shotsLeft = CFG.MAX_SHELLS;
    tank.cooldownEnd = 0;
    return true;
  }
  return false;
}

/** 当前能否开火（有弹 + 不在冷却中） */
function canFire(tank, now) {
  checkReload(tank, now);
  return tank.alive && tank.shotsLeft > 0 && now >= tank.cooldownEnd;
}

/** 扣一发弹药；打到 0 发时开始 10 秒冷却 */
function consumeShot(tank, now) {
  tank.shotsLeft--;
  tank.recoil = 1;
  if (tank.shotsLeft === 0) tank.cooldownEnd = now + CFG.COOLDOWN_MS;
}

function tryFire() {
  if (G.screen !== "game") return;
  if (G.state !== "playing") return;
  const now = performance.now();
  const p = G.player;
  if (!canFire(p, now)) return;              // 弹药耗尽且在冷却中 → 打不出去

  const ang = p.angle;                        // 用炮塔当前朝向（与屏幕上看到的一致）
  const muzzle = p.r + 10;
  const sx = p.x + Math.cos(ang) * muzzle;
  const sy = p.y + Math.sin(ang) * muzzle;

  // 炮口若在障碍里，就从车体中心发射，避免炮弹卡进墙里
  let spawnX = sx, spawnY = sy;
  for (const o of G.level.obstacles) {
    if (circleHitsObstacle(spawnX, spawnY, CFG.SHELL_R, o)) { spawnX = p.x; spawnY = p.y; break; }
  }

  G.shells.push(new Shell(spawnX, spawnY, ang, "player", p));
  consumeShot(p, now);                       // 逐发无冷却，可连点打空
}

/* =========================================================================
   8. NPC 行为：沿路径接近玩家 + 有视线就开火
   ========================================================================= */
function updateNpc(npc, dt) {
  if (!npc.alive) return;
  const p = G.player;
  const now = performance.now();
  const distToPlayer = Math.hypot(p.x - npc.x, p.y - npc.y);
  const seesPlayer = !lineBlocked(npc.x, npc.y, p.x, p.y, G.level.obstacles);

  // 瞄准：看得见就瞄玩家，看不见就朝路径方向
  const path = findPath(G.level.grid, npc, p) || G.level.path;
  let targetAngle;
  if (seesPlayer) {
    targetAngle = Math.atan2(p.y - npc.y, p.x - npc.x);
  } else {
    const nodeAim = path[Math.min(3, path.length - 1)] || p;
    targetAngle = Math.atan2(nodeAim.y - npc.y, nodeAim.x - npc.x);
  }
  // 炮塔用指数逼近（与玩家同一套手感，且与帧率无关）
  const kAim = 1 - Math.exp(-9 * dt);
  let diff = ((targetAngle - npc.angle + Math.PI * 3) % (Math.PI * 2)) - Math.PI;
  npc.angle += diff * kAim;
  // 车体朝向在移动分支里独立更新（这里是炮塔层，不再覆盖 hull）

  // 车体朝向 = 行进方向；炮塔朝向（npc.angle）独立指向目标。
  // 这样 NPC 一边后退拉开距离，炮口仍能咬住玩家 —— 与玩家坦克的分层结构一致。
  const speed = CFG.NPC_SPEED * dt;
  const node = path[Math.min(6, path.length - 1)];
  let mx = 0, my = 0;
  if (seesPlayer) {
    if (distToPlayer < 190) {
      // 太近：不要直线后退（会退到死角丢视线），而是绕着玩家转
      const turn = npc.orbitDir * 0.55 * Math.PI;
      mx = Math.cos(npc.angle + turn);
      my = Math.sin(npc.angle + turn);
    } else if (distToPlayer > 340) {
      mx = Math.cos(npc.angle); my = Math.sin(npc.angle);
    } else {
      // 横向机动，保持难打
      mx = Math.cos(npc.angle + npc.orbitDir * Math.PI / 2) * 0.8;
      my = Math.sin(npc.angle + npc.orbitDir * Math.PI / 2) * 0.8;
    }
  } else if (node) {
    const a = Math.atan2(node.y - npc.y, node.x - npc.x);
    mx = Math.cos(a); my = Math.sin(a);
  }
  if (mx || my) {
    const len = Math.hypot(mx, my) || 1;
    const moved = npc.move(mx / len * speed, my / len * speed, G.level.obstacles, [G.player, ...G.npcs]);
    if (!moved) {                                   // 卡住了：换个垂直方向试
      npc.move(-my / len * speed, mx / len * speed, G.level.obstacles, [G.player, ...G.npcs]);
    }
    // 车体朝行进方向平滑转过去（与炮塔解耦）
    const moveAng = Math.atan2(my, mx);
    let hd = ((moveAng - npc.hull + Math.PI * 3) % (Math.PI * 2)) - Math.PI;
    npc.hull += clamp(hd, -2.6 * dt, 2.6 * dt);
  }

  // ───────── 开火决策：打一发、看一发，而不是一次倾泻 5 发 ─────────
  // 每次只判断"这一发该不该打"，打完就把下次决策时间往后推。
  if (canFire(npc, now) && now >= npc.nextDecisionAt) {
    // ① 有视线：直射
    if (seesPlayer && distToPlayer < CFG.NPC_LOS_RANGE) {
      const want = Math.atan2(p.y - npc.y, p.x - npc.x);
      const aimErr = Math.abs(((want - npc.angle + Math.PI * 3) % (Math.PI * 2)) - Math.PI);

      if (aimErr > CFG.NPC_AIM_TOL) {
        // 还没瞄准好 —— 先转炮塔，这一发不打（这就是"适时应变"）
        npc.aiming = true;
        npc.nextDecisionAt = now + 60;
      } else {
        // 瞄准偏差随距离放宽；远距离更容易偏
        const spread = 0.02 + Math.min(0.05, distToPlayer / 12000);
        const ang = want + (Math.random() - 0.5) * spread;
        G.shells.push(new Shell(
          npc.x + Math.cos(ang) * (npc.r + 10),
          npc.y + Math.sin(ang) * (npc.r + 10),
          ang, "npc", npc
        ));
        consumeShot(npc, now);
        npc.aiming = false;
        npc.shotsFired++;
        npc.lastShotAt = now;
        // 上一发没命中 → 多等一会儿（像是在修正弹道）
        const base = CFG.NPC_PAUSE_MIN + Math.random() * (CFG.NPC_PAUSE_MAX - CFG.NPC_PAUSE_MIN);
        npc.nextDecisionAt = now + base + (npc.lastMissed ? CFG.NPC_MISS_PAUSE : 0);
        npc.lastMissed = false;
      }
    } else {
      // ② 没有视线：判断能不能靠一次反弹打到掩体后面
      if (now - (npc.lastBankAt || -1e9) > CFG.NPC_BANK_COOLDOWN) {
        const bank = solveBankShot(npc.x, npc.y, p.x, p.y, G.level.obstacles, CFG.NPC_BANK_MAX_DIST);
        if (bank) {
          const want = Math.atan2(bank.y - npc.y, bank.x - npc.x);
          const aimErr = Math.abs(((want - npc.angle + Math.PI * 3) % (Math.PI * 2)) - Math.PI);
          if (aimErr > CFG.NPC_AIM_TOL) {
            // 先转到弹着点方向（玩家能看见它在"瞄墙"，有预警）
            npc.aiming = true;
            npc.bankTarget = { x: bank.x, y: bank.y };
            npc.nextDecisionAt = now + 60;
          } else {
            const ang = want + (Math.random() - 0.5) * 0.03;
            G.shells.push(new Shell(
              npc.x + Math.cos(ang) * (npc.r + 10),
              npc.y + Math.sin(ang) * (npc.r + 10),
              ang, "npc", npc
            ));
            consumeShot(npc, now);
            npc.aiming = false;
            npc.shotsFired++;
            npc.lastBankAt = now;
            npc.bankTarget = null;
            npc.nextDecisionAt = now + CFG.NPC_PAUSE_MAX;      // 反弹射击后要缓一阵
            npc.lastMissed = false;
          }
        } else {
          // ③ 打不着：不浪费弹药，继续走位/等机会
          npc.aiming = false;
          npc.bankTarget = null;
          npc.nextDecisionAt = now + 260;
        }
      } else {
        npc.nextDecisionAt = now + 260;
      }
    }
  }
}

/* =========================================================================
   9. 主循环
   ========================================================================= */
let last = performance.now();
function loop(now) {
  const dt = Math.min((now - last) / 1000, 0.05);
  last = now;
  G.time += dt;

  // 开屏页：只推进标题动画，不跑对局逻辑（NPC 不动、炮弹不飞）
  if (G.screen === "title") {
    G.titleT += dt;
    draw();
    requestAnimationFrame(loop);
    return;
  }

  // 倒计时：只推进倒计时，坦克/炮弹全部冻结
  if (G.screen === "countdown") {
    G.countdownT -= dt;
    if (G.countdownT <= 0) {
      G.screen = "game";
      G.state = "playing";
      G.countdownT = 0;
      G.keys = Object.create(null);      // 清掉倒计时期间可能按下的键
      last = now;
    }
    draw();
    requestAnimationFrame(loop);
    return;
  }

  update(dt);

  // 胜负结算：胜利 → 连胜 +1，稍后自动进入下一局（不回开屏页）；
  //           失败 → 连胜清零，稍后返回开屏页
  if (G.state === "win" || G.state === "lose") {
    if (G.resultAt === 0) {
      G.resultAt = now;
      if (G.state === "win") G.winCount++;
      else G.winCount = 0;
    }
    const delay = G.state === "win" ? CFG.RESULT_WIN_DELAY : CFG.RESULT_LOSE_DELAY;
    if (now - G.resultAt >= delay) {
      if (G.state === "win") startRound();
      else newGame();
    }
  }

  draw();
  requestAnimationFrame(loop);
}

function update(dt) {
  if (G.state === "playing") {
    const now = performance.now();
    const p = G.player;

    // 自动续弹：冷却结束 → 玩家与 NPC 都回满 5 发
    checkReload(p, now);
    for (const n of G.npcs) checkReload(n, now);

    // ① 炮塔跟随鼠标（车体不转，键盘与鼠标互不影响）
    aimTurret(p, G.mouse.x, G.mouse.y, dt);
    p.hull = -Math.PI / 4;                      // 车体保持固定朝向

    // ② WASD 世界方向移动：W 上 / S 下 / A 左 / D 右
    let dx = 0, dy = 0;
    if (G.keys["w"]) dy -= 1;
    if (G.keys["s"]) dy += 1;
    if (G.keys["a"]) dx -= 1;
    if (G.keys["d"]) dx += 1;
    if (dx || dy) {
      const len = Math.hypot(dx, dy);
      p.move(dx / len * CFG.PLAYER_SPEED * dt, dy / len * CFG.PLAYER_SPEED * dt,
             G.level.obstacles, [p, ...G.npcs]);
    }

    // ③ 后坐衰减 + 奶龙呼吸相位推进
    if (p.recoil > 0) p.recoil = Math.max(0, p.recoil - dt * 5);
    p.nailongPhase += dt;

    for (const n of G.npcs) {
      if (n.recoil > 0) n.recoil = Math.max(0, n.recoil - dt * 5);
      n.nailongPhase += dt;
      updateNpc(n, dt);
    }
  }

  // 炮弹
  for (const s of G.shells) {
    if (!s.alive) continue;
    const prevBounces = s.bounces;
    s.update(dt, G.level.obstacles, [G.player, ...G.npcs], (t) => {      // 一个炮弹即摧毁
      t.alive = false;
      boom(t.x, t.y, t.isPlayer);
      // 胜负状态一律由 alive 决定，避免"已阵亡但仍判负"这类状态不同步
      if (t.isPlayer) {
        if (!G.player.alive) { G.state = "lose"; G.message = "你的坦克被击毁"; }
      } else {
        G.npcs = G.npcs.filter(n => n.alive);
        if (G.npcs.length === 0) { G.state = "win"; G.message = "全部敌军已摧毁"; }
      }
    }, (s2, hitSomething) => {
      // 炮弹生命周期结束后的反馈：让 NPC 知道"这一发打没打中"，从而调整节奏
      if (s2.owner !== "npc") return;
      const shooter = G.npcs.find(n => n.alive);
      if (!shooter) return;
      if (!hitSomething) {
        shooter.lastMissed = true;
        shooter.nextDecisionAt += CFG.NPC_MISS_PAUSE * 0.5;   // 没打中 → 再缓一下
      }
    });
    // 本帧发生了反弹 → 在弹着点崩出火花
    if (s.bounces > prevBounces && s.alive && s.lastHit) {
      spark(s.lastHit.x, s.lastHit.y);
      s.lastHit = null;
    }
    // 弹数用尽或撞墙 → 就地爆开
    if (!s.alive && s.hitWall) boom(s.x, s.y, false, 7);
  }
  G.shells = G.shells.filter(s => s.alive);

  // 粒子
  for (const q of G.particles) { q.x += q.vx * dt; q.y += q.vy * dt; q.vx *= 0.94; q.vy *= 0.94; q.life -= dt; }
  G.particles = G.particles.filter(q => q.life > 0);
}

function boom(x, y, isPlayer, n) {
  n = n || (isPlayer ? 34 : 46);
  const color = isPlayer ? "#ff7a59" : "#ffd166";
  for (let i = 0; i < n; i++) {
    const a = Math.random() * Math.PI * 2;
    const sp = 40 + Math.random() * 240;
    G.particles.push({
      x, y, vx: Math.cos(a) * sp, vy: Math.sin(a) * sp,
      life: 0.35 + Math.random() * 0.6, max: 0.95,
      color: Math.random() < 0.45 ? "#fff2cc" : color,
    });
  }
}

/** 反弹火花（比爆炸小，青色，用于标示弹着点） */
function spark(x, y, n) {
  n = n || 12;
  for (let i = 0; i < n; i++) {
    const a = Math.random() * Math.PI * 2;
    const sp = 60 + Math.random() * 150;
    G.particles.push({
      x, y, vx: Math.cos(a) * sp, vy: Math.sin(a) * sp,
      life: 0.18 + Math.random() * 0.28, max: 0.46,
      color: Math.random() < 0.5 ? "#ffffff" : "#9fe6ff",
    });
  }
}

/**
 * NPC 瞄准/反弹瞄准的预警提示：
 *   · 正在瞄准玩家 → 炮口前方画一段红色虚线
 *   · 正在瞄墙做反弹 → 画出"弹着点 + 折射方向"的虚线，玩家能看见并躲开
 */
function drawNpcTelegraph() {
  for (const n of G.npcs) {
    if (!n.alive || !n.aiming) continue;

    if (n.bankTarget) {
      // 反弹射击预警：NPC → 弹着点 → 玩家方向
      ctx.save();
      ctx.setLineDash([6, 6]);
      ctx.strokeStyle = "rgba(255,120,90,0.55)";
      ctx.lineWidth = 1.5;
      ctx.beginPath();
      ctx.moveTo(n.x, n.y);
      ctx.lineTo(n.bankTarget.x, n.bankTarget.y);
      ctx.stroke();

      // 弹着点标记
      ctx.setLineDash([]);
      ctx.strokeStyle = "rgba(255,120,90,0.9)";
      ctx.beginPath();
      ctx.arc(n.bankTarget.x, n.bankTarget.y, 7, 0, Math.PI * 2);
      ctx.stroke();
      ctx.beginPath();
      ctx.moveTo(n.bankTarget.x - 10, n.bankTarget.y);
      ctx.lineTo(n.bankTarget.x + 10, n.bankTarget.y);
      ctx.moveTo(n.bankTarget.x, n.bankTarget.y - 10);
      ctx.lineTo(n.bankTarget.x, n.bankTarget.y + 10);
      ctx.stroke();
      ctx.restore();
    } else {
      // 直射预警：炮口延长线
      ctx.save();
      ctx.setLineDash([5, 7]);
      ctx.strokeStyle = "rgba(255,120,90,0.35)";
      ctx.lineWidth = 1.5;
      ctx.beginPath();
      ctx.moveTo(n.x + Math.cos(n.angle) * (n.r + 8), n.y + Math.sin(n.angle) * (n.r + 8));
      ctx.lineTo(n.x + Math.cos(n.angle) * 220, n.y + Math.sin(n.angle) * 220);
      ctx.stroke();
      ctx.restore();
    }
  }
}

/* =========================================================================
   10. 绘制
   ========================================================================= */
/* =========================================================================
   10. 绘制 —— 风格：玩具军模（橄榄绿坦克 + 奶龙吉祥物 + 暖沙地面）
   ========================================================================= */

/** 各阵营配色 —— 取自像素风参考图：军绿车体 + 深墨绿履带/轮廓 + 黄奶龙 */
const SKIN = {
  player: {
    // 车体（军绿三档：亮面 / 主色 / 暗面）
    bodyHi: "#7f8a44", body: "#687339", bodyLo: "#4f5a29", bodyEdge: "#2f371a",
    // 履带（深色 + 纹路）
    trackDark: "#2b2f1c", trackMid: "#3d4326", trackLite: "#575e38",
    hatchRim: "#23261a",
    duck: "#ffd84d", duckBelly: "#fff6d5",
    rim: "rgba(255,244,200,0.5)",
  },
  npc: {
    bodyHi: "#a9714f", body: "#8a5636", bodyLo: "#6b3f26", bodyEdge: "#3f2415",
    trackDark: "#3a2418", trackMid: "#4e3120", trackLite: "#6b472c",
    hatchRim: "#2a1a10",
    duck: "#ffb648", duckBelly: "#fff0d0",
    rim: "rgba(255,214,170,0.5)",
  },
};

/** 圆角矩形路径（带回退） */
function rr(x, y, w, h, r) {
  ctx.beginPath();
  if (ctx.roundRect) ctx.roundRect(x, y, w, h, r);
  else {
    ctx.moveTo(x + r, y);
    ctx.arcTo(x + w, y, x + w, y + h, r);
    ctx.arcTo(x + w, y + h, x, y + h, r);
    ctx.arcTo(x, y + h, x, y, r);
    ctx.arcTo(x, y, x + w, y, r);
    ctx.closePath();
  }
}

/**
 * 像素风绘制基元。
 * 整个坦克/奶龙都用"逻辑像素格"作画：先按 PX 换算成整数格，
 * 再用 fillRect 画方块 —— 天然得到参考图那种硬边、块状的像素观感。
 */
const PX = 2;                              // 一"像素"占几个画布像素
/** 像素风字体栈：优先中文像素字体 Zpix / 方正像素，回退 Consolas */
const PIXFONT = '"Zpix", "Fusion Pixel", "方正像素", Consolas, "Microsoft YaHei", monospace';
function px(x, y, w, h, color) {           // 逻辑像素坐标 → 方块
  ctx.fillStyle = color;
  ctx.fillRect(x * PX, y * PX, w * PX, h * PX);
}

/**
 * 奶龙（像素版）。
 *
 * 造型依据正面全身参考图：一颗竖着的蛋、白眼球 + 绿虹膜、张嘴大笑、
 * 奶油色大肚皮、两颊腮红、短手短脚。用像素方块重绘以贴合新的像素画风。
 *
 * 局部坐标以身体中心为原点，总高约 24 格、总宽约 21 格。
 */
function drawNailong(body, belly, s, t) {
  const bob = Math.round(Math.sin(t * 2) * 0.5);     // 取整，避免半像素抖动
  const wave = Math.round(Math.sin(t * 2.4) * 1.2);  // 挥手

  const line = "#c98a1e";
  const claw = "#7a4a24";
  const eyeWhite = "#ffffff";
  const iris = "#5aa64b";

  ctx.save();
  ctx.scale(s, s);

  // 落地阴影
  px(-8, 13, 16, 2, "rgba(96,72,40,0.20)");
  px(-6, 12, 12, 1, "rgba(96,72,40,0.16)");

  ctx.save();
  ctx.translate(0, bob);

  // 脚（含深棕爪尖）
  for (const fx of [-6, 3]) {
    px(fx, 10, 4, 4, body);
    px(fx, 14, 4, 1, claw);
    px(fx, 10, 1, 4, line);
    px(fx + 3, 10, 1, 4, line);
  }

  // 手臂：左手垂下、右手举起挥手
  px(-11, 0, 3, 8, body);                    // 左臂
  px(-11, 8, 1, 1, claw); px(-10, 8, 1, 1, claw); px(-9, 8, 1, 1, claw);
  px(8, -6 + wave, 3, 8, body);              // 右臂（举起）
  px(8, -7 + wave, 1, 1, claw); px(9, -7 + wave, 1, 1, claw); px(10, -7 + wave, 1, 1, claw);
  px(9, -4 + wave, 2, 3, "rgba(255,150,150,0.6)");   // 掌心肉垫

  // 身体：一颗竖着的蛋（像素阶梯轮廓，上宽下收）
  px(-9, -12, 18, 10, body);
  px(-10, -9, 20, 8, body);
  px(-9, -1, 18, 6, body);
  px(-8, 5, 16, 5, body);
  px(-6, 10, 12, 2, body);
  // 下半部暗面，做出体积感
  px(-8, 5, 16, 1, line);
  px(-6, 10, 12, 1, line);
  px(-10, -9, 1, 8, line);
  px(9, -9, 1, 8, line);
  px(-9, -12, 18, 1, line);
  px(-9, 6, 18, 1, "rgba(180,130,30,0.35)");
  // 顶部高光
  px(-5, -11, 8, 1, "rgba(255,255,255,0.35)");

  // 奶油色大肚皮
  px(-6, 0, 12, 8, belly);
  px(-7, 2, 14, 5, belly);
  px(-5, 1, 10, 1, "rgba(230,205,150,0.55)");

  // 眼睛：白眼球 + 绿虹膜 + 黑瞳 + 高光
  for (const ex of [-6, 2]) {
    px(ex, -10, 4, 5, eyeWhite);
    px(ex, -9, 4, 3, iris);
    px(ex + 1, -9, 2, 2, "#20301c");
    px(ex, -10, 1, 1, eyeWhite);
  }

  // 腮红
  px(-11, -3, 3, 2, "rgba(255,146,132,0.55)");
  px(8, -3, 3, 2, "rgba(255,146,132,0.55)");

  // 张嘴大笑：暗色口腔 + 舌头 + 上排小牙
  px(-4, -3, 8, 4, "#5c2a24");
  px(-5, -2, 10, 2, "#5c2a24");
  px(-2, 1, 4, 2, "#ff8f9c");
  px(-3, -3, 6, 1, eyeWhite);

  ctx.restore();
  ctx.restore();
}

/** 车体层（像素风）：履带 + 车体 + 舱盖，朝行进方向
 *  像素坐标用 PX=2 绘制（一格 2 画布像素），所以整体再乘 0.5 缩回原尺寸，
 *  让坦克大小与碰撞判定半径 TANK_R=16（直径 32px）保持一致。 */
function drawHull(t, sk, k) {
  const r = t.r;
  ctx.save();
  ctx.translate(t.x, t.y);
  ctx.rotate(t.hull);
  ctx.scale((k || 1) * 0.5, (k || 1) * 0.5);

  // 履带：两条深色带，两侧留出纹路方块
  px(-r, -r - 3, r * 2, 7, sk.trackDark);
  px(-r, r - 4, r * 2, 7, sk.trackDark);
  for (let i = -r + 1; i < r - 1; i += 4) {
    const off = ((i + t.track) % 8 + 8) % 8;
    px(i, -r - 3, 2, 7, off < 4 ? sk.trackLite : sk.trackMid);
    px(i, r - 4, 2, 7, off < 4 ? sk.trackLite : sk.trackMid);
  }

  // 车体：军绿主体 + 上亮下暗的三档分色 + 深色描边
  px(-r + 1, -r + 1, (r - 1) * 2, (r - 1) * 2, sk.body);
  px(-r + 1, -r + 1, (r - 1) * 2, 4, sk.bodyHi);
  px(-r + 1, r - 6, (r - 1) * 2, 5, sk.bodyLo);
  px(-r + 1, -r + 1, (r - 1) * 2, 1, sk.bodyEdge);
  px(-r + 1, r - 2, (r - 1) * 2, 1, sk.bodyEdge);
  px(-r + 1, -r + 1, 1, (r - 1) * 2, sk.bodyEdge);
  px(r - 2, -r + 1, 1, (r - 1) * 2, sk.bodyEdge);

  // 车体上的方形舱盖（参考图里那几块方块）
  px(-r + 4, -r + 4, 8, 5, sk.bodyHi);
  px(-r + 4, r - 10, 8, 5, sk.bodyHi);
  px(-r + 4, -r + 4, 8, 1, sk.bodyEdge);
  px(-r + 4, r - 10, 8, 1, sk.bodyEdge);
  // 尾部散热格栅
  for (let i = 0; i < 4; i++) px(r - 9 + i * 2, -r + 3, 1, 6, sk.bodyEdge);

  ctx.restore();
}

/** 炮塔层（像素风）：炮管 + 炮塔 + 舱口里的奶龙，朝炮口方向 */
function drawTurret(t, isPlayer, sk, k) {
  const r = t.r;
  const kk = k || 1;
  ctx.save();
  ctx.translate(t.x, t.y);
  ctx.rotate(t.angle);
  ctx.scale(kk * 0.5, kk * 0.5);

  const rec = t.recoil > 0 ? -Math.round(t.recoil * 4) : 0;   // 开火后坐

  // 炮管：分两节 + 末端制退器（方块阶梯，像素感）
  px(rec + 4, -3, r + 6, 6, sk.body);
  px(rec + 4, -3, r + 6, 2, sk.bodyHi);
  px(rec + 4, 1, r + 6, 2, sk.bodyLo);
  px(rec + 4, -3, r + 6, 1, sk.bodyEdge);
  px(rec + 4, 2, r + 6, 1, sk.bodyEdge);
  px(rec + r + 8, -5, 5, 10, sk.bodyHi);      // 制退器
  px(rec + r + 8, -5, 5, 1, sk.bodyEdge);
  px(rec + r + 8, 4, 5, 1, sk.bodyEdge);
  px(rec + r + 8, -5, 1, 10, sk.bodyEdge);
  // 炮口
  px(rec + r + 13, -2, 2, 4, "#1d2015");

  // 炮塔：略小的军绿方块 + 上亮下暗
  px(-r + 3, -r + 3, (r - 3) * 2, (r - 3) * 2, sk.body);
  px(-r + 3, -r + 3, (r - 3) * 2, 4, sk.bodyHi);
  px(-r + 3, r - 7, (r - 3) * 2, 4, sk.bodyLo);
  px(-r + 3, -r + 3, (r - 3) * 2, 1, sk.bodyEdge);
  px(-r + 3, r - 4, (r - 3) * 2, 1, sk.bodyEdge);
  px(-r + 3, -r + 3, 1, (r - 3) * 2, sk.bodyEdge);
  px(r - 4, -r + 3, 1, (r - 3) * 2, sk.bodyEdge);
  // 侧挂储物箱
  px(-r + 5, -r + 5, 7, 4, sk.bodyHi);
  px(-r + 5, r - 9, 7, 4, sk.bodyHi);

  // 舱口：深色开口 + 开口里的奶龙（参考图的核心构图）
  const cr = r - 7;
  px(-cr, -cr, cr * 2, cr * 2, sk.hatchRim);
  px(-cr + 1, -cr + 1, (cr - 1) * 2, (cr - 1) * 2, "#171a12");

  // 奶龙坐在舱口里（只露上半身，随炮塔一起转）
  ctx.save();
  ctx.translate(0, 2);
  drawNailong(sk.duck, sk.duckBelly, 0.82, t.nailongPhase || 0);
  ctx.restore();

  // 开火瞬间的炮口火光（方块星形）
  if (t.recoil > 0.55) {
    const a = (t.recoil - 0.55) / 0.45;
    ctx.globalAlpha = a;
    px(rec + r + 15, -3, 4, 6, "#fff3c4");
    px(rec + r + 17, -2, 3, 4, "#ffd166");
    px(rec + r + 15, -5, 2, 2, "#fff3c4");
    px(rec + r + 15, 3, 2, 2, "#fff3c4");
    ctx.globalAlpha = 1;
  }

  ctx.restore();
}

function drawTank(t, isPlayer, k) {
  if (!t.alive) return;
  const sk = isPlayer ? SKIN.player : SKIN.npc;
  drawHull(t, sk, k);
  drawTurret(t, isPlayer, sk, k);
}

function drawGridFloor() {
  // 像素风浅沙底 + 横向细纹（参考图的沙地就是这种横向短线纹理）
  ctx.fillStyle = "#f0d9a8";
  ctx.fillRect(0, 0, CFG.W, CFG.H);

  // 固定图案的横向沙纹（不随时间抖动）
  ctx.fillStyle = "rgba(196,164,110,0.5)";
  for (let y = 6; y < CFG.H; y += 18) {
    for (let x = 0; x < CFG.W; x += 26) {
      const h = ((x * 73856093) ^ (y * 19349663)) >>> 0;
      if (h % 5 < 2) ctx.fillRect(x + (h % 17), y + ((h >> 4) % 5), 10, 3);
    }
  }
  // 零星的小石子
  ctx.fillStyle = "rgba(176,146,100,0.6)";
  for (let y = 14; y < CFG.H; y += 44) {
    for (let x = 12; x < CFG.W; x += 52) {
      const h = ((x * 40503) ^ (y * 12289)) >>> 0;
      if (h % 7 === 0) ctx.fillRect(x + (h % 23), y, 3, 3);
    }
  }
}

function drawObstacles() {
  for (const o of G.level.obstacles) {
    ctx.save();
    if (o.kind === "rect") {
      // 像素风木箱：主体 + 十字隔板 + 深色描边（和参考图的木箱一致）
      const x = Math.round(o.x), y = Math.round(o.y);
      const w = Math.round(o.w), h = Math.round(o.h);
      ctx.fillStyle = "#c99a63";
      ctx.fillRect(x, y, w, h);
      ctx.fillStyle = "#e0b47c";
      ctx.fillRect(x + 3, y + 3, w - 6, 4);                 // 顶部受光
      ctx.fillStyle = "#a87a49";
      ctx.fillRect(x + 3, y + h - 7, w - 6, 4);             // 底部阴影
      // 十字隔板
      ctx.fillStyle = "#8a6034";
      ctx.fillRect(x + Math.round(w / 2) - 2, y + 3, 4, h - 6);
      ctx.fillRect(x + 3, y + Math.round(h / 2) - 2, w - 6, 4);
      // 像素描边
      ctx.fillStyle = "#6b4a26";
      ctx.fillRect(x, y, w, 2); ctx.fillRect(x, y + h - 2, w, 2);
      ctx.fillRect(x, y, 2, h); ctx.fillRect(x + w - 2, y, 2, h);
      // 角上的小圆点（像素钉）
      ctx.fillStyle = "#e8c896";
      for (const [cx2, cy2] of [[2, 2], [w - 5, 2], [2, h - 5], [w - 5, h - 5]]) {
        ctx.fillRect(x + cx2, y + cy2, 3, 3);
      }
    } else {
      // 像素风岩石：阶梯方块拼出的不规则圆
      const cx2 = Math.round(o.cx), cy2 = Math.round(o.cy), r = Math.round(o.r);
      const step = Math.max(3, Math.round(r / 3));
      ctx.fillStyle = "#a9a094";
      for (let dy = -r; dy < r; dy += step) {
        const half = Math.round(Math.sqrt(Math.max(0, r * r - dy * dy)));
        ctx.fillRect(cx2 - half, cy2 + dy, half * 2, step);
      }
      ctx.fillStyle = "#c2bab0";                       // 受光面
      ctx.fillRect(cx2 - Math.round(r * 0.6), cy2 - Math.round(r * 0.6), Math.round(r * 0.9), step);
      ctx.fillStyle = "#857c70";                       // 背光面
      ctx.fillRect(cx2 - Math.round(r * 0.3), cy2 + Math.round(r * 0.4), Math.round(r * 0.9), step);
    }
    ctx.restore();
  }

  // 场地上散落的奶龙（像素版，纯装饰不影响判定）
  const now = G.time;
  for (const d of G.decor) {
    ctx.save();
    ctx.translate(d.x, d.y);
    ctx.rotate(d.rot);
    ctx.globalAlpha = d.alpha;
    drawNailong(d.body, d.belly, d.scale, now * 0.9 + d.phase);
    ctx.restore();
  }
  ctx.globalAlpha = 1;
}

function drawShells() {
  for (const s of G.shells) {
    // 反弹过的炮弹换色：直射=暖黄，反弹后=青蓝，便于判断威胁来源
    const hot = s.ricochet;
    const trailRGB = hot ? "127,212,255" : "255,214,102";
    const coreColor = hot ? "#bfe9ff" : "#ffe9a8";

    for (let i = 0; i < s.trail.length; i++) {
      const p = s.trail[i];
      const a = (i + 1) / s.trail.length * (hot ? 0.42 : 0.35);
      ctx.fillStyle = `rgba(${trailRGB},${a})`;
      ctx.beginPath(); ctx.arc(p.x, p.y, CFG.SHELL_R * (0.4 + a), 0, Math.PI * 2); ctx.fill();
    }
    // 外发光
    ctx.fillStyle = hot ? "rgba(127,212,255,0.28)" : "rgba(255,214,102,0.22)";
    ctx.beginPath(); ctx.arc(s.x, s.y, CFG.SHELL_R * 2.6, 0, Math.PI * 2); ctx.fill();

    ctx.fillStyle = coreColor;
    ctx.beginPath(); ctx.arc(s.x, s.y, CFG.SHELL_R, 0, Math.PI * 2); ctx.fill();
    ctx.fillStyle = "rgba(255,255,255,0.9)";
    ctx.beginPath(); ctx.arc(s.x, s.y, CFG.SHELL_R * 0.45, 0, Math.PI * 2); ctx.fill();

    // 剩余弹跳次数：小圆点提示
    for (let i = 0; i < CFG.MAX_BOUNCES - s.bounces; i++) {
      ctx.fillStyle = "rgba(255,255,255,0.5)";
      ctx.beginPath();
      ctx.arc(s.x - 6 + i * 4, s.y - 7, 1.2, 0, Math.PI * 2);
      ctx.fill();
    }
  }
}

function drawParticles() {
  for (const q of G.particles) {
    const a = clamp(q.life / q.max, 0, 1);
    ctx.fillStyle = q.color;
    ctx.globalAlpha = a;
    ctx.beginPath(); ctx.arc(q.x, q.y, 1.5 + a * 3.5, 0, Math.PI * 2); ctx.fill();
  }
  ctx.globalAlpha = 1;
}

function drawCrosshair() {
  if (G.state !== "playing") return;
  const m = G.mouse;
  // 像素风准星：方块准心 + 四向刻度（不用圆弧，保持硬边）
  const X = Math.round(m.x), Y = Math.round(m.y);
  const dark = "rgba(58,44,24,0.75)", lite = "rgba(255,228,120,0.95)";
  const block = (x, y, w, h, c) => { ctx.fillStyle = c; ctx.fillRect(X + x, Y + y, w, h); };
  for (const c of [dark, lite]) {
    const o = c === dark ? 1 : 0;          // 深色先画一圈当描边
    const t = c === dark ? 4 : 2;          // 线宽
    // 中心点
    block(-2 - o, -2 - o, 4 + o * 2, 4 + o * 2, c);
    // 四条刻度
    block(-15 - o, -1 - o, 11 + o * 2, t + o, c);
    block(4 - o, -1 - o, 11 + o * 2, t + o, c);
    block(-1 - o, -15 - o, t + o, 11 + o * 2, c);
    block(-1 - o, 4 - o, t + o, 11 + o * 2, c);
  }
}

/** 倒计时可视化：半透明遮罩 + 像素大数字 + 剩余时间进度条 */
function drawCountdown() {
  const cx = CFG.W / 2, cy = CFG.H / 2;
  const n = Math.max(1, Math.ceil(G.countdownT));

  ctx.fillStyle = "rgba(24,18,10,0.5)";
  ctx.fillRect(0, 0, CFG.W, CFG.H);
  ctx.textAlign = "center";

  // "准备" 标题
  ctx.font = "bold 28px " + PIXFONT;
  ctx.fillStyle = "#f6efdd";
  ctx.fillText("准 备", cx, cy - 120);

  // 大数字：先画一圈深色偏移做像素描边，再画黄色主体
  ctx.font = "bold 130px " + PIXFONT;
  const offs = [[-4, 0], [4, 0], [0, -4], [0, 4], [-4, -4], [4, 4], [-4, 4], [4, -4]];
  for (const [dx, dy] of offs) {
    ctx.fillStyle = "#2a2418";
    ctx.fillText(String(n), cx + dx, cy + dy + 45);
  }
  ctx.fillStyle = "#ffd447";
  ctx.fillText(String(n), cx, cy + 45);

  // 剩余时间进度条（倒计时 → 缩到 0）
  const barW = 320, barH = 16;
  const bx = cx - barW / 2, by = cy + 130;
  ctx.fillStyle = "#2a2418";
  ctx.fillRect(bx - 4, by - 4, barW + 8, barH + 8);
  ctx.fillStyle = "#5c5638";
  ctx.fillRect(bx, by, barW, barH);
  const pct = Math.max(0, Math.min(1, G.countdownT / CFG.COUNTDOWN_SECONDS));
  ctx.fillStyle = "#ffd447";
  ctx.fillRect(bx, by, Math.round(barW * pct), barH);

  ctx.textAlign = "start";
}

function drawOverlay() {
  if (G.state === "playing") return;
  ctx.fillStyle = "rgba(28,22,12,0.74)";
  ctx.fillRect(0, 0, CFG.W, CFG.H);
  ctx.textAlign = "center";
  ctx.fillStyle = G.state === "win" ? "#ffd447" : "#ff8a5c";
  ctx.font = "bold 44px " + PIXFONT;
  ctx.fillText(G.state === "win" ? "胜  利" : "被  击  毁", CFG.W / 2, CFG.H / 2 - 20);
  ctx.fillStyle = "#f2e9d8";
  ctx.font = "16px " + PIXFONT;
  ctx.fillText(G.message, CFG.W / 2, CFG.H / 2 + 22);
  ctx.fillStyle = "#b8a985";
  ctx.font = "15px " + PIXFONT;
  ctx.fillText(
    G.state === "win"
      ? `连胜 ${G.winCount} 局 · 即将进入下一局…`
      : `连胜已清零 · 即将返回主界面…`,
    CFG.W / 2, CFG.H / 2 + 58
  );
  ctx.textAlign = "start";
}

/** 生成标题页的静态场景（一次性，之后每帧只做动画位移，避免抖动） */
function buildTitleScene() {
  const rnd = mulberry32(0x5eed1a3);
  const plush = [];
  for (let i = 0; i < 9; i++) {
    // 沿画面外围散落，避免压在中央的坦克和标题上
    let x, y, tries = 0;
    do {
      const edge = rnd();
      if (edge < 0.5) { x = 60 + rnd() * (CFG.W - 120); y = rnd() < 0.5 ? 60 + rnd() * 90 : CFG.H - 60 - rnd() * 150; }
      else { x = rnd() < 0.5 ? 60 + rnd() * 150 : CFG.W - 60 - rnd() * 150; y = 90 + rnd() * (CFG.H - 180); }
      tries++;
    } while (tries < 40 && Math.abs(x - CFG.W / 2) < 130 && Math.abs(y - CFG.H * 0.55) < 190);
    plush.push({
      x, y,
      body: ["#ffce21", "#ffd93b", "#ffb648", "#fff0c9"][Math.floor(rnd() * 4)],
      belly: ["#fffbe8", "#fff5d2", "#ffffff"][Math.floor(rnd() * 3)],
      scale: 0.42 + rnd() * 0.30,
      rot: (rnd() - 0.5) * 1.2,
      phase: rnd() * Math.PI * 2,
    });
  }
  const smoke = [];
  for (let i = 0; i < 14; i++) {
    smoke.push({
      x: rnd() * CFG.W,
      y: rnd() < 0.55 ? 40 + rnd() * 180 : CFG.H - 40 - rnd() * 200,
      r: 26 + rnd() * 46,
      phase: rnd() * Math.PI * 2,
    });
  }
  const rocks = [];
  for (let i = 0; i < 7; i++) {
    rocks.push({ x: 40 + rnd() * (CFG.W - 80), y: 40 + rnd() * (CFG.H - 80), r: 8 + rnd() * 14, rot: rnd() });
  }
  const crates = [];
  for (const [cx, cy, w, h] of [[120, 250, 74, 62], [820, 300, 66, 56], [200, 470, 60, 52]]) {
    crates.push({ x: cx + rnd() * 20, y: cy + rnd() * 20, w, h });
  }
  // 标题页的大坦克：停在场地中下方（俯视，与参考图一致）
  const tank = {
    x: CFG.W / 2, y: CFG.H * 0.615,
    r: CFG.TANK_R, hull: -Math.PI / 2, angle: -Math.PI / 2,
    alive: true, recoil: 0, track: 0, nailongPhase: 0,
  };
  G.title = { plush, smoke, rocks, crates, tank };
}

/** 像素风白烟团：用阶梯方块堆出圆润的棉花糖轮廓（参考图的烟就是块状的） */
function drawSmokePuff(x, y, r, alpha) {
  ctx.fillStyle = "rgba(255,255,255," + alpha + ")";
  const step = Math.max(4, Math.round(r / 3));
  const lobes = [[0, 0, 1], [-0.6, 0.18, 0.66], [0.6, 0.15, 0.7], [-0.18, -0.5, 0.56], [0.24, 0.44, 0.54]];
  for (const [ox, oy, sc] of lobes) {
    const rr2 = r * sc;
    for (let dy = -rr2; dy < rr2; dy += step) {
      const half = Math.round(Math.sqrt(Math.max(0, rr2 * rr2 - dy * dy)));
      ctx.fillRect(Math.round(x + ox * r) - half, Math.round(y + oy * r + dy), half * 2, step);
    }
  }
}

/** 像素风木箱（标题页用） */
function drawPixelCrate(x, y, w, h) {
  x = Math.round(x); y = Math.round(y);
  ctx.fillStyle = "#c99a63"; ctx.fillRect(x, y, w, h);
  ctx.fillStyle = "#e0b47c"; ctx.fillRect(x + 3, y + 3, w - 6, 4);
  ctx.fillStyle = "#a87a49"; ctx.fillRect(x + 3, y + h - 7, w - 6, 4);
  ctx.fillStyle = "#8a6034";
  ctx.fillRect(x + Math.round(w / 2) - 2, y + 3, 4, h - 6);
  ctx.fillRect(x + 3, y + Math.round(h / 2) - 2, w - 6, 4);
  ctx.fillStyle = "#6b4a26";
  ctx.fillRect(x, y, w, 2); ctx.fillRect(x, y + h - 2, w, 2);
  ctx.fillRect(x, y, 2, h); ctx.fillRect(x + w - 2, y, 2, h);
  ctx.fillStyle = "#e8c896";
  for (const [cx2, cy2] of [[2, 2], [w - 5, 2], [2, h - 5], [w - 5, h - 5]]) {
    ctx.fillRect(x + cx2, y + cy2, 3, 3);
  }
}

/** 像素风岩石（标题页用） */
function drawPixelRock(cx2, cy2, r) {
  cx2 = Math.round(cx2); cy2 = Math.round(cy2); r = Math.round(r);
  const step = Math.max(3, Math.round(r / 3));
  ctx.fillStyle = "#a9a094";
  for (let dy = -r; dy < r; dy += step) {
    const half = Math.round(Math.sqrt(Math.max(0, r * r - dy * dy)));
    ctx.fillRect(cx2 - half, cy2 + dy, half * 2, step);
  }
  ctx.fillStyle = "#c2bab0";
  ctx.fillRect(cx2 - Math.round(r * 0.6), cy2 - Math.round(r * 0.6), Math.round(r * 0.9), step);
  ctx.fillStyle = "#857c70";
  ctx.fillRect(cx2 - Math.round(r * 0.3), cy2 + Math.round(r * 0.4), Math.round(r * 0.9), step);
}

/** 标题页：整幅"坦克 + 奶龙"的主题画面 */
function drawTitleScreen() {
  const T = G.title;
  if (!T) return;
  const time = G.titleT;

  // ── 暖沙地面 + 车辙
  const bg = ctx.createLinearGradient(0, 0, 0, CFG.H);
  bg.addColorStop(0, "#e0c79b");
  bg.addColorStop(0.5, "#d3b184");
  bg.addColorStop(1, "#c39f6f");
  ctx.fillStyle = bg;
  ctx.fillRect(0, 0, CFG.W, CFG.H);

  ctx.save();
  ctx.globalAlpha = 0.16;
  ctx.strokeStyle = "#8a6a3f";
  ctx.lineWidth = 7;
  for (const lane of [CFG.W / 2 - 150, CFG.W / 2 - 96, CFG.W / 2 + 96, CFG.W / 2 + 150]) {
    ctx.beginPath();
    for (let y = 0; y <= CFG.H; y += 16) {
      const wob = Math.sin(y * 0.012 + lane) * 7;
      y === 0 ? ctx.moveTo(lane + wob, y) : ctx.lineTo(lane + wob, y);
    }
    ctx.stroke();
  }
  ctx.restore();

  // 沙粒
  ctx.fillStyle = "rgba(120,92,58,0.08)";
  for (let x = 0; x < CFG.W; x += 14) {
    for (let y = 0; y < CFG.H; y += 14) {
      const h = ((x * 73856093) ^ (y * 19349663)) >>> 0;
      if (h % 4 === 0) ctx.fillRect(x + (h % 9), y + ((h >> 4) % 9), 2.5, 2.5);
    }
  }

  // 阴影（左下角投过来，和参考图的光照方向一致）
  ctx.fillStyle = "rgba(120,92,58,0.16)";
  ctx.beginPath();
  ctx.moveTo(0, 0); ctx.lineTo(0, CFG.H * 0.72); ctx.lineTo(CFG.W * 0.30, CFG.H); ctx.lineTo(0, CFG.H);
  ctx.closePath(); ctx.fill();

  // ── 烟雾
  for (const s of T.smoke) {
    const pulse = 1 + Math.sin(time * 0.5 + s.phase) * 0.06;
    drawSmokePuff(s.x, s.y + Math.sin(time * 0.35 + s.phase) * 4, s.r * pulse, 0.55);
  }

  // ── 岩石（像素块）
  for (const r of T.rocks) drawPixelRock(r.x, r.y, r.r);

  // ── 木箱（像素块）
  for (const c of T.crates) drawPixelCrate(c.x, c.y, c.w, c.h);

  // ── 地面上的奶龙玩偶
  for (const d of T.plush) {
    ctx.save();
    ctx.translate(d.x, d.y);
    ctx.rotate(d.rot);
    drawNailong(d.body, d.belly, d.scale, time * 0.8 + d.phase);
    ctx.restore();
  }

  // ── 主坦克（参考图的主体：俯视、炮口朝下）
  const tk = T.tank;
  tk.nailongPhase = time;
  tk.track = time * 26;
  const k = 3.4;      // 标题坦克放大倍率（drawTank 内部有 0.5 还原，实际=逻辑坐标×k）
  // 斜向像素投影（硬边，略大于坦克本体）
  ctx.save();
  ctx.fillStyle = "rgba(120,92,58,0.18)";
  const shW = 34 * k, shH = 42 * k, offX = 8 * k, offY = 9 * k;
  const stepS = Math.max(6, Math.round(shW / 8));
  for (let dy = -shH; dy < shH; dy += stepS) {
    const half = Math.round(shW * Math.sqrt(Math.max(0, 1 - (dy / shH) ** 2)));
    ctx.fillRect(Math.round(tk.x + offX) - half, Math.round(tk.y + offY + dy), half * 2, stepS);
  }
  ctx.restore();

  // 天线（细杆 + 顶端红点 + 冒出的白烟），位置按新坦克尺寸（顶缘在 -19k）
  const antX = Math.round(tk.x + 2);
  const antTop = Math.round(tk.y - 34 * k);
  const antBot = Math.round(tk.y - 16 * k);
  ctx.fillStyle = "#2f371a";
  ctx.fillRect(antX, antTop, Math.max(2, Math.round(k * 0.7)), antBot - antTop);
  ctx.fillStyle = "#d9463c";
  ctx.fillRect(antX - Math.round(k * 0.5), antTop - Math.round(k * 0.9), Math.round(k * 1.5), Math.round(k * 1.3));
  // 天线顶端冒的烟（随时间向上飘）
  for (let i = 0; i < 3; i++) {
    const ph = (time * 0.55 + i * 0.34) % 1;
    const sy = antTop - 8 - ph * 46;
    const sx = antX + 18 + ph * 42;
    drawSmokePuff(sx, sy, 8 + ph * 12, 0.85 * (1 - ph));
  }

  drawTank(tk, true, k);

  // ── 题目与提示
  const cx = CFG.W / 2;

  // 标题底板（深色半透明，保证压在沙滩上也看得清）
  ctx.save();
  ctx.fillStyle = "rgba(38,30,16,0.26)";
  ctx.fillRect(cx - 252, 74, 504, 112);
  ctx.restore();

  ctx.textAlign = "center";
  ctx.fillStyle = "rgba(60,42,18,0.55)";
  ctx.font = "bold 66px ${PIXFONT}";
  ctx.fillText("奶 龙 坦 克", cx + 4, 156);        // 描边阴影
  const tg = ctx.createLinearGradient(0, 90, 0, 160);
  tg.addColorStop(0, "#fff6d8");
  tg.addColorStop(1, "#ffd447");
  ctx.fillStyle = tg;
  ctx.fillText("奶 龙 坦 克", cx, 152);

  ctx.font = "16px ${PIXFONT}";
  ctx.fillStyle = "rgba(52,38,18,0.85)";
  ctx.fillText("开 着 玩 具 坦 克 ， 带 着 奶 龙 一 起 出 击", cx, 212);

  // 开始提示（呼吸闪烁）
  const blink = 0.62 + 0.38 * (0.5 + 0.5 * Math.sin(time * 3.2));
  ctx.save();
  ctx.globalAlpha = blink;
  ctx.fillStyle = "rgba(38,30,16,0.42)";
  ctx.fillRect(cx - 212, CFG.H - 120, 424, 56);
  ctx.fillStyle = "#fff8e2";
  ctx.font = "bold 24px ${PIXFONT}";
  ctx.fillText("点 击 任 意 位 置 开 始", cx, CFG.H - 80);
  ctx.restore();

  // 操作说明
  ctx.font = "15px ${PIXFONT}";
  ctx.fillStyle = "rgba(46,34,16,0.8)";
  ctx.fillText("W A S D 移动　　鼠标瞄准　　右键发射　　每发一炮、共 5 发　　打空后 10 秒续弹", cx, CFG.H - 34);
  ctx.fillText("炮弹会撞墙反弹，最多 3 次 —— 躲在掩体后面也不一定安全", cx, CFG.H - 12);
  ctx.textAlign = "start";
}

function draw() {
  if (G.screen === "title") { drawTitleScreen(); updateHud(); return; }
  drawGridFloor();
  drawObstacles();
  drawNpcTelegraph();
  drawShells();
  for (const n of G.npcs) drawTank(n, false);
  drawTank(G.player, true);
  drawParticles();
  drawCrosshair();
  drawOverlay();
  if (G.screen === "countdown") drawCountdown();
  updateHud();
}

function updateHud() {
  const now = performance.now();
  const p = G.player;
  // 连胜次数：任何界面都显示
  document.getElementById("hudWins").textContent = String(G.winCount);
  // 开屏页：弹药等数据尚未参与对局，只显示等待状态
  if (G.screen === "title") {
    document.getElementById("hudShells").textContent = "— / 5";
    const c0 = document.getElementById("hudCool");
    c0.textContent = "待开始";
    c0.className = "";
    document.getElementById("hudEnemy").textContent = "—";
    document.getElementById("hudLive").textContent = "0";
    return;
  }
  document.getElementById("hudShells").textContent = `${p.shotsLeft} / ${CFG.MAX_SHELLS}`;
  const cool = document.getElementById("hudCool");
  if (p.shotsLeft > 0) {
    cool.textContent = "就绪";
    cool.className = "ok";
  } else {
    const left = Math.max(0, (p.cooldownEnd - now) / 1000);
    cool.textContent = `续弹 ${left.toFixed(1)} s`;
    cool.className = "warn";
  }
  // 顺带显示 NPC 的弹药状态，便于确认它遵守同一套规则
  const n = G.npcs.find(x => x.alive);
  const e = document.getElementById("hudEnemy");
  if (n) {
    const s = n.shotsLeft > 0 ? `弹${n.shotsLeft}` : `冷却${Math.max(0, (n.cooldownEnd - now) / 1000).toFixed(0)}s`;
    e.textContent = `1 (${s})`;
  } else {
    e.textContent = "0";
  }
  // 空中炮弹数（含已反弹的）
  const live = G.shells.filter(s => s.alive);
  const bounced = live.filter(s => s.bounces > 0).length;
  document.getElementById("hudLive").textContent =
    bounced > 0 ? `${live.length} (弹跳${bounced})` : String(live.length);
}

/* =========================================================================
   11. 启动
   ========================================================================= */
newGame();
requestAnimationFrame(loop);
</script>
</body>
</html>
