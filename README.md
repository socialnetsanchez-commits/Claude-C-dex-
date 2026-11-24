# Indicadores TradingView · Pine Script v6

| Archivo | Para qué |
|---|---|
| [`CH_Zone_Signals.pine`](CH_Zone_Signals.pine) | **Principal: CH Zone Signals by Sanchez.** Zonas BUY/SELL de temporalidades superiores y order blocks swing, filtrados según su reacción medida en datos reales. |
| [`CEP_Confirmation_Engine_Pro.pine`](CEP_Confirmation_Engine_Pro.pine) | Capa de 13 filtros para validar las señales de cualquier otro indicador. |

> No es asesoramiento financiero. Ningún indicador garantiza resultados. Mide siempre fuera de muestra antes de arriesgar dinero.

## CH Zone Signals by Sanchez

Muestra dos cosas:

| Herramienta | Qué ves |
|---|---|
| **Zonas BUY / SELL** | Cajas **M15 SELL**, **M30 BUY**, **H1 SELL**... Solo se aceptan las que van a favor de la EMA 50 y de la estructura mayor de su temporalidad. Calculadas con velas cerradas, no repintan. |
| **Order blocks swing** | Solo los que van a favor de la EMA 200 y de la estructura mayor (4 veces el swing) y tienen volumen por encima de la media en su origen. El texto **OB ★★★** indica cuántos criterios de calidad cumple. Los **OB A+** (borde grueso) están además dentro de una zona BUY/SELL de temporalidad superior. Cada OB lleva una línea discontinua en su **50 %** como referencia de entrada. |

Todo es personalizable: temporalidades de las zonas, filtros, invalidación, colores, transparencias, etiquetas, línea del 50 % y tabla de reacción.

**Diseño:**

- **Zonas:** relleno suave, borde de entrada marcado (por donde llega el precio), borde lejano punteado y una etiqueta tipo píldora a la derecha (`▼ H1 SELL`, `▲ M30 BUY`).
- **OB:** relleno suave del color de su dirección y etiqueta `OB ★★★` arriba a la derecha. Los **A+** llevan borde y texto dorados.
- **Tabla:** tipo tarjeta con el título *CH Zone Signals · by Sanchez*. El acierto aparece en verde (≥ 60 %), ámbar (45–60 %) o rojo (< 45 %).
- **Fondo del gráfico (oscuro o claro):** adapta los textos y la tabla para que se lean bien.

Estos cambios son solo visuales: la detección y los filtros son idénticos a la versión anterior.

### Cómo se eligieron los filtros (datos reales)

Se replicó la lógica del indicador en Python y se midió la **reacción**: tras el primer toque, el precio se aleja al menos 1,5 ATR antes de que una vela cierre al otro lado de la zona. Si las dos cosas pasan en la misma vela, cuenta como fallo.

- **Datos:** EUR/USD 1 h, S&P 500 diario y Google diario.
- **Separación:** el 60 % inicial (dentro de muestra) se usó para elegir y el 40 % final (fuera de muestra) para comprobar.

| | Dentro de muestra | Fuera de muestra |
|---|---|---|
| OB con los filtros anteriores (desplazamiento + FVG) | 50 % (243 toques) | 43 % (158) |
| **OB con EMA 200 + estructura mayor + volumen** | **66 %** (167) | **66 %** (82) |
| Zonas sin filtro | 64 % (133) | 77 % (93) |
| **Zonas con EMA 50 + estructura mayor de su TF** | **68,5 %** (73) | **84 %** (50) |

**Lo que no funcionó** y por eso se quitó o es opcional:

- El FVG y el desplazamiento rebajaban la reacción.
- El descuento/premium no aportaba.
- El barrido de liquidez era inestable.
- "Solo BOS" queda como opción: mejora fuera de muestra pero no dentro.

