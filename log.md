# Log

## 2026-09-02 — landing page (web2app preland)

Сделано: `index.html` (разметка/стили/JS в одном файле, без сборки) + `assets/`
плейсхолдеры (`icon-512.png`, `shot-1..3.png`, `gameplay-poster.jpg`) по
`LANDING-SPEC.md`. Убран `index.md` — конфликтовал с `index.html` в сборке
Jekyll (оба целились в один и тот же вывод `/index.html`).

Как проверить: `python3 -m http.server` в корне репозитория, открыть
`http://localhost:8000/index.html` в мобильном режиме DevTools. Приёмка —
`LANDING-SPEC.md` §9, все пункты пройдены (детали в PR).
