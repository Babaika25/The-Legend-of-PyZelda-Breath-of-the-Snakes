# Музыка

Мы на финишной прямой. В `player.py` добавим демонов:

```python
self.weapon_attack_sound = pygame.mixer.Sound('../audio/attack/slash.wav')
self.weapon_attack_sound.set_volume(0.4)
```

Первый демон укажет на название файла, а второй на громкость. Далее, вызовем звук при нажатии на клавишу:

```python
self.weapon_attack_sound.play()
```

Повторим успех с ударом от монстра:

```python
self.hit_sound = pygame.mixer.Sound('../audio/attack/claw.wav')
self.hit_sound.set_volume(0.4)
```

```python
self.hit_sound.play()
```

Данный метод я вызвал в get\_damage после успешного попадания. Теперь мы слышим попадания при ударе. А монстр? Добавим ещё пару демонов и вызов:

```python
self.attack_sound = pygame.mixer.Sound(monster_info['attack_sound'])
self.attack_sound.set_volume(0.3)
```

```python
self.attack_sound.play()
```

Вызов функции в методе actions-методе в `enemy.py`. Осталось только главная музыка игры. Переходим в `main.py`:

```python
main_sound = pygame.mixer.Sound('../audio/main.wav')
main_sound.play(loops = -1)
```

Это два новых демона. Отличие только одно — метод `play(loops = -1)`. Тут мы создаём музыку, которая будет играть снова и снова во время игры и не закончится до выхода.

В конце, я забыл, что игра не заканчивается. Я исправил это функцией `final` в `level.py`:

```python
def final(self):
    if self.player.health <= 0:
        print('\n', '\n', 'Линк умер. Хана Хайрулу')
        exit()
    if self.player.exp > 5000:
        print('\n', '\n', 'Линк победил')
        exit()
```

Тут всё просто. При получении более 5000 очков — вы победили, менее — проиграли. Ну и конечно, вернул здоровье и энергию на 100%, а экспу на 0. [Все файлы с доработками оставлю тут](https://disk.yandex.ru/d/iTkKdIS\_vjVurA). Вот что получилось:

{% embed url="https://youtu.be/S56R-x9LNMQ?si=fYC4zy-FmkGwlLyL" %}
