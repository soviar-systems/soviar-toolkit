# Опыт `ai_engineering_handbook`: закрывающие документы

Заведено 2026-11-17. Соседний репозиторий `ai_engineering_handbook` — каталог
рядом с `soviar-toolkit`, на одном уровне, вне этого репозитория — прожил год и
архивирован 2026-10-24 под тегом `archive-2026-10-24`: читается, не меняется.
Ценность для этой задачи в том, что он ставил **те же три вопроса** — про
недетерминизм модели, про жизненный цикл промахов и про централизацию нормы — и
отвечал на них не рассуждением, а своим концом.

Читать его как улику, а не как зависимость: `ARCHIVED.md` запрещает переносить
схему, идентификаторы ADR и скрипты (мера M6). Переносится диагноз.

## Чем кончился проект

Три измеренных факта, на которых стоит архив (`ARCHIVED.md` и
`architecture/post-mortems/post_mortem_sunset_of_the_ai_engineering_book.md`,
далее R-26004):

1. **Инверсия пропорции.** 369 файлов управления и поддержки на 140 файлов
   знания — 72% репозитория существует, чтобы управлять остальными 28%.
   В абсолюте: 23 скрипта, 1379 тестов, 56 ADR, 32 скилла и 52 плана на 140
   документов.
2. **Распад принуждения.** Три блокирующих валидатора сообщают 130, 6 и 18
   ошибок и **все выходят с кодом 0**. Репозиторий месяцами был «зелёным»,
   нарушая собственные правила.
3. **Структурное сцепление.** Проект-преемник жил неотслеживаемым внутри
   предшественника и дал 58 из 130 ошибок разметки и 13 из 18 битых ссылок.

Корневая причина названа циклом: правило → новая поверхность нарушений →
валидатор → схема → миграция → долг → новое правило. «Nothing in the loop had a
sunset clause, a retirement budget, or a ratio cap»: знание росло линейно с
усилием, управление — надлинейно с числом правил.

Наблюдение, которое важнее всех мер:

> «The system measured *whether the gate ran*, never *whether the gate held*.»
> Рядом: «Because the repository reported success. Tests passed. Hooks ran.»

Хронология оттуда же: пик активности (январь 2026, 176 коммитов) — не
производство знания, а постройка инструментов; июль–сентябрь 2026 — ноль.
«The final three months of activity produced no new knowledge.»

## Что он говорит про недетерминизм модели

`architecture/post-mortems/post-mortem_slm_non-determinism_in_commit_generation.md`
(R-26000) — ответ на вопрос «не ставлю ли я недостижимую цель», полученный
экспериментом. Задача: сообщения коммитов, Stage 1. Взяли SLM класса 1B–3B.
Требование задачи — «100% deterministic, low-context output», возможность модели
— «non-deterministic synthesis». Два отказа: instruction fidelity (галлюцинация
ключей, игнор повелительного наклонения, пропуск обязательных ArchTag) и
synthesis overload (модель игнорирует содержимое диффа и предел длины).

Мандат:

> «LLMs must only be applied to generative, low-constraint transformation tasks,
> not deterministic parsing or rule-based enforcement.»

Решение — убрать модель из шага, отдать его детерминированному CLI, а модель
оставить в Stage 3: высококонтекстная трансформация коротких проверенных
фрагментов. Деталь, которую легко потерять: мандат требовал `gitlint`, а
`architecture/adr/adr_26003_adoption_of_gitlint_for_tiered_workflow.md` его
**отклонил** — условную логику ArchTag он не выражает — и взял свой скрипт на
Python. Принцип выжил, инструмент сменился: копировать надо не `gitlint`, а
«решение, которое может принять только машина, принимает машина».

## Что он говорит про жизненный цикл промахов

`architecture/evidence/retrospective/R-26001_validation_deadlock.md`: глобальный
блокирующий гейт плюс долг предшествующих файлов заперли все коммиты. Лечение —
проверка по составу коммита: застейдженное судится строго (код 1), остальное
печатается как `[LEGACY]` и выходит с кодом 0.

Ошибка была не в лечении, а в том, что у режима **не было критерия выхода**:

> «Every non-blocking mode needs an exit criterion. A legacy/warning mode is a
> transition state. It must carry a condition under which it ends — a target
> count, a date, or a scoped migration checklist — or it becomes the permanent
> architecture.»

R-26004 называет это «the emergency became the permanent state» и добавляет:
«health was measured by exit code, not by error count». Отсюда меры M1 (не более
одного блокирующего гейта; гейт, выходящий с 0 при ошибке, — не гейт) и M5 (код
выхода — единственный сигнал здоровья; не нормализовать 0 ради продвижения).

`architecture/evidence/retrospective/R-26002_link_validation_false_positives.md`
добавляет границу применимости: проверка ссылок, применённая к Python, дала три
класса ложных срабатываний — регэкспы, которые ищут ссылки; тестовые фикстуры с
нарочно битыми путями; слишком широкий хук. Вывод: «Tiered constraints beat
blanket ones», «what counts as a link must be defined per file type». Ложное
срабатывание дороже пропуска: оно учит обходить проверку.

## Что он говорит про текст самого правила

`architecture/evidence/retrospective/R-26003_instruction_drift.md`: валидатор
печатал пример формата идентификатора, захардкоженный в Python и разошедшийся с
конфигурацией. Агенты следовали ошибке дословно, потому что
`architecture/adr/adr_26050_tool_driven_instructional_feedback_for_agents.md`
(ADR-26050) объявляет вывод валидатора главным инструктором. Каждая попытка
починки воспроизводила дефект, потому что инструкция была стабильна.

> «The correction was not to make agents smarter. It was to make the instructor a
> pure pipe: the example now comes from the single source of truth, and a missing
> value raises an error instead of silently falling back.»

Четыре меры оттуда: L1 «the pipe must not invent the water» — текст «как
исправить» обязан быть производным от единственного источника правды; L2
захардкоженный fallback — скрытое правило, «silent drift»; L3 указание с высоким
авторитетом ревизуется как архитектурный артефакт, потому что «the blast radius
of a typo in an error message became the blast radius of an architectural rule»;
L4 конфигурация описывает предмет, а не внутренности инструмента.

ADR-26050 важен не только предупреждением. Он зафиксировал, **почему проза не
работает**: агент «suffers from context drift or ignore these guidelines during
the implementation phase», когда правила поданы издалека. Отсюда JIT-инструкция
в момент нарушения и контракт вывода
`file_path:field — <инструкция> [config_source]`, а список допустимых значений
берётся из machine-readable реестра, а не из памяти модели.

## Что он говорит про число скиллов и про эмпирику

R-26004 называет 32 скилла в `.qwen/skills/` «the highest-value asset in the
repository» — и они же входят в те 369 файлов управления, что дали инверсию.
Скилл сам по себе не проблема; проблема — поверхность управления, растущая
быстрее предмета.

`architecture/adr/adr_26057_adoption_of_skillopt_for_empirical_agent_skill_optimization.md`
(ADR-26057) шёл дальше: скилл как обучаемый документ, «no skill update shall be
merged into the trunk unless it demonstrates a statistically significant
improvement (or non-regression) on a held-out validation set». Осталось в
статусе **proposed и не реализовано**; цена названа честно: «Requires the
creation and maintenance of high-fidelity evaluation datasets for every
optimized skill». Принцип верный, позволить его себе проект не смог.