**Advertencia honesta:** no son datos de NQ en 5 minutos, y las muestras son de decenas o cientos de casos. Ningún porcentaje está garantizado en tu mercado. Por eso el indicador trae una **tabla que mide la reacción en tu propio gráfico**. Con menos de 30 toques aparece "poca muestra".

### Segunda ronda de pruebas (qué se añadió y qué se descartó)

| Idea | Dentro de muestra | Fuera de muestra | Decisión |
|---|---|---|---|
| OB dentro de una zona de TF superior | 84 % (19) frente a 63 % (139) | 100 % (2) frente a 64,5 % (76) | **Añadido** como OB A+, con fila propia en la tabla |
| OB tocado ≤ 50 velas frente a más antiguo | 61 % frente a 72,5 % | 66 % frente a 64 % | Descartado: el efecto no es estable |
| Toque en horario Londres/NY frente a fuera | 80 % frente a 54 % | 59 % frente a 91 % | Descartado: el efecto se invierte |
| Mostrar hasta 3 zonas por TF en vez de la última | 68 % frente a 69 % | 75 % frente a 82 % | Descartado: empeora |

Los OB A+ tienen muy pocos casos fuera de muestra (2). Por eso no se usan como filtro, solo se resaltan, y la tabla los mide aparte en tu gráfico.

### Errores corregidos en esta versión

- **El toque inmediato inflaba la estadística:** un OB recién creado podía contar como "tocado" sin que el precio hubiera salido antes de la zona. Ahora el toque solo cuenta tras salir por completo, y lo mismo vale para las zonas.
- **Las zonas no se medían:** ahora tienen su propia medición.
- **La estructura mayor no seguía al swing:** ahora es 4 veces la longitud del swing y se ajusta si cambias esa longitud.

### Tercera ronda: errores y pruebas de las zonas

**Error corregido:** una zona de temporalidad superior solo desaparecía cuando cerraba la vela de esa temporalidad. Mientras tanto, en tu gráfico el precio podía haber cerrado ya al otro lado y la zona seguía dibujada como válida. Ahora desaparece en cuanto tu gráfico la invalida (por cierre o por mecha, según el ajuste *Invalidación*).

**Ideas probadas para subir la reacción de las zonas (todas descartadas):**

| Idea | Resultado | Decisión |
|---|---|---|
| EMA 200 del gráfico a favor | Sin efecto: las zonas filtradas ya la cumplen | Descartada |
| Barrido de liquidez | 60 % frente a 69 % dentro; 83 % frente a 74 % fuera | Descartada: el efecto se invierte entre tramos |
| Solape con zona de otra temporalidad | Muy pocos casos (5 y 9) | Descartada |
| Vela de ruptura ≥ 1 ATR | +3 a 4 puntos con poca muestra | Descartada: probable ruido |
| Solo zonas grandes (> 1 ATR) | Sube el porcentaje, pero en múltiplos del riesgo no mejora | Descartada: sería engañoso (exige un stop mayor) |

Los filtros actuales (EMA 50 + estructura mayor de su temporalidad) siguen siendo los mejores encontrados con estos datos.

### Comprobaciones hechas

- Pasa el analizador sintáctico de Pine Script.
- Revisados: indentación, declaraciones antes de uso, número de argumentos y tamaño de las tuplas.
- No hay lecturas del historial dentro de llamadas condicionales.
- No se ha podido compilar dentro de TradingView desde este entorno. Si el editor muestra un error, pégalo y se corrige.

### Alertas

Nuevo OB alcista/bajista · Nuevo OB A+ · Precio vuelve a un OB alcista/bajista · Precio vuelve a un OB A+ · Precio vuelve a una zona BUY/SELL.

---

## Confirmation Engine Pro (CEP)

Capa de confirmación para TradingView. Toma una señal de compra o venta y solo la deja pasar si el contexto, la fuerza, la calidad de la vela y el riesgo están a favor. Puede funcionar sola o **encima de cualquier otro indicador** (modo *Externa*).

