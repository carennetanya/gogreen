<template>
  <div class="intro-root">
    <audio ref="audioEl"></audio>
    <audio ref="musicEl" loop></audio>
    <audio ref="sfxEl" loop></audio>
    <audio ref="radioEl" loop></audio>
    <audio ref="syncEl"></audio>
    <audio ref="natureEl" loop></audio>
    <audio ref="endingEl" loop></audio>
    <audio ref="fireEl" loop></audio>
    <audio ref="openingEl" loop></audio>
    <audio ref="mainEl" loop></audio>

    <transition name="fade" mode="out-in">
      <!-- LOADING -->
      <div class="stage" v-if="phase === 'loading'" key="loading">
        <div class="loading-pct">{{ pct }}%</div>
      </div>

      <!-- GO GREEN / CLICK TO START -->
      <div class="stage gg-bg" v-else-if="phase === 'goGreen'" key="goGreen" @click="onStartClick">
        <div class="gg-glow"></div>
        <div class="gg-clouds"><i></i><i></i><i></i><i></i><i></i></div>
        <div class="gg-title">Go Green</div>
        <div class="gg-sub">{{ t('goSub') }}</div>
      </div>

      <!-- HEADPHONE PROMPT -->
      <div class="stage" v-else-if="phase === 'headphone'" key="headphone">
        <svg class="hp-icon" viewBox="0 0 24 24">
          <path d="M3 15v-3a9 9 0 0 1 18 0v3" />
          <rect x="1" y="15" width="6" height="7" rx="2" />
          <rect x="17" y="15" width="6" height="7" rx="2" />
        </svg>
        <div class="hp-text">
          <div>{{ t('hpLine1') }}</div>
          <div>{{ t('hpLine2') }}</div>
        </div>
      </div>

      <!-- PHASE 0: BLACKOUT (hitam total + garis heartbeat merah) -->
      <div class="stage blackout" v-else-if="phase === 'blackout'" key="blackout">
        <div class="blackout-pulse"></div>
        <div class="ecg">
          <svg viewBox="0 0 200 40">
            <path
              class="ecg-line"
              pathLength="1"
              d="M0 20 H58 L64 20 L70 8 L77 33 L84 14 L89 20 H196"
            />
            <circle class="ecg-dot" cx="197" cy="20" r="2.2" />
          </svg>
        </div>
      </div>

      <!-- SCENE: waking (Phase 1-3) -> narrative -> transmission -->
      <div
        class="stage scene"
        :class="sceneState"
        v-else-if="phase === 'waking' || phase === 'narrative' || phase === 'transmission'"
        key="scene"
      >
        <div class="scene-eye">
          <div class="scene-world">
            <img class="scene-bg" src="/picture/city1.avif" alt="" draggable="false" />
            <div class="waking-glow"></div>

            <!-- ===== ASAP KEBAKARAN + POLUSI (DI BELAKANG KARAKTER) ===== -->
            <div class="scene-smoke" aria-hidden="true">
              <!-- kabut polusi tebal menggantung -->
              <div class="smog smog-top"></div>
              <div class="smog smog-mid"></div>
              <div class="smog smog-low"></div>

              <!-- cahaya api dari dasar reruntuhan (berkedip) -->
              <div class="fire-glow fg-1"></div>
              <div class="fire-glow fg-2"></div>
              <div class="fire-glow fg-3"></div>
              <div class="fire-glow fg-4"></div>

              <!-- kolom asap naik dari titik api -->
              <span
                v-for="p in scenePlumes"
                :key="'sp' + p.id"
                class="plume"
                :class="{ dark: p.dark }"
                :style="p.style"
              ></span>

              <!-- percikan bara -->
              <span
                v-for="e in sceneEmbers"
                :key="'se' + e.id"
                class="ember"
                :style="e.style"
              ></span>

              <!-- asap tipis melintas -->
              <span
                v-for="w in sceneWisps"
                :key="'sw' + w.id"
                class="wisp"
                :style="w.style"
              ></span>
            </div>

            <!-- karakter: people1 (sampai mata merem total) -> people2 (setelah melek) -->
            <div class="person-wrap">
              <div class="person-sway">
                <img
                  class="person-img"
                  :key="personSrc"
                  :src="personSrc"
                  alt=""
                  draggable="false"
                />
              </div>
            </div>
          </div>
        </div>

        <div class="sky-dark"></div>
        <div class="scene-vignette"></div>

        <!-- glitch / gangguan sinyal -->
        <div class="glitch" v-if="phase === 'transmission'">
          <div class="scan"></div>
          <i></i><i></i><i></i><i></i><i></i>
        </div>
        <div class="glitch-burst" v-if="glitchOut" aria-hidden="true">
          <i></i><i></i><i></i><i></i><i></i><i></i><i></i>
        </div>

        <!-- Phase 1-3: kotak teks di kanan atas -->
        <div class="waking-box" :key="wakeStage" v-if="phase === 'waking'">
          <div class="tx-header" v-if="wakeStage === 1">
            {{ t('txHeader') }}
          </div>
          <div class="waking-text">{{ lines[wakeStage - 1] }}</div>
        </div>
        <div
          class="narrative-text"
          :class="{ show: showLine }"
          v-else-if="phase === 'narrative'"
        >{{ currentLine }}</div>

        <!-- TRANSMISSION -->
        <template v-if="phase === 'transmission'">
          <div class="tx-box">
            <div class="tx-header">{{ t('txHeader') }}</div>
            <div class="waking-text tx-fade" :key="txStage >= 3 ? 'b' : 'a'">
              {{ txStage >= 3 ? TX_TOP_2 : TX_TOP_1 }}
            </div>
          </div>

          <div class="narrative-text" :class="{ show: txStage === 2 }">{{ TX_REPLY }}</div>

          <div class="choices" v-if="txStage >= 4 && !glitchOut">
            <button class="choice" @click="onChoose('whatever')">{{ t('whatever') }}</button>
            <button class="choice choice-warn" @click="onChoose('inspect')">{{ t('inspectBtn') }}</button>
          </div>
        </template>

      </div>

      <!-- INSPECT: 3 warning nodes -->
      <div
        class="stage inspect"
        :class="{ shifting: shifted }"
        v-else-if="phase === 'inspect'"
        key="inspect"
      >
        <img class="inspect-bg" :src="INSPECT_BG" alt="" draggable="false" />

        <!-- ===== ASAP KEBAKARAN + POLUSI (di belakang node) ===== -->
        <div class="smoke-layer" aria-hidden="true">
          <!-- kabut polusi tebal yang menggantung -->
          <div class="smog smog-top"></div>
          <div class="smog smog-mid"></div>
          <div class="smog smog-low"></div>

          <!-- cahaya api dari dasar reruntuhan (berkedip) -->
          <div class="fire-glow fg-1"></div>
          <div class="fire-glow fg-2"></div>
          <div class="fire-glow fg-3"></div>
          <div class="fire-glow fg-4"></div>

          <!-- kolom asap naik dari titik api -->
          <span
            v-for="p in plumes"
            :key="'p' + p.id"
            class="plume"
            :class="{ dark: p.dark }"
            :style="p.style"
          ></span>

          <!-- percikan bara -->
          <span
            v-for="e in embers"
            :key="'e' + e.id"
            class="ember"
            :style="e.style"
          ></span>
        </div>

        <!-- asap tipis di depan (parallax), tetap di bawah node -->
        <div class="smoke-front" aria-hidden="true">
          <span
            v-for="w in wisps"
            :key="'w' + w.id"
            class="wisp"
            :style="w.style"
          ></span>
        </div>

        <div class="fire-flicker" aria-hidden="true"></div>
        <div class="inspect-vignette"></div>
        <div class="glitch"><div class="scan"></div></div>

        <!-- petunjuk kiri atas -->
        <div class="inspect-hint" :class="{ gone: pinsHidden }">
          <svg class="hint-hand" viewBox="0 0 24 24">
            <path d="M9 11V5a1.5 1.5 0 0 1 3 0v5m0 0V8.5a1.5 1.5 0 0 1 3 0V11m0 0V10a1.5 1.5 0 0 1 3 0v4.5c0 3-2 6-5.5 6H12c-2 0-3-1-4.2-2.6L5 14a1.5 1.5 0 0 1 2.3-1.9L9 14" />
          </svg>
          <div class="hint-text">{{ t('hintClick') }} {{ t('hintRest') }}</div>
          <div class="hint-count">{{ seen.length }}/3</div>
        </div>

        <!-- 3 node berkedip -->
        <button
          v-for="n in nodes"
          :key="n.id"
          class="node"
          :class="{ seen: seen.includes(n.id), gone: pinsHidden }"
          :style="{ left: n.x + '%', top: n.y + '%', '--stem': n.stem + 'vh' }"
          @click="openNode(n.id)"
        >
          <span class="node-ring"></span>
          <span class="node-icon">
            <svg viewBox="0 0 24 24"><path :d="n.icon" /></svg>
          </span>
          <span class="node-tag">{{ n.label }}</span>
          <span class="node-stem"></span>
        </button>

        <!-- kartu data investigasi -->
        <transition name="fade">
          <div class="card-overlay" v-if="activeNode" @click.self="closeNode">
            <div class="data-card">
              <div class="tx-header">{{ t('scan', { label: activeNode.label }) }}</div>
              <div class="card-title">{{ activeNode.title }}</div>
              <div class="card-cols">
                <div class="card-col">
                  <div class="card-year">2026</div>
                  <div class="card-val">{{ activeNode.now.value }}</div>
                  <div class="card-note">{{ activeNode.now.note }}</div>
                </div>
                <div class="card-arrow">→</div>
                <div class="card-col danger">
                  <div class="card-year">2076</div>
                  <div class="card-val">{{ activeNode.future.value }}</div>
                  <div class="card-note">{{ activeNode.future.note }}</div>
                </div>
              </div>
              <div class="card-msg">"{{ activeNode.msg }}"</div>
              <button class="choice card-close" @click="closeNode">{{ t('close') }}</button>
            </div>
          </div>
        </transition>

        <!-- flash glitch merah-oranye saat 3 pin selesai (0.3 detik) -->
        <div class="sync-flash" v-if="flash" aria-hidden="true">
          <i></i><i></i><i></i>
        </div>

        <!-- DIALOG POV (bawah tengah, putih bersih) -->
        <div class="pov-text" :class="{ show: povShow }">{{ POV_LINE }}</div>

        <!-- PESAN PENUTUP 2076 (kanan atas, neon oranye berkedip) -->
        <div class="tx-box tx-neon" :class="{ show: txShow }">
          <div class="tx-header">{{ t('txHeader') }}</div>
          <div class="waking-text">{{ TX_FINAL }}</div>
        </div>

        <!-- SCROLL TEASER (hijau neon berdenyut) -->
        <button
          class="scroll-teaser"
          :class="{ show: unlocked && !shifted }"
          :tabindex="unlocked && !shifted ? 0 : -1"
          @click="startShift"
        >{{ t('unlocked') }}</button>

        <!-- ===== DUNIA HIJAU (cross-fade saat mulai scroll) ===== -->
        <div class="green-world" ref="greenEl" :class="{ on: shifted && !nicknamePrompt, waking: waking }" :style="worldStyle" aria-hidden="true">
          <!-- filter riak air (dipakai .gw-water img) -->
          <svg class="gw-defs" width="0" height="0" focusable="false">
            <defs>
              <filter id="gwWaterFx" x="0" y="0" width="100%" height="100%" color-interpolation-filters="sRGB">
                <feTurbulence type="fractalNoise" baseFrequency="0.004 0.035" numOctaves="1" seed="4" result="noise">
                  <animate
                    attributeName="baseFrequency"
                    dur="16s"
                    values="0.004 0.035;0.007 0.05;0.004 0.035"
                    repeatCount="indefinite"
                  />
                </feTurbulence>
                <feDisplacementMap in="SourceGraphic" in2="noise" scale="16" xChannelSelector="R" yChannelSelector="G" />
              </filter>
            </defs>
          </svg>

          <!-- pembungkus POV: rebahan -> berdiri + blur -> fokus -->
          <div class="gw-look">
          <div class="gw-tilt">
            <!-- kamera: zoom super pelan, panggung selalu se-rasio dengan gambar -->
            <div class="gw-cam">
              <div class="gw-stage">
                <img class="gw-bg" :src="GREEN_BG" alt="" draggable="false" />

                <!-- AIR -->
                <div class="gw-water">
                  <img :src="GREEN_BG" alt="" draggable="false" />
                  <div class="gw-sheen"></div>
                  <span
                    v-for="g in glints"
                    :key="'g' + g.id"
                    class="glint"
                    :style="g.style"
                  ></span>
                </div>

                <!-- MATAHARI -->
                <div class="gw-sunglow"></div>
                <div class="gw-flare"></div>
                <div class="gw-rays">
                  <span
                    v-for="r in rays"
                    :key="'r' + r.id"
                    class="ray"
                    :style="r.style"
                  ></span>
                </div>

                <!-- partikel hidup -->
                <span
                  v-for="l in fallLeaves"
                  :key="'fl' + l.id"
                  class="fleaf"
                  :style="l.style"
                ></span>
                <span
                  v-for="m in motes"
                  :key="'m' + m.id"
                  class="mote"
                  :style="m.style"
                ></span>
              </div>
            </div>

            <!-- burung terbang (putih) -->
            <div class="gw-birds">
              <span
                v-for="b in birds"
                :key="'b' + b.id"
                class="bird"
                :style="b.style"
              >
                <svg viewBox="0 0 40 20" aria-hidden="true">
                  <path class="wl" d="M20 14 Q10 1 0 6 Q10 8 20 14Z" />
                  <path class="wr" d="M20 14 Q30 1 40 6 Q30 8 20 14Z" />
                </svg>
              </span>
            </div>

            <!-- DAUN FOREGROUND -->
            <div class="gw-parallax">
            <div class="gw-cam">
              <div class="gw-stage">
                <div
                  v-for="c in fgClusters"
                  :key="'fc' + c.id"
                  class="fg-cluster"
                  :style="c.style"
                >
                  <svg
                    v-if="c.branch"
                    class="fg-branch"
                    :class="'br-' + c.branch"
                    viewBox="0 0 100 40"
                    preserveAspectRatio="none"
                    aria-hidden="true"
                  >
                    <path :d="c.branch === 'tl' ? 'M0 10 C22 14, 44 6, 78 0' : 'M100 12 C78 16, 58 26, 40 40'" />
                  </svg>

                  <svg
                    v-for="l in c.leaves"
                    :key="'fl' + l.id"
                    class="fg-leaf"
                    viewBox="0 0 100 60"
                    :style="l.style"
                    aria-hidden="true"
                  >
                    <path class="fl-body" d="M2 30 Q28 -4 98 30 Q28 64 2 30Z" />
                    <path class="fl-hi" d="M2 30 Q28 -4 98 30 Q50 22 2 30Z" />
                    <path class="fl-vein" d="M28 29 L44 14 M28 29 L44 44 M52 28.5 L68 17 M52 28.5 L68 41" />
                    <path class="fl-rib" d="M2 30 Q50 27 92 30" />
                  </svg>

                  <svg
                    v-for="f in c.flowers"
                    :key="'ff' + f.id"
                    class="fg-flower"
                    viewBox="-20 -20 40 40"
                    :style="f.style"
                    aria-hidden="true"
                  >
                    <ellipse
                      v-for="k in 5"
                      :key="k"
                      cx="0" cy="-9" rx="5.5" ry="9"
                      :transform="'rotate(' + (k - 1) * 72 + ')'"
                    />
                    <circle cx="0" cy="0" r="4.2" />
                  </svg>
                </div>
              </div>
            </div>
            </div>
          </div>
          </div>
        </div>

        <form
          v-if="nicknamePrompt"
          class="nickname-overlay"
          @submit.prevent="beginWakeSequence"
        >
          <div class="nickname-card">
            <div class="tx-header">{{ t('identity') }}</div>
            <label class="nickname-label" for="wake-nickname">{{ t('nickLabel') }}</label>
            <input
              id="wake-nickname"
              v-model.trim="nicknameInput"
              class="nickname-input"
              type="text"
              name="nickname"
              autocomplete="nickname"
              maxlength="24"
              :placeholder="t('nickPh')"
              required
            />
            <button class="choice nickname-submit" type="submit">{{ t('continue') }}</button>
          </div>
        </form>

        <!-- DIALOG BANGUN TIDUR (bawah tengah) -->
        <div
          class="pov-text wake-text"
          :class="{ show: wakeText, 'wake-text-first': wakeMessageIndex === 0 }"
        >{{ wakeMessage }}</div>

        <!-- ECO-PULSE INTRO (muncul setelah teks terakhir) -->
        <transition name="fade">
          <div class="eco-overlay" v-if="ecoIntro">
            <div class="eco-card">
              <div class="eco-title">
                <span class="eco-name">ECO-PULSE</span>
                <span class="eco-year">2026</span>
              </div>
              <div class="eco-tag">{{ t('ecoTag') }}</div>

              <button class="eco-start" @click="onStartGame">
                <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M6 4l14 8-14 8z" /></svg>
                {{ t('startGame') }}
              </button>

              <div class="eco-info">
                <svg class="eco-clock" viewBox="0 0 24 24" aria-hidden="true">
                  <circle cx="12" cy="13" r="8" />
                  <path d="M12 9v4l2.5 2M9.5 2h5" />
                </svg>
                <p>{{ t('ecoInfo', { s: GAME_SECONDS }) }}</p>
              </div>
            </div>
          </div>
        </transition>

        <!-- ECO-PULSE GAME -->
        <div class="game-root" v-if="gameActive" :class="{ paused }">
          <div class="game-smog" :style="{ opacity: smogOpacity }" aria-hidden="true">
            <div class="game-tint"></div>
            <div class="smog smog-top"></div>
            <div class="smog smog-mid"></div>
            <div class="smog smog-low"></div>
            <span v-for="p in gamePlumes" :key="'gp' + p.id" class="plume" :class="{ dark: p.dark }" :style="p.style"></span>
            <span v-for="w in gameWisps" :key="'gwp' + w.id" class="wisp" :style="w.style"></span>
          </div>

          <div class="g-hud">
            <div class="g-card g-timer" :class="{ danger: timeLeft <= 6 }">
              <svg class="g-clock" viewBox="0 0 24 24" aria-hidden="true">
                <circle cx="12" cy="13" r="8" /><path d="M12 9v4l2.5 2M9.5 2h5" />
              </svg>
              <div><small>{{ t('timeLeft') }}</small><b>{{ timeLeft }}s</b></div>
            </div>

            <div class="g-card g-bar">
              <div class="g-bar-label">
                <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M9.59 4.59A2 2 0 1 1 11 8H2m10.59 11.41A2 2 0 1 0 14 16H2m15.73-8.27A2.5 2.5 0 1 1 19.5 12H2" /></svg>
                {{ t('cleanAir', { n: cleanAir }) }}
              </div>
              <div class="g-segs"><i v-for="n in 10" :key="n" :class="{ on: cleanAir > (n - 1) * 10 }"></i></div>
            </div>

            <button class="g-card g-pause" :class="{ 'g-hide': gameState !== 'playing' }" @click="togglePause">{{ t('pause') }}</button>
          </div>

          <button
            v-for="b in bubbles"
            :key="b.id"
            class="g-bubble"
            :class="{ leaf: b.leaf, fading: b.life < 500 }"
            :style="b.style"
            :aria-label="b.leaf ? t('seedAria') : t('pollAria')"
            @pointerdown.prevent="hit(b)"
          >
            <span class="g-float">
              <svg v-if="!b.leaf" viewBox="0 0 100 100" aria-hidden="true">
                <g class="cl-out"><circle v-for="(c, i) in CLOUD" :key="'o' + i" :cx="c[0]" :cy="c[1]" :r="c[2]" /></g>
                <g class="cl-fill"><circle v-for="(c, i) in CLOUD" :key="'f' + i" :cx="c[0]" :cy="c[1]" :r="c[2]" /></g>
                <path class="cl-brow" d="M28 50 L44 57 M72 50 L56 57" />
                <circle class="cl-eye" cx="38" cy="62" r="5" /><circle class="cl-eye" cx="62" cy="62" r="5" />
                <circle class="cl-pupil" cx="39" cy="63" r="2.6" /><circle class="cl-pupil" cx="61" cy="63" r="2.6" />
                <path class="cl-brow" d="M40 77 Q50 69 60 77" />
              </svg>
              <span v-else class="g-orb">
                <svg viewBox="0 0 100 100" aria-hidden="true">
                  <path class="lf-body" d="M14 86C10 44 40 14 88 12c2 44-26 74-74 74z" />
                  <path class="lf-vein" d="M14 86L58 42" />
                </svg>
              </span>
            </span>
          </button>

          <div v-for="p in pops" :key="p.id" class="g-pop" :class="{ leaf: p.leaf }" :style="{ left: p.x + '%', top: p.y + '%' }">
            <span class="g-ring"></span>
            <i v-for="k in 8" :key="k" class="g-shard" :style="{ '--a': k * 45 + 'deg' }"></i>
            <b class="g-plus">+{{ p.gain }}%</b>
          </div>

          <div class="g-hint g-card" :class="{ show: gameHint }">
            <svg viewBox="0 0 24 24" aria-hidden="true">
              <path d="M9 11V5a1.5 1.5 0 0 1 3 0v5m0 0V8.5a1.5 1.5 0 0 1 3 0V11m0 0V10a1.5 1.5 0 0 1 3 0v4.5c0 3-2 6-5.5 6H12c-2 0-3-1-4.2-2.6L5 14a1.5 1.5 0 0 1 2.3-1.9L9 14" />
            </svg>
            <span>{{ t('gameHint') }}</span>
          </div>

          <div class="g-overlay" v-if="paused" @click="togglePause">
            <div class="g-result g-card">
              <div class="g-result-title">{{ t('paused') }}</div>
              <button class="g-btn" @click.stop="togglePause">{{ t('resume') }}</button>
            </div>
          </div>

          <div class="g-overlay" v-if="gameState === 'lost'">
            <div class="g-result g-card">
              <div class="g-result-title">{{ t('timesUp') }}</div>
              <p>{{ t('cleanLose', { n: cleanAir }) }}</p>
              <button class="g-btn" @click="startGame">{{ t('tryAgain') }}</button>
            </div>
          </div>

          <div class="g-win-flash" v-if="winFlash" aria-hidden="true"></div>
          <div class="g-center" v-if="gameState === 'won' && !callOpen">
            <div class="g-rays"></div>
            <div class="g-title">{{ t('mission') }}</div>
          </div>

          <transition name="fade">
            <div class="g-overlay call-overlay" :class="{ 'call-answered': callAnswered }" v-if="callOpen">
              <div class="call-card">
                <template v-if="!callAnswered">
                  <div class="tx-header">{{ t('callHeader') }}</div>
                  <div class="call-ring">
                    <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72 12.84 12.84 0 0 0 .7 2.81 2 2 0 0 1-.45 2.11L8.09 9.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45 12.84 12.84 0 0 0 2.81.7A2 2 0 0 1 22 16.92z" /></svg>
                  </div>
                  <div class="call-name">{{ t('callName') }}</div>
                  <p class="call-sub">{{ t('callSub') }}</p>
                  <button class="choice call-answer" @click="answerCall">{{ t('answer') }}</button>
                </template>
                <template v-else>
                  <div class="call-reveal">
                    <img class="call-person" src="/picture/people3.avif" :alt="t('callAlt')" />
                    <div class="call-dialogue">
                      <transition name="reality-text" mode="out-in" appear>
                        <p class="call-message" :key="callMessageIndex">{{ CALL_MESSAGES[callMessageIndex] }}</p>
                      </transition>
                    </div>
                  </div>
                  <div class="call-controls" :aria-label="t('callControls')">
                    <button
                      class="call-control"
                      :class="{ active: micMuted }"
                      type="button"
                      :aria-label="micMuted ? t('unmute') : t('mute')"
                      :title="micMuted ? t('unmute') : t('mute')"
                      @click="micMuted = !micMuted"
                    >
                      <svg viewBox="0 0 24 24" aria-hidden="true">
                        <path d="M12 2a3 3 0 0 0-3 3v6a3 3 0 0 0 6 0V5a3 3 0 0 0-3-3Z" />
                        <path d="M5 10v1a7 7 0 0 0 14 0v-1M12 18v4m-4 0h8" />
                        <path v-if="micMuted" class="control-slash" d="M4 4l16 16" />
                      </svg>
                    </button>
                    <button
                      class="call-control call-control-end"
                      type="button"
                      :aria-label="t('endCall')"
                      :title="t('endCall')"
                      @click="endCall"
                    >
                      <svg viewBox="0 0 24 24" aria-hidden="true">
                        <path d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72 12.84 12.84 0 0 0 .7 2.81 2 2 0 0 1-.45 2.11L8.09 9.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45 12.84 12.84 0 0 0 2.81.7A2 2 0 0 1 22 16.92z" />
                      </svg>
                    </button>
                    <button
                      class="call-control"
                      :class="{ active: cameraOff }"
                      type="button"
                      :aria-label="cameraOff ? t('videoOn') : t('videoOff')"
                      :title="cameraOff ? t('videoOn') : t('videoOff')"
                      @click="cameraOff = !cameraOff"
                    >
                      <svg viewBox="0 0 24 24" aria-hidden="true">
                        <rect x="3" y="6" width="12" height="12" rx="2" />
                        <path d="m15 10 6-3v10l-6-3z" />
                        <path v-if="cameraOff" class="control-slash" d="M4 4l16 16" />
                      </svg>
                    </button>
                  </div>
                </template>
              </div>
            </div>
          </transition>
        </div>
        <!-- ===== STAGE 1 COMPLETED BANNER ===== -->
        <transition name="fade">
          <div class="stage-banner-wrap" v-if="stageBanner">
            <div class="stage-banner">
              <div class="sb-check">
                <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M5 12.5l4.5 4.5L19 7.5" /></svg>
              </div>
              <div class="sb-line1">{{ t(bannerLine1) }}</div>
              <div class="sb-line2">{{ t(bannerLine2) }}</div>
              <svg class="sb-leaf sb-leaf-l" viewBox="0 0 100 100" aria-hidden="true"><path d="M14 86C10 44 40 14 88 12c2 44-26 74-74 74z" /><path d="M14 86L58 42" /></svg>
              <svg class="sb-leaf sb-leaf-r" viewBox="0 0 100 100" aria-hidden="true"><path d="M14 86C10 44 40 14 88 12c2 44-26 74-74 74z" /><path d="M14 86L58 42" /></svg>
              <i class="sb-spark" style="left:8%;top:-6%"></i>
              <i class="sb-spark" style="left:88%;top:-10%;animation-delay:-.5s"></i>
              <i class="sb-spark" style="left:96%;top:60%;animation-delay:-.9s"></i>
            </div>
          </div>
        </transition>

        <!-- ===== MINI GAME 2: AI TRASH SORTING SCANNER ===== -->
        <div class="trash-root" v-if="trashState !== 'idle'">
          <transition name="fade">
            <div class="eco-overlay" v-if="trashState === 'intro'">
              <div class="eco-card">
                <div class="eco-title">
                  <span class="eco-name">{{ t('trashName') }}</span>
                  <span class="eco-year">{{ t('trashYear') }}</span>
                </div>
                <div class="eco-tag">{{ t('trashTag') }}</div>
                <button class="eco-start" @click="startTrash">
                  <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M6 4l14 8-14 8z" /></svg>
                  {{ t('startGame') }}
                </button>
                <div class="eco-info">
                  <svg class="eco-clock" viewBox="0 0 24 24" aria-hidden="true">
                    <circle cx="12" cy="13" r="8" />
                    <path d="M12 9v4l2.5 2M9.5 2h5" />
                  </svg>
                  <p>{{ t('trashInfo', { n: TRASH_GOAL, s: TRASH_SECONDS }) }}</p>
                </div>
              </div>
            </div>
          </transition>

          <template v-if="trashState !== 'intro'">
            <div class="g-hud">
              <div class="g-card g-timer" :class="{ danger: trashTime <= 6 }">
                <svg class="g-clock" viewBox="0 0 24 24" aria-hidden="true">
                  <circle cx="12" cy="13" r="8" /><path d="M12 9v4l2.5 2M9.5 2h5" />
                </svg>
                <div><small>{{ t('timeLeft') }}</small><b>{{ trashTime }}s</b></div>
              </div>
              <div class="g-card g-bar">
                <div class="g-bar-label">{{ t('waterPurified', { n: Math.round((trashCorrect / TRASH_GOAL) * 100) }) }}</div>
                <div class="g-segs"><i v-for="n in TRASH_GOAL" :key="n" :class="{ on: trashCorrect >= n }"></i></div>
              </div>
            </div>

            <div class="t-belt"></div>

            <button
              v-for="it in trashItems"
              :key="it.id"
              class="t-item"
              :class="{ scanning: it.scanning, scanned: it.scanned, selected: selectedId === it.id }"
              :style="{ left: it.x + '%', top: '50%' }"
              :aria-label="t('trashAria')"
              @pointerdown.prevent="scanItem(it)"
            >
              <span class="t-laser"></span>
              <span class="t-tag" v-if="it.scanned">{{ t('aiDetected', { label: TRASH_LABELS[it.kind] }) }}</span>
              <svg viewBox="0 0 100 100" aria-hidden="true">
                <g v-if="it.kind === 'plastic'">
                  <rect x="42" y="6" width="16" height="9" rx="2" fill="#2f80ed" />
                  <path d="M44 15h12v10c0 6 14 10 14 24v38a7 7 0 0 1-7 7H37a7 7 0 0 1-7-7V49c0-14 14-18 14-24z" fill="#bfe8ff" stroke="#5aa9d6" stroke-width="3" />
                  <rect x="32" y="52" width="36" height="16" fill="#2f80ed" opacity="0.85" />
                </g>
                <g v-else-if="it.kind === 'ewaste'">
                  <rect x="42" y="6" width="16" height="10" rx="2" fill="#aab2bd" />
                  <rect x="30" y="16" width="40" height="76" rx="6" fill="#343a44" stroke="#f4c430" stroke-width="3" />
                  <path d="M55 32 L41 58 h10 l-5 22 L64 50 H54z" fill="#f4c430" />
                </g>
                <g v-else>
                  <path d="M50 30c-22-10-36 8-28 30 6 16 16 28 28 28s22-12 28-28c8-22-6-40-28-30z" fill="#d9573f" />
                  <circle cx="38" cy="62" r="5" fill="#7a3b1e" />
                  <circle cx="62" cy="50" r="4" fill="#7a3b1e" />
                  <path d="M50 30c0-8 2-14 6-18" stroke="#6b4a2b" stroke-width="4" stroke-linecap="round" fill="none" />
                  <path d="M56 16c8-6 16-2 18 2-6 6-14 4-18-2z" fill="#4caf50" />
                </g>
              </svg>
            </button>

            <div class="t-bins">
              <button
                v-for="b in BINS"
                :key="b.id"
                class="t-bin"
                :class="{ ok: binFlash.id === b.id && binFlash.ok, bad: binFlash.id === b.id && !binFlash.ok }"
                :style="{ '--bc': b.color, '--bcg': b.glow }"
                @click="sortInto(b.id)"
              >
                <b>{{ b.name }}</b>
                <small>[{{ b.key }}]</small>
                <span class="t-plus" v-if="binFlash.id === b.id && binFlash.ok">+1</span>
              </button>
            </div>

            <div class="t-toast" :class="{ show: trashToast }">{{ trashToast }}</div>
            <div class="g-hint g-card t-hint" :class="{ show: trashHint }">
              <svg viewBox="0 0 24 24" aria-hidden="true">
                <path d="M9 11V5a1.5 1.5 0 0 1 3 0v5m0 0V8.5a1.5 1.5 0 0 1 3 0V11m0 0V10a1.5 1.5 0 0 1 3 0v4.5c0 3-2 6-5.5 6H12c-2 0-3-1-4.2-2.6L5 14a1.5 1.5 0 0 1 2.3-1.9L9 14" />
              </svg>
              <span>{{ t('trashHint') }}</span>
            </div>
          </template>

          <div class="g-overlay" v-if="trashState === 'lost'">
            <div class="g-result g-card">
              <div class="g-result-title">{{ t('timesUp') }}</div>
              <p>{{ t('trashLose', { n: trashCorrect, g: TRASH_GOAL }) }}</p>
              <button class="g-btn" @click="startTrash">{{ t('tryAgain') }}</button>
            </div>
          </div>

          <div class="g-win-flash" v-if="trashWinFlash" aria-hidden="true"></div>
          <div class="g-center" v-if="trashState === 'won' && !trashTxOpen">
            <div class="g-rays"></div>
            <div class="g-title">{{ t('mission') }}</div>
          </div>

          <transition name="fade">
            <div class="g-overlay call-overlay" v-if="trashTxOpen">
              <div class="call-card">
                <div class="tx-header">{{ t('txHeader2') }}</div>
                <p class="reality-message">"{{ TRASH_TX }}"</p>
                <button class="choice call-answer" @click="onTrashContinue">{{ t('continueBtn') }}</button>
              </div>
            </div>
          </transition>
        </div>

        <!-- ===== MINI GAME 3: SATELLITE DRONE REFORESTATION ===== -->
        <div class="drone-root" v-if="droneState !== 'idle'">
          <transition name="fade">
            <div class="eco-overlay" v-if="droneState === 'intro'">
              <div class="eco-card">
                <div class="eco-title">
                  <span class="eco-name">{{ t('droneName') }}</span>
                  <span class="eco-year">{{ t('droneYear') }}</span>
                </div>
                <div class="eco-tag">{{ t('droneTag') }}</div>
                <button class="eco-start" @click="startDrone">
                  <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M6 4l14 8-14 8z" /></svg>
                  {{ t('startGame') }}
                </button>
                <div class="eco-info">
                  <svg class="eco-clock" viewBox="0 0 24 24" aria-hidden="true">
                    <circle cx="12" cy="13" r="8" /><path d="M12 9v4l2.5 2M9.5 2h5" />
                  </svg>
                  <p>{{ t('droneInfo', { n: DRONE_GOAL, s: DRONE_SECONDS }) }}</p>
                </div>
              </div>
            </div>
          </transition>

          <template v-if="droneState !== 'intro'">
            <div class="d-radar" aria-hidden="true"><div class="d-sweep"></div></div>
            <div class="g-hud">
              <div class="g-card g-timer" :class="{ danger: droneTime <= 6 }">
                <svg class="g-clock" viewBox="0 0 24 24" aria-hidden="true">
                  <circle cx="12" cy="13" r="8" /><path d="M12 9v4l2.5 2M9.5 2h5" />
                </svg>
                <div><small>{{ t('timeLeft') }}</small><b>{{ droneTime }}s</b></div>
              </div>
              <div class="g-card g-bar">
                <div class="g-bar-label">{{ t('forestRestored', { n: Math.round((droneGrown / DRONE_GOAL) * 100) }) }}</div>
                <div class="g-segs"><i v-for="n in DRONE_GOAL" :key="n" :class="{ on: droneGrown >= n }"></i></div>
              </div>
            </div>

            <div class="d-grid">
              <button
                v-for="p in dronePlots"
                :key="p.id"
                class="d-plot"
                :class="{ grown: p.grown, launching: p.launching }"
                :style="{ left: p.x + '%', top: p.y + '%' }"
                :aria-label="t('barrenAria')"
                @pointerdown.prevent="plantSeed(p)"
              >
                <span class="d-dot"></span>
                <span class="d-pod"></span>
                <svg class="d-tree" viewBox="0 0 100 100" aria-hidden="true">
                  <rect x="45" y="58" width="10" height="32" rx="3" fill="#7a4f2b" />
                  <circle cx="50" cy="42" r="26" fill="#2fbf5f" />
                  <circle cx="34" cy="54" r="18" fill="#3fd672" />
                  <circle cx="66" cy="54" r="18" fill="#3fd672" />
                </svg>
                <span class="d-signal"><i></i><i></i></span>
              </button>
            </div>

            <div class="g-hint g-card t-hint" :class="{ show: droneHint }">
              <svg viewBox="0 0 24 24" aria-hidden="true">
                <path d="M9 11V5a1.5 1.5 0 0 1 3 0v5m0 0V8.5a1.5 1.5 0 0 1 3 0V11m0 0V10a1.5 1.5 0 0 1 3 0v4.5c0 3-2 6-5.5 6H12c-2 0-3-1-4.2-2.6L5 14a1.5 1.5 0 0 1 2.3-1.9L9 14" />
              </svg>
              <span>{{ t('droneHint') }}</span>
            </div>
          </template>

          <div class="g-overlay" v-if="droneState === 'lost'">
            <div class="g-result g-card">
              <div class="g-result-title">{{ t('timesUp') }}</div>
              <p>{{ t('droneLose', { n: droneGrown, g: DRONE_GOAL }) }}</p>
              <button class="g-btn" @click="startDrone">{{ t('tryAgain') }}</button>
            </div>
          </div>

          <div class="g-win-flash" v-if="droneWinFlash" aria-hidden="true"></div>
          <div class="g-center" v-if="droneState === 'won' && !droneTxOpen">
            <div class="g-rays"></div>
            <div class="g-title">{{ t('mission') }}</div>
          </div>

          <transition name="fade">
            <div class="g-overlay call-overlay" v-if="droneTxOpen">
              <div class="call-card">
                <div class="tx-header">{{ t('txHeader2') }}</div>
                <p class="reality-message">"{{ DRONE_TX }}"</p>
                <button class="choice call-answer" @click="onDroneContinue">{{ t('continueBtn') }}</button>
              </div>
            </div>
          </transition>
        </div>

        <!-- ===== ALL STAGES DONE BANNER ===== -->
        <transition name="fade">
          <div class="stage-banner-wrap" v-if="allDoneBanner">
            <div class="stage-banner">
              <div class="sb-check">
                <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M5 12.5l4.5 4.5L19 7.5" /></svg>
              </div>
              <div class="sb-line1">{{ t('allDone') }}</div>
              <div class="sb-line2 sb-small">{{ t('allDoneSub') }}</div>
              <i class="sb-spark" style="left:8%;top:-6%"></i>
              <i class="sb-spark" style="left:88%;top:-10%;animation-delay:-.5s"></i>
              <i class="sb-spark" style="left:96%;top:60%;animation-delay:-.9s"></i>
            </div>
          </div>
        </transition>

        <!-- ===== HAPPY ENDING TEXT ===== -->
        <transition name="fade">
          <div class="be-layer he-layer" v-if="happyStage >= 1 && !happyContactOpen">
            <transition name="end-text" mode="out-in">
              <p class="be-thanks" v-if="happyStage === 1" key="h1">{{ happyThanksName }}</p>
              <div class="he-title" v-else-if="happyStage === 2" key="h2">
                <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M5 12.5l4.5 4.5L19 7.5" /></svg>
                <span>{{ t('heTitle') }}</span>
              </div>
              <p class="be-thanks" v-else-if="happyStage === 3" key="h3">{{ HE_SUB }}</p>
              <p class="be-thanks" v-else-if="happyStage === 4" key="h4">{{ BE_THANKS }}</p>
              <p class="be-edu" v-else key="h5">{{ BE_EDU }}</p>
            </transition>
            <transition name="fade">
              <button class="be-contact" v-if="happyStage >= 6" @click="happyContactOpen = true; emit('contact')">{{ t('contactUs') }}</button>
            </transition>
            <transition name="fade">
              <button class="be-restart" v-if="happyStage >= 6" @click="restart">{{ t('startOver') }}</button>
            </transition>
          </div>
        </transition>

        <transition name="fade">
          <div class="be-layer be-contact-layer he-contact" v-if="happyContactOpen">
            <div class="ct-title">{{ t('contactUs') }}</div>
            <div class="ct-grid">
              <div class="ct-card" v-for="m in TEAM" :key="m.id">
                <div class="ct-name">{{ m.name }}</div>
                <div class="ct-icons">
                  <a class="ct-icon" :class="{ off: !m.github }" :href="m.github || undefined" target="_blank" rel="noopener noreferrer" :aria-label="'GitHub ' + m.name" :aria-disabled="!m.github" @click="!m.github && $event.preventDefault()">
                    <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M12 .297c-6.63 0-12 5.373-12 12 0 5.303 3.438 9.8 8.205 11.385.6.113.82-.258.82-.577 0-.285-.01-1.04-.015-2.04-3.338.724-4.042-1.61-4.042-1.61C4.422 18.07 3.633 17.7 3.633 17.7c-1.087-.744.084-.729.084-.729 1.205.084 1.838 1.236 1.838 1.236 1.07 1.835 2.809 1.305 3.495.998.108-.776.417-1.305.76-1.605-2.665-.3-5.466-1.332-5.466-5.93 0-1.31.465-2.38 1.235-3.22-.135-.303-.54-1.523.105-3.176 0 0 1.005-.322 3.3 1.23.96-.267 1.98-.399 3-.405 1.02.006 2.04.138 3 .405 2.28-1.552 3.285-1.23 3.285-1.23.645 1.653.24 2.873.12 3.176.765.84 1.23 1.91 1.23 3.22 0 4.61-2.805 5.625-5.475 5.92.42.36.81 1.096.81 2.22 0 1.606-.015 2.896-.015 3.286 0 .315.21.69.825.57C20.565 22.092 24 17.592 24 12.297c0-6.627-5.373-12-12-12" /></svg>
                  </a>
                  <a class="ct-icon" :class="{ off: !m.linkedin }" :href="m.linkedin || undefined" target="_blank" rel="noopener noreferrer" :aria-label="'LinkedIn ' + m.name" :aria-disabled="!m.linkedin" @click="!m.linkedin && $event.preventDefault()">
                    <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M20.447 20.452h-3.554v-5.569c0-1.328-.027-3.037-1.852-3.037-1.853 0-2.136 1.445-2.136 2.939v5.667H9.351V9h3.414v1.561h.046c.477-.9 1.637-1.85 3.37-1.85 3.601 0 4.267 2.37 4.267 5.455v6.286zM5.337 7.433c-1.144 0-2.063-.926-2.063-2.065 0-1.138.92-2.063 2.063-2.063 1.14 0 2.064.925 2.064 2.063 0 1.139-.925 2.065-2.064 2.065zm1.782 13.019H3.555V9h3.564v11.452zM22.225 0H1.771C.792 0 0 .774 0 1.729v20.542C0 23.227.792 24 1.771 24h20.451C23.2 24 24 23.227 22.222 23.227h.003zM22.225 0H1.771C.792 0 0 .774 0 1.729v20.542C0 23.227.792 24 1.771 24h20.451C23.2 24 24 23.227 24 22.271V1.729C24 .774 23.2 0 22.222 0h.003z" /></svg>
                  </a>
                  <button type="button" class="ct-icon" :aria-label="t('copyEmail', { name: m.name })" :title="t('copyEmail', { name: m.name })" @click="copyEmail(m)">
                    <svg viewBox="0 0 24 24" class="ct-stroke" aria-hidden="true"><rect x="3" y="5" width="18" height="14" rx="2" /><path d="M3.5 7l8.5 6.5L20.5 7" /></svg>
                  </button>
                </div>
                <div class="ct-toast" :class="{ show: copiedId === m.id }">{{ t('emailCopied') }}</div>
              </div>
            </div>
            <button class="ct-back" @click="happyContactOpen = false">{{ t('back') }}</button>
          </div>
        </transition>

        <div class="he-dim" :class="{ on: happyStage >= 1 }" aria-hidden="true"></div>
      </div>

      <!-- DONE / handoff to Fase 2 -->
      <div class="stage narrative-bg" v-else-if="phase === 'done'" key="done">
        <div class="narrative-text show handoff-text">
          {{ t('handoff') }}
        </div>
      </div>

      <div class="stage whatever-video-stage" v-else-if="phase === 'whatever'" key="whatever">
        <video
          ref="videoEl"
          class="whatever-video"
          src="/picture/whatever.mp4"
          autoplay
          controls
          playsinline
          @loadedmetadata="applyMaster"
          @ended="onWhateverVideoEnded"
        ></video>
      </div>

      <!-- REALITY CHECK: 3 teks -> fade out -> jeda -> 2 tombol pilihan -->
      <div class="stage reality-check-stage" v-else-if="phase === 'realityCheck'" key="realityCheck">
        <div
          class="reality-check-copy"
          :class="{ hidden: realityHidden }"
          v-if="realityLine >= 0"
        >
          <div class="tx-header">{{ t('realityHeader') }}</div>
          <transition name="reality-text" mode="out-in" appear>
            <p class="reality-message" :key="realityLine">
              {{ realityMessages[realityLine] }}
            </p>
          </transition>
        </div>

        <transition name="fade">
          <div class="reality-choices" v-if="realityChoices">
            <button class="rc-btn rc-still" @click="onStillWhatever">
              {{ t('stillWhatever') }}
            </button>
            <button class="rc-btn rc-fix" @click="onFixThis">
              {{ t('fixThis') }}
            </button>
          </div>
        </transition>
      </div>

      <!-- BAD ENDING 1: "BAD ENDING INITIATED" (timeline rewriting) -->
      <div class="stage bad-init" v-else-if="phase === 'badInit'" key="badInit">
        <div class="bi-noise"></div>
        <div class="glitch"><div class="scan"></div><i></i><i></i><i></i><i></i><i></i></div>

        <div class="bi-panel">
          <svg class="bi-icon" viewBox="0 0 64 60" aria-hidden="true">
            <path d="M32 4 L61 55 H3 Z" />
            <path d="M32 21 V38" />
            <circle cx="32" cy="46" r="1.6" />
          </svg>
          <div class="bi-title">{{ t('badTitle') }}</div>
          <div class="bi-label">{{ t('badLabel') }}</div>
          <div class="bi-bar"><div class="bi-fill" :style="{ width: badPct + '%' }"></div></div>
          <div class="bi-scale"><span>0%</span><span>{{ badPct }}%</span><span>100%</span></div>
        </div>

        <div class="bi-log" aria-hidden="true">
          <p style="--i: 0">{{ t('badLog1') }}</p>
          <p style="--i: 1">{{ t('badLog2') }}</p>
          <p style="--i: 2">{{ t('badLog3') }}</p>
          <p style="--i: 3">{{ t('badLog4') }}</p>
          <p style="--i: 4">{{ t('badLog5') }}</p>
        </div>
      </div>

      <!-- BAD ENDING 2: city2 + asap/api -> warning -> teks -> penutup Team Moli -->
      <div class="stage bad-ending" v-else-if="phase === 'badEnding'" key="badEnding">
        <img class="be-bg" src="/picture/city2.avif" alt="" draggable="false" />

        <!-- asap kebakaran + polusi -->
        <div class="smoke-layer" aria-hidden="true">
          <div class="smog smog-top"></div>
          <div class="smog smog-mid"></div>
          <div class="smog smog-low"></div>

          <div class="fire-glow fg-1"></div>
          <div class="fire-glow fg-2"></div>
          <div class="fire-glow fg-3"></div>
          <div class="fire-glow fg-4"></div>

          <span
            v-for="p in badPlumes"
            :key="'bp' + p.id"
            class="plume"
            :class="{ dark: p.dark }"
            :style="p.style"
          ></span>

          <span
            v-for="e in badEmbers"
            :key="'be' + e.id"
            class="ember"
            :style="e.style"
          ></span>
        </div>

        <div class="smoke-front" aria-hidden="true">
          <span
            v-for="w in badWisps"
            :key="'bw' + w.id"
            class="wisp"
            :style="w.style"
          ></span>
        </div>

        <div class="fire-flicker" aria-hidden="true"></div>
        <div class="inspect-vignette"></div>
        <div class="be-dim" :class="{ on: endStage >= 3 }"></div>
        <div class="glitch"><div class="scan"></div></div>

        <transition name="fade">
          <div class="be-layer be-top" v-if="endStage >= 1 && endStage <= 2">
            <div class="be-warning">
              <svg class="be-warn-icon" viewBox="0 0 64 60" aria-hidden="true">
                <path d="M32 4 L61 55 H3 Z" />
                <path d="M32 21 V38" />
                <circle cx="32" cy="46" r="1.6" />
              </svg>
              <span>{{ t('beWarn') }}</span>
            </div>
            <transition name="fade">
              <p class="be-sub" v-if="endStage >= 2">{{ BE_SUB }}</p>
            </transition>
          </div>
        </transition>

        <!-- Contact Us -->
        <transition name="fade">
          <div class="be-layer be-contact-layer" v-if="contactOpen">
            <div class="ct-title">{{ t('contactUs') }}</div>
            <div class="ct-grid">
              <div class="ct-card" v-for="m in TEAM" :key="m.id">
                <div class="ct-name">{{ m.name }}</div>
                <div class="ct-icons">
                  <a
                    class="ct-icon"
                    :class="{ off: !m.github }"
                    :href="m.github || undefined"
                    target="_blank"
                    rel="noopener noreferrer"
                    :aria-label="'GitHub ' + m.name"
                    :aria-disabled="!m.github"
                    :title="m.github ? 'GitHub' : 'GitHub belum tersedia'"
                    @click="!m.github && $event.preventDefault()"
                  >
                    <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M12 .297c-6.63 0-12 5.373-12 12 0 5.303 3.438 9.8 8.205 11.385.6.113.82-.258.82-.577 0-.285-.01-1.04-.015-2.04-3.338.724-4.042-1.61-4.042-1.61C4.422 18.07 3.633 17.7 3.633 17.7c-1.087-.744.084-.729.084-.729 1.205.084 1.838 1.236 1.838 1.236 1.07 1.835 2.809 1.305 3.495.998.108-.776.417-1.305.76-1.605-2.665-.3-5.466-1.332-5.466-5.93 0-1.31.465-2.38 1.235-3.22-.135-.303-.54-1.523.105-3.176 0 0 1.005-.322 3.3 1.23.96-.267 1.98-.399 3-.405 1.02.006 2.04.138 3 .405 2.28-1.552 3.285-1.23 3.285-1.23.645 1.653.24 2.873.12 3.176.765.84 1.23 1.91 1.23 3.22 0 4.61-2.805 5.625-5.475 5.92.42.36.81 1.096.81 2.22 0 1.606-.015 2.896-.015 3.286 0 .315.21.69.825.57C20.565 22.092 24 17.592 24 12.297c0-6.627-5.373-12-12-12" /></svg>
                  </a>
                  <a
                    class="ct-icon"
                    :class="{ off: !m.linkedin }"
                    :href="m.linkedin || undefined"
                    target="_blank"
                    rel="noopener noreferrer"
                    :aria-label="'LinkedIn ' + m.name"
                    :aria-disabled="!m.linkedin"
                    :title="m.linkedin ? 'LinkedIn' : 'LinkedIn belum tersedia'"
                    @click="!m.linkedin && $event.preventDefault()"
                  >
                    <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M20.447 20.452h-3.554v-5.569c0-1.328-.027-3.037-1.852-3.037-1.853 0-2.136 1.445-2.136 2.939v5.667H9.351V9h3.414v1.561h.046c.477-.9 1.637-1.85 3.37-1.85 3.601 0 4.267 2.37 4.267 5.455v6.286zM5.337 7.433c-1.144 0-2.063-.926-2.063-2.065 0-1.138.92-2.063 2.063-2.063 1.14 0 2.064.925 2.064 2.063 0 1.139-.925 2.065-2.064 2.065zm1.782 13.019H3.555V9h3.564v11.452zM22.225 0H1.771C.792 0 0 .774 0 1.729v20.542C0 23.227.792 24 1.771 24h20.451C23.2 24 24 23.227 24 22.271V1.729C24 .774 23.2 0 22.222 0h.003z" /></svg>
                  </a>
                  <button
                    type="button"
                    class="ct-icon"
                    :aria-label="t('copyEmail', { name: m.name })"
                    :title="t('copyEmail', { name: m.name })"
                    @click="copyEmail(m)"
                  >
                    <svg viewBox="0 0 24 24" class="ct-stroke" aria-hidden="true">
                      <rect x="3" y="5" width="18" height="14" rx="2" />
                      <path d="M3.5 7l8.5 6.5L20.5 7" />
                    </svg>
                  </button>
                </div>
                <div class="ct-toast" :class="{ show: copiedId === m.id }">{{ t('emailCopied') }}</div>
              </div>
            </div>
            <button class="ct-back" @click="contactOpen = false">{{ t('back') }}</button>
          </div>
        </transition>

        <!-- penutup: terima kasih -> edukasi by Molly The Gank -> Contact Us -->
        <transition name="fade">
          <div class="be-layer be-final" v-if="endStage >= 4 && !contactOpen">
            <transition name="end-text" mode="out-in">
              <p class="be-thanks" v-if="endStage === 4" key="thanks">{{ BE_THANKS }}</p>
              <p class="be-edu" v-else key="edu">{{ BE_EDU }}</p>
            </transition>
            <transition name="fade">
              <button class="be-contact" v-if="endStage >= 6" @click="onContact">
                {{ t('contactUs') }}
              </button>
            </transition>
            <transition name="fade">
              <button class="be-restart" v-if="endStage >= 6" @click="restart">
                {{ t('startOver') }}
              </button>
            </transition>
          </div>
        </transition>
      </div>
    </transition>
    <div v-if="showVolume" class="vol-ctrl">
      <button
        class="vol-btn"
        type="button"
        @click="toggleMute"
        :aria-label="volMuted ? 'Unmute' : 'Mute'"
        :title="volMuted ? 'Unmute' : 'Mute'"
      >
        <svg viewBox="0 0 24 24" aria-hidden="true">
          <path d="M11 5L6 9H2v6h4l5 4V5z" />
          <template v-if="volMuted">
            <path d="M22 9l-6 6M16 9l6 6" />
          </template>
          <template v-else>
            <path d="M15.5 8.5a5 5 0 0 1 0 7" />
            <path v-if="masterVolume > 0.5" d="M19 5a10 10 0 0 1 0 14" />
          </template>
        </svg>
      </button>
      <input
        class="vol-range"
        type="range"
        min="0"
        max="1"
        step="0.01"
        :value="masterMuted ? 0 : masterVolume"
        @input="onVolInput($event.target.value)"
        aria-label="Volume"
      />
    </div>
    <button
      v-if="showLang"
      class="lang-toggle"
      type="button"
      @click="toggleLang"
      :aria-label="lang === 'en' ? 'Ganti ke Bahasa Indonesia' : 'Switch to English'"
    >
      <span :class="{ on: lang === 'id' }">ID</span><i></i><span :class="{ on: lang === 'en' }">EN</span>
    </button>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, onBeforeUnmount } from 'vue'
