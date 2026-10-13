# Segmentación semántica de deslizamientos con Sentinel-2 · La Convención, Cusco

Sistema de segmentación semántica basado en imágenes satelitales **Sentinel-2** para identificar zonas de deslizamiento a nivel de píxel en la provincia de **La Convención (Cusco, Perú)**, comparando las arquitecturas **U-Net** y **DeepLabV3+**.

> **Estado del proyecto:** en desarrollo (tesis de pregrado, UPCH). Objetivos OE1 (análisis) y OE2 (diseño) avanzados; OE3 (desarrollo) y OE4 (validación) pendientes. Este repositorio se irá completando conforme avance el desarrollo. Lo marcado como *por definir* aún no está decidido.

## Autores

- Mejia Lopez, Leonardo Camilo
- Huarcaya Pumacayo, Victor Nikolai

**Asesor:** Mg. Roy Marco Yali Samaniego · Universidad Peruana Cayetano Heredia (UPCH), Ciencias e Ingeniería

## Problema

En el Perú hay 264 distritos en riesgo muy alto de deslizamientos y el método vigente de mapeo, la fotointerpretación manual, toma de días a semanas por evento y no entrega métricas cuantitativas. La Convención suma lluvias de más de 2 000 mm anuales, suelos arcillosos saturables y alta nubosidad.

## Objetivos

| | Objetivo |
|---|---|
| **OE1** | Analizar los hallazgos previos de modelos de IA para la detección de deslizamientos en imágenes satelitales mediante el benchmarking, métricas y limitaciones, para establecer los requisitos técnicos de la arquitectura. |
| **OE2** | Diseñar la arquitectura del modelo de segmentación semántica identificada a partir del benchmarking, para definir el modelo de mayor precisión respecto del área analizada. |
| **OE3** | Desarrollar un sistema de segmentación semántica con módulos de adquisición, preprocesamiento e inferencia que genere máscaras binarias de deslizamiento. |
| **OE4** | Validar el desempeño del sistema a partir de IoU, F1-Score y tiempo de inferencia sobre un conjunto de datos independiente. |

## Cómo funciona

El proyecto separa dos sistemas que solo se conectan por el archivo de pesos (`.pt`):

1. **Artefacto (inferencia):** el usuario ingresa un área y fechas → el sistema adquiere escenas Sentinel-2 L2A desde Google Earth Engine, aplica Cloud Score+, calcula NDVI y NDWI, divide en parches, predice con el modelo y exporta una máscara binaria **GeoTIFF (EPSG:32718)**.
2. **Entrenamiento (fuera de línea, Google Colab Pro):** construye el dataset, entrena y compara U-Net y DeepLabV3+, y guarda el mejor checkpoint por IoU de validación.

```
Usuario → App web → API → Adquisición (GEE + Cloud Score+) → Preprocesamiento
        → Inferencia (.pt) → Exportación → GeoTIFF (QGIS)
```

### Parámetros de diseño

| Elemento | Valor |
|---|---|
| Entrada | 6 canales: 4 bandas espectrales + NDVI + NDWI |
| Parches | 256×256 px, solape 50 % |
| Filtro de nubes | Cloud Score+ ≥ 0.60 a nivel de píxel |
| Función de pérdida | BCE-Dice ponderada |
| Optimizador | AdamW (hiperparámetros por definir) |
| Selección de modelo | Mejor IoU de validación |
| Salida | GeoTIFF EPSG:32718 |

## Metas de desempeño

| Métrica | Meta |
|---|---|
| IoU | ≥ 0.70 |
| F1-Score | > 75.5 % |
| Recall | > 85 % |
| Falsos positivos | < 15 % |
| Tiempo de inferencia | < 1.5 s por parche de 256×256 px |

Los resultados se reportarán por separado para datos internacionales (Landslide4Sense) y datos locales (La Convención).

## Datos

| Fuente | Uso | Acceso |
|---|---|---|
| Sentinel-2 L2A (`COPERNICUS/S2_SR_HARMONIZED`) | Imágenes de entrada | Google Earth Engine, licencia CC BY |
| [Landslide4Sense](https://zenodo.org/records/10463239) | Benchmark internacional (3 799 parches con máscaras, 14 bandas, 128×128 px) | Público, CC BY 4.0 |
| SINPAD - INDECI | Inventario de eventos para la verdad de terreno local (consulta puntual) | Autorización en trámite |
| Landslide4SensePe | Dataset local propio (aporte del proyecto) | En construcción |

La partición entrenamiento/validación/prueba se hace **por zona geográfica**, no por parche.

## Estructura prevista del repositorio

```
.
├── docs/                 # Memoria descriptiva, diagramas y presentación
├── src/
│   ├── adquisicion/      # Cliente de Google Earth Engine, Cloud Score+
│   ├── preprocesamiento/ # NDVI/NDWI, normalización, parches
│   ├── entrenamiento/    # U-Net, DeepLabV3+, pérdida, checkpoints
│   ├── inferencia/       # Carga de pesos, reensamblado, binarización
│   ├── exportacion/      # GeoTIFF EPSG:32718
│   └── api/              # API de orquestación
├── tests/
├── configs/
├── requirements.txt
└── README.md
```

> Estructura propuesta; se ajustará durante el OE3.

## Instalación y uso

*Por definir en el OE3.* Se completará con las versiones de Python y librerías, la autenticación de Google Earth Engine, la descarga de los datos y los comandos de entrenamiento e inferencia.

## Hoja de ruta

- [x] OE1: revisión sistemática (PRISMA 2020) y requisitos
- [x] OE2: diseño de arquitectura (C4, vistas lógica y física, pipeline)
- [ ] OE3: desarrollo de módulos, dataset local y entrenamiento
- [ ] OE4: validación con conjunto independiente

## Alcance

El sistema segmenta zonas de deslizamiento en La Convención. **No** clasifica el tipo de movimiento ni su severidad, no se valida en otras provincias, no usa otras fuentes satelitales (Landsat, SAR) y no evalúa en esta fase otras arquitecturas (SegFormer, Vision Transformers). Es una herramienta de apoyo para priorizar inspecciones y **no reemplaza la evaluación técnica de campo**.

## Cómo citar los datos

Si usas Landslide4Sense, cita: Ghorbanzadeh et al., *Landslide4Sense: Reference Benchmark Data and Deep Learning Models for Landslide Detection*. [arXiv:2206.00515](https://arxiv.org/abs/2206.00515)

## Licencia

*Por definir.*
