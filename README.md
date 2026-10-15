# Chessbook Clone — Android

Clon de Chessbook con repetición espaciada SM-2, gestión de PGN y tablero interactivo.
Construido con Kotlin + Jetpack Compose + Room.

---

## Estructura del proyecto

```
com.chessbook/
├── data/
│   ├── entity/
│   │   ├── RepertoireNode.kt     ← Entidad Room con campos SRS
│   │   └── Repertoire.kt         ← Entidad de repertorio
│   ├── dao/
│   │   ├── RepertoireNodeDao.kt  ← Queries SQL para jugadas
│   │   └── RepertoireDao.kt      ← Queries SQL para repertorios
│   └── db/
│       └── ChessbookDatabase.kt  ← Base de datos Room + converters
├── domain/
│   ├── algorithm/
│   │   └── SM2Algorithm.kt       ← Algoritmo de repetición espaciada
│   └── model/
│       ├── PgnParser.kt          ← Parser de archivos PGN
│       └── ChessbookRepository.kt← Repositorio (une datos y dominio)
├── ui/
│   ├── training/
│   │   ├── TrainingViewModel.kt  ← Lógica de la sesión de entrenamiento
│   │   └── TrainingScreen.kt     ← UI Compose del tablero + botones SRS
│   ├── repertoire/
│   │   ├── RepertoireViewModel.kt← Lógica de gestión de repertorios
│   │   └── RepertoireScreen.kt   ← UI Compose + dialog de importar PGN
│   └── theme/
│       └── ChessbookTheme.kt     ← Colores y tipografía Material 3
├── di/
│   └── AppContainer.kt           ← Inyección de dependencias manual
└── MainActivity.kt               ← Entry point + navegación raíz
```

---

## Nivel 1 — Principiante: Cómo empezar

1. **Descarga Android Studio** (versión Hedgehog 2023.1.1 o superior)
   https://developer.android.com/studio

2. **Crea un proyecto nuevo**
   - File → New → New Project → Empty Compose Activity
   - Nombre: `Chessbook`
   - Package: `com.chessbook`
   - Lenguaje: Kotlin
   - Min SDK: API 26

3. **Agrega JitPack** en `settings.gradle`:
   ```gradle
   dependencyResolutionManagement {
       repositories {
           google()
           mavenCentral()
           maven { url 'https://jitpack.io' }   // ← agregar esto
       }
   }
   ```

4. **Reemplaza `app/build.gradle`** con el archivo `build.gradle` incluido.

5. **Copia los archivos `.kt`** a sus paquetes correspondientes dentro de
   `app/src/main/java/com/chessbook/`.

6. **En `AndroidManifest.xml`**, agrega el atributo `android:name`:
   ```xml
   <application
       android:name=".ChessbookApp"
       ...>
   ```

7. Haz clic en **Sync Now** y luego **Run ▶**.

---

## Nivel 2 — Intermedio: Flujo de datos

```
PGN (texto) → PgnParser → List<RepertoireNode> → Room DB
                                                       ↓
                                          RepertoireNodeDao.getNextDueNode()
                                                       ↓
                                          TrainingViewModel (StateFlow)
                                                       ↓
                                          TrainingScreen (Compose)
                                                       ↓
                                          Usuario califica: 0/1/2/3
                                                       ↓
                                          SM2Algorithm.calculateNextReview()
                                                       ↓
                                          Room DB actualiza dueDate + interval
```

---

## Nivel 3 — Avanzado: Próximos pasos

### Tablero interactivo real
Reemplaza el `ChessboardView` placeholder con una de estas opciones:

**Opción A — Librería XML (más estable):**
```kotlin
// En build.gradle:
implementation 'io.github.victorfernandez:chessboardview:1.2.0'

// En Compose:
AndroidView(factory = { context ->
    ChessboardView(context).apply {
        setPosition(fen)
        setOnMoveListener { move -> viewModel.validateMove(move) }
    }
})
```

**Opción B — Canvas propio en Compose:**
```kotlin
Canvas(modifier = Modifier.fillMaxSize()) {
    // Dibuja casillas
    for (row in 0..7) for (col in 0..7) {
        drawRect(
            color  = if ((row + col) % 2 == 0) lightColor else darkColor,
            topLeft = Offset(col * sqSize, row * sqSize),
            size   = Size(sqSize, sqSize)
        )
    }
    // Dibuja piezas con drawImage() usando sprites de /drawable
}
```

### Migrar a FSRS (el algoritmo de Chessbook real)
El algoritmo FSRS es más preciso que SM-2. Implementación de referencia:
https://github.com/open-spaced-repetition/fsrs4anki

### Agregar navegación con NavController
```kotlin
// build.gradle:
implementation 'androidx.navigation:navigation-compose:2.7.7'

// NavHost en MainActivity:
NavHost(navController, startDestination = "repertoires") {
    composable("repertoires") { RepertoireScreen(...) }
    composable("training/{id}") { backStack ->
        TrainingScreen(repertoireId = backStack.arguments?.getString("id")?.toLong()!!)
    }
}
```

---

## Algoritmo SM-2 — Resumen rápido

| Calificación | Acción                        | Próximo repaso |
|:------------:|-------------------------------|:--------------:|
| 0 — Otra vez | Reinicia el ciclo             | < 1 minuto     |
| 1 — Difícil  | Intervalo × 1.2 (mín +1 día) | Variable       |
| 2 — Bien     | Intervalo × EaseFactor        | Variable       |
| 3 — Fácil    | Intervalo × EaseFactor × 1.3  | Variable       |

El **EaseFactor** comienza en 2.5 y se ajusta con cada calificación.
Nunca baja de 1.3 para evitar intervalos demasiado cortos.

---

## Licencia

MIT — úsalo, modifícalo y distribúyelo libremente.
