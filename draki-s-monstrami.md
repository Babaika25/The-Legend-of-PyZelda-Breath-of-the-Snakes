# Драки с монстрами

Итак, для начала создадим два новых демона для атак в `level.py`:

```python
self.attack_sprites = pygame.sprite.Group() #атакующий спрайт
self.attackable_sprites = pygame.sprite.Group() #атакуемый спрайт
```

Думаю, по названиям понятно, что демоны нужны для обозначения процесса атаки. Дополним метод вызова врагов помимо видимых спрайтов, атакуемыми спрайтами (`self.attack_sprites`):

```python
Enemy(monster_name, (x, y), [self.visible_sprites, self.attackable_sprites], self.obstacle_sprites)
```

Также, дополним метод `create_attack` атакующим спрайтом (`self.attack_sprites`) :

```python
def create_attack(self):
    self.current_attack = Weapon(self.player, [self.visible_sprites, self.attack_sprites])
```

Далее пропишем новый метод с логикой атаки игрока:

```python
def player_attack_logic(self):
    if self.attack_sprites:
        for attack_sprites in self.attack_sprites:
            collision_sprites = pygame.sprite.spritecollide(attack_sprites, self.attackable_sprites, False)
            if collision_sprites:
                for target_sprite in collision_sprites:
                    target_sprite.kill()
```

Самая интересная строка тут, это строка с методом пайгейма `pygame.sprite.spritecollide`. Данный метод позволяет удалять спрайт из группы. Первый аргумент функции — спрайты для атаки, второй — группа спрайтов, из которой мы будем удалять спрайт, третий — DoKill. Если у DoKill установлено значение `True`, все спрайты, которые сталкиваются, будут удалены из группы. Далее, в run-методе пропишем наш метод. Исход:

<figure><img src=".gitbook/assets/42.gif" alt=""><figcaption></figcaption></figure>

Немного улучшим метод `player_attack_logic`:

```python
if target_sprite.sprite_type == 'enemy':
    target_sprite.get_damage(self.player, attack_sprites.sprite_type)
```

Мы стали сопоставлять наши спрайты по типам. В моём проекте типов только два (enemy и weapon). Я прописал в файле `weapon.py` в демоне строку для присваивания ему нового типа:

```python
self.sprite_type = 'weapon'
```

Далее нам не нужно удалять врага при ударе. Нам нужно прописывать ему урон от нашего оружия. Собственно, теперь нужно в файле enemy.py прописать новый метод — `get_damage`:

```python
def get_damage(player, attack_type):
    if attack_type == 'weapon':
        self.health -= player.get_full_weapon_damage()
```

Тут мы прописываем новый метод (`get_full_weapon_damage`), который должен высчитывать сумму урона от оружия и от силы самого Линка (прямо как в Dark Souls). Пропишем же данный метод в `player.py`:

```python
def get_full_weapon_damage(self):
    base_damage = self.stats['attack'] #урон самого Линка
    weapon_damage = weapon_data[self.weapon]['damage'] #урон от выбранного оружия
    return base_damage + weapon_damage
```

Находим и складываем уроны из наших списков. Возможно, тут встанет вопрос: "Зачем так сложно, если у нас только один меч и всё?". Я хотел бы сделать проект так, чтобы вы могли самостоятельно с ним "поиграться". Собственно и цель книги не номинация "Игра года" в The Game Awards, а лишь попытка продемонстрировать работоспособность языка Python как неплохого движка. Ну да вернёмся к коду. Помимо урона, я решил сразу прописать кулдаун оружия и заменил строку:

```python
if current_time - self.attack_time >= self.attack_cooldown
```

На строку:

```python
if current_time - self.attack_time >= self.attack_cooldown + weapon_data[self.weapon]['cooldown']
```

Напишем новый метод в `enemy.py` на проверку смерти монстра:

```python
def check_death(self):
    if self.health <= 0:
        self.kill()
```

Тут я даже не знаю что ещё подсветить в коде :) Не забудьте добавить данный метод в update-метод. Теперь один удар приводит к смерти врага. Таким образом, PyGame считает, что пока оружие соприкасается с врагом (вызывается метод коллизий), удары наносятся один за другим. Как итог — Линк танк, который уничтожает всё на своём пути. Исправим это. Для начала, создадим новых демонов `enemy.py`:

```python
self.vulnerable = True #флаг уязвимости
self.hit_time = None #время удара
self.invincibility_duration = 300 #продолжительность неуязвимости
```

Тут достаточно прозрачные демоны. Важный момент — флаг уязвимости. Если он поднят — враг может получать урон. Теперь встроим их в `get_damage`:

