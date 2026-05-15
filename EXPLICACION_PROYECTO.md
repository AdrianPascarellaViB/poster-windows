# Wordle en Windows Forms (C#) — Explicación completa del proyecto

---

## Índice
1. [Qué es el programa](#1-qué-es-el-programa)
2. [Estructura del proyecto](#2-estructura-del-proyecto)
3. [Cómo se hizo con Visual Studio](#3-cómo-se-hizo-con-visual-studio)
4. [Formularios — explicación detallada](#4-formularios--explicación-detallada)
5. [Lógica del juego — WordleGame.cs](#5-lógica-del-juego--wordlegamecs)
6. [Flujo completo de una partida](#6-flujo-completo-de-una-partida)
7. [Estructuras de datos clave](#7-estructuras-de-datos-clave)
8. [El algoritmo de evaluación de letras](#8-el-algoritmo-de-evaluación-de-letras)
9. [Preguntas del profesor y respuestas](#9-preguntas-del-profesor-y-respuestas)

---

## 1. Qué es el programa

Es una implementación del juego **Wordle** en catalán, como aplicación de escritorio con Windows Forms (.NET 10). El jugador tiene **6 intentos** para adivinar una palabra secreta de **5 letras**. Tras cada intento, el juego colorea las letras:

- **Verde**: la letra está en la posición correcta.
- **Amarillo-naranja**: la letra está en la palabra pero en otra posición.
- **Gris**: la letra no aparece en la palabra.

La particularidad de este Wordle es que la palabra secreta **la introduce el propio usuario** (otro jugador, o tú mismo), en vez de ser aleatoria.

---

## 2. Estructura del proyecto

```
WordleApp/
├── Program.cs               ← Punto de entrada de la aplicación
├── WordleGame.cs            ← Toda la lógica del juego (modelo)
├── Form1.cs                 ← Ventana principal del juego (vista + controlador)
├── Form1.Designer.cs        ← Layout generado por el diseñador de VS
├── InputWordForm.cs         ← Formulario para introducir la palabra secreta
├── InputWordForm.Designer.cs
├── StatsForm.cs             ← Formulario de estadísticas
├── StatsForm.Designer.cs
├── HelpForm.cs              ← Formulario de ayuda ("Com jugar")
├── HelpForm.Designer.cs
└── WordleApp.csproj         ← Configuración del proyecto (.NET 10, WinForms)
```

Los ficheros `.Designer.cs` son **generados automáticamente** por el diseñador visual de Visual Studio. Nunca se editan a mano.

---

## 3. Cómo se hizo con Visual Studio

### Paso 1 — Crear el proyecto
- Se abrió Visual Studio → "Nuevo proyecto" → **Windows Forms App (.NET)**.
- Se puso el nombre `WordleApp` y se seleccionó .NET 10.

### Paso 2 — Diseñar Form1 (ventana principal)
- Se abrió `Form1.cs` en el **diseñador visual** (doble clic en el archivo).
- Desde el **Cuadro de herramientas** (Toolbox) se arrastraron controles:
  - **Label** para el título "WORDLE" (con fuente Arial 24pt Bold desde el panel Propiedades).
  - **30 Labels** para la cuadrícula (6 filas × 5 columnas), colocados manualmente y alineados con las guías de snap del diseñador. Se nombran `lblGrid_0_0` hasta `lblGrid_5_4`.
  - **28 Buttons** para el teclado virtual (filas QWERTY, ASDFG, ZXCVB + ENTER y BACK).
  - **3 Buttons** más: ayuda (?), estadísticas (📈) y nueva partida.
- Las propiedades de cada control (color, fuente, tamaño, nombre) se ajustaron desde el panel **Propiedades** de Visual Studio.
- Para los event handlers, se hizo **doble clic** sobre cada botón en el diseñador para que VS generara automáticamente el método `btnX_Click` en `Form1.cs`.

### Paso 3 — Crear los formularios secundarios
- Menú → **Proyecto → Agregar → Windows Form** para añadir `InputWordForm`, `StatsForm` y `HelpForm`.
- Cada uno se diseñó igual: arrastrar controles desde el Toolbox, ajustar propiedades, doble clic para eventos.

### Paso 4 — Crear la clase de lógica
- Menú → **Proyecto → Agregar → Clase** → `WordleGame.cs`.
- Esta clase no tiene interfaz gráfica, contiene puro C#.

### Paso 5 — Conectar lógica con interfaz
- En `Form1.cs` se instancia `WordleGame` y se llaman sus métodos desde los event handlers de los botones y el teclado.

---

## 4. Formularios — explicación detallada

### Form1 — Ventana principal

Es el corazón de la aplicación. Contiene:

**Variables de instancia importantes:**
```csharp
WordleGame game;                        // instancia del motor del juego
GameStats stats;                        // estadísticas de la sesión
Label[,] gridLabels;                    // matriz 6x5 con los Labels de la cuadrícula
Dictionary<char, Button> keyboardButtons; // mapea cada letra a su botón del teclado
```

**Los colores** están definidos como constantes:
```csharp
Color colorCorrect   = Color.FromArgb(76, 175, 80);    // verde
Color colorMisplaced = Color.FromArgb(201, 180, 88);   // amarillo-naranja
Color colorIncorrect = Color.FromArgb(120, 124, 126);  // gris
```

**Flujo de input del usuario:**
- El formulario tiene `KeyPreview = true`, lo que hace que los eventos de teclado lleguen al formulario antes que a los controles hijos. Esto permite capturar letras, ENTER y BACKSPACE desde el teclado físico.
- Los botones del teclado virtual también llaman al mismo método central: `HandleVirtualKeyPress(string key)`.

**HandleVirtualKeyPress:** es el método más importante de Form1. Gestiona toda la entrada:
- Si es una letra → llama a `game.AddLetter()` y actualiza la fila actual visualmente.
- Si es "BACK" → llama a `game.RemoveLetter()` y actualiza la fila.
- Si es "ENTER" → llama a `game.SubmitGuess()`, colorea la fila y actualiza el teclado.

**AnimateRowAndColorize:** recibe el array de `GuessResult[]` y pinta cada celda de la fila. También actualiza los colores del teclado virtual, pero con una regla importante: **el color nunca puede retroceder**. Si una tecla ya está verde, no puede volverse gris aunque aparezca gris en un intento posterior.

---

### InputWordForm — Introducir la palabra

Formulario modal (se abre con `ShowDialog()`). Tiene:
- Un `TextBox` con `UseSystemPasswordChar = true` → muestra asteriscos para que nadie vea la palabra que se introduce.
- `CharacterCasing = Upper` → convierte automáticamente a mayúsculas.
- Validación al pulsar "COMENZAR": debe tener exactamente 5 letras y solo letras (sin números ni símbolos).
- La palabra validada se expone como `SelectedWord` (propiedad pública).

En Form1, se usa así:
```csharp
using (var inputForm = new InputWordForm()) {
    if (inputForm.ShowDialog() == DialogResult.OK) {
        game.StartNewGame(inputForm.SelectedWord);
        ResetUI();
    }
}
```

---

### StatsForm — Estadísticas

Muestra 4 métricas usando Labels:
- **Jugades** → `stats.GamesPlayed`
- **Èxits (%)** → `(stats.Wins / stats.GamesPlayed) * 100`
- **Ratxa Actual** → `stats.CurrentStreak` (rachas consecutivas)
- **Ratxa Màxima** → `stats.MaxStreak`

También tiene un botón **Compartir** que copia al portapapeles una cuadrícula de emojis (🟩🟧⬛) representando la partida, para poder pegarla en WhatsApp o Twitter.

---

### HelpForm — Ayuda

Formulario estático que explica las reglas. Todo el contenido está en el Designer (Labels con texto y Labels coloreados como ejemplos). No tiene lógica en el `.cs`, solo `InitializeComponent()`.

---

## 5. Lógica del juego — WordleGame.cs

### Enums y clases auxiliares

```csharp
enum LetterState { Empty, Checking, Correct, Misplaced, Incorrect }

class GuessResult {
    char Letter;
    LetterState State;
}

class GameStats {
    int GamesPlayed;
    int Wins;
    int CurrentStreak;
    int MaxStreak;
    string LastGameShareGrid; // para el botón compartir
}
```

### Clase WordleGame

Constantes: `MaxGuesses = 6`, `WordLength = 5`.

Campos principales:
- `string targetWord` → la palabra a adivinar (secreta).
- `List<string> guesses` → historial de intentos enviados.
- `string currentGuess` → letras escritas en el intento actual (aún no enviado).

Métodos principales:

| Método | Qué hace |
|--------|----------|
| `StartNewGame(string word)` | Inicia nueva partida con la palabra dada. Resetea todo. |
| `AddLetter(char c)` → bool | Añade una letra al intento actual. Devuelve false si ya hay 5 letras o el juego acabó. |
| `RemoveLetter()` → bool | Borra la última letra. Devuelve false si está vacío o el juego acabó. |
| `SubmitGuess()` → GuessResult[] | Evalúa el intento actual, actualiza estadísticas y devuelve los resultados por letra. |
| `UpdateStats(bool won)` | Privado. Actualiza GamesPlayed, Wins, CurrentStreak y MaxStreak. |

---

## 6. Flujo completo de una partida

```
1. Usuario hace clic en "Nova Partida"
        ↓
2. Se abre InputWordForm (modal)
   - El jugador 1 escribe la palabra secreta (se ve como asteriscos)
   - Pulsa "COMENZAR" → validación → DialogResult.OK
        ↓
3. Form1 llama a game.StartNewGame("PALABRA") y ResetUI()
   - ResetUI() limpia todos los labels y resetea colores del teclado
        ↓
4. Bucle de juego (hasta 6 intentos o acierto):
   a. Usuario escribe letras (teclado físico o virtual)
      → game.AddLetter(c) → UpdateCurrentRowDisplay()
   b. Pulsa BACK
      → game.RemoveLetter() → UpdateCurrentRowDisplay()
   c. Pulsa ENTER (con 5 letras)
      → results = game.SubmitGuess()
      → AnimateRowAndColorize(fila, results)
      → Si game.IsGameOver → mostrar mensaje + GenerateShareGrid() + StatsForm
        ↓
5. Fin de partida:
   - Ganó: MessageBox "Has encertat!" (con número de intentos)
   - Perdió: MessageBox con la palabra correcta
   - Se actualiza stats y se abre StatsForm automáticamente
```

---

## 7. Estructuras de datos clave

| Estructura | Tipo C# | Dónde está | Para qué sirve |
|---|---|---|---|
| `gridLabels` | `Label[6, 5]` | Form1 | Acceso rápido a cualquier celda de la cuadrícula por fila y columna |
| `keyboardButtons` | `Dictionary<char, Button>` | Form1 | Buscar el botón del teclado a partir de una letra |
| `guesses` | `List<string>` | WordleGame | Historial de palabras enviadas |
| `targetWord` | `string` | WordleGame | La palabra secreta |
| `currentGuess` | `string` | WordleGame | Las letras del intento en curso |
| `targetUsed` | `bool[]` | SubmitGuess (local) | Evitar contar la misma posición dos veces en el algoritmo |
| `results` | `GuessResult[]` | SubmitGuess (local) | Resultado letra a letra del intento |

---

## 8. El algoritmo de evaluación de letras

Este es el punto más técnico. El juego tiene que manejar letras repetidas correctamente. Por ejemplo, si la palabra es "PANEL" y el intento es "PAPAL":

```
P → Correct   (posición 0 correcta)
A → Misplaced (hay una A en posición 1 de PANEL)
P → Incorrect (la P ya fue usada en posición 0)
A → Incorrect (la A ya fue usada)
L → Correct   (posición 4 correcta)
```

El algoritmo usa **dos pasadas** y un array `targetUsed[5]` para marcar posiciones ya asignadas:

```
Primera pasada: marcar CORRECT (coincidencia exacta de posición)
  Para cada i de 0 a 4:
    Si guess[i] == target[i]:
      results[i] = Correct
      targetUsed[i] = true   ← esta posición ya no puede usarse para Misplaced

Segunda pasada: marcar MISPLACED (letra existe pero en otra posición)
  Para cada i de 0 a 4:
    Si results[i] ya es Correct → saltar
    Buscar en target una posición j donde:
      - target[j] == guess[i]
      - targetUsed[j] == false
    Si se encuentra:
      results[i] = Misplaced
      targetUsed[j] = true   ← consumida, no puede usarse de nuevo
    Si no → results[i] = Incorrect
```

---

## 9. Preguntas del profesor y respuestas

### Sobre la estructura general

**P: ¿Qué patrón de diseño usas?**
> Uso una separación entre lógica y presentación. `WordleGame.cs` es el modelo: no sabe nada de interfaz gráfica. `Form1.cs` es la vista y el controlador: recibe el input del usuario, llama al modelo y actualiza la UI con los resultados. No es MVC puro porque Form1 hace de vista y controlador a la vez, pero la lógica de negocio está completamente separada en `WordleGame`.

**P: ¿Por qué tienes un fichero `.Designer.cs` para cada formulario?**
> Visual Studio genera automáticamente ese fichero cuando usas el diseñador visual. Contiene el código de inicialización de los controles (`InitializeComponent()`). Se mantiene separado del `.cs` principal para no mezclar el código generado con el código que tú escribes. Si editas el `.Designer.cs` a mano y luego usas el diseñador, VS puede sobreescribir tus cambios.

---

### Sobre los formularios

**P: ¿Cómo abres un formulario desde otro en Windows Forms?**
> Hay dos formas. Con `Show()` el formulario secundario es no modal, ambos están activos a la vez. Con `ShowDialog()` es modal: bloquea el formulario padre hasta que se cierra. Yo uso `ShowDialog()` para `InputWordForm`, `StatsForm` y `HelpForm` porque necesito que el usuario complete la acción antes de volver al juego.

**P: ¿Cómo sabes qué palabra introdujo el usuario en InputWordForm?**
> La clase `InputWordForm` tiene una propiedad pública `SelectedWord`. Cuando el usuario pulsa "COMENZAR" y la validación es correcta, el método `BtnOk_Click` asigna `this.SelectedWord = txtWord.Text` y establece `this.DialogResult = DialogResult.OK`. En Form1, después del `ShowDialog()`, compruebo si el resultado fue `OK` y entonces leo `inputForm.SelectedWord`.

**P: ¿Por qué el TextBox de InputWordForm muestra asteriscos?**
> Porque tiene la propiedad `UseSystemPasswordChar = true`. Esto es importante para que el segundo jugador no vea la palabra que introduce el primero. Es la misma propiedad que usan los campos de contraseña.

**P: ¿Cómo haces que el TextBox solo acepte mayúsculas?**
> Con la propiedad `CharacterCasing = Upper`. Windows Forms convierte automáticamente cualquier letra a mayúscula al escribirla, sin necesidad de código en el evento `TextChanged`.

---

### Sobre el teclado y el input

**P: ¿Cómo capturas las teclas del teclado físico?**
> En `Form1` puse la propiedad `KeyPreview = true`. Esto hace que los eventos de teclado (`KeyPress`, `KeyDown`) lleguen primero al formulario antes que a los controles. En `Form1_KeyPress` capturo las letras, y en `ProcessCmdKey` capturo ENTER y BACKSPACE, porque esas teclas no llegan a `KeyPress` (son teclas de comando).

**P: ¿Por qué tienes un método `HandleVirtualKeyPress` en vez de poner el código en cada botón?**
> Porque tanto el teclado físico como los 28 botones virtuales hacen lo mismo. Si pusiera el código en cada `btnQ_Click`, `btnW_Click`, etc., estaría repitiendo el mismo código 28 veces. Con un método central, todos llaman a `HandleVirtualKeyPress("Q")`, `HandleVirtualKeyPress("W")`, etc. Es el principio DRY (Don't Repeat Yourself).

**P: ¿Cómo tienes mapeados los botones del teclado?**
> Uso un `Dictionary<char, Button>` llamado `keyboardButtons`. En el constructor de Form1, relleno el diccionario asociando cada carácter (como `'Q'`, `'A'`) con su botón correspondiente. Así, cuando necesito cambiar el color de la tecla `'G'`, solo hago `keyboardButtons['G'].BackColor = colorCorrect` sin tener que buscar el botón manualmente.

---

### Sobre la lógica del juego

**P: ¿Cómo funciona el algoritmo de evaluación de letras?**
> Usa dos pasadas sobre el intento. En la primera, marco las letras en posición correcta (verde) y marco esas posiciones como "usadas" con un array `bool[] targetUsed`. En la segunda pasada, para cada letra no verde, busco si esa letra existe en la palabra secreta en alguna posición no usada. Si la encuentro, es amarilla y marco esa posición como usada. Si no, es gris. Las dos pasadas y el array `targetUsed` son necesarios para manejar bien las letras repetidas.

**P: ¿Por qué el color del teclado no puede "retroceder"?**
> Porque es información acumulada. Si en un intento anterior sé que la 'A' está en la palabra (amarillo), esa información sigue siendo válida aunque en un intento posterior la 'A' no aparezca (porque la pusiste en posición incorrecta). Si el color retrocediera, perdería información. El código lo implementa con condiciones: si `kbBtn.BackColor == colorCorrect` no cambio nada; si es `colorMisplaced` y el nuevo resultado sería `colorIncorrect` tampoco cambio.

**P: ¿Cómo detectas si el jugador ha ganado o perdido?**
> En `SubmitGuess()`, después de evaluar el intento y añadirlo a la lista `guesses`:
> - **Ganado**: si `currentGuess == targetWord` (comparación exacta).
> - **Perdido**: si `guesses.Count >= MaxGuesses` (6 intentos) y no ha ganado.
> En ambos casos, pongo `IsGameOver = true` y llamo a `UpdateStats()`.

**P: ¿Qué pasa si el jugador escribe una letra cuando ya hay 5?**
> El método `AddLetter()` comprueba `if (currentGuess.Length >= WordLength) return false`. Devuelve `false` y Form1 simplemente ignora esa pulsación. No hay ningún error.

**P: ¿Qué pasa si pulsa ENTER con menos de 5 letras?**
> En `HandleVirtualKeyPress`, antes de llamar a `game.SubmitGuess()`, compruebo `if (game.CurrentGuess.Length < 5)` y en ese caso no hago nada (o muestro un feedback visual). El método `SubmitGuess()` también devolvería `null` si el intento no tiene 5 letras.

---

### Sobre las estadísticas

**P: ¿Cómo guardas las estadísticas?**
> Las estadísticas se guardan en un objeto `GameStats` que se crea en Form1 y se pasa al constructor de `WordleGame`. Como es el mismo objeto compartido por referencia, cuando `WordleGame` lo modifica, Form1 ve los cambios. Se actualiza en `UpdateStats(bool won)` dentro de `WordleGame`.

**P: ¿Las estadísticas persisten al cerrar la aplicación?**
> No, en la versión actual las estadísticas son solo en memoria (in-memory). Al cerrar el programa se pierden. Para persistirlas habría que usar `System.IO.File` para guardar en JSON o XML, o usar `Application.UserAppDataPath` con `StreamWriter`. Es una mejora que se podría añadir.

**P: ¿Cómo calculas el porcentaje de victorias?**
> `(stats.Wins / (float)stats.GamesPlayed) * 100`, redondeado a entero. El cast a `float` es importante porque si ambos son `int`, la división sería entera y daría 0 si `Wins < GamesPlayed`.

---

### Sobre la cuadrícula

**P: ¿Por qué usas `Label[,]` en lugar de `Label[][]`?**
> Porque la cuadrícula es rectangular (siempre 6×5). `Label[,]` es un array bidimensional de C#, que tiene mejor rendimiento y semántica más clara que un array irregular `Label[][]`. Accedo a cualquier celda con `gridLabels[fila, columna]`, que es más legible.

**P: ¿Cómo inicializas la matriz `gridLabels`?**
> En el constructor de Form1, después de `InitializeComponent()`, recorro los controles del formulario buscando los que se llaman `lblGrid_X_Y` y los asigno a su posición:
> ```csharp
> gridLabels = new Label[6, 5];
> for (int r = 0; r < 6; r++)
>   for (int c = 0; c < 5; c++)
>     gridLabels[r, c] = (Label)Controls.Find($"lblGrid_{r}_{c}", true)[0];
> ```

**P: ¿Cómo actualizas la fila que el usuario está escribiendo?**
> El método `UpdateCurrentRowDisplay()` coge `game.CurrentGuess` (el string con las letras escritas hasta ahora) y para cada columna de la fila actual pone el carácter correspondiente, o vacío si todavía no hay letra en esa posición. Así el usuario ve en tiempo real lo que está escribiendo.

---

### Sobre aspectos técnicos de C#

**P: ¿Qué es `DialogResult` y para qué sirve?**
> Es un enum de Windows Forms que indica cómo se cerró un formulario modal. Los valores más comunes son `OK`, `Cancel`, `Yes`, `No`. Cuando usas `ShowDialog()`, el valor de retorno es el `DialogResult` con el que se cerró el formulario. Yo lo uso para saber si el usuario realmente introdujo una palabra (OK) o cerró la ventana sin hacerlo (Cancel).

**P: ¿Qué es `KeyPreview`?**
> Es una propiedad de `Form` que, cuando es `true`, hace que los eventos de teclado se disparen en el formulario antes de enviarse al control que tiene el foco. Sin esto, si un botón tiene el foco, ENTER activaría ese botón en vez de enviar el intento.

**P: ¿Qué es `FormBorderStyle = FixedSingle`?**
> Es la propiedad que controla el borde de la ventana. `FixedSingle` muestra un borde fino y **no permite redimensionar** la ventana. Es la opción adecuada para juegos con layout fijo, porque si el usuario pudiera estirar la ventana, los controles no se reescalarían automáticamente.

**P: ¿Qué hace `Clipboard.SetText()`?**
> Copia un string al portapapeles del sistema operativo, como si el usuario hubiera hecho Ctrl+C. En el botón de compartir, genero una cadena de emojis (🟩🟧⬛) y la copio para que el usuario pueda pegarla donde quiera.

---

*Documento generado para apoyo de estudio del proyecto WordleApp.*
