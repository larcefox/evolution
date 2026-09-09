# Evolife MVP — Development Checklist

Этот checklist является рабочим планом разработки MVP. Codex/разработчик должен идти сверху вниз и не перескакивать крупные этапы.

## 0. Инициализация проекта

- [ ] Создать `pyproject.toml` для Python 3.11+.
- [ ] Настроить зависимости: NumPy, PyYAML, pytest, matplotlib, pygame и Pydantic либо dataclasses.
- [ ] Создать package layout `src/evolife`.
- [ ] Создать каталоги `configs`, `docs`, `tests`, `experiments`.
- [ ] Добавить `evolife` console script entry point.
- [ ] Добавить базовую конфигурацию `configs/basic.yaml`.
- [ ] Убедиться, что `pytest` запускается даже при нулевом наборе функциональных модулей.

**Критерий выхода:** проект устанавливается в editable mode, импорт `evolife` работает, `pytest` стартует без ошибок окружения.

---

## 1. Models + contracts

### Реализация

- [ ] Определить enum `GeneAction`:
  - [ ] `CREATE_NEURON`
  - [ ] `CREATE_CONNECTION`
  - [ ] `MODIFY_WEIGHT`
  - [ ] `SET_BIAS`
  - [ ] `SET_NEURON_TYPE`
  - [ ] `GROW_CONNECTION`
  - [ ] `ENABLE_PLASTICITY`
  - [ ] `STOP_GROWTH`
- [ ] Определить enum `NeuronType`: SENSOR / HIDDEN / MOTOR.
- [ ] Определить модели `Gene`, `Genome`.
- [ ] Определить модели `Neuron`, `Synapse`, `Phenotype`.
- [ ] Определить модели world/action/observation.
- [ ] Определить configuration models для всех секций YAML.
- [ ] Определить интерфейсы/Protocol/ABC для:
  - [ ] `GenomeMutator`
  - [ ] `DevelopmentEngine`
  - [ ] `Brain`
  - [ ] `World`
  - [ ] `FitnessEvaluator`
  - [ ] `EvolutionEngine`
- [ ] Зафиксировать serialization format моделей.

### Тесты

- [ ] Genome serializes -> deserializes без потери данных.
- [ ] Config загружается из YAML и валидируется.
- [ ] Некорректные enum/action отклоняются.
- [ ] Phenotype можно создать с пустой сетью.

**Критерий выхода:** доменные модели и контракты не зависят от конкретных реализаций движков.

---

## 2. Genome Engine

### Реализация

- [ ] Реализовать случайную генерацию `Genome` из переданного RNG.
- [ ] Поддержать configurable gene count.
- [ ] Гарантировать `min_genes <= len(genome.genes) <= max_genes`.
- [ ] Сгенерировать параметры каждого gene в ограниченных диапазонах.
- [ ] Добавить стабильный genome ID/hash, если это помогает метрикам/checkpoint.
- [ ] Не хранить полный список будущих нейронов/связей в Genome.

### Тесты

- [ ] Одинаковый seed -> одинаковый Genome.
- [ ] Разный seed обычно -> разный Genome.
- [ ] Соблюдаются min/max genes.
- [ ] Все действия принадлежат `GeneAction`.
- [ ] Serialization round-trip.

**Критерий выхода:** можно детерминированно создать валидную начальную популяцию genomes.

---

## 3. Mutation Engine

### Реализация

- [ ] Parameter mutation с `parameter_sigma`.
- [ ] `parameter_probability`.
- [ ] Mutation `probability` gene.
- [ ] Add gene.
- [ ] Delete gene.
- [ ] Duplicate gene.
- [ ] Change action.
- [ ] Не нарушать min/max gene count.
- [ ] Mutation возвращает новый Genome и не меняет родителя inplace.
- [ ] Предусмотреть интерфейс crossover без обязательной реализации первого эксперимента.

### Тесты

- [ ] Seed делает mutation детерминированной.
- [ ] Родитель после mutate не изменён.
- [ ] Forced add действительно добавляет gene.
- [ ] Forced delete удаляет gene, кроме min limit.
- [ ] Forced duplicate копирует gene.
- [ ] Forced action mutation меняет action.
- [ ] После тысяч mutations genome остаётся в bounds.

**Критерий выхода:** mutation может безопасно использоваться в population loop.

---

## 4. Development / Embryogenesis Engine