import { setCursorTheme } from '../utils/cursorTheme.js'

const emit = defineEmits(['finished', 'done', 'contact'])

// ================= I18N =================
const lang = ref('en')
try {
  const saved = localStorage.getItem('gg-lang')
  if (saved === 'en' || saved === 'id') lang.value = saved
} catch {}
watch(lang, (value) => { try { localStorage.setItem('gg-lang', value) } catch {} })
function toggleLang() {
  lang.value = lang.value === 'en' ? 'id' : 'en'
  if (lang.value === 'id') audioEl.value?.pause()
}

function t(key, params) {
  let text = I18N[lang.value][key] ?? I18N.en[key] ?? key
  if (params) {
    for (const keyName in params) text = text.replaceAll(`{${keyName}}`, params[keyName])
  }
  return text
}

const I18N = {
  en: {
    goSub: 'Click to start',
    handoff: 'Phase 1 complete — continue to Phase 2.',
    hpLine1: 'Use headphones for', hpLine2: 'best experience',
    txHeader: '[ ⚠️ INCOMING TRANSMISSION // 2076 ]',
    whatever: 'Whatever!',
    inspectBtn: '👆 CLICK THE 3 WARNING NODES TO INSPECT YOUR SURROUNDINGS',
    hintClick: 'CLICK', hintRest: 'THE 3 WARNING NODES\nTO INSPECT YOUR SURROUNDINGS',
    scan: '[ ⚠️ SCAN // {label} ]', close: 'Close',
    unlocked: '[ TRANSMISSION UNLOCKED — SCROLL TO PREVENT 2076 ↓ ]',
    identity: '[ IDENTITY CHECK // 2076 ]', nickLabel: 'What should I call you?',
    nickPh: 'Your nickname', continue: 'Continue',
    line1: 'Hey...', line2: 'Hey, wake up!', line3: 'Are you okay?',
    line4: "Hmm...? What happened?\nIt's just... so hot today...",
    line5: 'Who are you? Why are you calling me?',
    txTop1: 'You need to wake up! Look around you. The heat, the haze... this is where it all begins.',
    txReply: "What do you mean? It's just a bad weather day...",
    txTop2: "No. I'm speaking from 2076. Where I stand, there are no green trees left. The air burns. 50°C is our coolest day.",
    povLine: "Wait... those numbers... this isn't just bad weather. It's a slow collapse.",
    txFinal: 'Now you see the reality. Scroll down before these warnings become your permanent nightmare.',
    wakeLine: 'Huh...? Where am I?', hello: 'Hello, {name}!',
    welcome: 'Welcome to Go Green!', wake4: "Let's take care of our planet together!",
    ecoTag: 'CLEAN THE AIR, BRIGHTER TOMORROW', startGame: 'START GAME',
    ecoInfo: 'Clear the pollution within {s} seconds\nto restore clean air!',
    timeLeft: 'TIME LEFT', cleanAir: 'CLEAN AIR: {n}%', pause: '❚❚ PAUSE',
    paused: 'PAUSED', resume: 'RESUME', timesUp: "TIME'S UP!", tryAgain: 'TRY AGAIN',
    cleanLose: 'Only {n}% of the air is clean.\nThe city needs you. Try again!',
    mission: 'MISSION\nCOMPLETE!', seedAria: 'Green seed', pollAria: 'Pollution bubble',
    gameHint: 'Tap the pollution bubbles\nto clean the air!',
    callHeader: '[ ⚠️ INCOMING CALL // 2076 ]', callName: 'Caller from 2076',
    callSub: "The air is clean again... they're calling you!", answer: 'ANSWER',
    callAlt: 'A person from 2076 celebrating the restored air',
    callControls: 'Call controls', mute: 'Mute microphone', unmute: 'Unmute microphone',
    endCall: 'End call', videoOff: 'Turn video off', videoOn: 'Turn video on',
    call1: 'WAIT... WHAT IS HAPPENING?!',
    call2: 'The air quality just improved so much! I can see the sky again!',
    call3: 'You actually saved our future! Thank you!',
    stage1Done: 'STAGE 1 COMPLETED:', airPurified: 'AIR PURIFIED!',
    stage2Done: 'STAGE 2 COMPLETED:', waterDone: 'WATER PURIFIED!',
    allDone: 'ALL STAGES DONE!', allDoneSub: 'You completed all 3 stages!',
    trashName: 'AI TRASH', trashYear: 'SCANNER', trashTag: 'SORT THE WASTE, SAVE THE WATER',
    trashInfo: 'Tap an item to scan it, then tap the right bin.\nSort {n} items in {s} seconds!',
    waterPurified: 'WATER PURIFIED: {n}%', trashAria: 'Trash item',
    aiDetected: '[AI DETECTED: {label}]',
    lblPlastic: 'PET PLASTIC', lblEwaste: 'E-WASTE / BATTERY', lblOrganic: 'ORGANIC WASTE',
    binPlastic: 'PLASTIC', binEwaste: 'E-WASTE', binOrganic: 'ORGANIC',
    scanFirst: 'SCAN AN ITEM FIRST', wrongBin: 'WRONG BIN  -1s',
    trashHint: 'Tap an item to scan it,\nthen tap the right bin!',
    trashLose: 'Only {n} of {g} items sorted.\nThe water still needs you. Try again!',
    txHeader2: '[ 📡 TRANSMISSION 2076 ]', continueBtn: 'CONTINUE',
    trashTx: "Incredible! The microplastics in our bunker's drinking water just vanished completely!",
    droneName: 'SATELLITE', droneYear: 'DRONE', droneTag: 'REFOREST THE LAND, RESTORE THE OXYGEN',
    droneInfo: 'Tap the barren spots to launch Smart Seed Pods.\nGrow {n} trees in {s} seconds!',
    forestRestored: 'FOREST RESTORED: {n}%', barrenAria: 'Barren soil',
    droneHint: 'Tap the barren spots\nto plant trees!',
    droneLose: 'Only {n} of {g} trees grown.\nThe forest still needs you. Try again!',
    droneTx: 'Kalimantan forest is dense again! Atmospheric oxygen levels are rising fast!',
    heTitle: 'HAPPY ENDING UNLOCKED:\nTIMELINE RESTORED',
    heSub: 'You rewrote the timeline.\nIn 2076, the Earth breathes again.',
    thanksName: 'Thank you, {name}, for helping save this Earth.',
    realityHeader: '[ ⚠️ REALITY CHECK // KALIMANTAN 2026 ]',
    r1: 'Look closely. That was Kalimantan in 2026.',
    r2: 'Thousands of species lost, toxic smog choking millions... and you still looked away?',
    r3: "Still think it's 'whatever'?",
    stillWhatever: 'Still whatever', fixThis: 'I WAS WRONG. LET ME FIX THIS',
    badTitle: 'BAD ENDING INITIATED', badLabel: 'TIMELINE REWRITING ...',
    badLog1: 'IGNORING WARNINGS...', badLog2: 'CLIMATE SYSTEM FAILING...',
    badLog3: 'ECOSYSTEM COLLAPSED...', badLog4: 'HUMANITY AT RISK...',
    badLog5: 'LOADING 2076 TIMELINE...',
    beWarn: 'BAD ENDING UNLOCKED:\nTIMELINE ERASED',
    beSub: "You ignored every warning.\nIn 2076, the Earth didn't survive.\nThis is all that's left of our civilization.",
    beThanks: 'Thank you for visiting our website.',
    beEdu: 'This website was made for education,\nby Molly The Gank.',
    contactUs: 'Contact Us', startOver: 'Start over', back: 'Back',
    emailCopied: 'Email copied!', copyEmail: 'Copy email {name}',
    nodes: {
      heat: { label: 'HEAT', title: 'Heat Warning / Heatwave', nowV: '38°C', nowN: 'Extreme Heat',
        futV: '50°C - 52°C', futN: 'Protective gear / cooling masks required to go outside',
        msg: 'The AC you ran all day in 2026 is burning our atmosphere today.' },
      air: { label: 'AIR QUALITY', title: 'Air Pollution', nowV: 'AQI 175', nowN: 'Unhealthy',
        futV: 'AQI 450+', futN: 'Hazardous Toxic Air',
        msg: "You called it a thin haze. We call it toxic air we can't breathe." },
      eco: { label: 'ECOSYSTEM', title: 'Waste & Rivers', nowV: '70%', nowN: 'Plastic waste piling up in rivers',
        futV: 'Total Clean Water Crisis', futN: 'Our last drinking water supply',
        msg: 'The single-use plastic you threw away yesterday is swimming in our last drinking water supply.' }
    }
  },
  id: {
    goSub: 'Klik untuk mulai',
    handoff: 'Fase 1 selesai — lanjut ke Fase 2.',
    hpLine1: 'Gunakan headphone untuk', hpLine2: 'pengalaman terbaik',
    txHeader: '[ ⚠️ TRANSMISI MASUK // 2076 ]',
    whatever: 'Terserah!',
    inspectBtn: '👆 KLIK 3 NODE PERINGATAN UNTUK MEMERIKSA SEKITARMU',
    hintClick: 'KLIK', hintRest: 'KE-3 NODE PERINGATAN\nUNTUK MEMERIKSA SEKITARMU',
    scan: '[ ⚠️ PINDAI // {label} ]', close: 'Tutup',
    unlocked: '[ TRANSMISI TERBUKA — SCROLL UNTUK MENCEGAH 2076 ↓ ]',
    identity: '[ PEMERIKSAAN IDENTITAS // 2076 ]', nickLabel: 'Aku harus panggil kamu apa?',
    nickPh: 'Nama panggilanmu', continue: 'Lanjut',
    line1: 'Hei...', line2: 'Hei, bangun!', line3: 'Kamu baik-baik saja?',
    line4: 'Hmm...? Apa yang terjadi?\nCuma... panas banget hari ini...',
    line5: 'Kamu siapa? Kenapa kamu menghubungiku?',
    txTop1: 'Kamu harus bangun! Lihat sekelilingmu. Panasnya, kabutnya... di sinilah semuanya bermula.',
    txReply: 'Maksudmu apa? Ini cuma hari dengan cuaca buruk...',
    txTop2: 'Bukan. Aku bicara dari tahun 2076. Di tempatku berdiri, tidak ada lagi pohon hijau. Udaranya membakar. 50°C adalah hari terdingin kami.',
    povLine: 'Tunggu... angka-angka itu... ini bukan sekadar cuaca buruk. Ini keruntuhan yang perlahan.',
    txFinal: 'Sekarang kamu lihat kenyataannya. Scroll ke bawah sebelum peringatan ini jadi mimpi burukmu yang permanen.',
    wakeLine: 'Hah...? Aku di mana?', hello: 'Halo, {name}!',
    welcome: 'Selamat datang di Go Green!', wake4: 'Ayo kita jaga planet kita bersama-sama!',
    ecoTag: 'BERSIHKAN UDARA, MASA DEPAN LEBIH CERAH', startGame: 'MULAI GAME',
    ecoInfo: 'Bersihkan polusi dalam {s} detik\nuntuk memulihkan udara bersih!',
    timeLeft: 'SISA WAKTU', cleanAir: 'UDARA BERSIH: {n}%', pause: '❚❚ JEDA',
    paused: 'DIJEDA', resume: 'LANJUTKAN', timesUp: 'WAKTU HABIS!', tryAgain: 'COBA LAGI',
    cleanLose: 'Baru {n}% udara yang bersih.\nKota ini membutuhkanmu. Coba lagi!',
    mission: 'MISI\nSELESAI!', seedAria: 'Biji hijau', pollAria: 'Gelembung polusi',
    gameHint: 'Ketuk gelembung polusi\nuntuk membersihkan udara!',
    callHeader: '[ ⚠️ PANGGILAN MASUK // 2076 ]', callName: 'Penelepon dari 2076',
    callSub: 'Udaranya bersih lagi... mereka meneleponmu!', answer: 'ANGKAT',
    callAlt: 'Seseorang dari 2076 merayakan udara yang pulih',
    callControls: 'Kontrol panggilan', mute: 'Matikan mikrofon', unmute: 'Nyalakan mikrofon',
    endCall: 'Akhiri panggilan', videoOff: 'Matikan video', videoOn: 'Nyalakan video',
    call1: 'TUNGGU... APA YANG TERJADI?!',
    call2: 'Kualitas udaranya tiba-tiba membaik banget! Aku bisa lihat langit lagi!',
    call3: 'Kamu benar-benar menyelamatkan masa depan kami! Terima kasih!',
    stage1Done: 'TAHAP 1 SELESAI:', airPurified: 'UDARA BERSIH!',
    stage2Done: 'TAHAP 2 SELESAI:', waterDone: 'AIR BERSIH!',
    allDone: 'SEMUA TAHAP SELESAI!', allDoneSub: 'Kamu menyelesaikan ketiga tahap!',
    trashName: 'AI SAMPAH', trashYear: 'SCANNER', trashTag: 'PILAH SAMPAH, SELAMATKAN AIR',
    trashInfo: 'Ketuk item untuk memindai, lalu ketuk tong yang tepat.\nPilah {n} item dalam {s} detik!',
    waterPurified: 'AIR TERSARING: {n}%', trashAria: 'Item sampah',
    aiDetected: '[AI MENDETEKSI: {label}]',
    lblPlastic: 'PLASTIK PET', lblEwaste: 'LIMBAH ELEKTRONIK / BATERAI', lblOrganic: 'SAMPAH ORGANIK',
    binPlastic: 'PLASTIK', binEwaste: 'E-WASTE', binOrganic: 'ORGANIK',
    scanFirst: 'PINDAI ITEM DULU', wrongBin: 'TONG SALAH  -1d',
    trashHint: 'Ketuk item untuk memindai,\nlalu ketuk tong yang tepat!',
    trashLose: 'Baru {n} dari {g} item terpilah.\nAir masih membutuhkanmu. Coba lagi!',
    txHeader2: '[ 📡 TRANSMISI 2076 ]', continueBtn: 'LANJUT',
    trashTx: 'Luar biasa! Mikroplastik di air minum bunker kami hilang sepenuhnya!',
    droneName: 'SATELIT', droneYear: 'DRONE', droneTag: 'HIJAUKAN LAHAN, PULIHKAN OKSIGEN',
    droneInfo: 'Ketuk lahan gersang untuk meluncurkan Smart Seed Pod.\nTumbuhkan {n} pohon dalam {s} detik!',
    forestRestored: 'HUTAN PULIH: {n}%', barrenAria: 'Tanah gersang',
    droneHint: 'Ketuk lahan gersang\nuntuk menanam pohon!',
    droneLose: 'Baru {n} dari {g} pohon tumbuh.\nHutan masih membutuhkanmu. Coba lagi!',
    droneTx: 'Hutan Kalimantan rimbun lagi! Kadar oksigen di atmosfer naik dengan cepat!',
    heTitle: 'HAPPY ENDING TERBUKA:\nLINIMASA PULIH',
    heSub: 'Kamu menulis ulang linimasa.\nDi tahun 2076, Bumi bernapas lagi.',
    thanksName: 'Terima kasih, {name}, sudah membantu menyelamatkan Bumi ini.',
    realityHeader: '[ ⚠️ CEK REALITA // KALIMANTAN 2026 ]',
    r1: 'Lihat baik-baik. Itulah Kalimantan di tahun 2026.',
    r2: 'Ribuan spesies punah, kabut asap beracun mencekik jutaan orang... dan kamu masih berpaling?',
    r3: "Masih merasa ini 'terserah'?",
    stillWhatever: 'Tetap terserah', fixThis: 'AKU SALAH. BIAR AKU PERBAIKI',
    badTitle: 'BAD ENDING DIMULAI', badLabel: 'LINIMASA DITULIS ULANG ...',
    badLog1: 'MENGABAIKAN PERINGATAN...', badLog2: 'SISTEM IKLIM GAGAL...',
    badLog3: 'EKOSISTEM RUNTUH...', badLog4: 'UMAT MANUSIA TERANCAM...',
    badLog5: 'MEMUAT LINIMASA 2076...',
    beWarn: 'BAD ENDING TERBUKA:\nLINIMASA TERHAPUS',
    beSub: 'Kamu mengabaikan semua peringatan.\nDi tahun 2076, Bumi tidak selamat.\nInilah yang tersisa dari peradaban kami.',
    beThanks: 'Terima kasih sudah mengunjungi website kami.',
    beEdu: 'Website ini dibuat untuk edukasi,\noleh Molly The Gank.',
    contactUs: 'Hubungi Kami', startOver: 'Mulai ulang', back: 'Kembali',
    emailCopied: 'Email disalin!', copyEmail: 'Salin email {name}',
    nodes: {
      heat: { label: 'PANAS', title: 'Peringatan Panas / Gelombang Panas', nowV: '38°C', nowN: 'Panas Ekstrem',
        futV: '50°C - 52°C', futN: 'Wajib pakai pelindung / masker pendingin untuk keluar rumah',
        msg: 'AC yang kamu nyalakan seharian di 2026 membakar atmosfer kami hari ini.' },
      air: { label: 'KUALITAS UDARA', title: 'Polusi Udara', nowV: 'AQI 175', nowN: 'Tidak Sehat',
        futV: 'AQI 450+', futN: 'Udara Beracun Berbahaya',
        msg: 'Kamu bilang itu kabut tipis. Kami menyebutnya udara beracun yang tak bisa kami hirup.' },
      eco: { label: 'EKOSISTEM', title: 'Sampah & Sungai', nowV: '70%', nowN: 'Sampah plastik menumpuk di sungai',
        futV: 'Krisis Air Bersih Total', futN: 'Sisa terakhir air minum kami',
        msg: 'Plastik sekali pakai yang kamu buang kemarin kini berenang di sisa terakhir air minum kami.' }
    }
  }
}

