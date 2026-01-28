# Super Kirby Bro

##Descripción del Proyecto
**Super Kirby Bro** es un videojuego de plataformas 2D desarrollado en Python con la librería `pygame`. El juego presenta un sistema de generación procedural de mundos, donde el jugador debe atravesar diferentes biomas (Pasto, Desierto, Hielo), esquivar trampas, vencer enemigos y recolectar potenciadores (PowerUps) para llegar a la meta.

El proyecto destaca por su arquitectura de software modular y la implementación explícita de **7 Patrones de Diseño** clásicos, demostrando buenas prácticas de programación orientada a objetos.

---
### Diagrama de clases UML
El diagrama de el proyecto a continuación implementa 7 patrones de diseño:

**Observer** : Para notificación automática

**Strategy** : Para cambiar algoritmos en tiempo de ejecución

**Command** : Para ejecutar acciones con historial y deshacer

**Memento** : Para guardar y restaurar estados

**Template Method** : Para definir algoritmos generales que se especializan

**Registry (Service Locator)** : Para localizar servicios globales

**Flyweight** : Para optimizar objetos pesados reutilizables

<img width="2338" height="668" alt="image" src="https://github.com/user-attachments/assets/8f8c73b2-e721-4976-aaec-f3073eacda08" />


##Arquitectura y Patrones de Diseño Aplicados

A continuación se detallan los patrones de diseño implementados en el código fuente:

### 1. Flyweight Pattern

**¿Qué es el patrón?**
Flyweight (Peso Mosca) es un patrón estructural que permite soportar una gran cantidad de objetos de forma eficiente, compartiendo el estado común (intrínseco) entre ellos en lugar de mantenerlo en cada objeto individual.

**Donde fue aplicado junto con código**
Se aplicó en `Powerups_Enemies.py` para gestionar los recursos gráficos (sprites) de Enemigos y PowerUps. En lugar de cargar las imágenes para cada enemigo, todos comparten una única instancia de `SpriteFlyweight`.

*Archivo: `Powerups_Enemies.py`*
```python
class SpriteFlyweight:
    """Mantiene el estado intrínseco (imágenes compartidas)"""
    def __init__(self, sprite_type: str, sprites: list):
        self._sprite_type = sprite_type
        self._sprites = sprites  # Recurso pesado compartido

class SpriteFlyweightFactory:
    """Fábrica que asegura que los Flyweights se reutilicen"""
    _flyweights = {}
    
    @classmethod
    def get_flyweight(cls, sprite_type, width, height):
        key = f"{sprite_type}_{width}x{height}"
        if key not in cls._flyweights:
            # Crea solo si no existe
            cls._flyweights[key] = cls._create_flyweight(...)
        return cls._flyweights[key]
```

**¿Por qué fue utilizado?**
Para **optimizar el uso de memoria RAM**. El juego puede generar docenas de enemigos y powerups; cargar las mismas imágenes `.png` para cada instancia sería ineficiente. Con Flyweight, las imágenes se cargan una sola vez por tipo.

---

### 2. Strategy Pattern

**¿Qué es el patrón?**
Strategy (Estrategia) es un patrón de comportamiento que permite definir una familia de algoritmos, encapsular cada uno en una clase separada y hacer sus objetos intercambiables. Permite variar el comportamiento de un objeto en tiempo de ejecución.

**Donde fue aplicado junto con código**
Se aplicó en `Powerups_Enemies.py` para definir los efectos de los distintos PowerUps (Velocidad, Salto, Vida).

*Archivo: `Powerups_Enemies.py`*
```python
class PowerUpStrategy(ABC):
    """Interfaz común para todas las estrategias"""
    @abstractmethod
    def apply(self, player):
        pass

class SpeedBoostStrategy(PowerUpStrategy):
    """Estrategia concreta: Aumentar velocidad"""
    def apply(self, player):
        player.increase_speed(self.boost_amount)
        print(f"¡Velocidad aumentada a {player.speed}!")

class PowerUpContext:
    """Contexto que usa la estrategia"""
    def apply_power(self, player):
        if self._strategy:
            self._strategy.apply(player)
```

