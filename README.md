<div align="center">
  
  # REDBUG ENGINE
  
  **Игровой движок для каждого компьютера**
  
  [![Python](https://img.shields.io/badge/Python-3.12+-blue.svg)](https://python.org)
  [![OpenGL](https://img.shields.io/badge/OpenGL-2.1+-green.svg)](https://opengl.org)
  [![Size](https://img.shields.io/badge/Size-78KB-red.svg)](#)
  [![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
  
</div>

---

## Что это?

**REDBUG Engine** — это легкий, быстрый и простой 3D игровой движок на Python.

- **малый размер** — меньше всех остальных движков
- **Мгновенный запуск** — без компиляции шейдеров
- **Чистый Python** — легко учиться

---

## ДЕМО

Вот так выглядит REDBUG в действии:

```python
import random
import glfw
import numpy as np
import moderngl
from redbug3d import Engine3D, World
from redbug3d.ecs.component import Transform3D, MeshRenderer, Camera3D, FlyController, Rotator, Light
from redbug3d.ecs.system import System
from redbug3d.graphics.texture import texture_manager
from redbug3d.graphics.mesh import MeshCache
from redbug3d.graphics.camera import PerspectiveCamera
from redbug3d.graphics.shader import VERTEX_SHADER_3D, FRAGMENT_SHADER_3D
from redbug3d.io.input_manager import Input3D

from redbug3d.utils.pcs import PlayerPhysics, PlayerPhysicsSystem
from redbug3d.utils.rotator3d import RotatorSystem
from redbug3d.utils.hex import HEX

def main():
    engine = Engine3D(title="RedBug 3D Engine", width=1280, height=720)
    engine.init()

    texture_manager.initialize(engine.ctx)

    program = engine.ctx.program(
        vertex_shader=VERTEX_SHADER_3D,
        fragment_shader=FRAGMENT_SHADER_3D
    )

    mesh_cache = MeshCache(engine.ctx)
    mesh_cache.set_program(program)

    camera = PerspectiveCamera(1280, 720)
    # ВОТ ЗДЕСЬ МЕНЯЙ ВЫСОТУ (1.8 МЕТРА)
    camera.position = np.array([0.0, 1.8, 5.0])  # <-- ЭТО РЕАЛЬНАЯ ВЫСОТА

    world = World()

    player = world.create_entity()
    world.add_component(player, Camera3D())
    world.add_component(player, PlayerPhysics(
        speed=5.0,
        height=1.8,        # <-- И ЗДЕСЬ ТОЖЕ (для гравитации)
        gravity=-15.0,
        jump_speed=6.0
    ))

    light = world.create_entity()
    world.add_component(light, Transform3D(x=5, y=8, z=5))
    world.add_component(light, Light(
        color=(1.0, 1.0, 1.0),
        intensity=5.0,
        ambient=0.5
    ))
    
    grass = world.create_entity()
    world.add_component(grass, Transform3D(
        x=8,
        y=1,
        z=6,
        rot_y=0
    ))
    world.add_component(grass, MeshRenderer(mesh_type="grass"))

    gun = world.create_entity()
    world.add_component(gun, Transform3D(
        x=8,
        y=2,
        z=6,
        rot_y=0
    ))
    world.add_component(gun, MeshRenderer(mesh_type="gun"))
    
    world.add_system(PlayerPhysicsSystem(camera))

    light_entities = list(world.get_entities_with(Light, Transform3D))
    light_entity = light_entities[0]
    light_trans = world.get_component(light_entity, Transform3D)
    light_comp = world.get_component(light_entity, Light)

    engine.running = True
    engine._last_time = glfw.get_time()

    while engine.running and not glfw.window_should_close(engine.window):
        now = glfw.get_time()
        engine._delta_time = now - engine._last_time
        engine._last_time = now

        engine._frame_count += 1
        engine._fps_timer += engine._delta_time
        if engine._fps_timer >= 1.0:
            engine.fps = engine._frame_count
            engine._frame_count = 0
            engine._fps_timer -= 1.0
            glfw.set_window_title(engine.window, f"{engine.fps} FPS")

        Input3D._update()
        glfw.poll_events()

        if Input3D.get_key(glfw.KEY_ESCAPE):
            engine.running = False

        engine.ctx.clear(*engine.clear_color)
        world.update(engine._delta_time)
        camera._update_matrices()

        for entity in world.get_entities_with(Transform3D, MeshRenderer):
            t = world.get_component(entity, Transform3D)
            r = world.get_component(entity, MeshRenderer)

            model = t.get_matrix()
            
            vaos = mesh_cache.get_all_vaos(r.mesh_type)
            for vao, submesh in vaos:
                if not vao:
                    continue
                    
                program['uModel'].write(model.T.astype('f4').tobytes())
                program['uView'].write(camera.view_matrix.T.astype('f4').tobytes())
                program['uProjection'].write(camera.proj_matrix.T.astype('f4').tobytes())
                
                if submesh.texture:
                    submesh.texture.use(0)
                    program['uTexture'] = 0
                    program['uUseTexture'] = 1
                    program['uColor'] = (1.0, 1.0, 1.0, 1.0)
                elif submesh.color:
                    program['uColor'] = submesh.color
                    program['uUseTexture'] = 0
                else:
                    program['uColor'] = r.color
                    program['uUseTexture'] = 0
                
                program['uLightPos'] = (light_trans.x, light_trans.y, light_trans.z)
                program['uLightColor'] = light_comp.color
                program['uLightIntensity'] = light_comp.intensity
                program['uAmbient'] = light_comp.ambient
                program['uCameraPos'] = (camera.position[0], camera.position[1], camera.position[2])

                vao.render()

        glfw.swap_buffers(engine.window)

    engine.shutdown()

if __name__ == "__main__":
    main()
```
