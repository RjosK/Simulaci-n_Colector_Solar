# 🚀 GUÍA RÁPIDA: Cómo Leer los Resultados

---

## 📊 Las 3 Secciones Principales del Código

### ✅ SECCIÓN 1: SETUP (Celdas 1-3)
**¿Qué hace?** Define todas las constantes del sistema

```python
# 📍 Ubicación
phi = 19.7008              # Latitud
longitud = -101.1844       # Longitud
altitud_m = 1920           # Metros sobre el mar

# 💧 Agua
Ti_c = 20.0                # Temperatura entrada
Tf_c = 94.0                # Temperatura salida
Cp_agua = 4186             # Energía por grado

# ☀️ Lente
Aa = 0.188 m²              # Área de captación
tau_lente = 0.85           # % luz que pasa
```

**Output esperado:** Imprime resumen de geometría

---

### ✅ SECCIÓN 2: DATOS (Celdas 4-8)
**¿Qué hace?** Obtiene datos del sol y clima

```python
# De PVLib (posición solar cada hora)
fecha_hora, zenith, elevation, azimuth

# De NASA (clima cada hora)
temp_ambiente, v_viento

# Resultado: df_resultado (tabla unida)
```

**Verificación:** Los datos se cargan sin errores

---

### ✅ SECCIÓN 3: CÁLCULOS (Celdas 9-11)
**¿Qué hace?** Ejecuta todas las ecuaciones

```python
# 11 ESCALONES de cálculos en cadena
Q_opt → Q_loss → Q_util → Tiempo_hervir → Litros_hora → Litros_acum
```

**Output final:** Tabla con resultados

---

## 📈 Lectura Paso a Paso de Resultados

### Ejemplo Real de 3 Horas Consecutivas

```
HORA    G_DIRECTA   VIENTO   Q_útil   MIN_HERVIR   L/HORA   L_DÍA
────────────────────────────────────────────────────────────────
10:00   450 W/m²    2.1 m/s   25 W     548 min      0.49 L   0.49 L
11:00   620 W/m²    1.8 m/s   42 W     327 min      0.82 L   1.31 L
12:00   750 W/m²    2.0 m/s   58 W     237 min      1.14 L   2.45 L
```

### Qué significa cada columna:

1. **G_DIRECTA (450 → 750 W/m²)**
   - ¿Qué?: Intensidad de luz directa del sol
   - ¿Por qué sube?: El sol está más alto en el cielo (10 → 12h)
   - ¿Qué es normal?: 400-900 W/m² en invierno, hasta 1000+ en verano

2. **VIENTO (2.1 → 2.0 m/s)**
   - ¿Qué?: Velocidad del viento ajustada a la altura del colector
   - ¿Por qué baja?: Menos viento a mediodía (inversión térmica)
   - **Importancia**: ⚠️ CRÍTICA - El viento roba calor

3. **Q_útil (25 → 58 W)**
   - ¿Qué?: Calor disponible para calentar agua (la "joya")
   - ¿Por qué sube?: Más radiación y menos pérdidas
   - ¿Fórmula?: Q_útil = Q_absorbido - Q_pérdidas

4. **MIN_HERVIR (548 → 237 minutos)**
   - ¿Qué?: Tiempo para calentar UN LOTE de agua
   - ¿Por qué baja?: Más Q_útil = más rápido
   - ¿Qué significa?: A las 10h tarda 9 horas, a las 12h tarda 4 horas

5. **L/HORA (0.49 → 1.14 L/h)**
   - ¿Qué?: Agua caliente producida en esa hora
   - ¿Fórmula?: = (60 min / tiempo_hervir) × volumen_lote
   - ¿Potencial**: Máximo teórico ~3 L/h en día perfecto

6. **L_DÍA (0.49 → 2.45 L)**
   - ¿Qué?: Total acumulado desde las 10:00
   - ¿Fórmula?: Suma de litros de todas las horas anteriores
   - **Expectativa**: Un día completo (10-18h) = 8-15 L según clima

---

## ⚖️ Lógica de Decisión: ¿Está funcionando bien?

### Checklist de Resultados Normales

| Métrica | Rango Normal | Mi Valor | ✅/❌ |
|---------|--------------|----------|-------|
| G_Directa mediodía | 700-850 W/m² | ? | |
| Viento promedio | 1.5-3.5 m/s | ? | |
| Q_útil pico | 50-120 W | ? | |
| L/hora pico | 0.5-2.0 L/h | ? | |
| L/día (10-18h) | 5-20 L | ? | |
| Eficiencia pico | 15-35% | ? | |

**Interpretación:**
- ✅ Si está en rango: Diseño OK
- ❌ Si está bajo: Revisar viento o nubosidad
- ❌ Si está muy alto: Verificar que los parámetros sean realistas

---

## 🔍 Debugging: "¿Por qué mis resultados son bajos?"

### PROBLEMA 1: Radiación muy baja

**Síntoma:** G_Directa < 300 W/m² incluso al mediodía

**Causas posibles:**
- ☁️ Día nublado (NORMAL en invierno)
- 📅 Fecha muy baja en ángulosolar (octubre-febrero)
- 🗓️ Usar datos de diciembre es el peor mes para Morelia

**Solución:** Probar con datos de junio-septiembre

---

### PROBLEMA 2: Viento muy alto

**Síntoma:** v_viento_techo > 5 m/s constantemente

**Causas posibles:**
- 🌍 Ubicación en zona muy expuesta (montaña, llanura)
- 📊 Datos NASA incorrectos o de sitio ventoso

