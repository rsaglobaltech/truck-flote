# Proyecto: Sistema de Control de Flota de Camiones en Tiempo Real con Optimización de Carga y Pricing Dinámico

## Versión 1.0 | Julio 2026

---

## 1. Justificación del Proyecto

### 1.1 Contexto del Mercado

El mercado europeo de logística por carretera representa uno de los sectores más dinámicos y complejos del continente. Según datos recientes, el mercado europeo de 3PL (Third-Party Logistics) alcanzó los **233.200 millones de dólares en 2024**, con una proyección de crecimiento sostenido impulsada por el comercio electrónico y la digitalización de cadenas de suministro. Sin embargo, este crecimiento va acompañado de una creciente complejidad operativa: múltiples países con regulaciones distintas, restricciones de circulación variables, costos de combustible fluctuantes y una presión constante por la eficiencia.

### 1.2 Problema que se Resuelve

Las empresas de transporte por carretera en Europa enfrentan hoy tres problemas fundamentales que operan de forma interconectada:

1. **Subutilización de capacidad**: Los camiones circulan con espacio vacío. Estudios del sector indican que una parte significativa de la capacidad de carga se desperdicia por falta de visibilidad en tiempo real y herramientas de optimización.
2. **Decisiones operativas reactivas**: Cuando aparece una carga nueva mientras un camión ya está en ruta, la decisión de aceptarla o no se basa en estimaciones manuales, experiencia del despachador y, frecuentemente, conjeturas sobre si "cabrá" físicamente.
3. **Pricing inconsistente**: El cálculo de tarifas para cargas ad-hoc o en ruta rara vez considera el costo real del desvío, la oportunidad perdida, la demanda del corredor o las regulaciones de tiempos de conducción.

### 1.3 Propuesta de Valor

Este proyecto propone un sistema integral que resuelve los tres problemas simultáneamente:

- **Tracking en tiempo real** de toda la flota a través de Europa, integrando datos de GPS, tacógrafo inteligente y sensores del vehículo.
- **Optimización dinámica de rutas** que permite incorporar nuevas recogidas y entregas mientras el camión viaja, respetando restricciones de vehículo pesado, tráfico y normativa europea de tiempos de conducción.
- **Optimización física de carga (3D Bin Packing)** que calcula no solo si una carga "cabe" en términos de peso y volumen, sino **dónde físicamente** debe colocarse dentro del remolque, respetando fragilidad, orientación, secuencia de descarga y estabilidad del vehículo.
- **Pricing dinámico** que calcula el precio justo de cada carga considerando kilómetros reales, desvíos, espacio ocupado, demanda del corredor, costos operativos y regulaciones.

### 1.4 Impacto Esperado

| Métrica | Impacto Estimado |
| --- | --- |
| Utilización de capacidad | Incremento del 15-25% mediante pickups dinámicos en ruta |
| Tiempo de respuesta a peticiones | Reducción de horas a segundos mediante automatización |
| Precisión en pricing | Eliminación de subcotizaciones mediante cálculo basado en datos reales |
| Cumplimiento normativo | Reducción de infracciones por exceso de tiempo de conducción |
| Satisfacción del cliente | Transparencia total en tracking y tiempos de entrega |

### 1.5 Diferenciación Competitiva

Mientras que existen soluciones parciales (plataformas de freight matching, sistemas de tracking, optimizadores de ruta), **ninguna integra de forma nativa la optimización física de carga (3D) con la toma de decisiones dinámica en ruta y el pricing inteligente**. Esta triple integración es el núcleo diferenciador del proyecto.

---

## 2. Visión General del Sistema

### 2.1 Problemas de Optimización Acoplados

El sistema debe resolver tres problemas de optimización que ocurren **mientras el camión está en ruta**:

| Problema | Qué implica | Complejidad |
| --- | --- | --- |
| **Tracking + Ruta** | Saber dónde está cada camión, recalcular rutas en tiempo real considerando restricciones de camión (altura, peso, ejes) y tráfico | Media |
| **Pickup & Delivery Dinámico** | Aceptar nuevas cargas que aparecen mientras el camión ya viaja de A a B, decidir si caben en ruta, tiempo y capacidad | Alta |
| **Acomodo físico de carga** | Calcular cómo se acomodan físicamente las cajas/paletas en el remolque para maximizar espacio y estabilidad | Muy alta |
| **Pricing dinámico** | Calcular precio de cada nueva carga basado en km reales, desvío, espacio ocupado, demanda del corredor y costos operativos | Alta |

### 2.2 Arquitectura: Microservicios

