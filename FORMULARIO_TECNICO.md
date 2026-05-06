# 📐 FORMULARIO TÉCNICO: Todas las Ecuaciones

---

## 📚 Índice de Ecuaciones

| # | Ecuación | Página |
|---|----------|--------|
| 1 | Posición Solar | p.2 |
| 2 | Radiación Extraterrestre | p.2 |
| 3 | Radiación Directa Normal | p.2 |
| 4 | Viento Ajustado | p.3 |
| 5 | Convección (McAdams) | p.3 |
| 6 | Temperatura del Cielo | p.3 |
| 7 | Radiación Térmica | p.4 |
| 8 | Coeficiente U_L | p.4 |
| 9 | Calor Absorbido | p.4 |
| 10 | Pérdidas Térmicas | p.4 |
| 11 | Calor Útil | p.5 |
| 12 | Tiempo Hervor | p.5 |
| 13 | Producción Horaria | p.5 |
| 14 | Eficiencia | p.5 |

---

## ☀️ GRUPO 1: POSICIÓN Y RADIACIÓN SOLAR

### Ec. 1: Ángulo Cenital (Solar Position)

$$Z = \arccos(\sin \phi \sin \delta + \cos \phi \cos \delta \cos H)$$

**Variables:**
- $\phi$ = 19.7° (latitud de Morelia)
- $\delta$ = declinación solar (-23.44° a +23.44°)
- $H$ = ángulo horario (±180° en 24h)

**Interpretación:**
- $Z = 0°$ → Sol en zenit (directamente arriba)
- $Z = 90°$ → Sol en horizonte (amanecer/atardecer)
- $Z > 90°$ → Noche (sol bajo el horizonte)

**Cálculo en Polars:**
```python
expr_es_de_dia = (pl.col("zenith") < 90).alias("es_de_dia")
```

---

### Ec. 2: Radiación Extraterrestre

$$G_{on} = G_{sc} \left[1 + 0.033 \cos\left(\frac{360n}{365}\right)\right]$$

**Donde:**
- $G_{sc}$ = 1367 W/m² (constante solar)
- $n$ = número de día (1-365)

**Valores Típicos:**
| Fecha | n | Factor | $G_{on}$ |
|-------|---|--------|---------|
| 1 Ene | 1 | 1.033 | 1411 W/m² |
| 1 Abr | 91 | 1.010 | 1382 W/m² |
| 1 Jul | 182 | 0.967 | 1322 W/m² |
| 1 Oct | 274 | 1.010 | 1381 W/m² |
| 1 Dic | 335 | 1.033 | 1411 W/m² |

**En Polars:**
```python
expr_gon = (Gsc * (1 + 0.033 * ((360 * pl.col("fecha_hora").dt.ordinal_day() / 365) 
            * math.pi / 180).cos())).alias("Gon")
```

---

### Ec. 3: Radiación Normal Directa (Hottel)

$$G_{b,n} = G_{on} \left[A_0 + A_1 \exp\left(-\frac{K}{\cos Z}\right)\right]$$

**Coeficientes para Morelia (h = 1.92 km):**

| Coeficiente | Fórmula | Valor |
|-------------|---------|-------|
| $A_0$ | $0.4237 - 0.00821(6-h)^2$ | 0.4064 |
| $A_1$ | $0.5055 + 0.00595(6.5-h)^2$ | 0.5159 |
| $K$ | $0.2711 + 0.01858(2.5-h)^2$ | 0.2944 |

**Gráfico mental:**

```
G_b,n (W/m²)
      ↑
 900  │     ╱╲
      │    ╱  ╲
 700  │   ╱    ╲
      │  ╱      ╲
 500  │ ╱        ╲ 
      │╱___________╲___
      └─────────────────→ Hora del día (6-18h)
      6  8  10  12  14  16  18
      
      Pico máximo: ~800-850 W/m² al mediodía
```

**En Polars:**
```python
expr_gb_n = (pl.col("Gon") * (Ao + A1 * (-K / pl.col("cos_theta_z")).exp())).alias("gb_n")
```

---

## 💨 GRUPO 2: VIENTO Y CONVECCIÓN

### Ec. 4: Velocidad del Viento Ajustada (Ley Logarítmica)

$$V(z) = V_{ref} \left(\frac{z}{z_{ref}}\right)^\alpha$$

**Parámetros:**
- $V_{ref}$ = velocidad a 10m (dato NASA)
- $z$ = 3.0 m (altura del colector)
- $z_{ref}$ = 10 m (altura referencia)
- $\alpha$ = 0.22 (coeficiente terreno suburbano)