### Реализация

- [ ] Реализовать `Genome -> Phenotype`.
- [ ] Development идёт дискретными ticks.
- [ ] Gene conditions первой версии сделать минимальными и понятными.
- [ ] Gene actions способны порождать neurons/synapses.
- [ ] Поддержать recurrent connections.
- [ ] Поддержать bias/weight/neuron type.
- [ ] Поддержать per-synapse plasticity flag.
- [ ] `STOP_GROWTH` прекращает/ограничивает развитие по выбранной семантике.
- [ ] Соблюдать `max_neurons`.
- [ ] Соблюдать `max_synapses`.
- [ ] Пустой/плохой phenotype является валидным результатом.
- [ ] Development использует только переданный RNG/seed.

### Тесты

- [ ] Один Genome + один seed -> одинаковый Phenotype.
- [ ] Max neuron count не превышается.
- [ ] Max synapse count не превышается.
- [ ] CREATE_NEURON реально способен создать neuron.
- [ ] CREATE_CONNECTION реально способен создать synapse.
- [ ] STOP_GROWTH соблюдается.
- [ ] Неработоспособный genome не вызывает crash.

**Критерий выхода:** одинаковый genotype стабильно разворачивается в одинаковую сеть.

---

## 5. Brain Engine

### Реализация

- [ ] Создать runtime Brain из Phenotype.
- [ ] SENSOR neurons получают observation.
- [ ] HIDDEN/MOTOR обновляются дискретно.
- [ ] Реализовать recurrent state.
- [ ] Activation: `tanh(weighted_sum + bias)`.
- [ ] Вернуть motor output vector.
- [ ] Определить порядок обновления сети и закрепить тестами.
- [ ] Реализовать optional local Hebbian plasticity.
- [ ] Plasticity не использует global backprop.
- [ ] Добавить reset recurrent state.

### Тесты

- [ ] Простая вручную заданная сеть считает ожидаемый output.
- [ ] Recurrent state влияет на следующий tick.
- [ ] `reset()` очищает state.
- [ ] Plasticity disabled -> weights неизменны.
- [ ] Plasticity enabled -> локальное ожидаемое изменение weights.
- [ ] Пустой/дефектный brain безопасно обрабатывается.

**Критерий выхода:** Brain может детерминированно преобразовывать observation в motor outputs.

---

## 6. GridWorld

### Реализация

- [ ] Grid default 32x32.
- [ ] Cell/object types: EMPTY / WALL / FOOD / AGENT.
- [ ] Deterministic reset(seed).
- [ ] Random agent placement.
- [ ] Random food placement.
- [ ] Optional/random walls.
- [ ] Orientation агента.
- [ ] Actions:
  - [ ] MOVE_FORWARD
  - [ ] TURN_LEFT
  - [ ] TURN_RIGHT
  - [ ] IDLE
- [ ] Collisions не двигают агента и учитываются.
- [ ] Food pickup удаляет/перемещает food согласно выбранной простой модели.
- [ ] Sensors минимум:
  - [ ] food ahead
  - [ ] food left
  - [ ] food right
  - [ ] wall ahead
  - [ ] wall left
  - [ ] wall right
  - [ ] energy
  - [ ] previous reward
- [ ] Optional `food_dx`, `food_dy` по config.

### Тесты

- [ ] reset(seed) воспроизводим.
- [ ] Turn left/right меняет orientation правильно.
- [ ] MOVE_FORWARD перемещает в пустую cell.
- [ ] WALL collision блокирует movement.
- [ ] Food pickup работает.
- [ ] Sensor vector соответствует вручную созданной карте.
- [ ] Optional sensors включаются/выключаются config.

**Критерий выхода:** World полностью тестируем без pygame/UI.

---

## 7. Agent

### Реализация

- [ ] Связать Brain + World-facing state.
- [ ] Хранить energy.
- [ ] Хранить previous reward.
- [ ] Хранить food collected.
- [ ] Хранить collisions.
- [ ] Хранить survival ticks.
- [ ] `argmax(motor_outputs)` -> action.
- [ ] Graceful fallback для brain без корректных motor outputs: IDLE или фиксированное правило.
- [ ] Energy уменьшается каждый tick.
- [ ] Food добавляет configurable energy.

### Тесты

- [ ] Выбор action по motor outputs.
- [ ] Energy tick cost.
- [ ] Food energy reward.
- [ ] Agent dying at energy <= 0.
- [ ] Counters обновляются корректно.