### Objetivo

La mayoría de los indicadores de señales fallan por las mismas razones: dan señales contra la tendencia, en mercados laterales, en velas débiles, después de que el precio ya se ha movido o con un stop que no compensa. CEP filtra cada una de esas causas y además **mide** el resultado de cada señal en R (múltiplos del riesgo).

### Lógica

| Paso | Qué hace |
|---|---|
| 1. Señal base | Giro de Supertrend, cruce de EMAs o la señal de **tu** indicador (modo *Externa*). |
| 2. Memoria | Los filtros tienen hasta *N* barras (3 por defecto) para confirmar la señal base. Cada señal base se usa una sola vez. |
| 3. Puntuación | 13 filtros: precio vs EMA 200, pendiente de la EMA, tendencia en TF superior, ADX, DI+/DI−, RSI sin extremos, MACD, Choppiness, volumen, régimen de ATR, vela con cuerpo, anti-persecución y vela no gigante. Hacen falta 10 de 13 por defecto. |
| 4. Tendencia obligatoria | Aunque la puntuación sea alta, nunca opera contra la EMA, su pendiente o el TF superior. |
| 5. Gatillo | La entrada exige un **cierre** por encima del máximo (compra) o por debajo del mínimo (venta) de la vela de setup en ≤ 3 barras. Se cancela si cierra al otro lado de esa vela. |
| 6. Riesgo | Stop por ATR o swing. Si el stop queda a menos de 0,75 ATR se ensancha hasta ese mínimo; si supera 3 ATR, la señal se descarta. TP1, TP2 y TP3 en 1R, 2R y 3R con parciales del 33/33/34 %. Breakeven tras TP1 y stop a TP1 tras TP2. |
| 7. Control | Señal solo al cierre de vela, espera tras cerrar una operación, máximo diario, filtro de sesión y cierre al final de sesión. |

**Entrada larga:** señal base alcista vigente, tendencia obligatoria a favor, puntuación ≥ mínimo, cierre por encima del máximo de la vela de setup y riesgo entre el mínimo y el máximo en ATR.
**Entrada corta:** lo mismo en espejo.
**Invalidación del setup:** cierre por debajo del mínimo de la vela de setup (o por encima del máximo en cortos), fin de la ventana o pérdida de la tendencia.
**Salida:** stop, TP3, señal base contraria (opcional) o fin de sesión (opcional).

Las etiquetas muestran el grado: **A+** (todos los filtros), **A** (≥ 85 %), **B** (≥ 70 %) y **C**. Pasa el ratón sobre la etiqueta para ver entrada, stop y objetivos.

### Usarlo con tu indicador actual

1. Añade tu indicador y CEP al mismo gráfico.
2. En CEP, ve a **Señal base → Generador de señal → Externa (otro indicador)**.
3. Elige el formato según lo que dibuje tu indicador:
   - **Dos series:** una salida vale > 0 en compra y otra en venta. Es lo típico de `plotshape`; si no aparece en la lista, añade a tu indicador `plot(buy ? 1 : 0, "Buy", display = display.data_window)` y lo mismo para venta.
   - **Una serie:** una salida que vale +1 en compra y −1 en venta.
   - **Línea:** una línea (por ejemplo, una media o un trailing stop). Compra cuando el precio la cruza hacia arriba y vende cuando la cruza hacia abajo.
4. En **Externa · compra** (y **venta**), selecciona la salida correspondiente de tu indicador.

### Configuración recomendada inicial

