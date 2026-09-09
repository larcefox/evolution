# Evolife MVP — Acceptance Checklist

Этот документ — финальная приёмка MVP. Все пункты ниже должны быть выполнены одновременно.

## A. Запуск

- [ ] Проект устанавливается на Python 3.11+.
- [ ] `evolife --help` работает.
- [ ] `evolife run configs/basic.yaml` запускает эксперимент без ручного Python-кода.
- [ ] Эксперимент создаёт run directory.
- [ ] Effective config сохраняется в run directory.
- [ ] Headless run не требует pygame/display.

## B. Genome

- [ ] Genome состоит из genes с conditions/actions/params, а не из полного списка нейронов и связей.
- [ ] Поддерживаются действия: CREATE_NEURON, CREATE_CONNECTION, MODIFY_WEIGHT, SET_BIAS, SET_NEURON_TYPE, GROW_CONNECTION, ENABLE_PLASTICITY, STOP_GROWTH.
- [ ] Размер genome ограничен min/max.
- [ ] Random genome generation зависит только от заданного seed/RNG.
- [ ] Genome serializes/deserializes без потери данных.

## C. Mutation

- [ ] Parameter mutation работает.
- [ ] Probability mutation работает.
- [ ] Add gene работает.
- [ ] Delete gene работает.
- [ ] Duplicate gene работает.
- [ ] Action mutation работает.
- [ ] Mutation не меняет parent genome inplace.
- [ ] Mutation никогда не нарушает min/max genes.
- [ ] Crossover имеет интерфейс и может оставаться выключенным.

## D. Development / Embryogenesis

- [ ] `Genome -> Phenotype` реализовано отдельным engine.
- [ ] Развитие идёт за configurable number of ticks.
- [ ] Геном способен порождать разные числа neurons/synapses.
- [ ] Возможны recurrent connections.
- [ ] Возможны неработоспособные phenotype без аварии процесса.
- [ ] `max_neurons` соблюдается.
- [ ] `max_synapses` соблюдается.
- [ ] Один genome + seed даёт идентичный phenotype.

## E. Brain

- [ ] Есть SENSOR/HIDDEN/MOTOR neurons.
- [ ] Runtime brain является recurrent network.
- [ ] Activation использует `tanh`.
- [ ] `Brain.step()` детерминирован при одинаковом state/input.
- [ ] Recurrent state работает.
- [ ] Reset brain state работает.
- [ ] Опциональная Hebbian plasticity локальна для synapse.
- [ ] Глобальный backpropagation отсутствует.

## F. World + Agent

- [ ] Default GridWorld = 32x32.
- [ ] Есть EMPTY/WALL/FOOD/AGENT semantics.
- [ ] Actions: MOVE_FORWARD, TURN_LEFT, TURN_RIGHT, IDLE.
- [ ] Sensors минимум: food ahead/left/right, wall ahead/left/right, energy, previous reward.
- [ ] food_dx/food_dy можно отключить config.
- [ ] `World.reset(seed)` воспроизводим.
- [ ] Energy уменьшается каждый tick.
- [ ] Food увеличивает energy.
- [ ] Episode заканчивается по energy <= 0 или max_ticks.
- [ ] Collision учитывается и не приводит к некорректному movement.

## G. Fitness

- [ ] Fitness вынесен в отдельный module/evaluator.
- [ ] Базовая формула эквивалентна `food_collected * 100 + survival_ticks - collisions * 2`.
- [ ] Fitness можно заменить без переписывания World/Brain.

## H. Evolution

- [ ] Initial population генерируется случайно и детерминированно от root seed.
- [ ] Каждый organism проходит development и N episodes.
- [ ] Organism fitness = mean episode fitness.
- [ ] Parent pool = top selection fraction.
- [ ] Elite fraction переносится без mutation.
- [ ] Остальная population создаётся как mutated offspring.
- [ ] Population size сохраняется.
- [ ] Generation transition детерминирован от seed/RNG state.
- [ ] Tiny integration run проходит несколько generations.

