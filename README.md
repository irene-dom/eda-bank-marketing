# EDA · Campaña de telemarketing de un banco portugués

Proyecto 4 del Bootcamp Data & Analytics (ThePower): análisis exploratorio con **Python y pandas**.

## Objetivo
Entender **qué clientes contratan un depósito a plazo** tras una campaña de telemarketing y qué factores influyen: perfil del cliente, forma de contacto, historial con el banco y contexto económico.

## Estructura del repositorio
```
├── data/
│   ├── raw/
│   │   ├── bank-additional.csv        # contactos de la campaña (43.000 filas)
│   │   └── customer-details.xlsx      # datos de clientes, una hoja por año de alta
│   └── processed/
│       └── bank_clientes_limpio.csv   # dataset limpio y unido (43.000 × 39)
├── notebooks/
│   └── eda_bank_marketing.ipynb       # todo el análisis, ejecutado
├── img/                               # gráficos del análisis
└── README.md
```

## Datos
| Fuente | Contenido |
|---|---|
| `bank-additional.csv` | Perfil del cliente (edad, profesión, estado civil, estudios, impagos, hipoteca, préstamo), datos del contacto (canal, duración, nº de llamadas, contactos previos y su resultado), indicadores económicos (euríbor, IPC, confianza del consumidor, empleo), fecha y la variable objetivo **`y`** (¿contrata?). |
| `customer-details.xlsx` | Ingresos, hijos pequeños y adolescentes en casa, fecha de alta y visitas web al mes. Son tres hojas (2012, 2013 y 2014). |

## Proceso
### 1. Carga
- `pd.read_csv` para la campaña.
- `pd.read_excel(sheet_name=None)` para leer las tres hojas de clientes, que se unen con `pd.concat` añadiendo la columna `anio_alta`.

### 2. Exploración inicial
`shape`, `info()`, nulos por columna (número y %), duplicados y valores únicos de las categóricas. No hay duplicados.

### 3. Limpieza y transformación
| Problema | Solución |
|---|---|
| Columna `Unnamed: 0` (índice antiguo) | Eliminada |
| Nombres con puntos (`emp.var.rate`) | Cambiados a guion bajo (`emp_var_rate`) |
| Números como texto con coma decimal (`euribor3m`, `cons.price.idx`, `cons.conf.idx`, `nr.employed`) | Convertidos a `float` con una función propia |
| Fecha en texto con el mes en español (`2-agosto-2019`) | Convertida a `datetime` con un diccionario de meses; se crean `contact_year` y `contact_month` |
| Mayúsculas y minúsculas mezcladas en las categóricas | Todo a minúsculas |
| Nulos en `job`, `marital`, `education` | Categoría `unknown` |
| `default`, `housing`, `loan` como 0/1 con nulos | Pasadas a `no` / `yes` / `unknown` |
| 12 % de nulos en `age` | Imputados con la mediana de la profesión, con una columna `age_imputada` que los marca |
| 21 % de nulos en `euribor3m` (y algunos en `cons_price_idx`) | Imputados con la mediana de su periodo económico (`emp_var_rate` + `nr_employed`) |
| Columnas nuevas | `y_num`, `grupo_edad`, `contactado_antes`, `duracion_min`, `llamadas_campania`, `hijos_total`, `tramo_ingresos` y `tramo_euribor` |

### 4. Unión
Primero se comprobó que `id_` (campaña) e `ID` (clientes) son la misma clave: los 43.000 contactos tienen su cliente. Después se hizo un `merge` *inner* con `validate="one_to_one"`. Quedan fuera 170 clientes que no fueron contactados.

### 5. Análisis descriptivo
- Estadísticos (media, mediana, desviación, mínimo y máximo) de las variables numéricas.
- Tasa de conversión global y por segmento, con una función reutilizable (`conversion_por`) basada en `groupby` + `agg`.
- Correlaciones con la variable objetivo.

### 6. Visualización
Histograma, boxplot, gráficos de barras de conversión por segmento, evolución mensual y mapa de calor de correlaciones (matplotlib y seaborn).

## Resultados

![Conversión por profesión y edad](img/conversion_profesion_edad.png)
![Conversión según la campaña anterior y el mes](img/conversion_campania_mes.png)
![Conversión según año de alta y euríbor](img/conversion_alta_euribor.png)

### Conclusiones
La campaña tuvo una **tasa de conversión del 11,3 %**: 4.844 contrataciones en 43.000 contactos.

1. **El historial con el cliente es lo que más pesa:** quien contrató en una campaña anterior vuelve a contratar en el **65 %** de los casos, frente al 8,8 % de quien nunca había sido contactado.
2. **Los clientes recientes son más receptivos:** los dados de alta en 2014 convierten un **23 %**, frente al 7,7 % (2013) y el 4,6 % (2012).
3. **El perfil por edad y profesión tiene forma de U:** los mayores de 65 (46 %), los menores de 25 (24 %), los estudiantes (31 %) y los jubilados (25 %) son los que más contratan. Las franjas de 35 a 54 años y los perfiles *blue-collar* (6,9 %) los que menos.
4. **El contexto económico importa:** con el euríbor por debajo del 1 % la conversión llega al **46 %**; por encima del 3 % baja al 4,8 %.
5. **El canal y la insistencia:** el móvil convierte casi el triple que el fijo (14,7 % frente a 5,2 %). A partir de la tercera llamada la conversión cae (5,5 % con seis o más).
6. **La duración de la llamada** es la variable más correlacionada con el éxito (0,41), pero solo se conoce después de la llamada: describe, no predice.
7. **No marcan diferencias:** los ingresos, el número de hijos, tener hipoteca o préstamo, ni el mes del contacto.

**Recomendación:** priorizar a los clientes con éxito previo, los de alta reciente y los estudiantes y jubilados; contactar por móvil; no pasar de tres llamadas; e intensificar la campaña cuando los tipos de interés están bajos.

### Limitaciones
- La edad se imputó en un 12 % de los registros y el euríbor en un 21 %.
- Las fechas de contacto y los indicadores económicos no siempre son coherentes entre sí: el dataset parece modificado con fines didácticos.
- El análisis es descriptivo: muestra relaciones, no causas.

## Cómo ejecutarlo
```bash
pip install pandas numpy matplotlib seaborn openpyxl
```
Abre `notebooks/eda_bank_marketing.ipynb` en VS Code o Jupyter y ejecuta todas las celdas. Las rutas son relativas a la carpeta `notebooks/`.

## Autora
Irene Domínguez
