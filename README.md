# Pronóstico de ventas para la planificación del abastecimiento — Asertia Comercial

Código y datos del proyecto Capstone **«Modelo predictivo de ventas para la planificación del abastecimiento en una empresa distribuidora de productos de consumo masivo»**, de la Maestría en Inteligencia de Negocios y Ciencia de Datos de la Universidad de Las Américas (UDLA).

- **Autor:** Michael Wladimir Romero Nole
- **Tutor:** Manuel Eugenio Morocho Cayamcela
- **Empresa:** Asertia Comercial, línea Ferrero (8 agencias en Ecuador)

## De qué trata

Asertia Comercial planifica el abastecimiento de la línea Ferrero con el promedio de ventas de los meses anteriores y el ajuste manual de cada agencia. El resultado es sobre stock en algunas agencias y quiebres en otras. El proyecto construye un modelo que pronostica las unidades display mensuales por **agencia, grupo de cliente y marca** (297 series) y lo compara con la regresión lineal, con XGBoost y con reglas simples (ingenuo, ingenuo estacional y promedio móvil).

Resumen de resultados (prueba de mayo a agosto de 2026, 1 188 observaciones, pronóstico a un mes):

| Modelo | WAPE |
|---|---|
| Ingenuo (mes anterior) | 0,477 |
| Ingenuo estacional | 0,433 |
| Promedio móvil de 3 meses | 0,429 |
| Random Forest con rezago de 12 meses | 0,399 |
| Regresión lineal con rezago de 12 meses | 0,399 |

Random Forest reduce el error del pronóstico ingenuo en alrededor de 16 %, pero no se distingue de la regresión lineal con la misma información. El error baja al agregar: 0,30 por agencia y marca y 0,10 en el total nacional. El detalle, con sus tablas y figuras, está en el cuaderno y en el documento de la tesis.

## Contenido del repositorio

| Archivo | Descripción |
|---|---|
| `Pronostico_Ventas_Asertia_Capstone.ipynb` | Cuaderno de Google Colab con todo el análisis, ya ejecutado (tablas y figuras incluidas). |
| `panel_mensual.csv` | Panel mensual agregado: 297 series × 32 meses = 9 504 filas. |
| `meses_con_compra_cliente_marca.csv` | Cuántos pares cliente-marca compraron en 1, 2, … 32 meses (para la Figura 5). |
| `precios_medios_2026.csv` | Precio medio por unidad display de cada marca en 2026 (para valorar el error en dólares). |
| `Herramienta_Pronostico_Asertia.xlsx` | Libro de Excel con el pronóstico de septiembre a noviembre de 2026 por agencia y marca. |

### Diccionario de `panel_mensual.csv`

| Columna | Descripción |
|---|---|
| `agencia` | Ambato, Cuenca, Guayaquil, Ibarra, Lago Agrio, Manta, Quito o Santo Domingo. |
| `grupo` | Grupo de cliente: Cobertura, Retail, Mayorista, Subdistribuidor o Farmaenlace. |
| `marca` | Una de las 9 marcas de la línea Ferrero. |
| `mes` | Primer día del mes (enero de 2024 a agosto de 2026). |
| `unidades` | Unidades display netas del mes (devoluciones compensadas; los netos negativos se llevaron a cero). |
| `venta_neta` | Venta neta del mes en dólares. |

Los meses sin venta de una serie están completados con cero (15,9 % de las observaciones).

### Sobre los datos originales

La extracción de JD Edwards (1 248 487 líneas de factura de enero de 2024 a agosto de 2026, con nombres de clientes y vendedores) es de uso interno de la empresa y **no se publica**. Solo se publican datos agregados por agencia, grupo de cliente, marca y mes, sin información de clientes ni de vendedores. La primera parte del cuaderno documenta cómo se construyó el panel a partir de esa extracción y solo se ejecuta si los archivos originales están disponibles.

## Cómo ejecutar el cuaderno

**En Google Colab**

1. Sube la carpeta del repositorio a tu Google Drive (o abre el cuaderno desde GitHub con *Archivo > Abrir cuaderno > GitHub*).
2. En la sección 1.2 del cuaderno elige cómo se leen los datos. Opción A: montar Drive (descomenta `drive.mount` y ajusta la ruta de la carpeta). Opción B: ruta local. Opción C: descargar los CSV directo desde GitHub con la dirección raw de este repositorio.
3. Ejecuta *Entorno de ejecución > Ejecutar todas*. Tarda entre 15 y 25 minutos, sobre todo por la búsqueda de hiperparámetros del apartado 4.3.

Al terminar, el cuaderno crea las carpetas `figuras/` (las 12 figuras en PNG) y `salidas/` (las tablas en CSV y el libro de Excel).

**En local**

```bash
pip install numpy pandas scipy scikit-learn xgboost statsmodels matplotlib openpyxl jupyter
jupyter notebook Pronostico_Ventas_Asertia_Capstone.ipynb
```

## Estructura del cuaderno

| Sección | Contenido | Tablas y figuras de la tesis |
|---|---|---|
| 1 | Carga de datos, librerías y estilo de figuras | |
| 2 | Limpieza y construcción del panel mensual | Tabla 3 |
| 3 | Análisis exploratorio y pruebas (Dickey-Fuller, RESET, Spearman) | Tablas 4 y 5, Figuras 1 a 5 |
| 4 | Selección de variables, hiperparámetros y entrenamiento | Tabla 6 |
| 5 | Resultados: modelos, rezago de 12 meses, horizontes, agregación, segmentos y réplica del benchmark | Tablas 7 a 10, Figuras 6 a 10 |
| 6 | Interpretación: bootstrap, Wilcoxon, semillas, reentrenamiento mensual, importancia por permutación | Tabla 11, Figura 11 |
| 7 | Implicaciones: valor del error, pronóstico de septiembre a noviembre y libro de Excel | Figura 12, Tablas 12 y 13 |

## Decisiones de diseño

- **Sin fuga de información.** La división es cronológica (entrenamiento hasta diciembre de 2025, validación de enero a abril de 2026, prueba de mayo a agosto de 2026). Los rezagos y el promedio móvil usan solo meses anteriores, y la estandarización de la regresión lineal se ajusta con los datos de entrenamiento.
- **Hiperparámetros solo con validación.** Con los valores elegidos, cada modelo se reajusta con entrenamiento y validación y se evalúa una única vez en la prueba.
- **Modelos directos.** Para cada horizonte (1, 2 y 3 meses) se entrena un modelo distinto, con los rezagos calculados desde ese horizonte.
- **Rezago de 12 meses.** Se probó después de ver el desempeño de la especificación original en la prueba, así que su resultado debe leerse con esa reserva. La evidencia principal es la validación.
- **Semilla fija** (`SEED = 42`). Con otras semillas, el WAPE de Random Forest a un mes varía entre 0,397 y 0,401.

## Versiones probadas

Python 3.13, numpy 2.5, pandas 3.0, scikit-learn 1.9, xgboost 3.4, statsmodels 0.15, scipy 1.18, matplotlib 3.11 y openpyxl 3.1. En Colab funciona con las versiones que trae por defecto. Cambios de versión pueden mover el tercer decimal de los modelos con componente aleatorio.

## Cita

Romero Nole, M. W. (2026). *Modelo predictivo de ventas para la planificación del abastecimiento en una empresa distribuidora de productos de consumo masivo* [Proyecto Capstone, Maestría en Inteligencia de Negocios y Ciencia de Datos]. Universidad de Las Américas.
