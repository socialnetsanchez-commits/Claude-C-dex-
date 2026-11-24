# Indicadores TradingView · Pine Script v6

| Archivo | Para qué |
|---|---|
| [`SMC_Confirm_Pro.pine`](SMC_Confirm_Pro.pine) | **Principal.** Solo cinco herramientas: perfil de liquidez, zonas SELL/BUY de temporalidades superiores, order blocks swing fuertes con su tasa de reacción medida, SMT y MSS. |
| [`CEP_Confirmation_Engine_Pro.pine`](CEP_Confirmation_Engine_Pro.pine) | Capa de 13 filtros para validar las señales de cualquier otro indicador. |

> No es asesoramiento financiero. Ningún indicador garantiza resultados. Mide siempre fuera de muestra antes de arriesgar dinero.

## SMC Confirm Pro

Muestra solo cinco herramientas, todas configurables desde los ajustes:

| # | Herramienta | Qué ves en el gráfico |
|---|---|---|
| 1 | **Perfil de liquidez** | Histograma de volumen por precio del día, la semana, el mes o las últimas N velas: gris para compras y rojo para ventas. Incluye la línea **POC** con su precio, el value area (VAH/VAL) y los POC de periodos anteriores, que siguen extendiéndose mientras ningún precio los toque (POC vírgenes). |
| 2 | **Zonas SELL / BUY** | Cajas **M15 SELL**, **M30 SELL**, **H1 BUY**... con línea media discontinua. Muestran la última zona sin mitigar de cada temporalidad, calculada solo con velas cerradas (no repinta). |
| 3 | **Order blocks swing fuertes** | Solo los OB que cumplen los filtros de calidad. El texto **OB ★★** indica cuántos criterios cumple, y una tabla mide cuántos reaccionaron de verdad. |
| 4 | **SMT** | Línea entre dos máximos (o mínimos) cuando el gráfico y el activo correlacionado no confirman el mismo movimiento. |
| 5 | **MSS** | Cambio de estructura con etiqueta rellena **MSS**: línea continua para el swing y discontinua para el interno. No se muestra ni BOS ni CHoCH. |

### Cómo se decide que un OB es "fuerte"

Un OB se dibuja solo si cumple todos los filtros activos:

- **Desplazamiento** (por defecto): en las 3 velas siguientes al OB, el precio se aleja al menos 1 ATR. Para ser más estricto, súbelo a 1,5.
- **FVG** (por defecto): esas mismas velas dejan un hueco de liquidez.
- **Altura máxima** (por defecto): el OB mide como mucho 3 ATR, para evitar zonas enormes.
- **Barrido de liquidez** (opcional): el OB rompe los mínimos (o máximos) de las 20 velas anteriores.
- **Volumen** (opcional): las velas del impulso tienen volumen por encima de la media.

Cada OB se extiende hasta que una vela cierra al otro lado (o lo atraviesa con la mecha, si eliges *Mecha*).

**Tabla de reacción:** cuenta los OB que, tras el primer toque, se alejaron al menos 1,5 ATR antes de ser mitigados. Si en la misma vela ocurren las dos cosas, cuenta como fallo. Es una medición en tu gráfico, no una promesa. Con menos de 30 casos aparece "muestra pequeña" y el porcentaje todavía no significa nada.

### Cómo se detecta el SMT

- En **modo automático** compara NQ↔ES, YM/RTY→ES, GC↔SI, XAUUSD↔XAGUSD, EURUSD↔GBPUSD y NAS100/US30→SPX500. Si tu activo no está en la lista, usa el símbolo manual.
- **SMT bajista:** en dos máximos seguidos, uno de los activos hace un máximo más alto y el otro no.
- **SMT alcista:** lo mismo con los mínimos.
- Para activos con correlación inversa, como EURUSD frente a DXY, activa *Correlación inversa*.
- El activo comparado necesita datos en tu cuenta de TradingView. Si no los tiene, simplemente no aparecen SMT.

### Valores iniciales

| Ajuste | NQ / ES 5 m | XAUUSD 15 m |
|---|---|---|
| Perfil · periodo | Día | Día |
| Zonas · temporalidades | M15, M30, H1 | M30, H1, H4 |
| OB · longitud de swing | 25 | 20 |
| SMT · comparación | Automático (ES) | Automático (XAGUSD) |
| MSS · swing / interno | 25 / 5 | 20 / 5 |

### Alertas

- **Confluencia bajista o alcista:** el precio toca una zona SELL o un OB bajista, aparece un SMT bajista y luego un MSS bajista, todo dentro de 20 velas (y al revés para compras). Es la alerta más selectiva.
- **Alertas sueltas:** MSS swing e interno, SMT, nuevo OB, precio toca un OB y precio entra en una zona SELL o BUY.

### Comprobaciones hechas

- El código pasa un analizador sintáctico de Pine Script.
- La indentación y el orden de las declaraciones están revisados.
- Las 80 funciones integradas que usa existen en Pine v6.
- No hay lecturas del historial dentro de llamadas condicionales.
- La lógica de estructura, OB y value area se simuló en Python con datos sintéticos.
- No se ha podido compilar en el propio TradingView desde este entorno. Si el editor muestra un error, pégalo y se corrige.

### Limitaciones

- Los MSS y SMT se confirman unas velas después del pivote (la longitud elegida). Así no repintan.
- En CFD como XAUUSD el volumen es de ticks: el perfil es una aproximación de dónde se negoció, no volumen real de contratos.
- Cada temporalidad muestra solo su última zona SELL y BUY sin mitigar.

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