Un sistema de estas características no puede ser monolítico. La arquitectura separa dominios que escalan de forma independiente:

```javascript
┌─────────────────────────────────────────────────────────────┐
│                    CAPA DE PRESENTACIÓN                      │
│  Dashboard Web (React)  |  App del Conductor        │
│  Panel del Cliente              |  API para integraciones   │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│              CAPA DE LÓGICA DE NEGOCIO (Microservicios)      │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐   │
│  │  Routing    │ │  Load Opt.  │ │  Pricing Engine     │   │
│  │  Service    │ │  Service    │ │  (ML/Dynamic)       │   │
│  └─────────────┘ └─────────────┘ └─────────────────────┘   │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐   │
│  │  Tracking   │ │  Compliance │ │  Matching Engine    │   │
│  │  Service    │ │  (Tacógrafo)│ │  (Carga ↔ Camión)   │   │
│  └─────────────┘ └─────────────┘ └─────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│              CAPA DE DATOS Y PROCESAMIENTO                   │
│  Time-Series DB (GPS) | SQL (vehículos, rutas) | Cache     │
│  Message Queue (RabbitMQ) para eventos en tiempo real │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│              CAPA DE ADQUISICIÓN (Edge/IoT)                  │
│  GPS Tracker 4G/5G | OBD-II/CAN bus | Smart Tachograph      │
│  Sensores de puerta, temperatura, peso por eje               │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Tracking en Tiempo Real por Europa

### 3.1 Hardware y Conectividad

- **GPS + Telemetría**: Dispositivos OBD-II o CAN bus que transmiten cada 30-60 segundos vía 4G/5G. En 2026, el 52% del mercado de telemetría en Europa utiliza conectividad LTE.
- **Smart Tachograph v2**: Obligatorio en vehículos nuevos desde agosto de 2023. Registra automáticamente tiempos de conducción, descanso y posición vía GPS, y permite control en carretera por parte de las autoridades.
- **Protocolos**: MQTT para telemetría (ligero, ideal para conexiones intermitentes) o HTTP/REST para sincronización batch.

### 3.2 Datos a Rastrear

| Dato | Fuente | Frecuencia |
| --- | --- | --- |
| Posición GPS | Tracker GPS | 30-60 segundos |
| Velocidad, RPM, consumo | OBD-II/CAN | 60 segundos |
| Estado del tacógrafo | Smart Tachograph | Eventos |
| Peso por eje | Sensores de balanza | Cada parada |
| Apertura de puertas | Sensores magnéticos | Eventos |
| Temperatura (frigoríficos) | Sensores IoT | 5 minutos |

---

## 4. Optimización de Rutas: VRP Dinámico

### 4.1 Naturaleza del Problema

El caso de uso es un **Dynamic Vehicle Routing Problem with Pickup and Delivery (D-VRP-PD)**: el camión ya tiene una ruta planificada, pero aparecen nuevas peticiones de recogida y entrega en tiempo real. El sistema debe decidir si aceptarlas, qué camión es el más adecuado y cómo reordenar las paradas.

### 4.2 Algoritmos Recomendados

| Enfoque | Cuándo usarlo | Complejidad |
| --- | --- | --- |
| **ALNS (Adaptive Large Neighborhood Search)** | Para re-optimizar rutas rápidamente cuando llega una nueva petición. Es el estándar en la industria logística. | O(n²) por iteración |
| **k-opt + Transferencias de nodos** | Para mejorar rutas locales (reordenar paradas entre camiones). | O(k·n) |
| **BRKGA (Biased Random-Key Genetic Algorithm)** | Para la optimización profunda offline (planificación del día siguiente). | Poblacional |
| **Approximate Dynamic Programming (ADP)** | Para decisiones en tiempo real con incertidumbre (¿aparecerá otra carga mejor más adelante?). | Alta |

### 4.3 Flujo de Decisión en Tiempo Real

```javascript
1. Nueva petición de carga llega al sistema
2. ¿Qué camiones están "cerca" del punto de recogida? (Geo-query espacial)
3. Para cada camión candidato:
   a. ¿Tiene capacidad de peso/volumen restante?
   b. ¿Tiene tiempo de conducción disponible (tacógrafo)?
   c. ¿El desvío de ruta es aceptable?
   d. ¿Cabe físicamente la carga en el espacio remanente?