// phase: loading | goGreen | headphone | blackout | waking | narrative | transmission
//        | inspect | done | whatever | realityCheck | badInit | badEnding
const phase = ref('loading')
const realityLine = ref(-1)
const realityHidden = ref(false)   // teks reality check fade out
const realityChoices = ref(false)  // 2 tombol pilihan muncul
const pct = ref(0)
const narrativeIndex = ref(-1)
const showLine = ref(false)
const wakeStage = ref(1) // 1 | 2 | 3
const txStage = ref(0) // 1: kanan atas #1 | 2: balasan bawah | 3: kanan atas #2 | 4: pilihan

// how long the "Use headphones..." screen stays before moving on
const HEADPHONE_HOLD_MS = 3500
// Phase 0: how long the screen stays black (heartbeat + garis merah)
const BLACKOUT_MS = 1800

// durasi tiap fase waking (harus sama dengan durasi CSS animation)
const P1_MS = 4200 // "Hey..."
const P2_MS = 6200 // "Hey, wake up!"
const P3_MS = 4200 // "Are you okay?"

// Ganti people1 -> people2 dilakukan SAAT mata merem total di Phase 3.
// Di CSS eyeP3: merem total di 24% (~1.0s) sampai 40% (~1.7s) dari P3_MS.
// 1200ms jatuh tepat di tengah fase merem itu. Kalau P3_MS / keyframes eyeP3
// diubah, sesuaikan angka ini ke tengah-tengah fase merem.
const EYES_CLOSED_SWAP_MS = 1200

// durasi transmission
const TX1_MS = 5000
const TX2_MS = 3500
const TX3_MS = 6000

const lines = computed(() => [t('line1'), t('line2'), t('line3'), t('line4'), t('line5')])
const TX_TOP_1 = computed(() => t('txTop1'))
const TX_REPLY = computed(() => t('txReply'))
const TX_TOP_2 = computed(() => t('txTop2'))
const realityMessages = computed(() => [t('r1'), t('r2'), t('r3')])
let realityTimer = null

// --- timing reality check -> pilihan ---------------------------------------
const REALITY_LINE_MS = 4000  // tiap teks reality check tampil selama ini
const REALITY_FADE_MS = 1200  // durasi fade out teks (samakan dgn CSS .reality-check-copy)
const REALITY_PAUSE_MS = 2500 // jeda layar kosong sebelum 2 tombol muncul

// --- timing bad ending -----------------------------------------------------
const BAD_INIT_MS = 4200      // lama layar "BAD ENDING INITIATED"
const BAD_INIT_FILL_MS = 3600 // lama progress bar 0% -> 100%
const BE_WARNING_AT = 900     // warning "BAD ENDING UNLOCKED" muncul
const BE_SUB_AT = 3000        // teks "You ignored every warning..." muncul
const BE_CLEAR_AT = 10500     // warning + teks hilang bersamaan
const BE_THANKS_AT = 12500    // "Thank you for visiting..." muncul
const BE_EDU_AT = 17500       // ganti jadi teks edukasi Team Moli
const BE_CONTACT_AT = 19500   // tombol Contact Us muncul

const BE_SUB = computed(() => t('beSub'))
const BE_THANKS = computed(() => t('beThanks'))
const BE_EDU = computed(() => t('beEdu'))

// Contact Us: data anggota tim 
const TEAM = [
  {
    id: 'caren',
    name: 'Netanya Caren Hilary',
    github: 'https://github.com/carennetanya',
    linkedin: 'https://www.linkedin.com/in/caren-netanya-137a56379/',
    email: 'netanya.caren@gmail.com'
  },
  {
    id: 'kevin',
    name: 'Kevin Matthew Siregar',
    github: 'https://github.com/Kevin-Lng',
    linkedin: 'https://www.linkedin.com/in/kevin-matthew-siregar',
    email: 'kecinmatthewsiregar31@gmail.com'
  },
  {
    id: 'rafael',
    name: 'Rafael Julio Suseno',
    github: 'https://github.com/rafanih',
    linkedin: 'https://www.linkedin.com/in/rafael-julio-57b771384/',
    email: 'rafael.julio0807@gmail.com'
  }
]
const contactOpen = ref(false)
const copiedId = ref(null)
let copiedTimer = null

// --- dialog setelah 3 pin selesai ------------------------------------------
const POV_LINE = computed(() => t('povLine'))
const TX_FINAL = computed(() => t('txFinal'))

// --- gambar -----------------------------------------------------------
// Semua ada di /public/picture/
// people1 -> phase 1 & 2 (dan awal phase 3 sampai mata merem)
// people2 -> setelah mata merem total lalu melek (phase 3 akhir + narrative)
const PEOPLE_EARLY = '/picture/people1.avif'
const PEOPLE_AWAKE = '/picture/people2.avif'

// ganti gambar dikontrol manual (bukan dari wakeStage),
// supaya swap terjadi tepat saat mata sedang tertutup
const personAwake = ref(false)
const personSrc = computed(() =>
  personAwake.value ? PEOPLE_AWAKE : PEOPLE_EARLY
)

// --- audio ------------------------------------------------------------
// Voice files live in /public/voice/. Index sesuai `lines`.
const audioFiles = ['', 'kevin-1.ogg', 'kevin-2.ogg', '', '']

// Background music lives in /public/music/. Starts on the given line index.
const MUSIC_FILE = '/music/intro.mp3'
const MUSIC_START_LINE_INDEX = 2 // "Are you okay?"
let musicStarted = false

// SFX
const HEARTBEAT_FILE = '/music/heartbeat.mp3'
const RADIO_FILE = '/music/radio-signal.mp3'
const SYNC_FILE = '/music/data_synced.mp3'   // sinyal terhubung (atau ganti static radio)
const NATURE_FILE = '/music/nature.mp3'      // angin / burung halus (dunia hijau)

const audioEl = ref(null)
const musicEl = ref(null)
const sfxEl = ref(null)    // heartbeat
const radioEl = ref(null)  // radio signal
const syncEl = ref(null)   // data synced
const natureEl = ref(null) // suasana alam
const endingEl = ref(null) // lagu ending (mulai saat video "whatever" selesai)
const openingEl = ref(null) // lagu opening dunia hijau (mulai saat scroll / langit biru)
const mainEl = ref(null) // musik utama dunia hijau

// ================= MASTER VOLUME =================
const masterVolume = ref(1)
const masterMuted = ref(false)
const videoEl = ref(null)
const volMuted = computed(() => masterMuted.value || masterVolume.value === 0)
const showVolume = computed(() => phase.value !== 'loading' && phase.value !== 'whatever')

let audioCtx = null
let masterGain = null

function allAudioRefs() {
  return [audioEl, musicEl, sfxEl, radioEl, syncEl, natureEl, endingEl, fireEl, openingEl, mainEl]
}

function initAudioGraph() {
  if (audioCtx) { audioCtx.resume?.(); return }
  const Ctx = window.AudioContext || window.webkitAudioContext
  if (!Ctx) return
  try {
    audioCtx = new Ctx()
    masterGain = audioCtx.createGain()
    masterGain.connect(audioCtx.destination)
    for (const audioRef of allAudioRefs()) {
      if (audioRef.value) audioCtx.createMediaElementSource(audioRef.value).connect(masterGain)
    }
    audioCtx.resume?.()
    applyMaster()
  } catch {
    audioCtx = null
    masterGain = null
  }
}

function applyMaster() {
  const muted = volMuted.value
  if (masterGain && audioCtx) {
    masterGain.gain.setTargetAtTime(muted ? 0 : masterVolume.value, audioCtx.currentTime, 0.03)
  } else {
    for (const audioRef of allAudioRefs()) if (audioRef.value) audioRef.value.muted = muted
  }
  const video = videoEl.value
  if (video) {
    video.volume = masterVolume.value
    video.muted = muted
  }
}

function toggleMute() {
  audioCtx?.resume?.()
  if (volMuted.value) {
    masterMuted.value = false
    if (masterVolume.value === 0) masterVolume.value = 0.6
  } else {
    masterMuted.value = true
  }
}

function onVolInput(value) {
  audioCtx?.resume?.()
  masterVolume.value = Number(value)
  if (masterVolume.value > 0) masterMuted.value = false
}

watch([masterVolume, masterMuted], applyMaster)

// Lagu ending: mulai saat video selesai (layar hitam sebelum teks reality check)
const ENDING_FILE = '/music/ending.mp3'
const ENDING_VOLUME = 0.8
let endingIv = null

function playEnding() {
  const el = endingEl.value
  if (!el) return
  clearInterval(endingIv)
  el.src = ENDING_FILE
  el.currentTime = 0
  el.volume = ENDING_VOLUME
  el.play().catch(() => {})
}

function stopEnding(fadeMs = 1500) {
  const el = endingEl.value
  if (!el || el.paused) return
  clearInterval(endingIv)
  const steps = 20
  const startVol = el.volume
  let i = 0
  endingIv = setInterval(() => {
    i++
    el.volume = Math.max(0, startVol * (1 - i / steps))
    if (i >= steps) {
      clearInterval(endingIv)
      el.pause()
    }
  }, fadeMs / steps)
}

const fireEl = ref(null)   // suara api kebakar (scene buka mata)

// Suara api: mulai di "Hey..." (buka mata), loop selama scene kota terbakar
const FIRE_FILE = '/music/fire.mp3'
const FIRE_VOLUME = 0.5
let fireIv = null

function playFire(fadeMs = 1500) {
  const el = fireEl.value
  if (!el) return
  clearInterval(fireIv)
  el.src = FIRE_FILE
  el.currentTime = 0
  el.volume = 0
  el.play().catch(() => {})
  const steps = 20
  let i = 0
  fireIv = setInterval(() => {
    i++
    el.volume = Math.min(FIRE_VOLUME, FIRE_VOLUME * (i / steps))
    if (i >= steps) clearInterval(fireIv)
  }, fadeMs / steps)
}

function stopFire(fadeMs = 1500) {
  const el = fireEl.value
  if (!el || el.paused) return
  clearInterval(fireIv)
  const steps = 20
  const startVol = el.volume
  let i = 0
  fireIv = setInterval(() => {
    i++
    el.volume = Math.max(0, startVol * (1 - i / steps))
    if (i >= steps) {
      clearInterval(fireIv)
      el.pause()
    }
  }, fadeMs / steps)
}

let cancelled = false // true setelah pilihan, supaya timeout lama tidak jalan

function playLineAudio(i) {
  if (lang.value === 'id') return
  const file = audioFiles[i]
  if (!file || !audioEl.value) return
  audioEl.value.src = `/voice/${file}`
  audioEl.value.currentTime = 0
  audioEl.value.play().catch(() => {})
}

function playIntroMusic() {
  if (musicStarted || !musicEl.value) return
  musicStarted = true
  musicEl.value.src = MUSIC_FILE
  musicEl.value.currentTime = 0
  musicEl.value.volume = 1
  musicEl.value.play().catch(() => {})
}

function stopIntroMusic(fadeMs = 800) {
  if (!musicEl.value || !musicStarted) return
  const el = musicEl.value
  const steps = 16
  const stepTime = fadeMs / steps
  let i = 0
  const startVol = el.volume
  const iv = setInterval(() => {
    i++
    el.volume = Math.max(0, startVol * (1 - i / steps))
    if (i >= steps) {
      clearInterval(iv)
      el.pause()
    }
  }, stepTime)
}

// Heartbeat (Phase 0, lanjut sampai Phase 1)
function playHeartbeat() {
  if (!sfxEl.value) return
  sfxEl.value.src = HEARTBEAT_FILE
  sfxEl.value.volume = 0.6
  sfxEl.value.play().catch(() => {})
}
// Radio signal (mulai di Phase 1, "Hey...")
function playRadio() {
  if (!radioEl.value) return
  radioEl.value.src = RADIO_FILE
  radioEl.value.volume = 0.5
  radioEl.value.play().catch(() => {})
}
function stopSfx() {
  sfxEl.value?.pause()
  radioEl.value?.pause()
}

// Sinyal "data synced" saat pin ke-3 selesai
function playSync() {
  if (!syncEl.value) return
  syncEl.value.src = SYNC_FILE
  syncEl.value.volume = 0.8
  syncEl.value.currentTime = 0
  syncEl.value.play().catch(() => {})
}

// Suara alam: fade in pelan (dikecilkan supaya tidak menabrak opening.mp3)
function fadeInNature(fadeMs = 2000, target = 0.3) {
  const el = natureEl.value
  if (!el) return
  el.src = NATURE_FILE
  el.volume = 0
  el.play().catch(() => {})
  const steps = 20
  let i = 0
  const iv = setInterval(() => {
    i++
    el.volume = Math.min(target, target * (i / steps))
    if (i >= steps) clearInterval(iv)
  }, fadeMs / steps)
}

// Lagu opening: fade in saat langit berubah biru (dunia hijau)
const OPENING_FILE = '/music/opening.mp3'
function fadeInOpening(fadeMs = 2000, target = 0.8) {
  const el = openingEl.value
  if (!el) return
  el.src = OPENING_FILE
  el.currentTime = 0
  el.volume = 0
  el.play().catch(() => {})
  const steps = 20
  let i = 0
  const iv = setInterval(() => {
    i++
    el.volume = Math.min(target, target * (i / steps))
    if (i >= steps) clearInterval(iv)
  }, fadeMs / steps)
}

const MAIN_FILE = '/music/main.mp3'
function fadeInMain(fadeMs = 2000, target = 0.65) {
  const el = mainEl.value
  if (!el) return
  el.src = MAIN_FILE
  el.currentTime = 0
  el.volume = 0
  el.play().catch(() => {})
  const steps = 20
  let i = 0
  const iv = setInterval(() => {
    i++
    el.volume = Math.min(target, target * (i / steps))
    if (i >= steps) clearInterval(iv)
  }, fadeMs / steps)
}

// --- loading sequence --------------------------------------------------
function startLoading() {
  const steps = [0, 25, 50, 100]
  let i = 0
  pct.value = steps[0]
  const iv = setInterval(() => {
    i++
    if (i < steps.length) pct.value = steps[i]
    if (i >= steps.length - 1) {
      clearInterval(iv)
      setTimeout(() => { phase.value = 'goGreen' }, 900)
    }
  }, 550)
}

function onStartClick() {
  initAudioGraph()
  phase.value = 'headphone'
  setTimeout(() => {
    if (cancelled) return
    phase.value = 'blackout' // Phase 0
    playHeartbeat()
    setTimeout(() => {
      if (cancelled) return
      wakeStage.value = 1
      phase.value = 'waking'
      beginWaking()
    }, BLACKOUT_MS)
  }, HEADPHONE_HOLD_MS)
}

// --- waking sequence (Phase 1 -> 2 -> 3) ---------------------------------
function beginWaking() {
  // Phase 1: "Hey..." (tanpa voice, heartbeat + radio signal jalan)
  playRadio()
  playFire() // suara api kebakar mulai fade-in bersamaan dengan mata mulai terbuka
  setTimeout(() => {           // Phase 2: "Hey, wake up!"
    if (cancelled) return
    wakeStage.value = 2
    playLineAudio(1)
    stopSfx()
  }, P1_MS)
  setTimeout(() => {           // Phase 3: mata buka sebentar -> merem total -> melek
    if (cancelled) return
    wakeStage.value = 3
    playLineAudio(2)
    playIntroMusic()
  }, P1_MS + P2_MS)
  setTimeout(() => {           // ganti people1 -> people2 SAAT mata tertutup total
    if (cancelled) return
    personAwake.value = true
  }, P1_MS + P2_MS + EYES_CLOSED_SWAP_MS)
  setTimeout(() => {           // lanjut narrative dari "Hmm...?"
    if (cancelled) return
    phase.value = 'narrative'
    beginNarrative(3)
  }, P1_MS + P2_MS + P3_MS)
}

// --- narrative sequence -------------------------------------------------
function beginNarrative(startIndex = 0) {
  narrativeIndex.value = startIndex
  showLine.value = false
  // tiny delay so the text element mounts hidden, then fades in
  setTimeout(() => {
    if (cancelled) return
    showLine.value = true
    playLineAudio(startIndex)
    if (startIndex === MUSIC_START_LINE_INDEX) playIntroMusic()
    advanceNarrative()
  }, 60)
}

function advanceNarrative() {
  const holdMs = 3400
  const gapMs = 700

  const step = () => {
    if (cancelled) return
    showLine.value = false
    setTimeout(() => {
      if (cancelled) return
      if (narrativeIndex.value < lines.value.length - 1) {
        narrativeIndex.value++
        showLine.value = true
        playLineAudio(narrativeIndex.value)
        if (narrativeIndex.value === MUSIC_START_LINE_INDEX) playIntroMusic()
        setTimeout(step, holdMs)
      } else {
        beginTransmission()
      }
    }, gapMs)
  }

  setTimeout(step, holdMs)
}

// --- transmission sequence (langit gelap + glitch) ----------------------
function beginTransmission() {
  showLine.value = false
  phase.value = 'transmission'
  txStage.value = 1
  setTimeout(() => { if (!cancelled) txStage.value = 2 }, TX1_MS)
  setTimeout(() => { if (!cancelled) txStage.value = 3 }, TX1_MS + TX2_MS)
  setTimeout(() => { if (!cancelled) txStage.value = 4 }, TX1_MS + TX2_MS + TX3_MS)
}

function onChoose(choice) {
  cancelled = true
  stopSfx()
  stopFire(1500) // scene buka mata berakhir
  if (choice === 'inspect') {
    phase.value = 'inspect' // musik lanjut, masuk ke layar 3 node
    lockScroll()
    return
  }
  if (choice === 'whatever') {
    startWhateverGlitch()
    return
  }
  phase.value = 'done'
  stopIntroMusic()
  emit('finished', choice) // 'whatever'
}

// --- transisi glitch sebelum video "whatever" -------------------------------
const GLITCH_OUT_MS = 1400
const glitchOut = ref(false)
let glitchTimer = null

function startWhateverGlitch() {
  glitchOut.value = true
  stopIntroMusic(GLITCH_OUT_MS)
  playRadio()
  glitchTimer = setTimeout(() => {
    stopSfx()
    glitchOut.value = false
    phase.value = 'whatever'
  }, GLITCH_OUT_MS)
}

// --- reality check -> pilihan ---------------------------------------------
function onWhateverVideoEnded() {
  playEnding() // lagu mulai di layar hitam, sebelum teks pertama muncul
  phase.value = 'realityCheck'
  realityLine.value = -1
  realityHidden.value = false
  realityChoices.value = false
  realityTimer = setTimeout(() => showRealityMessage(0), 1500)
}

function showRealityMessage(index) {
  realityLine.value = index
  if (index < realityMessages.value.length - 1) {
    realityTimer = setTimeout(() => showRealityMessage(index + 1), REALITY_LINE_MS)
    return
  }
  // teks terakhir ("Still think it's 'whatever'?") -> fade out -> jeda -> tombol
  realityTimer = setTimeout(() => {
    realityHidden.value = true
    realityTimer = setTimeout(() => {
      realityChoices.value = true
    }, REALITY_FADE_MS + REALITY_PAUSE_MS)
  }, REALITY_LINE_MS)
}

// Kanan: tobat -> masuk ke layar inspect 3 node (jalur "good")
function onFixThis() {
  clearTimeout(realityTimer)
  realityChoices.value = false
  stopEnding(1500) // lagu ending memudar, masuk ke jalur "fix"
  phase.value = 'inspect'
  lockScroll()
}

// Kiri: tetap cuek -> bad ending
function onStillWhatever() {
  clearTimeout(realityTimer)
  realityChoices.value = false
  startBadInit()
}

// --- bad ending ------------------------------------------------------------
const badPct = ref(0)
const endStage = ref(0) // 1 warning | 2 +teks | 3 semua hilang | 4 thanks | 5 edukasi | 6 +Contact Us
const badTimers = []
let badIv = null

function badLater(fn, ms) {
  badTimers.push(setTimeout(fn, ms))
}

function startBadInit() {
  phase.value = 'badInit'
  badPct.value = 0
  endStage.value = 0
  playRadio() // static radio selama timeline "ditulis ulang"

  const tick = BAD_INIT_FILL_MS / 100
  badIv = setInterval(() => {
    if (badPct.value >= 100) {
      clearInterval(badIv)
      return
    }
    badPct.value++
  }, tick)

  badLater(() => {
    clearInterval(badIv)
    badPct.value = 100
    startBadEnding()
  }, BAD_INIT_MS)
}

function startBadEnding() {
  stopSfx()
  phase.value = 'badEnding'
  endStage.value = 0
  badLater(() => { endStage.value = 1 }, BE_WARNING_AT)
  badLater(() => { endStage.value = 2 }, BE_SUB_AT)
  badLater(() => { endStage.value = 3 }, BE_CLEAR_AT)
  badLater(() => { endStage.value = 4 }, BE_THANKS_AT)
  badLater(() => { endStage.value = 5 }, BE_EDU_AT)
  badLater(() => { endStage.value = 6 }, BE_CONTACT_AT)
}

function onContact() {
  contactOpen.value = true
  emit('contact')
}

async function copyEmail(m) {
  try {
    await navigator.clipboard.writeText(m.email)
  } catch {
    // fallback untuk browser / konteks non-HTTPS
    const t = document.createElement('textarea')
    t.value = m.email
    t.style.position = 'fixed'
    t.style.opacity = '0'
    document.body.appendChild(t)
    t.select()
    try { document.execCommand('copy') } catch {}
    t.remove()
  }
  copiedId.value = m.id
  clearTimeout(copiedTimer)
  copiedTimer = setTimeout(() => { copiedId.value = null }, 2000)
}