**Ejemplo:**
| $V_{10m}$ (NASA) | Factor | $V_{3m}$ (Colector) |
|-----------------|--------|-------------------|
| 0 m/s | $(0.3)^{0.22}$ | 0.0 m/s |
| 2 m/s | $(0.3)^{0.22}$ | 1.4 m/s |
| 5 m/s | $(0.3)^{0.22}$ | 3.5 m/s |
| 10 m/s | $(0.3)^{0.22}$ | 7.1 m/s |

**En Polars:**
```python
expr_viento_real = (pl.col("v_viento") * (altura_techo / 10.0)**alpha_terreno).alias("v_viento_techo")
```

---

### Ec. 5: Coeficiente Convectivo (McAdams - Aire Forzado)

$$h_w = 5.7 + 3.8 V$$

**Donde:**
- $h_w$ en W/(m²·K)
- $V$ en m/s (velocidad ajustada del viento)

**Tabla de Valores:**

| V (m/s) | $h_w$ (W/m²·K) | Interpretación |
|---------|-----------------|---|
| 0 | 5.7 | Aire quieto (peor caso) |
| 1 | 9.5 | Brisa ligera |
| 2 | 13.3 | Viento suave |
| 3 | 17.1 | Viento moderado |
| 5 | 24.7 | Viento fuerte ⚠️ |
| 10 | 43.7 | Viento muy fuerte ⚠️⚠️ |

**Observación:** El viento es CRÍTICO. A 5 m/s, el coeficiente es 4x mayor que sin viento.

**En Polars:**
```python
expr_hw = (5.7 + 3.8 * pl.col("v_viento_techo")).alias("hw")
```

---

## 🌡️ GRUPO 3: RADIACIÓN TÉRMICA

### Ec. 6: Temperatura Efectiva del Cielo (Swinbank)

$$T_{sky} = 0.0552 \cdot T_a^{1.5}$$

**Donde:**
- $T_a$ = temperatura ambiente en Kelvin
- $T_{sky}$ = temperatura equivalente del cielo en Kelvin

**Tabla de Valores:**

| $T_a$ (°C) | $T_a$ (K) | $T_{sky}$ (K) | $T_{sky}$ (°C) |
|-----------|-----------|--------------|---|
| 0 | 273.15 | 140 | -133 ❄️ |
| 10 | 283.15 | 166 | -107 |
| 20 | 293.15 | 194 | -79 |
| 25 | 298.15 | 208 | -65 |
| 30 | 303.15 | 223 | -50 |

**Concepto:** El cielo "ve" como un objeto frío hacia el cual irradia el tubo caliente.

**En Polars:**
```python
expr_t_sky = (0.0552 * (pl.col("Ta_k")**1.5)).alias("T_sky")
```

---

### Ec. 7: Coeficiente Radiativo (Stefan-Boltzmann Linealizado)

$$h_{rad} = \varepsilon \sigma (T_p + T_{sky})(T_p^2 + T_{sky}^2)$$

**Parámetros Constantes:**
- $\varepsilon$ = 0.95 (emisividad del cobre)
- $\sigma$ = $5.67 \times 10^{-8}$ W/(m²·K⁴)
- $T_p$ = 330 K (temperatura promedio agua)

**Valores Típicos:**

| Escenario | $T_p$ (K) | $T_{sky}$ (K) | $h_{rad}$ (W/m²K) |
|-----------|-----------|--------------|------------------|
| Día claro 20°C | 330 | 194 | 4.2 |
| Día nublado 20°C | 330 | 190 | 4.1 |
| Noche 15°C | 330 | 165 | 3.8 |

**En Polars:**
```python
expr_h_rad = (epsilon_p * SIGMA * (Tp_k + pl.col("T_sky")) 
              * (Tp_k**2 + pl.col("T_sky")**2)).alias("h_rad")
```

---

## 🔥 GRUPO 4: BALANCE TÉRMICO

### Ec. 8: Coeficiente Global de Pérdida (U_L)

#### Escenario A: Tubo DESNUDO
$$U_L^{desnudo} = (h_w + h_{rad}) + \frac{k}{L}$$

#### Escenario B: Tubo con CAJA y VIDRIO
$$U_L^{caja} = 6.5 + \frac{k}{L}$$

**Desglose:**
- Conducción aislante: $\frac{k}{L} = \frac{0.12}{0.0254} = 4.72$ W/(m²·K)

**Comparativa:**