**Критерий выхода:** один агент может прожить полный episode в world.

---

## 8. Fitness

### Реализация

- [ ] Отдельный `FitnessEvaluator`.
- [ ] Базовая формула:
  `food_collected * 100 + survival_ticks - collisions * 2`.
- [ ] Не смешивать fitness с World/Evolution.
- [ ] Предусмотреть возможность альтернативной fitness function через config/DI.

### Тесты

- [ ] Проверить точное значение на нескольких фиксированных состояниях.
- [ ] Food имеет ожидаемо положительный вклад.
- [ ] Collision имеет отрицательный вклад.

**Критерий выхода:** результат episode можно преобразовать в scalar fitness независимо от evolution engine.

---

## 9. Simulation + Evolution Engine

### Episode runner

- [ ] `develop genome -> brain -> agent -> world -> episode result`.
- [ ] Episode заканчивается при energy <= 0.
- [ ] Episode заканчивается при max_ticks.
- [ ] Все seeds выводятся детерминированно из root seed/generation/organism/episode.

### Population evaluation

- [ ] Default population = 100.
- [ ] `episodes_per_agent` default = 3.
- [ ] Fitness organism = mean episode fitness.
- [ ] Собирать ancillary metrics: food, survival, genome size, neurons, synapses.

### Selection/reproduction

- [ ] Rank population по fitness.
- [ ] Top `selection_fraction` = parent pool.
- [ ] Top `elite_fraction` проходят без mutation.
- [ ] Остальные offspring создаются из родителей и мутируются.
- [ ] Population size сохраняется точно.
- [ ] Parent selection использует deterministic RNG.
- [ ] Crossover может быть выключен.

### Тесты

- [ ] Elite genome остаётся byte/value-equivalent.
- [ ] Non-elites мутируют согласно forced config.
- [ ] Population size не меняется.
- [ ] Parent берётся только из selection pool.
- [ ] Fixed seed -> identical next generation.
- [ ] Минимальный integration test проходит 2-3 generations.

**Критерий выхода:** полный evolutionary loop работает headless.

---

## 10. CLI

### Реализация

- [ ] `evolife run <config.yaml>`.
- [ ] Создавать уникальный run directory.
- [ ] Сохранять effective config.
- [ ] Печатать progress generation/best/mean fitness.
- [ ] Корректно завершаться по configured generations.
- [ ] `--seed` override при необходимости, если предусмотрено.

### Тесты

- [ ] CLI help.
- [ ] Run на tiny test config завершается с кодом 0.
- [ ] Run directory создан.
- [ ] Metrics/config записаны.

**Критерий выхода:** эксперимент можно запустить одной командой без Python-кода пользователя.

---

## 11. Metrics + Checkpoints + Resume

### Metrics

Для каждого generation сохранять:

- [ ] generation
- [ ] best_fitness
- [ ] mean_fitness
- [ ] median_fitness
- [ ] best_genome_size
- [ ] mean_genome_size
- [ ] best_neuron_count
- [ ] mean_neuron_count
- [ ] best_synapse_count
- [ ] mean_synapse_count
- [ ] желательно best/mean food collected
- [ ] желательно best/mean survival ticks

### Checkpoint

Каждые `checkpoint_every`:

- [ ] population genomes
- [ ] generation
- [ ] RNG state / seed derivation state
- [ ] config
- [ ] fitness/metadata, необходимые для продолжения
- [ ] best organism replay data или данные для его реконструкции

### Resume

- [ ] `evolife resume runs/run_001`.
- [ ] Определить последний валидный checkpoint.
- [ ] Восстановить population.
- [ ] Восстановить RNG trajectory.
- [ ] Продолжить generation numbering.
- [ ] Не перетирать уже существующие metrics.

### Тесты

- [ ] Save/load checkpoint round-trip.
- [ ] Interrupted run + resume == uninterrupted run по genomes/fitness для контрольного tiny config.
- [ ] Повреждённый/неполный checkpoint даёт понятную ошибку.

**Критерий выхода:** остановка процесса не меняет научный результат после resume.

---

## 12. Replay

### Реализация

