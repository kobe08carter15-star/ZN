// ═══════════════════════════════════════════════════
// GBFR 专精树系统 — 风格(觉醒/真谛/秘义) × 阶级(1/2/3/EX)
// 数据来自 reference 站（依据游戏内截图校正），UI 与因子配装器一致的粉色基调
// ═══════════════════════════════════════════════════
(function() {
'use strict';

// 每个阶级全局可用点数（跨三种风格合计）
var RANK_MAX = { '1': 10, '2': 10, '3': 10, 'ex': 20 };
// 阶级符号
var SYMBOLS = { '1': '●', '2': '◆', '3': '⬢', 'ex': '⬣' };
// 阶级中文名
var RANK_LABEL = { '1': '1 阶', '2': '2 阶', '3': '3 阶', 'ex': 'EX 阶' };

// ─── 模拟器角色列表（仅保留有专精数据的角色） ───
var SIMULATOR_CHARACTERS = [];
function buildSimulatorCharList() {
  if (SIMULATOR_CHARACTERS.length) return;
  if (typeof CHARACTERS !== 'undefined' && CHARACTERS.length) {
    for (var i = 0; i < CHARACTERS.length; i++) {
      var id = CHARACTERS[i].id;
      if (getCharData(id)) SIMULATOR_CHARACTERS.push(id);
    }
  }
  if (!SIMULATOR_CHARACTERS.length && typeof MASTERY_DATA !== 'undefined') {
    SIMULATOR_CHARACTERS = Object.keys(MASTERY_DATA);
  }
}

// ─── 数据查找 ───
function getCharData(simName) {
  if (typeof MASTERY_DATA === 'undefined') return null;
  return MASTERY_DATA[simName] || null;
}
function getDisplayName(simName) {
  return simName;
}

// ─── 状态集成 ───
// state.masteryTree[simChar] = { '觉醒': {'1':[idx],'2':[...],'3':[...],'ex':[...]}, '真谛':{...}, '秘义':{...} }
function ensureMTState() {
  if (!window.state) return;
  if (!window.state.masteryTree) window.state.masteryTree = {};
}
function getMTCharState(simCharName) {
  ensureMTState();
  if (!window.state.masteryTree[simCharName]) window.state.masteryTree[simCharName] = {};
  return window.state.masteryTree[simCharName];
}
function getMTStyleSel(simCharName, styleType) {
  var cs = getMTCharState(simCharName);
  if (!cs[styleType]) cs[styleType] = {};
  return cs[styleType];
}
function getMTRankSel(simCharName, styleType, rank) {
  var ss = getMTStyleSel(simCharName, styleType);
  if (!ss[rank]) ss[rank] = [];
  return ss[rank];
}

// 全局某阶级已激活节点数（跨三种风格）
function globalRankSelected(simCharName, rank) {
  var cs = getMTCharState(simCharName);
  var total = 0;
  for (var st in cs) {
    if (cs[st] && cs[st][rank]) total += cs[st][rank].length;
  }
  return total;
}

// ─── 数值提取（用于人物属性联动） ───
function extractStat(text, stats) {
  var m;
  m = text.match(/最大HP\s*\+\s*([\d,]+)/); if (m) stats.hpFlat += parseInt(m[1].replace(/,/g, ''), 10);
  m = text.match(/攻击力\s*\+\s*(\d+)%/); if (m) stats.atkPct += parseInt(m[1], 10);
  m = text.match(/暴击率\s*\+\s*(\d+)%/); if (m) stats.crit += parseInt(m[1], 10);
  m = text.match(/伤害上限\s*\+\s*(\d+)%/); if (m) stats.cap += parseInt(m[1], 10);
  m = text.match(/暴击伤害\s*\+\s*(\d+)%/); if (m) stats.critDmg += parseInt(m[1], 10);
  m = text.match(/防御力\s*\+\s*(\d+)%/); if (m) stats.defPct += parseInt(m[1], 10);
}
// 供属性计算使用的提取：排除会映射到"人物专精"计数器(state.mastery)的节点，
// 避免与因子配装界面重复计算（20%/30% 攻击力、最大HP+15000 改由 state.mastery 体现）
// 识别 EX 阶特殊节点：攻击力+20% / 伤害上限+50% 且带自身弱化副作用
function isExAtkWeaknessNode(text) {
  if (!text) return false;
  return /攻击力\s*\+\s*20%/.test(text) && /伤害上限\s*\+\s*50%/.test(text) && /自身弱化/.test(text);
}

// 严格匹配"单纯的攻击力+X%"独立节点（排除带前缀/后缀/复合条件的类似文本）
function isPureAtkNode(text, pct) {
  if (!text) return false;
  return new RegExp('^攻击力\\s*\\+\\s*' + pct + '%$').test(text);
}
function extractStatCalc(text, stats) {
  if (text == null) return;
  if (/最大HP\s*\+\s*15,000/.test(text)) return; // → state.mastery.hp
  var m = text.match(/攻击力\s*\+\s*(\d+)%/);
  if (m) {
    var pct = parseInt(m[1], 10);
    // EX 特殊节点单独计入局外攻击力，不占用人物专精 +20% 的 3 次上限
    if (pct === 20 && isExAtkWeaknessNode(text)) {
      stats.atkPctOut = (stats.atkPctOut || 0) + pct;
      // 继续提取该节点上的其他属性（如伤害上限+50%）
    } else if (isPureAtkNode(text, pct)) {
      return; // → state.mastery.atk / atk30
    } else {
      stats.atkPct = (stats.atkPct || 0) + pct;
      // 继续提取该节点上的其他属性（复合节点可能同时带伤害上限等）
    }
  }
  m = text.match(/暴击率\s*\+\s*(\d+)%/); if (m) stats.crit += parseInt(m[1], 10);
  m = text.match(/伤害上限\s*\+\s*(\d+)%/); if (m) stats.cap += parseInt(m[1], 10);
  m = text.match(/暴击伤害\s*\+\s*(\d+)%/); if (m) stats.critDmg += parseInt(m[1], 10);
  m = text.match(/防御力\s*\+\s*(\d+)%/); if (m) stats.defPct += parseInt(m[1], 10);
}

// ─── 渲染专精树视图 ───
function renderMasteryView() {
  var sel = document.getElementById('mt-char-select');
  var container = document.getElementById('mt-columns');
  if (!sel || !container) return;

  var simCharName = sel.value;
  var cd = getCharData(simCharName);
  if (!cd || cd.missing) {
    container.innerHTML = '<div style="padding:24px;color:var(--accent-dark);font-size:0.95em;text-align:center;line-height:1.6;">'
      + (cd && cd.missing
          ? '角色「' + escapeHtml(simCharName) + '」的专精数据暂缺。<br>当前权威数据源（RPG Site 攻略）未收录该角色，待补充。'
          : '未找到该角色的专精数据')
      + '</div>';
    var pb = document.getElementById('mt-points'); if (pb) pb.innerHTML = '';
    var tb = document.getElementById('mt-totals'); if (tb) tb.innerHTML = '';
    var sm = document.getElementById('mt-summary'); if (sm) sm.innerHTML = '';
    return;
  }

  // 更新顶部剩余点数 & 总激活数
  updatePointsBar(simCharName);

  var html = '';
  for (var si = 0; si < cd.styles.length; si++) {
    var style = cd.styles[si];
    var emblem = ['✦', '✦✦', '✦✦✦'][si] || '✦';
    var styleSel = getMTStyleSel(simCharName, style.type);

    html += '<div class="mt-column">';
    html += '<div class="mt-col-header">';
    html += '<div class="mt-col-emblem">' + emblem + '</div>';
    html += '<div class="mt-col-namewrap">';
    html += '<div class="mt-col-type">' + style.type + '</div>';
    html += '<div class="mt-col-name">' + style.name + '</div>';
    html += '</div>';
    html += '</div>';
    html += '<div class="mt-col-body">';

      var styleSelected = 0;
      for (var ri = 0; ri < style.ranks.length; ri++) {
        var rank = style.ranks[ri];
        var rankKey = String(rank.rank);
        var sym = SYMBOLS[rankKey] || '?';
        var cap = rank.cap || 0;
        var reqVal = (rank.req === undefined ? null : rank.req);
        var selRank = styleSel[rankKey] || [];
        var selCount = selRank.length;
        styleSelected += selCount;
        var met = reqVal != null && selCount >= reqVal;
        var perkProg = reqVal != null ? Math.min(selCount, reqVal) : 0;

        html += '<div class="mt-tier">';
        // tier header
        html += '<div class="mt-tier-header">';
        html += '<div class="mt-tier-title">';
        html += '<span class="mt-tier-symbol">' + sym + '</span>';
        html += '<span class="mt-tier-label">' + RANK_LABEL[rankKey] + '专精技能</span>';
        html += '</div>';
        if (reqVal != null) {
          html += '<span class="mt-tier-count' + (met ? ' met' : '') + '">' + perkProg + '/' + reqVal + '</span>';
        } else {
          html += '<span class="mt-tier-count">' + selCount + ' 节点</span>';
        }
        html += '</div>';

        // progress bar (perk 进度)
        if (reqVal != null) {
          var pct = Math.min(100, (perkProg / reqVal) * 100);
          html += '<div class="mt-tier-bar"><div class="mt-tier-bar-fill' + (met ? ' met' : '') + '" style="width:' + pct + '%"></div></div>';
        }

        // node list — 统一 2 列网格矩阵：1阶2x2 2阶4x2 3阶4x2 EX阶5x2
        html += '<div class="mt-node-list" data-mt-rank="' + rankKey + '">';
        var nodeCount = rank.nodes.length;
        for (var ni = 0; ni < nodeCount; ni++) {
          var node = rank.nodes[ni];
          var text = (typeof node === 'object') ? node.t : node;
          var checked = selRank.indexOf(ni) >= 0;
          html += '<div class="mt-node' + (checked ? ' checked' : '') + '" '
              + 'data-mt-char="' + escapeAttr(simCharName) + '" '
              + 'data-mt-style="' + escapeAttr(style.type) + '" '
              + 'data-mt-rank="' + rankKey + '" '
              + 'data-mt-idx="' + ni + '" '
              + 'onclick="MTH.toggle(this)">';
          html += '<span class="mt-node-badge"></span>';
          html += '<span class="mt-node-text">' + renderEffectText(text) + '</span>';
          html += '</div>';
        }
        // 占位补全：末行不足 2 列时插入不可见占位格，保持网格对齐
        if (nodeCount % 2 !== 0) {
          html += '<div class="mt-node-placeholder"></div>';
        }
        html += '</div>';

        // perk
        if (rank.perk) {
          var pipsHtml = '';
          for (var pi = 0; pi < rank.perk.pips; pi++) pipsHtml += '<span class="mt-pip' + (pi < perkProg ? ' on' : '') + '"></span>';
          html += '<div class="mt-perk' + (met ? ' active' : '') + '">';
          html += '<div class="mt-perk-pips">' + pipsHtml + '</div>';
          for (var ei = 0; ei < rank.perk.effects.length; ei++) {
            html += '<div class="mt-perk-effect">' + renderEffectText(rank.perk.effects[ei]) + '</div>';
          }
          html += '</div>';
        }

        html += '</div>';
      }

    html += '<div class="mt-col-points">已选 ' + styleSelected + ' 节点</div>';
    html += '</div>';
    html += '</div>';
  }

  container.innerHTML = html;
  updateEffectSummary(simCharName);
}

function updatePointsBar(simCharName) {
  var ranks = ['1', '2', '3', 'ex'];
  // 剩余点数
  var ptsEl = document.getElementById('mt-points');
  if (ptsEl) {
    var hp = '';
    for (var i = 0; i < ranks.length; i++) {
      var r = ranks[i];
      var left = RANK_MAX[r] - globalRankSelected(simCharName, r);
      hp += '<span class="mt-point' + (left <= 0 ? ' used' : '') + '"><b>' + SYMBOLS[r] + '</b> 剩余 ' + left + '</span>';
    }
    ptsEl.innerHTML = hp;
  }
  // 总激活数
  var totEl = document.getElementById('mt-totals');
  if (totEl) {
    var th = '';
    for (var j = 0; j < ranks.length; j++) {
      var r2 = ranks[j];
      var sel = globalRankSelected(simCharName, r2);
      th += '<span class="mt-total' + (sel >= RANK_MAX[r2] ? ' full' : '') + '"><b>' + SYMBOLS[r2] + '</b> ' + sel + '/' + RANK_MAX[r2] + '</span>';
    }
    totEl.innerHTML = th;
  }
}

function updateEffectSummary(simCharName) {
  var summaryEl = document.getElementById('mt-summary');
  if (!summaryEl) return;
  var stats = window.getMasteryTreeStats(simCharName);
  var items = [];
  if (stats.atkPct > 0) items.push('攻击力 +' + stats.atkPct + '%');
  if (stats.hpFlat > 0) items.push('Max体力 +' + stats.hpFlat.toLocaleString('en-US'));
  if (stats.crit > 0) items.push('暴击率 +' + stats.crit + '%');
  if (stats.cap > 0) items.push('伤害上限 +' + stats.cap + '%');
  if (stats.critDmg > 0) items.push('暴击伤害 +' + stats.critDmg + '%');
  if (stats.defPct > 0) items.push('防御力 +' + stats.defPct + '%');
  if (items.length === 0) {
    summaryEl.innerHTML = '<span style="color:#bbb;">尚未选择效果节点</span>';
  } else {
    summaryEl.innerHTML = items.map(function(t) { return '<span class="mt-summary-item">' + t + '</span>'; }).join('');
  }
}

function escapeHtml(text) {
  return String(text).replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;');
}
function escapeAttr(text) {
  return escapeHtml(text).replace(/"/g, '&quot;');
}
// 渲染效果文本：将 "✦ N阶专精生效时：" 前缀高亮为条件标签
function renderEffectText(text) {
  if (text == null) return '';
  var m = /^(✦+\s*\d+阶专精生效时：)([\s\S]*)$/.exec(text);
  if (m) {
    return '<span class="mt-node-req">' + escapeHtml(m[1]) + '</span>' + escapeHtml(m[2]);
  }
  return escapeHtml(text);
}

// ─── 节点切换 ───
window.MTH = {
  toggle: function(el) {
    var simCharName = el.getAttribute('data-mt-char');
    var styleType = el.getAttribute('data-mt-style');
    var rankKey = el.getAttribute('data-mt-rank');
    var idx = parseInt(el.getAttribute('data-mt-idx'), 10);

    ensureMTState();
    var sel = getMTRankSel(simCharName, styleType, rankKey);
    var pos = sel.indexOf(idx);
    var cd = getCharData(simCharName);
    var style = null, rank = null;
    for (var si = 0; si < cd.styles.length; si++) {
      if (cd.styles[si].type === styleType) { style = cd.styles[si]; break; }
    }
    if (style) {
      for (var ri = 0; ri < style.ranks.length; ri++) {
        if (String(style.ranks[ri].rank) === rankKey) { rank = style.ranks[ri]; break; }
      }
    }

    if (pos >= 0) {
      sel.splice(pos, 1);
    } else {
      // 检查限制
      var cap = rank ? (rank.cap || 0) : 0;
      if (cap > 0 && sel.length >= cap) {
        alert('「' + styleType + '」' + RANK_LABEL[rankKey] + '最多只能选择 ' + cap + ' 个节点');
        return;
      }
      if (globalRankSelected(simCharName, rankKey) >= RANK_MAX[rankKey]) {
        alert(RANK_LABEL[rankKey] + '专精点数已用完（上限 ' + RANK_MAX[rankKey] + '）');
        return;
      }
      sel.push(idx);
    }

    // 保存滚动位置：renderMasteryView 内部 container.innerHTML = html 全量重建 DOM，
    // 导致 mt-columns 及每列 mt-col-body 的 scrollTop 都被重置为 0。
    // 需要保存并恢复：页面级 + 弹窗内每列 body 的滚动位置。
    var savedPageScrollY = window.scrollY || window.pageYOffset || 0;
    var columnsContainer = document.getElementById('mt-columns');
    var savedColScrollTops = [];
    if (columnsContainer) {
      var colBodies = columnsContainer.querySelectorAll('.mt-col-body');
      for (var ci = 0; ci < colBodies.length; ci++) {
        savedColScrollTops.push(colBodies[ci].scrollTop);
      }
    }

    renderMasteryView();
    if (typeof recalcAll === 'function') recalcAll();
    if (typeof autoSave === 'function') autoSave();

    // 用双重 requestAnimationFrame 确保浏览器完成所有 layout/paint 后再恢复
    // 单次 RAF 可能先于 DOM 布局完成触发
    var restoreScrollPositions = function() {
      window.scrollTo(0, savedPageScrollY);
      var newContainer = document.getElementById('mt-columns');
      if (newContainer && savedColScrollTops.length > 0) {
        var newColBodies = newContainer.querySelectorAll('.mt-col-body');
        for (var ri = 0; ri < Math.min(savedColScrollTops.length, newColBodies.length); ri++) {
          newColBodies[ri].scrollTop = savedColScrollTops[ri];
        }
      }
    };
    requestAnimationFrame(function() {
      restoreScrollPositions();
      // 保险：下一帧再次恢复，防止异步布局尚未完成
      requestAnimationFrame(restoreScrollPositions);
    });
  },

  open: function(simCharName) {
    ensureMTState();
    buildSimulatorCharList();
    var sel = document.getElementById('mt-char-select');
    var container = document.getElementById('mt-columns');
    if (!sel || !container) return;

    if (sel.options.length === 0) {
      for (var i = 0; i < SIMULATOR_CHARACTERS.length; i++) {
        var id = SIMULATOR_CHARACTERS[i];
        var opt = document.createElement('option');
        opt.value = id;
        opt.textContent = getDisplayName(id);
        sel.appendChild(opt);
      }
    }

    if (sel.options.length === 0) {
      container.innerHTML = '<div style="padding:20px;color:var(--accent-dark);font-size:0.95em;text-align:center;">未加载专精数据，请确认 mastery_data_compact.js 已正确放置。</div>';
      document.getElementById('mt-overlay').classList.add('active');
      return;
    }

    if (simCharName && typeof simCharName === 'string') {
      var found = false;
      for (var j = 0; j < sel.options.length; j++) {
        if (sel.options[j].value === simCharName) { sel.selectedIndex = j; found = true; break; }
      }
      if (!found && sel.options.length > 0) sel.selectedIndex = 0;
    }

    var overlay = document.getElementById('mt-overlay');
    if (overlay) overlay.classList.add('active');
    renderMasteryView();
  },

  close: function() {
    var overlay = document.getElementById('mt-overlay');
    if (overlay) overlay.classList.remove('active');
  },

  onCharChange: function() {
    renderMasteryView();
  },

  reset: function() {
    var sel = document.getElementById('mt-char-select');
    if (!sel) return;
    var simCharName = sel.value;
    if (confirm('确定清空「' + simCharName + '」的全部专精选择？')) {
      ensureMTState();
      window.state.masteryTree[simCharName] = {};
      // 专精树清空 → 人物专精计数器同步归零（仅当前配装角色）
      if (typeof window.state !== 'undefined' && window.state.character === simCharName && window.state.mastery) {
        window.state.mastery.hp = 0;
        window.state.mastery.atk = 0;
        window.state.mastery.atk30 = 0;
        window.state.mastery.synBasic = 0;
        window.state.mastery.synAtk = 0;
        window.state.mastery.synDef = 0;
      }
      renderMasteryView();
      if (typeof recalcAll === 'function') recalcAll();
      if (typeof autoSave === 'function') autoSave();
    }
  },

  // ─── 构建专精树截图紧凑视图（风格与因子配装器截图一致）───
  buildScreenshotView: function() {
    var sel = document.getElementById('mt-char-select');
    var simCharName = sel ? sel.value : '';
    var cd = getCharData(simCharName);
    var html = '';

    // 统计各风格选中数
    var totalSel = 0;
    var stylesSummary = [];
    for (var si = 0; si < cd.styles.length; si++) {
      var style = cd.styles[si];
      var styleSel = getMTStyleSel(simCharName, style.type);
      var cnt = 0;
      for (var ri = 0; ri < style.ranks.length; ri++) {
        var rankKey = String(style.ranks[ri].rank);
        cnt += (styleSel[rankKey] || []).length;
      }
      totalSel += cnt;
      stylesSummary.push({ type: style.type, name: style.name, count: cnt });
    }

    // ═══ Header ═══
    html += '<div class="sv-header">';
    html += '<h1>专精树 — ' + escapeHtml(simCharName) + '</h1>';
    html += '<div class="sv-sub">觉醒 / 真谛 / 秘义</div>';
    var parts = [];
    for (var pi = 0; pi < stylesSummary.length; pi++) {
      parts.push(stylesSummary[pi].type + '·' + stylesSummary[pi].count);
    }
    html += '<div style="margin-top:0.3em;font-size:0.8em;color:#9e7a8c;">已选 ' + totalSel + ' 节点（' + parts.join('  ') + '）</div>';
    html += '</div>';

    if (totalSel === 0) {
      html += '<div style="padding:24px;text-align:center;color:#c48da0;">暂未选择任何节点</div>';
      return html;
    }

    // ═══ 各风格 Section ═══
    var emblems = ['✦', '✦✦', '✦✦✦'];
    for (var si = 0; si < cd.styles.length; si++) {
      var style = cd.styles[si];
      var styleSel = getMTStyleSel(simCharName, style.type);
      var emblem = emblems[si] || '✦';

      var hasAnyInStyle = false;
      for (var ri = 0; ri < style.ranks.length; ri++) {
        var rankKey = String(style.ranks[ri].rank);
        if ((styleSel[rankKey] || []).length > 0) { hasAnyInStyle = true; break; }
      }
      if (!hasAnyInStyle) continue;

      html += '<div class="sv-section" style="margin-bottom:20px;padding-bottom:12px;border-bottom:2px solid #f48fb1;">';
      html += '<div class="sv-section-title" style="font-size:17px;padding-bottom:6px;margin-bottom:12px;">'
        + emblem + ' ' + style.type + ' — ' + style.name + '</div>';

      for (var ri = 0; ri < style.ranks.length; ri++) {
        var rank = style.ranks[ri];
        var rankKey = String(rank.rank);
        var sym = SYMBOLS[rankKey] || '?';
        var reqVal = (rank.req === undefined ? null : rank.req);
        var selRank = styleSel[rankKey] || [];
        var selCount = selRank.length;
        if (selCount === 0) continue;

        var met = reqVal != null && selCount >= reqVal;
        var perkProg = reqVal != null ? Math.min(selCount, reqVal) : 0;

        // 阶位标签：更醒目，并作为各阶分隔
        html += '<div style="margin-bottom:16px;padding-bottom:8px;border-bottom:1px solid #f0c4d4;">';
        html += '<div style="display:flex;align-items:center;justify-content:space-between;font-size:15px;font-weight:800;color:#c2185b;background:#fff5f8;border-left:4px solid #f48fb1;border-radius:0 4px 4px 0;padding:7px 10px;margin-bottom:8px;">'
          + '<span>' + sym + ' ' + RANK_LABEL[rankKey] + '专精技能</span>'
          + '<span style="font-size:13px;font-weight:600;color:#9e7a8c;">' + selCount + '/' + rank.nodes.length + ' 节点</span></div>';

        // 节点：2 列网格 + 边框，显示全部节点
        html += '<div class="sv-summary-grid" style="gap:4px 8px;">';
        for (var ni = 0; ni < rank.nodes.length; ni++) {
          var node = rank.nodes[ni];
          var text = (typeof node === 'object') ? node.t : node;
          var isSel = (selRank.indexOf(ni) >= 0);
          var itemBg = isSel ? 'background:#fce4ec;border-color:#f48fb1;' : 'background:#fff;border-color:#eee;';
          var textColor = isSel ? 'color:#6d4c5a;font-weight:600;' : 'color:#c4a4b4;';
          html += '<div class="sv-summary-item" style="border:1px solid ' + (isSel ? '#f48fb1' : '#eee') + ';border-radius:3px;padding:5px 8px;margin:0;' + itemBg + '">'
            + '<span class="sv-sname" style="font-size:14px;' + textColor + '">' + renderEffectText(text) + '</span></div>';
        }
        html += '</div>';

        // Perk：总是显示，带激活进度 pip
        if (rank.perk) {
          var pipsHtml = '';
          for (var pi = 0; pi < rank.perk.pips; pi++) {
            var pipOn = (pi < perkProg);
            pipsHtml += '<span style="display:inline-block;width:10px;height:10px;border-radius:50%;margin-right:4px;background:'
              + (pipOn ? '#4caf50' : '#e0e0e0') + ';border:1px solid ' + (pipOn ? '#43a047' : '#d6d6d6') + ';"></span>';
          }
          var perkBg = met ? '#fff5f8' : '#fafafa';
          var perkBorder = met ? '#f48fb1' : '#e0e0e0';
          var perkTitle = met ? '🎯 已激活特效' : '⚪ 专精觉醒（未激活）';
          var titleColor = met ? '#c2185b' : '#9e9e9e';
          html += '<div style="background:' + perkBg + ';border-left:3px solid ' + perkBorder + ';padding:8px 12px;margin-top:8px;border-radius:3px;">';
          html += '<div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:4px;">';
          html += '<div style="font-weight:700;color:' + titleColor + ';font-size:14px;">' + perkTitle + '</div>';
          html += '<div style="font-size:0;">' + pipsHtml + '</div>';
          html += '</div>';
          for (var ei = 0; ei < rank.perk.effects.length; ei++) {
            var effectColor = met ? '#6d4c5a' : '#b0b0b0';
            html += '<div class="sv-entry" style="padding:3px 0;border-bottom:none;">'
              + '<span class="sv-ename" style="font-size:15px;font-weight:500;line-height:1.55;color:' + effectColor + ';">'
              + renderEffectText(rank.perk.effects[ei]) + '</span></div>';
          }
          html += '</div>';
        }

        html += '</div>';
      }

      html += '</div>';
    }


    // ═══ Total bar ═══
    html += '<div class="sv-total-bar"><span>总激活节点</span><span>' + totalSel + '</span></div>';

    return html;
  },

  captureScreenshot: function() {
    try {
      var sel = document.getElementById('mt-char-select');
      if (!sel) {
        if (typeof showToast === 'function') showToast('截图失败：未找到角色选择', 'error');
        return;
      }
      var simCharName = sel.value;
      var cd = getCharData(simCharName);
      if (!cd || cd.missing) {
        if (typeof showToast === 'function') showToast('截图失败：无专精数据', 'error');
        return;
      }
      if (typeof html2canvas === 'undefined') {
        if (typeof showToast === 'function') showToast('截图库未加载，请刷新页面重试', 'error');
        return;
      }

      if (typeof showToast === 'function') showToast('正在生成完整截图...', 'info');

      // 构建紧凑截图视图到 #screenshotView（容器宽度与因子配装器一致，scale 2 导出为 840px）
      var sv = document.getElementById('screenshotView');
      sv.innerHTML = this.buildScreenshotView();
      sv.style.width = '420px';
      sv.style.maxWidth = 'none';
      sv.style.display = 'block';

      html2canvas(sv, {
        backgroundColor: '#ffffff',
        scale: 2,
        useCORS: true,
        logging: false,
        width: sv.scrollWidth,
        height: sv.scrollHeight
      }).then(function(canvas) {
        sv.style.display = 'none';

        var now = new Date();
        var dateStr = now.toISOString().slice(0, 10);
        var filename = 'gbf-mastery-' + simCharName + '-' + dateStr + '.png';
        var timeStr = now.getFullYear() + '-' +
          String(now.getMonth() + 1).padStart(2, '0') + '-' +
          String(now.getDate()).padStart(2, '0') + ' ' +
          String(now.getHours()).padStart(2, '0') + ':' +
          String(now.getMinutes()).padStart(2, '0') + ':' +
          String(now.getSeconds()).padStart(2, '0');
        var dataUrl = canvas.toDataURL('image/png');

        if (typeof MTH.close === 'function') MTH.close();

        var srImage = document.getElementById('srImage');
        var srInfo = document.getElementById('srInfo');
        var srOverlay = document.getElementById('srOverlay');
        if (srImage && srInfo && srOverlay) {
          window.lastScreenshot = { dataUrl: dataUrl, filename: filename };
          srImage.src = dataUrl;
          srInfo.innerHTML = '<div class="sr-info-item"><span class="sr-info-label">📄</span><span class="sr-info-value">' + filename + '</span></div>' +
            '<div class="sr-info-item"><span class="sr-info-label">🕐</span><span class="sr-info-value">' + timeStr + '</span></div>' +
            '<div class="sr-info-item"><span class="sr-info-label">📐</span><span class="sr-info-value">' + canvas.width + ' × ' + canvas.height + ' px</span></div>';
          srOverlay.classList.remove('hidden');
          if (typeof showToast === 'function') showToast('截图已生成，可下载保存', 'success');
        } else {
          var a = document.createElement('a');
          a.href = dataUrl;
          a.download = filename;
          document.body.appendChild(a);
          a.click();
          document.body.removeChild(a);
          if (typeof showToast === 'function') showToast('截图已下载', 'success');
        }
      }).catch(function(err) {
        sv.style.display = 'none';
        console.error('专精树截图失败:', err);
        if (typeof showToast === 'function') showToast('截图失败：' + (err.message || '未知错误'), 'error');
      });
    } catch (e) {
      console.error('专精树截图异常:', e);
      if (typeof showToast === 'function') showToast('截图异常：' + (e.message || '未知错误'), 'error');
    }
  }
};

// ─── 人物属性联动 ───
window.getMasteryTreeStats = function(simCharName) {
  var stats = { hpFlat: 0, atkPct: 0, crit: 0, cap: 0, critDmg: 0, defPct: 0 };
  if (!simCharName) return stats;
  var cd = getCharData(simCharName);
  if (!cd) return stats;
  var cs = getMTCharState(simCharName);
  for (var si = 0; si < cd.styles.length; si++) {
    var style = cd.styles[si];
    var styleSel = cs[style.type] || {};
    for (var ri = 0; ri < style.ranks.length; ri++) {
      var rank = style.ranks[ri];
      var sel = styleSel[String(rank.rank)] || [];
      for (var ni = 0; ni < sel.length; ni++) {
        var node = rank.nodes[sel[ni]];
        var text = (typeof node === 'object') ? node.t : node;
        extractStat(text, stats);
      }
    }
  }
  return stats;
};
// 供属性计算使用：排除"人物专精"计数器已覆盖的节点，避免重复计算
window.getMasteryTreeCalcStats = function(simCharName) {
  var stats = { hpFlat: 0, atkPct: 0, atkPctOut: 0, crit: 0, cap: 0, critDmg: 0, defPct: 0 };
  if (!simCharName) return stats;
  var cd = getCharData(simCharName);
  if (!cd) return stats;
  var cs = getMTCharState(simCharName);
  for (var si = 0; si < cd.styles.length; si++) {
    var style = cd.styles[si];
    var styleSel = cs[style.type] || {};
    for (var ri = 0; ri < style.ranks.length; ri++) {
      var rank = style.ranks[ri];
      var sel = styleSel[String(rank.rank)] || [];
      for (var ni = 0; ni < sel.length; ni++) {
        var node = rank.nodes[sel[ni]];
        var text = (typeof node === 'object') ? node.t : node;
        extractStatCalc(text, stats);
      }
    }
  }
  return stats;
};
// 统计专精树已激活的"人物专精"类节点数量（用于联动因子配装界面）
window.getMasteryTreeMasteryCounts = function(simCharName) {
  var counts = { hp: 0, atk20: 0, atk30: 0, synBasic: 0, synAtk: 0, synDef: 0, synDef2: 0 };
  if (!simCharName) return counts;
  var cd = getCharData(simCharName);
  if (!cd) return counts;
  var cs = getMTCharState(simCharName);
  for (var si = 0; si < cd.styles.length; si++) {
    var style = cd.styles[si];
    var styleSel = cs[style.type] || {};
    for (var ri = 0; ri < style.ranks.length; ri++) {
      var rank = style.ranks[ri];
      var sel = styleSel[String(rank.rank)] || [];
      for (var ni = 0; ni < sel.length; ni++) {
        var node = rank.nodes[sel[ni]];
        var text = (typeof node === 'object') ? node.t : node;
        if (!text) continue;
        if (/最大HP\s*\+\s*15,000/.test(text)) counts.hp++;
        if (isPureAtkNode(text, 30)) counts.atk30++;
        if (isPureAtkNode(text, 20) && !isExAtkWeaknessNode(text)) counts.atk20++;
        if (/基础能力类因子/.test(text)) counts.synBasic++;
        if (/攻击类因子/.test(text)) counts.synAtk++;
        // 防御类因子：区分 HP版本(+10,000)与 防御%版本(+6%)
        if (/防御力随/.test(text) && /\+6%/.test(text)) counts.synDef2++;
        else if (/防御类因子/.test(text)) counts.synDef++;
      }
    }
  }
  return counts;
}
// 联动：把专精树已激活的对应节点数量同步到因子配装界面的"人物专精"计数器(state.mastery)
// 仅在专精树确实有过选择时覆盖，避免清空手动配置
window.syncMasteryFromTree = function(simCharName) {
  if (typeof state === 'undefined' || !simCharName) return;
  if (!hasMasteryTreeSelections(simCharName)) return;
  var counts = getMasteryTreeMasteryCounts(simCharName);
  if (!state.mastery) state.mastery = { hp: 0, atk: 0, atk30: 0, synBasic: 0, synAtk: 0, synDef: 0, synDef2: 0 };
  state.mastery.hp = Math.min(counts.hp, 4);
  state.mastery.atk = Math.min(counts.atk20, 3);
  state.mastery.atk30 = Math.min(counts.atk30, 2);
  state.mastery.synBasic = Math.min(counts.synBasic, 2);
  state.mastery.synAtk = Math.min(counts.synAtk, 2);
  state.mastery.synDef = Math.min(counts.synDef, 1);
  state.mastery.synDef2 = Math.min(counts.synDef2, 1);
};

window.hasMasteryTreeSelections = function(simCharName) {
  ensureMTState();
  var mt = window.state.masteryTree[simCharName];
  if (!mt) return false;
  for (var st in mt) {
    if (mt[st]) {
      for (var r in mt[st]) {
        if (mt[st][r] && mt[st][r].length > 0) return true;
      }
    }
  }
  return false;
};

// 反向联动：根据人物专精面板(state.mastery)的计数，自动勾选/取消专精树中对应节点
window.syncMasteryTreeFromPanel = function(simCharName) {
  if (typeof window.state === 'undefined' || !window.state.mastery || !simCharName) return;
  ensureMTState();
  if (!window.state.masteryTree[simCharName]) window.state.masteryTree[simCharName] = {};

  var mst = window.state.mastery;
  var targets = {
    hp: Math.min(mst.hp || 0, 4),
    atk20: Math.min(mst.atk || 0, 3),
    atk30: Math.min(mst.atk30 || 0, 2),
    synBasic: Math.min(mst.synBasic || 0, 2),
    synAtk: Math.min(mst.synAtk || 0, 2),
    synDef: Math.min(mst.synDef || 0, 1),
    synDef2: Math.min(mst.synDef2 || 0, 1)
  };

  var cd = getCharData(simCharName);
  if (!cd) return;

  function classifyNodeText(text) {
    if (!text) return null;
    if (/最大HP\s*\+\s*15,000/.test(text)) return 'hp';
    if (isPureAtkNode(text, 30)) return 'atk30';
    if (isPureAtkNode(text, 20) && !isExAtkWeaknessNode(text)) return 'atk20';
    if (/基础能力类因子/.test(text)) return 'synBasic';
    if (/攻击类因子/.test(text)) return 'synAtk';
    // 防御类因子：区分 HP版本(+10,000)与 防御%版本(+6%)
    if (/防御力随/.test(text) && /\+6%/.test(text)) return 'synDef2';
    if (/防御类因子/.test(text)) return 'synDef';
    return null;
  }

  // Step 1: 从当前专精树中移除所有面板控制类型的节点（保留非面板节点，如暴击/上限等）
  var cs = getMTCharState(simCharName);
  for (var st in cs) {
    var styleObj = cs[st];
    for (var rk in styleObj) {
      var sel = styleObj[rk];
      var rankData = null;
      for (var si = 0; si < cd.styles.length; si++) {
        if (cd.styles[si].type === st) {
          for (var ri = 0; ri < cd.styles[si].ranks.length; ri++) {
            if (String(cd.styles[si].ranks[ri].rank) === rk) {
              rankData = cd.styles[si].ranks[ri];
              break;
            }
          }
          break;
        }
      }
      if (!rankData) continue;
      cs[st][rk] = sel.filter(function(idx) {
        var node = rankData.nodes[idx];
        var text = (typeof node === 'object') ? node.t : node;
        return !classifyNodeText(text);
      });
    }
  }

  // Step 2: 按类型收集候选节点，按 风格→阶级→节点 顺序排列
  var candidates = {
    hp: [], atk20: [], atk30: [], synBasic: [], synAtk: [], synDef: [], synDef2: []
  };
  for (var si = 0; si < cd.styles.length; si++) {
    var style = cd.styles[si];
    for (var ri = 0; ri < style.ranks.length; ri++) {
      var rank = style.ranks[ri];
      var rankKey = String(rank.rank);
      for (var ni = 0; ni < rank.nodes.length; ni++) {
        var node = rank.nodes[ni];
        var text = (typeof node === 'object') ? node.t : node;
        var type = classifyNodeText(text);
        if (type) {
          candidates[type].push({
            styleType: style.type,
            rankKey: rankKey,
            idx: ni,
            cap: rank.cap || 0,
            max: RANK_MAX[rankKey] || 0
          });
        }
      }
    }
  }

  // Step 3: 按优先级依次为各类型勾选目标数量，遵循阶级上限（cap / max）
  var typePriority = ['hp', 'atk20', 'atk30', 'synBasic', 'synAtk', 'synDef', 'synDef2'];
  for (var pi = 0; pi < typePriority.length; pi++) {
    var type = typePriority[pi];
    var target = targets[type];
    var list = candidates[type];
    for (var i = 0; i < list.length && target > 0; i++) {
      var cand = list[i];
      var sel = getMTRankSel(simCharName, cand.styleType, cand.rankKey);
      if (cand.cap > 0 && sel.length >= cand.cap) continue;
      if (globalRankSelected(simCharName, cand.rankKey) >= cand.max) continue;
      if (sel.indexOf(cand.idx) < 0) {
        sel.push(cand.idx);
        target--;
      }
    }
  }
};

window.clearMasteryTreeFor = function(simCharName) {
  ensureMTState();
  if (simCharName) window.state.masteryTree[simCharName] = {};
};

})();
