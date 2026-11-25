# Reto 3 · Predicción de tráfico y congestión con IA
**Hackathon ITS Tepeaca 2026 · Autotraffic · 25–26 de noviembre de 2026**

Paquete de datos y herramientas para el Reto 3. Complementa la *Ficha del Reto 3 (v1.1)*.

## Objetivo
Construir un modelo que, con datos históricos de flujo vehicular, **estime el volumen de los siguientes 60 minutos** y supere a la línea base de *promedio histórico*. Se espera una prueba de concepto que funcione y se pueda demostrar, no un producto de producción.

## Contenido del paquete
```
Reto3_Prediccion_Trafico/
├── README.md                    <- este archivo
├── datos/
│   └── trafico_desarrollo.csv   <- 24 días (25-oct a 17-nov) · 6,912 filas · para entrenar y validar
└── starter/
    ├── starter.py               <- carga de datos, línea base, formato de entrega y gráfica de ejemplo
    ├── evaluar.py               <- el mismo cálculo de MAE que usará el jurado
    └── requirements.txt
```

## Descripción de los datos
El CSV de desarrollo y el reservado (que se libera al final, ver más abajo) tienen las mismas columnas. Hay 3 vialidades, 96 intervalos de 15 minutos por día y una fila por vialidad e intervalo (sin huecos).

| Columna | Tipo | Descripción |
|---|---|---|
| `fecha` | texto `AAAA-MM-DD` | Día calendario, hora local. |
| `hora` | texto `HH:MM` | **Inicio** del intervalo de 15 minutos (`08:15` = de 08:15 a 08:29). |
| `vialidad` | texto | Identificador anónimo de la vialidad: `V1`, `V2`, `V3`. Son puntos de medición distintos; no se revela su ubicación. |
| `volumen` | entero ≥ 0 | Vehículos distintos detectados en la vialidad durante el intervalo. **Es la variable a predecir.** Cero significa que no hubo lecturas. |
| `velocidad_prom` | decimal (km/h) | Velocidad promedio de las lecturas válidas del intervalo (se descartan valores fuera de 5–200 km/h). Vacío si no hubo lecturas válidas. |

Datos anonimizados: no contienen placas, imágenes ni coordenadas. Se obtuvieron agregando lecturas reales de una red de lectores de placas.

**Calidad de los datos.** Son datos reales y pueden incluir intervalos con fallas de captura (ceros o caídas bruscas), cambios de nivel o días atípicos (fin de semana, festivo). Detectarlos y decidir cómo tratarlos forma parte del reto: **documenten qué encontraron y qué decidieron**.

## La tarea y cómo se evalúa
- **Origen *t*:** cada intervalo de 15 min del conjunto reservado (6 días, 18 a 23 de noviembre). Al predecir el origen *t* se conocen los datos reales hasta *t* (inclusive), como en operación real, y ninguno posterior.
- **Objetivo:** volumen **total** de los 4 intervalos siguientes (*t*+15, +30, +45 y +60 min), por vialidad.
- **Métrica:** **MAE** (error absoluto medio, en vehículos) sobre los 1,716 orígenes del reservado, comparado contra la línea base.
- **Línea base (promedio histórico):** media de `volumen` por vialidad × día de la semana × hora, calculada solo con el conjunto de desarrollo y sumada sobre los 4 intervalos siguientes.

Pregunta del jurado: *¿supera el promedio histórico? ¿Cómo separaron entrenamiento y prueba?*

### Conjunto reservado: cómo funciona
El **conjunto reservado** (6 días: 18 a 23 de noviembre, 1,728 filas) **no se entrega al inicio**. Mide qué tan bien generaliza su modelo con datos que nunca vio.

1. **Durante el desarrollo**, trabajen solo con `trafico_desarrollo.csv`. Para validar, partan el histórico cronológicamente (por ejemplo, entrenar con los primeros 18 días y validar con los últimos 6). `starter.py` ya lo hace así.
2. **Congelen su modelo antes de la liberación.** Suban a la carpeta de entrega un ZIP con su código y modelo entrenado, para dejar registro de la hora. Esa carpeta y esa hora las confirma el comité.
3. **La organización libera `trafico_reservado.csv` el jueves 26 de noviembre a las [07:00, por confirmar].** Cópienlo en `datos/`.
4. **Solo ejecuten el modelo congelado** sobre el reservado, generen `predicciones.csv` y midan con `evaluar.py`. **No reentrenen ni ajusten hiperparámetros con el reservado.** Al predecir el origen *t* pueden usar los datos reales hasta *t*, nunca posteriores.
5. El jurado recalcula el MAE con su propia copia y puede revisar que el código entregado coincida con el modelo congelado.

### Formato de entrega de las predicciones
Un archivo `predicciones.csv` con una fila por origen del reservado (1,716 filas):
```
fecha,hora,vialidad,pred_60min
2026-11-18,00:00,V1,214.5
```
`fecha` y `hora` identifican el origen *t*. Se evalúan los orígenes que tienen 4 intervalos siguientes dentro del reservado (los 4 últimos de cada vialidad no se evalúan). Para medir su resultado:
```bash
cd starter
python evaluar.py predicciones.csv
```

## Cómo empezar
```bash
cd starter
pip install -r requirements.txt
python starter.py        # modo validación: línea base sobre los últimos 6 días de desarrollo
```
Genera `predicciones_baseline.csv` y `grafica_ejemplo.html`. Cuando reciban el reservado y lo copien en `datos/`, el mismo script pasa solo al modo final.
`starter.py` **no es una solución**: solo carga los datos, calcula la línea base y muestra el formato. Sustituyan el bloque `TU MODELO`.

## Alcance
**Obligatorio:** procesar el histórico de al menos una vialidad · identificar horarios críticos y patrones por día y hora · predecir los siguientes 60 min · comparar contra el reservado y el promedio histórico (MAE) · mostrar resultados en gráficas.
**Fuera de alcance:** mapas reales, clima, semáforos y fuentes en vivo. El volumen indica demanda; confirmar congestión requiere velocidad u ocupación. `velocidad_prom` se incluye por si quieren proponer un indicador de congestión.

## Entregables (jueves 26, 08:00–10:00)
1. Prototipo ejecutable (aplicación web, notebook o escritorio).
2. Código fuente (ZIP o repositorio) con README: instalación, ejecución, dependencias y **herramientas de IA utilizadas**.
3. Evidencia de pruebas: resultados (validación y evaluación final), métrica y limitaciones conocidas, incluido el archivo `predicciones.csv` sobre el reservado.
4. Presentación de máximo cinco diapositivas.

Demostración: 5 minutos + 3 de preguntas.

## Condiciones de uso
- Uso exclusivo durante el hackathon. **No publiquen ni compartan los datos ni los resultados fuera del evento.**
- Se permiten asistentes de IA, librerías y modelos preentrenados, declarados en el README del equipo.
- El código se desarrolla durante el evento; los componentes previos o de terceros deben identificarse.