- [ ] Сохранять/reconstruct best genome выбранного generation.
- [ ] Использовать фиксированный replay seed либо записанный episode seed.
- [ ] `evolife replay <run>` выбирает последний/best generation по документированной логике.
- [ ] `evolife replay <run> --generation N`.
- [ ] Pygame показывает grid.
- [ ] Показывать agent orientation.
- [ ] Показывать food.
- [ ] Показывать walls.
- [ ] Отображать ticks/energy/food/fitness info.
- [ ] Replay не влияет на экспериментальные checkpoints/metrics.

### Тесты

- [ ] Headless тест корректного выбора replay genome/seed.
- [ ] Ошибка на отсутствующий generation понятна.

**Критерий выхода:** можно визуально увидеть поведение лучшего организма.

---

## 13. Visualization

### Evolution plot

- [ ] Best fitness по generations.
- [ ] Mean fitness.
- [ ] Median fitness.
- [ ] Сохранение PNG.
- [ ] `evolife visualize <run>`.

### Brain viewer

- [ ] Sensors отдельно.
- [ ] Hidden отдельно.
- [ ] Motors отдельно.
- [ ] Synapses отображаются связями.
- [ ] Различать recurrent/обычные связи хотя бы визуально/подписью при возможности.
- [ ] Не требовать networkx, если без него проще.

### World viewer

- [ ] Реализован через replay или переиспользуемый renderer.

### Тесты

- [ ] Plot генерируется из fixture metrics.
- [ ] Brain layout не падает на пустой сети.

**Критерий выхода:** пользователь может увидеть динамику fitness, brain structure и поведение в world.

---

## 14. Full deterministic test suite

Обязательные тестовые категории:

- [ ] Genome serialization.
- [ ] Config parsing.
- [ ] Seed determinism.
- [ ] Mutation operations.
- [ ] Gene min/max.
- [ ] Development determinism.
- [ ] Brain.step.
- [ ] World.step.
- [ ] Fitness.
- [ ] Selection.
- [ ] Elitism.
- [ ] Checkpoint.
- [ ] Resume.
- [ ] CLI smoke.
- [ ] Tiny end-to-end experiment.

### Критический тест

- [ ] Запустить два независимых tiny experiments с одним config + seed.
- [ ] Проверить generation 0 population equality.
- [ ] Проверить equality mutations/next generation.
- [ ] Проверить fitness equality.
- [ ] Проверить population equality после нескольких generations.

**Обязательная инварианта:**

`same config + same seed -> same generation 0 -> same mutations -> same fitness -> same population`

---

## 15. README + experiment protocol

- [ ] Описать гипотезу.
- [ ] Описать архитектуру.
- [ ] Установка.
- [ ] Quick start.
- [ ] Все CLI команды.
- [ ] Config reference.
- [ ] Seed/reproducibility policy.
- [ ] Где лежат metrics/checkpoints.
- [ ] Как replay generation.
- [ ] Как visualize run.
- [ ] Ограничения MVP.
- [ ] Что не является биологической моделью.
- [ ] Protocol основного эксперимента.

---

# Основной эксперимент MVP

После завершения реализации:

- [ ] Population = 100.
- [ ] Generations = 500.
- [ ] Episodes per organism = 3.
- [ ] Зафиксировать root seed.
- [ ] Сохранить checkpoints минимум поколений 0 / 100 / 250 / 500 или ближайших checkpoint generations.
- [ ] Сравнить поколения 0, 100, 250, 500 по:
  - [ ] fitness
  - [ ] food collected
  - [ ] survival time
  - [ ] genome size
  - [ ] neurons
  - [ ] synapses
- [ ] Построить evolution plot.
- [ ] Сделать replay лучших organisms контрольных поколений.
- [ ] Оценить устойчивость роста fitness, а не единичный lucky spike.
- [ ] Зафиксировать наблюдаемые наследуемые структуры поведения.

# Решение по гипотезе этапа 1

Первый этап считается успешным, если воспроизводимо наблюдаются:

- [ ] устойчивый/statistically credible рост fitness относительно generation 0;
- [ ] улучшение food collection и/или survival без искусственного задания правильной brain architecture;
- [ ] наследование полезных genome structures;
- [ ] развитие разных network topologies из компактных genomes;
- [ ] результат повторяется хотя бы на нескольких seeds либо явно указана зависимость от seed.

После этого можно переходить к следующему исследовательскому этапу: genome-program embryogenesis с cell types, division, chemical gradients, apoptosis, axon growth и более богатой локальной plasticity.