// --- Start over: reset semuanya, balik ke layar "Go Green / Click to start" --
// (sengaja tidak balik ke loading; klik "Click to start" juga dibutuhkan
//  supaya browser mengizinkan audio diputar lagi)
function restart() {
  // hentikan semua timer & listener
  cancelled = true
  clearTimeout(glitchTimer)
  glitchOut.value = false
  clearGameTimers()
  clearTrashTimers()
  clearDroneTimers()
  happyTimers.forEach(clearTimeout); happyTimers.length = 0
  clearTimeout(realityTimer)
  clearInterval(badIv)
  clearInterval(endingIv)
  clearInterval(fireIv)
  clearTimeout(copiedTimer)
  badTimers.forEach(clearTimeout); badTimers.length = 0
  inspectTimers.forEach(clearTimeout); inspectTimers.length = 0
  detachScrollListeners()
  releaseScrollLock()
  stopLook()

  // hentikan semua audio
  stopSfx()
  for (const el of [audioEl, musicEl, syncEl, natureEl, endingEl, fireEl, openingEl, mainEl]) {
    if (el.value) { el.value.pause(); el.value.currentTime = 0 }
  }
  musicStarted = false

  // reset state
  personAwake.value = false
  wakeStage.value = 1
  txStage.value = 0
  narrativeIndex.value = -1
  showLine.value = false
  realityLine.value = -1
  realityHidden.value = false
  realityChoices.value = false
  badPct.value = 0
  endStage.value = 0
  contactOpen.value = false
  copiedId.value = null
  seen.value = []
  activeId.value = null
  completionStarted.value = false
  flash.value = false
  pinsHidden.value = false
  povShow.value = false
  txShow.value = false
  unlocked.value = false
  shifted.value = false
  waking.value = false
  wakeText.value = false
  wakeMessage.value = ''
  wakeMessageIndex.value = -1
  nicknamePrompt.value = false
  nicknameInput.value = ''
  nickname.value = ''
  ecoIntro.value = false
  gameState.value = 'idle'
  paused.value = false
  bubbles.value = []
  pops.value = []
  cleanAir.value = 0
  timeLeft.value = GAME_SECONDS
  winFlash.value = false
  gameHint.value = false
  callOpen.value = false
  callAnswered.value = false
  callMessageIndex.value = 0
  micMuted.value = false
  cameraOff.value = false
  stageBanner.value = false
  trashState.value = 'idle'
  trashItems.value = []
  trashCorrect.value = 0
  selectedId.value = null
  trashTxOpen.value = false
  stageBanner.value = false
  bannerLine1.value = 'stage1Done'
  bannerLine2.value = 'airPurified'
  droneState.value = 'idle'
  dronePlots.value = []
  droneTime.value = DRONE_SECONDS
  droneGrown.value = 0
  droneHint.value = false
  droneTxOpen.value = false
  droneWinFlash.value = false
  allDoneBanner.value = false
  happyStage.value = 0
  happyContactOpen.value = false

  cancelled = false
  phase.value = 'goGreen'
}

// --- inspect: 3 warning nodes ---------------------------------------------
// Taruh gambar background-nya di /public/picture/ (ganti nama kalau beda)
const INSPECT_BG = '/picture/inspect.avif'

// x / y = posisi node (% layar) | stem = panjang garis ke bawah (vh)
const NODE_BASE = [
  {
    id: 'heat',
    x: 33, y: 20, stem: 16,
    icon: 'M14 14.76V3.5a2.5 2.5 0 0 0-5 0v11.26a4.5 4.5 0 1 0 5 0z'
  },
  {
    id: 'air',
    x: 54, y: 36, stem: 14,
    icon: 'M9.59 4.59A2 2 0 1 1 11 8H2m10.59 11.41A2 2 0 1 0 14 16H2m15.73-8.27A2.5 2.5 0 1 1 19.5 12H2'
  },
  {
    id: 'eco',
    x: 78, y: 62, stem: 16,
    icon: 'M2 6c.6.5 1.2 1 2.5 1C7 7 7 5 9.5 5c2.6 0 2.4 2 5 2 2.5 0 2.5-2 5-2 1.3 0 1.9.5 2.5 1M2 12c.6.5 1.2 1 2.5 1 2.5 0 2.5-2 5-2 2.6 0 2.4 2 5 2 2.5 0 2.5-2 5-2 1.3 0 1.9.5 2.5 1M2 18c.6.5 1.2 1 2.5 1 2.5 0 2.5-2 5-2 2.6 0 2.4 2 5 2 2.5 0 2.5-2 5-2 1.3 0 1.9.5 2.5 1'
  }
]
const nodes = computed(() => NODE_BASE.map((node) => {
  const data = I18N[lang.value].nodes[node.id]
  return {
    ...node,
    label: data.label,
    title: data.title,
    msg: data.msg,
    now: { value: data.nowV, note: data.nowN },
    future: { value: data.futV, note: data.futN }
  }
}))

const seen = ref([])        // id node yang sudah dibuka
const activeId = ref(null)  // node yang kartunya sedang tampil
const activeNode = computed(() => nodes.value.find(n => n.id === activeId.value) || null)

// --- inspect: state urutan "completion" -----------------------------------
const completionStarted = ref(false)
const flash = ref(false)      // flash glitch merah-oranye 0.3 detik
const pinsHidden = ref(false) // pin + hint memudar
const povShow = ref(false)    // dialog POV (bawah tengah)
const txShow = ref(false)     // pesan 2076 (kanan atas)
const unlocked = ref(false)   // scroll indicator + scroll dibuka
const shifted = ref(false)    // red-to-green world shift

// --- tema kursor per fase ---------------------------------------------------
// off   : loading / go green / headphone / video "whatever" (kontrol video butuh kursor asli)
// red   : blackout -> inspect, reality check, bad ending
// green : mulai scroll ke dunia hijau
watch(
  [phase, shifted],
  () => {
    const p = phase.value
    let t = 'off'
    if (['blackout', 'waking', 'narrative', 'transmission', 'inspect',
         'realityCheck', 'badInit', 'badEnding', 'done'].includes(p)) t = 'red'
    if (p === 'inspect' && shifted.value) t = 'green'
    setCursorTheme(t)
  },
  { immediate: true }
)

const inspectTimers = []
function later(fn, ms) {
  const id = setTimeout(() => { if (!cancelled || phase.value === 'inspect') fn() }, ms)
  inspectTimers.push(id)
}

function openNode(id) {
  if (completionStarted.value) return
  activeId.value = id
  if (!seen.value.includes(id)) seen.value.push(id)
}

function closeNode() {
  activeId.value = null
  // pin ke-3 selesai dibaca & ditutup -> trigger completion
  if (seen.value.length === 3 && !completionStarted.value) startCompletion()
}

// Timeline (ms dari saat pop-up pin ke-3 ditutup):
//   0     flash glitch 0.3s + suara sinyal
//   300   pin mulai memudar (1.4s)
//   1900  POV muncul (fade-in + slide-up)
//   5400  POV hilang
//   6900  (jeda 1.5s) pesan 2076 muncul, neon berkedip
//   10400 scroll dibuka + teaser hijau muncul
function startCompletion() {
  completionStarted.value = true

  flash.value = true
  playSync()
  later(() => { flash.value = false }, 300)

  later(() => { pinsHidden.value = true }, 300)

  later(() => { povShow.value = true }, 1900)
  later(() => { povShow.value = false }, 5400)

  later(() => { txShow.value = true }, 6900)

  later(unlockScroll, 10400)
}

// --- scroll lock / unlock -------------------------------------------------
let touchY0 = null

function lockScroll() {
  document.documentElement.style.overflow = 'hidden'
  document.body.style.overflow = 'hidden'
}
function releaseScrollLock() {
  document.documentElement.style.overflow = ''
  document.body.style.overflow = ''
}

function unlockScroll() {
  unlocked.value = true
  releaseScrollLock() // kunci scroll browser dilepas
  window.addEventListener('wheel', onWheel, { passive: true })
  window.addEventListener('touchstart', onTouchStart, { passive: true })
  window.addEventListener('touchmove', onTouchMove, { passive: true })
  window.addEventListener('keydown', onKey)
}
function detachScrollListeners() {
  window.removeEventListener('wheel', onWheel)
  window.removeEventListener('touchstart', onTouchStart)
  window.removeEventListener('touchmove', onTouchMove)
  window.removeEventListener('keydown', onKey)
}

function onWheel(e) { if (e.deltaY > 0) startShift() }
function onTouchStart(e) { touchY0 = e.touches[0]?.clientY ?? null }
function onTouchMove(e) {
  if (touchY0 == null) return
  const y = e.touches[0]?.clientY ?? touchY0
  if (touchY0 - y > 24) startShift() // geser ke bawah (jari naik)
}
function onKey(e) {
  if (['ArrowDown', 'PageDown', ' ', 'Spacebar', 'End'].includes(e.key)) {
    e.preventDefault()
    startShift()
  }
}

// --- Look around: POV ikut kursor (dunia hijau) -----------------------------
// tx/ty = target (-1..1) dari posisi kursor, cx/cy = nilai yang dihaluskan (lerp)
const greenEl = ref(null)
let lookRaf = null
let tx = 0, ty = 0, cx = 0, cy = 0, tk = 0, ck = 0

function onLookMove(e) {
  tx = (e.clientX / window.innerWidth - 0.5) * 2
  ty = (e.clientY / window.innerHeight - 0.5) * 2
}
function onLookTouch(e) {
  const t = e.touches[0]
  if (t) onLookMove(t)
}
function lookLoop() {
  cx += (tx - cx) * 0.05
  cy += (ty - cy) * 0.05
  ck += (tk - ck) * 0.04
  const el = greenEl.value
  if (el) {
    el.style.setProperty('--lx', cx.toFixed(4))
    el.style.setProperty('--ly', cy.toFixed(4))
    el.style.setProperty('--lk', ck.toFixed(4))
  }
  lookRaf = requestAnimationFrame(lookLoop)
}
function startLook() {
  if (lookRaf) return
  tk = 1
  window.addEventListener('mousemove', onLookMove, { passive: true })
  window.addEventListener('touchmove', onLookTouch, { passive: true })
  lookRaf = requestAnimationFrame(lookLoop)
}
function stopLook() {
  if (lookRaf) cancelAnimationFrame(lookRaf)
  lookRaf = null
  window.removeEventListener('mousemove', onLookMove)
  window.removeEventListener('touchmove', onLookTouch)
  tx = ty = cx = cy = tk = ck = 0
  const el = greenEl.value
  if (el) {
    el.style.removeProperty('--lx')
    el.style.removeProperty('--ly')
    el.style.removeProperty('--lk')
  }
}

// --- Red-to-Green World Shift ---------------------------------------------
// Urutan (11 detik, sama polanya dengan intro):
//   0-1.5s   mata tertutup total (detak jantung)
//   1.5-4.8s celah tipis -> merem lagi          (kayak Phase 1)
//   5.5-8.6s kedip 2x lebih lebar -> tetap buka (kayak Phase 2)
//   8.6-11s  terbuka penuh, blur hilang, POV berdiri tegak
// WAKE_MS harus sama dengan durasi animasi CSS wakeEye dan wakeTilt (11s).
const WAKE_MS = 11000
const SHIFT_HOLD_MS = 1600

const WAKE_LINE = 'Huh...? Where am I?'
const waking = ref(false)
const wakeText = ref(false)
const wakeMessage = ref('')
const wakeMessageIndex = ref(-1)
const ecoIntro = ref(false)
const nicknamePrompt = ref(false)
const nicknameInput = ref('')
const nickname = ref('')
const wakeMessages = computed(() => [
  t('wakeLine'), t('hello', { name: nickname.value }), t('welcome'), t('wake4')
])
const WAKE_MESSAGE_DURATION_MS = 2600
const WAKE_MESSAGE_GAP_MS = 500

function startShift() {
  if (shifted.value || !unlocked.value) return
  shifted.value = true   // tampilkan dunia hijau sebelum animasi bangun
  nicknamePrompt.value = true
  detachScrollListeners()

  povShow.value = false
  txShow.value = false
  stopIntroMusic(2500)
}

function beginWakeSequence() {
  const enteredNickname = nicknameInput.value.trim()
  if (!enteredNickname || !nicknamePrompt.value) return

  nickname.value = enteredNickname
  nicknamePrompt.value = false
  waking.value = true

  // audio: detak jantung selama mata masih menutup
  playHeartbeat()

  // detak jantung berhenti setelah celah tipis pertama
  later(() => { stopSfx() }, 4800)

  // suara alam masuk saat mata hampir terbuka penuh
  later(() => {
    fadeInNature(2500)
    fadeInOpening(2500)
    fadeInMain(2500)
  }, 8600)

  // selesai: lepas mask/filter (tampilan sudah identik, jadi tidak loncat)
  later(() => { waking.value = false }, WAKE_MS + 50)
  // setelah bangun selesai, POV boleh nengok ikut kursor
  later(startLook, WAKE_MS + 50)
  // kalimat pertama muncul saat mata mulai terbuka; pesan berikutnya menyusul
  later(() => showWakeMessage(0), 8600)
}

function showWakeMessage(index) {
  if (index >= wakeMessages.value.length) {
    later(() => { ecoIntro.value = true }, WAKE_MESSAGE_GAP_MS)
    return
  }

  wakeText.value = false
  later(() => {
    wakeMessageIndex.value = index
    wakeMessage.value = wakeMessages.value[index]
    wakeText.value = true
    later(() => {
      wakeText.value = false
      later(() => showWakeMessage(index + 1), WAKE_MESSAGE_GAP_MS)
    }, WAKE_MESSAGE_DURATION_MS)
  }, index === 0 ? 0 : WAKE_MESSAGE_GAP_MS)
}

function onStartGame() {
  ecoIntro.value = false
  startGame()
}

// --- ASAP KEBAKARAN + POLUSI + BARA ---------------------------------------
// Random yang "deterministik" (seed tetap) supaya posisi asap konsisten tiap render.
function seeded(seed) {
  let s = seed
  return () => {
    s = (s * 9301 + 49297) % 233280
    return s / 233280
  }
}

// --- dunia hijau: background gambar (taruh di /public/picture/) ------------
const GREEN_BG = '/picture/city3.avif'

// --- dunia hijau: sinar matahari -------------------------------------------
// Pusat matahari di gambar city3.avif kira-kira x=80%, y=4% (lihat .gw-rays di CSS).
// Berkas melebar ke bawah, kipas ke kiri-bawah & sedikit ke kanan-bawah.
const rays = (() => {
  const r = seeded(11)
  const out = []
  const N = 18
  for (let i = 0; i < N; i++) {
    const a = -28 + (i / (N - 1)) * 110 + (r() - 0.5) * 4
    out.push({
      id: i,
      style: {
        '--a': `${a}deg`,
        '--w': `${10 + r() * 22}vmin`,
        '--o': (0.3 + r() * 0.5).toFixed(2),
        '--d': `${5 + r() * 6}s`,
        '--dl': `${-r() * 8}s`
      }
    })
  }
  return out
})()

// --- dunia hijau: kilau di permukaan sungai --------------------------------
// Posisi acak di kotak sungai; yang jatuh di luar sungai otomatis kepotong clip-path.
const glints = (() => {
  const r = seeded(31)
  const out = []
  for (let i = 0; i < 38; i++) {
    const size = 3 + r() * 5
    const dur = 2 + r() * 3.5
    out.push({
      id: i,
      style: {
        left: `${28 + r() * 46}%`,
        top: `${71 + r() * 22}%`,
        width: `${size}px`,
        height: `${size}px`,
        animationDuration: `${dur}s`,
        animationDelay: `${-r() * dur}s`
      }
    })
  }
  return out
})()

// --- dunia hijau: daun kecil jatuh + serbuk cahaya -------------------------
const fallLeaves = (() => {
  const r = seeded(47)
  const out = []
  for (let i = 0; i < 12; i++) {
    const size = 9 + r() * 12
    const dur = 16 + r() * 14
    out.push({
      id: i,
      style: {
        left: `${r() * 100}%`,
        top: '-4%',
        width: `${size}px`,
        height: `${size * 0.55}px`,
        '--dx': `${-8 + r() * 22}vw`,
        '--hue': `${85 + Math.floor(r() * 45)}`,
        animationDuration: `${dur}s`,
        animationDelay: `${-r() * dur}s`
      }
    })
  }
  return out
})()

const motes = (() => {
  const r = seeded(59)
  const out = []
  for (let i = 0; i < 28; i++) {
    const size = 2 + r() * 4
    const dur = 7 + r() * 9
    out.push({
      id: i,
      style: {
        left: `${r() * 100}%`,
        top: `${30 + r() * 65}%`,
        width: `${size}px`,
        height: `${size}px`,
        '--mx': `${(r() - 0.4) * 6}vw`,
        '--my': `${-(4 + r() * 10)}vh`,
        animationDuration: `${dur}s`,
        animationDelay: `${-r() * dur}s`
      }
    })
  }
  return out
})()

// --- dunia hijau: daun foreground (blur, di depan segalanya) ---------------
// Empat gerombol daun di pojok layar. Koordinat = % dari panggung (gambar).
//   ax/ay   : titik pangkal gerombol (pojok)     n     : jumlah daun
//   rot     : rentang arah daun (deg, 0 = kanan, -90 = atas, 90 = bawah)
//   ox/oy   : sebaran pangkal daun dari pojok    w     : lebar daun (% lebar gambar)
//   blur    : rentang blur (px)                  wind  : besar goyangan gerombol (deg)
const fgClusters = (() => {
  const r = seeded(71)
  const rnd = (a, b) => a + r() * (b - a)
  const defs = [
    { id: 'bl', ax: 0,   ay: 100, n: 10, rot: [-88, -4],  ox: [-2, 9],  oy: [-14, 3], w: [15, 28], blur: [7, 13],
      hue: [118, 155], sat: [45, 60], lit: [22, 34], sw: [2, 5], wind: 1.3, flowers: 3 },
    { id: 'br', ax: 100, ay: 100, n: 9,  rot: [184, 262], ox: [-9, 2],  oy: [-12, 3], w: [14, 26], blur: [7, 13],
      hue: [118, 155], sat: [45, 60], lit: [22, 34], sw: [2, 5], wind: 1.1, flowers: 2 },
    { id: 'tl', ax: 0,   ay: 0,   n: 9,  rot: [8, 82],    ox: [0, 24],  oy: [-1, 7],  w: [9, 17],  blur: [3, 7],
      hue: [72, 105],  sat: [60, 75], lit: [46, 56], sw: [3, 7], wind: 1.6, flowers: 0, branch: 'tl' },
    { id: 'tr', ax: 100, ay: 0,   n: 9,  rot: [98, 172],  ox: [-20, 0], oy: [-1, 9],  w: [9, 17],  blur: [3, 7],
      hue: [78, 108],  sat: [60, 75], lit: [46, 56], sw: [3, 7], wind: 1.6, flowers: 0, branch: 'tr' }
  ]
  return defs.map((d) => {
    const leaves = []
    for (let i = 0; i < d.n; i++) {
      const h = rnd(d.hue[0], d.hue[1])
      const sa = rnd(d.sat[0], d.sat[1])
      const li = rnd(d.lit[0], d.lit[1])
      const dur = rnd(3.5, 7)
      leaves.push({
        id: i,
        style: {
          left: `${d.ax + rnd(d.ox[0], d.ox[1])}%`,
          top: `${d.ay + rnd(d.oy[0], d.oy[1])}%`,
          width: `${rnd(d.w[0], d.w[1])}%`,
          '--rot': `${rnd(d.rot[0], d.rot[1]).toFixed(1)}deg`,
          '--sw': `${rnd(d.sw[0], d.sw[1]).toFixed(1)}deg`,
          '--b': `${rnd(d.blur[0], d.blur[1]).toFixed(1)}px`,
          '--c1': `hsl(${h | 0} ${sa | 0}% ${li | 0}%)`,
          '--c2': `hsl(${(h + 8) | 0} ${sa | 0}% ${(li + 14) | 0}%)`,
          '--c3': `hsl(${h | 0} ${sa | 0}% ${(li - 12) | 0}%)`,
          animationDuration: `${dur.toFixed(2)}s`,
          animationDelay: `${(-r() * dur).toFixed(2)}s`
        }
      })
    }
    const flowers = []
    for (let i = 0; i < d.flowers; i++) {
      const dur = rnd(4, 7)
      flowers.push({
        id: i,
        style: {
          left: `${d.ax === 0 ? rnd(2, 14) : rnd(86, 98)}%`,
          top: `${rnd(84, 96)}%`,
          width: `${rnd(4, 6.5)}%`,
          '--b': `${rnd(3, 6).toFixed(1)}px`,
          '--sw': `${rnd(4, 8).toFixed(1)}deg`,
          animationDuration: `${dur.toFixed(2)}s`,
          animationDelay: `${(-r() * dur).toFixed(2)}s`
        }
      })
    }
    return {
      id: d.id,
      branch: d.branch || null,
      leaves,
      flowers,
      style: {
        transformOrigin: `${d.ax}% ${d.ay}%`,
        '--wa': `${d.wind}deg`,
        animationDuration: `${rnd(7, 11).toFixed(2)}s`,
        animationDelay: `${(-rnd(0, 8)).toFixed(2)}s`
      }
    }
  })
})()

// --- dunia hijau: burung terbang melintas kiri -> kanan ---------------------
const birds = (() => {
  const r = seeded(23)
  const out = []
  for (let i = 0; i < 9; i++) {
    const dur = 22 + r() * 16
    out.push({
      id: i,
      style: {
        top: `${6 + r() * 38}%`,
        width: `${18 + r() * 26}px`,
        opacity: (0.55 + r() * 0.4).toFixed(2),
        '--dy': `${(r() - 0.5) * 8}vh`,
        '--flap': `${0.5 + r() * 0.35}s`,
        animationDuration: `${dur}s`,
        animationDelay: `${-r() * dur}s`
      }
    })
  }
  return out
})()

// ========== A) SCENE (di belakang karakter, layar city1 + people) =========
// Titik sumber api untuk scene. x = % lebar, y = % dari atas.
// Horizon kota kira-kira di 55-80% tinggi layar; sesuaikan kalau mau geser.
const SCENE_FIRE_SOURCES = [
  { x: 8,  y: 82 },
  { x: 24, y: 74 },
  { x: 40, y: 68 },
  { x: 58, y: 70 },
  { x: 76, y: 75 },
  { x: 92, y: 80 }
]

// Kolom asap naik. Jumlah/ukuran bisa diubah lewat parameter.
function buildPlumes(sources, seedNum, perSource) {
  const r = seeded(seedNum)
  const out = []
  let id = 0
  sources.forEach((src) => {
    for (let i = 0; i < perSource; i++) {
      const size = 160 + r() * 220              // px
      const dur = 9 + r() * 9                   // detik
      out.push({
        id: id++,
        dark: r() > 0.45,
        style: {
          left: `calc(${src.x + (r() - 0.5) * 6}% - ${size / 2}px)`,
          top: `calc(${src.y}% - ${size / 2}px)`,
          width: `${size}px`,
          height: `${size}px`,
          '--dx': `${(r() - 0.3) * 18}vw`,      // hanyut ke kanan (angin)
          '--rise': `${-(38 + r() * 34)}vh`,    // naik setinggi ini
          '--grow': (2 + r() * 1.6).toFixed(2), // membesar
          '--peak': (0.38 + r() * 0.3).toFixed(2),
          animationDuration: `${dur}s`,
          animationDelay: `${-r() * dur}s`
        }
      })
    }
  })
  return out
}

function buildEmbers(sources, seedNum, count) {
  const r = seeded(seedNum)
  const out = []
  for (let i = 0; i < count; i++) {
    const src = sources[Math.floor(r() * sources.length)]
    const size = 2 + r() * 3.5
    const dur = 5 + r() * 7
    out.push({
      id: i,
      style: {
        left: `${src.x + (r() - 0.5) * 12}%`,
        top: `${src.y + (r() - 0.5) * 6}%`,
        width: `${size}px`,
        height: `${size}px`,
        '--ex': `${(r() - 0.35) * 22}vw`,
        '--ey': `${-(28 + r() * 48)}vh`,
        animationDuration: `${dur}s`,
        animationDelay: `${-r() * dur}s`
      }
    })
  }
  return out
}

function buildWisps(seedNum, count) {
  const r = seeded(seedNum)
  const out = []
  for (let i = 0; i < count; i++) {
    const w = 45 + r() * 45 // vw
    const dur = 40 + r() * 40
    out.push({
      id: i,
      style: {
        top: `${8 + r() * 78}%`,
        width: `${w}vw`,
        height: `${16 + r() * 22}vh`,
        opacity: (0.18 + r() * 0.2).toFixed(2),
        animationDuration: `${dur}s`,
        animationDelay: `${-r() * dur}s`
      }
    })
  }
  return out
}

const scenePlumes = buildPlumes(SCENE_FIRE_SOURCES, 33, 5)
const sceneEmbers = buildEmbers(SCENE_FIRE_SOURCES, 77, 42)
const sceneWisps = buildWisps(58, 5)

// ========== B) INSPECT (layar 3 node) ====================================
// Titik sumber api (x = % layar, y = % dari atas layar).
// Sesuaikan dengan gambar inspect.avif kalau mau geser sumber apinya.
const FIRE_SOURCES = [
  { x: 14, y: 78 },  // reruntuhan kiri bawah
  { x: 30, y: 66 },  // bawah jembatan
  { x: 47, y: 62 },  // asap tebal tengah (sudah ada di gambar)
  { x: 62, y: 72 },  // sungai tengah
  { x: 82, y: 70 },  // tumpukan sampah kanan
  { x: 92, y: 58 }   // pagar kanan
]

// Kolom asap yang naik dari tiap sumber api
const plumes = (() => {
  const r = seeded(7)
  const out = []
  let id = 0
  FIRE_SOURCES.forEach((src) => {
    const count = 4
    for (let i = 0; i < count; i++) {
      const size = 160 + r() * 220              // px
      const dur = 9 + r() * 9                   // detik
      out.push({
        id: id++,
        dark: r() > 0.45,
        style: {
          left: `calc(${src.x + (r() - 0.5) * 6}% - ${size / 2}px)`,
          top: `calc(${src.y}% - ${size / 2}px)`,
          width: `${size}px`,
          height: `${size}px`,
          '--dx': `${(r() - 0.3) * 18}vw`,      // hanyut ke kanan (angin)
          '--rise': `${-(38 + r() * 34)}vh`,    // naik setinggi ini
          '--grow': (2 + r() * 1.6).toFixed(2), // membesar
          '--peak': (0.38 + r() * 0.3).toFixed(2),
          animationDuration: `${dur}s`,
          animationDelay: `${-r() * dur}s`
        }
      })
    }
  })
  return out
})()

// Asap tipis di depan, melintas horizontal pelan (kesan polusi bergerak)
const wisps = (() => {
  const r = seeded(21)
  const out = []
  for (let i = 0; i < 7; i++) {
    const w = 45 + r() * 45 // vw
    const dur = 40 + r() * 40
    out.push({
      id: i,
      style: {
        top: `${8 + r() * 78}%`,
        width: `${w}vw`,
        height: `${16 + r() * 22}vh`,
        opacity: (0.18 + r() * 0.2).toFixed(2),
        animationDuration: `${dur}s`,
        animationDelay: `${-r() * dur}s`
      }
    })
  }
  return out
})()

// Bara api yang beterbangan naik
const embers = (() => {
  const r = seeded(99)
  const out = []
  for (let i = 0; i < 46; i++) {
    const src = FIRE_SOURCES[Math.floor(r() * FIRE_SOURCES.length)]
    const size = 2 + r() * 3.5
    const dur = 5 + r() * 7
    out.push({
      id: i,
      style: {
        left: `${src.x + (r() - 0.5) * 12}%`,
        top: `${src.y + (r() - 0.5) * 6}%`,
        width: `${size}px`,
        height: `${size}px`,
        '--ex': `${(r() - 0.35) * 22}vw`,
        '--ey': `${-(28 + r() * 48)}vh`,
        animationDuration: `${dur}s`,
        animationDelay: `${-r() * dur}s`
      }
    })
  }
  return out
})()

// ========== C) BAD ENDING (city2.avif) ====================================
// Sumber api untuk layar bad ending. Geser x/y kalau posisi apinya kurang pas
// dengan gambar city2.avif.
const BAD_FIRE_SOURCES = [
  { x: 6,  y: 86 },
  { x: 22, y: 78 },
  { x: 38, y: 72 },
  { x: 54, y: 76 },
  { x: 70, y: 70 },
  { x: 86, y: 80 },
  { x: 96, y: 74 }
]
const badPlumes = buildPlumes(BAD_FIRE_SOURCES, 41, 5)
const badEmbers = buildEmbers(BAD_FIRE_SOURCES, 63, 56)
const badWisps = buildWisps(19, 7)

// ========== ECO-PULSE MINI GAME ==========
const GAME_SECONDS = 15
const WIN_MIN_CLEAN = 70
const TICK_MS = 100
const MAX_BUBBLES = 7
const LEAF_CHANCE = 0.14
const CALL_MESSAGES = computed(() => [t('call1'), t('call2'), t('call3')])
const CALL_MESSAGE_MS = 3800
const CLOUD = [[32, 58, 22], [68, 58, 22], [50, 40, 24], [50, 64, 22]]

const GAME_FIRE_SOURCES = [
  { x: 10, y: 88 }, { x: 30, y: 82 }, { x: 52, y: 86 }, { x: 72, y: 80 }, { x: 92, y: 86 }
]
const gamePlumes = buildPlumes(GAME_FIRE_SOURCES, 88, 3)
const gameWisps = buildWisps(29, 5)

const gameState = ref('idle')
const paused = ref(false)
const timeLeft = ref(GAME_SECONDS)
const cleanAir = ref(0)
const bubbles = ref([])
const pops = ref([])
const winFlash = ref(false)
const gameHint = ref(false)
const callOpen = ref(false)
const callAnswered = ref(false)
const callMessageIndex = ref(0)
const micMuted = ref(false)
const cameraOff = ref(false)

let gameIv = null
let gameStartTimer = null
let timeMs = 0
let spawnAcc = 0
let nextSpawn = 0
let uid = 0
const gameTimers = []

const gameActive = computed(() => gameState.value !== 'idle')
const smogOpacity = computed(() => 1 - cleanAir.value / 100)
const worldStyle = computed(() => {
  if (!gameActive.value) return null
  const cleanliness = cleanAir.value / 100
  return {
    filter: `saturate(${(0.5 + 0.5 * cleanliness).toFixed(2)}) brightness(${(0.7 + 0.4 * cleanliness).toFixed(2)}) sepia(${((1 - cleanliness) * 0.3).toFixed(2)})`,
    transition: 'opacity 2s ease, filter 0.7s ease'
  }
})

function clearGameTimers() {
  clearTimeout(gameStartTimer)
  clearInterval(gameIv)
  gameTimers.forEach(clearTimeout)
  gameTimers.length = 0
}

function startGame() {
  clearGameTimers()
  bubbles.value = []
  pops.value = []
  cleanAir.value = 0
  timeLeft.value = GAME_SECONDS
  timeMs = GAME_SECONDS * 1000
  spawnAcc = 0
  nextSpawn = 300
  paused.value = false
  winFlash.value = false
  callOpen.value = false
  callAnswered.value = false
  callMessageIndex.value = 0
  micMuted.value = false
  cameraOff.value = false
  gameHint.value = true
  gameState.value = 'starting'
  gameStartTimer = setTimeout(() => {
    gameState.value = 'playing'
    gameIv = setInterval(gameTick, TICK_MS)
    gameTimers.push(setTimeout(() => { gameHint.value = false }, 4500))
  }, 1800)
}

function gameTick() {
  if (paused.value || gameState.value !== 'playing') return
  timeMs -= TICK_MS
  timeLeft.value = Math.max(0, Math.ceil(timeMs / 1000))

  spawnAcc += TICK_MS
  if (spawnAcc >= nextSpawn) {
    spawnAcc = 0
    nextSpawn = 450 + Math.random() * 450
    spawnBubble()
  }
  bubbles.value.forEach((bubble) => { bubble.life -= TICK_MS })
  bubbles.value = bubbles.value.filter((bubble) => bubble.life > 0)

  if (timeMs <= 0) {
    if (cleanAir.value >= WIN_MIN_CLEAN) winGame()
    else loseGame()
  }
}

