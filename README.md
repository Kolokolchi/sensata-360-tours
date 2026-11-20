# Sensata Group — 3D-туры 360°

Готовый статический сайт: все туры по жилым комплексам без редактора Studio.
Загружается на любой хостинг как есть, туры встраиваются на другие сайты (в том числе на Тильду).

Сборка от 20.11.2026, тайлы до 6144 px на грань куба.

## Что внутри

| Адрес | Что это |
| --- | --- |
| `https://kolokolchi.github.io/sensata-360-tours/` | витрина: все объекты в одном окне |
| `https://kolokolchi.github.io/sensata-360-tours/embed.html` | **ссылки и готовый код для Тильды** по каждому объекту |
| `https://kolokolchi.github.io/sensata-360-tours/01_mangilik/` | ЖК MÁŃGİLİK |
| `https://kolokolchi.github.io/sensata-360-tours/02_baiqadam/` | ЖК BAIQADAM |
| `https://kolokolchi.github.io/sensata-360-tours/03_nygmet/` | ЖК NYGMET |
| `https://kolokolchi.github.io/sensata-360-tours/04_olympic_park/` | ЖК OLYMPIC PARK |
| `https://kolokolchi.github.io/sensata-360-tours/05_qaharman/` | ЖК QAHARMAN |

## Как разместить

1. Нужен хостинг с **HTTPS** (Тильда открывается по https, тур по http браузер заблокирует).
   Подойдёт поддомен, например `https://tours.ваш-домен.kz/`, или папка на существующем сайте.
2. Загрузите **всё содержимое** этой папки (включая скрытый файл `.htaccess`) в корень поддомена
   или в нужную папку. Пути внутри относительные — переносить можно куда угодно без пересборки.
   Через панель хостинга удобнее загрузить архив и распаковать его там.
3. Проверьте: `https://kolokolchi.github.io/sensata-360-tours/` открывает витрину, `https://kolokolchi.github.io/sensata-360-tours/01_mangilik/` — тур.
4. Откройте `https://kolokolchi.github.io/sensata-360-tours/embed.html` — там ссылки и код для встраивания.

### Требования к серверу

- Сервер **не должен** отдавать заголовок `X-Frame-Options` и `Content-Security-Policy: frame-ancestors 'self'` —
  иначе тур не покажется внутри сайта. Для Apache это уже сделано в `.htaccess`.
- Типы файлов: `.webp` → `image/webp`, `.glb` → `model/gltf-binary`, `.wasm` → `application/wasm`, `.json` → `application/json`.
- Если после загрузки сайт отвечает ошибкой 500 — хостинг не принимает какую-то директиву из `.htaccess`: удалите файл.

Пример для nginx (VPS):

```nginx
server {
    listen 443 ssl http2;
    server_name tours.example.kz;
    root /var/www/sensata-tours;
    index index.html;

    gzip on;
    gzip_types application/json text/css application/javascript application/wasm;

    location ~* \.(html|json)$          { add_header Cache-Control "no-cache"; }
    location ~* \.(webp|avif|jpe?g|png|glb)$ { add_header Cache-Control "public, max-age=86400"; }
    location ~* \.(js|css|wasm)$       { add_header Cache-Control "public, max-age=2592000"; }
    # не добавляйте X-Frame-Options — иначе тур не встроится на сайт
}
```

## Как встроить на Тильду

Откройте `https://kolokolchi.github.io/sensata-360-tours/embed.html`, выберите объект, стартовую точку и язык — код соберётся сам. Варианты:

- **Блок на странице** — Тильда: «Добавить блок → Другое → T123 HTML-код», вставить код, сохранить, опубликовать.
- **Zero Block** — элемент «HTML», вставить код и задать размер элемента для каждой ширины экрана.
- **Кнопка + попап** — блок T868 (попап с HTML-кодом) и ссылка `#popup:…` на кнопке. Тур грузится только при открытии попапа.
- **Просто ссылка** — на кнопку или в меню, «Открывать в новом окне».

Простейший код вручную:

```html
<iframe src="https://kolokolchi.github.io/sensata-360-tours/01_mangilik/" width="100%" height="680" style="border:0;border-radius:20px"
  loading="lazy" allow="fullscreen; gyroscope; accelerometer; magnetometer; clipboard-write" allowfullscreen></iframe>
```

Поведение внутри чужого сайта:

- колесо мыши не перехватывает прокрутку страницы, пока посетитель не кликнет по туру (или зажмёт Ctrl);
- кнопка «На весь экран» там, где полноэкранный режим невозможен (iPhone), открывает тур в новой вкладке с тем же ракурсом;
- на телефоне блок из `embed.html` занимает 72% высоты экрана, чтобы страницу можно было пролистать.

## Параметры ссылки

| Параметр | Пример | Что делает |
| --- | --- | --- |
| `zone` | `?zone=apartment` | начать с зоны: `aero`, `yard`, `hall`, `apartment` |
| `scene` | `?scene=pano20958` | начать с конкретной сцены (id меняется, если сцену пересоздать в Studio) |
| `yaw`, `pitch`, `fov` | `&yaw=120&pitch=-5&fov=70` | ракурс камеры (кнопка «Поделиться ракурсом» в туре собирает такую ссылку) |
| `lang` | `?lang=kz` | язык интерфейса: `ru`, `kz`, `en` |
| `fill` | `?fill=1` | тур занимает всё окно (попап, отдельная страница): колесо сразу приближает |

## Управление туром со страницы (postMessage)

```html
<script>
  const frame = document.querySelector('iframe[src*="01_mangilik"]');
  frame.contentWindow.postMessage({ type: 'tour:navigate', zone: 'yard' }, '*');   // перейти во двор
  frame.contentWindow.postMessage({ type: 'tour:setLocale', locale: 'kz' }, '*');  // сменить язык
  frame.contentWindow.postMessage({ type: 'tour:toggleDollhouse', active: true }, '*'); // 3D-планировка
  // тур сообщает о себе: { source: "sensata-3d-tour", type: "tour:ready" | "tour:sceneChange", ... }
</script>
```

## Обновление

После правок в Studio пересоберите папку из движка тура (`06_Modern_Tour_Engine`) и загрузите заново:

```
npm run build:hosting
```

Код, вставленный в Тильду, менять не нужно: адреса туров остаются прежними.
