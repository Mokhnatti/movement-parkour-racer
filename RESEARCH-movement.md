# Ресёрч: архитектура движения (UE 5.8, паркур-рейсер) — 18.06.2026

Глубокий веб-ресёрч (2024-2026) по этапу 1. Источники в тексте. Confidence помечен.

## 1. ДВИЖЕНИЕ: CMC vs Custom vs Mover 2.0
**Вывод — две дороги, выбор по сроку:**
- **Шипить <12 мес / минимум риска:** subclass `UCharacterMovementComponent` + custom-режимы (`MOVE_Custom`/`PhysCustom`) для вал-рана/дэша/слайда. Так делали ВСЕ UE-игры где есть данные (Echo Point Nova — модифицировали CMC; Ghostrunner — расширяли CMC). Референс-архитектура: GitHub `tjysdsg/ExtendedCharacterMovement` (CMC subclass, wall-run+jump+momentum, network-predicted).
- **2+ года / детерминизм критичен:** **Mover 2.0** — в UE 5.8 стал **Beta, сеть готова (клиент-предсказание+реконсиляция)**. Архитектурно создан под кастомные «глаголы» движения и детерминированную ресимуляцию (inputs/sim/aux как явные структуры), async fixed-tick. «Правильный» долгосрочный ответ, НО будут breaking changes 5.8→5.9.
- Neon White = Unity (не UE), только для фила-референса. Полностью кастомный UMovementComponent оправдан редко.

## 2. ДЕТЕРМИНИЗМ / ФИКС-ТАЙМСТЕП
- Полный кроссплатформенный детерминизм в стоке UE5 **недостижим** (float, корректировки). Цель — **«достаточно честно»**: фикс-таймстеп + clamp + серверная валидация лидербордов.
- Способы: **Mover async fixed-tick** ИЛИ **manual accumulator** (Gaffer "Fix Your Timestep") вокруг CMC, интерполяция визуала.
- ⚠️ Substepping влияет ТОЛЬКО на физику, НЕ на Actor Tick (нужен AddCustomPhysics).
- Настройки: **Smoothed Frame Rate Range** (НЕ Use Fixed Frame Rate — он замедляет игру при просадке), Max Physics Delta Time, Max Substep Delta Time × Max Substeps. Тест `t.MaxFPS 30/60/120`.
- 🚨 **ГЛАВНЫЙ РИСК честности:** CMC `AddForce` и based-velocity (платформы) **НЕ frame-rate independent** → дуги прыжков и эйр-контроль разные на разном железе. Главная причина за фикс-таймстеп-обёртку или Mover.

## 3. ПРИЗРАКИ / РЕПЛЕИ
- **Запись transform/pose + интерполяция** (надёжно, путь точный), НЕ input-replay (хрупко из-за FP-дрейфа). Input-запись держать ТОЛЬКО как серверную валидацию/античит.
- Готовое: бесплатный **«Ghost Replay System [Free for UE5]»** (Epic forums, авг 2025) — гонка против своего лучшего лапа, pose+transform с квантованием/сжатием, мульти-актёр, работает с Lyra. Самый релевантный ресурс.
- UE Replay System (DemoNetDriver) — тяжёл для одного призрака.
- Лидерборды: файлы реплеев подделываемы → валидировать серверно.

## 4. CCD / КОЛЛИЗИИ НА СКОРОСТИ
- Swept moves всегда (`bSweep=true` на ручных перемещениях; CMC/PMC свипуют сами), **per-body CCD на быстром актёре** (`SetUseCCD(true)`), `bForceSubStepping` для дэшей/снарядов.
- CMC свипует per-tick, но на высокой скорости+низком FPS displacement за тик может пробить тонкую стену → держать фикс физ-тик.
- Точные `p.Chaos.CCD*` cvar не подтверждены — проверить в редакторе + Chaos Visual Debugger.

## 5. ВАЛ-РАН
- **Custom movement mode (`PhysCustom`)**, НЕ Tick-хак (хак фигтит CMC и хрупок для предсказания).
- Техника (ssgames tutorial, окт 2024): сфера-трейс ±50 по бокам, z-компонента нормали отсекает пол/рампы, направление = поворот нормали стены на 90°, friction-затухание, наклон камеры ~0.15с (для 3-лица — наклон spring arm/FOV-кик, не roll).
- Готовое: `tjysdsg/ExtendedCharacterMovement` (лучшая архитектура), AEDude/Unreal_Engine_5_Parkour_System, Fab-паки (TNT Parkour, Dynamic Parkour Trace, Ultimate Parkour). Game Animation Sample (Motion Matching) для анимслоя 3-лица.

