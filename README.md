<img width="488" height="419" alt="image" src="https://github.com/user-attachments/assets/bc0e58fb-254b-41fb-b21e-19318d053a0c" />
чтобы сделать такое же вам нужно создать любой файл с таким содержимым:

```python
import random
import glfw
import numpy as np
from redbug3d import Engine3D, World
from redbug3d.ecs.component import Transform3D, MeshRenderer, Camera3D, FlyController, Rotator, Light
from redbug3d.ecs.system import System
from redbug3d.graphics.texture import texture_manager
from redbug3d.graphics.mesh import MeshCache
from redbug3d.graphics.camera import PerspectiveCamera
from redbug3d.graphics.shader import VERTEX_SHADER_3D, FRAGMENT_SHADER_3D
from redbug3d.io.input_manager import Input3D

from redbug3d.easy.fcs import FlyControllerSystem
from redbug3d.easy.rotator3d import RotatorSystem
from redbug3d.easy.hex import HEX

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

    world = World()

    player = world.create_entity()
    world.add_component(player, Camera3D())
    world.add_component(player, FlyController(speed=10.0))

    light = world.create_entity()
    world.add_component(light, Transform3D(x=5, y=8, z=5))
    world.add_component(light, Light(color=(1.0, 0.95, 0.8), intensity=1.5, ambient=0.3))

    bug = world.create_entity()
    world.add_component(bug, Transform3D(
        x=8,
        y=3,
        z=6,
        rot_y=random.uniform(0, 6.28),
        scale_x=1,
        scale_y=1,
        scale_z=1
    ))
    world.add_component(bug, MeshRenderer(mesh_type="cube", color=HEX("#ffffff", 1)))
    world.add_component(bug, Rotator(speed_y=random.uniform(0.2, 0.5)))

    world.add_system(FlyControllerSystem(camera))
    world.add_system(RotatorSystem())

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

        # FPS counter
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

            vao = mesh_cache.get_vao(r.mesh_type)
            if not vao:
                continue

            model = t.get_matrix()

            program['uModel'].write(model.T.astype('f4').tobytes())
            program['uView'].write(camera.view_matrix.T.astype('f4').tobytes())
            program['uProjection'].write(camera.proj_matrix.T.astype('f4').tobytes())
            program['uColor'] = r.color
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

как скачать:
```bash
pip install glfw>=2.7.0 moderngl>=5.10.0 numpy>=1.26.0 Pillow>=10.0.0
```