**Solución:** Verificar ubicación de estación meteorológica NASA

---

### PROBLEMA 3: Litros/hora cero o muy bajo

**Síntoma:** Incluso al mediodía, L/hora < 0.1 L/h

**Causas posibles:**
```
Si es porque Q_útil es negativo:
  → Pérdidas > radiación (clima muy frío o nublado)
  → NORMAL en invierno/noches
  
Si es porque Tiempo_hervir es muy alto (>1000 min):
  → Q_útil es muy pequeño (< 20 W)
  → Revisar parámetros de aislación
```

---

## 📊 Gráficos Mentales: Entender la Física

### El "Combate Térmico" Cada Hora

```
MEDIODÍA SOLEADO (12:00)
─────────────────────────────

☀️ GANA CALOR (Q_opt)
   └─→ Radiación solar golpea lente
       └─→ Lente concentra (85% transmite)
           └─→ Cobre absorbe (95%)
               └─→ 110 Watts hacia el agua ✅

❄️ PIERDE CALOR (Q_loss)  
   ├─→ Convección por viento (10 W) ← Caja reduce esto
   ├─→ Radiación al cielo (20 W)
   └─→ Conducción aislante (8 W)
       └─→ Total 38 Watts hacia afuera ❌

⚖️ RESULTADO NETO: 110 - 38 = 72 W ÚTILES ✅


TARDE NUBLADA (14:00)  
─────────────────────────────

☀️ GANA CALOR (Q_opt)
   └─→ Nubes bloquean 60% de radiación
       └─→ Solo 40 Watts hacia el agua ⚠️

❄️ PIERDE CALOR (Q_loss)
   └─→ Sigue siendo ~35 Watts hacia afuera ❌
   
⚖️ RESULTADO NETO: 40 - 35 = 5 W ÚTILES ❌ ¡CASI NADA!
```

---

## 🎯 Casos de Uso Real

### CASO 1: "Quiero saber si puedo usar esto para ducha"

**Necesitas:** ~40-50 L de agua a 60°C

**Cálculo:**
- Si produces 10 L/día a 94°C
- Mezclas con agua fría para bajar a 60°C
- Resultado: ~15-20 L útiles para ducha
- **Conclusión**: 2-3 duchas máximo por día

---

### CASO 2: "¿Cuándo es mejor usar el colector?"

**Óptimo:** 11:00 - 15:00 (máxima radiación)

**Mediocre:** 10:00-11:00 y 15:00-18:00 (energía marginal)

**Inutilizable:** 18:00+ (atardecer = radiación muy baja)

**Tabla de horarios:**

| Hora | G_Directa | Utilidad | Recomendación |
|------|-----------|----------|---|
| 10:00 | 450 W/m² | 50% | Iniciar si es urgente |
| 11:00 | 620 W/m² | 75% | ✅ Bueno |
| 12:00 | 750 W/m² | 90% | ✅✅ Excelente |
| 13:00 | 780 W/m² | 95% | ✅✅✅ PICO |
| 14:00 | 750 W/m² | 90% | ✅✅ Excelente |
| 15:00 | 620 W/m² | 75% | ✅ Bueno |
| 16:00 | 450 W/m² | 50% | Marginal |
| 17:00 | 200 W/m² | 10% | ❌ No recomendado |

---

## 💡 Fórmulas Rápidas Mentales

### "¿Cuánto calor estoy ganando/perdiendo?"

```
RADIACIÓN ABSORBIDA (Q_opt):
  Q_opt = G_directa × 0.188 m² × 0.85 × 0.95
  
  Si G=750 W/m²:
  Q_opt = 750 × 0.19 × 0.81 ≈ 115 W

PÉRDIDAS (Q_loss):
  Q_loss = 11.2 W/(m²K) × 0.188 m² × ΔT
  
  Si ΔT = 70°C (agua a 90°C, aire a 20°C):
  Q_loss = 11.2 × 0.188 × 70 ≈ 147 W
  
  ⚠️ ¡Perdemos MÁS de lo que ganamos!
  
CALOR ÚTIL:
  Q_u = 115 - 147 = -32 W ❌ NEGATIVO (no hay producción)
  
  Necesitamos MORE RADIACIÓN o HIGHER Q_opt!
```

---

## 🔧 Ajustes Simples que Puedes Hacer

### En el Código

```python
# Para mejorar radiación absorbida:
tau_lente = 0.90  # Cambiar de 0.85 (mejor vidrio)
alpha_cobre = 0.98  # Cambiar de 0.95 (mejor pintura negra)

# Para reducir pérdidas:
L_madera = 0.05  # Aumentar de 0.025 (aislante más grueso)
# O cambiar a mejor material: k_madera = 0.04

# Para caso sin viento:
# Usa expr_u_l_desnudo en lugar de expr_u_l_caja
# (pero NO lo recomendamos)
```

---

## ✅ Validación Final

Antes de confiar en los resultados, verifica:

- [ ] G_Directa es física (0-900 W/m² típico)
- [ ] Viento está en rango (0-10 m/s)
- [ ] Temperatura ambiente es realista (10-35°C)
- [ ] Litros/hora tiene sentido (0-3 L/h)
- [ ] Los picos son al mediodía (no al amanecer)
- [ ] En noches, Qu_Caja = negativo o nulo ✅
- [ ] Litros_acum es monotónico (siempre sube o igual)

---

**Última actualización: Diciembre 2025**
**Sistema: Morelia, Michoacán | Lente: 49cm | Agua: 94°C**