| Componente | Desnudo | Caja | Cambio |
|-----------|---------|------|--------|
| Convección $h_w$ | 13-25 | ~0 | -95% ✅ |
| Radiación $h_{rad}$ | 4.2 | 4.2 | 0% |
| Conducción $k/L$ | 4.7 | 4.7 | 0% |
| **TOTAL U_L** | **22-34** | **9** | **-73%** ✅ |

**En Polars:**
```python
expr_u_l_desnudo = (pl.col("hw") + pl.col("h_rad")) + (k_madera / L_madera)
expr_u_l_caja = pl.lit(6.5) + (k_madera / L_madera)
```

---

### Ec. 9: Calor Solar Absorbido (Q_opt)

$$Q_{opt} = G_{b,n} \times A_a \times \tau_{lente} \times \tau_{vidrio} \times \alpha$$

**Con CAJA (recomendado):**
$$Q_{opt}^{caja} = G_{b,n} \times 0.188 \times 0.85 \times 0.90 \times 0.95$$
$$= G_{b,n} \times 0.1367$$

**Ejemplo en mediodía:**
- $G_{b,n}$ = 800 W/m²
- $Q_{opt}$ = 800 × 0.1367 = **109 W**

**En Polars:**
```python
expr_q_opt_caja = (pl.col("gb_n") * Aa * tau_lente * tau_vidrio * alpha_cobre)
```

---

### Ec. 10: Pérdidas Térmicas (Q_loss)

$$Q_{loss} = U_L \times A_{perdida} \times (T_p - T_a)$$

**Con CAJA:**
$$Q_{loss}^{caja} = 9.0 \times 0.188 \times (57 - T_a)$$

**Ejemplo:**
- $T_a$ = 20°C
- $Q_{loss}^{caja}$ = 9.0 × 0.188 × 37 = **63 W**

**Vs. DESNUDO:**
- $U_L^{desnudo}$ = 28 W/(m²·K)
- $Q_{loss}^{desnudo}$ = 28 × 0.188 × 37 = **195 W** ⚠️⚠️ Mucho peor

**En Polars:**
```python
expr_q_loss_caja = (pl.col("U_L_Caja") * area_perdida_efectiva * (Tp_prom_c - pl.col("temp_ambiente")))
```

---

### Ec. 11: CALOR ÚTIL NETO ⭐

$$Q_u = Q_{opt} - Q_{loss}$$

**Ejemplo MEDIODÍA CLARO (con caja):**
| Variable | Valor | Impacto |
|----------|-------|--------|
| $Q_{opt}$ | +109 W | ✅ Gana |
| $Q_{loss}$ | -63 W | ❌ Pierde |
| **$Q_u$** | **46 W** | ✅ ÚTIL |

**Ejemplo TARDÍA (con caja):**
| Variable | Valor | Impacto |
|----------|-------|--------|
| $Q_{opt}$ | +40 W | ⚠️ Bajo |
| $Q_{loss}$ | -63 W | ❌ Mucho |
| **$Q_u$** | **-23 W** | ❌ NEGATIVO |

**En Polars:**
```python
expr_qu_caja = (pl.col("Q_opt_Caja") - pl.col("Q_loss_Caja")).alias("Qu_Caja_W")
```

---

## 📊 GRUPO 5: PRODUCCIÓN

### Ec. 12: Tiempo para Hervir un Lote

$$t_{herv} = \frac{E_{lote}}{Q_u}$$

**Donde:**
$$E_{lote} = m \times C_p \times \Delta T = 4.53 \text{ kg} \times 4186 \text{ J/(kg·K)} \times 74 \text{ K}$$
$$= 1,404,000 \text{ J} = 1.4 \text{ MJ}$$

**Tabla de Tiempos:**

| $Q_u$ (W) | $t_{herv}$ (seg) | $t_{herv}$ (min) | $t_{herv}$ (h) |
|-----------|-----------------|-----------------|---|
| 25 W | 56,160 | 936 | 15.6 ❌ |
| 50 W | 28,080 | 468 | 7.8 ⚠️ |
| 100 W | 14,040 | 234 | 3.9 ✅ |
| 200 W | 7,020 | 117 | 2.0 ✅✅ |

**En Polars:**
```python
expr_tiempo_herv = pl.when(pl.col("ventana_productiva") & (pl.col("Qu_Caja_W") > 0))
                     .then((pl.col("E_lote_joules") / pl.col("Qu_Caja_W")) / 60)
                     .otherwise(None).alias("Tiempo_Hervir_min")
```

---

### Ec. 13: Producción Horaria (Litros/Hora)

