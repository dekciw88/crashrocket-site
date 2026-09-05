# Crash Rocket — веб-витрина (web2app преленд)

Статус: спека готова к реализации. Автор: Cowork-сессия 02.09.2026. Исполнитель: Claude Code.

## 1. Зачем

Льём платный трафик (Meta, EU) на приложение **Crash Rocket** (`games.crash.rocket`, Google Play).
Схема web2app: объявление → эта страница → кнопка → Google Play. Страница нужна, чтобы
(а) прогреть и отфильтровать клик до стора, (б) поставить свой пиксель и отдать Meta событие клика,
(в) прокинуть sub_id/UTM в `referrer` Play-ссылки для сквозной атрибуции.

Референс конкурента (сеть PiKoYa, go.games4slots.com): FB-креатив с геймплеем + текст
«Free to play — No real money», одна большая жёлтая кнопка, дисклеймер в каждом блоке.
Мы берём их скелет, но **без каталога чужих игр** — страница рекламирует одну апку.

## 2. Где живёт

- Репозиторий: `dekciw88/crashrocket-site` (публичный, GitHub Pages уже включён,
  там лежит `/privacy` — не трогать).
- Ветка: `feat/landing` → PR в `main`. Прямо в `main` не коммитить.
- Файлы:
  - `index.html` — вся страница, один файл, inline CSS + inline JS, без сборки и без фреймворков.
  - `assets/icon-512.png`, `assets/shot-1.png`, `assets/shot-2.png`, `assets/shot-3.png`,
    `assets/gameplay.mp4`, `assets/gameplay-poster.jpg` — медиа (см. §7, пока плейсхолдеры).
  - `log.md` — дописать запись о том, что сделано.
- Проверка после деплоя: `https://dekciw88.github.io/crashrocket-site/`
  (Pages собирается из `main`, кастомный домен — отдельная задача, не сейчас).

## 3. Конфиг (верх `<script>`, всё редактируемое — в одном месте)

```js
const CONFIG = {
  appName: 'Crash Rocket',
  packageId: 'games.crash.rocket',
  playUrl: 'https://play.google.com/store/apps/details?id=games.crash.rocket',
  metaPixelId: 'PIXEL_ID_TODO',       // пустая строка = пиксель не подключать
  rating: '4.8', installs: '100K+',   // текст под названием, править руками
  bonusCoins: '1,000,000',
  privacyUrl: './privacy',
  defaultLang: 'en'
};
```

## 4. Структура страницы (mobile-first, порядок сверху вниз)

1. **Шапка**: иконка 64px, `Crash Rocket`, строка `★ 4.8 · 100K+ installs · Free`, справа кнопка `Get it on Google Play` (бейдж).
2. **Hero**: телефон-мокап с `gameplay.mp4` (autoplay muted loop playsinline, `poster=gameplay-poster.jpg`,
   `preload=metadata`); под ним H1 `Launch. Cash out. Repeat.` и подзаголовок
   `Free casual crash game — no real money, just the rush.`
3. **Бонус-крючок**: карточка `🎁 1,000,000 coins on install` + кнопка `Claim & Download`.
4. **Три фичи** (иконка + строка): `Daily missions & streaks` · `Planet map, 50+ levels` · `Play offline`.
5. **Скриншоты**: горизонтальный скролл-ряд из 3 картинок 9:16 (`scroll-snap`).
6. **Отзывы**: 3 карточки в стиле стора (имя, ★★★★★, одна фраза). Тексты — в COPY, помечены как placeholder.
7. **Финальный CTA**: повтор кнопки `Download free`.
8. **Футер**: дисклеймер (см. §6), ссылки `Privacy` (`CONFIG.privacyUrl`), `Contact`.
9. **Sticky-кнопка** внизу экрана (`position: fixed`), появляется после прокрутки hero
   (IntersectionObserver), на десктопе скрыта.

Все кнопки — один класс `.cta`, один обработчик `onCtaClick()`.

