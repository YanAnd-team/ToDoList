---
---

# Assignment 1 — 任务清单

[编程任务](#coding) · [美术任务](#art) · [依赖关系](#deps)

# 编程任务 {#coding}

对照作业要求（*GameDevelopment_Assignment1.pdf*）和老师的引擎模板整理的**编程任务**，按由易到难排列。美术任务在下方的[美术任务](#art)板块，关卡设计和 README 文案不在此页。

玩家能力：**二段跳**、**冲刺**、**贴墙下滑 + 贴墙跳**（必须跳到对面的墙才能再次贴墙/贴墙跳，落地后重置）。

**状态图例：** ⬜ 未开始 · 🟨 进行中（已分配负责人） · ✅ 完成（Issue 已关闭）

**怎么用：** 点某一行的 **⬜ 认领**，会打开一个已经填好标题的 GitHub Issue，提交即可；在 Issue 里分配负责人，做完后关闭 Issue，刷新本页状态就会更新。

---

## 开工前必读：水平速度的控制权

`Player::GetPhysicsValues()` 在**每帧开头把水平速度清零**（`velocity = { 0, velocity.y }`），再由 `Move()` 重新设置。后果：

- 贴墙跳把角色推离墙的水平速度，下一帧就被清掉；
- 冲刺的水平速度会被 `Move()` 覆盖。

所以做三个能力之前，要先想清楚「什么时候由玩家输入控制水平速度，什么时候由能力接管」—— 见 **#22**。

---

## 第一档：改几行就能完成

| # | 状态 | 负责人 | 任务 | 可能用到的资源 | 提示 |
|---|---|---|---|---|---|
| 1 | ⬜ | | F9 显示碰撞体（替换现在的 F1），默认关闭 | `Physics::PostUpdate()`、成员 `debug`、`SDL_SCANCODE_F9` | 构造函数里 `debug = true` 要改 |
| 2 | ⬜ | | 删除 Scene 里用方向键移动镜头的代码 | `Scene::Update()` | 模板遗留 |
| 3 | ⬜ | | 音乐、音效、地图路径移到 `config.xml` | `Scene::Start()`、`Player::Start()`、pugixml `.child().attribute().as_string()` | 参考 `Player::Start()` 读 texture 的写法 |
| 4 | ⬜ | | 删除测试金币，或改为从地图读取 | `Scene::Awake()` 里的 `Vector2D(200, 672)` | 坐标写死在代码里属于 “poorly implemented” |
| 5 | ⬜ | | 新能力的参数全部放进 `config.xml` | `Player::Start()` 的读取方式 | `maxJumps`、`dashSpeed`、`dashDuration`、`dashCooldown`、`wallSlideSpeed`、`wallJumpForceX/Y`… 老师会改 config 测试 |
| 6 | ⬜ | | 标题栏：FPS / 平均 FPS / 上一帧 ms / Vsync 开关 | `Engine::FinishUpdate()`、`window->SetTitle()` | `Render::vsync` 是 private，要加 getter |

## 第二档：需要自己设计一点

| # | 状态 | 负责人 | 任务 | 可能用到的资源 | 提示 |
|---|---|---|---|---|---|
| 7 | ⬜ | | 角色朝向翻转 | `Render::DrawTexture()`、`SDL_RenderTextureRotated()` 的 flip 参数（`Render.cpp:172` 写死为 `SDL_FLIP_NONE`）、`SDL_FLIP_HORIZONTAL` | 加一个带默认值的参数，其他调用处不用改；贴墙时要朝向墙的反方向 |
| 8 | ⬜ | | F11 切换 30 FPS 上限 | `Engine::maxFrameDuration`、`Engine::Awake()` 里读 `targetFrameRate` | 记住原来的值才能切回去 |
| 9 | ⬜ | | H 显示帮助菜单 | `SDL_RenderDebugText()`（SDL3 自带）、`DrawRectangle(..., useCamera=false)` | 模板没有文字渲染；备选：画一张做好的 PNG |
| 10 | ⬜ | | 碰撞体类型 DAMAGE、PIT | `Physics.h` 的 `enum class ColliderType`、`Player::OnCollision()` | 先只加类型 |
| 11 | ⬜ | | 地图按图层属性生成不同类型的碰撞体 | `Map::Load()`（`Map.cpp:170` 起）、`Properties::GetProperty()`、`AsString()` | 例：图层属性 `Collision = platform / damage / pit` |
| 12 | ⬜ | | 动画：idle、move、jump、fall、die、dash、wallslide | Tiled 编辑 `.tsx`、`LoadFromTSX()` 的 aliases、`Animation::SetLoop(false)`、`HasFinishedOnce()` | 换素材后 frame 尺寸会变，碰撞体尺寸跟着 config 走 |
| 13 | ⬜ | | 玩家碰撞体摩擦力设为 0 | `Physics::CreateCapsule()` 的 shape 定义、Box2D 的 `material.friction` | 否则顶着墙会被摩擦力卡住，干扰贴墙下滑 |

## 第三档：核心玩法逻辑

| # | 状态 | 负责人 | 任务 | 可能用到的资源 | 提示 |
|---|---|---|---|---|---|
| 14 | ⬜ | | 解析对象层，读取出生点 | `Map::Load()`、pugixml `children("objectgroup")`、`<object>` 的 `x` `y` `name` | 模板只解析 `<layer>`；图块对象的 y 在左下角 |
| 15 | ⬜ | | 玩家状态机：Idle / Run / Jump / Fall / Dash / WallSlide / Dead | 自定义 `enum class PlayerState` | 每个状态决定：接受哪些输入、播哪个动画、谁控制速度 |
| 16 | ⬜ | | 地面传感器（脚底） | `physics->AddSensorShape()`（已写好） | 难点：`BeginContact()` 只传 `PhysBody*`，分不出是身体还是传感器碰到的 |
| 17 | ⬜ | | 左右墙面传感器 | 同 #16 | 用**计数**（进入 +1、离开 −1）而不是 bool，同时碰到多个图块时才不会出错 |
| 18 | ⬜ | | 合并地图碰撞体 | `Map::Load()` 的碰撞体循环 | 现在每个图块一个矩形，贴墙下滑会在接缝处卡住（ghost collision）；相邻图块合并成长矩形 |
| 19 | ⬜ | | `Die()` 与 `Respawn()` | `b2Body_SetTransform()`（在 Physics 里封装）、`SetLinearVelocity(0,0)` | 碰撞回调里只设标志，在 `Update()` 里传送；死亡动画期间锁输入；复活时重置跳跃次数、冲刺冷却、上次贴的墙、状态、镜头 |
| 20 | ⬜ | | 镜头钳制在地图范围内 + 竖直跟随 | `Player::UpdateCamera()`、`Scene::SetCameraX/Y()`、`map->GetMapSizeInPixels()`、`std::clamp` | `camera.x` 是负数，注意符号 |
| 21 | ⬜ | | F10 上帝模式 | `b2Body_SetGravityScale()`、W/S 上下移动、`OnCollision` 跳过死亡 | 进入时中断冲刺和贴墙，退出时恢复重力 |
| 22 | ⬜ | | 重构水平速度的控制权 | `GetPhysicsValues()`、`Move()`、`ApplyPhysics()` | 见页首说明 |
| 23 | ⬜ | | 帧率无关测试（16 / 32 / 64） | `config.xml` 的 `targetFrameRate`、`Physics.cpp` 的 `FIXED_TIMESTEP`、`MAX_STEPS` | 冲刺时长、冷却等所有计时器都用 dt 累加；最后再测一遍 |

## 第四档：三个能力与架构

| # | 状态 | 负责人 | 任务 | 可能用到的资源 | 提示 |
|---|---|---|---|---|---|
| 24 | ⬜ | | 二段跳 | `jumpCount`、`maxJumps`、先 `SetYVelocity(0)` 再 `ApplyLinearImpulseToCenter()` | 依赖 #16；落地重置；决定贴墙时是否也重置 |
| 25 | ⬜ | | 冲刺 | `dashTimer`、`dashCooldown`、`SetGravityScale(0)`、`SetLinearVelocity` | 依赖 #15、#22；定规则：空中几次、何时恢复、能否向上冲 |
| 26 | ⬜ | | 贴墙下滑 | 墙面传感器（#17）、`GetYVelocity()`、`SetYVelocity()` | 条件：空中 + 碰墙 + 下落中；把下落速度限制在 `wallSlideSpeed`。**+y 向下** |
| 27 | ⬜ | | 贴墙跳 | 冲量（向上 + 离墙方向）、短暂的输入锁定 | 不锁输入的话，按着朝墙方向键会马上被拉回墙上 |
| 28 | ⬜ | | 左右墙交替规则 | `lastWallSide = LEFT / RIGHT / NONE` | 当前墙方向 ≠ `lastWallSide` 才能贴墙跳；落地重置为 NONE；决定同一面墙是「贴不上」还是「能贴不能跳」 |
| 29 | ⬜ | | F9 显示「逻辑」 | `Physics::PostUpdate()`、`SDL_RenderDebugText()` | 状态名、传感器计数、跳跃次数、冲刺冷却、出生点 |
| 30 | ⬜ | | 关卡卸载 / 加载（为 Assignment 2 准备） | 新写 `Map::Unload()`，记录 `PhysBody*` 并 `physics->DeletePhysBody()`；`entityManager->DestroyEntity()`；`Scene::LoadLevel(n)` | 现在 `Map::CleanUp()` 和 `Player::CleanUp()` 都不销毁碰撞体 |
| 31 | ⬜ | | Release 打包 | `.vcxproj` 的 `TargetName = Game`、PostBuild 的 xcopy、Release 版 DLL | Release 用 `box2d.dll` 而非 `box2dd.dll`；在没装 VS 的电脑上测试 |

---

# 美术任务 {#art}

点 **＋** 在 GitHub 上新建一个美术任务（会自动带上 `art` 标签），刷新本页就能看到。还没人负责的任务会显示 **⬜ 认领**，点它打开 Issue，在右侧 Assignees 点 *assign yourself* 即可；做完后把 Issue **关闭**，状态就会变成 ✅。

<div id="art-tasks" data-repo="{{ site.github.repository_nwo }}">
  <p class="art-toolbar">
    <a id="art-add" class="art-add" href="#" title="添加美术任务">＋ 添加美术任务</a>
    <a id="art-all" href="#">在 GitHub 上查看全部</a>
  </p>
  <div id="art-list"><p>正在加载…</p></div>
</div>

<style>
  .art-toolbar { display: flex; gap: 1rem; align-items: center; flex-wrap: wrap; }
  .art-add {
    display: inline-block; padding: 0.4rem 1rem; border-radius: 0.3rem;
    background: #159957; color: #fff !important; font-weight: bold; text-decoration: none !important;
  }
  .art-add:hover { background: #127a46; }
  .code-claim {
    display: inline-block; padding: 0 0.5rem; border: 1px solid #159957; border-radius: 0.3rem;
    white-space: nowrap; text-decoration: none !important;
  }
  .code-claim:hover { background: #eef5f1; }
  .art-done td { color: #999; }
  .art-done td a { text-decoration: line-through; }
  .art-tag {
    display: inline-block; margin: 0 0.2rem 0.2rem 0; padding: 0 0.4rem;
    border-radius: 0.8rem; font-size: 0.8em; background: #eef5f1; color: #155d3b;
  }
</style>

<script>
(function () {
  var root = document.getElementById('art-tasks');
  var list = document.getElementById('art-list');
  var repo = root.getAttribute('data-repo');
  if (!repo || repo.indexOf('{') !== -1) {
    list.innerHTML = '<p>找不到仓库名：这个板块只在 GitHub Pages 上运行时有效。</p>';
    return;
  }
  var base = 'https://github.com/' + repo;
  document.getElementById('art-add').href = base + '/issues/new?template=art-task.yml';
  document.getElementById('art-all').href = base + '/issues?q=label%3Aart';
  function esc(s) {
    return String(s).replace(/[&<>"']/g, function (c) {
      return { '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;' }[c];
    });
  }
  // 从 Issue 表单生成的正文里取出「### 类型」下面那一行
  function field(body, label) {
    var m = new RegExp('###\\s*' + label + '\\s*\\n+([^\\n]+)').exec(body || '');
    return m && m[1].trim() !== '_No response_' ? m[1].trim() : '';
  }
  // 一次取回全部 Issue（每页 100 个，最多 5 页），编程和美术两个板块共用
  function fetchAll(page, acc) {
    return fetch('https://api.github.com/repos/' + repo + '/issues?state=all&per_page=100&page=' + page)
      .then(function (r) {
        if (!r.ok) throw new Error(r.status);
        return r.json();
      })
      .then(function (batch) {
        acc = acc.concat(batch);
        return (batch.length === 100 && page < 5) ? fetchAll(page + 1, acc) : acc;
      });
  }
  function hasLabel(issue, name) {
    return issue.labels.some(function (l) { return l.name === name; });
  }
  function whoOf(issue) {
    return issue.assignees.map(function (a) { return esc(a.login); }).join(', ');
  }
  // 编程表格：按标题「[编程 #N]」把 Issue 对应到第 N 行，填入状态和负责人
  function fillCodeTables(issues) {
    var byTask = {};
    issues.forEach(function (i) {
      var m = /^\[编程\s*#(\d+)\]/.exec(i.title);
      if (!m) return;
      var prev = byTask[m[1]];
      // 同一项有多个 Issue 时，优先用未关闭的，其次用最新的
      if (!prev || (prev.state === 'closed' && i.state === 'open') ||
          (prev.state === i.state && i.number > prev.number)) {
        byTask[m[1]] = i;
      }
    });
    var page = location.href.split('#')[0];
    document.querySelectorAll('table').forEach(function (table) {
      if (root.contains(table)) return;
      table.querySelectorAll('tbody tr').forEach(function (tr) {
        var td = tr.children;
        if (td.length < 4 || !/^\d+$/.test(td[0].textContent.trim())) return;
        var n = td[0].textContent.trim();
        var issue = byTask[n];
        if (!issue) {
          var title = '[编程 #' + n + '] ' + td[3].textContent.trim();
          var body = '编程任务清单第 ' + n + ' 项：' + page + '#coding';
          var url = base + '/issues/new?labels=code&title=' + encodeURIComponent(title) +
            '&body=' + encodeURIComponent(body);
          td[1].innerHTML = '<a class="code-claim" href="' + esc(url) + '">⬜ 认领</a>';
          td[2].innerHTML = '';
          return;
        }
        var done = issue.state === 'closed';
        var who = whoOf(issue);
        td[1].innerHTML = (done ? '✅' : (who ? '🟨' : '⬜')) +
          ' <a href="' + esc(issue.html_url) + '">#' + issue.number + '</a>';
        td[2].innerHTML = who;
        if (done) tr.classList.add('art-done');
      });
    });
  }
  fetchAll(1, [])
    .then(function (issues) {
      issues = issues.filter(function (i) { return !i.pull_request; });
      fillCodeTables(issues);
      issues = issues.filter(function (i) { return hasLabel(i, 'art'); });
      if (issues.length === 0) {
        list.innerHTML = '<p>还没有美术任务，点上面的 ＋ 添加第一个。</p>';
        return;
      }
      // 未完成的排前面，同状态按编号排
      issues.sort(function (a, b) {
        if (a.state !== b.state) return a.state === 'open' ? -1 : 1;
        return a.number - b.number;
      });
      var rows = issues.map(function (i) {
        var done = i.state === 'closed';
        var who = i.assignees.map(function (a) { return esc(a.login); }).join(', ');
        var tags = i.labels
          .filter(function (l) { return l.name !== 'art'; })
          .map(function (l) { return '<span class="art-tag">' + esc(l.name) + '</span>'; })
          .join('');
        var title = i.title.replace(/^\[美术\]\s*/, '');
        return '<tr' + (done ? ' class="art-done"' : '') + '>' +
          '<td>' + i.number + '</td>' +
          '<td>' + (done ? '✅' : (who ? '🟨' :
            '<a class="code-claim" href="' + esc(i.html_url) + '" title="打开 Issue，在右侧 Assignees 点 assign yourself">⬜ 认领</a>')) + '</td>' +
          '<td>' + (who || '') + '</td>' +
          '<td>' + esc(field(i.body, '类型')) + '</td>' +
          '<td><a href="' + esc(i.html_url) + '">' + esc(title) + '</a> ' + tags + '</td>' +
          '<td>' + esc(field(i.body, '规格')) + '</td>' +
          '<td>' + esc(field(i.body, '对应的编程任务')) + '</td>' +
          '</tr>';
      }).join('');
      list.innerHTML =
        '<table><thead><tr><th>#</th><th>状态</th><th>负责人</th><th>类型</th>' +
        '<th>任务</th><th>规格</th><th>对应编程任务</th></tr></thead>' +
        '<tbody>' + rows + '</tbody></table>';
    })
    .catch(function (e) {
      list.innerHTML = '<p>加载失败（' + esc(e.message) + '）。仓库需要是公开的；' +
        '也可能是 GitHub API 的访问次数超限（每小时 60 次），稍后再刷新。' +
        '<a href="' + base + '/issues?q=label%3Aart">直接在 GitHub 上查看</a>。</p>';
    });
})();
</script>

---

## 依赖关系 {#deps}

```
#15 状态机 ─┬─> #22 速度控制权 ─┬─> #25 冲刺
            │                   └─> #27 贴墙跳 ──> #28 交替规则
#16 地面传感器 ──> #24 二段跳
#17 墙面传感器 ──> #26 贴墙下滑 ──> #27
#18 合并碰撞体 ──> #26
#10 → #11 → #19 死亡复活 <── #14 出生点
```

## 建议顺序

1. 第一、二档 + #14、#19 —— 做完**必做功能**，保底 7–8 分。
2. 额外功能按 **二段跳 → 冲刺 → 贴墙** 的顺序。
3. 作业原文：「做了一半的额外功能不算分」—— 每个能力都要做完整、手感好，再写进发布说明。

## 已由模板完成

- Player 继承 `Entity`，由 `EntityManager` 管理
- `Move()` / `Jump()` 分函数、Box2D 固定步长
- 玩家参数（速度、跳跃力、贴图、帧尺寸）从 `config.xml` 读取
- 动画从 `.tsx` 读取
- 加载地图时从图层动态生成碰撞体（只生成一次）
- 帧率上限（关闭 vsync 时 60 FPS）