$$L_{hora} = \frac{60 \text{ min}}{t_{herv}} \times V_{lote}$$

**Donde:**
- $V_{lote}$ = 4.53 L (volumen de agua a calentar)
- $t_{herv}$ = tiempo para hervir en minutos

**Tabla de Producción:**

| $t_{herv}$ (min) | $L_{hora}$ | Interpretación |
|-----------------|-----------|---|
| 50 | 5.4 L/h | ✅✅ Excelente (raro) |
| 100 | 2.7 L/h | ✅ Muy bueno |
| 200 | 1.4 L/h | ✅ Bueno (típico mediodía) |
| 300 | 0.9 L/h | ⚠️ Aceptable |
| 600 | 0.45 L/h | ❌ Marginal |
| ∞ | 0 L/h | ❌ Noche (sin producción) |

**En Polars:**
```python
expr_litros_hora = pl.when(pl.col("Tiempo_Hervir_min") > 0)
                     .then((60 / pl.col("Tiempo_Hervir_min")) * v_litros_batch)
                     .otherwise(0.0).alias("Litros_Hora")
```

---

### Ec. 14: Eficiencia Térmica

$$\eta = \frac{Q_u}{G_{b,n} \times A_a} \times 100\%$$

**Donde:**
- $Q_u$ = calor útil para agua (W)
- $G_{b,n} \times A_a$ = potencia solar incidente (W)

**Tabla de Eficiencias:**

| Condición | $G_{b,n}$ | $Q_u$ | η | Estado |
|-----------|-----------|-------|---|--------|
| Claro mediodía | 800 W/m² | 46 W | 30.6% | ✅ Bueno |
| Nublado moderado | 400 W/m² | 15 W | 20.0% | ⚠️ OK |
| Nublado pesado | 200 W/m² | -10 W | Negativo | ❌ Malo |
| Amanecer/Atardecer | 100 W/m² | -40 W | Negativo | ❌ No usar |

**En Polars:**
```python
expr_eficiencia = pl.when(pl.col("gb_n") > 0)
                    .then((pl.col("Qu_Caja_W") / (pl.col("gb_n") * Aa)) * 100)
                    .otherwise(0.0).alias("Eficiencia_pct")
```

---

## 📈 RESUMEN DE VARIABLES PRINCIPALES

```
ENTRADA (datos horarios):
├─ fecha_hora: timestamp
├─ zenith: ángulo cenital (°)
├─ temp_ambiente: temperatura (°C)
└─ v_viento: velocidad viento (m/s)

CÁLCULOS INTERMEDIOS (11 escalones):
├─ Escala 1-3: Ambiente → es_de_dia, v_viento_techo, Ta_k
├─ Escala 4-5: Ángulos → cos_theta_z, T_sky, hw
├─ Escala 6-7: Radiación → Gb_n, h_rad, U_L
├─ Escala 8-9: Energía → Q_opt, Q_loss, Qu
└─ Escala 10-11: Producción → t_herv, L_hora, η

SALIDA (resultados finales):
├─ Qu_Caja_W: Calor útil (W)
├─ Tiempo_Hervir_min: Tiempo para lote (min)
├─ Litros_Hora: Producción (L/h)
├─ Eficiencia_pct: Rendimiento (%)
└─ Litros_Acum_Dia: Total día (L)
```

---

## 🎓 Notas Pedagógicas

### ¿Por qué tanto detalle?

Cada ecuación tiene un propósito físico:
1. **Posición solar** → Define la radiación incidente
2. **Viento** → Factor crítico de pérdidas (por eso la caja)
3. **Radiación térmica** → Pérdidas al cielo nocturno
4. **Balance energético** → Dinero (literalmente)

### ¿Qué está simplificado?

- No consideramos reflexión difusa del terreno (pequeño efecto)
- Asumimos tubo en equilibrio térmico (aproximación válida)
- Ignoramos estratificación dentro del agua (mezcla perfecta)
- No modelamos deformación térmica del tubo

### ¿Qué puedo cambiar?

Fácil (sin mucho efecto):
- Pintura del tubo (α = 0.90-0.98)
- Espesor aislante ($L$ = 0.02-0.10 m)

Moderado (efecto significativo):
- Tamaño lente (Aa = 0.1-0.3 m²)
- Temperatura objetivo (Ti, Tf)

Difícil (cambios de diseño):
- Usar colector parabólico
- Sistema de seguimiento solar
- Almacenamiento térmico

---

**Documento técnico | Proyecto Colector Solar | Morelia 2025**

