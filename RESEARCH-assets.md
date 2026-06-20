# RESEARCH-assets.md — бесплатные ассеты/инструменты + лицензии (глубокий ресёрч 20.06.2026)

> Полный отчёт глубокого ресёрча (deep-research: 111 агентов, 28 источников, 88 утверждений → 25 проверено голосованием 3/3, 24 подтверждено, 1 опровергнуто).
> Сжатая версия живёт в PLAN.md §21.7 и §14.3-бис. Здесь — полные находки с пруфами.
> 🚨 ГЛАВНОЕ ПРАВИЛО: перед ЛЮБЫМ GitHub/Fab-ассетом В КОД — проверять лицензию. **GPL/AGPL = ЯД** (заставит открыть исходники всей игры), **MIT/Apache/BSD/CC0 = чисто**, **«no license» = по умолчанию копирайт = НЕЛЬЗЯ**.

## TL;DR
Для соло-UE5.8-паркур-рейсера со Stellar-Blade-стилизацией сильнейшая бесплатная коммерческая база — first-party Epic: **GASP** (500+ Motion-Matching анимаций движения/traversal на UE-манекене) + **MetaHuman** (ядро MIT, стилизация реальна). Кастом-мувмент бери MIT (**DaftMover** на Mover 2.0, **ALSXT**) — НЕ GPL ExtendedCharacterMovement. Steam-лидерборды — MIT **SteamSAL**. Текстуры/whitebox — CC0 (**Poly Haven / ambientCG / Kenney**).

---

## ✅ ПОДТВЕРЖДЕНО (бесплатно + коммерция OK)

### Движение / анимация
- **GASP — Game Animation Sample Project** (3/3). 500+ AAA-анимаций, нативно на UE5-манекене: Motion-Matching локомоция (walk/run/jump/fall) + traversal (vault, ledge-scale/climb, drop) + slide (с 5.7, Mover). Лицензия: «only licensed for use with Unreal Engine, but can be used in commercial projects» → коммерция ОК. ⚠️ «паркур» — аспирационный маркетинг Epic («think parkour»); из коробки вал-ран/прыжок-от-стены НЕТ, traversal частично montage-driven. Ретаргет нужен только для не-манекен персонажей.
  - https://www.fab.com/listings/880e319a-a59e-4ed2-b268-b32dac7fa016
  - https://www.unrealengine.com/en-US/blog/game-animation-sample

### Персонаж / лицо
- **MetaHuman** (2/1, verifier high). Полный фреймворк цифровых людей; с 5.8 ядро **RigLogic + DNA (OpenRigLogic) под MIT** → коммерция. Стилизация под Stellar Blade реальна — кейс Epic «Daughter of the Inner Stars» (но потребовал внешнего Blender-моделинга). Широкая лицензия персонажей — free при доходе <$1M/год, юзабельно в любом движке. Встроен в UE 5.6+ (отдельной загрузки нет — галка «MetaHuman Creator Core Data» при установке UE).
  - https://www.metahuman.com/
  - https://www.unrealengine.com/spotlights/crafting-stylized-digital-humans-on-daughter-of-the-inner-stars-with-metahuman-and-ue5
  - ⚠️ MIT — только ядро RigLogic/DNA, НЕ весь стек; из коробки Creator склонен к фотореализму.

### Steam-бэкенд
- **SteamSAL** (HakanCopur/steam-achievements-leaderboards) (3/3). MIT, blueprint-first, async, без C++ → Steam Leaderboards/Achievements/Stats. Под PLAN §23/§24.1. ⚠️ заявлен UE5.0-5.6, на 5.8 пересобрать (открытый). Оборачивает Steamworks SDK (отдельная free-для-Steam-devs лицензия).
  - https://github.com/HakanCopur/steam-achievements-leaderboards

### CC0 (коммерция, БЕЗ атрибуции — подтверждено 3/3)
- **Poly Haven** — текстуры/HDRI/модели. «any purpose, including commercial», «no credit or attribution». https://polyhaven.com/textures
- **ambientCG** — PBR-текстуры/материалы, CC0. https://ambientcg.com/
- **Kenney** — game-ассеты, CC0, идеально под low-poly whitebox. https://kenney.nl/assets
- Сводный список CC0: https://github.com/madjin/awesome-cc0

---

## 🟡 MIT, но СЫРОЕ (кастом-мувмент — замена GPL-варианту)
- **DaftMover** (3/3). MIT, кастом-мувмент на **Mover 2.0** (= наш «Mover 2.0 кандидат» §21.1), UE5.4+. ⚠️ work-in-progress, заточен под factory-game, фит к паркуру ограничен. https://github.com/daftsoftware/DaftMover
- **ALSXT** (3/3). MIT (AGPL-карваут только при >$1M / megacorp — соло не касается). Модульный GAS-персонаж на ALS-Refactored, глубокая рантайм-кастомизация + mesh painting; реализованы Slide/Flip. ⚠️ «under heavy development, не production-ready»; **вал-ран/прыжок-от-стены/vault ещё НЕ готовы** (в планах); требует UE из исходников + зависимость ALS-Refactored (её лицензию тоже проверить). https://github.com/Voidware-Prohibited/ALSXT

---

## ⛔ НЕ БРАТЬ
- **ExtendedCharacterMovement** (3/3). Лицензия **GPL-3.0** (полный текст GPLv3, не LGPL, не dual). Включение в игру → обязан открыть исходники ВСЕЙ игры под GPL = смерть платному Steam-релизу. Только читать как идею, код НЕ копировать. https://github.com/tjysdsg/ExtendedCharacterMovement
- **asee5g/Character-Customization** (3/3 на «нет лицензии»; функционал ОПРОВЕРГНУТ 1/2). license=null → по умолчанию копирайт, прав на использование НЕТ. И заявленный рантайм-кастомайзер не подтверждён. https://github.com/asee5g/Character-Customization
- **itch.io киберпанк-UI киты** (напр. djy66 Cyberpunk HUD/UI Kit) (3/3). Коммерция разрешена, НО это **только картинки** (прозрачные PNG + JPG-превью), без скриптов/UMG/проекта → UMG собирать руками. Часть «Created with AI assistance» → копирайт мутный в части юрисдикций. https://djy66.itch.io/cyberpunk-hud-ui-kit-vol-1

---

## ⚠️ РЕСЁРЧ НЕ ЗАКРЫЛ (добор на обычный чат / позже)
1. График раздач Fab/Epic 2026 + что бесплатно сейчас с дедлайном (частично закрыто вручную 20.06 → PLAN §14.3-бис: Polyart/DOCKS/White castle до 30.06).
2. Текущие термы Quixel/Megascans (Megascans переехал в Fab, условия менялись — перепроверить).
3. Конкретные киберпанк/sci-fi city киты, неон-материалы, City Sample.
4. VFX скорости (стрик-линии, ветер, Niagara speed-примеры).
5. Аудио/SFX (ветер, шаги, ambient).
6. Ghost-replay + фикс-таймстеп/детерминизм тулинг (у нас есть Ghost Replay System §21.3, но шире не копали).
7. Подтверждение, что GASP/SteamSAL/DaftMover/ALSXT заводятся именно на UE5.8 (все forward-portable, но turnkey не подтверждён).

## Версии (на момент ресёрча)
GASP — UE5.4-5.7 (не нативно 5.8); SteamSAL — UE5.0-5.6; DaftMover — 5.4+; ALSXT — UE из исходников. Все переносимы вперёд, но не turnkey на 5.8.