## 6. ТЮНИНГ-СТЕНД ФИЛА
- Params в `UDataAsset` + **`UDeveloperSettings` backed by CVars** (`DeveloperSettingsBackedByCVars`) → live-правка в консоли без рекомпиляции + Project Settings + INI-персист (VCS-friendly). Паттерн Tom Looman.
- **Пресеты:** Console Variables Editor (сохраняемые наборы CVar) ИЛИ N DataAsset'ов («Default»/«Aggressive»/«Floaty») со сменой активного указателя.
- **Метрики прогона:** Visual Logger (пространственное — движение/коллизии, timestamped), Gameplay Debugger (live on-screen), **CSV Profiler `CSV_CUSTOM_STAT`** (время/макс.скорость/кол-во столкновений → CSV → diff прогонов).
- Готового «тюнинг-дашборда» плагина НЕТ — собирается из этого.
- ⚠️ **Грабля:** редактирование DataAsset во время PIE **ПЕРСИСТИТ после PIE** → snapshot/restore если нужны session-only твики.
- Фил-референс: Celeste Player.cs + доклад Maddy Thorson GDC 2017.

## 7. ОФИЦИАЛЬНЫЙ MCP в UE 5.8 — ПОДТВЕРЖДЕНО (с важной поправкой)
- Официальный плагин Epic `ModelContextProtocol` («Unreal MCP»), статус **Experimental**. MCP-сервер в редакторе, `http://127.0.0.1:8000/mcp`, loopback, HTTP+SSE, без auth локально. Meta-тулы `list_toolsets`/`describe_toolset`/`call_tool`. Клиенты: Claude Code, Cursor, VS Code, Gemini, Codex.
- **Умеет из коробки:** спавн/настройка актёров, освещение, material instances, инспекция Slate, автотесты. Тулсеты SceneTools/ActorTools/MaterialInstanceTools/ObjectTools + AttributeSet (GAS). Расширяемо своим ToolsetDefinition (Python/C++), без авторинга Blueprint.
- 🚨 **ОПРОВЕРГНУТО для официального плагина:** PIE start/stop, захват скриншота, выполнение консольных команд — **НЕ встроены**. Это умеют СТОРОННИЕ серверы (remiphilippe/mcp-unreal ~49 тулзов с PIE; ChiR24 ~23 со скринами; один сервер 370+).
- **Вывод для нас:** чтобы агент ИГРАЛ уровень + скринил + читал метрики (для «крутить фил») — писать свой ToolsetDefinition (PIE/скрин/CVar) ИЛИ ставить сторонний сервер. Официальный — для скриптовой сборки сцены + автотестов.

## ⭐ ИТОГОВАЯ РЕКОМЕНДАЦИЯ ПО АРХИТЕКТУРЕ
1. **Движение:** <12 мес → CMC-subclass + custom-режимы (референс ExtendedCharacterMovement). 2+ года/детерминизм критичен → прототипировать **Mover 2.0** (Beta в 5.8). Прототип Mover можно пощупать сейчас, но готовься к churn 5.8→5.9.
2. **Честность:** фикс-таймстеп (Mover async ИЛИ manual accumulator вокруг CMC), clamp Max Physics Delta Time, Smoothed Frame Rate Range, тест `t.MaxFPS 30/60/120`. Детерминизм — «достаточно», лидерборды валидировать серверно.
3. **Призраки:** transform/pose-запись + интерполяция (старт с бесплатного Ghost Replay System), input-запись — только серверная валидация.
4. **Коллизии:** свипы, per-body CCD на игроке, sub-stepping для дэшей, Chaos Visual Debugger.
5. **Тюнинг-стенд:** DataAsset + UDeveloperSettings(CVars) + Console Variables Editor (пресеты) + CSV Profiler/Visual Logger (метрики). Помнить про PIE-DataAsset-персист.
6. **MCP:** официальный для сборки сцены/автотестов; для «агент играет/скринит/читает метрики» — свой ToolsetDefinition или сторонний сервер.

**Честные дыры:** нет first-party постмортемов Ghostrunner/Severed Steel (CMC — вторичные источники); нет подтверждённого шипнутого тайтла на Mover; точные `p.Chaos.CCD` cvar не подтверждены; PIE/скрин в официальном MCP — скорее отсутствуют встроенно. Главный практический риск честной гонки — frame-rate-зависимое поведение CMC force/based-velocity.