```python
def get_damage(self, player, attack_type):
    if self.vulnerable: 
        if attack_type == 'weapon':
            self.health -= player.get_full_weapon_damage()
        self.hit_time = pygame.time
```

Далее необходимо дополнить код cooldown-метода:

```python
if not self.vulnerable:
    if current_time - self.hit_time >= self.invincibility_duration:
        self.vulnerable = True
```

Тут, как я говорил ранее, сразу добавляем вариацию флага. У нас было место, где флаг опускается и теперь в cooldown-методе он поднимается по истечению указанного времени неуязвимости. Теперь все враги убиваются весьма приятно. Теперь, нужно добавить отбивание врага на дистанцию, которая указана у каждого врага. Создадим ещё метод:

```python
def hit_reaction(self):
    if not self.vulnerable:
        self.direction *= -self.resistance
```

Осталось вычислить положение в методе `get_damage`:

```python
self.direction = self.get_player_distance_direction(player)[1]
```

Теперь добавим мерцание во время удара, чтобы понять что удар был сделан. В PyGame все сигнатуры (синусоидные функции) лежат в диапазоне от -255 до 255, а позиции удобно брать из синусоид, так как интерпретация в Python будет работать на основе степеней (как и любой калькулятор), а затем будет процесс получения точки на синусоиде. Этот процесс я описывал дольше, чем будет писаться метод отображения пульсации:

```python
def wave_value(self):
    value = sin(pygame.time.get_ticks())
    if value >= 0:
        return 255
    else:
        return 0
```

Данный метод написан в `entity.py`. Не забудьте импортировать sin-метод из библиотеки math. Теперь вызовем данный метод при ударе по врагу в методе `animate`:

```python
if not self.vulnerable:
    alpha = self.wave_value()
    self.image.set_alpha(alpha)
else:
    self.image.set_alpha(255)
```

Сейчас мы бьём врагов абсолютно верно и можем отследить когда монстры атакуют нас (в терминал приходит сообщение "attack"). Осталось сделать метод, который делает урон Линку. Пропишем его в `level.py`:

```python
def damage_player(self, amount, attack_type):
    if self.player.vulnerable:
        self.player.health -= amount
        self.player.vulnerable = False
        self.player.hurt_time = pygame.time.get_ticks()
```

Дополним в `create_map` в строку вызова монстров нанесение урона Линку:

```python
Enemy(monster_name, (x, y), [self.visible_sprites, self.attackable_sprites], self.obstacle_sprites, self.damage_player)
```

Добавим в Enemy-класс параметр (`damage_player`) и пропишем нового демона — `self.damage_player = damage_player`. Теперь вместо простого вывода сообщения "attack" выполним вызов нашего метода:

```python
self.damage_player(amount, attack_type)
```

Добавим параметры таймеров в демонов нашего player-класса:

```python
self.vulnerable = True
self.hurt_time = None
self.invulnerablity_duration = 500
```

Они аналогичны демонам в `enemy.py`. Далее традиционно пропишем смену флага в наш кулдаун-метод:

```python
if not self.vulnerable:
    if current_time - self.hurt_time >= self.invulnerablity_duration:
        self.vulnerable = True
```

Теперь пропишем наше мерцание. Тут всё также, как и ранее. Прописывать будем в animate-методе:

```python
if not self.vulnerable:
    alpha = self.wave_value()
    self.image.set_alpha(alpha)
else:
    self.image.set_alpha(255)
```

Теперь нам нужно восстанавливать энергию (повторим механику из Dark Souls). Для этого создадим метод `energy_recover`:

```python
def energy_recover(self):
    if self.energy < self.stats['energy']:
        self.energy += 0.1
    else:
        self.energy = self.stats['energy']
```

Далее пропишем, что при ударе у нас теряется 10 очков стамины, а если её не хватает — атака не проходит:

```python
if keys[pygame.K_SPACE]:
    if self.energy >= 10:
        self.energy -= 10
        self.attacking = True
        self.attack_time = pygame.time.get_ticks()
        self.create_attak()
        self.weapon_attack_sound.play()
```

Последнее, что я хотел бы сделать — добавить экспу за убийство врага в `level.py`:

```python
def add_xp(self, amount):
    self.player.exp += amount
```

Добавим этот же метод для `create_map` в `Enemy`. Далее повторим всё, что делали ранее с `damage_player`.

[Файлы данного шага тут](https://disk.yandex.ru/d/fH7ArtoIx-sQSw). Приступим к последнему шагу — музыке.