## I. Metrics

Для каждого generation сохраняются минимум:

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

Дополнительно для основного эксперимента должны быть доступны:

- [ ] food collected
- [ ] survival time

## J. Checkpoint / Resume

- [ ] Checkpoint создаётся каждые configured N generations.
- [ ] В checkpoint есть population.
- [ ] В checkpoint есть generation.
- [ ] В checkpoint есть config.
- [ ] В checkpoint есть RNG state или эквивалентная полностью детерминированная схема восстановления.
- [ ] `evolife resume <run>` работает.
- [ ] Resume продолжает numbering/metrics корректно.
- [ ] Interrupted+resumed run совпадает с uninterrupted run для контрольного deterministic test.

## K. Visualization / Replay

- [ ] `evolife visualize <run>` строит best/mean/median fitness plot.
- [ ] Есть Brain viewer для SENSOR/HIDDEN/MOTOR + synapses.
- [ ] `evolife replay <run>` работает.
- [ ] `evolife replay <run> --generation N` работает.
- [ ] Replay показывает grid, agent, food, walls.
- [ ] Replay показывает хотя бы tick/energy/food state.
- [ ] Replay выбранного genome воспроизводим через сохранённый/фиксированный seed.

## L. Tests

- [ ] Genome serialization test.
- [ ] Config test.
- [ ] Seed determinism test.
- [ ] Mutation tests.
- [ ] Min/max genes test.
- [ ] Development determinism test.
- [ ] Brain.step test.
- [ ] World.step test.
- [ ] Fitness test.
- [ ] Selection test.
- [ ] Elitism test.
- [ ] Checkpoint test.
- [ ] Resume test.
- [ ] CLI smoke test.
- [ ] End-to-end tiny evolution test.
- [ ] Полный `pytest` проходит.

## M. Критическая воспроизводимость

Два чистых запуска с одинаковыми config + root seed должны дать:

- [ ] одинаковую generation 0 population;
- [ ] одинаковое development каждого genome;
- [ ] одинаковые episode seeds/world states;
- [ ] одинаковый fitness;
- [ ] одинаковый selection;
- [ ] одинаковые mutations;
- [ ] одинаковую population после нескольких generations.

Формула приёмки:

`same config + same seed -> same generation 0 -> same mutations -> same fitness -> same population`

## N. Производительность MVP

- [ ] Архитектура не требует multiprocessing/GPU для корректности.
- [ ] CPU run поддерживает population=100, episodes=3, ticks=500.
- [ ] Реализация способна выполнить не менее 100 generations без неконтролируемого роста памяти.
- [ ] Нет преждевременной сложной оптимизации, мешающей читаемости/тестируемости.

## O. Основной эксперимент

Параметры:

- [ ] Population = 100.
- [ ] Generations = 500.
- [ ] Episodes/organism = 3.
- [ ] Root seed зафиксирован.

Сравнить generations 0 / 100 / 250 / 500 по:

- [ ] fitness
- [ ] food collected
- [ ] survival time
- [ ] genome size
- [ ] neurons
- [ ] synapses

Артефакты:

- [ ] metrics сохранены;
- [ ] checkpoints сохранены;
- [ ] evolution plot сохранён;
- [ ] replay лучших organisms доступен.

## P. Критерий подтверждения гипотезы этапа 1

Результат считается положительным основанием двигаться дальше, если:

- [ ] fitness растёт устойчиво, а не только единичным выбросом;
- [ ] наблюдается улучшение поведения (food/survival);
- [ ] полезные свойства наследуются между generations;
- [ ] genome порождает разные brain topologies без заранее заданной «правильной» сети;
- [ ] вывод проверен более чем на одном seed либо явно показана высокая seed sensitivity.

Если эти пункты не выполнены, это не означает провал реализации MVP: MVP всё равно считается технически готовым при выполнении A–O, а гипотеза фиксируется как не подтверждённая данным экспериментом.
