# Evolife

Evolife — экспериментальный MVP по эволюции искусственных организмов, в котором оптимизируется не полный набор весов нейросети, а компактный геном правил развития.

Основная гипотеза:

> Может ли эволюция улучшать компактный genome-program, который через embryogenesis порождает phenotype и recurrent brain, а затем наследуется и мутирует без глобального backpropagation между поколениями?

Основной цикл:

```text
Genome
  ↓
Embryogenesis
  ↓
Phenotype / Brain
  ↓
Life in World
  ↓
Fitness
  ↓
Selection + Mutation
  ↓
Next Generation
```

## Статус

Репозиторий подготовлен под последовательную разработку MVP через Codex/AI-агента.

Главные документы:

- [`AGENTS.md`](AGENTS.md) — обязательные правила работы агента и архитектурные ограничения.
- [`docs/DEVELOPMENT_CHECKLIST.md`](docs/DEVELOPMENT_CHECKLIST.md) — пошаговый checklist разработки от contracts до основного эксперимента.
- [`docs/MVP_ACCEPTANCE_CHECKLIST.md`](docs/MVP_ACCEPTANCE_CHECKLIST.md) — финальные критерии приёмки MVP.

## Целевой стек

- Python 3.11+
- NumPy
- Pydantic или dataclasses
- PyYAML
- pytest
- matplotlib
- pygame для replay

Не входят в MVP: LLM, Transformer, 3D, сложная физика, Kubernetes, микросервисы, внешняя БД, глобальный backpropagation.

## Планируемая структура

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

## Целевой CLI MVP

```bash
evolife run configs/basic.yaml
evolife resume runs/run_001
evolife visualize runs/run_001
evolife replay runs/run_001 --generation 50
```

## Ключевое требование

Вся случайность должна быть воспроизводимой.

```text
same config + same seed
    -> same generation 0
    -> same development
    -> same fitness
    -> same mutations
    -> same population
```

## Порядок разработки

Не начинать реализацию «с середины». Выполнять пункты из `docs/DEVELOPMENT_CHECKLIST.md` строго сверху вниз. После каждого крупного этапа должны появляться тесты, и полный `pytest` должен оставаться зелёным.

Основной эксперимент после готовности MVP:

```text
Population:          100
Generations:         500
Episodes/organism:   3
```

Контрольные поколения: 0, 100, 250, 500.

Сравниваются fitness, food collected, survival time, genome size, neurons и synapses.