function spawnBubble() {
  if (bubbles.value.length >= MAX_BUBBLES) return
  const leaf = Math.random() < LEAF_CHANCE
  const size = leaf ? 13 : 11 + Math.random() * 6
  const x = 8 + Math.random() * 84
  const y = 22 + Math.random() * 56
  bubbles.value.push({
    id: ++uid,
    leaf,
    x,
    y,
    life: 2800 + Math.random() * 1400,
    style: {
      left: `${x}%`,
      top: `${y}%`,
      '--s': `${size}vmin`,
      '--fx': `${(Math.random() - 0.5) * 3}vmin`,
      '--fy': `${(Math.random() - 0.5) * 3}vmin`,
      '--fd': `${(1.6 + Math.random() * 1.4).toFixed(2)}s`
    }
  })
}

function hit(bubble) {
  if (gameState.value !== 'playing' || paused.value) return
  const gain = bubble.leaf ? 20 : 10
  bubbles.value = bubbles.value.filter((item) => item.id !== bubble.id)

  const id = ++uid
  pops.value.push({ id, x: bubble.x, y: bubble.y, gain, leaf: bubble.leaf })
  gameTimers.push(setTimeout(() => {
    pops.value = pops.value.filter((pop) => pop.id !== id)
  }, 900))

  if (bubble.leaf) playSync()
  cleanAir.value = Math.min(100, cleanAir.value + gain)
  if (cleanAir.value >= 100) winGame()
}

function togglePause() {
  if (gameState.value !== 'playing') return
  paused.value = !paused.value
}

function loseGame() {
  clearGameTimers()
  bubbles.value = []
  gameState.value = 'lost'
}

function winGame() {
  clearGameTimers()
  bubbles.value = []
  cleanAir.value = 100
  gameState.value = 'won'
  winFlash.value = true
  gameTimers.push(setTimeout(() => { winFlash.value = false }, 1600))
  gameTimers.push(setTimeout(() => {
    callOpen.value = true
    playRadio()
  }, 3200))
}

function answerCall() {
  stopSfx()
  callAnswered.value = true
  callMessageIndex.value = 0
  micMuted.value = false
  cameraOff.value = false
  showCallMessage(0)
}

function showCallMessage(index) {
  if (!callOpen.value || !callAnswered.value) return
  if (index >= CALL_MESSAGES.value.length) {
    gameTimers.push(setTimeout(endCall, 1200))
    return
  }
  callMessageIndex.value = index
  gameTimers.push(setTimeout(() => showCallMessage(index + 1), CALL_MESSAGE_MS))
}

function endCall() {
  clearGameTimers()
  stopSfx()
  callOpen.value = false
  trashLater(() => {
    gameState.value = 'idle'
    showStageBanner()
  }, 700)
}

// ========== STAGE BANNER + AI TRASH SORTING SCANNER ==========
const STAGE_BANNER_MS = 3800
const TRASH_SECONDS = 25
const TRASH_GOAL = 8
const TRASH_TICK_MS = 50
const TRASH_MAX_ITEMS = 6
const TRASH_TX = computed(() => t('trashTx'))
const TRASH_KINDS = ['plastic', 'ewaste', 'organic']
const TRASH_LABELS = computed(() => ({
  plastic: t('lblPlastic'),
  ewaste: t('lblEwaste'),
  organic: t('lblOrganic')
}))
const BINS = computed(() => [
  { id: 'plastic', name: t('binPlastic'), key: '1', color: '#4fc3ff', glow: 'rgba(79,195,255,0.55)' },
  { id: 'ewaste', name: t('binEwaste'), key: '2', color: '#ffcf3f', glow: 'rgba(255,207,63,0.55)' },
  { id: 'organic', name: t('binOrganic'), key: '3', color: '#6bff9c', glow: 'rgba(107,255,156,0.55)' }
])

const stageBanner = ref(false)
const bannerLine1 = ref('stage1Done')
const bannerLine2 = ref('airPurified')
const allDoneBanner = ref(false)
const trashState = ref('idle')
const trashItems = ref([])
const trashTime = ref(TRASH_SECONDS)
const trashCorrect = ref(0)
const selectedId = ref(null)
const binFlash = ref({ id: null, ok: false })
const trashToast = ref('')
const trashHint = ref(false)
const trashTxOpen = ref(false)
const trashWinFlash = ref(false)

let trashIv = null
let trashMs = 0
let trashSpawnAcc = 0
let trashNextSpawn = 0
let trashUid = 0
let toastTimer = null
const trashTimers = []

function trashLater(fn, ms) {
  trashTimers.push(setTimeout(fn, ms))
}
function stopTrashLoop() {
  clearInterval(trashIv)
  window.removeEventListener('keydown', onTrashKey)
}
function clearTrashTimers() {
  stopTrashLoop()
  clearTimeout(toastTimer)
  trashTimers.forEach(clearTimeout)
  trashTimers.length = 0
}

function showStageBanner(line1 = 'stage1Done', line2 = 'airPurified', next = null) {
  bannerLine1.value = line1
  bannerLine2.value = line2
  stageBanner.value = true
  trashLater(() => {
    stageBanner.value = false
    trashLater(() => {
      if (next) next()
      else trashState.value = 'intro'
    }, 900)
  }, STAGE_BANNER_MS)
}

function startTrash() {
  clearTrashTimers()
  trashItems.value = []
  trashCorrect.value = 0
  selectedId.value = null
  binFlash.value = { id: null, ok: false }
  trashTime.value = TRASH_SECONDS
  trashMs = TRASH_SECONDS * 1000
  trashSpawnAcc = 0
  trashNextSpawn = 500
  trashToast.value = ''
  trashTxOpen.value = false
  trashWinFlash.value = false
  trashHint.value = true
  trashState.value = 'starting'
  trashLater(() => {
    trashState.value = 'playing'
    trashIv = setInterval(trashTick, TRASH_TICK_MS)
    window.addEventListener('keydown', onTrashKey)
  }, 1200)
  trashLater(() => { trashHint.value = false }, 5500)
}

function trashTick() {
  if (trashState.value !== 'playing') return
  trashMs -= TRASH_TICK_MS
  trashTime.value = Math.max(0, Math.ceil(trashMs / 1000))

  trashSpawnAcc += TRASH_TICK_MS
  if (trashSpawnAcc >= trashNextSpawn) {
    trashSpawnAcc = 0
    trashNextSpawn = 900 + Math.random() * 500
    spawnTrash()
  }

  const elapsed = 1 - trashMs / (TRASH_SECONDS * 1000)
  const speed = 0.45 + elapsed * 0.25
  trashItems.value.forEach((item) => {
    if (item.id !== selectedId.value) item.x += speed
  })
  trashItems.value = trashItems.value.filter((item) => item.x < 108)

  if (trashMs <= 0) loseTrash()
}

function spawnTrash() {
  if (trashItems.value.length >= TRASH_MAX_ITEMS) return
  const kind = TRASH_KINDS[Math.floor(Math.random() * TRASH_KINDS.length)]
  trashItems.value.push({ id: ++trashUid, kind, x: -8, scanning: false, scanned: false })
}

function scanItem(item) {
  if (trashState.value !== 'playing' || selectedId.value === item.id) return
  selectedId.value = item.id
  item.scanning = true
  item.scanned = false
  trashLater(() => {
    item.scanning = false
    item.scanned = true
  }, 450)
}

function flashToast(message) {
  trashToast.value = message
  clearTimeout(toastTimer)
  toastTimer = setTimeout(() => { trashToast.value = '' }, 900)
}

function sortInto(binId) {
  if (trashState.value !== 'playing') return
  const item = trashItems.value.find((entry) => entry.id === selectedId.value)
  if (!item) { flashToast(t('scanFirst')); return }

  const correct = item.kind === binId
  binFlash.value = { id: binId, ok: correct }
  trashLater(() => { binFlash.value = { id: null, ok: false } }, 450)
  selectedId.value = null

  if (correct) {
    trashItems.value = trashItems.value.filter((entry) => entry.id !== item.id)
    trashCorrect.value++
    playSync()
    if (trashCorrect.value >= TRASH_GOAL) winTrash()
  } else {
    trashMs = Math.max(0, trashMs - 1000)
    trashTime.value = Math.max(0, Math.ceil(trashMs / 1000))
    flashToast(t('wrongBin'))
  }
}

function onTrashKey(e) {
  const bin = BINS.value.find((item) => item.key === e.key)
  if (bin) sortInto(bin.id)
}

function loseTrash() {
  stopTrashLoop()
  trashItems.value = []
  selectedId.value = null
  trashState.value = 'lost'
}

function winTrash() {
  stopTrashLoop()
  trashItems.value = []
  selectedId.value = null
  trashState.value = 'won'
  trashWinFlash.value = true
  trashLater(() => { trashWinFlash.value = false }, 1600)
  trashLater(() => {
    trashTxOpen.value = true
    playRadio()
  }, 3000)
}

function onTrashContinue() {
  clearTrashTimers()
  stopSfx()
  trashTxOpen.value = false
  trashLater(() => {
    trashState.value = 'idle'
    showStageBanner('stage2Done', 'waterDone', () => {
      droneState.value = 'intro'
    })
  }, 500)
}

// ========== MINI GAME 3: SATELLITE DRONE REFORESTATION ==========
const DRONE_SECONDS = 20
const DRONE_GOAL = 10
const DRONE_TX = computed(() => t('droneTx'))
const HE_SUB = computed(() => t('heSub'))

const droneState = ref('idle')
const dronePlots = ref([])
const droneTime = ref(DRONE_SECONDS)
const droneGrown = ref(0)
const droneHint = ref(false)
const droneTxOpen = ref(false)
const droneWinFlash = ref(false)
let droneIv = null
let droneMs = 0
const droneTimers = []

function droneLater(fn, ms) {
  droneTimers.push(setTimeout(fn, ms))
}
function clearDroneTimers() {
  clearInterval(droneIv)
  droneTimers.forEach(clearTimeout)
  droneTimers.length = 0
}

function buildPlots() {
  const columns = 5
  const rows = 4
  const random = seeded(Math.floor(Math.random() * 1000) + 1)
  const plots = []
  for (let row = 0; row < rows; row++) {
    for (let column = 0; column < columns; column++) {
      plots.push({
        id: row * columns + column,
        x: 14 + column * (72 / (columns - 1)) + (random() - 0.5) * 3,
        y: 30 + row * (48 / (rows - 1)) + (random() - 0.5) * 3,
        grown: false,
        launching: false
      })
    }
  }
  return plots
}

function startDrone() {
  clearDroneTimers()
  dronePlots.value = buildPlots()
  droneGrown.value = 0
  droneTime.value = DRONE_SECONDS
  droneMs = DRONE_SECONDS * 1000
  droneTxOpen.value = false
  droneWinFlash.value = false
  droneHint.value = true
  droneState.value = 'starting'
  droneLater(() => {
    droneState.value = 'playing'
    droneIv = setInterval(droneTick, 100)
  }, 1200)
  droneLater(() => { droneHint.value = false }, 5000)
}

function droneTick() {
  if (droneState.value !== 'playing') return
  droneMs -= 100
  droneTime.value = Math.max(0, Math.ceil(droneMs / 1000))
  if (droneMs <= 0) loseDrone()
}

function plantSeed(plot) {
  if (droneState.value !== 'playing' || plot.grown || plot.launching) return
  plot.launching = true
  droneLater(() => {
    if (droneState.value !== 'playing' || plot.grown) return
    plot.launching = false
    plot.grown = true
    droneGrown.value++
    playSync()
    if (droneGrown.value >= DRONE_GOAL) winDrone()
  }, 350)
}

function loseDrone() {
  clearDroneTimers()
  droneState.value = 'lost'
}

function winDrone() {
  clearDroneTimers()
  droneState.value = 'won'
  droneWinFlash.value = true
  droneLater(() => { droneWinFlash.value = false }, 1600)
  droneLater(() => {
    droneTxOpen.value = true
    playRadio()
  }, 3000)
}

function onDroneContinue() {
  clearDroneTimers()
  stopSfx()
  droneTxOpen.value = false
  droneState.value = 'idle'
  allDoneBanner.value = true
  droneLater(() => {
    allDoneBanner.value = false
    droneLater(startHappyEnding, 900)
  }, STAGE_BANNER_MS)
}

// ========== HAPPY ENDING ==========
const happyStage = ref(0)
const happyContactOpen = ref(false)
const happyTimers = []
const happyThanksName = computed(() => t('thanksName', { name: nickname.value || 'friend' }))
const showLang = computed(() =>
  phase.value !== 'loading' && phase.value !== 'whatever' &&
  gameState.value !== 'playing' && trashState.value !== 'playing' && droneState.value !== 'playing'
)

function happyLater(fn, ms) {
  happyTimers.push(setTimeout(fn, ms))
}

function startHappyEnding() {
  happyStage.value = 0
  happyContactOpen.value = false
  happyLater(() => { happyStage.value = 1 }, 300)
  happyLater(() => { happyStage.value = 2 }, 5000)
  happyLater(() => { happyStage.value = 3 }, 9000)
  happyLater(() => { happyStage.value = 4 }, 14000)
  happyLater(() => { happyStage.value = 5 }, 19000)
  happyLater(() => { happyStage.value = 6 }, 21000)
}

onMounted(startLoading)

onBeforeUnmount(() => {
  cancelled = true
  clearTimeout(glitchTimer)
  clearGameTimers()
  clearTrashTimers()
  clearDroneTimers()
  happyTimers.forEach(clearTimeout)
  clearTimeout(realityTimer)
  clearInterval(badIv)
  clearInterval(endingIv)
  endingEl.value?.pause()
  clearInterval(fireIv)
  fireEl.value?.pause()
  clearTimeout(copiedTimer)
  badTimers.forEach(clearTimeout)
  inspectTimers.forEach(clearTimeout)
  detachScrollListeners()
  releaseScrollLock()
  stopLook()
  stopSfx()
  natureEl.value?.pause()
  openingEl.value?.pause()
  mainEl.value?.pause()
  audioCtx?.close?.()
})

const currentLine = computed(() => lines.value[narrativeIndex.value] || '')

// st-p1 / st-p2 / st-p3: fase waking  |  st-open: narrative (mata terbuka penuh)
// st-dark: transmission (langit gelap + glitch)
const sceneState = computed(() => {
  if (phase.value === 'waking') return `st-p${wakeStage.value}`
  if (phase.value === 'transmission') {
    return glitchOut.value ? 'st-open st-dark st-glitch-out' : 'st-open st-dark'
  }
  return 'st-open'
})
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Permanent+Marker&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Fredoka:wght@600;700&display=swap');

.intro-root {
  position: fixed;
  inset: 0;
  background: #000;
  color: #f4f0e8;
  font-family: 'Permanent Marker', cursive;
  overflow: hidden;
  -webkit-font-smoothing: antialiased;
  z-index: 9999;
}
.stage {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}

.whatever-video-stage {
  background: #000;
}

