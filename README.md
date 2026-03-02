# SynthScore (AiZynthFinder + RDKit)

Репозиторий содержит решение тестового задания:  
1) ретросинтетический поиск для 500 молекул с помощью **AiZynthFinder**,  
2) расчёт числового скоринга синтетической сложности **SynthScore** для каждой молекулы,  
3) экспорт результата в SDF-файл с исходными структурами и полем `SynthScore`.   

## Входные и выходные файлы

### Вход
- `output_500.sdf` — входной датасет из 500 молекул.  

### Промежуточные файлы 
- `output_500.smiles` — SMILES, извлечённые из `output_500.sdf`.
- `results_500.csv` — агрегированные метрики поиска AiZynthFinder по каждой молекуле (используются для скоринга без повторного ретросинтеза).
- `routes_500.jsonl` — деревья найденных маршрутов по каждой молекуле, удобно для ручного анализа примеров.

### Выход
- `output_500_scored.sdf` — итоговый SDF с полем `SynthScore` для каждой молекулы.

## Окружение и зависимости

- Python 3.10+
- RDKit
- pandas
- AiZynthFinder
- Public data для AiZynthFinder (policy модели + stock).

### Установка (пример, conda)
```bash
conda create -n aizynth-env python=3.10 -y
conda activate aizynth-env
conda install -c conda-forge rdkit pandas -y
pip install aizynthfinder
download_public_data .
```

### Настройка AiZynthFinder 

Взяла максимально дефолтные настройки - MCTS, USPTO, ZINC. Более конкретно в файле `config.yml`.

### Определение SynthScore
Направление шкалы: выше = сложнее.
#### Случай 1 — AiZynthFinder нашёл путь
Если  status = "solved"  и  n_routes > 0 , то SynthScore использует метрики AiZynthFinder:

`SynthScore = 1.0 * max_transforms + 3.0 * (1 - best_state_score) + 0.5 * log10(1 + number_of_nodes) + 0.5 * (search_time / 60)`

где:

- `max_transforms` — прокси числа стадий; больше стадий → сложнее.
- `best_state_score` — прокси “качества” найденного решения; чем меньше, тем хуже/сложнее.
- `number_of_nodes` и `search_time` — отражают усилие поиска; большое дерево/долгий поиск часто соответствуют более сложным целям.



#### Случай 2 — путь не найден (no-route)
Если  status != "solved"  или  n_routes == 0 , скоринг всё равно должен быть численным и различать такие молекулы.
Используется RDKit SA_score (Ertl SA Score, примерно 1–10; 1 — легко, 10 — сложно):

`SynthScore = 15 + SA_score`

где:

- `SA_score` — обеспечивает различие молекул внутри no-route (не один baseline).
- Константа `15` — отделяет no-route от большинства solved-случаев, сохраняя порядок по `SA_score`.








  