| Input | NQ / YM (5 m) | Oro GC / XAUUSD (15 m) | Para qué sirve |
|---|---:|---:|---|
| Generador de señal | Supertrend 3 / 10 | Supertrend 3 / 10 | Señal base |
| TF superior | 60 | 240 | Tendencia de fondo |
| Confirmaciones mínimas | 10 | 10 | Exigencia del filtro |
| ADX ≥ | 20 | 18 | Evitar mercado sin fuerza |
| Operar solo en sesión | 0930-1600 NY | 0300-1200 NY | Londres + Nueva York en oro |
| Cerrar al terminar la sesión | Sí | Opcional | Evitar mantener posiciones de noche |
| Stop | ATR × 1,5 | ATR × 1,5–2 | El oro tiene mechas más largas |
| Máx. operaciones por día | 2–3 | 2–3 | Evitar sobreoperar |
| Coste ida+vuelta (ticks) | 2 | GC: 2 · XAUUSD: 25–35 | Comisión, slippage y spread |
| Estadísticas desde | Fecha de corte | Fecha de corte | Separar datos de ajuste y de prueba |

El **tamaño sugerido** del panel usa `syminfo.pointvalue` (NQ 20 $/punto, YM 5 $, GC 100 $). Si sale menos de 1 contrato, usa micros (MNQ, MYM, MGC = 1/10) o no operes.

### Alertas

1. Haz clic en **Crear alerta** y elige *Confirmation Engine Pro* en Condición.
2. Opción A, con niveles en el mensaje (recomendada): **Cualquier llamada a la función alert()**. Envía entrada, SL, TP1, TP2, TP3 y grado. Sirve también para webhook.
3. Opción B: elige una condición concreta (*Compra confirmada*, *Venta confirmada*, *TP1/TP2/TP3*, *Stop loss*, *Stop protegido*, *Salida anticipada* o *Setup pendiente*).
4. Para las entradas, usa la frecuencia **Una vez por cierre de barra**. Para TP y SL, **Una vez por barra** si quieres el aviso en cuanto se toca el nivel.

### Validación sin engañarse

1. Fija **Estadísticas desde** en una fecha de corte. Ajusta parámetros solo con datos anteriores y juzga solo con los posteriores (fuera de muestra).
2. Exige al menos 200 operaciones en total y más de 30 fuera de muestra. El panel avisa con "muestra pequeña" si hay menos de 30.
3. Mira el profit factor (> 1,3), la expectativa (> 0,15R), el drawdown máximo en R y la racha perdedora máxima, no solo el porcentaje de acierto.
4. Comprueba la sensibilidad: cambia ±20 % el ADX, el ATR del stop y las confirmaciones mínimas. Si el resultado se hunde, el ajuste no es robusto.
5. Compáralo con la señal base sin filtros (pon *Confirmaciones mínimas* = 0 y desactiva *Tendencia obligatoria* y *Gatillo*). Los filtros deben mejorar la expectativa, no solo reducir el número de operaciones.
6. Las estadísticas usan un supuesto conservador: si el stop y un objetivo caen en la misma vela, cuenta el stop. Ya descuentan los costes configurados.

### Riesgos y limitaciones

- **Menos señales:** con 10 de 13 filtros y gatillo habrá días sin operaciones. Es intencionado.
- **Filtros correlacionados:** DI, MACD y RSI suelen coincidir. Una puntuación alta no equivale a 13 pruebas independientes.
- **Noticias (CPI, FOMC, NFP):** el filtro de ATR evita parte del ruido, pero no conoce el calendario. No operes en los minutos alrededor de noticias de alto impacto.
- **Mercado lateral:** Choppiness y ADX lo reducen, pero no lo eliminan. Una tendencia que se agota sigue pareciendo tendencia unas barras.
- **Modelo de ejecución:** entra al cierre de la vela de señal. En vivo habrá slippage. Ajusta los ticks de coste a tu bróker.
- **Es un indicador, no una estrategia:** las estadísticas son una aproximación. Para un backtest completo con comisiones y órdenes reales hace falta convertirlo en `strategy()`.
- **Repintado:** las decisiones se toman al cierre y el TF superior usa solo velas cerradas. Con *Señales solo al cierre de vela* desactivado, la señal puede aparecer y desaparecer dentro de la vela en curso.