**¿Por qué fue utilizado?**
Para cumplir con el **Principio Abierto/Cerrado (OCP)**. Podemos agregar nuevos tipos de PowerUps (ej: Invisibilidad, Fuerza) simplemente creando nuevas clases de estrategia sin modificar la clase base del PowerUp ni el código del jugador.

---

### 3. Registry Pattern

**¿Qué es el patrón?**
Registry es un patrón (a menudo considerado una variación de Service Locator) que permite almacenar y recuperar servicios u objetos globales conocidos (como configuraciones o prototipos) a través de una interfaz común, desacoplando su acceso.

**Donde fue aplicado junto con código**
Se aplicó en `world_generator.py` para registrar qué tipos de PowerUps están disponibles para aparecer en un mundo generado.

*Archivo: `world_generator.py`*
```python
class PowerUpTypeRegistry:
    """Registro de tipos disponibles para generación"""
    def __init__(self):
        self._available_types = set()
    
    def register(self, powerup_type: str):
        self._available_types.add(powerup_type)
        
    def get_random_type(self, probabilities):
        # Lógica para seleccionar un tipo registrado aleatoriamente
        pass
```

**¿Por qué fue utilizado?**
Para **desacoplar la generación de la definición**. Permite configurar dinámicamente qué powerups pueden aparecer en un nivel específico sin "hardcodear" la lista en el algoritmo de generación.

---

### 4. Template Method Pattern

**¿Qué es el patrón?**
Template Method define el esqueleto de un algoritmo en una operación, difiriendo algunos pasos a las subclases. Permite redefinir ciertos pasos de un algoritmo sin cambiar su estructura general.

**Donde fue aplicado junto con código**
Se aplicó en `world_generator.py` para el proceso de generación de niveles. La clase base define el orden (Checkpoints -> Plataformas -> Trampas -> Enemigos) y las subclases definen los detalles específicos (configuración de bioma).

*Archivo: `world_generator.py`*
```python
class WorldGenerator(ABC):
    def generate_world(self, width, height):
        """El Template Method: Define el esqueleto del algoritmo"""
        config = self.get_world_config() # Paso abstracto
        
        # Pasos definidos (comunes o parametrizados)
        checkpoints = self._generate_checkpoints(width)
        platforms = self._generate_platforms_with_config(...)
        enemies = self._generate_enemies(...)
        
        return world_data

class IceWorldGenerator(WorldGenerator):
    def get_world_config(self):
        """Implementación específica para el nivel de hielo"""
        return WorldConfig(name="Mundo Hielo", colors={...}, ...)
```

**¿Por qué fue utilizado?**
Para **reutilizar código** y garantizar una estructura consistente en la generación de niveles. Todos los mundos siguen las mismas reglas físicas y de lógica, pero varían en parámetros (colores, dificultad, probabilidad de trampas).

---

### 5. Memento Pattern

**¿Qué es el patrón?**
Memento permite capturar y externalizar el estado interno de un objeto para que el objeto pueda ser restaurado a este estado más tarde, sin violar la encapsulación.

**Donde fue aplicado junto con código**
Se aplicó en `memento.py` y `entities.py` para el sistema de **Checkpoints**.

*Archivo: `entities.py` (Originator)*
```python
class Player:
    def create_memento(self):
        """Guarda el estado actual (posición, velocidad, poderes)"""
        return PlayerMemento(self.x, self.y, self.speed, self.jump_power)

    def restore_from_memento(self, memento):
        """Restaura el estado desde el memento"""
        state = memento.get_state()
        self.x = state['x']
        self.y = state['y']
        # ... restaura otros atributos
```

*Archivo: `memento.py` (Caretaker)*
```python
class CheckpointManager:
    """Gestiona los mementos guardados"""
    def save_checkpoint(self, id, memento):
        self._checkpoints[id] = memento
        
    def get_last_checkpoint(self):
        return self._checkpoints.get(self._current_checkpoint)
```