.whatever-video {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

/* ============ REALITY CHECK + PILIHAN ============ */
.reality-check-stage {
  background: #000;
}

.reality-check-copy {
  width: min(860px, 86vw);
  text-align: center;
  transition: opacity 1.2s ease; /* samakan dengan REALITY_FADE_MS */
}
.reality-check-copy.hidden { opacity: 0; }

.reality-check-copy .tx-header {
  margin-bottom: 24px;
}

.reality-message {
  color: #f4f0e8;
  font-size: clamp(1.2rem, 2.8vw, 2rem);
  line-height: 1.4;
  text-shadow: 0 0 22px rgba(255, 120, 70, 0.28);
}

.reality-text-enter-active,
.reality-text-leave-active {
  transition: opacity 0.65s ease, transform 0.65s ease;
}

.reality-text-enter-from,
.reality-text-leave-to {
  opacity: 0;
  transform: translateY(12px);
}

.reality-choices {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  gap: 24px;
  width: min(92vw, 780px);
  align-items: stretch;
  justify-content: center;
}
.rc-btn {
  flex: 1;
  font-family: 'Permanent Marker', cursive;
  font-size: clamp(0.85rem, 1.7vw, 1.2rem);
  letter-spacing: 0.05em;
  line-height: 1.35;
  padding: 18px 20px;
  cursor: pointer;
  transition: transform 0.2s ease, border-color 0.25s ease, box-shadow 0.25s ease,
    color 0.25s ease, background 0.25s ease;
}
.rc-btn:hover { transform: translateY(-2px); }
.rc-btn:focus-visible { outline: 2px solid #f4f0e8; outline-offset: 3px; }

/* kiri: dark red / grey */
.rc-still {
  color: #a89a96;
  background: linear-gradient(180deg, rgba(48, 16, 16, 0.9), rgba(24, 24, 26, 0.92));
  border: 1px solid rgba(140, 40, 40, 0.65);
  box-shadow: inset 0 0 14px rgba(0, 0, 0, 0.6);
}
.rc-still:hover {
  color: #d8b8b0;
  border-color: #b23030;
  box-shadow: 0 0 18px rgba(170, 30, 30, 0.45), inset 0 0 14px rgba(0, 0, 0, 0.6);
}

/* kanan: green / cyan glow */
.rc-fix {
  color: #9dffe0;
  background: rgba(4, 30, 28, 0.8);
  border: 1px solid rgba(80, 255, 210, 0.75);
  text-shadow: 0 0 8px rgba(80, 255, 200, 0.85);
  animation: fixGlow 2s ease-in-out infinite;
}
.rc-fix:hover { color: #e2fff5; border-color: #8dffe6; }
@keyframes fixGlow {
  0%, 100% {
    box-shadow: 0 0 12px rgba(60, 255, 190, 0.35), inset 0 0 10px rgba(60, 255, 190, 0.15);
  }
  50% {
    box-shadow: 0 0 32px rgba(60, 255, 210, 0.85), inset 0 0 16px rgba(60, 255, 210, 0.3);
  }
}
@media (max-width: 560px) {
  .reality-choices { flex-direction: column; gap: 16px; }
}

.fade-enter-active, .fade-leave-active { transition: opacity 0.9s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; }

.loading-pct {
  font-size: clamp(1rem, 2.5vw, 1.6rem);
  letter-spacing: 0.05em;
  font-variant-numeric: tabular-nums;
}

.gg-bg {
  background: #000;
  overflow: hidden;
}
.gg-glow {
  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  height: 260px;
  transform: translateY(-50%);
  background: radial-gradient(ellipse 60% 100% at 50% 50%,
    rgba(150, 215, 255, 0.98) 0%,
    rgba(70, 160, 235, 0.7) 32%,
    rgba(25, 90, 180, 0.3) 58%,
    transparent 78%);
  filter: blur(36px);
  pointer-events: none;
  animation: ggGlowPulse 3.4s ease-in-out infinite;
}
@keyframes ggGlowPulse {
  0%, 100% { opacity: 0.85; }
  50% { opacity: 1; }
}

/* clouds drifting inside the sky-blue band */
.gg-clouds {
  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  height: 240px;
  transform: translateY(-50%);
  pointer-events: none;
  overflow: hidden;
  -webkit-mask-image: radial-gradient(ellipse 48% 50% at 50% 50%, #000 30%, transparent 100%);
  mask-image: radial-gradient(ellipse 48% 50% at 50% 50%, #000 30%, transparent 100%);
}
.gg-clouds i {
  position: absolute;
  left: -30%;
  width: 26%;
  height: 34%;
  border-radius: 50%;
  background:
    radial-gradient(circle at 30% 60%, rgba(255,255,255,0.95) 0 28%, transparent 30%),
    radial-gradient(circle at 55% 40%, rgba(255,255,255,0.98) 0 34%, transparent 36%),
    radial-gradient(circle at 78% 62%, rgba(255,255,255,0.92) 0 26%, transparent 28%),
    radial-gradient(ellipse at 50% 75%, rgba(255,255,255,0.85) 0 45%, transparent 47%);
  filter: blur(9px);
  opacity: 0.85;
  animation: ggCloudDrift 26s linear infinite;
}
.gg-clouds i:nth-child(1) { top: 18%; animation-duration: 30s; animation-delay: -4s; }
.gg-clouds i:nth-child(2) { top: 48%; width: 32%; animation-duration: 38s; animation-delay: -18s; opacity: 0.7; }
.gg-clouds i:nth-child(3) { top: 30%; width: 22%; animation-duration: 24s; animation-delay: -10s; }
.gg-clouds i:nth-child(4) { top: 58%; width: 28%; animation-duration: 34s; animation-delay: -26s; opacity: 0.75; }
.gg-clouds i:nth-child(5) { top: 8%;  width: 20%; animation-duration: 42s; animation-delay: -32s; opacity: 0.6; }
@keyframes ggCloudDrift {
  from { transform: translateX(0); }
  to   { transform: translateX(520%); }
}
.gg-title {
  position: relative;
  font-size: clamp(2rem, 7vw, 4.5rem);
  letter-spacing: 0.02em;
  text-shadow: 0 2px 18px rgba(10, 40, 100, 0.65), 0 0 6px rgba(10, 40, 100, 0.5);
}
.gg-sub {
  position: absolute;
  bottom: 12vh;
  font-size: 1rem;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  color: #8a8378;
  cursor: pointer;
  animation: pulse 2s ease-in-out infinite;
}
@keyframes pulse { 0%, 100% { opacity: 0.4; } 50% { opacity: 1; } }

.hp-icon { width: 56px; height: 56px; stroke: #f4f0e8; fill: none; stroke-width: 1.4; }
.hp-text {
  margin-top: 20px;
  text-align: center;
  font-size: 1.1rem;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  color: #8a8378;
  line-height: 1.6;
}

/* ============ PHASE 0: BLACKOUT + GARIS HEARTBEAT MERAH ============ */
.blackout { background: #000; overflow: hidden; }

/* semburat merah tipis yang berdenyut, ikut ritme heartbeat */
.blackout-pulse {
  position: absolute;
  inset: 0;
  pointer-events: none;
  background: radial-gradient(ellipse 70% 60% at 50% 60%,
    rgba(150, 12, 12, 0.32) 0%, rgba(90, 6, 6, 0.14) 45%, transparent 75%);
  opacity: 0;
  animation: bloodPulse 1.6s ease-in-out infinite;
}
@keyframes bloodPulse {
  0%, 100% { opacity: 0; }
  18%      { opacity: 0.9; }
  30%      { opacity: 0.25; }
  42%      { opacity: 0.7; }
  60%      { opacity: 0; }
}

.ecg {
  position: absolute;
  left: 6vw;
  bottom: 10vh;
  width: clamp(160px, 22vw, 260px);
}
.ecg svg { width: 100%; height: auto; overflow: visible; }
.ecg-line {
  fill: none;
  stroke: #ff2a2a;
  stroke-width: 1.6;
  stroke-linecap: round;
  stroke-linejoin: round;
  stroke-dasharray: 1;
  stroke-dashoffset: 1;
  filter: drop-shadow(0 0 4px rgba(255, 30, 30, 0.9));
  animation: ecgDraw 1.6s linear infinite;
}
@keyframes ecgDraw {
  0%   { stroke-dashoffset: 1; opacity: 1; }
  65%  { stroke-dashoffset: 0; opacity: 1; }
  90%  { stroke-dashoffset: 0; opacity: 0; }
  100% { stroke-dashoffset: 1; opacity: 0; }
}
.ecg-dot {
  fill: #ff2a2a;
  filter: drop-shadow(0 0 5px rgba(255, 30, 30, 1));
  opacity: 0.2;
  animation: ecgDot 1.6s ease-in-out infinite;
}
@keyframes ecgDot {
  0%, 55% { opacity: 0.2; }
  68%     { opacity: 1; }
  100%    { opacity: 0.2; }
}

/* ============ SCENE ============ */
.scene { background: #000; overflow: hidden; }

/* dunia (city1.avif) — blur & warna dikontrol per fase */
.scene-world {
  position: absolute;
  top: -4vh; bottom: -4vh; left: -10vw; right: -10vw;
  z-index: 1;
  filter: blur(20px) saturate(0.6);          /* Phase 1 */
  transition: filter 1.2s ease;
}
.st-p2 .scene-world {
  filter: blur(10px) saturate(0.8);          /* Phase 2 */
  animation: panAround 6.2s ease-in-out forwards;
}
.st-p3 .scene-world,
.st-open .scene-world {
  filter: blur(0) saturate(1.5);             /* Phase 3: fokus + warna tajam */
}
/* panning kiri-kanan pelan, baru mulai setelah 2x kedip selesai */
@keyframes panAround {
  0%, 50%   { transform: translateX(0) rotate(0); }
  63%       { transform: translateX(5vw) rotate(0.6deg); }
  76%       { transform: translateX(-5vw) rotate(-0.6deg); }
  92%, 100% { transform: translateX(0) rotate(0); }
}

.scene-bg {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  user-select: none;
  -webkit-user-drag: none;
}
.waking-glow {
  position: absolute;
  inset: 0;
  background: radial-gradient(ellipse 55% 30% at 50% 50%,
    rgba(255, 110, 40, 0.6) 0%, rgba(168, 68, 32, 0.35) 40%, transparent 78%);
  filter: blur(24px);
  mix-blend-mode: screen;
  opacity: 0.45;
  pointer-events: none;
}

/* ============ ASAP KEBAKARAN + POLUSI DI BELAKANG KARAKTER ============ */
/* Lapisan ini duduk di antara city1.avif dan karakter (person-wrap z-index: 2).
   Class .smog / .fire-glow / .plume / .ember / .wisp dipakai bersama layar inspect
   dan layar bad ending. */
.scene-smoke {
  position: absolute;
  inset: 0;
  z-index: 1;
  overflow: hidden;
  pointer-events: none;
}

/* ============ KARAKTER (gerak halus biar nggak kaku) ============ */
.person-wrap {
  position: absolute;
  z-index: 2;                  /* di atas asap */
  left: 0; right: 0;
  bottom: 4vh;
  height: 80%;                 /* ubah ini kalau karakter mau lebih besar/kecil */
  display: flex;
  justify-content: center;
  align-items: flex-end;
  pointer-events: none;
}
/* lapisan 1: goyang / miring pelan (kepala oleng) */
.person-sway {
  height: 100%;
  transform-origin: 50% 100%;
  animation: personSway 7s ease-in-out infinite;
}
/* phase 1 & 2: masih pusing, geraknya lebih oleng & pelan */
.st-p1 .person-sway,
.st-p2 .person-sway {
  animation: personGroggy 4.6s ease-in-out infinite;
}
/* lapisan 2: napas (naik-turun tipis) + fade in tiap ganti gambar */
.person-img {
  height: 100%;
  width: auto;
  display: block;
  transform-origin: 50% 100%;
  user-select: none;
  -webkit-user-drag: none;
  animation: personBreath 3.4s ease-in-out infinite, personIn 1.2s ease both;
}
@keyframes personSway {
  0%, 100% { transform: translate(0, 0) rotate(0deg); }
  30%      { transform: translate(-0.6vw, -0.4vh) rotate(-0.5deg); }
  65%      { transform: translate(0.7vw, 0) rotate(0.6deg); }
}
@keyframes personGroggy {
  0%, 100% { transform: translate(0, 0) rotate(0deg); }
  20%      { transform: translate(-1vw, 0.4vh) rotate(-1.6deg); }
  45%      { transform: translate(0.4vw, -0.3vh) rotate(0.8deg); }
  70%      { transform: translate(1.2vw, 0.5vh) rotate(1.8deg); }
}
@keyframes personBreath {
  0%, 100% { transform: scale(1, 1); }
  50%      { transform: scale(1.012, 1.02); }
}
@keyframes personIn {
  from { opacity: 0; }
  to   { opacity: 1; }
}

.scene-vignette {
  position: absolute; inset: 0; z-index: 3; pointer-events: none;
  background: radial-gradient(ellipse 75% 70% at 50% 50%,
    transparent 30%, rgba(0,0,0,0.55) 68%, rgba(0,0,0,0.95) 100%);
}

/* ============ LANGIT GELAP + GLITCH ============ */
.st-dark .scene-world {
  filter: blur(0) saturate(0.9) brightness(0.5) contrast(1.15);
  transition: filter 3s ease;
}
.st-dark .scene-bg { animation: bgGlitch 4s steps(1) infinite; }
@keyframes bgGlitch {
  0%, 100% { transform: translate(0, 0); }
  8%   { transform: translate(-6px, 1px); filter: hue-rotate(40deg); }
  9%   { transform: translate(5px, -1px); }
  10%  { transform: translate(0, 0); filter: none; }
  47%  { transform: translate(4px, 0); }
  48%  { transform: translate(-3px, 1px); filter: hue-rotate(-40deg); }
  49%  { transform: translate(0, 0); filter: none; }
  81%  { transform: translate(-8px, 0); }
  82%  { transform: translate(0, 0); }
}

/* ============ TRANSISI GLITCH -> VIDEO WHATEVER ============ */
.st-glitch-out { animation: stageShake 0.16s steps(2) infinite; }
@keyframes stageShake {
  0% { transform: translate(0, 0); }
  25% { transform: translate(-6px, 2px); }
  50% { transform: translate(5px, -3px); }
  75% { transform: translate(-3px, -2px); }
  100% { transform: translate(4px, 3px); }
}
.st-glitch-out .scene-bg { animation: bgGlitchHard 0.28s steps(1) infinite; }
@keyframes bgGlitchHard {
  0% { transform: translate(0, 0) scale(1); filter: none; }
  20% { transform: translate(-14px, 2px) scale(1.03); filter: hue-rotate(70deg) saturate(2); }
  40% { transform: translate(12px, -3px) scale(1); filter: invert(1) hue-rotate(180deg); }
  60% { transform: translate(-6px, 4px) scale(1.05); filter: hue-rotate(-60deg) contrast(1.6); }
  80% { transform: translate(9px, 0) scale(1); filter: saturate(3) brightness(1.4); }
}
.glitch-burst {
  position: absolute;
  inset: 0;
  z-index: 20;
  pointer-events: none;
  overflow: hidden;
  animation: burstBlack 1.4s linear forwards;
}
@keyframes burstBlack {
  0%, 55% { background: rgba(0, 0, 0, 0); }
  100% { background: #000; }
}
.glitch-burst::before {
  content: '';
  position: absolute;
  inset: 0;
  background: #fff;
  mix-blend-mode: overlay;
  animation: burstFlash 0.45s steps(5) forwards;
}
@keyframes burstFlash {
  0% { opacity: 0.9; }
  30% { opacity: 0.1; }
  55% { opacity: 0.7; }
  100% { opacity: 0; }
}
.glitch-burst i {
  position: absolute;
  left: 0;
  right: 0;
  opacity: 0;
  mix-blend-mode: screen;
  animation: burstBar 0.35s steps(1) infinite;
}
.glitch-burst i:nth-child(1) { top: 6%; height: 12px; background: rgba(255, 50, 50, 0.6); }
.glitch-burst i:nth-child(2) { top: 20%; height: 5px; background: rgba(80, 220, 255, 0.6); animation-delay: -0.1s; }
.glitch-burst i:nth-child(3) { top: 38%; height: 22px; background: rgba(255, 255, 255, 0.35); animation-delay: -0.2s; }
.glitch-burst i:nth-child(4) { top: 55%; height: 8px; background: rgba(255, 90, 40, 0.6); animation-delay: -0.05s; }
.glitch-burst i:nth-child(5) { top: 68%; height: 16px; background: rgba(80, 255, 170, 0.45); animation-delay: -0.25s; }
.glitch-burst i:nth-child(6) { top: 82%; height: 6px; background: rgba(255, 255, 255, 0.5); animation-delay: -0.15s; }
.glitch-burst i:nth-child(7) { top: 92%; height: 10px; background: rgba(255, 40, 40, 0.55); animation-delay: -0.3s; }
@keyframes burstBar {
  0%, 100% { opacity: 0; transform: translateX(0); }
  15% { opacity: 1; transform: translateX(-8%); }
  35% { opacity: 0; }
  55% { opacity: 1; transform: translateX(10%); }
  70% { opacity: 0; }
  85% { opacity: 1; transform: translateX(-4%); }
}

/* langit makin gelap dari atas */
.sky-dark {
  position: absolute; inset: 0; z-index: 2; pointer-events: none;
  background: linear-gradient(to bottom,
    rgba(8, 0, 0, 0.92) 0%, rgba(25, 6, 4, 0.65) 35%, transparent 70%);
  opacity: 0;
  transition: opacity 3.5s ease;
}
.st-dark .sky-dark { opacity: 1; }

.glitch { position: absolute; inset: 0; z-index: 4; pointer-events: none; overflow: hidden; }
.glitch .scan {
  position: absolute; inset: 0;
  background: repeating-linear-gradient(0deg,
    rgba(255, 255, 255, 0.05) 0 1px, transparent 1px 3px);
  mix-blend-mode: overlay;
  animation: scanFlicker 0.15s steps(2) infinite;
}
@keyframes scanFlicker { 0% { opacity: 0.6; } 100% { opacity: 1; } }

.glitch i {
  position: absolute; left: 0; right: 0;
  height: 6px;
  background: rgba(255, 90, 60, 0.22);
  mix-blend-mode: screen;
  opacity: 0;
  animation: glitchBar 3.2s steps(1) infinite;
}
.glitch i:nth-child(2) { top: 14%; height: 3px; animation-delay: -0.4s; }
.glitch i:nth-child(3) { top: 33%; height: 10px; background: rgba(80, 200, 255, 0.18); animation-delay: -1.3s; }
.glitch i:nth-child(4) { top: 52%; height: 4px; animation-delay: -2.1s; }
.glitch i:nth-child(5) { top: 71%; height: 8px; background: rgba(255, 255, 255, 0.12); animation-delay: -2.8s; }
.glitch i:nth-child(6) { top: 88%; height: 3px; animation-delay: -0.9s; }
@keyframes glitchBar {
  0%, 100% { opacity: 0; transform: translateX(0); }
  6%  { opacity: 1; transform: translateX(-4%); }
  8%  { opacity: 0; }
  41% { opacity: 1; transform: translateX(6%); }
  43% { opacity: 0; }
  77% { opacity: 1; transform: translateX(-2%); }
  79% { opacity: 0; }
}

/* ============ MATA (bukaan elips) ============ */
/* Layar hitam, dunia cuma kelihatan lewat lubang elips berpinggir lembut.
   --ry = tinggi bukaan, --rx = lebar bukaan. Makin besar = mata makin terbuka. */
@property --rx { syntax: '<percentage>'; inherits: false; initial-value: 40%; }
@property --ry { syntax: '<percentage>'; inherits: false; initial-value: 0.5%; }

.scene-eye {
  position: absolute; inset: 0;
  z-index: 1;
  --rx: 40%;
  --ry: 0.5%;                  /* tertutup */
  -webkit-mask-image: radial-gradient(ellipse var(--rx) var(--ry) at 50% 50%, #000 55%, transparent 100%);
          mask-image: radial-gradient(ellipse var(--rx) var(--ry) at 50% 50%, #000 55%, transparent 100%);
}
.st-p1 .scene-eye { animation: eyeP1 4.2s ease-in-out forwards; }
.st-p2 .scene-eye { animation: eyeP2 6.2s ease-in-out forwards; }
.st-p3 .scene-eye { animation: eyeP3 4.2s ease-in-out forwards; }
.st-open .scene-eye { -webkit-mask-image: none; mask-image: none; }

/* Phase 1: celah tipis (~20%), tahan, lalu merem lagi */
@keyframes eyeP1 {
  0%, 10%  { --rx: 40%; --ry: 0.5%; }
  32%      { --rx: 70%; --ry: 12%; }
  60%      { --rx: 70%; --ry: 12%; }
  85%, 100% { --rx: 40%; --ry: 0.5%; }
}
/* Phase 2: kedip 2x lebih lebar (~60%), lalu tetap terbuka */
@keyframes eyeP2 {
  0%, 8%   { --rx: 40%; --ry: 0.5%; }
  17%      { --rx: 85%; --ry: 45%; }
  23%      { --rx: 60%; --ry: 3%; }
  33%      { --rx: 85%; --ry: 45%; }
  40%      { --rx: 60%; --ry: 3%; }
  52%, 100% { --rx: 85%; --ry: 45%; }
}
/* Phase 3: buka sebentar -> MEREM TOTAL -> (gambar people1 -> people2 ganti
   di tengah fase merem, ~1.2s) -> melek -> terbuka penuh sampai bukaan hilang */
@keyframes eyeP3 {
  0%, 8%    { --rx: 85%;  --ry: 45%; }    /* masih terbuka dari phase 2 */
  24%       { --rx: 40%;  --ry: 0.5%; }   /* merem total (~1.0s) */
  40%       { --rx: 40%;  --ry: 0.5%; }   /* tahan merem, gambar ganti di sini (~1.2s) */
  56%       { --rx: 85%;  --ry: 45%; }    /* buka mata, sudah people2 */
  85%, 100% { --rx: 160%; --ry: 160%; }   /* terbuka penuh */
}

/* ============ TEKS ============ */
/* kotak teks kanan atas (Hey... / Hey, wake up! / Are you okay?) */
.waking-box {
  position: absolute;
  top: 6vh;
  right: 4vw;
  z-index: 6;                  /* di atas overlay supaya tetap terbaca */
  max-width: 60vw;
  padding: 10px 22px;
  text-align: center;
  background: rgba(64, 24, 20, 0.85);
  border: 1px solid rgba(255, 140, 90, 0.22);
  box-shadow: 0 4px 22px rgba(0, 0, 0, 0.55);
  opacity: 0;
}
.tx-header {
  font-family: 'Courier New', monospace;
  font-size: clamp(0.55rem, 1.2vw, 0.75rem);
  letter-spacing: 0.15em;
  color: #ff9a5c;
  margin-bottom: 6px;
}
.waking-text {
  font-size: clamp(0.8rem, 1.7vw, 1.15rem);
  line-height: 1.25;
  color: #f4f0e8;
}
/* teks baru muncul setelah mata "sempat" kebuka, bukan berebut sama kedipan */
.st-p1 .waking-box { animation: textP1 4.2s ease-in-out forwards; }
.st-p2 .waking-box { animation: textP2 6.2s ease-in-out forwards; }
.st-p3 .waking-box { animation: textP3 4.2s ease-in-out forwards; }

@keyframes textP1 {              /* "Hey..." muncul, hilang sebelum merem */
  0%, 30%   { opacity: 0; }
  48%       { opacity: 1; }
  68%       { opacity: 1; }
  86%, 100% { opacity: 0; }
}
@keyframes textP2 {              /* muncul setelah kedipan ke-2, tetap ada */
  0%, 42%   { opacity: 0; }
  62%, 100% { opacity: 1; }
}
@keyframes textP3 {              /* muncul setelah mata melek lagi (people2), fade out di akhir */
  0%, 52%   { opacity: 0; }
  68%       { opacity: 1; }
  90%       { opacity: 1; }
  100%      { opacity: 0; }
}

/* posisi narrative: bawah tengah (seperti subtitle) */
.scene .narrative-text {
  position: absolute;
  left: 50%;
  bottom: 12vh;
  transform: translateX(-50%);
  width: max-content;
  max-width: 80vw;
}

.narrative-bg {
  background: radial-gradient(ellipse at 50% 30%, #6b2e21 0%, #3a2019 35%, #2b2b2b 70%, #1c1c1c 100%);
}
.narrative-text {
  position: relative;
  z-index: 6;
  max-width: 80vw;
  text-align: center;
  white-space: pre-line;
  font-size: clamp(0.8rem, 1.7vw, 1.15rem);
  line-height: 1.25;
  letter-spacing: 0.01em;
  text-shadow: 0 2px 14px rgba(0, 0, 0, 0.7);
  opacity: 0;
  transition: opacity 1.4s ease;
}
.narrative-text.show { opacity: 1; }
.handoff-text {
  font-size: 1.1rem;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: #8a8378;
}

/* ============ TRANSMISSION TEXT & CHOICES ============ */
.tx-box {
  position: absolute;
  top: 6vh;
  right: 4vw;
  z-index: 6;
  max-width: min(60vw, 420px);
  padding: 10px 22px;
  text-align: center;
  background: rgba(64, 24, 20, 0.85);
  border: 1px solid rgba(255, 140, 90, 0.22);
  box-shadow: 0 4px 22px rgba(0, 0, 0, 0.55);
  animation: txBoxIn 0.9s ease both;
}
@keyframes txBoxIn { from { opacity: 0; } to { opacity: 1; } }
.tx-fade { animation: txFade 0.9s ease both; }
@keyframes txFade { from { opacity: 0; } to { opacity: 1; } }

.choices {
  position: absolute;
  left: 50%;
  bottom: 14vh;
  transform: translateX(-50%);
  z-index: 8;
  display: flex;
  gap: 16px;
  width: min(92vw, 820px);
  justify-content: center;
  align-items: stretch;
  animation: txFade 1s ease both;
}
.choice {
  flex: 1;
  font-family: 'Permanent Marker', cursive;
  font-size: clamp(0.7rem, 1.4vw, 1rem);
  letter-spacing: 0.04em;
  line-height: 1.3;
  color: #f4f0e8;
  background: rgba(20, 8, 8, 0.8);
  border: 1px solid rgba(255, 140, 90, 0.35);
  padding: 12px 16px;
  cursor: pointer;
  transition: border-color 0.2s, box-shadow 0.2s, color 0.2s;
}
.choice:hover {
  border-color: #ff9a5c;
  color: #ffb98a;
  box-shadow: 0 0 16px rgba(255, 110, 40, 0.35);
}
.choice-warn { border-color: rgba(255, 90, 60, 0.6); }

/* ============ INSPECT: 3 WARNING NODES ============ */
.inspect { background: #000; overflow: hidden; display: block; }
.inspect-bg {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  user-select: none;
  -webkit-user-drag: none;
  /* gambar dasar sedikit lebih pekat & berasap */
  filter: saturate(1.1) contrast(1.05);
}

/* ---------- ASAP KEBAKARAN + POLUSI ---------- */
.smoke-layer,
.smoke-front {
  position: absolute;
  inset: 0;
  overflow: hidden;
  pointer-events: none;
}
.smoke-layer { z-index: 1; }
.smoke-front { z-index: 2; }

/* kabut polusi: pita gelap kecokelatan yang menggantung, bergerak pelan */
.smog {
  position: absolute;
  left: -20%;
  right: -20%;
  filter: blur(40px);
  will-change: transform, opacity;
}
.smog-top {
  top: -6%;
  height: 34%;
  background: radial-gradient(ellipse 60% 100% at 50% 40%,
    rgba(30, 10, 6, 0.85) 0%, rgba(60, 18, 8, 0.5) 45%, transparent 75%);
  animation: smogDrift 34s ease-in-out infinite alternate;
}
.smog-mid {
  top: 26%;
  height: 30%;
  background: radial-gradient(ellipse 55% 100% at 50% 50%,
    rgba(70, 30, 14, 0.5) 0%, rgba(45, 16, 8, 0.32) 50%, transparent 78%);
  animation: smogDrift 46s ease-in-out infinite alternate-reverse;
}
.smog-low {
  bottom: -4%;
  height: 34%;
  background: radial-gradient(ellipse 60% 100% at 50% 70%,
    rgba(20, 8, 5, 0.85) 0%, rgba(50, 20, 10, 0.45) 50%, transparent 80%);
  animation: smogDrift 28s ease-in-out infinite alternate;
}
@keyframes smogDrift {
  0%   { transform: translateX(-6%) scaleY(1);    opacity: 0.85; }
  50%  { transform: translateX(2%)  scaleY(1.12); opacity: 1; }
  100% { transform: translateX(7%)  scaleY(0.95); opacity: 0.8; }
}

/* cahaya api dari bawah (berkedip acak) */
.fire-glow {
  position: absolute;
  width: 34vw;
  height: 22vh;
  border-radius: 50%;
  background: radial-gradient(ellipse at 50% 60%,
    rgba(255, 150, 40, 0.75) 0%, rgba(255, 80, 20, 0.4) 40%, transparent 72%);
  filter: blur(22px);
  mix-blend-mode: screen;
  opacity: 0.6;
  animation: fireFlick 1.9s ease-in-out infinite;
}
.fg-1 { left: -4vw;  bottom: 2vh;  width: 30vw; animation-delay: -0.2s; }
.fg-2 { left: 26vw;  bottom: 14vh; width: 24vw; height: 18vh; animation-duration: 2.4s; animation-delay: -1.1s; }
.fg-3 { left: 58vw;  bottom: 6vh;  width: 26vw; animation-duration: 1.6s; animation-delay: -0.7s; }
.fg-4 { right: -4vw; bottom: 10vh; width: 28vw; animation-duration: 2.1s; animation-delay: -1.6s; }
@keyframes fireFlick {
  0%, 100% { opacity: 0.45; transform: scale(1, 1); }
  18%      { opacity: 0.85; transform: scale(1.06, 1.14); }
  34%      { opacity: 0.55; transform: scale(0.97, 0.94); }
  52%      { opacity: 0.95; transform: scale(1.08, 1.18); }
  74%      { opacity: 0.5;  transform: scale(1, 1.02); }
}

/* kolom asap yang naik */
.plume {
  position: absolute;
  border-radius: 50%;
  /* bagian bawah oranye kena cahaya api, atasnya abu kecokelatan */
  background: radial-gradient(circle at 50% 62%,
    rgba(120, 50, 22, 0.75) 0%,
    rgba(72, 34, 20, 0.62) 32%,
    rgba(48, 26, 20, 0.34) 58%,
    transparent 74%);
  filter: blur(16px);
  opacity: 0;
  will-change: transform, opacity;
  animation-name: plumeRise;
  animation-timing-function: ease-out;
  animation-iteration-count: infinite;
}
/* asap pekat (hitam, dari plastik/ban terbakar) */
.plume.dark {
  background: radial-gradient(circle at 50% 60%,
    rgba(18, 10, 8, 0.92) 0%,
    rgba(28, 14, 10, 0.72) 36%,
    rgba(34, 18, 12, 0.36) 60%,
    transparent 76%);
  filter: blur(20px);
}
@keyframes plumeRise {
  0%   { transform: translate(0, 0) scale(0.35); opacity: 0; }
  14%  { opacity: var(--peak, 0.5); }
  60%  { opacity: calc(var(--peak, 0.5) * 0.8); }
  100% {
    transform: translate(var(--dx, 8vw), var(--rise, -50vh)) scale(var(--grow, 2.4));
    opacity: 0;
  }
}

/* bara api */
.ember {
  position: absolute;
  border-radius: 50%;
  background: radial-gradient(circle, #ffe2a0 0%, #ff8a2a 55%, rgba(255, 60, 20, 0) 100%);
  box-shadow: 0 0 8px 2px rgba(255, 120, 40, 0.8);
  opacity: 0;
  will-change: transform, opacity;
  animation-name: emberRise;
  animation-timing-function: cubic-bezier(0.2, 0.6, 0.4, 1);
  animation-iteration-count: infinite;
}
@keyframes emberRise {
  0%   { transform: translate(0, 0) scale(1);   opacity: 0; }
  10%  { opacity: 1; }
  45%  { transform: translate(calc(var(--ex, 8vw) * 0.45), calc(var(--ey, -40vh) * 0.5)) scale(0.85); opacity: 0.9; }
  75%  { opacity: 0.55; }
  100% { transform: translate(var(--ex, 8vw), var(--ey, -40vh)) scale(0.2); opacity: 0; }
}

/* asap tipis di depan: melintas horizontal (parallax) */
.wisp {
  position: absolute;
  left: -60vw;
  border-radius: 50%;
  background: radial-gradient(ellipse 50% 50% at 50% 50%,
    rgba(60, 26, 16, 0.75) 0%, rgba(80, 36, 20, 0.35) 48%, transparent 76%);
  filter: blur(30px);
  animation-name: wispCross;
  animation-timing-function: linear;
  animation-iteration-count: infinite;
  will-change: transform;
}
@keyframes wispCross {
  from { transform: translateX(0); }
  to   { transform: translateX(200vw); }
}

/* lampu api berkedip: seluruh layar sedikit lebih terang-oranye lalu redup */
.fire-flicker {
  position: absolute;
  inset: 0;
  z-index: 2;
  pointer-events: none;
  background: radial-gradient(ellipse 90% 60% at 50% 100%,
    rgba(255, 110, 30, 0.22) 0%, transparent 70%);
  mix-blend-mode: screen;
  animation: screenFlick 2.6s steps(1) infinite;
}
@keyframes screenFlick {
  0%   { opacity: 0.7; }
  10%  { opacity: 1; }
  16%  { opacity: 0.6; }
  38%  { opacity: 0.95; }
  44%  { opacity: 0.65; }
  71%  { opacity: 1; }
  78%  { opacity: 0.75; }
  100% { opacity: 0.7; }
}

.inspect-vignette {
  position: absolute; inset: 0; z-index: 3; pointer-events: none;
  background: radial-gradient(ellipse 80% 75% at 50% 50%,
    transparent 40%, rgba(0,0,0,0.5) 80%, rgba(0,0,0,0.85) 100%);
}
.inspect .glitch { z-index: 3; }

/* hemat performa untuk yang mematikan animasi di OS */
@media (prefers-reduced-motion: reduce) {
  .plume, .ember, .wisp, .smog, .fire-glow, .fire-flicker { animation: none; }
  .plume { opacity: 0.35; }
  .ember { display: none; }
}

/* petunjuk kiri atas */
.inspect-hint {
  position: absolute;
  top: 3vh;
  left: 2vw;
  z-index: 6;
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px 18px 10px 12px;
  background: rgba(30, 10, 8, 0.85);
  border: 1px solid rgba(255, 140, 90, 0.3);
  box-shadow: 0 4px 22px rgba(0, 0, 0, 0.55);
  animation: txFade 1s ease both;
}
.inspect-hint.gone {
  opacity: 0;
  pointer-events: none;
  transition: opacity 1.4s ease;
  animation: none;
}
.hint-hand {
  width: 30px; height: 30px; flex: none;
  fill: none; stroke: #ff6a3d; stroke-width: 1.6;
  stroke-linecap: round; stroke-linejoin: round;
  animation: handTap 1.4s ease-in-out infinite;
}
@keyframes handTap {
  0%, 100% { transform: translateY(0); }
  50%      { transform: translateY(-4px); }
}
.hint-text {
  font-family: 'Courier New', monospace;
  font-size: clamp(0.55rem, 1.1vw, 0.75rem);
  letter-spacing: 0.1em;
  line-height: 1.5;
  color: #f4f0e8;
}
.hint-text b { color: #ff9a5c; }
.hint-count {
  font-family: 'Courier New', monospace;
  font-size: 0.8rem;
  color: #ff9a5c;
  padding-left: 10px;
  border-left: 1px solid rgba(255, 140, 90, 0.3);
}

/* node */
.node {
  position: absolute;
  z-index: 5;
  width: 54px;
  height: 54px;
  padding: 0;
  transform: translate(-50%, -50%);
  background: none;
  border: none;
  cursor: pointer;
  transition: opacity 1.4s ease;
}
/* pin memudar halus setelah ketiganya selesai */
.node.gone { opacity: 0; pointer-events: none; }
.node-icon {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: rgba(60, 8, 8, 0.85);
  border: 2px solid #ff3a2a;
  box-shadow: 0 0 14px rgba(255, 50, 30, 0.7), inset 0 0 10px rgba(255, 50, 30, 0.35);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}
.node-icon svg {
  width: 26px; height: 26px;
  fill: none; stroke: #ffd8cc; stroke-width: 1.8;
  stroke-linecap: round; stroke-linejoin: round;
}
.node:hover .node-icon {
  transform: scale(1.1);
  box-shadow: 0 0 22px rgba(255, 90, 60, 0.95), inset 0 0 12px rgba(255, 90, 60, 0.5);
}
/* ring berkedip */
.node-ring {
  position: absolute;
  inset: -12px;
  border-radius: 50%;
  border: 1.5px solid rgba(255, 60, 40, 0.8);
  animation: nodePing 1.8s ease-out infinite;
  pointer-events: none;
}
@keyframes nodePing {
  0%   { transform: scale(0.75); opacity: 0.95; }
  100% { transform: scale(1.35); opacity: 0; }
}
/* label di kanan */
.node-tag {
  position: absolute;
  left: calc(100% + 10px);
  top: 50%;
  transform: translateY(-50%);
  white-space: nowrap;
  padding: 4px 10px;
  font-family: 'Courier New', monospace;
  font-size: clamp(0.55rem, 1vw, 0.7rem);
  font-weight: bold;
  letter-spacing: 0.12em;
  color: #f4f0e8;
  background: rgba(15, 5, 5, 0.9);
  border: 1px solid rgba(255, 140, 90, 0.25);
}
/* garis putus-putus + titik merah di bawah */
.node-stem {
  position: absolute;
  left: 50%;
  top: 100%;
  width: 0;
  height: var(--stem, 14vh);
  margin-top: 4px;
  border-left: 1.5px dotted rgba(255, 90, 60, 0.85);
  pointer-events: none;
}
.node-stem::after {
  content: '';
  position: absolute;
  left: -4.5px;
  bottom: -4px;
  width: 8px; height: 8px;
  border-radius: 50%;
  background: #ff2a2a;
  box-shadow: 0 0 8px rgba(255, 30, 30, 1);
}
/* sudah dibuka: redup, ring berhenti */
.node.seen .node-ring { animation: none; opacity: 0.25; }
.node.seen .node-icon { border-color: #ff9a5c; box-shadow: 0 0 8px rgba(255, 140, 90, 0.4); }

/* kartu data */
.card-overlay {
  position: absolute;
  inset: 0;
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(0, 0, 0, 0.6);
}
.data-card {
  width: min(90vw, 520px);
  padding: 20px 24px 22px;
  text-align: center;
  background: rgba(30, 10, 8, 0.95);
  border: 1px solid rgba(255, 140, 90, 0.35);
  box-shadow: 0 0 40px rgba(255, 60, 30, 0.25), 0 8px 30px rgba(0, 0, 0, 0.7);
  animation: cardIn 0.4s ease both;
}
.nickname-overlay {
  position: absolute;
  inset: 0;
  z-index: 20;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 24px;
  background: rgba(0, 0, 0, 0.2);
  animation: txFade 0.7s ease both;
}
.nickname-card {
  width: min(90vw, 440px);
  padding: 24px;
  text-align: center;
  background: rgba(30, 10, 8, 0.96);
  border: 1px solid rgba(255, 140, 90, 0.5);
  box-shadow: 0 0 40px rgba(255, 60, 30, 0.24), 0 8px 30px rgba(0, 0, 0, 0.7);
  animation: cardIn 0.4s ease both;
}
.nickname-label {
  display: block;
  margin: 18px 0 12px;
  color: #f4f0e8;
  font-size: clamp(1.1rem, 2.5vw, 1.5rem);
}
.nickname-input {
  display: block;
  width: 100%;
  min-height: 48px;
  padding: 12px 14px;
  color: #f4f0e8;
  font: inherit;
  background: rgba(0, 0, 0, 0.5);
  border: 1px solid rgba(255, 140, 90, 0.5);
  outline: none;
}
.nickname-input:focus { border-color: #ff9a5c; box-shadow: 0 0 14px rgba(255, 110, 40, 0.28); }
.nickname-input::placeholder { color: #8a8378; }
.nickname-submit { width: 100%; margin-top: 16px; }
@keyframes cardIn {
  from { opacity: 0; transform: translateY(10px) scale(0.98); }
  to   { opacity: 1; transform: none; }
}
.card-title {
  font-size: clamp(1rem, 2.2vw, 1.4rem);
  margin: 6px 0 16px;
  color: #f4f0e8;
}
.card-cols {
  display: flex;
  align-items: stretch;
  justify-content: center;
  gap: 12px;
}
.card-col {
  flex: 1;
  padding: 12px 10px;
  border: 1px solid rgba(255, 255, 255, 0.12);
  background: rgba(255, 255, 255, 0.03);
}
.card-col.danger {
  border-color: rgba(255, 60, 40, 0.55);
  background: rgba(255, 40, 20, 0.08);
}
.card-arrow { align-self: center; color: #ff6a3d; font-size: 1.4rem; }
.card-year {
  font-family: 'Courier New', monospace;
  font-size: 0.7rem;
  letter-spacing: 0.2em;
  color: #8a8378;
}
.danger .card-year { color: #ff9a5c; }
.card-val {
  margin: 6px 0;
  font-size: clamp(1.1rem, 2.6vw, 1.7rem);
  line-height: 1.15;
  color: #f4f0e8;
}
.danger .card-val { color: #ff5a3c; text-shadow: 0 0 12px rgba(255, 60, 30, 0.6); }
.card-note {
  font-family: 'Courier New', monospace;
  font-size: clamp(0.55rem, 1vw, 0.7rem);
  line-height: 1.4;
  color: #b8b0a2;
}
.card-msg {
  margin: 18px 0 16px;
  font-size: clamp(0.8rem, 1.6vw, 1.05rem);
  line-height: 1.35;
  color: #ffb98a;
}
.card-close { flex: none; }

/* ============ COMPLETION: FLASH SINYAL TERHUBUNG (0.3s) ============ */
.sync-flash {
  position: absolute;
  inset: 0;
  z-index: 12;
  pointer-events: none;
  overflow: hidden;
  background: radial-gradient(ellipse 90% 80% at 50% 50%,
    rgba(255, 130, 50, 0.85) 0%, rgba(255, 60, 30, 0.6) 55%, rgba(200, 20, 10, 0.5) 100%);
  mix-blend-mode: screen;
  animation: syncFlash 0.3s steps(6) both;
}
.sync-flash i {
  position: absolute;
  left: 0; right: 0;
  background: rgba(255, 240, 220, 0.55);
}
.sync-flash i:nth-child(1) { top: 18%; height: 8px; transform: translateX(-6%); }
.sync-flash i:nth-child(2) { top: 47%; height: 14px; transform: translateX(8%); }
.sync-flash i:nth-child(3) { top: 74%; height: 5px; transform: translateX(-4%); }
@keyframes syncFlash {
  0%   { opacity: 0; }
  15%  { opacity: 1; }
  35%  { opacity: 0.35; }
  55%  { opacity: 0.9; }
  100% { opacity: 0; }
}

/* ============ COMPLETION: DIALOG POV (bawah tengah) ============ */
.pov-text {
  position: absolute;
  z-index: 7;
  left: 50%;
  bottom: 12vh;
  width: max-content;
  max-width: min(80vw, 720px);
  text-align: center;
  font-size: clamp(0.9rem, 1.9vw, 1.3rem);
  line-height: 1.3;
  color: #fff;
  text-shadow: 0 2px 14px rgba(0, 0, 0, 0.85);
  opacity: 0;
  transform: translate(-50%, 18px);
  transition: opacity 1s ease, transform 1s ease;
  pointer-events: none;
}
.pov-text.show {
  opacity: 1;
  transform: translate(-50%, 0);
}

/* ============ COMPLETION: PESAN 2076 (kanan atas, neon berkedip) ============ */
.tx-neon {
  animation: none;
  opacity: 0;
  transition: opacity 0.8s ease;
  pointer-events: none;
}
.tx-neon.show {
  opacity: 1;
  border-color: rgba(255, 140, 40, 0.75);
  animation: neonBox 1.3s ease-in-out infinite;
}
.tx-neon .tx-header,
.tx-neon .waking-text {
  color: #ff9a2e;
  text-shadow: 0 0 6px rgba(255, 140, 30, 0.9), 0 0 18px rgba(255, 100, 20, 0.7);
}
.tx-neon.show .waking-text,
.tx-neon.show .tx-header {
  animation: neonText 1.3s steps(1) infinite;
}
@keyframes neonBox {
  0%, 100% { box-shadow: 0 0 10px rgba(255, 120, 30, 0.5), 0 4px 22px rgba(0, 0, 0, 0.55); }
  50%      { box-shadow: 0 0 26px rgba(255, 140, 40, 0.95), 0 4px 22px rgba(0, 0, 0, 0.55); }
}
@keyframes neonText {
  0%, 100% { opacity: 1; }
  8%       { opacity: 0.55; }
  12%      { opacity: 1; }
  46%      { opacity: 1; }
  50%      { opacity: 0.45; }
  54%      { opacity: 1; }
  78%      { opacity: 0.7; }
  82%      { opacity: 1; }
}

/* ============ COMPLETION: SCROLL TEASER (hijau neon berdenyut) ============ */
.scroll-teaser {
  position: absolute;
  z-index: 8;
  left: 50%;
  bottom: 5vh;
  transform: translateX(-50%);
  max-width: 92vw;
  padding: 10px 20px;
  font-family: 'Courier New', monospace;
  font-weight: bold;
  font-size: clamp(0.6rem, 1.3vw, 0.9rem);
  letter-spacing: 0.12em;
  white-space: nowrap;
  color: #6bff9c;
  background: rgba(4, 22, 10, 0.75);
  border: 1px solid rgba(80, 255, 140, 0.6);
  text-shadow: 0 0 8px rgba(80, 255, 140, 0.9);
  cursor: pointer;
  opacity: 0;
  pointer-events: none;
  transition: opacity 1s ease;
}
.scroll-teaser.show {
  opacity: 1;
  pointer-events: auto;
  animation: teaserPulse 1.6s ease-in-out infinite;
}
@keyframes teaserPulse {
  0%, 100% {
    box-shadow: 0 0 10px rgba(80, 255, 140, 0.35), inset 0 0 8px rgba(80, 255, 140, 0.15);
    color: #6bff9c;
  }
  50% {
    box-shadow: 0 0 28px rgba(80, 255, 140, 0.9), inset 0 0 14px rgba(80, 255, 140, 0.35);
    color: #b8ffd0;
  }
}
@media (max-width: 560px) {
  .scroll-teaser { white-space: normal; text-align: center; width: 88vw; }
}

/* ============ RED-TO-GREEN WORLD SHIFT ============ */
/* asap, api, glitch, vignette, teks memudar (cross-dissolve 1.8s) */
.inspect .smoke-layer,
.inspect .smoke-front,
.inspect .fire-flicker,
.inspect .inspect-vignette,
.inspect .glitch,
.inspect .inspect-bg {
  transition: opacity 1.8s ease;
}
.inspect.shifting .smoke-layer,
.inspect.shifting .smoke-front,
.inspect.shifting .fire-flicker,
.inspect.shifting .inspect-vignette,
.inspect.shifting .glitch,
.inspect.shifting .inspect-bg {
  opacity: 0;
}

.green-world {
  position: absolute;
  inset: 0;
  z-index: 4;
  overflow: hidden;
  pointer-events: none;
  opacity: 0;
  transition: opacity 2s ease;
  background: #000;
}
.green-world.on { opacity: 1; }

/* ============ BANGUN TIDUR: kedip natural kayak intro + POV rebahan -> berdiri ============ */
.wake-text {
  z-index: 9;
  top: 50%;
  bottom: auto;
  transform: translate(-50%, calc(-50% + 18px));
  width: min(88vw, 900px);
}
.wake-text.show { transform: translate(-50%, -50%); }
.wake-text.wake-text-first {
  top: auto;
  bottom: 12vh;
  transform: translate(-50%, 18px);
}
.wake-text.wake-text-first.show { transform: translate(-50%, 0); }

/* ============ ECO-PULSE INTRO ============ */
.eco-overlay {
  position: absolute;
  inset: 0;
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 24px;
  pointer-events: auto;
}
.eco-card {
  width: min(92vw, 460px);
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  font-family: 'Fredoka', 'Permanent Marker', sans-serif;
}
.eco-title { display: flex; flex-direction: column; line-height: 0.95; }
.eco-name,
.eco-year {
  font-weight: 700;
  font-size: clamp(2.4rem, 8vw, 4rem);
  letter-spacing: 0.02em;
  color: #e9fff0;
  -webkit-text-stroke: 6px #1f8a4c;
  paint-order: stroke fill;
  text-shadow: 0 4px 0 #14683a, 0 8px 18px rgba(0, 60, 40, 0.45);
}
.eco-year { font-size: clamp(2.8rem, 9.5vw, 4.6rem); }
.eco-tag {
  margin-top: 10px;
  font-size: clamp(0.6rem, 1.6vw, 0.85rem);
  font-weight: 600;
  letter-spacing: 0.12em;
  color: #fff;
  text-shadow: 0 2px 6px rgba(0, 40, 60, 0.85);
}
.eco-start {
  margin-top: 26px;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 14px;
  padding: 16px 20px;
  font-family: inherit;
  font-weight: 700;
  font-size: clamp(1.1rem, 3vw, 1.5rem);
  letter-spacing: 0.06em;
  color: #fff;
  cursor: pointer;
  background: linear-gradient(180deg, #4fdc7a 0%, #1fa84f 100%);
  border: 3px solid #b8ffd0;
  border-radius: 999px;
  box-shadow: 0 6px 0 #14803a, 0 10px 24px rgba(0, 80, 40, 0.5);
  transition: transform 0.15s ease, box-shadow 0.15s ease;
  animation: ecoPulse 1.8s ease-in-out infinite;
}
.eco-start svg { width: 22px; height: 22px; fill: #fff; }
.eco-start:hover { transform: translateY(-2px); }
.eco-start:active { transform: translateY(4px); box-shadow: 0 2px 0 #14803a; }
.eco-start:focus-visible { outline: 3px solid #fff; outline-offset: 4px; }
@keyframes ecoPulse {
  0%, 100% { filter: brightness(1); }
  50%      { filter: brightness(1.12); }
}

.eco-info {
  margin-top: 20px;
  width: 100%;
  display: flex;
  align-items: center;
  gap: 14px;
  padding: 12px 16px;
  text-align: left;
  background: rgba(12, 44, 74, 0.82);
  border: 1px solid rgba(160, 220, 255, 0.35);
  border-radius: 14px;
  box-shadow: 0 6px 20px rgba(0, 20, 40, 0.4);
}
.eco-info p {
  margin: 0;
  font-size: clamp(0.75rem, 1.8vw, 0.95rem);
  font-weight: 600;
  line-height: 1.35;
  color: #eaf6ff;
}
.eco-clock {
  width: 38px;
  height: 38px;
  flex: none;
  padding: 7px;
  fill: none;
  stroke: #eaf6ff;
  stroke-width: 1.8;
  stroke-linecap: round;
  stroke-linejoin: round;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.12);
}

/* ============ ECO-PULSE GAME ============ */
.game-root {
  position: absolute;
  inset: 0;
  z-index: 8;
  pointer-events: none;
  font-family: 'Fredoka', 'Permanent Marker', sans-serif;
}
.game-root.paused *,
.game-root.paused *::before,
.game-root.paused *::after { animation-play-state: paused !important; }

.game-smog {
  position: absolute;
  inset: 0;
  z-index: 1;
  overflow: hidden;
  transition: opacity 0.6s ease;
  animation: smogIn 1.6s ease backwards;
}
@keyframes smogIn { from { opacity: 0; } }
.game-tint {
  position: absolute;
  inset: 0;
  background: radial-gradient(ellipse 90% 80% at 50% 55%,
    rgba(170, 50, 30, 0.45) 0%, rgba(90, 25, 15, 0.7) 100%);
}

.g-hud {
  position: absolute;
  top: 3vh;
  left: 0;
  right: 0;
  z-index: 5;
  padding: 0 2.5vw;
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 12px;
}
.g-card {
  background: rgba(12, 44, 74, 0.82);
  border: 1px solid rgba(160, 220, 255, 0.35);
  border-radius: 14px;
  box-shadow: 0 6px 20px rgba(0, 20, 40, 0.4);
  color: #eaf6ff;
}
.g-timer { display: flex; align-items: center; gap: 10px; padding: 8px 14px; }
.g-timer small { display: block; font-size: 0.6rem; font-weight: 600; letter-spacing: 0.1em; opacity: 0.8; }
.g-timer b { font-size: clamp(1.2rem, 3vw, 1.8rem); line-height: 1; font-variant-numeric: tabular-nums; }
.g-timer.danger { border-color: rgba(255, 90, 70, 0.7); }
.g-timer.danger b { color: #ff6a5a; }
.g-clock {
  width: 34px; height: 34px; flex: none;
  fill: none; stroke: currentColor; stroke-width: 1.8;
  stroke-linecap: round; stroke-linejoin: round;
}
.g-timer.danger .g-clock { stroke: #ff6a5a; }
.g-bar { flex: 1; max-width: 420px; padding: 8px 14px 10px; }
.g-bar-label {
  display: flex; align-items: center; gap: 8px;
  margin-bottom: 6px;
  font-size: clamp(0.7rem, 1.6vw, 0.9rem);
  font-weight: 700;
  letter-spacing: 0.04em;
}
.g-bar-label svg {
  width: 18px; height: 18px;
  fill: none; stroke: #6bffb0; stroke-width: 2;
  stroke-linecap: round; stroke-linejoin: round;
}
.g-segs { display: flex; gap: 4px; }
.g-segs i {
  flex: 1; height: 12px; border-radius: 3px;
  background: rgba(255, 255, 255, 0.14);
  transition: background 0.3s ease, box-shadow 0.3s ease;
}
.g-segs i.on {
  background: linear-gradient(180deg, #7dffb0, #22c55e);
  box-shadow: 0 0 8px rgba(80, 255, 150, 0.7);
}
.g-pause {
  pointer-events: auto;
  cursor: pointer;
  padding: 10px 16px;
  font-family: inherit;
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.08em;
}
.g-pause:hover { border-color: #fff; }
.g-hide { visibility: hidden; pointer-events: none; }

.g-bubble {
  position: absolute;
  z-index: 3;
  width: var(--s);
  aspect-ratio: 1;
  padding: 0;
  border: none;
  background: none;
  transform: translate(-50%, -50%);
  cursor: pointer;
  pointer-events: auto;
  transition: opacity 0.5s ease;
  animation: bubbleIn 0.35s cubic-bezier(0.3, 1.6, 0.5, 1) backwards;
  -webkit-tap-highlight-color: transparent;
}
.g-bubble.fading { opacity: 0; pointer-events: none; }
@keyframes bubbleIn { from { opacity: 0; transform: translate(-50%, -50%) scale(0.2); } }
.g-float { display: block; width: 100%; height: 100%; animation: bubbleFloat var(--fd, 2s) ease-in-out infinite alternate; }
@keyframes bubbleFloat {
  from { transform: translate(0, 0) rotate(-3deg); }
  to { transform: translate(var(--fx, 1vmin), var(--fy, 1vmin)) rotate(3deg); }
}
.g-float > svg { width: 100%; height: 100%; display: block; filter: drop-shadow(0 6px 10px rgba(60, 0, 0, 0.5)); }
.cl-out circle { fill: #6e1613; stroke: #6e1613; stroke-width: 7; }
.cl-fill circle { fill: #b8362d; }
.cl-brow { fill: none; stroke: #2a0503; stroke-width: 4; stroke-linecap: round; }
.cl-eye { fill: #fff; }
.cl-pupil { fill: #3a0806; }
.g-orb {
  display: flex; align-items: center; justify-content: center;
  width: 100%; height: 100%;
  border-radius: 50%;
  background: radial-gradient(circle at 40% 35%, #b8ffd0 0%, #33d16e 55%, #148a44 100%);
  border: 4px solid #d8ffe6;
  box-shadow: 0 0 0 6px rgba(120, 255, 170, 0.25), 0 0 26px 8px rgba(80, 255, 150, 0.7);
  animation: orbPulse 1.2s ease-in-out infinite;
}
.g-orb svg { width: 56%; height: 56%; }
.lf-body { fill: #fff; }
.lf-vein { fill: none; stroke: #1aa552; stroke-width: 5; stroke-linecap: round; }
@keyframes orbPulse {
  0%, 100% { filter: brightness(1); }
  50% { filter: brightness(1.2); }
}

.g-pop { position: absolute; z-index: 4; width: 0; height: 0; pointer-events: none; }
.g-ring {
  position: absolute; left: -3vmin; top: -3vmin;
  width: 6vmin; height: 6vmin;
  border-radius: 50%;
  border: 4px solid #ff5a4a;
  animation: ringOut 0.5s ease-out forwards;
}
.g-pop.leaf .g-ring { border-color: #6bffb0; }
@keyframes ringOut { to { transform: scale(2.6); opacity: 0; } }
.g-shard {
  position: absolute; left: -5px; top: -5px;
  width: 10px; height: 10px;
  border-radius: 50%;
  background: #d9574b;
  animation: shardOut 0.6s ease-out forwards;
}
.g-pop.leaf .g-shard { background: #3fe07a; border-radius: 0 100% 0 100%; }
@keyframes shardOut {
  from { transform: rotate(var(--a)) translateX(0) scale(1); opacity: 1; }
  to { transform: rotate(var(--a)) translateX(8vmin) scale(0.2); opacity: 0; }
}
.g-plus {
  position: absolute; left: 0; top: 0;
  transform: translate(-50%, -50%);
  font-size: clamp(1rem, 3vw, 1.7rem);
  color: #ff6a5a;
  -webkit-text-stroke: 4px #fff;
  paint-order: stroke fill;
  white-space: nowrap;
  animation: plusUp 0.9s ease-out forwards;
}
.g-pop.leaf .g-plus { color: #1fc25c; }
@keyframes plusUp {
  from { transform: translate(-50%, -50%); opacity: 1; }
  to { transform: translate(-50%, -14vmin); opacity: 0; }
}

.g-hint {
  position: absolute; left: 50%; bottom: 6vh; z-index: 4;
  transform: translateX(-50%);
  display: flex; align-items: center; gap: 12px;
  padding: 12px 18px;
  font-size: clamp(0.75rem, 1.8vw, 1rem);
  font-weight: 600;
  line-height: 1.35;
  text-align: left;
  opacity: 0;
  transition: opacity 0.6s ease;
}
.g-hint.show { opacity: 1; }
.g-hint svg {
  width: 30px; height: 30px; flex: none;
  fill: #fff; stroke: #0c2c4a; stroke-width: 1.2;
  stroke-linecap: round; stroke-linejoin: round;
  animation: handTap 1.4s ease-in-out infinite;
}

.g-overlay {
  position: absolute; inset: 0; z-index: 12;
  display: flex; align-items: center; justify-content: center;
  padding: 24px;
  background: rgba(0, 10, 20, 0.45);
  pointer-events: auto;
}
.g-result { padding: 24px 32px; text-align: center; }
.g-result-title { font-size: clamp(1.6rem, 5vw, 2.6rem); font-weight: 700; letter-spacing: 0.04em; color: #ffd5cc; }
.g-result p { margin: 10px 0 0; font-weight: 600; line-height: 1.4; }
.g-btn {
  margin-top: 18px; padding: 12px 34px;
  font-family: inherit; font-weight: 700; font-size: 1.1rem; letter-spacing: 0.06em;
  color: #fff; cursor: pointer;
  background: linear-gradient(180deg, #4fdc7a 0%, #1fa84f 100%);
  border: 3px solid #b8ffd0; border-radius: 999px;
  box-shadow: 0 5px 0 #14803a;
}
.g-btn:active { transform: translateY(3px); box-shadow: 0 2px 0 #14803a; }

.g-win-flash {
  position: absolute; inset: 0; z-index: 9; pointer-events: none;
  background: radial-gradient(circle at 50% 50%,
    rgba(230, 255, 235, 0.95) 0%, rgba(90, 255, 150, 0.65) 40%,
    rgba(40, 200, 110, 0.25) 75%, transparent 100%);
  mix-blend-mode: screen;
  animation: winFlash 1.6s ease-out both;
}
@keyframes winFlash { 0% { opacity: 0; } 15% { opacity: 1; } 100% { opacity: 0; } }
.g-center {
  position: absolute; inset: 0; z-index: 6;
  display: flex; align-items: center; justify-content: center;
  pointer-events: none;
}
.g-rays {
  position: absolute;
  width: min(140vmin, 1200px);
  aspect-ratio: 1;
  border-radius: 50%;
  background: repeating-conic-gradient(rgba(255, 255, 190, 0.35) 0 7deg, transparent 7deg 18deg);
  -webkit-mask-image: radial-gradient(circle, #000 0%, transparent 68%);
          mask-image: radial-gradient(circle, #000 0%, transparent 68%);
  animation: raysSpin 40s linear infinite;
}
@keyframes raysSpin { to { transform: rotate(360deg); } }
.g-title {
  position: relative;
  text-align: center;
  font-weight: 700;
  font-size: clamp(2.6rem, 9vw, 5.2rem);
  line-height: 0.95;
  color: #e9fff0;
  -webkit-text-stroke: 8px #1f8a4c;
  paint-order: stroke fill;
  text-shadow: 0 5px 0 #14683a, 0 10px 24px rgba(0, 60, 40, 0.5);
  animation: titleIn 0.7s cubic-bezier(0.3, 1.5, 0.5, 1) both;
}
@keyframes titleIn { from { opacity: 0; transform: scale(0.4); } }

.call-card {
  width: min(92vw, 520px);
  padding: 22px 24px;
  text-align: center;
  background: rgba(30, 10, 8, 0.95);
  border: 1px solid rgba(255, 140, 90, 0.5);
  box-shadow: 0 0 40px rgba(255, 60, 30, 0.25), 0 8px 30px rgba(0, 0, 0, 0.7);
  font-family: 'Permanent Marker', cursive;
  animation: cardIn 0.4s ease both;
}
.call-overlay.call-answered {
  background: transparent;
}
.call-overlay.call-answered .call-card {
  position: relative;
  z-index: 1;
  width: min(92vw, 520px);
  padding: 22px 24px;
  background: rgba(30, 10, 8, 0.94);
  border: 1px solid rgba(255, 140, 90, 0.5);
  box-shadow: 0 0 40px rgba(255, 60, 30, 0.25), 0 8px 30px rgba(0, 0, 0, 0.7);
  font-family: 'Fredoka', 'Permanent Marker', sans-serif;
  animation: none;
}
.call-reveal {
  min-height: 210px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
}
.call-person {
  width: min(38%, 180px);
  aspect-ratio: 1;
  height: auto;
  flex: none;
  align-self: center;
  object-fit: cover;
  object-position: center;
  filter: drop-shadow(0 12px 24px rgba(0, 15, 30, 0.55));
}
.call-dialogue {
  flex: 1;
  min-height: 170px;
  display: flex;
  align-items: center;
  padding: 10px;
}
.call-message {
  width: 100%;
  color: #eafaff;
  font-size: clamp(1.1rem, 2.4vw, 2rem);
  font-weight: 600;
  line-height: 1.45;
  text-align: left;
  text-shadow: 0 2px 14px rgba(0, 10, 24, 0.55);
}
.call-ring {
  width: 72px; height: 72px; margin: 18px auto 12px;
  display: flex; align-items: center; justify-content: center;
  border-radius: 50%;
  background: rgba(60, 255, 150, 0.15);
  border: 2px solid #6bff9c;
  animation: callShake 0.9s ease-in-out infinite;
}
.call-ring svg { width: 32px; height: 32px; fill: none; stroke: #6bff9c; stroke-width: 1.8; stroke-linecap: round; stroke-linejoin: round; }
@keyframes callShake {
  0%, 100% { transform: rotate(0); box-shadow: 0 0 0 0 rgba(107, 255, 156, 0.6); }
  25% { transform: rotate(-10deg); }
  50% { transform: rotate(10deg); box-shadow: 0 0 0 16px rgba(107, 255, 156, 0); }
  75% { transform: rotate(-6deg); }
}
.call-name { font-size: clamp(1.1rem, 2.6vw, 1.6rem); color: #f4f0e8; }
.call-sub { margin: 8px 0 4px; font-size: clamp(0.75rem, 1.5vw, 0.95rem); color: #b8b0a2; }
.call-answer { width: 100%; margin-top: 16px; color: #9dffe0; border-color: rgba(80, 255, 210, 0.75); }
.call-controls {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 18px;
  margin-top: 18px;
}
.call-control {
  width: 48px;
  height: 48px;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 12px;
  color: #eafaff;
  background: rgba(29, 105, 132, 0.92);
  border: 1px solid rgba(141, 229, 248, 0.3);
  border-radius: 50%;
  cursor: pointer;
  transition: background 0.18s ease, transform 0.18s ease;
}
.call-control:hover { transform: translateY(-2px); background: rgba(42, 139, 166, 0.98); }
.call-control.active { background: rgba(75, 80, 91, 0.95); }
.call-control-end { width: 54px; height: 54px; padding: 14px; background: #ed4654; border-color: #ff9ba2; }
.call-control-end:hover { background: #ff5965; }
.call-control svg { width: 100%; height: 100%; fill: none; stroke: currentColor; stroke-width: 1.8; stroke-linecap: round; stroke-linejoin: round; }
.call-control-end svg {
  fill: #fff;
  stroke: #fff;
  stroke-width: 1;
  transform: rotate(135deg);
}
.call-control .control-slash { stroke-width: 2.4; }
.call-control:focus-visible { outline: 2px solid #fff; outline-offset: 3px; }

@media (max-width: 600px) {
  .call-overlay.call-answered .call-card { padding: 16px; }
  .call-reveal { min-height: auto; flex-direction: column; gap: 8px; }
  .call-person { width: min(42vw, 150px); }
  .call-dialogue { width: 100%; min-height: 120px; flex: none; padding: 12px; }
  .call-message { font-size: clamp(1rem, 5vw, 1.35rem); }
  .call-controls { gap: 14px; margin-top: 12px; }
  .call-control { width: 44px; height: 44px; }
  .call-control-end { width: 50px; height: 50px; }
}

@media (max-width: 560px) {
  .g-hud { top: 2vh; padding: 0 3vw; gap: 8px; }
  .g-timer { gap: 6px; padding: 7px 9px; }
  .g-clock { width: 27px; height: 27px; }
  .g-bar { padding: 7px 9px 8px; }
  .g-segs { gap: 2px; }
  .g-segs i { height: 9px; }
  .g-pause { padding: 9px 10px; font-size: 0.68rem; }
  .g-hint { bottom: 4vh; max-width: 88vw; }
}

.green-world.waking {
  --rx: 40%;
  --ry: 0.5%;
  transition: none;
  -webkit-mask-image: radial-gradient(ellipse var(--rx) var(--ry) at 50% 50%, #000 55%, transparent 100%);
          mask-image: radial-gradient(ellipse var(--rx) var(--ry) at 50% 50%, #000 55%, transparent 100%);
  animation: wakeEye 11s ease-in-out forwards;
}

.gw-tilt {
  position: absolute;
  inset: 0;
  transform-origin: 50% 65%;
}
.green-world.waking .gw-tilt {
  animation: wakeTilt 11s ease-in-out forwards;
  will-change: transform, filter;
}

@keyframes wakeEye {
  0%, 14%   { --rx: 40%;  --ry: 0.5%; }
  25%       { --rx: 70%;  --ry: 12%; }
  36%       { --rx: 70%;  --ry: 12%; }
  44%, 48%  { --rx: 40%;  --ry: 0.5%; }
  54%       { --rx: 85%;  --ry: 45%; }
  58%       { --rx: 60%;  --ry: 3%; }
  65%       { --rx: 85%;  --ry: 45%; }
  70%       { --rx: 60%;  --ry: 3%; }
  78%, 84%  { --rx: 85%;  --ry: 45%; }
  96%, 100% { --rx: 160%; --ry: 160%; }
}

@keyframes wakeTilt {
  0%, 14%   { transform: translate(0, 14vh) rotate(-14deg) scale(1.5);  filter: blur(20px) saturate(0.7); }
  36%       { transform: translate(-2vw, 12vh) rotate(-10deg) scale(1.45); filter: blur(15px) saturate(0.75); }
  48%       { transform: translate(-2vw, 12vh) rotate(-10deg) scale(1.45); filter: blur(15px) saturate(0.75); }
  58%       { transform: translate(2vw, 8vh) rotate(-6deg) scale(1.32); filter: blur(10px) saturate(0.85); }
  70%       { transform: translate(-1vw, 9vh) rotate(-8deg) scale(1.34); filter: blur(9px) saturate(0.9); }
  84%       { transform: translate(1vw, 4vh) rotate(-3deg) scale(1.15); filter: blur(5px) saturate(1); }
  100%      { transform: translate(0, 0) rotate(0deg) scale(1); filter: blur(0) saturate(1); }
}

/* ============ LOOK AROUND: POV ikut kursor ============ */
/* --lx / --ly = -1..1 (posisi kursor), --lk = 0..1 (zoom masuk), diisi dari JS */
.gw-look {
  position: absolute;
  inset: 0;
  transform:
    translate(calc(var(--lx, 0) * -2.2vw), calc(var(--ly, 0) * -2.2vh))
    scale(calc(1 + var(--lk, 0) * 0.08));
  will-change: transform;
}

/* daun foreground geser lebih jauh = kesan kedalaman */
.gw-parallax {
  position: absolute;
  inset: 0;
  transform: translate(calc(var(--lx, 0) * -2.8vw), calc(var(--ly, 0) * -2.8vh));
  will-change: transform;
}

@media (prefers-reduced-motion: reduce) {
  .green-world.waking { animation: none; -webkit-mask-image: none; mask-image: none; }
  .green-world.waking .gw-tilt { animation: none; }
}

/* ---------- panggung: se-rasio gambar, menutupi layar seperti object-fit: cover ----------
  Semua overlay memakai % dari panggung ini, jadi posisinya selalu pas dengan gambar.
  --ar = lebar / tinggi city3.avif. Kalau gambarmu beda rasio, ubah angka ini. */
.green-world { --ar: 2.095; }
.gw-defs { position: absolute; width: 0; height: 0; }

.gw-cam {
  position: absolute;
  inset: 0;
  transform-origin: 80% 10%;
}
.green-world.on .gw-cam { animation: camBreath 40s ease-in-out infinite alternate; }
@keyframes camBreath {
  from { transform: scale(1); }
  to   { transform: scale(1.05); }
}

.gw-stage {
  position: absolute;
  left: 50%;
  top: 50%;
  width: max(100vw, calc(100vh * var(--ar)));
  height: max(100vh, calc(100vw / var(--ar)));
  transform: translate(-50%, -50%);
}
.gw-bg,
.gw-water img {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: fill;
  user-select: none;
  -webkit-user-drag: none;
}

/* ---------- AIR ---------- */
/* poligon = bentuk sungai (x% y% dari gambar). Geser titiknya kalau tepi airnya kurang pas. */
.gw-water {
  position: absolute;
  inset: 0;
  clip-path: polygon(
    27% 78%, 36% 74%, 45% 72%, 52% 70%, 60% 70%, 63% 72%,
    70% 80%, 74% 85%, 72% 88%, 60% 90%, 50% 93%, 42% 91%, 34% 86%
  );
}
.gw-water img {
  filter: url(#gwWaterFx);   /* riak: gambar sungai "bergoyang" pelan */
  will-change: filter;
}
/* kilatan cahaya yang menyusuri permukaan air */
.gw-sheen {
  position: absolute;
  inset: 0;
  opacity: 0.55;
  mix-blend-mode: soft-light;
  background:
    radial-gradient(ellipse 46px 3px at 50% 50%, rgba(255,255,255,0.95), transparent 70%),
    radial-gradient(ellipse 30px 2px at 50% 50%, rgba(255,255,255,0.8), transparent 70%);
  background-size: 140px 26px, 140px 26px;
  background-position: 0 0, 70px 13px;
  animation: sheenSlide 14s linear infinite;
}
@keyframes sheenSlide {
  to { background-position: 280px 0, 350px 13px; }
}
.glint {
  position: absolute;
  border-radius: 50%;
  background: radial-gradient(circle, #fff 0%, rgba(255,255,255,0.85) 35%, transparent 70%);
  box-shadow: 0 0 8px 2px rgba(255, 255, 255, 0.7);
  opacity: 0;
  animation-name: glintTwinkle;
  animation-timing-function: ease-in-out;
  animation-iteration-count: infinite;
}
@keyframes glintTwinkle {
  0%, 100% { opacity: 0;    transform: scale(0.4); }
  45%      { opacity: 0.95; transform: scale(1.2); }
  70%      { opacity: 0.3;  transform: scale(0.8); }
}

/* ---------- MATAHARI ---------- */
.gw-sunglow {
  position: absolute;
  left: 80%;
  top: 4%;
  width: 34%;
  aspect-ratio: 1;
  transform: translate(-50%, -50%);
  border-radius: 50%;
  background: radial-gradient(circle,
    rgba(255, 255, 240, 0.95) 0%,
    rgba(255, 250, 200, 0.55) 22%,
    rgba(255, 244, 180, 0.22) 45%,
    transparent 70%);
  mix-blend-mode: screen;
  animation: sunGlow 5s ease-in-out infinite;
}
@keyframes sunGlow {
  0%, 100% { transform: translate(-50%, -50%) scale(1);    opacity: 0.8; }
  50%      { transform: translate(-50%, -50%) scale(1.12); opacity: 1; }
}
/* lingkaran lens flare di bawah-kiri matahari, redup-terang pelan */
.gw-flare {
  position: absolute;
  left: 72%;
  top: 16%;
  width: 9%;
  aspect-ratio: 1;
  transform: translate(-50%, -50%);
  border-radius: 50%;
  background: radial-gradient(circle,
    rgba(255, 255, 255, 0.28) 0%, rgba(200, 235, 255, 0.16) 60%, transparent 72%);
  border: 1px solid rgba(255, 255, 255, 0.18);
  mix-blend-mode: screen;
  animation: flareFade 7s ease-in-out infinite;
}
@keyframes flareFade {
  0%, 100% { opacity: 0.35; }
  50%      { opacity: 0.9; }
}

/* sinar: titik asal = pusat matahari (80%, 4%) */
.gw-rays {
  position: absolute;
  left: 80%;
  top: 4%;
  width: 0;
  height: 0;
  filter: blur(9px);              /* blur di parent supaya pinggiran berkas halus */
  mix-blend-mode: screen;
  animation: raysSweep 18s ease-in-out infinite alternate;
}
.ray {
  position: absolute;
  top: 0;
  left: calc(var(--w) / -2);
  width: var(--w);
  height: 150vmax;
  transform-origin: 50% 0;
  transform: rotate(var(--a));
  clip-path: polygon(42% 0, 58% 0, 100% 100%, 0 100%); /* melebar ke bawah */
  background: linear-gradient(to bottom,
    rgba(255, 252, 225, 0.55) 0%,
    rgba(255, 252, 225, 0.22) 35%,
    transparent 78%);
  opacity: var(--o);
  animation: rayShimmer var(--d) ease-in-out infinite;
  animation-delay: var(--dl);
}
@keyframes rayShimmer {
  0%, 100% { opacity: calc(var(--o) * 0.35); }
  50%      { opacity: var(--o); }
}
@keyframes raysSweep {
  from { transform: rotate(-2deg); }
  to   { transform: rotate(2deg); }
}

/* ---------- PARTIKEL HIDUP ---------- */
.fleaf {
  position: absolute;
  border-radius: 0 100% 0 100%;
  background: linear-gradient(135deg,
    hsl(var(--hue), 70%, 60%) 0%, hsl(var(--hue), 65%, 36%) 100%);
  opacity: 0;
  animation-name: leafFall;
  animation-timing-function: linear;
  animation-iteration-count: infinite;
  will-change: transform, opacity;
}
@keyframes leafFall {
  0%   { transform: translate(0, 0) rotate(0deg);                                  opacity: 0; }
  8%   { opacity: 0.9; }
  25%  { transform: translate(calc(var(--dx) * 0.25 + 3vw), 28vh) rotate(90deg); }
  50%  { transform: translate(calc(var(--dx) * 0.5 - 3vw), 55vh) rotate(180deg); }
  75%  { transform: translate(calc(var(--dx) * 0.75 + 3vw), 82vh) rotate(270deg); opacity: 0.9; }
  100% { transform: translate(var(--dx), 110vh) rotate(360deg);                    opacity: 0; }
}
.mote {
  position: absolute;
  border-radius: 50%;
  background: radial-gradient(circle, #fffbe0 0%, rgba(255, 245, 170, 0.6) 45%, transparent 75%);
  box-shadow: 0 0 8px 2px rgba(255, 245, 170, 0.55);
  opacity: 0;
  animation-name: moteFloat;
  animation-timing-function: ease-in-out;
  animation-iteration-count: infinite;
}
@keyframes moteFloat {
  0%   { transform: translate(0, 0);                        opacity: 0; }
  30%  { opacity: 0.9; }
  70%  { opacity: 0.6; }
  100% { transform: translate(var(--mx), var(--my));        opacity: 0; }
}

/* burung terbang (putih) */
.gw-birds { position: absolute; inset: 0; overflow: hidden; }
.bird {
  position: absolute;
  left: -8vw;
  display: block;
  aspect-ratio: 2 / 1;
  animation-name: birdFly;
  animation-timing-function: linear;
  animation-iteration-count: infinite;
  filter: drop-shadow(0 1px 3px rgba(0, 40, 80, 0.35));
}
.bird svg { width: 100%; height: 100%; overflow: visible; fill: #fff; }
.bird path {
  transform-box: fill-box;
  animation-duration: var(--flap, 0.6s);
  animation-timing-function: ease-in-out;
  animation-iteration-count: infinite;
}
.bird .wl { transform-origin: 100% 100%; animation-name: flapL; }
.bird .wr { transform-origin: 0% 100%;   animation-name: flapR; }
@keyframes flapL {
  0%, 100% { transform: rotate(24deg); }
  50%      { transform: rotate(-22deg); }
}
@keyframes flapR {
  0%, 100% { transform: rotate(-24deg); }
  50%      { transform: rotate(22deg); }
}
@keyframes birdFly {
  0%   { transform: translate(0, 0); }
  25%  { transform: translate(35vw, var(--dy)); }
  50%  { transform: translate(70vw, calc(var(--dy) * -1)); }
  75%  { transform: translate(105vw, var(--dy)); }
  100% { transform: translate(140vw, 0); }
}

/* ---------- DAUN FOREGROUND (SVG + blur + goyang) ---------- */
.fg-cluster {
  position: absolute;
  inset: 0;
  pointer-events: none;
  /* angin: seluruh gerombol miring pelan bolak-balik dari pojoknya */
  animation: fgWind 9s ease-in-out infinite alternate;
}
@keyframes fgWind {
  from { transform: rotate(calc(var(--wa, 1deg) * -1)); }
  to   { transform: rotate(var(--wa, 1deg)); }
}

.fg-leaf {
  position: absolute;
  aspect-ratio: 100 / 60;
  height: auto;
  overflow: visible;
  transform-origin: 2% 50%;                 /* pangkal daun = tempat "menempel" */
  transform: translate(0, -50%) rotate(var(--rot, 0deg));
  filter: blur(var(--b, 8px));              /* efek out-of-focus */
  animation: fgLeafSway 5s ease-in-out infinite;
  will-change: transform;
}
@keyframes fgLeafSway {
  0%, 100% { transform: translate(0, -50%) rotate(calc(var(--rot, 0deg) - var(--sw, 4deg))); }
  50%      { transform: translate(0, -50%) rotate(calc(var(--rot, 0deg) + var(--sw, 4deg))); }
}
.fl-body { fill: var(--c1); }
.fl-hi   { fill: var(--c2); opacity: 0.75; }
.fl-rib  { fill: none; stroke: var(--c3); stroke-width: 1.6; stroke-linecap: round; }
.fl-vein { fill: none; stroke: var(--c3); stroke-width: 0.9; stroke-linecap: round; opacity: 0.6; }

.fg-branch {
  position: absolute;
  top: 0;
  width: 38%;
  height: 14%;
  overflow: visible;
  filter: blur(2px);
}
.fg-branch.br-tl { left: 0; }
.fg-branch.br-tr { right: 0; }
.fg-branch path {
  fill: none;
  stroke: #4a3320;
  stroke-width: 5px;
  stroke-linecap: round;
  vector-effect: non-scaling-stroke;
}

.fg-flower {
  position: absolute;
  aspect-ratio: 1;
  height: auto;
  overflow: visible;
  transform: translate(-50%, -50%);
  filter: blur(var(--b, 4px));
  animation: fgFlowerSway 5s ease-in-out infinite;
}
.fg-flower ellipse { fill: #fff; }
.fg-flower circle  { fill: #ffd95a; }
@keyframes fgFlowerSway {
  0%, 100% { transform: translate(-50%, -50%) rotate(calc(var(--sw, 6deg) * -1)); }
  50%      { transform: translate(-48%, -52%) rotate(var(--sw, 6deg)); }
}

/* ============ BAD ENDING 1: "BAD ENDING INITIATED" ============ */
.bad-init {
  background: #080000;
  overflow: hidden;
  font-family: 'Courier New', monospace;
}
.bi-noise {
  position: absolute;
  inset: 0;
  pointer-events: none;
  background:
    repeating-linear-gradient(0deg, rgba(255, 30, 30, 0.09) 0 2px, transparent 2px 5px),
    repeating-linear-gradient(90deg, rgba(255, 60, 40, 0.05) 0 1px, transparent 1px 7px),
    radial-gradient(ellipse 80% 70% at 50% 50%, rgba(120, 0, 0, 0.45) 0%, rgba(20, 0, 0, 0.9) 100%);
  animation: biNoise 0.18s steps(2) infinite;
}
@keyframes biNoise {
  0%   { opacity: 0.85; transform: translate(0, 0); }
  100% { opacity: 1; transform: translate(-2px, 1px); }
}
.bad-init .glitch { z-index: 2; }
.bad-init .glitch i { background: rgba(255, 40, 40, 0.3); }

.bi-panel {
  position: relative;
  z-index: 5;
  width: min(86vw, 560px);
  text-align: center;
  color: #ff3a3a;
}
.bi-icon {
  width: clamp(54px, 9vw, 84px);
  fill: none;
  stroke: #ff2a2a;
  stroke-width: 3.2;
  stroke-linecap: round;
  stroke-linejoin: round;
  filter: drop-shadow(0 0 8px rgba(255, 30, 30, 0.95));
  animation: biBlink 1.1s steps(1) infinite;
}
.bi-icon circle { fill: #ff2a2a; stroke: none; }
@keyframes biBlink {
  0%, 100% { opacity: 1; }
  50%      { opacity: 0.55; }
}
.bi-title {
  margin-top: 18px;
  font-size: clamp(1.1rem, 3.8vw, 2.2rem);
  font-weight: bold;
  letter-spacing: 0.16em;
  text-shadow: 0 0 10px rgba(255, 40, 40, 0.95), 0 0 28px rgba(255, 30, 30, 0.6);
}
.bi-label {
  margin-top: 26px;
  font-size: clamp(0.6rem, 1.3vw, 0.8rem);
  letter-spacing: 0.22em;
  color: #ff7a6a;
}
.bi-bar {
  margin-top: 8px;
  height: 9px;
  border: 1px solid #ff3a3a;
  border-radius: 6px;
  overflow: hidden;
  background: rgba(60, 0, 0, 0.6);
  box-shadow: 0 0 10px rgba(255, 30, 30, 0.45);
}
.bi-fill {
  height: 100%;
  background: linear-gradient(90deg, #a10d0d, #ff2a2a);
  box-shadow: 0 0 12px rgba(255, 40, 40, 0.95);
}
.bi-scale {
  margin-top: 6px;
  display: flex;
  justify-content: space-between;
  font-size: clamp(0.55rem, 1vw, 0.7rem);
  letter-spacing: 0.1em;
  color: #ff7a6a;
  font-variant-numeric: tabular-nums;
}
.bi-log {
  position: absolute;
  z-index: 5;
  left: 4vw;
  bottom: 6vh;
  font-size: clamp(0.5rem, 1vw, 0.7rem);
  letter-spacing: 0.12em;
  line-height: 1.7;
  color: #d94a4a;
  text-align: left;
}
.bi-log p {
  opacity: 0;
  animation: biLog 0.3s steps(3) forwards;
  animation-delay: calc(0.4s + var(--i) * 0.6s);
}
@keyframes biLog { to { opacity: 1; } }

/* ============ BAD ENDING 2: city2 + warning + penutup ============ */
.bad-ending {
  background: #000;
  overflow: hidden;
  display: block;
}
.be-bg {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  user-select: none;
  -webkit-user-drag: none;
  filter: saturate(1.05) contrast(1.08) brightness(0.9);
}
/* menggelapkan latar saat teks penutup tampil supaya terbaca */
.be-dim {
  position: absolute;
  inset: 0;
  z-index: 3;
  pointer-events: none;
  background: rgba(0, 0, 0, 0.55);
  opacity: 0;
  transition: opacity 1.6s ease;
}
.be-dim.on { opacity: 1; }
.bad-ending .glitch { z-index: 3; }

.be-layer {
  position: absolute;
  inset: 0;
  z-index: 6;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 0 6vw;
  text-align: center;
}
.be-top { pointer-events: none; }

.be-warning {
  display: flex;
  align-items: center;
  gap: 18px;
  padding: 14px 30px;
  font-family: 'Courier New', monospace;
  font-weight: bold;
  font-size: clamp(0.85rem, 2.1vw, 1.5rem);
  letter-spacing: 0.12em;
  line-height: 1.35;
  text-align: left;
  color: #ff3a3a;
  background: rgba(30, 0, 0, 0.8);
  border: 2px solid #ff2a2a;
  text-shadow: 0 0 10px rgba(255, 40, 40, 0.95);
  animation: warnPulse 1.4s ease-in-out infinite;
}
.be-warn-icon {
  width: clamp(30px, 4.4vw, 52px);
  flex: none;
  fill: none;
  stroke: #ff2a2a;
  stroke-width: 3.6;
  stroke-linecap: round;
  stroke-linejoin: round;
  filter: drop-shadow(0 0 6px rgba(255, 30, 30, 0.95));
}
.be-warn-icon circle { fill: #ff2a2a; stroke: none; }
@keyframes warnPulse {
  0%, 100% { box-shadow: 0 0 14px rgba(255, 30, 30, 0.5), inset 0 0 12px rgba(255, 30, 30, 0.15); }
  50%      { box-shadow: 0 0 34px rgba(255, 40, 40, 0.95), inset 0 0 18px rgba(255, 30, 30, 0.3); }
}

.be-sub {
  margin-top: 22px;
  white-space: pre-line;
  font-size: clamp(0.85rem, 1.8vw, 1.25rem);
  line-height: 1.55;
  color: #f4f0e8;
  text-shadow: 0 2px 14px rgba(0, 0, 0, 0.95);
}

.be-thanks,
.be-edu {
  white-space: pre-line;
  color: #f4f0e8;
  text-shadow: 0 2px 18px rgba(0, 0, 0, 0.9), 0 0 24px rgba(255, 120, 70, 0.25);
}
.be-thanks {
  font-size: clamp(1rem, 2.2vw, 1.5rem);
  line-height: 1.35;
}
.be-edu {
  font-size: clamp(0.85rem, 1.7vw, 1.15rem);
  line-height: 1.55;
}
.end-text-enter-active,
.end-text-leave-active {
  transition: opacity 0.9s ease, transform 0.9s ease;
}
.end-text-enter-from,
.end-text-leave-to {
  opacity: 0;
  transform: translateY(12px);
}

.be-contact {
  margin-top: 34px;
  font-family: 'Permanent Marker', cursive;
  font-size: clamp(0.9rem, 1.8vw, 1.2rem);
  letter-spacing: 0.08em;
  color: #f4f0e8;
  background: rgba(30, 10, 8, 0.85);
  border: 1px solid rgba(255, 140, 90, 0.6);
  padding: 14px 40px;
  cursor: pointer;
  transition: border-color 0.2s, box-shadow 0.2s, color 0.2s, transform 0.2s;
}
.be-contact:hover {
  border-color: #ff9a5c;
  color: #ffb98a;
  box-shadow: 0 0 20px rgba(255, 110, 40, 0.45);
  transform: translateY(-2px);
}
.be-contact:focus-visible { outline: 2px solid #f4f0e8; outline-offset: 3px; }

/* tombol sekunder di bawah Contact Us: sengaja lebih pelan supaya tidak menyaingi CTA utama */
.be-restart {
  margin-top: 16px;
  font-family: 'Permanent Marker', cursive;
  font-size: clamp(0.75rem, 1.3vw, 0.95rem);
  letter-spacing: 0.08em;
  color: #8a8378;
  background: none;
  border: none;
  padding: 8px 14px;
  cursor: pointer;
  transition: color 0.2s ease;
}
.be-restart:hover { color: #f4f0e8; }
.be-restart:focus-visible { outline: 2px solid #f4f0e8; outline-offset: 3px; }

/* ---------- Contact Us: 3 card ---------- */
.be-contact-layer {
  z-index: 7;
  gap: 26px;
  overflow-y: auto;
  padding-top: 6vh;
  padding-bottom: 6vh;
}
.ct-title {
  font-size: clamp(1.1rem, 2.4vw, 1.7rem);
  letter-spacing: 0.06em;
  color: #f4f0e8;
  text-shadow: 0 2px 16px rgba(0, 0, 0, 0.9);
}
.ct-grid {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 20px;
  width: min(94vw, 900px);
}
.ct-card {
  flex: 1 1 220px;
  max-width: 270px;
  padding: 22px 16px 14px;
  text-align: center;
  background: rgba(30, 10, 8, 0.88);
  border: 1px solid rgba(255, 140, 90, 0.35);
  box-shadow: 0 0 26px rgba(255, 60, 30, 0.18), 0 8px 26px rgba(0, 0, 0, 0.6);
}
.ct-name {
  min-height: 2.6em;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: clamp(0.9rem, 1.5vw, 1.1rem);
  line-height: 1.3;
  color: #f4f0e8;
}
.ct-icons {
  margin-top: 16px;
  display: flex;
  justify-content: center;
  gap: 14px;
}
.ct-icon {
  width: 44px;
  height: 44px;
  padding: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  color: #f4f0e8;
  background: rgba(60, 18, 12, 0.7);
  border: 1px solid rgba(255, 140, 90, 0.5);
  cursor: pointer;
  transition: transform 0.2s ease, border-color 0.2s ease, box-shadow 0.2s ease, color 0.2s ease;
}
.ct-icon svg { width: 21px; height: 21px; fill: currentColor; }
.ct-icon svg.ct-stroke {
  fill: none;
  stroke: currentColor;
  stroke-width: 1.8;
  stroke-linecap: round;
  stroke-linejoin: round;
}
.ct-icon:hover {
  transform: translateY(-2px);
  color: #ffb98a;
  border-color: #ff9a5c;
  box-shadow: 0 0 16px rgba(255, 110, 40, 0.5);
}
.ct-icon:focus-visible { outline: 2px solid #f4f0e8; outline-offset: 3px; }
.ct-icon.off {
  opacity: 0.28;
  cursor: not-allowed;
  pointer-events: none;
}
.ct-toast {
  margin-top: 10px;
  height: 1.2em;
  font-family: 'Courier New', monospace;
  font-size: 0.72rem;
  letter-spacing: 0.1em;
  color: #6bff9c;
  opacity: 0;
  transition: opacity 0.3s ease;
}
.ct-toast.show { opacity: 1; }
.ct-back {
  font-family: 'Permanent Marker', cursive;
  font-size: 0.9rem;
  letter-spacing: 0.06em;
  color: #8a8378;
  background: none;
  border: 1px solid #444;
  padding: 8px 22px;
  cursor: pointer;
  transition: color 0.2s, border-color 0.2s;
}
.ct-back:hover { color: #f4f0e8; border-color: #f4f0e8; }

/* ============ STAGE 1 COMPLETED BANNER ============ */
.stage-banner-wrap {
  position: absolute; inset: 0; z-index: 14;
  display: flex; align-items: center; justify-content: center;
  pointer-events: none;
}
.stage-banner {
  position: relative;
  width: min(88vw, 640px);
  padding: 64px 28px 30px;
  text-align: center;
  font-family: 'Fredoka', 'Permanent Marker', sans-serif;
  border-radius: 34px;
  background: linear-gradient(180deg, #5cf08a 0%, #22b85a 100%);
  border: 5px solid #c9ffd9;
  box-shadow: 0 0 0 4px rgba(60, 220, 120, 0.35), 0 0 50px rgba(80, 255, 150, 0.75), 0 12px 30px rgba(0, 60, 30, 0.5);
  animation: sbPop 0.7s cubic-bezier(0.3, 1.5, 0.5, 1) both;
}
@keyframes sbPop { from { opacity: 0; transform: scale(0.6); } }
.sb-check {
  position: absolute; left: 50%; top: 0;
  transform: translate(-50%, -55%);
  width: clamp(64px, 12vw, 92px); aspect-ratio: 1;
  display: flex; align-items: center; justify-content: center;
  border-radius: 50%;
  background: #2fd16b;
  border: 5px solid #eaffef;
  box-shadow: 0 0 26px rgba(120, 255, 170, 0.9);
}
.sb-check svg { width: 58%; fill: none; stroke: #fff; stroke-width: 3.4; stroke-linecap: round; stroke-linejoin: round; }
.sb-line1, .sb-line2 {
  font-weight: 700; color: #fff;
  -webkit-text-stroke: 6px #16793f; paint-order: stroke fill;
  text-shadow: 0 3px 0 #14683a;
}
.sb-line1 { font-size: clamp(1.1rem, 3.6vw, 1.9rem); letter-spacing: 0.05em; }
.sb-line2 { margin-top: 4px; font-size: clamp(1.9rem, 6.6vw, 3.4rem); letter-spacing: 0.04em; }
.sb-leaf {
  position: absolute; bottom: 12px;
  width: clamp(34px, 7vw, 54px);
  fill: #d9ffe6; stroke: #1aa552; stroke-width: 4;
  stroke-linecap: round; stroke-linejoin: round;
}
.sb-leaf-l { left: 16px; transform: rotate(-8deg); }
.sb-leaf-r { right: 16px; transform: scaleX(-1) rotate(-8deg); }
.sb-spark {
  position: absolute; width: 14px; height: 14px; background: #fff;
  clip-path: polygon(50% 0, 62% 38%, 100% 50%, 62% 62%, 50% 100%, 38% 62%, 0 50%, 38% 38%);
  filter: drop-shadow(0 0 6px #fff);
  animation: sbTwinkle 1.4s ease-in-out infinite;
}
@keyframes sbTwinkle {
  0%, 100% { opacity: 0.3; transform: scale(0.6); }
  50% { opacity: 1; transform: scale(1.2); }
}

/* ============ MINI GAME 2: AI TRASH SORTING SCANNER ============ */
.trash-root {
  position: absolute; inset: 0; z-index: 8;
  pointer-events: none;
  font-family: 'Fredoka', 'Permanent Marker', sans-serif;
}
.t-belt {
  position: absolute; left: 0; right: 0;
  top: calc(50% - 7vmin); height: 14vmin;
  background: linear-gradient(180deg, #1b2a36, #0e1720);
  border-top: 4px solid #5b7a90; border-bottom: 4px solid #5b7a90;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.45);
  opacity: 0.92; overflow: hidden;
}
.t-belt::before {
  content: ''; position: absolute; inset: 0;
  background: repeating-linear-gradient(90deg, rgba(255, 255, 255, 0.1) 0 3px, transparent 3px 34px);
  background-size: 34px 100%; background-repeat: repeat-x;
  animation: beltMove 0.9s linear infinite;
}
@keyframes beltMove { to { background-position-x: 34px; } }
.t-item {
  position: absolute; z-index: 4;
  width: 11vmin; aspect-ratio: 1;
  padding: 0; border: none; background: none;
  transform: translate(-50%, -50%);
  cursor: pointer; pointer-events: auto;
  transition: left 0.05s linear;
  -webkit-tap-highlight-color: transparent;
}
.t-item svg { width: 100%; height: 100%; display: block; filter: drop-shadow(0 4px 6px rgba(0, 0, 0, 0.5)); }
.t-item.selected { z-index: 6; }
.t-item.selected svg { animation: tHover 0.9s ease-in-out infinite alternate; }
@keyframes tHover { to { transform: translateY(-1.2vmin); } }
.t-item.scanning::before,
.t-item.scanned::before {
  content: ''; position: absolute; inset: -10%;
  border: 2px solid #4fc3ff; border-radius: 8px;
  box-shadow: 0 0 12px rgba(79, 195, 255, 0.8), inset 0 0 10px rgba(79, 195, 255, 0.25);
  pointer-events: none;
}
.t-laser {
  position: absolute; left: -25%; right: -25%; top: 0; height: 3px;
  background: #ff3b3b; box-shadow: 0 0 10px 3px rgba(255, 60, 60, 0.9);
  opacity: 0; pointer-events: none; z-index: 2;
}
.t-item.scanning .t-laser { opacity: 1; animation: laserSweep 0.45s linear forwards; }
@keyframes laserSweep {
  from { top: 0; background: #ff3b3b; box-shadow: 0 0 10px 3px rgba(255, 60, 60, 0.9); }
  to { top: 100%; background: #4fc3ff; box-shadow: 0 0 10px 3px rgba(79, 195, 255, 0.9); }
}
.t-tag {
  position: absolute; left: 50%; bottom: calc(100% + 14px);
  transform: translateX(-50%);
  white-space: nowrap; padding: 4px 10px;
  font-family: 'Courier New', monospace; font-weight: bold;
  font-size: clamp(0.6rem, 1.3vw, 0.85rem); letter-spacing: 0.08em;
  color: #8fe3ff;
  background: rgba(4, 22, 36, 0.92);
  border: 1px solid rgba(79, 195, 255, 0.8);
  box-shadow: 0 0 12px rgba(79, 195, 255, 0.5);
  animation: txFade 0.25s ease both;
}
.t-bins {
  position: absolute; left: 50%; bottom: 4vh; z-index: 5;
  transform: translateX(-50%);
  width: min(94vw, 680px);
  display: flex; gap: 2.5vw;
  pointer-events: none;
}
.t-bin {
  position: relative; flex: 1;
  padding: 14px 8px 10px;
  font-family: inherit; color: #eaf6ff; text-align: center;
  cursor: pointer; pointer-events: auto;
  background: rgba(12, 44, 74, 0.85);
  border: 2px solid var(--bc); border-radius: 14px;
  box-shadow: 0 0 14px var(--bcg), 0 6px 20px rgba(0, 20, 40, 0.4);
  transition: transform 0.15s ease;
}
.t-bin:hover { transform: translateY(-3px); }
.t-bin b { display: block; font-size: clamp(0.8rem, 1.9vw, 1.15rem); letter-spacing: 0.06em; color: var(--bc); }
.t-bin small { font-size: 0.68rem; letter-spacing: 0.1em; opacity: 0.75; }
.t-bin.ok { animation: binOk 0.45s ease; }
.t-bin.bad { animation: binBad 0.45s ease; border-color: #ff5a4a; }
@keyframes binOk { 50% { transform: scale(1.08); box-shadow: 0 0 34px rgba(80, 255, 150, 0.95); } }
@keyframes binBad {
  0%, 100% { transform: translateX(0); }
  20% { transform: translateX(-8px); }
  40% { transform: translateX(8px); }
  60% { transform: translateX(-6px); }
  80% { transform: translateX(6px); }
}
.t-plus {
  position: absolute; left: 50%; top: 0;
  transform: translate(-50%, -50%);
  font-weight: 700; font-size: clamp(1.1rem, 3vw, 1.8rem);
  color: #1fc25c; -webkit-text-stroke: 4px #fff; paint-order: stroke fill;
  animation: plusUp 0.9s ease-out forwards;
}
.t-toast {
  position: absolute; left: 50%; top: 24vh; z-index: 6;
  transform: translateX(-50%);
  padding: 6px 16px;
  font-family: 'Courier New', monospace; font-weight: bold; letter-spacing: 0.1em;
  color: #ff6a5a;
  background: rgba(30, 6, 6, 0.9);
  border: 1px solid rgba(255, 90, 70, 0.7);
  opacity: 0; transition: opacity 0.2s ease;
}
.t-toast.show { opacity: 1; }
.t-hint { top: 17vh; bottom: auto; }

/* ============ MINI GAME 3: DRONE ============ */
.drone-root {
  position: absolute; inset: 0; z-index: 8;
  pointer-events: none;
  font-family: 'Fredoka', 'Permanent Marker', sans-serif;
}
.d-radar {
  position: absolute; inset: 0; overflow: hidden;
  background:
    repeating-linear-gradient(0deg, rgba(80, 255, 170, 0.07) 0 1px, transparent 1px 6vmin),
    repeating-linear-gradient(90deg, rgba(80, 255, 170, 0.07) 0 1px, transparent 1px 6vmin),
    radial-gradient(ellipse 80% 70% at 50% 55%, rgba(30, 60, 30, 0.35), rgba(0, 12, 8, 0.75));
  animation: radarZoom 1.4s ease-out both;
}
@keyframes radarZoom { from { transform: scale(1.6); opacity: 0; } }
.d-sweep {
  position: absolute; left: 50%; top: 55%;
  width: 150vmax; height: 150vmax;
  margin: -75vmax 0 0 -75vmax;
  background: conic-gradient(from 0deg, rgba(80, 255, 170, 0.28), transparent 25%);
  animation: raysSpin 5s linear infinite;
}
.d-grid { position: absolute; inset: 0; }
.d-plot {
  position: absolute; z-index: 4;
  width: 9vmin; aspect-ratio: 1;
  padding: 0; border: none; background: none;
  transform: translate(-50%, -50%);
  cursor: pointer; pointer-events: auto;
  -webkit-tap-highlight-color: transparent;
}
.d-dot {
  position: absolute; inset: 30%;
  border-radius: 50%;
  background: #8a6a3f;
  border: 2px solid #c9a56a;
  box-shadow: 0 0 10px rgba(201, 165, 106, 0.6);
  animation: dotBlink 1.6s ease-in-out infinite;
}
@keyframes dotBlink { 50% { opacity: 0.45; transform: scale(0.85); } }
.d-plot:hover .d-dot { box-shadow: 0 0 18px rgba(255, 220, 140, 0.95); }
.d-pod {
  position: absolute; left: 50%; top: 50%;
  width: 22%; height: 22%; border-radius: 50%;
  background: #6bffb0; box-shadow: 0 0 12px 4px rgba(107, 255, 176, 0.9);
  transform: translate(-50%, -400%); opacity: 0;
}
.d-plot.launching .d-pod { animation: podDrop 0.35s ease-in forwards; }
@keyframes podDrop {
  from { transform: translate(-50%, -600%); opacity: 1; }
  to { transform: translate(-50%, -50%); opacity: 1; }
}
.d-tree {
  position: absolute; inset: -30% -30% 0 -30%;
  width: 160%; height: 160%;
  transform: scale(0); transform-origin: 50% 90%;
  filter: drop-shadow(0 0 10px rgba(107, 255, 156, 0.9));
}
.d-plot.grown { cursor: default; pointer-events: none; }
.d-plot.grown .d-dot { display: none; }
.d-plot.grown .d-tree { animation: treePop 0.55s cubic-bezier(0.3, 1.6, 0.5, 1) forwards; }
@keyframes treePop { from { transform: scale(0); } to { transform: scale(1); } }
.d-signal { position: absolute; left: 50%; top: 10%; width: 0; height: 0; opacity: 0; }
.d-plot.grown .d-signal { opacity: 1; }
.d-signal i {
  position: absolute; left: -3vmin; top: -3vmin;
  width: 6vmin; height: 6vmin; border-radius: 50%;
  border: 2px solid rgba(107, 255, 176, 0.9);
  animation: sigPing 1.8s ease-out infinite;
}
.d-signal i:nth-child(2) { animation-delay: -0.9s; }
@keyframes sigPing { from { transform: scale(0.3); opacity: 1; } to { transform: scale(1.8); opacity: 0; } }

/* ============ ALL STAGES + HAPPY ENDING ============ */
.sb-small { font-size: clamp(1.1rem, 3.6vw, 1.9rem); }
.he-dim {
  position: absolute; inset: 0; z-index: 13; pointer-events: none;
  background: radial-gradient(ellipse 80% 70% at 50% 50%, rgba(0, 25, 15, 0.7), rgba(0, 10, 6, 0.92));
  opacity: 0; transition: opacity 1.6s ease;
}
.he-dim.on { opacity: 1; }
.he-layer, .he-contact { z-index: 15; }
.he-title {
  display: flex; align-items: center; gap: 18px;
  padding: 14px 30px;
  font-family: 'Courier New', monospace; font-weight: bold;
  font-size: clamp(0.85rem, 2.1vw, 1.5rem);
  letter-spacing: 0.12em; line-height: 1.35; text-align: left;
  color: #6bff9c;
  background: rgba(4, 30, 16, 0.8);
  border: 2px solid #6bff9c;
  text-shadow: 0 0 10px rgba(80, 255, 140, 0.95);
  animation: teaserPulse 1.6s ease-in-out infinite;
}
.he-title svg {
  width: clamp(30px, 4.4vw, 52px); flex: none;
  fill: none; stroke: #6bff9c; stroke-width: 3.2;
  stroke-linecap: round; stroke-linejoin: round;
}
@media (max-width: 560px) {
  .d-plot { width: 12vmin; }
  .d-radar { background-size: auto; }
  .he-title { gap: 12px; padding: 12px 16px; letter-spacing: 0.06em; }
}

@media (max-width: 560px) {
  .stage-banner { padding: 54px 18px 26px; border-radius: 26px; }
  .sb-leaf { width: 32px; bottom: 8px; }
  .t-bins { width: 94vw; gap: 6px; bottom: 3vh; }
  .t-bin { padding: 10px 4px 8px; border-width: 1px; border-radius: 10px; }
  .t-bin b { font-size: 0.68rem; letter-spacing: 0.02em; }
  .t-bin small { font-size: 0.6rem; }
  .t-item { width: 14vmin; }
  .t-tag { font-size: 0.55rem; letter-spacing: 0.02em; }
}

@media (max-width: 560px) {
  .be-warning { padding: 12px 16px; gap: 12px; letter-spacing: 0.06em; }
  .bi-log { display: none; }
  .be-contact-layer { justify-content: flex-start; }
  .ct-card { flex-basis: 100%; }
}

/* preserve line breaks from translated copy */
.hp-text div, .hint-text, .eco-info p, .g-hint span, .g-result p,
.g-title, .he-title span, .be-warning span, .call-sub { white-space: pre-line; }

.lang-toggle {
  position: absolute;
  right: 2vw;
  bottom: 2.5vh;
  z-index: 30;
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 14px;
  font-family: 'Courier New', monospace;
  font-weight: bold;
  font-size: 0.8rem;
  letter-spacing: 0.12em;
  color: #8a8378;
  background: rgba(20, 8, 8, 0.8);
  border: 1px solid rgba(255, 140, 90, 0.4);
  border-radius: 999px;
  cursor: pointer;
  transition: border-color 0.2s, box-shadow 0.2s;
}
.lang-toggle:hover { border-color: #ff9a5c; box-shadow: 0 0 14px rgba(255, 110, 40, 0.35); }
.lang-toggle:focus-visible { outline: 2px solid #f4f0e8; outline-offset: 3px; }
.lang-toggle span { transition: color 0.2s, text-shadow 0.2s; }
.lang-toggle span.on { color: #ff9a5c; text-shadow: 0 0 8px rgba(255, 140, 90, 0.8); }
.lang-toggle i { width: 1px; height: 14px; background: rgba(255, 140, 90, 0.4); }

/* ============ VOLUME (kiri bawah) ============ */
.vol-ctrl {
  position: absolute;
  left: 2vw;
  bottom: 2.5vh;
  z-index: 30;
  display: flex;
  align-items: center;
  gap: 0;
  padding: 4px;
  background: rgba(20, 8, 8, 0.8);
  border: 1px solid rgba(255, 140, 90, 0.4);
  border-radius: 999px;
  transition: border-color 0.2s, box-shadow 0.2s, gap 0.25s ease, padding 0.25s ease;
}
.vol-ctrl:hover,
.vol-ctrl:focus-within {
  gap: 8px;
  padding-right: 14px;
  border-color: #ff9a5c;
  box-shadow: 0 0 14px rgba(255, 110, 40, 0.35);
}
.vol-btn {
  width: 34px;
  height: 34px;
  padding: 7px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #ff9a5c;
  background: none;
  border: none;
  border-radius: 50%;
  cursor: pointer;
}
.vol-btn svg {
  width: 100%;
  height: 100%;
  fill: none;
  stroke: currentColor;
  stroke-width: 1.8;
  stroke-linecap: round;
  stroke-linejoin: round;
}
.vol-btn:focus-visible { outline: 2px solid #f4f0e8; outline-offset: 2px; }
.vol-range {
  width: 0;
  opacity: 0;
  accent-color: #ff9a5c;
  cursor: pointer;
  transition: width 0.25s ease, opacity 0.2s ease;
}
.vol-ctrl:hover .vol-range,
.vol-ctrl:focus-within .vol-range {
  width: 90px;
  opacity: 1;
}

@media (prefers-reduced-motion: reduce) {
  .st-glitch-out, .st-glitch-out .scene-bg, .glitch-burst i, .glitch-burst::before { animation: none; }
  .eco-start { animation: none; }
  .g-float, .g-orb, .g-rays, .call-ring, .g-hint svg { animation: none; }
  .stage-banner, .sb-spark, .t-belt::before, .t-item.selected svg,
  .t-item.scanning .t-laser, .t-bin.ok, .t-bin.bad, .t-plus { animation: none; }
  .d-radar, .d-sweep, .d-dot, .d-plot.launching .d-pod,
  .d-plot.grown .d-tree, .d-signal i, .he-title { animation: none; }
  .bird, .bird path, .gw-cam, .gw-sheen, .glint, .gw-sunglow, .gw-flare,
  .gw-rays, .ray, .fleaf, .mote, .fg-cluster, .fg-leaf, .fg-flower,
  .scroll-teaser.show, .tx-neon.show,
  .rc-fix, .be-warning, .bi-icon, .bi-noise { animation: none; }
}
</style>