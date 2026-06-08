# Documentación: Simulación Matemática de Colector Solar en Espiral

## 1. Descripción General del Proyecto
Esta simulación está diseñada para predecir el rendimiento térmico y la producción de agua caliente de un colector solar de diseño innovador en espiral, ubicado en Morelia, México. A diferencia de los calentadores solares convencionales (tubos al vacío o placas planas), este diseño utiliza una lente concentradora y un serpentín espiral compacto, lo que optimiza la concentración solar sobre un área pequeña.

El simulador se nutre de datos de radiación solar (calculados geométricamente con la librería `pvlib`) y de datos meteorológicos reales provistos por la NASA (como temperatura ambiente y velocidad del viento).

## 2. Parámetros del Sistema

### 2.1. Condiciones Geográficas
- **Latitud**: 19.7008° N (Morelia)
- **Longitud**: -101.1844° O
- **Altitud**: 1920 metros (1.92 km, clave para el cálculo de atenuación atmosférica usando el modelo de Hottel).

### 2.2. Diseño Geométrico y Óptico
- **Lente Concentradora**: Su diámetro de 49 cm genera un área de apertura de captación solar directa de ~0.188 m². 
- **Transmitancias y Absortancias**:
  - Transmitancia de la Lente ($\tau_{lente}$): 85%.
  - Transmitancia de la Caja protectora de Vidrio ($\tau_{vidrio}$): 90%.
  - Absortancia ($\alpha$) y Emisividad ($\epsilon$) del Cobre: Ambas al 95%.
- **Dimensiones del Serpentín**: Un tubo de 15 metros enrollado de forma compacta (diámetro exterior de media pulgada y diámetro interno de tres octavos de pulgada).

### 2.3. Dinámica del Fluido (Vaciado Parcial)
La simulación aprovecha una estrategia de **Vaciado Parcial**. Al calentar únicamente el 50% de la capacidad de agua de la espiral por ciclo, se logran ciclos de hervido más rápidos (aproximadamente de 2 a 4 horas), reduciendo enormemente la inercia térmica total del sistema y propiciando de múltiples lotes producidos por día.
- **Rango de Calentamiento**: 20 °C a 94 °C.

## 3. Modelo Matemático

El cuaderno se basa en las siguientes ecuaciones fundamentales en equilibrio dinámico para cada hora:

### Paso 1. Irradiancia y Ángulos
A través de `pvlib`, se obtiene el ángulo zenital $\theta_z$.
La radiación extraterrestre normal $G_{on}$ se calcula mediante la fórmula de Spencer:
$$ G_{on} = G_{sc} \times \left(1 + 0.033 \cos\left(\frac{360n}{365}\right)\right) $$
Luego, a través del modelo de Hottel para la radiación directa:
$$ G_{b,n} = G_{on} \times \left(A_0 + A_1 \exp\left(\frac{-K}{\cos \theta_z}\right)\right) $$

### Paso 2. Resolución Iterativa Newton-Raphson
Al no conocer la temperatura exacta de la placa receptora de la espiral ($T_{p}$), el cuaderno resuelve de forma iterativa el balance de energía en la placa:
$$ Q_{opt} - Q_{loss} - Q_{agua} = 0 $$
$$ Q_{opt} = G_{b,n} \times A_a \times \tau_{lente} \times \tau_{vidrio} \times \alpha_{cobre} $$
La pérdida de calor se da por convección y radiación. La aproximación emplea un ciclo de corrección de la forma $T_p^{nueva} = T_p - \frac{f(T_p)}{f'(T_p)}$.

### Paso 3. Eficiencia y Producción
Una vez obtenido el Calor Útil neto provisto al fluido ($Q_u$), la eficiencia se calcula comparando el calor absorbido versus la irradiación solar directa en el concentrador.
$$ \eta = \frac{Q_u}{G_{b,n} A_a} \times 100 $$

Finalmente, el tiempo que tarda un lote en alcanzar la temperatura objetivo es:
$$ \text{Tiempo (min)} = \frac{m_{lote} \times C_p \times \Delta T}{Q_u \times 60} $$