4. Ejecutar ALNS para re-optimizar la ruta de los candidatos
5. Calcular precio de cada opción
6. Presentar opciones al cliente / asignar automáticamente
```

### 4.4 APIs de Mapas para Camiones en Europa

Para rutas de camiones es imprescindible usar APIs que consideren **restricciones de vehículo pesado**:

- **HERE WeGo Truck Routing**: Altura, peso, carga peligrosa, restricciones por país.
- **TomTom Truck Routing**: Muy usado en Europa, con datos de tráfico en tiempo real.
- **Google Maps Platform (Routes API)**: Soporta `vehicleType=TRUCK` con restricciones de peso/dimensiones.
- **OpenStreetMap + OSRM** (self-hosted): Opción open source, requiere curar datos de restricciones de camiones.

---

## 5. Optimización de Carga: 3D Bin Packing + Estabilidad

### 5.1 El Problema

No basta con saber que "caben 5 toneladas más"; hay que saber **dónde físicamente** se colocan las cajas para que:

- No se dañen (fragilidad, "this side up")
- El peso se distribuya correctamente (centro de gravedad)
- Se pueda descargar primero lo que se entrega primero (LIFO/sequencing)
- Se respeten las restricciones de apilamiento

### 5.2 Modelo Matemático

Es una variante del **3D Bin Packing Problem (3D-BPP)** con restricciones adicionales:

**Entradas:**

- Vehículo: largo × ancho × alto, peso máximo, distribución por eje
- Cargas: dimensiones, peso, fragilidad, orientaciones permitidas, punto de descarga (orden), apilable sí/no

**Salida:**

- Posición (x, y, z) de cada caja dentro del remolque
- Orientación (hasta 6 rotaciones posibles)
- Orden de carga/descarga
- Métricas: % de utilización, centro de masa, estabilidad

### 5.3 Algoritmos

| Algoritmo | Uso | Resultado |
| --- | --- | --- |
| **First Fit Decreasing (FFD)** | Respuesta rápida (< 1s) para decidir si una carga cabe | Sub-óptimo pero rápido |
| **GRASP + VND** | Para generar un plan de carga detallado cuando se confirma la petición | Buen balance calidad/velocidad |
| **Metaheurísticas híbridas** (Genético + Simulated Annealing) | Para la planificación óptima al inicio del día | Mejor calidad, más lento |

### 5.4 Consideraciones Prácticas

- **Secuenciación**: Si el camión va A → B → C, la carga para C debe ir cargada primero (atrás) y la de A última (adelante). Esto convierte el problema en un **3D-BPP con restricciones de precedencia**.
- **Centro de masa**: Un mal reparto de peso afecta el consumo de combustible y la seguridad. El algoritmo debe minimizar la distancia del centro de masa al centro geométrico.
- **Peso por eje**: En Europa, los límites de peso por eje son estrictos. El sistema debe calcular cómo el peso de la carga se transfiere a cada eje.

---

## 6. Cálculo de Precios Dinámico

### 6.1 Factores de Entrada

| Factor | Origen de datos |
| --- | --- |
| **Costo base del corredor** | Histórico + APIs de mercado (FreightPulse, etc.) |
| **Desvío de ruta** | Cálculo de km adicionales vs. ruta original |
| **Espacio ocupado** | % del volumen/peso total que consume la carga |
| **Urgencia / Time windows** | Cuánto reduce la flexibilidad operativa |
| **Capacidad remanente del camión** | Si va casi vacío, precio más bajo para llenar; si va casi lleno, premium |
| **Combustible** | Precio diario por país (varía mucho en Europa) |
| **Peajes** | Vignettes, Eurovignette, peajes en Francia/Italia/España |
| **Demanda del corredor** | Spot rates en tiempo real |

### 6.2 Modelo de Pricing

El enfoque más robusto según la literatura actual es:

1. **Predicción de demanda**: Modelos de ML (Gradient Boosting, XGBoost) que estiman la probabilidad de encontrar otra carga en el destino.
2. **Costo oportunidad**: Si acepto esta carga con desvío de 50 km, ¿pierdo la oportunidad de una mejor que aparecerá más adelante?
3. **Optimización de precio**: Resolver un problema de optimización (ej. programación lineal) que maximice el beneficio esperado:

```javascript
Precio = Costo_variable(km, peajes, combustible) 
       + Costo_oportunidad(desvío, tiempo) 
       + Margen_objetivo 
       ± Ajuste_demanda(corredor, temporada)
