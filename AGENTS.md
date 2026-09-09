# AGENTS.md — Evolife MVP

Этот файл задаёт обязательный порядок работы Codex/AI-агента в репозитории.

## Цель MVP

Проверить гипотезу: может ли эволюция оптимизировать компактный геном правил развития, который порождает фенотип и recurrent neural network, вместо прямой эволюции полного списка нейронов/связей или использования глобального backpropagation.

Главный цикл:

`Genome -> Embryogenesis -> Phenotype/Brain -> Life in World -> Fitness -> Selection + Mutation -> Next Generation`

## Жёсткие ограничения

- Python 3.11+.
- Основные зависимости: NumPy, Pydantic или dataclasses, PyYAML, pytest, matplotlib, pygame.
- Никакого глобального backpropagation между поколениями.
- Геном хранит правила развития, а не полный список нейронов и связей.
- MVP: CPU-first, GridWorld 32x32, небольшие RNN.
- Не добавлять LLM, Transformer, 3D, сложную физику, Kubernetes, микросервисы, внешнюю БД.
- Не усложнять архитектуру до появления измеримой необходимости.
- Любая случайность должна идти через явно переданный seed/RNG.
- Один и тот же config + seed обязан воспроизводить один и тот же результат.
- Связь между модулями только через явные структуры/контракты.

## Порядок разработки

Работать строго по этапам из `docs/DEVELOPMENT_CHECKLIST.md`:

1. Models + contracts
2. Genome Engine
3. Mutation Engine
4. Development Engine
5. Brain Engine
6. GridWorld
7. Agent
8. Fitness
9. Evolution Engine
10. CLI
11. Checkpoints / resume
12. Replay
13. Visualization
14. Full test suite
15. README / experiment instructions

Не переходить к следующему крупному этапу, пока текущий:

- имеет минимально рабочую реализацию;
- имеет тесты;
- проходит `pytest`;
- не ломает уже реализованные тесты;
- отмечен в checklist как завершённый.

## Правило изменений

Перед изменением:

1. Определить конкретный пункт checklist.
2. Реализовать минимальный объём, нужный для этого пункта.
3. Добавить/обновить тесты.
4. Запустить релевантные тесты.
5. Запустить полный `pytest`.
6. Только после успеха двигаться дальше.

Не делать большие рефакторинги одновременно с добавлением новой функциональности без необходимости.

## Базовая структура

```text
evolife/
├── pyproject.toml
├── README.md
├── AGENTS.md
├── configs/
├── docs/
├── src/evolife/
│   ├── genome/
│   ├── development/
│   ├── brain/
│   ├── world/
│   ├── agent/
│   ├── evolution/
│   ├── simulation/
│   ├── metrics/
│   └── visualization/
├── tests/
└── experiments/
```

## Контракты, которые должны оставаться стабильными

Минимально требуются интерфейсы, эквивалентные:

```python
class GenomeMutator:
    def mutate(self, genome: Genome) -> Genome: ...

class DevelopmentEngine:
    def develop(self, genome: Genome) -> Phenotype: ...

class Brain:
    def step(self, inputs: np.ndarray) -> np.ndarray: ...

class World:
    def reset(self, seed: int): ...
    def observe(self, agent) -> np.ndarray: ...
    def step(self, action): ...

class FitnessEvaluator:
    def evaluate(self, agent) -> float: ...

class EvolutionEngine:
    def run_generation(self): ...
```

Допускается уточнять сигнатуры, если это улучшает типизацию/тестируемость, но ответственность модулей нельзя смешивать.

## Детерминизм

Запрещено использовать неуправляемый глобальный random state внутри доменных модулей.

Предпочтительно:

- `numpy.random.Generator`;
- создание RNG из корневого seed;
- детерминированное порождение дочерних seed для generation / organism / episode;
- сохранение RNG state в checkpoint.

Критическая инварианта:

`same config + same seed -> same generation 0 -> same mutations -> same fitness -> same population`

## Ограничения развития

Конфигурацией должны ограничиваться как минимум:

- `min_genes`, `max_genes` (целевой диапазон MVP: 8..256);
- `max_neurons` (по базовому config 128);
- `max_synapses` (по базовому config 1024);
- `development.ticks`;
- population size;
- episode max ticks.

Неработоспособный геном/фенотип является допустимым результатом эволюции и должен получать низкий fitness, а не падать с необработанным исключением.

## Brain

MVP — дискретная recurrent neural network:

`activation = tanh(sum(input_i * weight_i) + bias)`

Типы нейронов:

- SENSOR
- HIDDEN
- MOTOR

Локальная Hebbian plasticity допустима только как локальное правило конкретных связей:

`dw = learning_rate * pre * post`

Никакого backpropagation.

## World

MVP — GridWorld.

Минимальные действия:

- MOVE_FORWARD
- TURN_LEFT
- TURN_RIGHT
- IDLE

Минимальные наблюдения:

- food ahead / left / right;
- wall ahead / left / right;
- energy;
- previous reward.

`food_dx/food_dy` — только как отключаемая config-опция.

## Fitness

Fitness должен быть отдельной заменяемой сущностью. Базовая версия:

`food_collected * 100 + survival_ticks - collisions * 2`

World/Agent не должны содержать скрытую логику отбора поколений.

## Checkpoints

Checkpoint должен позволять продолжить эксперимент без изменения траектории эволюции и содержать минимум:

- generation;
- population genomes;
- fitness/необходимое состояние поколения;
- RNG state;
- config;
- данные, необходимые для replay лучшего организма.

## CLI — Definition of Done

К концу MVP должны работать:

```bash
evolife run configs/basic.yaml
evolife resume runs/run_001
evolife visualize runs/run_001
evolife replay runs/run_001 --generation 50
```

## Что считать завершением MVP

MVP готов только если одновременно выполняются пункты `docs/MVP_ACCEPTANCE_CHECKLIST.md`, включая запуск минимум 100 поколений, checkpoint/resume, replay и тест полного детерминизма.
