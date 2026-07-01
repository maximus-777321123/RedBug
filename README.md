<div align="center">
  
  <img src="https://github.com/user-attachments/assets/bc0e58fb-254b-41fb-b21e-19318d053a0c" alt="RedBug Engine Logo" width="488">
  
  # 🐞 REDBUG ENGINE
  
  **Игровой движок для каждого компьютера**
  
  [![Python](https://img.shields.io/badge/Python-3.12+-blue.svg)](https://python.org)
  [![OpenGL](https://img.shields.io/badge/OpenGL-2.1+-green.svg)](https://opengl.org)
  [![Size](https://img.shields.io/badge/Size-78KB-red.svg)](#)
  [![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
  
</div>

---

## 📦 Что это?

**REDBUG Engine** — это легкий, быстрый и простой 3D игровой движок на Python.

- 🔥 **73 KB** — меньше одной фотографии
- 🚀 **Мгновенный запуск** — без компиляции шейдеров
- 🐍 **Чистый Python** — легко учиться

---

## 🎮 ДЕМО

Вот так выглядит REDBUG в действии:

```python
import random
from redbug3d import Engine3D, World
from redbug3d.ecs.component import Transform3D, MeshRenderer, Camera3D, FlyController, Rotator, Light
from redbug3d.easy.hex import HEX

def main():
    engine = Engine3D(title="RedBug Engine", width=1280, height=720)
    
    world = World()
    
    # Камера (свободный полет)
    player = world.create_entity()
    world.add_component(player, Camera3D())
    world.add_component(player, FlyController(speed=10.0))
    
    # Свет
    light = world.create_entity()
    world.add_component(light, Transform3D(x=5, y=8, z=5))
    world.add_component(light, Light(color=(1.0, 0.95, 0.8), intensity=1.5, ambient=0.3))
    
    # Божья коровка (красный жук!)
    bug = world.create_entity()
    world.add_component(bug, Transform3D(
        x=8, y=3, z=6,
        rot_y=random.uniform(0, 6.28)
    ))
    world.add_component(bug, MeshRenderer(mesh_type="cube", color=HEX("#ffffff")))
    world.add_component(bug, Rotator(speed_y=random.uniform(0.2, 0.5)))
    
    # Запуск!
    engine.run(world)

if __name__ == "__main__":
    main()