**¿Por qué fue utilizado?**
Para implementar una funcionalidad de **"Guardar y Cargar"** (Checkpoints) segura. Permite que el jugador respawnee con las mismas estadísticas que tenía al tocar el checkpoint, sin que el gestor del juego necesite conocer las variables internas del jugador.

---

### 6. Command Pattern

**¿Qué es el patrón?**
Command encapsula una petición como un objeto, permitiendo parametrizar clientes con diferentes solicitudes, encolar o registrar solicitudes y soportar operaciones que pueden deshacerse.

**Donde fue aplicado junto con código**
Se aplicó en `commands.py` para manejar los controles del jugador (Input).

*Archivo: `commands.py`*
```python
class Command(ABC):
    @abstractmethod
    def execute(self, player):
        pass

class MoveRightCommand(Command):
    def execute(self, player):
        player.move_right()

class JumpCommand(Command):
    def execute(self, player):
        player.jump()

class InputHandler:
    """Invoker: Mapea teclas a comandos"""
    def handle_input(self, keys, player):
        if keys[pygame.K_RIGHT]:
            self.commands['right'].execute(player)
        if keys[pygame.K_SPACE]:
            self.commands['jump'].execute(player)
```

**¿Por qué fue utilizado?**
Para **desacoplar la entrada del teclado de la lógica del juego**. Facilita la configuración de controles (remapping) y permite tratar las acciones del usuario como objetos.

---

### 7. Observer Pattern

**¿Qué es el patrón?**
Observer define una dependencia uno-a-muchos entre objetos para que cuando un objeto cambie de estado, todos sus dependientes sean notificados y actualizados automáticamente.

**Donde fue aplicado junto con código**
Se aplicó en `game_events.py` para gestionar los eventos del juego (Muerte, Meta alcanzada, Checkpoint activado, etc.).

*Archivo: `game_events.py`*
```python
class GameEventManager:
    """Subject: Notifica eventos a los observadores"""
    def __init__(self):
        self._observers = []

    def subscribe(self, observer):
        self._observers.append(observer)

    def notify(self, event):
        for observer in self._observers:
            observer.on_notify(event)
```

*Archivo: `game.py`*
```python
# Observer concreto
class ConsoleLogger(GameEventObserver):
    def on_notify(self, event):
        print(f"[LOG] Evento: {event.type.name} -> {event.data}")

# Uso
self.event_manager.notify(GameEvent(GameEventType.PLAYER_DIED, {}))
```

**¿Por qué fue utilizado?**
Para tener un **sistema de eventos desacoplado**. Diferentes subsistemas (UI, Audio, Logros, lógica de fin de juego) pueden reaccionar a lo que sucede en el núcleo del juego sin que el núcleo tenga que llamarlos explícitamente.

---

##Patrón NO Implementado

### Singleton Pattern

**¿Qué es el Singleton?**
Es un patrón creacional que garantiza que una clase tenga una única instancia y proporciona un punto de acceso global a ella.

**¿Por qué NO se implementó en este programa?**
Aunque clases como `Game` o `TextureManager` suelen ser candidatos comunes para Singleton, decidimos **no implementarlo** por las siguientes razones:

1.  **Testabilidad y Flexibilidad**: El patrón Singleton introduce un estado global difícil de limpiar entre pruebas o reinicios completos del sistema. Al evitarlo, podemos crear y destruir instancias de la clase `Game` libremente (por ejemplo, para volver al menú principal y empezar una partida totalmente nueva desde cero).
2.  **Inyección de Dependencias**: Preferimos pasar las instancias necesarias (como `GameEventManager` o `CheckpointManager`) explícitamente a las clases que las necesitan (ej: `CollisionManager` recibe el `event_manager` en su constructor). Esto hace que las dependencias sean claras y evita el acoplamiento oculto que genera el Singleton.
3.  **Control del Ciclo de Vida**: Al instanciar las clases explícitamente en el `main` o en el constructor de `Game`, tenemos control total sobre cuándo se crean y destruyen, evitando problemas de inicialización estática.

---
*Proyecto final realizado para la asignatura Modelos de Programación*