## 5. Логика кнопки и трекинг

```
onCtaClick():
  1. fbq('track', 'Lead')                       // если pixel подключён
  2. window.location.href = buildPlayUrl()
```

`buildPlayUrl()`:
- берёт из `location.search` параметры `utm_source, utm_medium, utm_campaign, utm_content, utm_term, sub1..sub5, fbclid`;
- собирает строку `k=v&k=v` из непустых, `encodeURIComponent` каждого значения;
- возвращает `CONFIG.playUrl + '&referrer=' + encodeURIComponent(строка)`;
- если параметров нет — просто `CONFIG.playUrl`.

Пиксель: если `CONFIG.metaPixelId` непустой — вставить стандартный сниппет Meta Pixel и `fbq('track','PageView')`.
Никаких других внешних скриптов. Шрифты — системный стек, без Google Fonts.

Язык: весь текст в объекте `COPY = { en: {...} }`; при загрузке — `?lang=xx` → `COPY[xx] || COPY[CONFIG.defaultLang]`.
Сейчас только `en`; структура нужна, чтобы потом добавить `de`, `fr`, `es` без правки разметки.

## 6. Комплаенс (обязательно, иначе Meta отклонит объявления)

- В hero и в футере дословно: `Free to play. No real money gambling. Contains in-app purchases.`
- В футере: `18+` и `Success in this game does not imply future success at real money gambling.`
- Нигде на странице нет слов: casino, bet, win money, cash prize, jackpot, aviator.
  Это же правило для alt-текстов и `<title>`.
- `<title>`: `Crash Rocket — Free Casual Crash Game`. `<meta name="robots" content="noindex">` — страница под платный трафик, в поиск не нужна.

## 7. Медиа

Настоящих ассетов пока нет (релиз апки 03.09). Claude Code делает плейсхолдеры:
- `icon-512.png` — красная ракета на тёмном фоне (SVG → PNG или чистый SVG inline);
- три скриншота 1080×1920 — тёмный градиент + надпись `Screenshot 1/2/3`;
- `gameplay-poster.jpg` — тот же градиент; `gameplay.mp4` — не создавать, в разметке `<video>` с `poster`, если файла нет — показывает постер.
Замена на реальные — просто перезаписать файлы с теми же именами.

## 8. Производительность и ограничения

- `index.html` без медиа ≤ 60 KB. Никаких библиотек.
- Медиа с `loading="lazy"` кроме иконки и постера.
- Lighthouse mobile: Performance ≥ 90, Accessibility ≥ 90.
- Работает без JS хотя бы как ссылка: у всех `.cta` `href=CONFIG.playUrl`, JS только дополняет referrer.

## 9. Как проверить (критерии приёмки)

1. Открыть `index.html` локально на телефоне или в DevTools mobile — все 9 блоков на месте, горизонтального скролла страницы нет.
2. Открыть `index.html?utm_source=fb&utm_campaign=test&sub1=abc`, нажать любую кнопку →
   уходит на `play.google.com/...?id=games.crash.rocket&referrer=utm_source%3Dfb%26utm_campaign%3Dtest%26sub1%3Dabc`.
3. Без параметров → уходит на чистый `CONFIG.playUrl`.
4. Консоль без ошибок при пустом `metaPixelId` и при тестовом.
5. `grep -i -E "casino|bet|jackpot|aviator|win money|cash prize" index.html` — пусто.
6. Ссылка `Privacy` открывает существующую `/privacy`.
7. Lighthouse mobile ≥ 90 / ≥ 90.

## 10. Вне скоупа (не делать)

Каталог игр, PWA/APK-раздача, iOS-ветка, кастомный домен, A/B-тесты, серверная часть. Всё это — отдельные задачи.

## 11. Git

Ветка `feat/landing`, коммиты по шагам (разметка → стили → JS кнопки → плейсхолдеры → log.md), PR в `main`
с описанием «что и как проверить». Секретов в репозитории нет — pixel ID это публичное значение.