```

### 6.3 APIs de Mercado de Fletes

- **FreightPulse**: Datos de tarifas spot por corredor, congestión de puertos, precios de combustible.
- **Upply / IRU**: Índices de tarifas de transporte por carretera en Europa.

---

## 7. Regulaciones Europeas Clave

El sistema **debe** integrar estas reglas en el motor de optimización; de lo contrario, las rutas propuestas serán ilegales:

| Regla | Límite | Impacto en el sistema |
| --- | --- | --- |
| **Tiempo diario de conducción** | 9h (extendible a 10h solo 2 veces/semana) | El VRP debe calcular si el camión puede llegar a recoger y entregar dentro del tiempo restante |
| **Descanso tras 4.5h** | 45 min (puede dividirse en 15+30) | La ruta debe incluir paradas obligatorias |
| **Descanso diario** | 11h (reducible a 9h máx. 3 veces/semana) | Si una entrega implica esperar 3h, ¿puede contar como descanso reducido? |
| **Tiempo quincenal** | Máx. 90h en 2 semanas | El sistema debe llevar un registro acumulativo por conductor |
| **Smart Tachograph v2** | Obligatorio en nuevos vehículos desde 2023 | El sistema puede leer datos del tacógrafo vía REMOTO (DSRC) para verificar tiempos en tiempo real |
| **Eurovignette / peajes** | Dependen del país y clase Euro del vehículo | El pricing debe incluir estos costos |

> **Nota importante**: A partir del 1 de julio de 2026, los tacógrafos serán obligatorios también en vehículos de >2.5 toneladas (antes era >3.5t) para transporte internacional.

---

## 8. Stack Tecnológico Recomendado

| Capa | Tecnologías                                                                           |
| --- |---------------------------------------------------------------------------------------|
| **Backend** | Java Spring boot para microservicios; Python para ML/optimización             |
| **Optimización de rutas** | OR-Tools (Google), Gurobi, CPLEX, o bibliotecas especializadas de VRP                 |
| **3D Bin Packing** | Implementación propia (GRASP/VND) o librerías especializadas                          |
| **Base de datos** | PostgreSQL (datos operativos) + TimescaleDB/InfluxDB (GPS time-series) + Redis (cache) |
| **Colas/Mensajería** | RabbitMQ para eventos entre microservicios                             |
| **Mapas/Rutas** | HERE Maps API, TomTom, o OSRM self-hosted                                             |
| **Frontend** | React/Vue con biblioteca de mapas (Mapbox GL JS, Leaflet)                             |
| **App conductor** | React Native con modo offline para zonas sin cobertura                       |
| **ML/Forecasting** | Python + scikit-learn / XGBoost / Prophet para predicción de demanda                  |

---

## 9. Desafíos y Consideraciones Estratégicas

### 9.1 Latencia Computacional

El cálculo de 3D bin packing + VRP no puede tardar más de 2-3 segundos cuando llega una petición. **Estrategia**: pre-calcular "plantillas de carga" y usar heurísticas rápidas (FFD) para la decisión inicial; metaheurísticas solo para la confirmación definitiva.

### 9.2 Conectividad

En rutas europeas (especialmente Alpes, zonas rurales de Francia/Alemania), la cobertura 4G falla. La app del conductor debe funcionar **offline** y sincronizar al recuperar señal.

### 9.3 Multi-país

Cada país tiene peajes diferentes, restricciones de circulación (ej. prohibición de camiones los domingos en Alemania/Austria), normas de peso. El motor de rutas debe cargar estas restricciones por país.

### 9.4 Integración con Tachograph

Integrar con el smart tachograph para leer tiempos restantes de conducción en tiempo real es complejo pero necesario para cumplir la normativa.

### 9.5 Competencia

El mercado europeo de logística es enorme, pero ya existen actores establecidos (Sennder, FreightHub, Ontruck). El diferenciador debe ser precisamente la **optimización de carga física + pricing dinámico en ruta**.

---

## 10. Resumen de Fases del Proyecto

| Fase | Alcance | Duración Estimada |
| --- | --- | --- |
| **Fase 1: MVP Tracking** | GPS en tiempo real + dashboard básico + integración tacógrafo | 2-3 meses |
| **Fase 2: Routing** | VRP estático + API de mapas con restricciones de camión | 2-3 meses |
| **Fase 3: Pickup Dinámico** | VRP dinámico + matching engine carga-camión | 3-4 meses |
| **Fase 4: 3D Load Optimization** | Bin packing con secuenciación + centro de masa | 4-6 meses |
| **Fase 5: Pricing Engine** | Modelo ML de demanda + pricing dinámico | 3-4 meses |
| **Fase 6: Escalado** | Multi-flota, multi-país, integraciones con TMS/ERP | Continuo |

---

*Documento generado para planificación estratégica. No incluye código ni implementación técnica.*