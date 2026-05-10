# Map from Image

Claude может воссоздать карту или layout по скриншоту или референсному изображению.
Это один из самых мощных способов работы — быстрее любого текстового описания.

## Как использовать

### Базовый промпт
```
[прикрепи скриншот карты]

Recreate this map layout in Roblox Studio. Use Parts and BaseParts to match 
the layout, proportions, and zones visible in the image. Label each zone 
with a Model named after its function (e.g. "WashingArea", "NPCZone", "UpgradeShop").
```

### Для воссоздания чужой карты
```
[скриншот из другой игры]

Recreate the general layout and zone structure of this map. 
Don't copy exact assets — use the spatial layout and proportions as reference.
Zone names: [перечисли]
Style: [опиши стиль]
```

---

## Что Claude делает по изображению

- Определяет зоны и их расположение
- Строит базовые Parts с примерными размерами
- Расставляет Model контейнеры с правильными именами
- Добавляет цвета по визуальному стилю на изображении
- Предлагает где разместить SpawnLocation, освещение, границы

## Что Claude НЕ может по изображению

- Воссоздать точные 3D модели и мешы — только базовые Parts
- Определить точные размеры в студах без масштаба
- Импортировать текстуры с изображения

Для детальных моделей — используй Toolbox после того как Claude построил layout.

---

## Пример промпт-цепочки для карты

```
Шаг 1:
[скриншот референса]
"Analyze this image and describe the zones you see. 
List them with approximate relative sizes before building anything."

Шаг 2:
"Good. Now build the basic layout using Parts in Workspace. 
Use colored Parts to distinguish zones. Don't add scripts yet."

Шаг 3:
"Now add SpawnLocation at the entrance, 
3 spots for washing machines in the center zone,
and a ProximityPrompt placeholder at the return counter."

Шаг 4:
"Add PointLight sources above each washer spot with soft blue color (150, 200, 255).
Add a SpotLight above the entrance pointing down, warm yellow (255, 220, 100)."
```

---

## Советы

- **Скетч от руки тоже работает.** Нарисуй план на бумаге, сфоткай — Claude поймёт.
- **Скриншот сверху лучше всего.** Top-down вид даёт Claude лучшее понимание layout.
- **Укажи масштаб если знаешь.** "Карта примерно 200x200 студов" помогает с пропорциями.
- **Несколько референсов одновременно.** Можно скинуть 2-3 изображения — общий layout + детали зоны.
